---
title: 2023.3.20 - 2023.3.24
date: 2023-03-24 18:35:48
password: 1062597351
tags:
---

> - 工作相关：Upstream data extracts (数据获取、解析、落库）
> - 博客更新：[Java Language](../../../../2022/05/08/java-language/)

通过 SFTP 获取上游数据，分为 SSH 连接配置、数据的获取、解析及落库这几步。

### 数据获取

通过 sftp 获取 feed 文件有两种实现方式，一种是通过 shell 脚本，另一种是通过 Java code，sftp 连接及获取数据相关的脚本如下

```bash
#!/bin/sh
# FILE: run_autosys_jobs.sh

echo "`date +%Y-%m-%d` `date +%H:%M:%S`: >>>> ARG1: $1"
echo "`date +%Y-%m-%d` `date +%H:%M:%S`: >>>> ARG2: $2"

LOG_FILE=CSI\_`date +%Y%m%d%H%M%S`.txt
cat /dev/null > $LOG_FILE
./$1 $2 $LOG_FILE 2>&1 | tee -a $LOG_FILE
```



```bash
#!/bin/sh
# FILE: launchCSIBatchJob.sh

FILE="launchCSIBatchJob.sh"
ENV=$1
LOG_FILE=$2
sourceServerName=""
sourceFileLocation=""
sourceFileName=""
destLocation=""

checkParameter() {
		echo "`date +%Y-%m-%d` `date +%H:%M:%S`: >>>> FILE: $FILE"
		echo "`date +%Y-%m-%d` `date +%H:%M:%S`: >>>> ENV: $ENV"
		echo "`date +%Y-%m-%d` `date +%H:%M:%S`: >>>> LOG_FILE: $LOG_FILE"
		echo "`date +%Y-%m-%d` `date +%H:%M:%S`: >>>> sourceServerName: $sourceServerName"
		echo "`date +%Y-%m-%d` `date +%H:%M:%S`: >>>> sourceFileLocation: $sourceFileLocation"
		echo "`date +%Y-%m-%d` `date +%H:%M:%S`: >>>> sourceFileName: $sourceFileName"
		echo "`date +%Y-%m-%d` `date +%H:%M:%S`: >>>> destLocation: $destLocation"
		
		if [ -z $ENV ]; then
				echo "`date +%Y-%m-%d` `date +%H:%M:%S`: ENV Information can not be blank"
				exit 0
		fi
		case "$ENV" in
		"SIT")
				APIServer=""
				;;
		"UAT")
				APIServer=""
				;;
		"PROD")
				APIServer=""
				;;
		*)
				echo "`date +%Y-%m-%d` `date +%H:%M:%S`: ENV Information should be SIT, UAT or PROD"
				exit 0
				;;
		esac
		
		feedDataAPI=$APIServer/..
		startAPI=$APIServer/..
		completeAPI=$APIServer/..
		failureAPI=$APIServer/..
		echo "`date +%Y-%m-%d` `date +%H:%M:%S`: >>>> feedDataAPI: $feedDataAPI"
		echo "`date +%Y-%m-%d` `date +%H:%M:%S`: >>>> startAPI: $startAPI"
		echo "`date +%Y-%m-%d` `date +%H:%M:%S`: >>>> completeAPI: $completeAPI"
		echo "`date +%Y-%m-%d` `date +%H:%M:%S`: >>>> failureAPI: $failureAPI"
		echo ""
}

loggingStart() {
		echo -e "`date +%Y-%m-%d` `date +%H:%M:%S`: $FILE starts executing\n"
		curl -s -k $startAPI
}

prepareDestDir() {
		if [ ! -d "$destLocation" ]; then
				mkdir -p $destLocation;
				echo "`date +%Y-%m-%d` `date +%H:%M:%S`: $destLocation Directory creation done"
		fi
		if [ ! -d "$destLocation/archive" ]; then
				mkdir -p $destLocation/archive;
				echo "`date +%Y-%m-%d` `date +%H:%M:%S`: $destLocation/archive Directory creation done"
		fi
}

connectingSFTP() {
		echo "`date +%Y-%m-%d` `date +%H:%M:%S`: Connecting to SFTP"
		sftp -i /etc/secret-volume/ssh-privatekey $sourceServerName <<END_SCRIPT
			cd $sourceFileLocation
			lcd $destLocation
			get $sourceFileName
END_SCRITP
		if [ ! -f "$destLocation/$sourceFileName" ]; then
				echo -e "`date +%Y-%m-%d` `date +%H:%M:%S`: Get $sourceFileName Failed\n"
				loggingFailure
		else
				chmod 775 *.out
				chmod 775 *.xml
				echo -e "`date +%Y-%m-%d` `date +%H:%M:%S`: SFPT Connecting Completed\n"
		fi
}

processingFeed() {
		cd $destLocation
		
		# Invoking API
		echo "`date +%Y-%m-%d` `date +%H:%M:%S`: Processing $sourceFileName file"
		response=`curl -s -k -F 'file=@'"$sourceFileName"'' $feedDataAPI`
		echo "`date +%Y-%m-%d` `date +%H:%M:%S`: Response: $response"
		
		# Parse result
		success=${response#*"\"success\":"}
		success=${success%%,*}
		echo "`date +%Y-%m-%d` `date +%H:%M:%S`: Success: $success"
		
		if [[ $success == "true" ]]; then
				echo "`date +%Y-%m-%d` `date +%H:%M:%S`: Processing Completed"
				moveFeedFile
				cd ..
		else
				cd ..
				echo "`date +%Y-%m-%d` `date +%H:%M:%S`: Processing Failed"
				loggingFailure
		fi
}

moveFeedFile() {
		fileName=${sourceFileName%.*}
		fileType=${sourceFileName#*.}
		archiveName=$fileName\_`date +%Y%m%d%H%M%S`.$fileType
		mv $sourceFileName archive/$archiveName;
		echo -e "`date +%Y-%m-%d` `date +%H:%M:%S`: Previous Version $sourceFileName moved to archive/$archiveName\n"
}

loggingFailure() {
		echo -e "`date +%Y-%m-%d` `date +%H:%M:%S`: $FILE execution failed\n"
		cp -f $LOG_FILE log.txt
		curl -s -k -F `file=@log.txt` $failureAPI
		exit 0
}

loggingComplete() {
		echo -e "`date +%Y-%m-%d` `date +%H:%M:%S`: $FILE execution completed\n"
		cp -f $LOG_FILE log.txt
		curl -s -k -F `file=@log.txt` $completeAPI
		exit 0
}

###############################################################
# Main starts here
###############################################################
checkParameter
loggingStart
prepareDestDir
connectingSFTP
processingFeed
loggingComplete
```

一直抽不出大块时间去系统学 Linux，工作中需要写脚本时只能现查现用了

### 解析和落库

从 SpringMVC 获取到的文件是 MultipartFile 类型的，在获取到 feed 文件后，解析数据时 IO 相关的代码如下

```java
@Service
public class FeedDataService {

    @Autowired
    private InfoInventoryDao infoInventoryDao;

    @Transactional
    public void load(MultipartFile file) throws Exception {
        if (null == file || file.isEmpty()) {
            throw new Exception("feed file is null or empty");
        }
        load(file.getInputStream());
    }

    private void load(InputStream feedStream) {
        String feedData = IOUtils.toString(feedStream, CharSet.defaultCharset());
        ByteArrayInputStream inputStream = new ByteArrayInputStream(feedData.getBytes());
        CsvReader csvReader = new CsvReader(inputStream, '|', CharSet.defaultCharset());
        List<InfoInventory> inventoryList = new LinkedList<>();
        readRecords(csvReader, inventoryList);
        updateRecords(inventoryList);
    }

    private void readRecords(CsvReader csvReader, List<InfoInventory> inventoryList) {
        csvReader.setSafetySwitch(false);
        while (csvReader.readRecord()) {
            String[] fieldValues = csvReader.getValues();
            inventoryList.add(buildInfoInventory(fieldValues));
        }
    }

}
```

解析这块所涉及的 IO 相关的知识点也不太熟悉，这次就只是照着别人现成的代码抄，有空的时候要去系统学习下 IO 这块内容

在做落库时使用到了`@Transactional`注解，需要注意的是该注解不能用于同一个类中的方法自调用，否则可能会出现一些意想不到的异常。

### 写在最后

近期做的这个需求，很多地方组里没有现成可供参考的以往示例，所以很多问题都只能靠自己摸索，摸索到没有头绪的时候就开始试图找各种其他组的人请教，我从来都没有如此社牛过...

并不存在一个能够完整解答完我所有问题的人，也不存在一个完全可以给我照猫画虎的现成的方案，但是找的每一个人和每一个不算完整的方案，都能解答我的一部分疑惑，或者是带给我一些新的思路和启发，当我问的人足够多，尝试的思路足够多，就能一点点构建出我想要的。

解决问题也永远不止一种方案，条条大路通罗马。

### 参考资料

- https://blog.csdn.net/m0_51971452/article/details/115263995
- https://linuxhandbook.com/if-else-bash/
- https://blog.csdn.net/a745233700/article/details/79322757
- https://docs.oracle.com/javase/8/docs/api/java/time/format/DateTimeFormatter.html
- https://csod.my.site.com/supportcentral/s/article/Maximum-column-length-of-100-000-exceeded-in-column-1-Set-the-SafetySwitch-property-to-false?language=en_US
