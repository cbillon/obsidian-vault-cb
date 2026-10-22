---
tags:
  - docker-compose
  - database
  - start
---
I would like to create a MySQL database using environment variables in docker-compose.yml file.

There is also an option to provide an init file for `mysql` container which will be applied each time a container is created.

```
# Source - https://stackoverflow.com/a/43331395

database:
    image: mysql:5.7
    ports:
        - "33061:3306"
    command: --init-file /data/application/init.sql
    volumes:
        - ./init.sql:/data/application/init.sql
    environment:
        MYSQL_ROOT_USER: root
        MYSQL_ROOT_PASSWORD: secret
        MYSQL_DATABASE: homestead
        MYSQL_USER: root
        MYSQL_PASSWORD: secret

```

Such file (`init.sql`) could contain your initial database structure and data - for example:

```
CREATE DATABASE IF NOT EXISTS dev;
CREATE DATABASE IF NOT EXISTS test;
USE dev;
CREATE TABLE IF NOT EXISTS (...);

```


The database is probably already initialized and the configuration is stored in `/var/lib/mysql`. Since you defined a volume for that location the config will survive a restart. The MySQL image will not reconfigure the database over and over again, it only does this once.

```
volumes:
    - dbdata:/var/lib/mysql
```
If your database is empty you can reset the database by performing `docker compose down -v` where the `-v` removes the volumes defined in the volume section. See [https://docs.docker.com/compose/reference/down/](https://docs.docker.com/compose/reference/down/). On the next `docker-compose up` the MySQL image will start fresh and will initialize the database with the configuration you've provided thru the environment section.

Here are the steps I took to solve creating multiple database and users in the MySQL docker image:

1. Create the init file (the Docker image recognizes .sh, .sql, and .sql.gz files) named `setup.sql` in the local directory named `.docker`
2. Place the commands inside `setup.sql` (see below for an example)
3. Mount the setup script into the directory `f` within the `docker-compose.yaml` file (see below for an example)
4. Run `docker-compose up -d` and MySQL will run the code inside `setup.sql`

> Note: the script will run files alphabetically, so keep that in mind.

**Example docker-compose.yaml**
```
# Source - https://stackoverflow.com/a/59653823
services:
    mysql:
        image: mysql
        ports:
            - 3306:3306
        environment:
            MYSQL_ROOT_PASSWORD: SomeRootPassword1!
            MYSQL_USER: someuser
            MYSQL_PASSWORD: Password1!
            MYSQL_DATABASE: somedatabase
        volumes:
            - .docker/setup.sql:/docker-entrypoint-initdb.d/setup.sql
            - db_data:/var/lib/mysql
volumes:
    db_data:

```
Example setup.sql

```
-- Source - https://stackoverflow.com/a/59653823

-- create the databases
CREATE DATABASE IF NOT EXISTS projectone;

-- create the users for each database
CREATE USER 'projectoneuser'@'%' IDENTIFIED BY 'somepassword';
GRANT CREATE, ALTER, INDEX, LOCK TABLES, REFERENCES, UPDATE, DELETE, DROP, SELECT, INSERT ON `projectone`.* TO 'projectoneuser'@'%';

FLUSH PRIVILEGES;

```
This answer works when you need to **grant privileges to users other than root** , whether they are created using the environmental variable MYSQL_USER or in the setup.sql file.

One thing I use to do when building a new docker container is understand what the image I pull from does when is builded.

In your docker-compose.yml tou have this

```
# Source - https://stackoverflow.com/a/48981915
# The Database
database:
  image: mysql:5.7

```
This is the image you pull from, "mysql:5.7"

Dockerhub is a repository where you can find info of this images.
Do a google search "mysql:5.7 dockerhub"
First result is [https://hub.docker.com/_/mysql/](https://hub.docker.com/_/mysql/)
There you have your image 5.7, if you click on 5.7 you have this

[https://github.com/docker-library/mysql/blob/607b2a65aa76adf495730b9f7e6f28f146a9f95f/5.7/Dockerfile](https://github.com/docker-library/mysql/blob/607b2a65aa76adf495730b9f7e6f28f146a9f95f/5.7/Dockerfile)

Which is the Dockerfile from the image, you can have a look at interesting things that happen when building the image.

One of this is ENTRYPOINT ["docker-entrypoint.sh"]

This is the file that got executed when image is ready
I you go one level up in the repo you will see this file

[https://github.com/docker-library/mysql/tree/607b2a65aa76adf495730b9f7e6f28f146a9f95f/5.7](https://github.com/docker-library/mysql/tree/607b2a65aa76adf495730b9f7e6f28f146a9f95f/5.7)
The you can see your environment variables being used to create new database etc..

```
# Source - https://stackoverflow.com/a/48981915

file_env 'MYSQL_DATABASE'
        if [ "$MYSQL_DATABASE" ]; then
            echo "CREATE DATABASE IF NOT EXISTS \`$MYSQL_DATABASE\` ;" | "${mysql[@]}"
            mysql+=( "$MYSQL_DATABASE" )
fi

```

Example wit docker compose create database

```
# Source - https://stackoverflow.com/a/4332288

volumes:
 dbdata:

services:
 mysql:
  image: mysql:5.7
  container_name: mysql
  volumes:
    - dbdata:/var/lib/mysql
  restart: always
  environment:
    - MYSQL_ROOT_PASSWORD=secret
    - MYSQL_DATABASE=homestead
    - MYSQL_USER=root
    - MYSQL_PASSWORD=secret
  ports:
    - "33061:3306"

```

Then start with docker compose up -d
```-- Source - https://stackoverflow.com/a/43322883

$ docker ps
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS                     NAMES
a3567fb78d0d        mysql:5.7           "docker-entrypoint..."   2 minutes ago       Up 2 minutes        0.0.0.0:33061->3306/tcp   mysql

docker exec -it a3567fb78d0d bash
root@a3567fb78d0d:/# mysql -u root -p homestead
Enter password:
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 7
Server version: 5.7.17 MySQL Community Server (GPL)
Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

mysql> show databases;
+--------------------+
| Database           |
+--------------------+
| information_schema |
| homestead          |
| mysql              |
| performance_schema |
| sys                |
+--------------------+
5 rows in set (0.00 sec)

```
Your volume will be persisted in docker volume `nameoffolder_dbdata` (`/var/lib/docker/volumes/...`)

If I understand your question correctly, you want to to have a container with a specific database in it. Like have a MySQL container with `CREATE DATABASE mydb`, etc. already executed. If so you need to use `docker-entrypoint-initdb.d`
When the official MySQL container is started for the first time, a new database will be created first. Then it will execute files with extensions .sh, .sql and .sql.gz that are found in `/docker-entrypoint-initdb.d`. So all you need to do is create `/docker-entrypoint-initdb.d` directory and put your initialization script there.

The "Initializing a fresh instance" here [https://hub.docker.com/_/mysql/](https://hub.docker.com/_/mysql/) states to copy all .sql,.sh,.sql.gz files into `/docker-entrypoint-initdb.d` folder. Below is the docker-compose file that I am using for plenty of my projects and all working fine with latest version of MySQL

```
# Source - https://stackoverflow.com/a/66006202

services:   
  mysql:
      environment:
          - MYSQL_DATABASE=root
          - MYSQL_ROOT_PASSWORD=changeme
          - MYSQL_USER=dbuser
          - MYSQL_PASSWORD=changeme
      command: 
         - --table_definition_cache=100
         - --performance_schema=0
         - --default-authentication-plugin=mysql_native_password
         - --innodb_use_native_aio=0
      volumes: 
          - ./init:/docker-entrypoint-initdb.d
          - ./dbdata:/var/lib/mysql
      container_name: mysqldb
      image: mysql  

```
and my init folder has an `init.sql` file containing the entire SQL data dump that I need to recreate when my container starts.

Below commands are used to limit memory consumed by MySQL container to 100 MB:
```
         - --table_definition_cache=100
         - --performance_schema=0

```
where `dbdata` is a folder you need to create on your host system.
Keep in mind if you want to docker-compose automatically create database from MYSQL_DATABASE your data folder (**dbdata**) must be **empty**!
