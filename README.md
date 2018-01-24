# Liquibase Test

This simple repository specifies a sample Liquibase project. The instructions below use this project to demonstrate how Liquibase can be run within a Docker container to apply a changeset to a pgsql database.

## Example
### Step 1 - Create a data directory
mkdir ~/data/pgsql-test

### Step 2 - Start a pgsql compute container
docker run --name pgsql-cc -v /home/stu/data/crowdcompass:/var/lib/postgresql/data -e POSTGRES_USER=admin -e POSTGRES_PASSWORD=pw -e POSTGRES_DB=default -d postgres

### Step 3 - Run a pgsql client tools container
docker run -it --rm --link pgsql-cc:pgsql-cc --name pg-tools postgres pgsql -h pgsql-cc -U admin -d default
     (enter password "pw", from step 2)

### Step 4 - Create a table "test" in database "default"
\l
\c default
\dt
create table test (id int);
\dt
select * from test;
\q

### Step 5 - Clone Liquibase test project
git clone git@github.com:stuartthompson/liquibase-test.git liquibase-test

