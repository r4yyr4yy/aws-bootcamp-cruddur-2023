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
Run the command below to create Extension
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

