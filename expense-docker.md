Using docker containers, we will run our `expense` project

* Here, we don't need to install mysql (database), nodejs (backend) and nginx (frontend) servers. We will take official image from dockerhub.

## MySQL (Database)
* Here we will use mysql server official image of version 8.0
* We can run few sql commands to configure the server 
    * Here we will use backend to connect to mysql server and run the queries
    * To run backend, first we need to provide root password of mysql server
    * Backend script should be saved with `.sql` extension and should be copied to `/docker-entrypoint-initdb.d`

**CODE**
```
FROM mysql:8.0
ENV MYSQL_ROOT_PASSWORD=ExpenseApp@1 
ADD scripts/*.sql /docker-entrypoint-initdb.d
    # MYSQL_DATABASE=transactions \
    # MYSQL_USER=expense \
    # MYSQL_PASSWORD=ExpenseApp@1
```
**Explanation**
* `FROM` instruction will pull mysql:8.0 from docker hub
* `ENV` instruction will set root password of mysql server
* `ADD` instruction will copy the script from local to the specified location
    * It will create a schema with name `transactions`
    * Creates a user with name `expense`
    * Sets password for expense user as `ExpenseApp@1`

**After writing Dockerfile run the follwing commands**
* build docker image: `docker build -t mysql:v1 .` 
* run docker container: `docker run -d mysql:v1` 
* access docker container: `docker exec -it <container-id> bash` 
* login to mysql-server: `mysql -u root -pExpenseApp@1`
* list databases: `show databases;`
* select database: `use transactions;`
* list tables: `show tables;` 

Here, sql commands are typically used in sequence to first check what databases are available, then select a specific database, and finally view the tables within that database.

## NodeJS (Backend)
