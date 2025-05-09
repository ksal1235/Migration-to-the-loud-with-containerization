# Migration to the Сloud with containerization (Docker & Docker Compose):

In this project, we will use a CI tool that is already well-known to us Jenkins - for Continous Integration (CI). So, when it is time to write Jenkinsfile.

To begin our migration project from VM based workload, we need to implement a Proof of Concept (POC). In many cases, it is good to start with a small-scale project with minimal functionality to prove that technology can fulfill specific requirements. So, this project will be a precursor before you can move on to deploy enterprise-grade microservice solutions with Docker. 

We can start with our own workstation or spin up an EC2 instance to install Docker engine that will host our Docker containers.

So, let us migrate the Tooling Web Application from a VM-based solution into a containerized one.


## Install Docker and prepare for migration to the Cloud

First, we need to install Docker Engine, which is a client-server application that contains:

- A server with a long-running daemon process dockerd.
- APIs that specify interfaces that programs can use to talk to and instruct the Docker daemon.
- A command-line interface (CLI) client docker.

You can learn how to install Docker Engine on PC.

First, update your existing list of packages:
```
sudo apt-get update
sudo apt-get install docker-ce docker-ce-cli containerd.io
```

Next, install a few prerequisite packages which let apt use packages over HTTPS:

```
sudo apt install apt-transport-https ca-certificates curl software-properties-common
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
```

Add the Docker repository to APT sources:
```
sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu focal stable"
```

Make sure you are about to install from the Docker repo instead of the default Ubuntu repo
```
apt-cache policy docker-ce
```

Finally, install Docker:

```
sudo apt install docker-ce
sudo systemctl start docker
sudo systemctl enable docker
```

Verify the installation:

```
docker --version
```

## Executing the Docker Command Without Sudo

If you want to avoid typing sudo whenever you run the docker command, add your username to the docker group:

```
sudo usermod -aG docker ${USER}
```
#### Before we proceed further, let us understand why we even need to move from VM to Docker.

As you have already learned – unlike a VM, Docker allocated not the whole guest OS for your application, but only isolated minimal part of it – this isolated container has all that your application needs and at the same time is lighter, faster, and can be shipped as a Docker image to multiple physical or virtual environments, as long as this environment can run Docker engine. This approach also solves the environment incompatibility issue. It is a well-known problem when a developer sends his application to you, you try to deploy it, deployment fails, and the developer replies, "- It works on my machine!". With Docker – if the application is shipped as a container, it has its own environment isolated from the rest of the world, and it will always work the same way on any server that has Docker engine. IT WORKS ON MY MACHINE

Now, when we understand the benefits we can get by using Docker containerization, let us learn what needs to be done to migrate to Docker. Read this excellent article for some insight.

## MySQL in container

Let us start assembling our application from the Database layer - we will use a pre-built MySQL database container, configure it, and make sure it is ready to receive requests from our PHP application.

#### Step 1: Pull MySQL Docker Image from Docker Hub Registry

