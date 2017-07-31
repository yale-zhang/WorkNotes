###hive与mysql转化
Mysql2Hive   
增量导入：   
```sql
sqoop import -m 1 --connect jdbc:mysql://10.1.70.130:3306/bd_source --username bdkg --password bdkg#$ --table wifi_locationmap  --check-column ID_ROW_SEQ_NO --last-value '0'  --fields-terminated-by "," --lines-terminated-by "\n" --incremental append  --hive-table bd_ods.src_wifi_locationmap_his --hive-import  --hive-partition-key 'prt_month' --hive-partition-value   ` date +%Y%m `
```
- - - 
partition 出现2个分区的情况下：   
全量导入:   
```sql
sqoop import -m 1 --connect jdbc:mysql://10.1.70.130:3306/bdkg_system --username bdkg --password bdkg#$ --table wifi_locationdata --hcatalog-database bd_ods --hcatalog-table src_wifi_loacationdata_his --hcatalog-partition-keys prt_month,prt_date --hcatalog-partition-values 201707,20170701
```
增量导入   
```sql
sqoop import -m 1 --connect jdbc:mysql://10.1.70.130:3306/bdkg_system --username bdkg --password bdkg#$ --query 'select * from wifi_locationdata WHERE ID_ROW_SEQ_NO <100 and $CONDITIONS' --hcatalog-database bd_ods --hcatalog-table src_wifi_loacationdata_his --hcatalog-partition-keys prt_month,prt_date --hcatalog-partition-values 201707,20170701 
```   
```sql
sqoop import -m 1 --connect jdbc:mysql://10.1.70.130:3306/bdkg_system --username bdkg --password bdkg#$ --query 'select * from wifi_locationdata WHERE ID_ROW_SEQ_NO >=100 and ID_ROW_SEQ_NO <200 and $CONDITIONS' --hcatalog-database bd_ods --hcatalog-table src_wifi_loacationdata_his --hcatalog-partition-keys prt_month,prt_date --hcatalog-partition-values 201707,20170702 
```
src_flow_intsummary_base 2 flow_intsummary_base

- - -
数据导出：  
 1.导出完全一致====
 ```sql
 sqoop export --connect jdbc:mysql://10.1.70.130:3306/bd_source?characterEncoding=utf8 --username bdkg --P --table flow_intsummary_base --hcatalog-database bd_ods --hcatalog-table src_flow_intsummary_base --input-fields-terminated-by '\t' --input-null-string '\\N' --input-null-non-string '\\N'
 ```
_ _ _

*error 1:
ERROR manager.SqlManager: Error executing statement: com.mysql.jdbc.exceptions.jdbc4.CommunicationsException: Communications link failure
The last packet sent successfully to the server was 0 milliseconds ago. The driver has not received any packets from the server.
com.mysql.jdbc.exceptions.jdbc4.CommunicationsException: Communications link failure*
源mysql和集群机器不在同一个网段中，导致执行导入命令，网络连接失败。先ping下看是否能连接上远程地址

*ERROR 2：
ERROR tool.ExportTool: Encountered IOException running export job: org.apache.hadoop.mapreduce.lib.input.InvalidInputException: Input path does not exist: hdfs://name.hadoop.demo:8020/Database/bd_obs/src_flow_intsummary_base*

*ERROR 3:
mapreduce.Job: Job job_1500858852491_0004 failed with state FAILED due to: Task failed task_1500858852491_0004_m_000000
Job failed as tasks failed. failedMaps:1 failedReduces:0
ERROR tool.ExportTool: Error during export: Export job failed!*
-- 文件名没写全(000000_0)

2.按列导出数据
```sql
sqoop export --connect jdbc:mysql://10.1.70.130:3306/bd_source?characterEncoding=utf8 --username bdkg --P --columns ID_ROW_SEQ_NO,KY_SITEKEY,KY_COUNTDATE,LG_INIT_TIME,LG_CHG_TIME --table flow_intsummary_base --hcatalog-database bd_ods --hcatalog-table src_flow_intsummary_base --input-fields-terminated-by '\t' --input-null-string '\\N' --input-null-non-string '\\N' -m 1
```
**1，按列导出数据如果目标表内有不为空的列，则必须导入该列
  2，列名的顺序可以不一致**