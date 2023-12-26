---
title: 2023.5.2 - 2023.5.5
date: 2023-05-03 18:56:24
password: 1062597351
tags:
---

> - 工作相关：Upstream data extracts（关于性能优化和代码重构的踩坑记录和心得体会）
> - 博客更新：[Java Language](../../../../2022/05/08/java-language/)

### 踩坑记录

- 令 REST 接口的返回形式为 JSON

  ```java
  @GetMapping(path = "xxx", produces = MediaType.APPLICATION_JSON_VALUE)
  ```

- 切面方法不能被声明为泛型方法，泛型切面方法会导致切面不生效

- 切面方法获取拦截方法的返回值，只能是引用类型，不能使用基本类型

- 子类的类注解所开启的切面是不生效的，可以改用子类的方法注解来开启切面

- 解析 excel 时，先判断单元格的类型（数字还是字符串），再获取单元格的内容

- 当需要执行的 SQL 逻辑比较简单时，倾向于直接使用 JPQL，而非 Native SQL

- `@Transactional`注解的标注位置应该尽可能小，不需要令事务包含不必要的逻辑范围

- 在等待多个异步方法的返回结果时，注意不要在某个方法抛出异常后就直接结束`for`循环了，否则可能在异步方法没有全部返回时就执行了后面的逻辑

- 有前端传输文件至后端处理时，如果后端希望进行异步处理，则应在先将文件转换成一个临时文件保存在本地机器，再返回请求结果，否则请求一旦返回，接口所接收到的文件会变为 null 或 empty

- 多线程开发时，对于已经在某个线程做出处理的异常，当该异常被传递回主线程时，无需再做出重复处理

- 在 Java 中通过获取数据库连接来执行原生 SQL 时，无法执行`truncate`（会找不到表），需要改成`delete`，感觉像是无法执行 DDL，只能执行 DML

- 由于不同客户端发出的请求的时区不同，根据时间排序来获取最新请求记录不一定准确，可以改为根据自增列排序来获取最新数据记录

- 当`Map`的键为`Enum`时，使用`EnumMap`而不是`HashMap`


### 经验体会

针对多个不同的数据来源，但是基本一致的落库规则，重构时大量使用了泛型和模板方法模式，最终目的都是将通用的一致的逻辑抽象出来，尽可能减少重复的逻辑编写。

针对与解析和落库主流程无关的逻辑（比如邮件通知，操作记录，节点定时任务的触发等）通过编写切面来完成，避免不同逻辑的代码混杂，简单的旁置逻辑直接放到切面，复杂的旁置逻辑单独写一个`Service`，并在切面中调用该`Service`。

当需要操作的数据量很大时，直接执行原生的 SQL 比使用 JPA 接口要快，批量 insert 语句和 merge into 语句比批量 update 语句要快，所以可以先往一张空表里 insert，再使用 merge into，来间接实现真正需要更新的表的 update。

在做 crud 时，先明确需求的最终目的是什么，目的是查询（把数据从数据库转移到应用），则倾向于由应用程序来实现数据处理等逻辑，若目的是落库（把数据从应用转移到数据库，尤其是数据量很大时），则倾向于直接执行原生 SQL 来批量进行数据处理等逻辑。

永远不要预设对方应该做出合规的行为，而是要尽可能增强自身代码对各种情况的处理，尽可能最大程度地保留对别人的不信任感。

重复的工作虽然可以直接照着做不用动脑，但实在无趣，不如想办法从看似重复的工作中挖掘出有意思的东西，比如建立漂亮的代码结构。

不必记住，但要见过。

### 代码附录

```sql
UPDATE TABLE T1 SET (COL1, COL2) = (SELECT 'VAL1', 'VAL2' FROM DUAL) WHERE ...;

UPDATE TABLE T1 SET (COL1, COL2) = (SELECT COL1, COL2 FROM T2 WHERE T1.ID = T2.ID) WHERE ...;

MERGE INTO TABLE T1 dest USING (SELECT * FROM T2) source ON dest.ID = source.ID 
WHEN MATCHED THEN UPDATE SET dest.COL1 = source.COL1, dest.COL2 = source.COL2
WHEN NOT MATCHED THEN INSERT (COL1, COL2) VALUES (source.COL1, source.COL2);

INSERT INTO T1 (COL1, COL2) VALUES ('VAL1', 'VAL2');

INSERT INTO T1 (COL1, COL2) SELECT COL1, COL2 FROM T2 WHERE ...;

INSERT ALL
INTO T1 (COL1, COL2) VALUES ('VAL1', 'VAL2')
INTO T2 (COL1, COL2) VALUES ('VAL1', 'VAL2')
...
SELECT 1 FROM DUAL;
```

```java
@Component
public class XLSXParseStrategy extends ParseStrategy {

    @Override
    protected Iterator<Row> parseFile(Object f) throws Exception {
        MultipartFile file = (MultipartFile) f;
        File tempFile = null;
        try {
            String dir = "xxx";
            File tempDir = new File(dir);
            if (!tempDir.exist() || !tempDir.mkdirs) {
                throw new Exception(String.format("failed to create tempDir %s", dir));
            }
            tempFile = File.createTempfile("prefix", "suffix", tempDir);
            file.transferTo(tempFile);
            FileInputStream fis = new FileInputStream(tempFile);
            return new XSSFWorkBook(fis).getSheetAt(0).iterator();
        } catch (Exception e) {
            throw new Exception(String.format("CSV file parse failed - %s", e.getMessage()), e);
        } finally {
            if (null != tempFile) {
                try {
                    tempFile.delete();
                } catch (Exception e) {
                    throw new Exception(String.format("Error encountered when deleting temporary file - %s", e.getMessage()), e);
                }
            }
        }
    }

    @Override
    protected boolean hasNext(Object content) throws Exception {
        Iterator<Row> iterator = (Iterator<Row>) content;
        return iterator.hasNext();
    }

    @Override
    protected String[] getRowData(Object content) throws Exception {
        Iterator<Row> iterator = (Iterator<Row>) content;
        return iterator.next();
    }

    @Override
    protected Long getRowNum(Object content) {
        // ...
    }

    @Override
    protected boolean checkIfDataRow(Object rowData) {
        // ...
    }

    @Override
    protected void parseDataRow(Long rowNum, Object rowData) {
        Row row = (Row) rowData;
        Cell cell = row.getCell(index);
        String value;
        if (cell.getCellType == 1) {
           value = cell.getStringCellValue();
        }
        // ...
    }

}
```

