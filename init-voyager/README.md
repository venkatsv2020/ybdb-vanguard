# ybdb-vanguard-enablement

## yb-voyager migration tool :: MYSQL to YUGABYTEDB

### Export schema and data from the source database

#### Step 0: Load data
Run the following from `mysql` shell
```
source init-voyager/chinook.sql
```

Run Step 1 to Step 7 from `yb-voyager` shell
#### Step 1: Export Schema
```
yb-voyager export schema --export-dir ${GITPOD_REPO_ROOT}/voyager-data \
        --source-db-type mysql \
        --source-db-host ${INSTANCE} \
        --source-db-user root \
        --source-db-password ${SECRET} \
        --source-db-name Chinook
```

#### Step 2: Analyze Schema
```
yb-voyager analyze-schema --export-dir ${GITPOD_REPO_ROOT}/voyager-data --output-format html
```


#### Step 3: Export Data
```
yb-voyager export data --export-dir ${GITPOD_REPO_ROOT}/voyager-data \
        --source-db-type mysql \
        --source-db-host ${INSTANCE} \
        --source-db-user root \
        --source-db-password ${SECRET} \
        --source-db-name Chinook
```

### Import schema and data into the target database

#### Step 4: Import Schema
```
yb-voyager import schema --export-dir ${GITPOD_REPO_ROOT}/voyager-data \
        --target-db-host ${INSTANCE} \
        --target-db-user yugabyte \
        --target-db-password yugabyte \
        --target-db-name yugabyte \
        --target-db-schema public
```

#### Step 5: Import Data
```
yb-voyager import data --export-dir ${GITPOD_REPO_ROOT}/voyager-data \
        --target-db-host ${INSTANCE} \
        --target-db-user yugabyte \
        --target-db-password yugabyte \
        --target-db-name yugabyte \
        --target-db-schema public
```

#### Step 6: Import indexes and triggers
```
yb-voyager import schema --export-dir ${GITPOD_REPO_ROOT}/voyager-data \
        --target-db-host ${INSTANCE} \
        --target-db-user yugabyte \
        --target-db-password yugabyte \
        --target-db-name yugabyte \
        --target-db-schema public --post-import-data
```

### Step 7: Check the imported data status
```
yb-voyager import data status --export-dir ${GITPOD_REPO_ROOT}/voyager-data
```
