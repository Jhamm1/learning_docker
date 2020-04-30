# learning_docker

#Docker tutorial

##Docker Flow

**Docker image:** is every file that makes up just enough of the OS to do what you need to do. Slim down version of an OS. The docker run command takes an image and turns it into a container with a process in it that is doing something. 

- docker images are read only

- `-ti` command is an interactive terminal that allows the user to interact with the container terminal

containers and images have different IDs (they are not overlapped).

You can make changes to the container which doesn’t affect the base image

`docker ps -l` command returns the last stopped container

docker commit command - takes containers and turns it into new base image

grab the outputted image id to name the image > docker tag [image Id] [image name]

grab the name of the container > docker commit [name of the container] [image name]

**Running things in docker**

Containers have a main process
The container stops when that process is terminated
Containers have names

run `docker run -d -ti ubuntu bash` > start a container and leave it running (detach)

run `docker run -ti ubuntu bash -c "sleep 3; echo all done”` >  sequential processes in a container


run, execute a command and then remove the container > docker run --rm -ti ubuntu sleep 5

detach and keep running > ctrl + p, ctrl + q

run more things in a container > docker exec -ti [container name] bash

keep the output of containers > docker logs [container_name]

Kill container > docker kill [container_name]

Remove container > docker rm [container_name]

Resource constraints

Memory limits > docker run —memory maximum-allowed-memory image-name command

CPU limits > docker run —cpu-shares relative to containers + docker run —cpu-quota to limit  it in general

Lessons from the field

Don’t let your containers fetch dependencies when they start
Make your dependencies be included in the containers themselves - save time
Don’t leave important things in unnamed stopped containers

Container networking
Programs in containers are isolated from the internet by default
You can group your containers into “private” networks - put each container on a network and they can talk to each other and still be isolated from the other containers that are running
You explicitly choose who na connect to whom

Expose/publishing ports to let connections in - that lets the ports accessible from outside the machine on which docker is being hosted

Private networks to connect between containers 

Exposing a specific port

Specify the port inside the container and outside

Expose many ports as you want

Requires coordination between containers

Makes it easy to find the exposed ports

Referring to the machine on which the docker container is running (as localhost or the ip address of the host machine is not recognised inside the container) >  host.docker.internal / if on windows, put the ip address of your machine or if a dns name - use that

Publish ports to be accessible from inside and out > docker run --rm -ti -p 45678:45678 -p 45679:45679 --name echo-server ubuntu:14.04 bash

Take the output from what’s running on port 45678 and sending it to whatever is running on port 45679 > nc -lp 45678 | nc -lp 45679

Exporting ports dynamically

The port inside the container is fixed

The port on the host is chosen from the unused ports

This allows many containers running the programs with fixed ports

This often is used with a service discovery program (e.g. Kubernetes)

Don’t specify external ports > docker run --rm -ti -p 45678 -p 45679 --name echo-server ubuntu:14.04 bash

Exposing UDP ports

docker run -p outside-port:inside-port/protocol (tcp/udp)

docker run -p 1234:1234/udp

Connecting between containers

When we expose a container’s port in docker, it creates a network path from outside of that machine and down through the networking layers and into that container Client container —> Server container

Virtual networks
bridge - is the network used by containers that don’t specify a preference to be put into any other network
host - no network isolation - this has some security concerns 
none - when a container shouldn’t have any networking

create a network in docker > docker network create learning

Create a machine on the network > docker run --rm -ti --net [network-name] --name [name-server] [image-name] bash

Put the machine on a particular network > docker network connect [network-name] [server-name]

List all running networks > docker network ls

Legacy linking

Links all ports, though only one way - only operates in one direction

Secret environment variables are shared only one way - visible to any machine that links to it, but not visible the other way round

Depends on startup order

Restarts only sometimes break the links

List images

List all docker images on local machine > docker images

Docker is space efficient

docker commit tags images for you

version number comes after the tag. If not specified, it defaults to latest > docker commit 5d727b3d524a my-image-14:v2.1

This is an example of the name structure: 

Tagging images > registry.example.come:port/organisation/image-name:version-tag

You can leave the other parts you don’t need > Organisation/image-name is often enough

Getting images

docker pull

Run automatically by docker run

Useful for offline work 

Opposite: docker push

Cleaning up

Images can accumulate quickly

docker rmi image-name:tag

docker rmi image:id

Volumes

Sharing data between containers and between container and hosts

Volumes are like shared folders 

Virtual “discs” to store and share data

Two main varieties: 
persistent: Put data there and it will be available on the host and when the container is removed the data will still be there
Ephemeral: This exists as long as the container is using them, but when no container is using them, they evaporate - not permanent 

Not part of images 

“Shared folders” with the host

Sharing a “single file” into a container 

Persistent data share (the folder will be on the host when you exit from the container) > docker run -ti -v [path to the host folder to be shared]:[path in the container] [image-name] bash

Sharing data between containers
volumes-from

Shared “discs” that exist only as long as they are being used

Can be shared between containers 

Docker registries

Docker images are retrieved from registries and published through registries

