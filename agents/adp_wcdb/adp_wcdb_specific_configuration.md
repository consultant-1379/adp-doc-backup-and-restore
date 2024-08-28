# Agent Configuration Wide Column Database CD

## Configuration of Backup Location Credentials

By default, backups are transferred to BR-O, which in turn is responsible for storing the data.
Optionally, backups can be transferred to Amazon S3 instead. For more information,
see [Amazon S3](https://aws.amazon.com/s3/).

If the backup location is set to Amazon S3 then WCDBCD must be configured with a secret containing
S3 authentication credentials. This secret must be present **before** installing or upgrading WCDBCD with S3
as the backup location. The secret name can be defined by the user and must be provided through the
`brAgent.locationAuthKey` Helm parameter. The secret can be generated as follows:

```bash
kubectl create secret generic <secret-name> --from-literal=aws-s3-access-key-user=<access-key-user> --from-literal=aws-s3-access-key=<access-key>
kubectl get secret/s3-auth-credentials -o yaml
```

The `secret-name` placeholder should be replaced with the chosen secret name.
The `access-key-user` and `access-key` placeholders should be replaced with the credentials for the S3 provider.
Example output of above `kubectl get` command:

```yaml
kind: Secret
apiVersion: v1
metadata:
  name: s3-auth-credentials
data:
  aws-s3-access-key: <base64 encoded credentials>
  aws-s3-access-key-user: <base64 encoded credentials>
type: Opaque
```

## Configuration of Selective Backup

The default behavior is full backup (and full restore). Selective backup can be performed by
listing the inclusion and/or exclusion subset in a configuration file. Selective backup filters
can be specified on keyspace or table level.
To perform selective backup of keyspaces and/or tables in Wide Column Database CD (WCDBCD), the
following parameters are mandatory:

```yaml
brAgent:
  enabled: true
  backupTypeList:
    - <backup-type>
    ...
  backupDataModelConfigMap: <config-map-name>
```

> **Note**
>
> If `brAgent.backupDataModelConfigMap` is not specified, a full backup of WCDBCD will be performed.

Perform the following steps to prepare and create a ConfigMap **before** installation of Wide Column Database CD.

1. Prepare a JSON file named `backup-data-model-config.json`, as in the example shown below.

   > **Note**
   >
   > The file name needs to be exactly `backup-data-model-config.json`, it is not configurable.

2. Create the ConfigMap from the JSON file by executing the following command.

   ```bash
   kubectl create configmap wcdbcd-backup-data-model-config --from-file=backup-data-model-config.json
   ```

3. Set the necessary Backup and Restore parameters in the WCDBCD Helm Chart.

   ```yaml
   brAgent:
     enabled: true
     backupTypeList:
       - "config"
       - "customers"
     backupDataModelConfigMap: "wcdbcd-backup-data-model-config"
   ```

### Exclusions Example

In this example, two (2) backup types have been configured, `config` and `customers`.
In addition, the `DEFAULT` backup type is also supported (it always is).

```json
  {
    "exclusions": [
      {
        "backupType": "config",
        "keyspaces": [
          {
            "name": "k1",
            "tables": []
          },
          {
            "name": "k2",
            "tables": []
          },
          {
            "name": "k3",
            "tables": ["t31", "t32", "t33"]
          }
        ]
      },
      {
        "backupType": "customers",
        "keyspaces": [
          {
            "name": "k3",
            "tables": ["t31", "t32", "t33"]
          },
          {
            "name": "k4",
            "tables": ["t41", "t42", "t43"]
          }
        ]
      }
    ]
  }
  ```

- When a backup is performed with the `config` backup type, all tables in keyspaces
  `k1` and `k2` will be excluded, as well as the specific tables `k3.t31`, `k3.t32`, `k3.t33`.

- When a backup is performed with the `customers` backup type, the tables `k3.t31`, `k3.t32`,
  `k3.t33`, `k4.t41`, `k4.t42`, and `k4.t43` will be excluded.

- When a backup is performed with the `DEFAULT` backup type, the **intersection** of all the
  other **exclusions** is excluded, in this case the tables `k3.t31`, `k3.t32`, and `k3.t33`. To
  put it differently, if a keyspace or table is excluded in **all** configured backup types,
  it will also be excluded in the `DEFAULT` backup.

### Inclusions Example

Inclusion can be used to limit the scope of a backup type to a specific list of keyspaces and/or tables.
In this example, the same two (2) backup types from the previous example have been configured again,
`config` and `customers`. In addition, the `DEFAULT` backup type is also supported for inclusions.
If a backup type, except for the `DEFAULT`, does not have inclusions defined it will be interpreted
as **include everything**.

```json
  {
    "inclusions": [
      {
        "backupType": "config",
        "keyspaces": [
          {
            "name": "k1",
            "tables": []
          },
          {
            "name": "k2",
            "tables": []
          },
          {
            "name": "k3",
            "tables": ["t31", "t32", "t33"]
          }
        ]
      },
      {
        "backupType": "customers",
        "keyspaces": [
          {
            "name": "k3",
            "tables": ["t31", "t32", "t33"]
          },
          {
            "name": "k4",
            "tables": ["t41", "t42", "t43"]
          }
        ]
      }
    ],
    "exclusions": []
  }
  ```

- When a backup is performed with the `config` backup type, all tables in keyspaces
  `k1` and `k2` will be included, as well as the specific tables `k3.t31`, `k3.t32`, `k3.t33`.
  The rest of keyspace `k3` will be excluded, as it is not in the defined inclusion.

- When a backup is performed with the `customers` backup type, the tables `k3.t31`,
 `k3.t32`, `k3.t33`, `k4.t41`, `k4.t42`, and `k4.t43` will be included. The rest of
  keyspaces `k3` and `k4` will not be included, as they are not in the defined inclusions.

- When a backup is performed with the `DEFAULT` backup type, the **union** of all the
  other **inclusions** is included, in this case `k1` and `k2` will be included, as well as
  the tables `k3.t31`, `k3.t32`, `k3.t33`, `k4.t41`, `k4.t42`, and `k4.t43`. To put it
  differently, if a keyspace or table is included in **any** configured backup types,
  it will also be included in the `DEFAULT` backup.

For any of the three backup types above, all other keyspaces that are not part of the inclusions
will be excluded from the backup.

### Exclusions and Inclusions Simultaneously

If there are exclusions and inclusions for the same backup type and they have an
intersection of their content (they both are referring to the same keyspace or table),
then the exclusion has precedence over the inclusion.

  ```json
  {
    "exclusions": [
      {
        "backupType": "config",
        "keyspaces": [
          {
            "name": "k1",
            "tables": ["t11"]
          }
        ],
    "inclusions": [
      {
        "backupType": "config",
        "keyspaces": [
          {
            "name": "k1",
            "tables": []
          }
        ]
      }
    ]
  }
  ```

In this example, when a backup is performed with the `config` backup type, all tables
from keyspace `k1` will be included, except for table `k1.t11`, that will be excluded.

On the other hand, if there are inclusions and exclusions for the same backup type, and
the inclusions do not intersect with any exclusion, the exclusions will be ignored.

  ```json
  {
    "exclusions": [
      {
        "backupType": "config",
        "keyspaces": [
          {
            "name": "k1",
            "tables": ["t11"]
          },
          {
            "name": "k2",
            "tables": ["t21"]
          }
        ],
    "inclusions": [
      {
        "backupType": "config",
        "keyspaces": [
          {
            "name": "k3",
            "tables": []
          },
          {
            "name": "k4",
            "tables": []
          }
        ]
      }
    ]
  }
  ```

In this example, when a backup is performed with the `config` backup type, only `k3` and `k4` will
be included, as specified in the inclusions. All other keyspaces will be excluded, regardless of
them being explicitly excluded (such as `k1` and `k2`) or not.

### Using Regular Expressions to Configure Exclusion and Inclusion Filters

It is also possible to use regular expression patterns to configure exclusions and inclusions. This
enables them to match keyspaces and tables based on more complex and dynamic criteria, rather than
explicitly listing each individual keyspace and table name. Essentially, when a given keyspace or
table name has a match with a regular expression pattern, it will be included/excluded. Refer to
previous sections [Exclusions Example](#exclusions-example) and [Inclusions Example](#inclusions-example)
for instructions on how to use them.

> **Note**
>
> The field `backupType` does not support regular expression patterns.

### User-defined Default Backup Example

It is also possible to exclude keyspaces and tables for the `DEFAULT` backup type. If this is
configured, it must be the **single** entry, so no other backup types are allowed.

  ```json
  {
    "exclusions": [
      {
        "backupType": "DEFAULT",
        "keyspaces": [
          {
            "name": "k1",
            "tables": ["t11"]
          }
        ]
      }
    ],
    "inclusions": [
      {
        "backupType": "DEFAULT",
        "keyspaces": [
          {
            "name": "k1",
            "tables": []
          },
          {
            "name": "k2",
            "tables": []
          }
        ]
      }
    ]
  }
  ```

In this example, when a backup is performed with the `DEFAULT` backup type, for keyspace `k1`
only table `k1.t11` is excluded (as exclusions have precedence over intersecting inclusions).
Keyspace `k2` is fully included.

> **Note**
>
> The `system_auth` keyspace is included by default in any backup type where inclusions are defined.

> **Note**
>
> The following keyspaces are needed for WCDBCD functionality and will **always** be included in a backup,
> even when specified for exclusion (either directly or via matching regular expression).
>
> - `system`
> - `system_local`
> - `system_schema`
> - `ecchronos` (if Automatic Repair is enabled)

> **Note**
>
> Only full restore is supported at this moment. The restore process will always remove all
> data (but keeping the tables intact) before starting the restore.

### Excluding the Authentication Keyspace

It is possible to exclude the `system_auth` keyspace in a selective backup, meaning that keyspace
will not be present in the backup. Doing this makes it possible to restore the backup in an
environment which has different authentication credentials.

When restoring a backup where the `system_auth` keyspace is not present, this keyspace will not be
deleted from the existing database. Instead it will be untouched during the restore and all its
data will remain the same as prior to the restore. This includes user credentials, authorization
and auditing configuration. Note that this behavior requires the entire keyspace `system_auth` to
be excluded, excluding only some of its tables is not enough.

To delete the `system_auth` keyspace add its name (or a matching regular expression) to the
exclusions in the `backup-data-model-config.json` file:

```json
{
  "exclusions": [
     {
       "backupType": "exclude_system_auth",
       "keyspaces": [
         {
           "name": "system_auth",
           "tables": []
         }
       ]
     }
  ]
}
```

The behavior of not removing a keyspace at restore is only true for the `system_auth` keyspace.
All other keyspaces and tables will always be removed in the restore process.

### Backup Location Using Amazon S3

It is possible to store the backups in an S3 location instead of using the BR-O.
For this, additional configuration is required, as shown in the example below.

```json
{
  "backupTypes":
  {
    "DEFAULT":
    {
      "location": "aws-s3-v1",
      "s3Region": "",
      "s3Host": "OSMN_HOST",
      "s3Port": 9000,
      "s3BucketName": "bucket_dc1",
      "requireFragmentValidation": true
    }
  },
  "exclusions": [...]
  "inclusions": [...]
}
```

In this example, the backup is configured to be stored in Object Storage MN (OSMN) service.
The fields `location` and `s3BucketName` are mandatory. Additionally, either `s3Region` or
`s3Host` must be set. When `s3Host` is set, `s3Port` must also be set. The default value for
`requireFragmentValidation` is true.


## Configuration of gRPC Maximum Message Size

Backups in WCDBCD are built upon Cassandra's "snapshot" functionality. Each file in the snapshot
corresponds to a backup fragment. As the first step during restore, the Backup and Restore
Orchestrator (BR-O) sends a list of fragments to the Backup and Restore Agent (BR-A). If there are
many fragments (with a lot of metadata), then the size of this list would exceed the maximum gRPC
message size, which is 4 MB by default. To allow for greater message sizes, override the parameter
`brAgent.restore.grpc.maxMessageSize`. The example below doubles the maximum message size to 8 MB
(8388608 bytes).

```yaml
brAgent:
  restore:
    grpc:
      maxMessageSize: 8388608
```
