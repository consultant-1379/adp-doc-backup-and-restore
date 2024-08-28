# Agent Configuration Document Database PG

## Configuration of Logical Database Backup

To support logical database backup, the following parameters shall be set as
a mandatory:

-   brAgent.enabled=true

-   brAgent.logicalDBBackupEnable=true

Configuration of `brAgent.backupDataModelConfig`:
-   It is an optional parameter. However, it is mandatory when mTLS is enabled
    in DDB PG where the access mode is 'mTLS+Cleartext parallel' or 'mTLS only'.

-   When mTLS is disabled(i.e. in cleartext mode) and the parameter is not
    defined, Document Database PG will do backup and restore of the default
    logical database (defined in `postgresDatabase`) under the specified backup
    type (defined in `brAgent.backupTypeList`).

-   If the user wants to customize the backup data model, this parameter
    shall be set to the name of a Configmap which defines the custom backup
    data model.

Performs the following steps to prepare and create such a Configmap
before installation of Document Database PG.

1.  Prepare for the Configmap.

    This configmap defines the custom backup data model. Configurable
    parameters are listed in the following tables.

    <table>
    <colgroup>
    <col width="33%" />
    <col width="33%" />
    <col width="33%" />
    </colgroup>
    <tbody>
    <tr class="odd">
    <td colspan="3"><p>Table 1 Generic Configurable Parameters</p></td>
    </tr>
    <tr class="even">
    <td><p>Key Name</p></td>
    <td><p>Description</p></td>
    <td><p>Allowed to change? (Y/N)</p></td>
    </tr>
    <tr class="odd">
    <td><p>Metadata.name</p></td>
    <td><p>Name of the configmap</p></td>
    <td><p>Y</p></td>
    </tr>
    <tr class="even">
    <td><p>Metadata.labels</p></td>
    <td><p>Label of configmap</p></td>
    <td><p>Y</p></td>
    </tr>
    </tbody>
    </table>

    <table>
    <colgroup>
    <col width="33%" />
    <col width="33%" />
    <col width="33%" />
    </colgroup>
    <tbody>
    <tr class="odd">
    <td colspan="3"><p>Table 2 Parameters under <code>data.brm_backup.yaml.backupRestoreMetadataConfig</code></p></td>
    </tr>
    <tr class="even">
    <td><p>Key Name</p></td>
    <td><p>Description</p></td>
    <td><p>Allowed to change? (Y/N)</p></td>
    </tr>
    <tr class="odd">
    <td><p>backupType</p></td>
    <td><p>A specific backup type. It maps to one option in `brAgent.backupTypeList`. </p></td>
    <td><p>Y</p></td>
    </tr>
    <tr class="even">
    <td><p>username</p></td>
    <td><p>The user for the target database. This parameter is optional. If the parameter is not defined, Document Database PG will do backup and restore using the DB owner.</p>
       <p>Notes: The username string require additional restriction from kubernetes, because the username will be part of CR name. The username should not contain character underscore such as "user_db".</p></td>
    <td><p>Y</p></td>
    </tr>
    <tr class="odd">
    <td><p>database</p></td>
    <td><p>The target database name that will be taken in backup and restore.</p>
    <p>Notes: For non-mTLS, this database name must be the same as the one created from parameter <code>--set postgresDatabase=cmdb</code>.</p>
    <p>Notes: For mTLS, this is the database created by SC.</p></td>
    <td><p>Y</p></td>
    </tr>
    <tr class="even">
    <td><p>inOutTables</p></td>
    <td><p>Indicates how the tables in tablesList should be considered. The value can be &quot;in&quot;, &quot;out&quot; or &quot;none&quot;:</p>
    <p>- &quot;in&quot;: Only the tables in &quot;tablesList&quot; ,which will be included in the backup and restore.</p>
    <p>- &quot;out&quot;: The tables in &quot;tablesList&quot;, which will be excluded in the backup and restore.</p>
    <p>- &quot;none&quot;: all tables defined in the database, which will be included in the backup and restore. &quot;tablesList&quot; parameter will be ignored.</p></td>
    <td><p>Y</p></td>
    </tr>
    <tr class="odd">
    <td><p>tablesList</p></td>
    <td><p>The table list.</p></td>
    <td><p>Y</p></td>
    </tr>
    </tbody>
    </table>

    An example of the Configmap:

    **Note:** The file name `brm_backup.yaml` does not allow to change
    in the data section of configmap.

          apiVersion: v1
          kind: ConfigMap
          metadata:
            name: eric-data-document-database-pg-brm-backup-new
            labels:
              app: eric-data-document-database-pg
          data:
            brm_backup.yaml: |
              backupRestoreMetadataConfig:
                  # Supported backup types are listed in "backupType".
                  # In each "backupType", "database" specifies the logical database for backup and restore.
                  # In each "database", "inOutTables" specifies the tables to be included or excluded for
                  # backup and restore. It can take values "in", "out" or "none":
                  #    - "in": Only the tables in "tablesList" will be included in the backup and restore.
                  #    - "out": The tables in "tablesList" will be excluded in the backup and restore.
                  #    - "none": all tables defined in the database will be included in the backup and restore.
                  #     "tablesList" parameter will be ignored.
                  - backupType: configuration-data-ah
                    # username is an optional parameter. If the parameter is not defined, Document Database PG will
                    #    do backup and restore with db owner.
                    username: ahuser1
                    database: ahdb1
                    inOutTables: in
                    tablesList:
                      - pgbench_accounts
                      - pgbench_branches

                   - backupType: configuration-data-ah
                    username: ahuser2
                    database: ahdb2
                    inOutTables: in
                    tablesList:
                      - table_A
                      - table_b

                    - backupType: configuration-data-cm
                    username: cmuser1
                    database: cmdb1
                    inOutTables: in
                    tablesList:
                      - table_c
                      - table_d

                    - backupType: configuration-data-cm
                    username: cmuser2
                    database: cmdb2
                    inOutTables: in
                    tablesList:
                      - pgbench_accounts
                      - pgbench_branches

2.  Create the configmap

        $kubectl create -f <configmap-file> --namespace=<namespace>

    For example:

        $kubectl create -f configmap-brm-backup-new.yaml --namespace=example

3.  Set the configmap related parameters when do the helm installation.

    Example of parameter setting:

    `--set brAgent.enabled=true,brAgent.logicalDBBackupEnable=true,brAgent.backupDataModelConfig=<configmap-name>`

Configuration of `brAgent.backupTypeList`:

-   It is an optional parameter. If it is not defined, Document Database PG only supports the `DEFAULT` backup type.


-   When mTLS is enabled, multiple backup types are supported from Document Database PG 6.1.0 by specifying a backup type list in the "backupTypeList".


-   When mTLS is disabled, only one backup type is supported. The first backup type in the "backupTypeList" is taken.


-   Backup and restore is performed in Document Database PG according to the backup type(s) received from Backup and Restore Orchestrator. If the “DEFAULT” backuptype is received, all the supported backup type(s) configured in "brAgent.backupTypeList" will be considered.

    Example of parameter setting:

    `--set brAgent.backupTypeList[0]=configuration-data-ah,brAgent.backupTypeList[1]=configuration-data-cm`


## Configuration of Service Level Backup

To support service level backup, the following parameters are set as
a mandatory:

-   brAgent.enabled=true

-   brAgent.logicalDBBackupEnable=false

This level is appropriate for backing up and restoring the whole database cluster in Document
Database PG service. Please know that the restoration of this level
will result to the restart of the Postgres pods. Thus, there will
be a service downtime around 0.5 ~ 2 min during the data restoration.