Pieces of software that manage and distribute docker images

Allows users to upload images to and download images from them

You can setup your own registry inside your IT infrastructure - private

Finding docker images

docker search [image-name] command

Pull an image, make changes and then push to a registry
docker login - enter docker login credentials
docker pull debian:sid
docker tag debian:sid jhamm27/test-image-42:v99.9 - tag image
docker push jhamm27/test-image-42:v99.9 - push the docker image

Considerations
Don’t push any images containing your password to docker hub

Clean up your images regularly

Be aware of the images that you’re fetching - who created it? and where?

If you have a container running bash in detached mode. In terminal A, you attach to this container. Next, in terminal B, you ‘docker exec bash’ to this container. Then, back in terminal A, you exit the session. What happens to the container? > The container and all its processes are terminated.

Building docker images

Docker file - Is a small program that describes how to build an image

You run this program with this command > docker build -t name-of-result . 

. - is where to find the docker file in the current directory

When it has finished being built, the result will be in your local docker registry

Each step in a docker file is cached

Docker skips lines that have not been changed since the last build

Dockerfiles look like shell scripts, but they are not!

Build a docker file

Provide the path to the example Dockerfiles inside the repo

docker build -t hello .

docker run --rm hello

Storing files in the image

install programs inside a container

cd learning_docker/installing_programs

docker build -t example/nanoer .

docker run --rm -ti example/nanoer

Adding a file through docker build

cd learning_docker/Add_a_file

docker build -t example/notes .

docker run -ti --rm example/notes

You can have multiple FROM statements in the Dockerfile

ENV statements sets env vars during the duration of the container

ENTRYPOINT - specifies the start of the command to run. If your container acts like a command-line program, you can use this command

CMD - specifies the whole command to run. Shell form

Multi-project Dockerfiles

Multi stage builds (Split docker file into 2 parts)

cd learning_docker/Multi_stage_builds

docker build -t tooo-big .

docker run tooo-big

Find out how big this image is > docker images

Preventing the golden image problem

Include the installers in your project

Have a canonical build that builds everything completely from scratch

Tag your builds with the git hash of the code that built it

Use small base images, such as Alpine

Build images you share publicly from Dockerfiles, always

Don’t ever leave passwords in layers; delete files in the same step!

Under the hood

Starting another container from client within a container

docker run -ti --rm -v /var/run/docker.sock:/var/run/docker.sock docker sh

docker run -ti --rm ubuntu bash

Client within a docker container controlling a server that is outside that container

Networking in brief

Ethernet: moves “frames” (of data in a local area) on a wire (or Wi-Fi)

IP layer: moves packets on a local network

Routing: forwards packets between networks

Ports: address particular programs on a computer

Bridging

Docker uses bridges to create virtual networks in your computer

These are software switches

They control the Ethernet layer

docker run -ti --rm --net=host ubuntu:16.04 bash

Routing

Moves packets between networks and containers 

Use the build-in linux firewall tables

NAT

sudo iptables -n -L -t nat

docker run -ti --rm --net=host --privileged=true ubuntu bash

apt-get update

apt-get install iptables

open another terminal > docker run -ti --rm -p 8080:8080 ubuntu bash

Exposing a port is really port forwarding

Namespaces

They allow processes to be attached to private network segments

These private networks are bridged into a shared network with the rest of the containers

Containers have virtual network cards

Containers get their own copy of the networking stack

Primer on Linux Processes

Processes come from other processes -- parent-child relationship

When a child process exits, it returns an exit code to its parent

Process Zero is special; called init, the process that starts the rest

In Docker, your container starts with an init process and vanish when that process exits

docker run -ti --rm --name hello ubuntu bash

docker inspect --format '{{.State.Pid}}' hello

> outputs a pid number

docker run -ti --rm --net=host --privileged=true --pid=host ubuntu bash

kill [pid number]

The other container is then terminated

Resource Limiting

Scheduling CPU time

Memory allocation limits

Inherited limitations and quotas

Storage

Copy On Write (COW)

Base image > over layered container

Moving Cows

The contents of layers are moved between containers in gzip files

Containers are independent of the storage engine

Any container can be loaded (almost) anywhere

It is possible to run out of layers on some of the storage engines

Volumes and bind Mounting 

The Linux VFS (Virtual File System)

Mounting devices on the VFS

Mounting directories on the VFS

Get the mount order correct

Mounting volumes — always mounts the host’s filesystem over the guest filesystem


Docker registry

registry as a service

authentication and security on registry > docs.docker.com/registry > AWS ECS

docker run -d -p 5000:5000 --restart=always --name registry registry:2

docker tag ubuntu:14.04 localhost:5000/mycompany/my-ubuntu:99

docker push localhost:5000/mycompany/my-ubuntu:99

Saving and loading containers

docker save 

docker load

docker save -o my-images.tar.gz debian:sid busybox ubuntu:14.04

docker load -i my-images.tar.gz

Migrating between storage types

Shipping images on disks (or never underestimate the bandwidth of a thumb drive in a jetliner)


