# hive-metadata-upgrade

先创建个emr 5.30.0和rds mysql，然后创建emr 6.1.0, 手动在hive-site.xml里面把metastore指到rds mysql上，然后用schematool升级，报错

$ hive --service schemaTool -dbType mysql -upgradeSchema 
Metastore connection URL: jdbc:mysql://hive.cdh87nkaaavb.rds.cn-north-1.amazonaws.com.cn:3306/hive?createDatabaseIfNotExist=true 
Metastore Connection Driver : org.mariadb.jdbc.Driver 
Metastore connection User: admin 
Starting upgrade metastore schema from version 2.3.0 to 3.1.0 
Upgrade script upgrade-2.3.0-to-3.0.0.mysql.sql ...... 
Error: (conn=78) Duplicate key name 'MV_UNIQUE_TABLE' (state=42000,code=1061) org.apache.hadoop.hive.metastore.HiveMetaException: Upgrade FAILED! Metastore state would be inconsistent !! Underlying cause: java.io.IOException : Schema script failed, errorcode 2

错误应该是前一次启动emr 6.1.0的时候引入的, 如果启动的时候就指定了rds mysql，启动过程中会自动创建schema. 所以3.1.0版本的schema被创建之后 你后面自己手动升级的时候就会报duplicate key的错误

2021-01-19 04:37:23 +0000 /Stage[main]/Hadoop_hive::Init_metastore_schema/Exec[init hive-metastore schema]/returns (notice): 0: jdbc:mysql://hive.cdh87aaaaivb.rds.cn-nort> CREATE TABLE `CTLGS` ( `CTLG_ID`  
BIGINT PRIMARY KEY, `NAME` VARCHAR(256), `DESC` VARCHAR(4000), `LOCATION_URI` VA 
RCHAR(4000) NOT NULL, UNIQUE KEY `UNIQUE_CATALOG` (`NAME`) ) ENGINE=InnoDB DEFAU 
LT CHARSET=latin1
2021-01-19 04:37:23 +0000 /Stage[main]/Hadoop_hive::Init_metastore_schema/Exec[init hive-metastore schema]/returns (notice): Error: (conn=46) Table 'CTLGS' already exists (state=42S01,code=1050)
2021-01-19 04:37:23 +0000 /Stage[main]/Hadoop_hive::Init_metastore_schema/Exec[init hive-metastore schema]/returns (notice): Closing: 0: jdbc:mysql://hive.cdh87nkcaivb.rds.cn-north-1.amazonaws.com.cn:3306/hive?createDatabaseIfNotExist=true&amp;useUnicode=true&characterEncoding=UTF-8
2021-01-19 04:37:23 +0000 /Stage[main]/Hadoop_hive::Init_metastore_schema/Exec[init hive-metastore schema]/returns (notice): org.apache.hadoop.hive.metastore.HiveMetaException: Schema initialization FAILED! Metastore state would be inconsistent !!
2021-01-19 04:37:23 +0000 /Stage[main]/Hadoop_hive::Init_metastore_schema/Exec[init hive-metastore schema]/returns (notice): Underlying cause: java.io.IOException : Schema script failed, errorcode 2
2021-01-19 04:37:23 +0000 /Stage[main]/Hadoop_hive::Init_metastore_schema/Exec[init hive-metastore schema]/returns (notice): org.apache.hadoop.hive.metastore.HiveMetaException: Schema initialization FAILED! Metastore state would be inconsistent !!

log里面可以看到在初始化schema的时候，因为原来rds mysql已经有了CTLGS这个表，所以就初始化失败了，然后集群就创建失败了

$ hive --service schemaTool -dbType mysql -upgradeSchema 
Metastore connection URL:        jdbc:mysql://hive.cdh87naaaivb.rds.cn-north-1.amazonaws.com.cn:3306/hive?createDatabaseIfNotExist=true
Metastore Connection Driver :    org.mariadb.jdbc.Driver
Metastore connection User:       admin
Starting upgrade metastore schema from version 2.3.0 to 3.1.0
Upgrade script upgrade-2.3.0-to-3.0.0.mysql.sql


Completed upgrade-2.3.0-to-3.0.0.mysql.sql
Upgrade script upgrade-3.0.0-to-3.1.0.mysql.sql


Completed upgrade-3.0.0-to-3.1.0.mysql.sql
schemaTool completed
