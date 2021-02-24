### PID namespace

Docker uses `PID namespace` to isolate the processes of the contianer. In the Linux host system we have
a handful processes when it boots up starting from PID:1. When we create a container, it create
a separate child process system where the PID again start from 1, but since on the same system you cannot have
the same PID, the PID inside the container is mapped to a different PID on the host system. For example, the PIDs 1 and 2
of the container are mapped to the PIDs 5 and 6 on the host machine yealding the illusion of having a separate isolated process system.
So, we leared that the underlying docker host as well as the docker container share the same system resources (CPU, Memory).

![image](https://user-images.githubusercontent.com/26717179/69493446-0db12400-0eaf-11ea-9759-d464ab864ee7.png)


How much resources are allocated to the host and containers, and how the docker manages the resources dedicated to the containers?
By default, there is no restriction on how much resources the containers can use. But, there is a way to set a limit. Docker uses
`cgroups` to restrict the amount of hardware resources allocated to each container. This can be done as follows:

# This ensures that the container doesn't use more than 50% of the CPUs at any given time
docker run --cpus=.5 ubuntu
# This limits the amount of memory the container can use at any given time to 100MB
docker run --memory=100m ubuntu


### How docker stores date on the local filesystem?

When we install docker on the system, it creates the following structure at `/var/lib/docker`, which contains multiple folders
where docker stores its data (files related to the images) by default.

/var/lib/docker
----aufs
----containers
----image
----volumes

So, how exactly does docker store files of an image and a container? For this, we need to understand docker's layered architecture.
Each line of instruction in the `Dockerfile` creates a new layer in the image with just the changes from the previous layer.
So, the advantage of using this layered architecture in docker is that if we create multiple docker container in which some of the commands
are the same then docker will create their layers once and reuse them from the cache in bulding the other containers that use the
same command:
![image](https://user-images.githubusercontent.com/26717179/69493432-e0647600-0eae-11ea-9d84-a2096667aff3.png)

This way docker builds the application contianers faster and saves the disk space. This is also useful when we want to upgrade the
application source code. In this case again, docker will buld only this layer while using the other layers from the cache.

Once the container is created, it is `read only` and you cannot modify its layers. In order to do any changes, you need
to rebuld the container. When we run `docker run` command speifying a container, docker creates a new `writable container layer` on
top of this `read only` image layers. This writable layer is used to store data created by the contianer (logs, etc). The life of this
writable layer though, is only as long as the container is alive.

![image](https://user-images.githubusercontent.com/26717179/69493560-633a0080-0eb0-11ea-87c1-be7df822e57a.png)

It is, however, possible to modify the files in the image layer by copying it in the read write layer, as docker does.
So, in fact, we will modify the copy of the file in the read only layer not the original.

When we kill the container, its all data store in the `writable layer` will de deleted as well. So, how to persist the data in the
container? We shall first create a volume:

# this command will create a directory under /var/lib/docker/volumes on the host machine
docker volume create data_volume
# this will mount data_volume folder on the host machine to /var/lib/mysql folder on the contianer
docker run -v data_volume:/var/lib/mysql mysql

What if we don't run the `volume create` command before `docker run` command? Docker will still authomatically create the
volume named `data_volume` and mount it to the container. What is we would like to store data on another volume which is not
under `/var/lib/docker/volumes` folder? In this case, we would have to provide the complete path where we would like to mount the volume

![image](https://user-images.githubusercontent.com/26717179/69493833-2bcd5300-0eb4-11ea-96e4-e9f894579470.png)


# This is called bind mounting. The other type is volume mounting, which mounts on the volumes directory
docker run -v /data/mysql:/var/lib/mysql mysql
# the same goal can be obtained by this new-styled command
docker run --mount=type=bind,source=/data/mysql, target=/var/lib/mysql mysql


Docker uses a `storage driver` such as AUFS, ZFB, Device Mapper, Overlay, to enable layered architecture.
The storage driver depends on the underlying OS used. For example, the default storage driver for ubuntu is AUFS.

![image](https://user-images.githubusercontent.com/63472885/89022646-689af500-d333-11ea-98d2-d6653afdf886.png)
![image](https://user-images.githubusercontent.com/63472885/89022863-bc0d4300-d333-11ea-8a98-506cd8f38784.png)
![image](https://user-images.githubusercontent.com/63472885/89023180-38078b00-d334-11ea-8329-e4f209c33e60.png)
![image](https://user-images.githubusercontent.com/63472885/89023423-9cc2e580-d334-11ea-9585-8ecd77e5a6a1.png)
![image](https://user-images.githubusercontent.com/63472885/90724847-44965800-e2d0-11ea-809d-fd13011a5a29.png)
![image](https://user-images.githubusercontent.com/63472885/90745362-47e60f80-e2e1-11ea-95a8-fc4e69f7e279.png)
![image](https://user-images.githubusercontent.com/63472885/90748423-3b62b680-e2e3-11ea-9a35-bf77d05e9078.png)
![image](https://user-images.githubusercontent.com/63472885/90748636-7d8bf800-e2e3-11ea-9442-8f34ac6a7451.png)
![image](https://user-images.githubusercontent.com/63472885/90750518-f724e580-e2e5-11ea-82f0-755dbd54b045.png)
![image](https://user-images.githubusercontent.com/63472885/90750747-4cf98d80-e2e6-11ea-8c06-d2a7b2f02f04.png)
![image](https://user-images.githubusercontent.com/63472885/90750986-99dd6400-e2e6-11ea-9c0a-c49108e91cb4.png)
![image](https://user-images.githubusercontent.com/63472885/90751373-1c662380-e2e7-11ea-9d7f-8ef5e982ae1a.png)
![image](https://user-images.githubusercontent.com/63472885/90751684-7e268d80-e2e7-11ea-9194-cded5639d0c6.png)
![image](https://user-images.githubusercontent.com/63472885/90752279-3eac7100-e2e8-11ea-8338-e25b4f4d1b85.png)
![image](https://user-images.githubusercontent.com/63472885/90752525-8f23ce80-e2e8-11ea-9098-4e1fc951a639.png)
![image](https://user-images.githubusercontent.com/63472885/90758420-0f016700-e2f0-11ea-8b83-49457ac79625.png)
![image](https://user-images.githubusercontent.com/63472885/90759214-30af1e00-e2f1-11ea-829d-e96908de6897.png)
![image](https://user-images.githubusercontent.com/63472885/90759943-348f7000-e2f2-11ea-8c43-e04ee37d4168.png)
```
docs.docker.com (click get docker)
cat /etc/*release* (to see version os)
1.sudo apt-get remove docker docker-engine docker.io containerd runc
2.curl -fsSL https://get.docker.com -o get-docker.sh
3.sudo sh get-docker.sh
4.sudo docker version
hub.docker.com
search whalesay
======================================
      Docker commands
docker ps
docker ps -a
docker exec distracted_mcclintock cat /etc/hosts (distracted_mcclintock this name of contayner)
docker exec cla19d3a7ca7 /cat /etc/*release*
docker run -d kodekloud/simple-webapp
docker attach a043d
docker run -it centos bash
cat /etc/*release*
docker run -d centos sleep 20
docker images 
docker rmi ubuntu
docker pull ubuntu
docker run --name webapp nginx:1.14-alpine
docker run ubuntu:17.10 cat /etc/*release*
docker run timer
docker run -d timer
docker run -p 38282:8080 kodekloud/simple-webapp:blue
mkdir my-jenkins-data
docker run -p 8080:8080 -v /root/my-jenkins-data:/var/jenkins_home -u root jenkins
docker inspect (name conainer)
docker run -p 38282:8080 --name blue-app -e APP_COLOR=blue -d kodekloud/simple-webapp
docker run -d -e MYSQL_ROOT_PASSWORD=db_pass123 --name mysql-db mysql
docker build -t webapp-color .
docker run -p 8282:8080 webapp-color
docker run python:3.6 cat /etc/*release*
docker run -p 8383:8080 webapp-color:lite
docker run --name db -e POSTGRES_PASSWORD=mysecretpassword -d postgres
docker run -d --name=wordpress --link db:db -p 8085:80 wordpress
docker run -d --name mysql-db -e MYSQL_ROOT_PASSWORD=db_pass123 mysql
docker run -v /opt/data:/var/lib/mysql -d --name mysql-db -e MYSQL_ROOT_PASSWORD=db_pass123 mysql
docker network create --driver bridge --subnet 182.18.0.0/16 custom-isolated-network
docker network ls
docker network inspect bridge
docker run --name alpine-2 --network=none alpine
docker network create --driver bridge --subnet 182.18.0.1/24 --gateway 182.18.0.1 wp-mysql-network
sudo docker network create --driver=bridge --subnet="10.1.0.0/16" net2
sudo docker run --network=net2 --name=container2 -d nginx:latest                                                                             inx:latest
sudo docker network create --driver=bridge --subnet="10.0.0.0/16" net1
sudo docker run --network=net1 --name=container1 -d nginx:latest
sudo docker exec -it container1 /bin/bash
sudo docker container ls
sudo docker container inspect [id of image] | grep IP
sudo docker container run -d nginx
elinks 172.17.0.2:80
sudo docker container run -d -P nginx
sudo docker container run -d -p 80:80 httpd
sudo docker volume ls
sudo docker volume create devvolume
sudo docker volume inspect devvolume
sudo docker container run -d --name devcont --mount source=devvolume,target=/app nginx
sudo ls /var/lib/docker/volumes/devvolume/_data
sudo docker container run -d --name devcont2 -v devvolume:/app nginx


docker run -d -e MYSQL_ROOT_PASSWORD=db_pass123 --name mysql-db --network wp-mysql-network mysql:5.6
docker run --network=wp-mysql-network -e DB_Host=mysql-db -e DB_Password=db_pass123 -p 38080:8080 --name webapp --link mysql-db:mysql-db -d kodekloud/simple-webapp-mysql
docker volume ls
docker volume inspect myvolume
docker volume rm myvolume
docker pull jenkins
sudo docker run --name MyJenkins1 -v myvolume:/var/jenkins_home -p 8080:8080 -p 50000:50000 jenkins (on the phisical host its controling by docker filesystem meneger, volume mounte )
sudo docker run -d --name MyJenkins4 -v /home/ubuntu/roman:/var/jenkins_home -p 8282:8080 -p 55000:56000 jenkins (on the phisical host its controling by phisical filesystem meneger, called bined mount )

===========================================
docker run -d -p 5000:5000 --name registry registry:2
docker image tag my-image localhost:5000/my-image
docker push localhost:5000/my-image
docker pull localhost:/my-image
docker pull 192.168.56.100:5000/my-image

Dockerfiles example                     =
=====================                   =
mkdir example                           =
cd example                              =
   nano Dockerfile                      =
FROM busybox                            =
RUN echo "building simple docker image" =
CMD echo "hello container"              =
=========================               =
docker build -t hello .                 =
docker run --rm hello                   =
=========================================
    nano Dockerfile                     =
FROM debian:sid                         =
RUN apt-get -y update                   =
RUN apt-get install nano                =
CMD ["/bin/nano", "/tmp/notes"]         =
=========================================
docker build -t example/nanoer          =
docker run --rm -ti example/nanoer      =
=========================================
     nano Dockerfile                    =
FROM example/nanoer                     =
ADD notes.txt /notes.txt                =
CMD ["/bin/nano", "/notes.txt"]         =
=========================================
nano notes.txt                          =
docker build -t examle/notes .          =
docker run -ti --rm example/notes       =============
=========================================           =
        Multi-project Docker files                  =
            nano Dockerfile                         =
FROM ubuntu:16.04                                   =
RUN apt-get update                                  =
RUN apt-get -y install curl                         =
RUN curl https://google.com | wc -c > google-size   =
ENTRYPOINT echo google is this big; cat google-size =
=====================================================
docker build -t tooo-big .                          =
docker run tooo-big                                 =
=================================================== =
           nano Dockerfile                          =
FROM ubuntu:16.04 as builder                        =
RUN apt-get update                                  =
RUN apt-get -y install curl                         =
RUN curl https://google.com | wc -c > google-size   =
                                                    =
FROM alpine                                         =
COPY --from=builder /google-size /google-size       =
ENTRYPOINT echo google is this big; cat google-size =
=====================================================
docker build -t google-size .                       =
docker run google-size                              =
=====================================================

    wordpress with mysql
mkdir wordpress
cd wordpress
nano docker-compose.yaml
===============================
version: "3"
services:
  db_host:
    container_name: db
    image: mysql:5.7
    volumes:
       - db_data:/var/lib/mysql
    restart: always
    environment:
       MYSQL_ROOT_PASSWORD: 1234
       MYSQL_DATABASE: wordpress
       MYSQL_USER: wordpress
       MYSQL_PASSWORD: wordpress
    networks:
      - my-custom-bridge
  wordpress:
    depends_on:
      - db_host
    image: wordpress:latest
    ports:
      - "8000:80"
    restart: always
    environment:
      WORDPRESS_DB_HOST: db_host:3306
      WORDPRESS_DB_USER: wordpress
      WORDPRESS_DB_PASSWORD: wordpress
      WORDPRESS_DB_NAME: wordpress
    networks:
      - my-custom-bridge
volumes:
    db_data: {}
networks:
  my-custom-bridge:
    driver: bridge
=================================
docker-compose up -d
==================================

    
    




```
![image](https://user-images.githubusercontent.com/63472885/91305041-7e2cfe80-e7bb-11ea-90d1-1ae6d20ca1b3.png)
![image](https://user-images.githubusercontent.com/63472885/91305539-3ce91e80-e7bc-11ea-9ed1-e4dc85714494.png)
![image](https://user-images.githubusercontent.com/63472885/91305714-7e79c980-e7bc-11ea-809f-73e59c7cbb29.png)
![image](https://user-images.githubusercontent.com/63472885/91306498-a3227100-e7bd-11ea-9d7e-f3bd76f67bbe.png)
![image](https://user-images.githubusercontent.com/63472885/91306897-46738600-e7be-11ea-8edb-16453cf90aeb.png)
![image](https://user-images.githubusercontent.com/63472885/91310133-88063000-e7c2-11ea-95ba-1edfc2666f14.png)
![image](https://user-images.githubusercontent.com/63472885/91310329-ca2f7180-e7c2-11ea-8943-eba3ad393245.png)
![image](https://user-images.githubusercontent.com/63472885/91489020-342e4080-e8c1-11ea-8fae-e3e1c9fedd8c.png)
![image](https://user-images.githubusercontent.com/63472885/91489824-7dcb5b00-e8c2-11ea-938c-7bcd6dabe36b.png)
![image](https://user-images.githubusercontent.com/63472885/91490337-56c15900-e8c3-11ea-8f10-adc2b1a9d0ee.png)
![image](https://user-images.githubusercontent.com/63472885/91490830-23cb9500-e8c4-11ea-9e17-2d2c609840d3.png)
![image](https://user-images.githubusercontent.com/63472885/91491276-db60a700-e8c4-11ea-8aca-11d4b4277afb.png)
![image](https://user-images.githubusercontent.com/63472885/91491545-44481f00-e8c5-11ea-8d6e-dec1ddafb880.png)
![image](https://user-images.githubusercontent.com/63472885/91491982-ed8f1500-e8c5-11ea-8475-30a0098620d7.png)
![image](https://user-images.githubusercontent.com/63472885/91652730-00167380-eaab-11ea-81a1-c9ed50bbb9a9.png)
![image](https://user-images.githubusercontent.com/63472885/91652804-9185e580-eaab-11ea-9c9f-5d33d4a5ea11.png)
![image](https://user-images.githubusercontent.com/63472885/91652926-b6c72380-eaac-11ea-8d17-8f4539954bbe.png)
![image](https://user-images.githubusercontent.com/63472885/91652983-47056880-eaad-11ea-8614-e65082919dc5.png)
![image](https://user-images.githubusercontent.com/63472885/91653028-99df2000-eaad-11ea-99b2-5f04608ceb96.png)
![image](https://user-images.githubusercontent.com/63472885/91653059-fa6e5d00-eaad-11ea-9d9e-8f5ff1f82c38.png)
![image](https://user-images.githubusercontent.com/63472885/91653441-fd6b4c80-eab1-11ea-9d4e-c7e2fd010dd8.png)

```
Management command were introduced in Docker engine v1.13
Management Commands:

builder Manage builds
config Manage Docker configs
container Manage containers
engine Manage the docker engine
image Manage images
network Manage networks
node Manage Swarm nodes
plugin Manage plugins
secret Manage Docker secrets
service Manage services
stack Manage Docker stacks
swarm Manage Swarm
system Manage Docker
trust Manage trust on Docker images
volume Manage volumes
docker image:

build Build an image from a dockerfile
history Show the history of an image
import Import the contents from a tarball to create a filesystem image
inspect Display detailed information on one or more images
load Load an image from a tar file or STDIN
ls List images
prune Remove unused images
pull Pull an image or a repository from a registry
push Push an image or a repository to a registry
rm Remove one or more images
save Save one or more images to a tar file (streamed to STDOUT by default)
tag Create a tag TARGET_IMAGE that refers to SOURCE_IMAGE
docker container:

attach Attach local standard input, output, and error streams to a running container
commit Create a new image from a container's changes
cp Copy files/folders between a container and the local filesystem
create Create a new container
diff Inspect changes to files or directories on a container's filesystem
exec Run a command in a running container
export Export a container's filesystem as a tar archive
inspect Display detailed information on one or more containers
kill Kill one or more running containers
logs Fetch the logs of a container
ls List containers
pause Pause all processes within one or more containers
port List port mappings or a specific mapping for the container
prune Remove all stopped containers
rename Rename a container
restart Restart one or more containers
rm Remove one or more containers
run Run a command in a new container
start Start one or more stopped containers
stats Display a live stream of container(s) resource usage statistics
stop Stop one or more running containers
top Display the running processes of a container
unpause Unpause all processes within one or more containers
update Update configuration of one or more containers
wait Block until one or more containers stop, then print their exit codes
```
![image](https://user-images.githubusercontent.com/63472885/107846917-0e5e0f00-6e01-11eb-8b0f-b55c83723d76.png)
![image](https://user-images.githubusercontent.com/63472885/107847103-313cf300-6e02-11eb-9f4c-b7f03077dd5b.png)
![image](https://user-images.githubusercontent.com/63472885/107855877-8d256d00-6e3e-11eb-850f-beb83bd550ce.png)
![image](https://user-images.githubusercontent.com/63472885/107855904-b1814980-6e3e-11eb-870f-6bdb567aa800.png)
![image](https://user-images.githubusercontent.com/63472885/107856182-f3f75600-6e3f-11eb-9fc2-8f034d7db5a5.png)
![image](https://user-images.githubusercontent.com/63472885/107856197-0d000700-6e40-11eb-9bd0-23be391d6b45.png)
![image](https://user-images.githubusercontent.com/63472885/107856217-2c972f80-6e40-11eb-8870-eed4f60c8bc7.png)
![image](https://user-images.githubusercontent.com/63472885/107856243-4f294880-6e40-11eb-8e83-d342fa3b92c4.png)

```
docker network create frontend
docker network create localhost --internal
docker container run -d --name database --network localhost -e MYSQL_ROOT_PASSWORD=P4ssW0rd0! mysql:5.7
docker container run -d --name frontend-app --network frontend nginx:latest
docker network connect localhost frontend-app
```
![image](https://user-images.githubusercontent.com/63472885/107874534-bba15780-6ed3-11eb-86aa-3b1ad12e4d27.png)
![image](https://user-images.githubusercontent.com/63472885/107874554-d96ebc80-6ed3-11eb-909d-e5161c322183.png)
![image](https://user-images.githubusercontent.com/63472885/107874579-fefbc600-6ed3-11eb-800d-48985e73cccb.png)
![image](https://user-images.githubusercontent.com/63472885/107874601-18047700-6ed4-11eb-8fdd-ed7d7756987f.png)
![image](https://user-images.githubusercontent.com/63472885/107874697-b2fd5100-6ed4-11eb-81dd-fc5efa9b89a4.png)
![image](https://user-images.githubusercontent.com/63472885/107874708-cf00f280-6ed4-11eb-92c0-b2eea4faf0af.png)
Create an nginx.conf file:
```
mkdir nginx
cat << EOF >  nginx/nginx.conf
user  nginx;
worker_processes  1;

error_log  /var/log/nginx/error.log warn;
pid        /var/run/nginx.pid;


events {
    worker_connections  1024;
}


http {
    include       /etc/nginx/mime.types;
    default_type  application/octet-stream;

    log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                      '$status $body_bytes_sent "$http_referer" '
                      '"$http_user_agent" "$http_x_forwarded_for"';

    access_log  /var/log/nginx/access.log  main;

    sendfile        on;
    #tcp_nopush     on;

    keepalive_timeout  65;

    #gzip  on;

    include /etc/nginx/conf.d/*.conf;
}
EOF
```
![image](https://user-images.githubusercontent.com/63472885/107874738-14252480-6ed5-11eb-97f0-fab07db01844.png)
![image](https://user-images.githubusercontent.com/63472885/107874937-82b6b200-6ed6-11eb-8eaa-c376942e72e6.png)
![image](https://user-images.githubusercontent.com/63472885/107874965-a2e67100-6ed6-11eb-91ad-fe8420c90150.png)
![image](https://user-images.githubusercontent.com/63472885/107874995-bc87b880-6ed6-11eb-8c4c-a6841f136eb3.png)
![image](https://user-images.githubusercontent.com/63472885/107875294-9400be00-6ed8-11eb-980c-4305526d68f4.png)
```
[cloud_user@host]$ docker volume create mysql_data

[cloud_user@host]$ docker container run -d --name app-database \
 --mount type=volume,source=mysql_data,target=/var/lib/mysql \
 -e MYSQL_ROOT_PASSWORD=P4ssW0rd0! \
 mysql:latest
```
![image](https://user-images.githubusercontent.com/63472885/107875831-839e1280-6edb-11eb-894c-a0b4ce45b0df.png)
![image](https://user-images.githubusercontent.com/63472885/107875845-9add0000-6edb-11eb-932b-d377de9b9bb2.png)
![image](https://user-images.githubusercontent.com/63472885/107877737-a6362880-6ee7-11eb-9681-76798658fccd.png)
![image](https://user-images.githubusercontent.com/63472885/107877770-d382d680-6ee7-11eb-913a-e034dcf259cd.png)
![image](https://user-images.githubusercontent.com/63472885/107877796-f1e8d200-6ee7-11eb-84ba-579e91b55cb2.png)
![image](https://user-images.githubusercontent.com/63472885/107881935-151f7b80-6f00-11eb-957d-4f249ef1f4c5.png)
![image](https://user-images.githubusercontent.com/63472885/107881961-36806780-6f00-11eb-8ead-ee63156e8a29.png)
![image](https://user-images.githubusercontent.com/63472885/107881982-50ba4580-6f00-11eb-93e0-9354ad99b5cb.png)
![image](https://user-images.githubusercontent.com/63472885/107882186-5c5a3c00-6f01-11eb-88ca-19fc1310786b.png)
![image](https://user-images.githubusercontent.com/63472885/107882211-74ca5680-6f01-11eb-975e-9bd3ab4427dc.png)
![image](https://user-images.githubusercontent.com/63472885/107882222-8c094400-6f01-11eb-8daa-23cdf5909c99.png)
![image](https://user-images.githubusercontent.com/63472885/107882264-bfe46980-6f01-11eb-9309-e8de36a00d9f.png)
![image](https://user-images.githubusercontent.com/63472885/107882289-e0142880-6f01-11eb-8be2-420ee1b8ac43.png)
![image](https://user-images.githubusercontent.com/63472885/107882318-033ed800-6f02-11eb-9ed5-b4607e36a796.png)
![image](https://user-images.githubusercontent.com/63472885/107882467-01c1df80-6f03-11eb-9730-dfe5435c4cf8.png)
![image](https://user-images.githubusercontent.com/63472885/107882489-18683680-6f03-11eb-8f42-97539fc44918.png)
![image](https://user-images.githubusercontent.com/63472885/107885798-b95fed00-6f15-11eb-9e9e-aae37f577262.png)
![image](https://user-images.githubusercontent.com/63472885/107885846-d98fac00-6f15-11eb-9041-35f66329b322.png)
![image](https://user-images.githubusercontent.com/63472885/107885864-0643c380-6f16-11eb-902c-b19d772ca62d.png)
![image](https://user-images.githubusercontent.com/63472885/107885880-1c518400-6f16-11eb-959c-50de6ba76c0d.png)
![image](https://user-images.githubusercontent.com/63472885/107885894-34290800-6f16-11eb-942e-d41ee56bb7ef.png)
![image](https://user-images.githubusercontent.com/63472885/107886181-b1a14800-6f17-11eb-9e32-2973e1a90db7.png)
![image](https://user-images.githubusercontent.com/63472885/107886203-cc73bc80-6f17-11eb-9e82-4b60732e2aab.png)
![image](https://user-images.githubusercontent.com/63472885/107921888-a5a19e80-6f88-11eb-90b3-ff51312bc3bf.png)
![image](https://user-images.githubusercontent.com/63472885/107921968-c23dd680-6f88-11eb-9c8b-9c4cb82c8577.png)
![image](https://user-images.githubusercontent.com/63472885/107922044-e0a3d200-6f88-11eb-9583-1946dac96101.png)
![image](https://user-images.githubusercontent.com/63472885/107922889-17c6b300-6f8a-11eb-9ee6-a9207c378af3.png)
![image](https://user-images.githubusercontent.com/63472885/107922957-32009100-6f8a-11eb-9666-67bbe3c21e22.png)
![image](https://user-images.githubusercontent.com/63472885/107923540-147ff700-6f8b-11eb-90c0-46c11a154132.png)
![image](https://user-images.githubusercontent.com/63472885/107923622-2c577b00-6f8b-11eb-8042-4d485f82feb1.png)
![image](https://user-images.githubusercontent.com/63472885/107946625-b6630c00-6faa-11eb-9c71-39c15c919ed6.png)
![image](https://user-images.githubusercontent.com/63472885/107946705-d5619e00-6faa-11eb-8f5e-88d7a6dc554e.png)
![image](https://user-images.githubusercontent.com/63472885/107946777-ef02e580-6faa-11eb-9c1c-6407bffec107.png)
![image](https://user-images.githubusercontent.com/63472885/107951818-5a03ea80-6fb2-11eb-8b89-4b256750c3f2.png)
![image](https://user-images.githubusercontent.com/63472885/107951934-8455a800-6fb2-11eb-8d91-31874bd1cf6c.png)
![image](https://user-images.githubusercontent.com/63472885/107962869-ea492c00-6fc0-11eb-8b25-a03acee120a6.png)
![image](https://user-images.githubusercontent.com/63472885/107962983-08169100-6fc1-11eb-8e9a-748c679f7799.png)
![image](https://user-images.githubusercontent.com/63472885/107963636-dd790800-6fc1-11eb-8139-313ecd784250.png)
![image](https://user-images.githubusercontent.com/63472885/107963735-faadd680-6fc1-11eb-9ff2-ac6b612eee36.png)
![image](https://user-images.githubusercontent.com/63472885/107964571-ddc5d300-6fc2-11eb-910d-08a84a85fe19.png)
![image](https://user-images.githubusercontent.com/63472885/107964765-0f3e9e80-6fc3-11eb-92dc-4d910ecccc75.png)
![image](https://user-images.githubusercontent.com/63472885/107964826-28474f80-6fc3-11eb-8ada-1d925bec0238.png)
![image](https://user-images.githubusercontent.com/63472885/107965427-f4b8f500-6fc3-11eb-86f2-7a2fc847bbc3.png)
![image](https://user-images.githubusercontent.com/63472885/107965541-187c3b00-6fc4-11eb-8fee-0e7d7861835b.png)
![image](https://user-images.githubusercontent.com/63472885/107965665-3b0e5400-6fc4-11eb-8b2d-01fefb7f5e53.png)
![image](https://user-images.githubusercontent.com/63472885/108030279-6cc8fe80-7048-11eb-9fde-c8b3715b7d50.png)
![image](https://user-images.githubusercontent.com/63472885/108030360-8bc79080-7048-11eb-94d6-8c57aac5da2d.png)
![image](https://user-images.githubusercontent.com/63472885/108031806-e9f57300-704a-11eb-8559-90899b07bb6b.png)
![image](https://user-images.githubusercontent.com/63472885/108052593-cc81d280-7065-11eb-9d98-5db39c5cc043.png)
![image](https://user-images.githubusercontent.com/63472885/108032551-1958af80-704c-11eb-907e-350be8516804.png)
![image](https://user-images.githubusercontent.com/63472885/108032633-38574180-704c-11eb-8243-88042a0ff015.png)
![image](https://user-images.githubusercontent.com/63472885/108032690-50c75c00-704c-11eb-9970-3a5b6136d274.png)
![image](https://user-images.githubusercontent.com/63472885/108056750-2042ea80-706b-11eb-89c4-41bd44e23093.png)
![image](https://user-images.githubusercontent.com/63472885/108056828-394b9b80-706b-11eb-9dbe-b8a020ade928.png)
![image](https://user-images.githubusercontent.com/63472885/108056911-584a2d80-706b-11eb-84e1-20afddae3316.png)
![image](https://user-images.githubusercontent.com/63472885/108057028-7c0d7380-706b-11eb-8b71-52cb0b7ce555.png)
![image](https://user-images.githubusercontent.com/63472885/108066074-4e7af700-7078-11eb-90e9-925944a5a73e.png)
![image](https://user-images.githubusercontent.com/63472885/108068749-ed552280-707b-11eb-98be-e83dff30e766.png)
![image](https://user-images.githubusercontent.com/63472885/108068839-0c53b480-707c-11eb-81e0-a9cc0f8c6d84.png)
![image](https://user-images.githubusercontent.com/63472885/108068898-255c6580-707c-11eb-9ff8-8af1c8e1a7c4.png)
![image](https://user-images.githubusercontent.com/63472885/108068972-3e651680-707c-11eb-984d-72ccb9b6877e.png)
![image](https://user-images.githubusercontent.com/63472885/108070065-83d61380-707d-11eb-93c8-25bfbb78b778.png)
![image](https://user-images.githubusercontent.com/63472885/108070156-9cdec480-707d-11eb-90df-b16bbbea314e.png)
![image](https://user-images.githubusercontent.com/63472885/108070312-ce579000-707d-11eb-904d-327162039787.png)
docker-compose.yml:
```
version: '3'
services:
  ghost:
    container_name: ghost
    image: ghost:latest
    ports:
      - "80:2368"
    environment:
      - database__client=mysql
      - database__connection__host=mysql
      - database__connection__user=root
      - database__connection__password=P4SSw0rd0!
      - database__connection__database=ghost
    volumes:
      - ghost-volume:/var/lib/ghost
    networks:
      - ghost_network
      - mysql_network
    depends_on:
      - mysql

  mysql:
    container_name: mysql
    image: mysql:5.7
    environment:
      - MYSQL_ROOT_PASSWORD=P4SSw0rd0!
    volumes:
      - mysql-volume:/var/lib/mysql
    networks:
      - mysql_network

volumes:
  ghost-volume:
  mysql-volume:

networks:
  ghost_network:
  mysql_network:
```
![image](https://user-images.githubusercontent.com/63472885/108070494-065ed300-707e-11eb-9846-76df0e9df46e.png)
![image](https://user-images.githubusercontent.com/63472885/108074703-f3023680-7082-11eb-9656-9285cf5ef409.png)
The contents of docker-compose.yml should be
```
version: '3'
services:
  weather-app1:
    build:
      context: ./weather-app
      args:
        - VERSION=v2.0
    ports:
      - "8080:3000"
    networks:
     - weather_app
    environment:
      - NODE_ENV=production
  weather-app2:
    build:
      context: ./weather-app
      args:
        - VERSION=v2.0
    ports:
      - "8081:3000"
    networks:
     - weather_app
    environment:
      - NODE_ENV=production
  weather-app3:
    build:
      context: ./weather-app
      args:
        - VERSION=v2.0
    ports:
      - "8082:3000"
    networks:
     - weather_app
    environment:
      - NODE_ENV=production
  nginx:
      build: ./nginx
      tty: true
      ports:
       - '80:80'
      networks:
       - frontend
       - weather_app

networks:
  frontend:
  weather_app:
    internal: true
```
![image](https://user-images.githubusercontent.com/63472885/108074941-33fa4b00-7083-11eb-9bea-d44d6e5b19e5.png)
The contents of nginx.conf should be
```
events { worker_connections 1024; }

http {
  upstream localhost {
    server weather-app1:3000;
    server weather-app2:3000;
    server weather-app3:3000;
  }
  server {
    listen 80;
    server_name localhost;
    location / {
      proxy_pass http://localhost;
      proxy_set_header Host $host;
    }
  }
}
```
![image](https://user-images.githubusercontent.com/63472885/108075079-5e4c0880-7083-11eb-97b5-a2927028b6e1.png)
![image](https://user-images.githubusercontent.com/63472885/108075602-fe099680-7083-11eb-9bc6-e41cd2ea158a.png)
![image](https://user-images.githubusercontent.com/63472885/108075702-1974a180-7084-11eb-9233-f93ef6fe7caf.png)
![image](https://user-images.githubusercontent.com/63472885/108076330-c0593d80-7084-11eb-9066-43ce3d2705a3.png)
Uninstall old versions:
```
sudo yum remove -y docker \
                  docker-client \
                  docker-client-latest \
                  docker-common \
                  docker-latest \
                  docker-latest-logrotate \
                  docker-logrotate \
                  docker-engine
```
Install Docker CE
Add the Docker repository:
```
sudo yum install -y yum-utils \
  device-mapper-persistent-data \
  lvm2
```
Set up the stable repository:
```
sudo yum-config-manager \
    --add-repo \
    https://download.docker.com/linux/centos/docker-ce.repo
```
Install Docker CE:
```
sudo yum -y install docker-ce
```
Enable and Start Docker:
```
sudo systemctl start docker && sudo systemctl enable docker
```
Add cloud_user to the docker group:
```
sudo usermod -aG docker cloud_user
```
Initialize the manager:
```
docker swarm init \
--advertise-addr [PRIVATE_IP]
```
Add the worker to the cluster:
```
docker swarm join --token [TOKEN] \
[PRIVATE_IP]:2377
```
List the nodes in the swarm:
```
docker node ls
```
![image](https://user-images.githubusercontent.com/63472885/108102108-8b0f1880-70a1-11eb-8804-3ebc856d9f65.png)
![image](https://user-images.githubusercontent.com/63472885/108102175-a4b06000-70a1-11eb-8c06-469837ac24b0.png)
![image](https://user-images.githubusercontent.com/63472885/108102253-be51a780-70a1-11eb-93d8-f46263ba57ea.png)
![image](https://user-images.githubusercontent.com/63472885/108109100-e5f93d80-70aa-11eb-90e6-2c7f67aae2b3.png)
![image](https://user-images.githubusercontent.com/63472885/108109222-05906600-70ab-11eb-9ecd-63f99fcedcab.png)
![image](https://user-images.githubusercontent.com/63472885/108109314-1ccf5380-70ab-11eb-8af9-9d4d061c2ab3.png)
![image](https://user-images.githubusercontent.com/63472885/108110141-476ddc00-70ac-11eb-99b6-439c86b7ad2b.png)
![image](https://user-images.githubusercontent.com/63472885/108110221-666c6e00-70ac-11eb-961e-a2a2389cd5d5.png)
![image](https://user-images.githubusercontent.com/63472885/108110297-7e43f200-70ac-11eb-895d-8238d53c8924.png)
![image](https://user-images.githubusercontent.com/63472885/108168933-82562b00-7111-11eb-8e5a-fd82abe28180.png)
![image](https://user-images.githubusercontent.com/63472885/108169051-a6197100-7111-11eb-9645-139dae0f754a.png)
![image](https://user-images.githubusercontent.com/63472885/108169121-c0534f00-7111-11eb-9c5a-42d1364dc63d.png)
![image](https://user-images.githubusercontent.com/63472885/108171770-73717780-7115-11eb-978a-0a13e4b95209.png)
prometheus.yml contents:
```
global:
  scrape_interval: 15s
  scrape_timeout: 15s
  evaluation_interval: 15s

scrape_configs:
  - job_name: prometheus
    scrape_interval: 5s
    static_configs:
    - targets:
      - prometheus_main:9090

  - job_name: nodes
    scrape_interval: 5s
    static_configs:
    - targets:
      - [MANAGER]:9100
      - [WORKER1]:9100
      - [WORKER2]:9100

  - job_name: cadvisor
    scrape_interval: 5s
    static_configs:
    - targets:
      - [MANAGER]:8081
      - [WORKER1]:8081
      - [WORKER2]:8081
```
![image](https://user-images.githubusercontent.com/63472885/108172706-8f294d80-7116-11eb-8519-cbd9814fe6aa.png)

```
version: '3'
services:
  main:
    image: prom/prometheus:latest
    container_name: prometheus
    ports:
      - 8080:9090
    command:
      - --config.file=/etc/prometheus/prometheus.yml
      - --storage.tsdb.path=/prometheus/data
    volumes:
    - ./prometheus.yml:/etc/prometheus/prometheus.yml:ro
    - data:/prometheus/data
    depends_on:
      - cadvisor
      - node-exporter
  cadvisor:
    image: google/cadvisor:latest
    container_name: cadvisor
    deploy:
      mode: global
    restart: unless-stopped
    ports:
      - 8081:8080
    volumes:
      - /:/rootfs:ro
      - /var/run:/var/run:rw
      - /sys:/sys:ro
      - /var/lib/docker/:/var/lib/docker:ro
  node-exporter:
    image: prom/node-exporter:latest
    container_name: node-exporter
    deploy:
      mode: global
    restart: unless-stopped
    ports:
      - 9100:9100
    volumes:
      - /proc:/host/proc:ro
      - /sys:/host/sys:ro
      - /:/rootfs:ro
    command:
      - '--path.procfs=/host/proc'
      - '--path.sysfs=/host/sys'
      - --collector.filesystem.ignored-mount-points
      - "^/(sys|proc|dev|host|etc|rootfs/var/lib/docker/containers|rootfs/var/lib/docker/overlay2|rootfs/run/docker/netns|rootfs/var/lib/docker/aufs)($$|/)"
  grafana:
    image: grafana/grafana
    container_name: grafana
    ports:
      - 8082:3000
    volumes:
    - grafana_data:/var/lib/grafana
    - grafana_plugins:/var/lib/grafana/plugins
    environment:
      - GF_SECURITY_ADMIN_PASSWORD=P4ssW0rd0!
    depends_on:
      - prometheus
      - cadvisor
      - node-exporter

volumes:
  data:
  grafana_data:
  grafana_plugins:
```
![image](https://user-images.githubusercontent.com/63472885/108172798-b5e78400-7116-11eb-8dc1-9cc09e99a4f0.png)

![image](https://user-images.githubusercontent.com/63472885/108174494-f1834d80-7118-11eb-9b41-c5c9f858f4c5.png)
![image](https://user-images.githubusercontent.com/63472885/108174591-0d86ef00-7119-11eb-84f9-473e58ac6fea.png)
![image](https://user-images.githubusercontent.com/63472885/108177854-2d201680-711d-11eb-8730-017535723cdf.png)
![image](https://user-images.githubusercontent.com/63472885/108177945-4b861200-711d-11eb-905c-b851876190c6.png)
```
version: '3'

services:
   db:
     image: mysql:5.7
     volumes:
       - db_data:/var/lib/mysql
     networks:
       mysql_internal:
     environment:
       MYSQL_ROOT_PASSWORD: P4ssw0rd0!
       MYSQL_DATABASE: wordpress
       MYSQL_USER: wordpress
       MYSQL_PASSWORD: P4ssw0rd0!

   blog:
     depends_on:
       - db
     image: wordpress:latest
     networks:
       mysql_internal:
       wordpress_public:
     ports:
       - "80:80"
     environment:
       WORDPRESS_DB_HOST: db:3306
       WORDPRESS_DB_USER: wordpress
       WORDPRESS_DB_PASSWORD: P4ssw0rd0!

volumes:
    db_data:
networks:
  mysql_internal:
    internal: true
  wordpress_public:
```
![image](https://user-images.githubusercontent.com/63472885/108178053-6fe1ee80-711d-11eb-8e44-4b148ca09c7d.png)
![image](https://user-images.githubusercontent.com/63472885/108181673-a6ba0380-7121-11eb-8f48-9a1e69b546fb.png)
![image](https://user-images.githubusercontent.com/63472885/108181785-c3563b80-7121-11eb-8c40-0e9ef7f2d0e1.png)
![image](https://user-images.githubusercontent.com/63472885/108181876-dc5eec80-7121-11eb-8a2f-48389f6a7b75.png)
![image](https://user-images.githubusercontent.com/63472885/108181980-fac4e800-7121-11eb-8459-fae801bd6aaa.png)
```
docker container run --rm -it --network host --pid host --userns host --cap-add audit_control \
    -e DOCKER_CONTENT_TRUST=$DOCKER_CONTENT_TRUST \
    -v /var/lib:/var/lib \
    -v /var/run/docker.sock:/var/run/docker.sock \
    -v /usr/lib/systemd:/usr/lib/systemd \
    -v /etc:/etc --label docker_bench_security \
    docker/docker-bench-security
```
![image](https://user-images.githubusercontent.com/63472885/108185680-ff8b9b00-7125-11eb-8754-9b1c4fd0cc6c.png)
![image](https://user-images.githubusercontent.com/63472885/108185800-1fbb5a00-7126-11eb-916b-172be9fd7b2b.png)
![image](https://user-images.githubusercontent.com/63472885/108185963-4d080800-7126-11eb-8d29-e2e873e7695e.png)
![image](https://user-images.githubusercontent.com/63472885/108192794-2352df00-712e-11eb-9b64-d10a4f8652da.png)
![image](https://user-images.githubusercontent.com/63472885/108192900-42517100-712e-11eb-8e70-3bf85627756a.png)
![image](https://user-images.githubusercontent.com/63472885/108192992-5f863f80-712e-11eb-98ab-6615d825f9d0.png)
```
version: '3.1'

services:
   db:
     image: mysql:5.7
     volumes:
       - db_data:/var/lib/mysql
     networks:
       mysql_internal:
         aliases: ["db"]
     environment:
       MYSQL_ROOT_PASSWORD_FILE: /run/secrets/db_root_password
       MYSQL_DATABASE: wordpress
       MYSQL_USER: wordpress
       MYSQL_PASSWORD_FILE: /run/secrets/db_password
     secrets:
       - db_root_password
       - db_password

   wordpress:
     depends_on:
       - db
     image: wordpress:latest
     networks:
       mysql_internal:
         aliases: ["wordpress"]
       wordpress_public:
     ports:
       - "8001:80"
     environment:
       WORDPRESS_DB_HOST: db:3306
       WORDPRESS_DB_USER: wordpress
       WORDPRESS_DB_PASSWORD_FILE: /run/secrets/db_password
     secrets:
       - db_password

secrets:
   db_password:
     file: db_password.txt
   db_root_password:
     file: db_root_password.txt

volumes:
    db_data:
networks:
  mysql_internal:
    driver: "overlay"
    internal: true
  wordpress_public:
    driver: "overlay"
```
![image](https://user-images.githubusercontent.com/63472885/108193112-8775a300-712e-11eb-88b9-1eab8b3e3946.png)
![image](https://user-images.githubusercontent.com/63472885/108200360-ccea9e00-7137-11eb-9fdc-d717da6472bd.png)
![image](https://user-images.githubusercontent.com/63472885/108200468-f3a8d480-7137-11eb-8ebd-e3aef3ca9411.png)
![image](https://user-images.githubusercontent.com/63472885/108200541-0ae7c200-7138-11eb-894b-af2ac453c600.png)
```
docker service create \
     --name mysql_secrets \
     --replicas 1 \
     --network mysql_private \
     --mount type=volume,destination=/var/lib/mysql \
     --secret mysql_root_password \
     --secret mysql_password \
     -e MYSQL_ROOT_PASSWORD_FILE="/run/secrets/mysql_root_password" \
     -e MYSQL_PASSWORD_FILE="/run/secrets/mysql_password" \
     -e MYSQL_USER="myUser" \
     -e MYSQL_DATABASE="myDB" \
     mysql:5.7
```
![image](https://user-images.githubusercontent.com/63472885/108888537-07b37100-7625-11eb-9e9b-ad550e99286f.png)
![image](https://user-images.githubusercontent.com/63472885/108889772-2e25dc00-7626-11eb-94c9-55ba7f192677.png)
![image](https://user-images.githubusercontent.com/63472885/108890051-8361ed80-7626-11eb-9dc5-eabbe80d34d4.png)
![image](https://user-images.githubusercontent.com/63472885/108890958-9c1ed300-7627-11eb-92a6-271ba2a8b72a.png)
![image](https://user-images.githubusercontent.com/63472885/108891185-dab48d80-7627-11eb-8cf4-b1006bdf6fee.png)
![image](https://user-images.githubusercontent.com/63472885/108891833-9f668e80-7628-11eb-8461-c2011437a9fb.png)
![image](https://user-images.githubusercontent.com/63472885/108892491-5cf18180-7629-11eb-9754-ff7afe739a30.png)
![image](https://user-images.githubusercontent.com/63472885/108892575-772b5f80-7629-11eb-9587-e2810b87c18c.png)
![image](https://user-images.githubusercontent.com/63472885/108893915-10a74100-762b-11eb-9f90-e26114cc2ffc.png)
![image](https://user-images.githubusercontent.com/63472885/108894099-4fd59200-762b-11eb-8072-9d4188102666.png)
![image](https://user-images.githubusercontent.com/63472885/108894335-962af100-762b-11eb-9154-e87bd4d91b1e.png)
![image](https://user-images.githubusercontent.com/63472885/108894626-f3bf3d80-762b-11eb-8d5b-2cefe2f2eed3.png)
![image](https://user-images.githubusercontent.com/63472885/108894958-67614a80-762c-11eb-9cf5-9f193fb7ca1a.png)




































































