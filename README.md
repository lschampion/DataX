
# DataX

DataX 是阿里云 [DataWorks数据集成](https://www.aliyun.com/product/bigdata/ide) 的开源版本，在阿里巴巴集团内被广泛使用的离线数据同步工具/平台。DataX 实现了包括 MySQL、Oracle、OceanBase、SqlServer、Postgre、HDFS、Hive、ADS、HBase、TableStore(OTS)、MaxCompute(ODPS)、Hologres、DRDS 等各种异构数据源之间高效的数据同步功能。

# DataX 商业版本
阿里云DataWorks数据集成是DataX团队在阿里云上的商业化产品，致力于提供复杂网络环境下、丰富的异构数据源之间高速稳定的数据移动能力，以及繁杂业务背景下的数据同步解决方案。目前已经支持云上近3000家客户，单日同步数据超过3万亿条。DataWorks数据集成目前支持离线50+种数据源，可以进行整库迁移、批量上云、增量同步、分库分表等各类同步解决方案。2020年更新实时同步能力，2020年更新实时同步能力，支持10+种数据源的读写任意组合。提供MySQL，Oracle等多种数据源到阿里云MaxCompute，Hologres等大数据引擎的一键全增量同步解决方案。

商业版本参见：  https://www.aliyun.com/product/bigdata/ide


# Features

DataX本身作为数据同步框架，将不同数据源的同步抽象为从源头数据源读取数据的Reader插件，以及向目标端写入数据的Writer插件，理论上DataX框架可以支持任意数据源类型的数据同步工作。同时DataX插件体系作为一套生态系统, 每接入一套新数据源该新加入的数据源即可实现和现有的数据源互通。



# DataX详细介绍

##### 请参考：[DataX-Introduction](https://github.com/alibaba/DataX/blob/master/introduction.md)



# Quick Start

##### Download [DataX下载地址](http://datax-opensource.oss-cn-hangzhou.aliyuncs.com/datax.tar.gz)

##### 请点击：[Quick Start](https://github.com/alibaba/DataX/blob/master/userGuid.md)



# idea本地调试

具体参考：https://blog.csdn.net/qq_38232753/article/details/108007463

（1）编译并把target 下生成datax.tar.gz 文件解压到固定目录，作为datax.home使用。

（2）配置idea，在core模块下找到Engine类，在类左侧的小三角下的（ Edit 'Engine.main()'... ），创建Run/Debug Configuration启动项。

​          在VM options 设置：`-Ddatax.home=刚刚datax.tar.gz文件解压目录 -Dfile.encoding=UTF-8`

​          在ProgramArguments设置： `-mode standalone -jobid -1 -job datax_mysql2mysql.json` ，这里datax_mysql2mysql.json文件放在项目目录，其他地方也都行的。

（3）确认已经完成跑完编译了。就可以在Run/Debug Configuration 最下边的位置的before launch删除build项目，每次启动前就不要build了。

测试SQL:

```sql
--测试mysql2mysql
DROP TABLE IF EXISTS addax_test.`books`;
CREATE TABLE addax_test.`books` (
  `b_id` int(20) DEFAULT NULL,
  `b_name` varchar(100) DEFAULT NULL,
  `authors` varchar(100) DEFAULT NULL,
  `price` int(20) DEFAULT NULL,
  `pubdate` int(4) DEFAULT NULL,
  `note` varchar(100) DEFAULT NULL,
  `num` int(11) DEFAULT NULL
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4;

INSERT INTO addax_test.books (b_id,b_name,authors,price,pubdate,note,num) VALUES 
(1,'Tal of AAA','Dickes',23,1995,'novel',11)
,(2,'EmmaT','Jane lura',35,1993,'Joke',22)
,(3,'Story of Jane','Jane Tim',40,2001,'novel',0)
,(4,'Lovey Day','George Byron',20,2005,'novel',30)
,(5,'Old land','Honore Blade',30,2010,'Law',0)
,(6,'The Battle','Upton Sara',30,1999,'medicine',40)
,(7,'Rose Hood','Richard haggard',28,2008,'cartoon',28)
;

select * from addax_test.books;

create table addax_test.books_dist like addax_test.books;
truncate  addax_test.books_dist;
select * from  addax_test.books_dist;
```

测试JSON任务文件 datax_mysql2mysql.json

```json
{
  "job": {
    "setting": {
      "speed": {
        "channel": 3,
        "bytes": -1
      }
    },
    "content": {
      "reader": {
        "name": "mysqlreader",
        "parameter": {
          "username": "root",
          "password": "123456",
          "column": [
            "*"
          ],
          "connection": [
            {
              "table": [
                "books"
              ],
              "jdbcUrl": [
                "jdbc:mysql://192.168.100.110:3306/addax_test?useUnicode=true&characterEncoding=utf-8&useSSL=false"
              ],
              "driver": "com.mysql.jdbc.Driver"
            }
          ],
          "where": "1=1"
        }
      },
      "writer": {
        "name": "mysqlwriter",
        "parameter": {
          "writeMode": "insert",
          "username": "root",
          "password": "123456",
          "column": [
            "*"
          ],
          "session": [
            "set session sql_mode='ANSI'"
          ],
          "preSql": [
            "delete from @table"
          ],
          "connection": [
            {
              "jdbcUrl": "jdbc:mysql://192.168.100.110:3306/addax_test?useUnicode=true&characterEncoding=utf-8&useSSL=false",
              "table": [
                "books_dist"
              ],
              "driver": "com.mysql.jdbc.Driver"
            }
          ]
        }
      }
    }
  }
}
```



# Kerberos integration

datax [hdfs](https://so.csdn.net/so/search?q=hdfs&spm=1001.2101.3001.7020) 支持parquet 和 hbase支持 kerberos

具体请参考 https://blog.csdn.net/gelonSun/article/details/119034469

无法解析 realm问题：KrbException: Cannot locate default realm
解决方法：把krb5.conf文件放在/etc/即可。

缺少包情况

```shell
# 将jar_files 的jar 手动加载本地maven仓库
# 文件的目录根据实际情况需要修改一下
mvn install:install-file -DgroupId=eigenbase -DartifactId=eigenbase-properties -Dversion=1.1.4 -Dpackaging=jar -Dfile=/Users/***/jars/eigenbaseeigenbaseproperties/eigenbase-properties-1.1.4.jar


mvn install:install-file -DgroupId=org.pentaho -DartifactId=pentaho-aggdesigner-algorithm -Dversion=5.1.5-jhyde -Dpackaging=jar -Dfile=/Users/***/pentaho-aggdesigner-algorithm-5.1.5-jhyde.jar

# 此jar包是自己编译的，220430，从git（https://github.com/aliyun/aliyun-tsdb-java-sdk.git）拉取指定为0.4.0版本的。
# 如果直接使用0.3.7 release 版本会缺少类，编译不过去。
mvn install:install-file -DgroupId=com.aliyun -DartifactId=hitsdb-client -Dversion=0.4.0 -Dpackaging=jar -Dfile=/Users/***/aliyun-tsdb-java-sdk/target/hitsdb-client-0.4.0.jar
```



- 升级hive version后，datax支持的kerberos校验会有问题导致报错，故在json配置中增加如下固定配置

  ```json
  "hadoopConfig":{
       "mapreduce.framework.name":"classic",
       "mapreduce.jobtracker.kerberos.principal":"xxxxx",
       "mapreduce.jobtracker.keytab.file":"keytab path"
  }
  ```



## datax hbase11x 修改支持kerberos

- 修改后需增加配置如下

```json
"parameter": {
    "kerberos":"true",
    "keyTabKey":"xxx.com",
    "keyTabValue":"${keytab_path}",
    "hbaseSiteXml":"hbase-site.xml path",
    "system": {
        "javax.security.auth.useSubjectCredsOnly": "false",
        "java.security.krb5.conf": "${krb5.conf_path}", 
        "HADOOP_USER_NAME": "kerberos.principal"
    },
    "hbaseConfig": {
        "hadoop.security.authentication": "Kerberos",
        "hbase.client.ipc.pool.size": "20",
        "hadoop.user.name": "xxx.com" ---xxx.com
    },
	
	.....
}
```



## datax配置[hadoop](https://so.csdn.net/so/search?q=hadoop&spm=1001.2101.3001.7020) HA（高可用）
方式一（不推荐）
defaultFS 只能配置一个namenode节点 当namenode为高可用时，挂掉配置的那个节点datax任务就会报错，文档上写不支持MA，但通过参数配置是可以支持的，故配置为HA模式。
方式一有问题，主要是还是需要配置defaultFS，且必须指明单一的IP域名和端口号，且不能是数组。
```plain
"hadoopConfig":{
   "dfs.nameservices":"yournamespace",
   "dfs.ha.namenodes.yournamespace":"namenode1,namenode2",
   "dfs.namenode.rpc-address.yournamespace.namenode1":"xxxxx:8020",
   "dfs.namenode.rpc-address.yournamespace.namenode2":"xxxxx:8020",
   "dfs.client.failover.proxy.provider.yournamespace": "org.apache.hadoop.hdfs.server.namenode.ha.ConfiguredFailoverProxyProvider",
  "dfs.ha.automatic-failover.enabled.yournamespace":"true"
},
```
方式二（推荐）
参考：https://blog.csdn.net/weixin_44441757/article/details/118082138 文章
推荐把core-site.xml,hdfs-site.xml,yarn-site.xml,hive-site.xml添加至hdfsreader-版本-.jar 和 hdfswriter-版本-.jar 里边
然后hadoopConfig配置项就可以删除了，而defaultFS可以配置 "hdfs://${dfs.nameservices}"的方式了。其中dfs.nameservices可以到hdfs-site.xml查找

## datax的限速

需要限制datax的速度(直接修改speed)，阅读文档发现如下描述片段,直接添加在配置文件job中报错

```json
    "core": {
         "transport" : {
              "channel": {
                   "speed": {
                       "byte": 2000000  //单个channel 2M
                    }
               }
         }
    },
    "job": {
        "setting": {
            "speed": {
                "channel": 5,   // 5个channel
                "byte": 15000000  //总共15M
            }
        },
```

# Support Data Channels 

DataX目前已经有了比较全面的插件体系，主流的RDBMS数据库、NOSQL、大数据计算系统都已经接入，目前支持数据如下图，详情请点击：[DataX数据源参考指南](https://github.com/alibaba/DataX/wiki/DataX-all-data-channels)

| 类型           | 数据源        | Reader(读) | Writer(写) |文档|
| ------------ | ---------- | :-------: | :-------: |:-------: |
| RDBMS 关系型数据库 | MySQL      |     √     |     √     |[读](https://github.com/alibaba/DataX/blob/master/mysqlreader/doc/mysqlreader.md) 、[写](https://github.com/alibaba/DataX/blob/master/mysqlwriter/doc/mysqlwriter.md)|
|              | Oracle     |     √     |     √     |[读](https://github.com/alibaba/DataX/blob/master/oraclereader/doc/oraclereader.md) 、[写](https://github.com/alibaba/DataX/blob/master/oraclewriter/doc/oraclewriter.md)|
|              | OceanBase  |     √     |     √     |[读](https://open.oceanbase.com/docs/community/oceanbase-database/V3.1.0/use-datax-to-full-migration-data-to-oceanbase) 、[写](https://open.oceanbase.com/docs/community/oceanbase-database/V3.1.0/use-datax-to-full-migration-data-to-oceanbase)|
|              | SQLServer  |     √     |     √     |[读](https://github.com/alibaba/DataX/blob/master/sqlserverreader/doc/sqlserverreader.md) 、[写](https://github.com/alibaba/DataX/blob/master/sqlserverwriter/doc/sqlserverwriter.md)|
|              | PostgreSQL |     √     |     √     |[读](https://github.com/alibaba/DataX/blob/master/postgresqlreader/doc/postgresqlreader.md) 、[写](https://github.com/alibaba/DataX/blob/master/postgresqlwriter/doc/postgresqlwriter.md)|
|              | DRDS |     √     |     √     |[读](https://github.com/alibaba/DataX/blob/master/drdsreader/doc/drdsreader.md) 、[写](https://github.com/alibaba/DataX/blob/master/drdswriter/doc/drdswriter.md)|
|              | 通用RDBMS(支持所有关系型数据库)         |     √     |     √     |[读](https://github.com/alibaba/DataX/blob/master/rdbmsreader/doc/rdbmsreader.md) 、[写](https://github.com/alibaba/DataX/blob/master/rdbmswriter/doc/rdbmswriter.md)|
| 阿里云数仓数据存储    | ODPS       |     √     |     √     |[读](https://github.com/alibaba/DataX/blob/master/odpsreader/doc/odpsreader.md) 、[写](https://github.com/alibaba/DataX/blob/master/odpswriter/doc/odpswriter.md)|
|              | ADS        |           |     √     |[写](https://github.com/alibaba/DataX/blob/master/adswriter/doc/adswriter.md)|
|              | OSS        |     √     |     √     |[读](https://github.com/alibaba/DataX/blob/master/ossreader/doc/ossreader.md) 、[写](https://github.com/alibaba/DataX/blob/master/osswriter/doc/osswriter.md)|
|              | OCS        |           |     √     |[写](https://github.com/alibaba/DataX/blob/master/ocswriter/doc/ocswriter.md)|
| NoSQL数据存储    | OTS        |     √     |     √     |[读](https://github.com/alibaba/DataX/blob/master/otsreader/doc/otsreader.md) 、[写](https://github.com/alibaba/DataX/blob/master/otswriter/doc/otswriter.md)|
|              | Hbase0.94  |     √     |     √     |[读](https://github.com/alibaba/DataX/blob/master/hbase094xreader/doc/hbase094xreader.md) 、[写](https://github.com/alibaba/DataX/blob/master/hbase094xwriter/doc/hbase094xwriter.md)|
|              | Hbase1.1   |     √     |     √     |[读](https://github.com/alibaba/DataX/blob/master/hbase11xreader/doc/hbase11xreader.md) 、[写](https://github.com/alibaba/DataX/blob/master/hbase11xwriter/doc/hbase11xwriter.md)|
|              | Phoenix4.x   |     √     |     √     |[读](https://github.com/alibaba/DataX/blob/master/hbase11xsqlreader/doc/hbase11xsqlreader.md) 、[写](https://github.com/alibaba/DataX/blob/master/hbase11xsqlwriter/doc/hbase11xsqlwriter.md)|
|              | Phoenix5.x   |     √     |     √     |[读](https://github.com/alibaba/DataX/blob/master/hbase20xsqlreader/doc/hbase20xsqlreader.md) 、[写](https://github.com/alibaba/DataX/blob/master/hbase20xsqlwriter/doc/hbase20xsqlwriter.md)|
|              | MongoDB    |     √     |     √     |[读](https://github.com/alibaba/DataX/blob/master/mongodbreader/doc/mongodbreader.md) 、[写](https://github.com/alibaba/DataX/blob/master/mongodbwriter/doc/mongodbwriter.md)|
|              | Hive       |     √     |     √     |[读](https://github.com/alibaba/DataX/blob/master/hdfsreader/doc/hdfsreader.md) 、[写](https://github.com/alibaba/DataX/blob/master/hdfswriter/doc/hdfswriter.md)|
|              | Cassandra       |     √     |     √     |[读](https://github.com/alibaba/DataX/blob/master/cassandrareader/doc/cassandrareader.md) 、[写](https://github.com/alibaba/DataX/blob/master/cassandrawriter/doc/cassandrawriter.md)|
| 无结构化数据存储     | TxtFile    |     √     |     √     |[读](https://github.com/alibaba/DataX/blob/master/txtfilereader/doc/txtfilereader.md) 、[写](https://github.com/alibaba/DataX/blob/master/txtfilewriter/doc/txtfilewriter.md)|
|              | FTP        |     √     |     √     |[读](https://github.com/alibaba/DataX/blob/master/ftpreader/doc/ftpreader.md) 、[写](https://github.com/alibaba/DataX/blob/master/ftpwriter/doc/ftpwriter.md)|
|              | HDFS       |     √     |     √     |[读](https://github.com/alibaba/DataX/blob/master/hdfsreader/doc/hdfsreader.md) 、[写](https://github.com/alibaba/DataX/blob/master/hdfswriter/doc/hdfswriter.md)|
|              | Elasticsearch       |         |     √     |[写](https://github.com/alibaba/DataX/blob/master/elasticsearchwriter/doc/elasticsearchwriter.md)|
| 时间序列数据库 | OpenTSDB | √ |  |[读](https://github.com/alibaba/DataX/blob/master/opentsdbreader/doc/opentsdbreader.md)|
|  | TSDB | √ | √ |[读](https://github.com/alibaba/DataX/blob/master/tsdbreader/doc/tsdbreader.md) 、[写](https://github.com/alibaba/DataX/blob/master/tsdbwriter/doc/tsdbhttpwriter.md)|

# 阿里云DataWorks数据集成

目前DataX的已有能力已经全部融和进阿里云的数据集成，并且比DataX更加高效、安全，同时数据集成具备DataX不具备的其它高级特性和功能。可以理解为数据集成是DataX的全面升级的商业化用版本，为企业可以提供稳定、可靠、安全的数据传输服务。与DataX相比，数据集成主要有以下几大突出特点：

支持实时同步：

- 功能简介：https://help.aliyun.com/document_detail/181912.html
- 支持的数据源：https://help.aliyun.com/document_detail/146778.html
- 支持数据处理：https://help.aliyun.com/document_detail/146777.html

离线同步数据源种类大幅度扩充：

- 新增比如：DB2、Kafka、Hologres、MetaQ、SAPHANA、达梦等等，持续扩充中
- 离线同步支持的数据源：https://help.aliyun.com/document_detail/137670.html
- 具备同步解决方案：
    - 解决方案系统：https://help.aliyun.com/document_detail/171765.html
    - 一键全增量：https://help.aliyun.com/document_detail/175676.html
    - 整库迁移：https://help.aliyun.com/document_detail/137809.html
    - 批量上云：https://help.aliyun.com/document_detail/146671.html
    - 更新更多能力请访问：https://help.aliyun.com/document_detail/137663.html


# 我要开发新的插件

请点击：[DataX插件开发宝典](https://github.com/alibaba/DataX/blob/master/dataxPluginDev.md)


# 项目成员

核心Contributions: 言柏 、枕水、秋奇、青砾、一斅、云时

感谢天烬、光戈、祁然、巴真、静行对DataX做出的贡献。

# License

This software is free to use under the Apache License [Apache license](https://github.com/alibaba/DataX/blob/master/license.txt).

# 
请及时提出issue给我们。请前往：[DataxIssue](https://github.com/alibaba/DataX/issues)

# 开源版DataX企业用户

![Datax-logo](https://github.com/alibaba/DataX/blob/master/images/datax-enterprise-users.jpg)

```
长期招聘 联系邮箱：datax@alibabacloud.com
【JAVA开发职位】
职位名称：JAVA资深开发工程师/专家/高级专家
工作年限 : 2年以上
学历要求 : 本科（如果能力靠谱，这些都不是条件）
期望层级 : P6/P7/P8

岗位描述：
    1. 负责阿里云大数据平台（数加）的开发设计。 
    2. 负责面向政企客户的大数据相关产品开发；
    3. 利用大规模机器学习算法挖掘数据之间的联系，探索数据挖掘技术在实际场景中的产品应用 ；
    4. 一站式大数据开发平台
    5. 大数据任务调度引擎
    6. 任务执行引擎
    7. 任务监控告警
    8. 海量异构数据同步

岗位要求：
    1. 拥有3年以上JAVA Web开发经验；
    2. 熟悉Java的基础技术体系。包括JVM、类装载、线程、并发、IO资源管理、网络；
    3. 熟练使用常用Java技术框架、对新技术框架有敏锐感知能力；深刻理解面向对象、设计原则、封装抽象；
    4. 熟悉HTML/HTML5和JavaScript；熟悉SQL语言；
    5. 执行力强，具有优秀的团队合作精神、敬业精神；
    6. 深刻理解设计模式及应用场景者加分；
    7. 具有较强的问题分析和处理能力、比较强的动手能力，对技术有强烈追求者优先考虑；
    8. 对高并发、高稳定可用性、高性能、大数据处理有过实际项目及产品经验者优先考虑；
    9. 有大数据产品、云产品、中间件技术解决方案者优先考虑。
````
钉钉用户群：

- DataX开源用户交流群
    - <img src="https://github.com/alibaba/DataX/blob/master/images/DataX%E5%BC%80%E6%BA%90%E7%94%A8%E6%88%B7%E4%BA%A4%E6%B5%81%E7%BE%A4.jpg" width="20%" height="20%">

- DataX开源用户交流群2
    - <img src="https://github.com/alibaba/DataX/blob/master/images/DataX%E5%BC%80%E6%BA%90%E7%94%A8%E6%88%B7%E4%BA%A4%E6%B5%81%E7%BE%A42.jpg" width="20%" height="20%">

- DataX开源用户交流群3
    - <img src="https://github.com/alibaba/DataX/blob/master/images/DataX%E5%BC%80%E6%BA%90%E7%94%A8%E6%88%B7%E4%BA%A4%E6%B5%81%E7%BE%A43.jpg" width="20%" height="20%">

- DataX开源用户交流群4
    - <img src="https://github.com/alibaba/DataX/blob/master/images/DataX%E5%BC%80%E6%BA%90%E7%94%A8%E6%88%B7%E4%BA%A4%E6%B5%81%E7%BE%A44.jpg" width="20%" height="20%">

- DataX开源用户交流群5
    - <img src="https://github.com/alibaba/DataX/blob/master/images/DataX%E5%BC%80%E6%BA%90%E7%94%A8%E6%88%B7%E4%BA%A4%E6%B5%81%E7%BE%A45.jpg" width="20%" height="20%">

- DataX开源用户交流群6 
    - <img src="https://user-images.githubusercontent.com/1905000/124073771-139cbd00-da75-11eb-9a3f-598cba145a76.png" width="20%" height="20%">

