## Docker Volumes
* Containers are ephemeral. Once you remove container you lose data. Data is not persisted by default 
* We need some kind of mechanism to store the data even if the container is removed. 
* This can be possible throgh the volumes 

They are two types of volumes:
1. Un-named volumes 
2. Named volumes

**1. Un-named Volumes**
* To persist the data we have to Mount the storage in host to the container 
* At some where in the host we have to create the directory and it should be mount to the container

* Create a directory of any name in /home/ec2-user
```
mkdir nginx-data
```
* Run the following command
```
docker run -d -p -v /home/ec2-user/nginx-data/:/usr/share/nginx/html
```
* Here, whatvever the data present in /usr/share/nginx/html (container path) will be mounted in /home/ec2-user/nginx-data (host path)
    * `v` represents volumes

* Login to the container
```
docker exec -it containerid bash
```
* Go to path `/usr/share/nginx/html/` and create a hello.html file using echo command
```
echo "Hello Docker volumes" > hello.html
```
Now, if we browse the IP address, we can access hello.html page. Then delete the container
```
docker rm -f containerid
```
Now, again execute `docker run -d -v /home/ec2-user/nginx-data/:/usr/share/nginx/html -p 80:80 nginx` command and if we browse it we can still access the hello.html page. Because it is mounted in host path

Now, go to path `/home/ec2-user/nginx-data/` and create a volume.html file using echo command
```
echo "<h1>Hello, this is from host storage</h1>" > volume.html
```
If we access the container IP address in the browser, we can see this page

* So, by this we can persist the data instead of loosing it 
* Unnamed volumes are not in the docker control. We have to manage the directory and its related data, permissions etc

**2. Named Volumes**
* Here, we create docker volumes with the help of docker commands
* Use the following command to see list of commands to create and manage docker volumes
```
docker volume --help
```
* Using docker commands we can manage the life-cycle of the volumes

* Create a docker volume
```
docker volume create volumename
```
* Listing the docker volumes
```
docker volume ls
```
Here, we can see volumes with some random names which is difficult to understand 
   * But, using proper-naming convention we have created our docker volume

* If we inspect the docker volume, we can see the location where volume is mounted on host path
```
docker inspect volumename
``` 

* It is better to have docker volumes under the control of docker. It is recommended to use named volumes instead of un-named volumes

```
docker run -d -v nginx-html:/usr/share/nginx/html -p 80:80 nginx
```

> If we remove the container, volume won't get deleted
