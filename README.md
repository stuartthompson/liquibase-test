# Liquibase Test

This simple repository specifies a sample Liquibase project. The instructions below
use this project to demonstrate how Liquibase can be run within a Docker container
to apply a changeset to a pgsql database.

## Pre-requisites
* Docker Engine #.#+
* Docker Compose #.#+

## Example
### Step 1 - Start the pgsql in a detached container

```bash
> docker-compose up -d db
```

### Step 2 - Run pgsql client to create a new table in the database

```bash
> docker-compose run psql  
```

```sql
default=# \dt
No relations found.

default=# create table test (id int);
CREATE TABLE

default=# \dt
List of relations
Schema | Name | Type  | Owner
--------+------+-------+-------
public | test | table | admin
(1 row)

default=# select * from test;
id
----
(0 rows)

\q
```

### Step 3 - Run Liquibase container to preview the update sql
```bash
> docker-compose run -rm liquibase updateSQL
```

### Step 4 - Run Liquibase container to execute the update
```bash
> docker-compose run -rm liquibase update
```

### Step 5 - Run pgsql client to validate the update occurred
```bash
> docker-compose run -rm psql
```

```sql
\dt
List of relations
Schema |         Name          | Type  | Owner
--------+-----------------------+-------+-------
public | bar                   | table | admin
public | databasechangelog     | table | admin
public | databasechangeloglock | table | admin
public | foo                   | table | admin
public | test                  | table | admin
(5 rows)

\q
```

### Step 6 - Run Liquibase container to preview that a subsequent run does not attempt to create the tables again
```bash
docker-compose run liquibase updateSQL
```

    (the update sql should indicate that no changes are required)
