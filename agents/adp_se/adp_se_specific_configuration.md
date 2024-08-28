# Agent Configuration Search Engine

## Configuration of Selective Backup

To perform selective backup of Search Engine indices, the following parameters are mandatory:

```yaml
brAgent:
  enabled: true
  backupTypeList:
  backupDataModelConfigMap:
```

>**IMPORTANT:** If `brAgent.backupTypeList` or `brAgent.backupDataModelConfigMap` are not specified, a full Search Engine cluster backup will be performed.

Perform the following steps to prepare and create a ConfigMap before installation of Search Engine.

1. Prepare Kubernetes ConfigMap.

    Table Parameters for custom ConfigMap for Selective backup.

    All the parameters in this table are to be defined under `data.backupconfig.yaml.backupMetadataConfig`

    | Key Name | Description |
    |---|---|
    | `backupType` | A specific backup type. <br>**Note:** The backup type should be available in `brAgent.backupTypeList`. |
    | `includeIndices` | Indicates the indices that are to be included for backup. |
    | `excludeIndices` | Indicates the required that are to be excluded for backup. |
    | `name` | Name (or) Regex name of the index that needs to be included (or) excluded for backup. |
    | `minAge` | Minimum age of index that needs to be considered for backup. |

    An example of the ConfigMap:
>**NOTE:** The file name backupconfig.yaml does not allow to change in the data section of ConfigMap.

    ```yaml
    apiVersion: v1
    kind: ConfigMap
    metadata:
      name: eric-data-search-engine-backupconfig
      labels:
        app: eric-data-search-engine
    data:
      backupconfig.yaml: |
        backupMetadataConfig:
          # Supported backup types are listed in "backupType".
          # In each "backupType", "includeIndices" specifies the required indices for backup.
          #    - ".*": all indices available in the cluster will be included in the backup.
          # In each "backupType", "excludeIndices" specifies the excluded indices for backup.
          # In each "includeIndices" or "excludeIndices":
          #    - "name": Name of the index that needs to be considered for backup.
          #    - "minAge": Minimum age of index that needs to be considered for backup.
          - backupType: "dailybackup"
            includeIndices:
              - name: "test-logs*"
                minAge: 1
          - backupType: "applogs"
            excludeIndices:
              - name: "adp-app-asi-logs*"
                minAge: 3
    ```

1. Create the ConfigMap.

    ```yaml
    $ kubectl create -f <configmap-file> --namespace=<NAMESPACE>
    ```
1. Set the ConfigMap related parameters in the Helm chart.

    ```yaml
    brAgent:
      enabled: true
      backupDataModelConfigMap: "eric-data-search-engine-backupconfig"
    ```
