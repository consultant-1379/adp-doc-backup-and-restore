# Agent Configuration Distributed Coordinator ED

## Configuration of Selective Backup
Selective backup is not available for Distributed Coordinator ED as it has not been implemented.

## Configuration Parameters for applicationProperties

| Parameter | Description | Default value |
| --------- | ----------- | ------------- |
| dced.excluded.paths | A comma separated list of key-value prefixes in a etcd host that should not be stored during a backup operation and/or should not be written to during a restore. Each list of key prefixes should be a complete/full path starting from the etcd root. | `empty` |
| dced.included.paths | A comma separated list of key-value prefixes in a etcd host that should be stored during a backup operation and/or should be written to during a restore. Each list of key prefixes should be a complete/full path starting from the etcd root. | `empty` |
| dced.agent.restore.type | The user can choose how the Distributed CoordinatorED BrAgent handles writing of key-value pairs, if present in the prefixes provided during restore action. `overwrite`: Distributed Coordinator ED BrAgent will delete all existing key-value pairs present in the `dced.included.paths` prefixes(except for the prefixes defined in `dced.excluded.paths`) on Distributed Coordinator ED before restoring with the backup data. `merge`: Distributed Coordinator ED BrAgent will push key-value pairs from the backup to Distributed Coordinator ED while leaving untouched any existing key-value pairs (on Distributed Coordinator ED) in the backup.| "overwrite" |