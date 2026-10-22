---
link:
---
I am attempting to have my docker db container automatically populate the database with a dataset when created. According to the [mariadb documentation](https://hub.docker.com/_/mariadb/), there is an `docker-entrypoint-initdb.d` folder in the volume that can be used for this purpose.

I set up my `docker-compose.yml` file to mirror examples that I have found on StackOverflow, but am still unable to get my SQL scripts to execute. Here are the relevant pieces of my `docker-compose.yml` file:

`version: '3.6' services:    db:     image: mariadb:10.5.4-focal     container_name: db     volumes:       - ./cms/conf/mysql/data/:/var/lib/mysql/:rw       - ./cms/sql/:/docker-entrypoint-initdb.d/:ro     environment:       - MYSQL_ROOT_PASSWORD=password       - MYSQL_USER=root       - MYSQL_PASSWORD=password       - MYSQL_DATABASE=wordpress     restart: always    adminer:     image: adminer:4.7.7-standalone     container_name: adminer     links:       - db     ports:       - 8080:8080     restart: always`

Next to the `docker-compose.yml` file, I have a `cms/sql/init.sql` file with the following contents:

``CREATE DATABASE /*!32312 IF NOT EXISTS*/ `test` /*!40100 DEFAULT CHARACTER SET utf8mb4 */; USE `test`;``

Prior to each test iteration, I clean up any old/cached Docker content:

1. `$ docker-compose down -v` I've found this doesn't actually remove the db volumes. Running a `docker volume ls` still shows past volumes present.
2. `$ docker volume prune` This clears out the volumes. Running `docker volume ls` after running this shows an empty list.
3. `$ rm -rf ./cms/conf/mysql` This cleans out any actual filesystem changes that the db container made.
4. `$ docker-compose up -d --build` To rebuild the containers. This should trigger my initialization SQL scripts.

Docker runs, and the `db` container gets successfully created. I am able to access the `adminer` container running on `localhost:8080`. I am able to log in as the `root` user with the password specified in the `docker-compose.yml` file.

The Adminer interface shows the standard MySQL databases (`information_schema`, `mysql`, `performance_schema`) and additionally shows the `wordpress` database defined in the `MYSQL_DATABASE` value of the `docker-compose.yml` file. But my `test` database from the `init.sql` is nowhere to be found.

I found some other similar StackOverflow questions that have had similar problems. I've followed the accepted answers, and am still unable to get any SQL scripts to get executed.

1. [docker-compose.yml, postgress, how to setup db, tables and prepop in an init.sql file?](https://stackoverflow.com/questions/53729813/docker-compose-yml-postgress-how-to-setup-db-tables-and-prepop-in-an-init-sql/53730572#53730572)
2. [Issue with docker compose initial DB setup once](https://stackoverflow.com/questions/50703184/issue-with-docker-compose-initial-db-setup-once)
3. [mySQL init scripts not running with docker-compose](https://stackoverflow.com/questions/59657489/mysql-init-scripts-not-running-with-docker-compose)
4. [Create database on docker-compose startup](https://stackoverflow.com/questions/43322033/create-database-on-docker-compose-startup)