# Agent Configuration Object Storage MN

## Configuration of Selective Backup

To perform selective backup of data stored in Object Storage the following parameters are mandatory:
 * brAgent.enabled
 * brAgent.backupTypeList
 * brAgent.properties.backup_data_path

Example configuration of selective backup:
 * set brAgent.enabled to true
 * set brAgent.backupTypeList[0] to "configuration-data"
 * set brAgent.properties.backup_data_path to "bucket1/data1.txt,bucket2/data2.txt,bucket3"
With that configuration the following will be included in the backup: files bucket1/data1.txt and bucket2/data2.txt and all files in bucket3.

For all backup-restore related parameters see the following table:

| Parameter | Description | Default value |
|---|---|---|
| brAgent.backupTypeList | A list that determines the scope of the backup ("configuration-data", "subscribers-data", "all-application-data"). Only one value is supported in the current release. Not specifying any value implies that the scope of this BRA will be "all-application-data".<br><br>Example of valid configurations (other than the default):<br><br>brAgent.backupTypeList:<br> - subscribers-data<br><br>brAgent.backupTypeList:<br> - configuration-data | null |
| brAgent.properties.backup_data_path | The data path specify some files(objects) or folders(buckets) to backup and restore in Object Storage MN server. Note that does not support including regexp and wildcards, but support including space.<br><br>Example:<br>backup_data_path: "bucket1/data1.txt,bucket2/data2.txt,bucket3"<br>backup_data_path: "bucket1/data1.txt, bucket2/data2.txt, bucket3" | "" |
| brAgent.retryTimes | Retry times when Object Storage BRA backup or restore fail. | 3 |
| brAgent.tls.paths.clientCerts | Path of the secret holding Object Storage BRA credentials for internal components. When global.security.tls.enabled is true, if this value is set as non-empty, it will enable mTLS channel with BRO. Otherwise, it will only enable TLS. | /etc/tls/cert/client |

