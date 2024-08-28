# Agent Configuration DCZK

## Selective Backup
Selective backup is not needed for Data Coordinator ZK hence it is not supported.

## Configuration Parameters for applicationProperties

| Parameter | Description | Default value |
| --------- | ----------- | ------------- |
| dczk.excluded.paths | A comma separated list of paths in a zookeeper host that should not be stored during a backup operation and/or should not be written to during a restore. Each path should be a complete path starting from the zookeeper root. | /zookeeper,/eric-data-message-bus-kf |
| dczk.included.paths | A comma separated list of paths in a zookeeper host that should be stored during a backup operation and/or should be written to during a restore. The implication is that, only these specified nodes and their children will be affected during a backup/restore operation. Each path should be a complete path starting from the zookeeper root.  | "" |
| dczk.agent.restore.type | The user can choose how the *Data Coordinator ZK BrAgent* handles overwriting of data, if present in the paths provided during *restore* action.                                                                                                                   `default`: Use this when restoring to a *Data Coordinator ZK* cluster that does not have any data,                                               `overwriteall`: *Data Coordinator ZK BrAgent* will delete all existing data (except for excluded data) on *Data Coordinator ZK* before restoring with the backup data,                                                                                                                      `overwritemerge`: *Data Coordinator ZK BrAgent* will push data from the backup to *Data Coordinator ZK* while leaving untouched any existing data (on *Data Coordinator ZK*) without a version in the backup. | "default" |
