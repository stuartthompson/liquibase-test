# Liquibase Test

This simple repository specifies a sample Liquibase project. The instructions below use this project to demonstrate how Liquibase can be run within a Docker container to apply a changeset to a pgsql database.

## Example
### Step 1 - Create a data directory
mkdir ~/data/pgsql-test

### Step 2 - Start a pgsql compute container
docker run --name pgsql-test -v ~/data/pgsql-test:/var/lib/postgresql/data -e POSTGRES_USER=admin -e POSTGRES_PASSWORD=pw -e POSTGRES_DB=default -d postgres

### Step 3 - Run a pgsql client tools container
docker run -it --rm --link pgsql-test:pgsql-test --name pg-tools postgres pgsql -h pgsql-cc -U admin -d default
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
git clone git@github.com:stuartthompson/liquibase-test.git ~/home/dev/github.com/stuartthompson/liquibase-test

### Step 6 - Run Liquibase container to preview the update sql
docker run --rm -v ~/dev/github.com/stuartthompson/liquibase-test:/liquibase --link pgsql-test:pgsql-test -e "LIQUIBASE_CHANGELOG=/liquibase/changelog.json" -e "LIQUIBASE_URL=jdbc:postgresql://pgsql-test:5432/default" -e "LIQUIBASE_USERNAME=admin" -e "LIQUIBASE_PASSWORD=pw" webdevops/liquibase:postgres updateSQL

### Step 7 - Run Liquibase container to execute the update
docker run --rm -v ~/dev/github.com/stuartthompson/liquibase-test:/liquibase --link pgsql-test:pgsql-test -e "LIQUIBASE_CHANGELOG=/liquibase/changelog.json" -e "LIQUIBASE_URL=jdbc:postgresql://pgsql-test:5432/default" -e "LIQUIBASE_USERNAME=admin" -e "LIQUIBASE_PASSWORD=pw" webdevops/liquibase:postgres update

### Step 8 - Run pgsql client tools to validate the update occurred
docker run -it --rm --link pgsql-test:pgsql-test --name pg-tools postgres pgsql -h pgsql-cc -U admin -d default
     (enter password "pw", from step 2)
\c default
\dt
     (the list of tables should now include foo and bar)
\q

### Step 9 - Run Liquibase container to preview that a subsequent run does not attempt to create the tables again
docker run --rm -v ~/dev/github.com/stuartthompson/liquibase-test:/liquibase --link pgsql-test:pgsql-test -e "LIQUIBASE_CHANGELOG=/liquibase/changelog.json" -e "LIQUIBASE_URL=jdbc:postgresql://pgsql-test:5432/default" -e "LIQUIBASE_USERNAME=admin" -e "LIQUIBASE_PASSWORD=pw" webdevops/liquibase:postgres updateSQL
     (the update sql should indicate that no changes are required)