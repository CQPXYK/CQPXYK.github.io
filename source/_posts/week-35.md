---
title: 2023.4.24 - 2023.4.28
date: 2023-04-27 20:31:52
password: 1062597351
tags:
---

> - 工作相关：Upstream data extracts (tech enhancement 收尾、code review、编码习惯、思维习惯)
> - 博客更新：[Java Language](../../../../2022/05/08/java-language/)

### Tech Enhancement

上游提供的数据有两种文件格式，csv 和 xml，之前一直是用解析的 csv 文件，但是考虑到 xml 文件中标明各个字段名称，而csv 没有，且使用`CsvReader`读取比较大的 csv 文件时可能会漏读一些行，原因还没找到，决定改成解析 xml 文件，但是并不删除用于解析 csv 的相关代码，而是结合模板方法模式和策略模式，来根据传入的文件格式自动选择相应的解析策略，大致的解析代码如下

```java
@Service
public class ParseService {

    private final Map<String, ParseStrategy> strategySelector = new HashMap<>();

    @Autowired
    private CSVParseStrategy csvParseStrategy;

    @Autowired
    private XMLParseStrategy xmlParseStrategy;

    @PostConstruct
    public void init() {
        strategySelector.put("out", csvParseStrategy);
        strategySelector.put("xml", xmlParseStrategy);
    }

    public void parseRecords(MultipartFile file) throws Exception {
        if (null == file || file.isEmpty()) {
            throw new Exception("parseRecords failed due to file is null or empty");
        }
        String fileName = file.getOriginalFilename();
        if (StringUtils.isEmpty(fileName)) {
            throw new Exception("parseRecords failed due to file format cannot be obtained");
        }
        String fileType = fileName.substring(fileName.lastIndexOf(".") + 1);
        if (!strategySelector.containsKey(fileType)) {
            throw new Exception(String.format("parseRecords failed due to the lack of parse strategy for %s", fileName));
        }
        strategySelector.get(fileType).handle(file);
    }

}
```

```java
public abstract class ParseStrategy {

    public final void handle(Object f) throws Exception {
        Object content = parseFile(f);
        while (hasNext(content)) {
            Object rowData = getRowData(content);
            Long rowNum = getRowNum(content);
            if (checkIfDataRow(rowData)) {
                parseDataRow(rowNum, rowData);
            }
        }
    }

    protected abstract Object parseFile(Object f) throws Exception;

    protected abstract boolean hasNext(Object content) throws Exception;

    protected abstract Object getRowData(Object content) throws Exception;

    protected abstract Long getRowNum(Object content);

    protected abstract boolean checkIfDataRow(Object rowData);

    protected abstract void parseDataRow(Long rowNum, Object rowData);

}
```

```java
@Component
public class CSVParseStrategy extends ParseStrategy {

    @Override
    protected CsvReader parseFile(Object f) throws Exception {
        MultipartFile file = (MultipartFile) f;
        try (String feedData = IOUtils.toString(file.getInputStream(), Charset.defaultCharset());
             ByteArrayInputStream inputStream = new ByteArrayInputStream(feedData.getBytes())) {
            CsvReader csvReader = new CsvReader(inputStream, '|', Charset.defaultCharset());
            csvReader.setSafetySwitch(false);
            return csvReader;
        } catch (Exception e) {
            throw new Exception(String.format("CSV file parse failed - %s", e.getMessage()), e);
        }
    }

    @Override
    protected boolean hasNext(Object content) throws Exception {
        try {
            CsvReader csvReader = (CsvReader) content;
            return csvReader.readRecord();
        } catch (Exception e) {
            throw new Exception(String.format("CSV file parse failed - %s", e.getMessage()), e);
        }
    }

    @Override
    protected String[] getRowData(Object content) throws Exception {
        try {
            CsvReader csvReader = (CsvReader) content;
            return csvReader.getValues();
        } catch (Exception e) {
            throw new Exception(String.format("CSV file parse failed - %s", e.getMessage()), e);
        }
    }

    @Override
    protected Long getRowNum(Object content) {
        CsvReader csvReader = (CsvReader) content;
        return csvReader.getCurrentRecord() + 1;
    }

    @Override
    protected boolean checkIfDataRow(Object rowData) {
        String[] values = (String[]) rowData;
        return null != values && values.length != 0 && !"H".equals(values[0]) && !"T".equals(values[0]);
    }

    @Override
    protected void parseDataRow(Long rowNum, Object rowData) {
        // ...
    }

}
```

