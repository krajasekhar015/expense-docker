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
Now, if we browse the IP address, we will get this page. Then delete the container
```
docker rm -f containerid
```
 