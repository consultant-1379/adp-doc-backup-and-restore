# Agent Configuration MBKF

## Selective Backup
Selective backup is not needed for Zookeeper in Message Bus Kafka Strimzi Operator hence it is not supported.

## Configuration Parameters for applicationProperties

| Parameter | Description | Default value |
| --------- | ----------- | ------------- |
| zk.agent.restore.type | The user can choose how the *ZK BrAgent* in Message Bus Kafka Strimzi Operator handles overwriting of data, if present in the paths provided during *restore* action.                                                                                                                   `default`: Use this when restoring to a *ZK* cluster that does not have any data,                                               `overwriteall`: *ZK BrAgent* will delete all existing data on *ZK* before restoring with the backup data,                                                                                                                      `overwritemerge`: *ZK BrAgent* will push data from the backup to *ZK* while leaving untouched any existing data on *ZK* without a version in the backup. | "default" |
