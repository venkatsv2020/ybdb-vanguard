# ybdb-vanguard



## yb-voyager migration tool :: MYSQL to YUGABYTEDB

### Export schema and data from the source database

#### Step 1: Export Schema
yb-voyager export schema --export-dir ${GITPOD_REPO_ROOT}/yb-voyager \
        --source-db-type mysql \
        --source-db-host 127.0.0.1 \
        --source-db-user root \
        --source-db-password 'password' \
        --source-db-name chinook

#### Step 2: Analyze Schema
yb-voyager analyze-schema --export-dir ${GITPOD_REPO_ROOT}/yb-voyager --output-format html


#### Step 3: Export Data
yb-voyager export data --export-dir ${GITPOD_REPO_ROOT}/yb-voyager \
        --source-db-type mysql \
        --source-db-host 127.0.0.1 \
        --source-db-user root \
        --source-db-password 'password' \
        --source-db-name chinook



### Import schema and data into the target database

#### Step 4: Import Schema
yb-voyager import schema --export-dir ${GITPOD_REPO_ROOT}/yb-voyager \
        --target-db-host 127.0.0.1 \
        --target-db-user yugabyte \
        --target-db-password yugabyte \
        --target-db-name yugabyte \
        --target-db-schema public

#### Step 5: Import Data
yb-voyager import data --export-dir ${GITPOD_REPO_ROOT}/yb-voyager \
        --target-db-host 127.0.0.1 \
        --target-db-user yugabyte \
        --target-db-password yugabyte \
        --target-db-name yugabyte \
        --target-db-schema public

#### Step 6: Import indexes and triggers
yb-voyager import schema --export-dir ${GITPOD_REPO_ROOT}/yb-voyager \
        --target-db-host 127.0.0.1 \
        --target-db-user yugabyte \
        --target-db-password yugabyte \
        --target-db-name yugabyte \
        --target-db-schema public --post-import-data

### Step 7: Check the imported data status
yb-voyager import data status --export-dir ${GITPOD_REPO_ROOT}/yb-voyager \
