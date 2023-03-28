# Week 4 — Postgres and RDS

## Provisioning an RDS Instance

```
aws rds create-db-instance \
  --db-instance-identifier cruddur-db-instance \
  --db-instance-class db.t3.micro \
  --engine postgres \
  --engine-version  14.6 \
  --master-username root \
  --master-user-password Test@awsbootcamp! \
  --allocated-storage 20 \
  --availability-zone ca-central-1a \
  --backup-retention-period 0 \
  --port 5432 \
  --no-multi-az \
  --db-name cruddur \
  --storage-type gp2 \
  --publicly-accessible \
  --storage-encrypted \
  --enable-performance-insights \
  --performance-insights-retention-period 7 \
  --no-deletion-protection 
  ```
  
  Use this command to connect to PostgreSQL via PostgreSQL client cli
  
  ```
  psql -Upostgres --host localhost
  ```
  ### PostgreSQL commands
  
  ```
\x on -- expanded display when looking at data
\q -- Quit PSQL
\l -- List all databases
\c database_name -- Connect to a specific database
\dt -- List all tables in the current database
\d table_name -- Describe a specific table
\du -- List all users and their roles
\dn -- List all schemas in the current database
CREATE DATABASE database_name; -- Create a new database
DROP DATABASE database_name; -- Delete a database
CREATE TABLE table_name (column1 datatype1, column2 datatype2, ...); -- Create a new table
DROP TABLE table_name; -- Delete a table
SELECT column1, column2, ... FROM table_name WHERE condition; -- Select data from a table
INSERT INTO table_name (column1, column2, ...) VALUES (value1, value2, ...); -- Insert data into a table
UPDATE table_name SET column1 = value1, column2 = value2, ... WHERE condition; -- Update data in a table
DELETE FROM table_name WHERE condition; -- Delete data from a table

```

## Create cruddur database

```
createdb cruddur -h localhost -U postgres
```
```
psql -U postgres -h localhost
```
[*postgresql docs*](https://www.postgresql.org/docs/current/app-createdb.html)

We can create the database within the PSQL client
```
CREATE database cruddur;
```
### Add UUID Extension
Create a SQL file called schema.sql and import it in backend-flask/db

Add UUID Extension to the schema.sql file

```
CREATE EXTENSION IF NOT EXISTS "uuid-ossp";
```
Run the command below to import script
```
psql cruddur < db/schema.sql -h localhost -U postgres
```

To set password to easily connect to postgresql
```
psql postgresql://postgres:password@localhost:5432/cruddur
```

Set environment variables
```
export CONNECTION_URL="postgresql://postgres:password@localhost:5432/cruddur"
```

Use command below to test connection
```
psql $CONNECTION_URL
```

Set for gitpod environment

```
gp env CONNECTION_URL="postgresql://postgres:password@localhost:5432/cruddur"
```
Set for db Instance

```
gp env PRO_CONNECTION_URL="postgresql://postgres:password@[endpoint]:5432/cruddur"

export PRO_CONNECTION_URL="postgresql://postgres:password@[endpoint]:5432/cruddur"
```

### Creation of tables


```
CREATE EXTENSION IF NOT EXISTS "uuid-ossp";
```

```
DROP TABLE IF EXISTS public.users;
DROP TABLE IF EXISTS public.activities;
```

```
CREATE TABLE public.users (
  uuid UUID DEFAULT uuid_generate_v4() PRIMARY KEY,
  display_name text,
  handle text,
  cognito_user_id text,
  created_at TIMESTAMP default current_timestamp NOT NULL
);
```

```
CREATE TABLE public.activities (
  uuid UUID DEFAULT uuid_generate_v4() PRIMARY KEY,
  user_uuid UUID NOT NULL,
  message text NOT NULL,
  replies_count integer DEFAULT 0,
  reposts_count integer DEFAULT 0,
  likes_count integer DEFAULT 0,
  reply_to_activity_uuid integer,
  expires_at TIMESTAMP,
  created_at TIMESTAMP default current_timestamp NOT NULL
);
```
### Insert content into table
```
INSERT INTO public.users (display_name, handle, cognito_user_id)
VALUES
  ('Andrew Brown', 'andrewbrown' ,'MOCK'),
  ('Andrew Bayko', 'bayko' ,'MOCK');
```
```
INSERT INTO public.activities (user_uuid, message, expires_at)
VALUES
  (
    (SELECT uuid from public.users WHERE users.handle = 'andrewbrown' LIMIT 1),
    'This was imported as seed data!',
    current_timestamp + interval '10 day'
  );
  ```
[https://www.postgresql.org/docs/current/sql-createtable.html](https://www.postgresql.org/docs/current/sql-createtable.html)

### Shell Script to Connect to DB

create a folder to keep all the scripts

mkdir /workspace/aws-bootcamp-cruddur-2023/backend-flask/bin

Connection bash script *bin/db-connect*

```
#! /usr/bin/bash

psql $CONNECTION_URL

```
Change user permission to make it executable

```
chmod u+x bin/db-connect
```
To execute the script:

```
./bin/db-connect
```

### Shell script to drop the database

Drop bash script *bin/db-drop*

```
#! /usr/bin/bash

NO_DB_CONNECTION_URL=$(sed 's/\/cruddur//g' <<<"$CONNECTION_URL")
psql $NO_DB_CONNECTION_URL -c "DROP database cruddur;
```

### Shell script to create the database

Create a *bin/db-create*

```
#! /usr/bin/bash

NO_DB_CONNECTION_URL=$(sed 's/\/cruddur//g' <<<"$CONNECTION_URL")
createdb cruddur $NO_DB_CONNECTION_URL
```

### Shell script to load the schema

Load schema bin/db-schema-load

```
#! /usr/bin/bash

schema_path="$(realpath .)/db/schema.sql"

echo $schema_path

NO_DB_CONNECTION_URL=$(sed 's/\/cruddur//g' <<<"$CONNECTION_URL")
psql $NO_DB_CONNECTION_URL cruddur < $schema_path

```

### Shell script to load the seed data

```
#! /usr/bin/bash

#echo "== db-schema-load"


schema_path="$(realpath .)/db/schema.sql"

echo $schema_path

psql $CONNECTION_URL cruddur < $schema_path
```

### Shell script to setup (reset) everything for our database

```
#! /usr/bin/bash
-e # stop if it fails at any point

#echo "==== db-setup"

bin_path="$(realpath .)/bin"

source "$bin_path/db-drop"
source "$bin_path/db-create"
source "$bin_path/db-schema-load"
source "$bin_path/db-seed"
```
### Install Postgres Client

Add the drivers below in the backend-flask/requirements.txt and install the PostgreSQL driver for python.

```
psycopg[binary]
psycopg[pool]
```
```
pip install -r requirements.txt
```

### Create DB Object and Connection Pool

```
backen-flask/lib/db.py
```
```
from psycopg_pool import ConnectionPool
import os

connection_url = os.getenv("CONNECTION_URL")
pool = ConnectionPool(connection_url)