```
docker pull mysql/mysql-server:latest
```
![image](https://github.com/user-attachments/assets/5210117a-1275-4dd3-9309-58f534ca0c83)

List the images to check that you have downloaded them successfully:

```
docker images ls
```
### Step 2: Deploy the MySQL Container to your Docker Engine.

1. Once you have the image, move on to deploying a new MySQL container with:

```
docker run --name <container_name> -e MYSQL_ROOT_PASSWORD=<my-secret-pw> -d mysql/mysql-server:latest
```

![image](https://github.com/user-attachments/assets/d5ab58e5-83ff-4715-b300-c772e28c542f)

2. Then, check to see if the MySQL container is running: Assuming the container name specified is mysql-server

```
docker ps -a

```
![image](https://github.com/user-attachments/assets/0b6397d5-4c26-4460-b380-189c3847abc1)

You should see the newly created container listed in the output. It includes container details, one being the status of this virtual environment. The status changes from health: starting to healthy, once the setup is complete.

Step 3: Connecting to the MySQL Docker Container

We can either connect directly to the container running the MySQL server or use a second container as a MySQL client. Let us see what the first option looks like.

## Approach 1

Connecting directly to the container running the MySQL server:

```
docker exec -it <container_name> mysql -u root -p
```

![image](https://github.com/user-attachments/assets/c8f02228-120f-40be-901c-1e14bfe408ec)

Now Stop the running container

## Approach 2

At this stage you are now able to create a docker container but we will need to add a network. So, stop and remove the previous mysql docker container.

```
docker ps -a
docker stop mysql-server
docker rm mysql-server or <container ID> ea751c5996d1
```
![image](https://github.com/user-attachments/assets/427236dc-e83a-4851-b0f3-fbd5db59ad5a)

verify that the container is deleted

```
docker ps -a
```

Next, create a network:

```
docker network create --subnet=172.18.0.0/24 tooling_app_network
```
Creating a custom network is not necessary because even if we do not create a network, Docker will use the default network for all the containers you run. By default, the network we created above is of DRIVER Bridge. So, also, it is the default network. You can verify this by running the docker network ls command.

```
docker network ls
```
![image](https://github.com/user-attachments/assets/d86d1831-99e9-494f-b900-967003568e4c)

to see the details:
```
sudo docker network  inspect  tooling_app_network
```
But there are use cases where this is necessary. For example, if there is a requirement to control the cidr range of the containers running the entire application stack. This will be an ideal situation to create a network and specify the --subnet.

For clarity's sake, we will create a network with a subnet dedicated for our project and use it for both MySQL and the application so that they can connect.

### Run the MySQL Server container using the created network.

First, let us create an environment variable to store the root password:

```
export MYSQL_PW=PassWord.1
```

verify the environment variable is created

```
echo $MYSQL_PW
```

Then, pull the image and run the container, all in one command like below

```
docker run --network tooling_app_network -h mysqlserverhost --name=mysql-server -e MYSQL_ROOT_PASSWORD=$MYSQL_PW  -d mysql/mysql-server:latest

```

![image](https://github.com/user-attachments/assets/bdf8a0f5-68a8-4e73-9f24-d40eb9facf6a)

#### Flags used

-d runs the container in detached mode
--network connects a container to a network
-h specifies a hostname

If the image is not found locally, it will be downloaded from the registry.

Verify the container is running:

```
docker ps -a
```
![image](https://github.com/user-attachments/assets/95ae5709-4488-4a1b-8b13-68473874587a)

It is best practice not to connect to the MySQL server remotely using the root user. Therefore, we will create an SQL script that will create a user we can use to connect remotely.

```
CREATE USER 'shoeb'@'%' IDENTIFIED BY 'PassWord.1';
GRANT ALL PRIVILEGES ON * . * TO 'shoeb'@'%';
```
![image](https://github.com/user-attachments/assets/909fad14-f85e-44c7-9d5f-77ceadd6302d)

Run the script to install create user:

```
docker exec -i mysql-server mysql -u root -p $MYSQL_PW < ./create_user.sql
```


### Connecting to the MySQL server from a second container running the MySQL client utility.

The good thing about this approach is that you do not have to install any client tool on your laptop, and you do not need to connect directly to the container running the MySQL server.


## Connecting to the MySQL server from a second container running the MySQL client utility.

The good thing about this approach is that you do not have to install any client tool on your laptop, and you do not need to connect directly to the container running the ```MySQL server.```


```
docker run --network tooling_app_network --name mysql-client -it --rm mysql mysql -h mysqlserverhost -u <user-created-from-the-SQL-script> -p
```

![image](https://github.com/user-attachments/assets/576c1803-b742-4d3b-aa8f-565c3804ff69)

Flags used:

* --name gives the container a name
*  -it runs in interactive mode and Allocate a pseudo-TTY
* --rm automatically removes the container when it exits
* --network connects a container to a network
* -h a MySQL flag specifying the MySQL server Container hostname
* -u user created from the SQL script
* -p password specified for the user created from the SQL script

