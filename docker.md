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