```java
@Component
public class XMLParseStrategy extends ParseStrategy {

    @Override
    protected Iterator<Element> parseFile(Object file) throws Exception {
        MultipartFile file = (MultipartFile) f;
        try (String feedData = IOUtils.toString(file.getInputStream(), Charset.defaultCharset());
             ByteArrayInputStream inputStream = new ByteArrayInputStream(feedData.getBytes())) {
            SAXReader reader = new SAXReader();
            reader.setFeature("http://apache.org/xml/features/disallow-doctype-decl", true);
            Document document = reader.read(inputStream);
            return document.getRootElement().elementIterator();
        } catch (IOException e) {
            throw new Exception(String.format("XML file parse failed - %s", e.getMessage()), e);
        }
    }

    @Override
    protected boolean hasNext(Object content) {
        Iterator<Element> iterator = (Iterator<Element>) content;
        return iterator.hasNext();
    }

    @Override
    protected Element getRowData(Object content) {
        Iterator<Element> iterator = (Iterator<Element>) content;
        return iterator.next();
    }

    @Override
    protected Long getRowNum(Object content) {
        return null;
    }

    @Override
    protected boolean checkIfDataRow(Object rowData) {
        Element element = (Element) rowData;
        return "row".equals(element.getName());
    }

    @Override
    protected void parseDataRow(Long rowNum, Object rowData) {
        Element element = (Element) rowData;
        Iterator<Element> childIterator = element.elementIterator();
        while (childIterator.hasNext()) {
            Element e = childIterator.next();
            String nodeName = StringUtils.trim(e.getName());
            String nodeValue = StringUtils.trim(e.getStringValue());
            // ...
        }
    }

}
```

### Code Review

- 使用`future`来等待异步执行的结构，避免使用`while`循环

- 使用`@EqualsAndHashCode`，避免编写大段`equals`和`hashCode`代码，`@EqualsAndHashCode`标注在类上，`@EqualsAndHashCode.Exclude`标注在字段上（题外话：没有找到合适的给 lombok 方法加切面的方式）

- 在使用 JPA 接口时，简单的 SQL 使用 JPQL 接口比原生 SQL 更好（原生 SQL 适用于复杂的 SQL 查询），官网查询关键字（supported keywords inside method names）

- 涉及到 IO 资源时使用`try-with-resource`

  

### 编码和思维习惯

- 代码结构在刚开始的时候总是比较简单，基本无分层，但会随着功能逐渐完善多样而需要重构为分层更加清晰的结构，每个类的功能逐渐细化的更加明确单一
- 对于一个 Service Bean 来说，除了注入的其他成员 Bean 以外，不要使用太多成员变量
  - 非静态变量可以合并为一个 DTO
  - 静态常量统一放到一个不可实例化（私有无参构造器）的 const 常量类中
  - 只保留非静态常量和静态变量（由于 Spring Bean 默认都是单例，静态变量的情况好像比较少见）
- 编写 Java Code 时需要结合考虑 DB 中的表结构表数据，结合实际业务来考虑数据处理功能（主要是约束和校验）放在哪里实现更加合适
- 测试性能时要考虑到 dev db 和 prod db 的数据量不同的因素
- 代码的每一处设计和编写都是有原因有理由的，不能凭空造方案
- 尽可能保留对外部数据的不信任感，在落库之前尽可能地增加外部数据的校验和预处理，尽可能做全面的容错处理
- 考虑尽可能多的场景去测试自己的代码，交付功能完善强大的代码

### 参考资料

- https://docs.spring.io/spring-data/jpa/docs/current/reference/html/#jpa.query-methods
- https://www.jianshu.com/p/9b3835299b99