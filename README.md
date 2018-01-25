# Liquibase Test

This simple repository specifies a sample Liquibase project. The instructions below
use this project to demonstrate how Liquibase can be run within a Docker container
to apply a changeset to a pgsql database.

## Pre-requisites
This sample leverages Docker Compose to simplify the orchestration between Docker
containers. It simplifies the steps necessary to execute the example, but is not
necessary for running Liquibase within a Docker container. In order to run the
example below, you must have the Docker tool versions below:
* Docker Engine 1.13.0+
* Docker Compose 1.17.0+

## Example
### Step 1 - Start the pgsql in a detached container

```bash
> docker-compose up -d db
```

### Step 2 - Run pgsql client to create a new table in the database

```bash
> docker-compose run --rm psql  
```

```psql
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

default=# \q
```

### Step 3 - Run Liquibase container to preview the update sql
```bash
> docker-compose run --rm liquibase updateSQL
```

### Step 4 - Run Liquibase container to execute the update
```bash
> docker-compose run --rm liquibase update
```

### Step 5 - Run pgsql client to validate the update occurred
```bash
> docker-compose run --rm psql
```

```psql
default=# \dt
List of relations
Schema |         Name          | Type  | Owner
--------+-----------------------+-------+-------
public | bar                   | table | admin
public | databasechangelog     | table | admin
public | databasechangeloglock | table | admin
public | foo                   | table | admin
public | test                  | table | admin
(5 rows)

default=# \q
```

### Step 6 - Run Liquibase container to preview that a subsequent run does not attempt to create the tables again
```bash
> docker-compose run --rm liquibase updateSQL
```

(the update sql should indicate that no changes are required)

### Step 7 - Cleanup the pgsql container and related volumes
```bash
> docker-compose down --volumes
```

## Usage
To run the Liquibase Docker container against your own Postgres database, substitue the
following environment vars either in your local environment or in the `.env` file.

* POSTGRES_HOST-- hostname of the Postgres database server (default=db).
* POSTGRES_PORT-- port number of the Postgres database server (default=5432).
* POSTGRES_USER-- username of the Postgres database user (default=admin).
* POSTGRES_PASSWORD-- password of the Postrgres database user (default=pw).
* POSTGRES_DB-- name of the database on the Postgres database server (default=default)

In order to prevent the test Docker containers from launching, suppress the override
file for Docker Compose by specifying the main compose file. As an example

```bash
> docker-compose -f docker-compose.yml run --rm liquibase updateSQL
```
