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
* Here we will use nodejs server official image of version 20
* We need to download our code from `https://expense-builds.s3.us-east-1.amazonaws.com/expense-backend-v2.zip` and extract in backend folder

**CODE**
```
FROM node:20
EXPOSE 8080
ENV DB_HOST="mysql"
RUN mkdir /opt/server
WORKDIR /opt/server
COPY package.json .
COPY *.js .
RUN npm install
CMD ["node", "index.js"]
```

**Explanation**
* `FROM` instruction will pull node:20 (nodeJS) from docker hub
* Since our application is opening port-no.8080, we will expose it to user through `EXPOSE` instruction
* Using `ENV` instruction, we will set the DB_HOST value (Since, we cannot create R53 records inside containers, we need the database container name as mysql)
* Using `RUN` instruction we will create a directory `/opt/server`
* To move to that directory we will use `WORKDIR` instruction
* Now, we will copy *.json & *.js files to this directory using `COPY` instruction
* Install dependencies using `RUN` instruction
* Using `CMD` instruction, we will run the `node index.js` to start the backend 

**Troubleshooting**
* To check logs: `docker logs container-name`
* To check connection with database: `telnet mysql 3306`

> We cannot communicate between containers in docker using default network. We need to create our own network

**Creating Own Network in Docker**
* Using `docker network --help` command, we can see the commands used in docker network.
* Command to create docker network
```
docker network create name
```
* Command to list docker network
```
docker network ls
```
* command to connect container to docker network
```
docker network connect network-name container-name 
```
* command to disconnect container from default docker network
```
docker network disconnect bridge container-name
```
Here, bridge is the name of default docker network

* command to set network and name to a container
```
docker run -d --network network-name --name container-name image-name
```

**Installing telnet in docker container**
First we need to check the underlying OS
```
cat /etc/*release
```
Suppose if it is Debian OS, then the command is
```
apt update && apt install telnet
```
* After completing troubleshooting, we can uninstall telnet

## Nginx (Frontend)
* Here we will use nginx server official image of latest version
* We need to download our code from `https://expense-builds.s3.us-east-1.amazonaws.com/expense-frontend-v2.zip` and extract in frontend folder

**CODE**
```
FROM nginx
COPY code /usr/share/nginx/html
RUN rm -rf /etc/nginx/nginx.conf
RUN rm -rf /etc/nginx/conf.d/default.conf
COPY nginx.conf /etc/nginx/nginx.conf
```

**Explanation**
* `FROM` instruction pull nginx server (latest version) from docker hub 
* `COPY` instruction will copy  the code from local to specified location in nginx server
* Bydefault, there will be nginx.conf and default.conf files in nginx server. We need to remove them using `RUN` instruction
* Now, using `COPY` instruction we will copy nginx.conf to the specified location

**Docker Directory Location**
```
cd /var/lib/docker/
```


