## Docker Networking
**Home Network**

![alt text](images/docker-network.drawio.svg)

* Here, Internet Service Provider (ISP) will provide internet to home router.
*  one external and one internai IPs to the router. Outside of our home is external IP and inside our home is internal IP
ISP provides `public IP address` (External) and `private IP address` (Internal) to the router. External IP address will be accessed from outside of home and internal IP address will be accessed internally inside home
* So, here router provides IP addresses to laptops and mobiles and Internet Service provider (ISP) provides IP address for router 
* In general, internal IP address range will be like `192.168.1.0`
* For, every laptop there will be an interface like Wi-Fi, Ethernet etc., If we connect through Ethernet we will get one IP Address and if we connect through Wi-Fi we will get another IP address. 
* If we turn-off Wi-Fi and connect to mobile network, then SIM provider will provide the IP Address


**Docker Network**

If we check the network interfaces on the system using `ifconfig`, we will get different network interfaces on a Linux system like `docker0`, `ens5`, and `lo`

![alt text](images/ifconfig.png)

**1. docker0** `Virtual bridge for Docker containers to communicate with each other and the host`

**Type:**
Virtual Network Interface

**Description:** 
* Here, docker creates seperate virtual network interface i.e., `docker0`
* `docker0` provides ip address to the containers which is called `bridge network`
* It allows containers to communicate with each other and with the host system
* It acts as modem to the containers inside VM
* Containers connected to this bridge can access the internet through the host's network. 

**Explanation:**
* Suppose, if we create a container then docker0 will give IP address to that container 

**2. ens5** `Physical or virtual interface used for external network communication`

**Type:** 
Physical or Virtual Network Interface 

**Description:** 
Every VM's will get internet access through Ethernet from AWS ISP. Here, `ens5` is the network provided by AWS ISP

**3. lo (Loopback)** `Loopback interface for internal communication within the host`

**Type:** 
Virtual Network Interface

**Description:**
The loopback interface is a special virtual interface used for internal communication within the host 

## IP Address Assignment for docker0

The IP address for the `docker0` interface is assigned by Docker itself during the initialization of the Docker daemon. Specifically, Docker configures the docker0 bridge network with a default subnet (commonly 172.17.0.0/16) and assigns the IP address 172.17.0.1 to the docker0 interface

**Assignment Process**

**1. Docker Daemon Startup:**
When you install Docker and start the Docker daemon (`dockerd`), it automatically creates the `docker0` bridge network interface if it doesn't already exist.

**2. Default Configuration:**
Docker has a default IP address range it uses for the docker0 interface, typically `172.17.0.1` with a subnet mask of `255.255.0.0`. This configuration allows Docker to allocate IP addresses to containers connected to this bridge.

**3. IP Address Allocation:**
The Docker daemon automatically selects an available IP address from the predefined subnet for the `docker0` interface.

**4. Dynamic Configuration:**
If you wish to customize the IP address or subnet for the docker0 interface, you can do so by modifying the Docker daemon’s configuration file (e.g., `/etc/docker/daemon.json`) before starting the Docker service.

Example of a configuration to change the default subnet:
```
{
  "bip": "172.26.0.1/16"
}
```

**5. Dynamic Management:**
Docker’s internal DHCP server manages the IP address allocation for containers connected to this bridge network, assigning them unique IP addresses within the `docker0` subnet.

**Summary**
The Docker daemon is responsible for creating the docker0 interface and assigning its IP address when it starts. This allows it to manage IP address allocation for containers connected to the bridge network


## Types of Docker Networking
1. Host Network
2. Bridge Network (Default docker network)
3. Overlay Network (Communication between multiple docker hosts)
   * Overlay networks are designed to allow communication between containers that are running on different Docker hosts

![alt text](images/networks.png)

* In the above snapshot, we cannot see overlay network because overlay network is communication between multiple hosts. Here in snapshot, it is within the host.

**1. Host Network**
A `host network` in Docker is a networking mode that allows docker containers to share the host's networking stack. When a container is run in host mode, it does not get its own network namespace; instead, it uses the host's network interfaces directly

* Containers which are running on host network will not get ip address i.e., containers are sharing host IP address
* Containers on host network will open host ports
    * mysql --> 3306
    * backend --> 8080
    * frontend --> 80

> When specific port is opened in a server, same port cannot be used by another container in that server. Since containers are using host network, they should run on different ports.

*Assigning Host Network to a container:*
```
docker run -d --name containername --network host imagename:version
```

*Check Whether the port is opened or not:*
```
sudo netstat -lntp
```

*Check Container Full Details:*
```
docker inspect containername
```
* Here, we can see that no IP is allocated to this container, since we have created in host network.

* In backend, we use `ENV DB_HOST="mysql"` but it won't work in host network, since it is of bridge network. When we run the backend we will get an error. Here, bridge network connects with the container names but not the host network
* So, we need to give `ENV DB_HOST="localhost"` since this is the communication between two services within same server
* When we run frontend, we may not expose the port numbers since it is not an container network

**Disadvantages**
* There is no isolated network for docker containers

> We need to remember that Container ports are not host ports, they are logical ports

**2. Bridge Network**


* In default network, DNS doesn't works internally. It doesn't have any components related to DNS
* Here, we won't use default bridge network. We will create one bridge network
```
docker network create expense
```

![alt text](images/bridge-network.png)

Now, if we run containers, we can see the IP address of the container allocated through bridge network

* When we run frontend, we need to expose the port in command

**3. Overlay Network** 
* One network doesn't enough for us, so sometimes we have to use multiple networks 
* Two containers in two different networks can connect and communicate through modem 
* Keeping docker in multiple networks is risky, so for this we need orchestration 

