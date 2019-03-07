# Docker

Docker is a Container managing software. Containers or in general 
Container based virtualization uses the kernel on the host's OS 
to tun multiple guest istances. Each guest istance is called a 
container, and each container has its own:

* root fs
* processes
* memory
* devices
* network ports

An example is for example even if we have to run multiple 
versions of java for different applications, each one using a 
different virtual machine.

The advantages of using Containers vs VMs are:

* containers are more lightweight
* no need to install guest OS
* less cpu, ram, storage space required
* more containers per machine than VMs
* greater protability


## Docker Installation


Docker engine is the program that enables containers to be built, 
shipped and run, docker engine uses Linux kernel namespaces and 
control groups, namespaces give us the isolated workspace. Una 
volta installato e avviato il demone, con:

```sh
 systemctl start docker
```

oppure se non abbiamo il demone possiamo avviare l'applicazione 
con "sudo docker -d &" questa è la modalità demone.

Possiamo provarne il funzionamento andando ad avviare:

```sh
 sudo docker run hello-world
```

questa istruzione dovrebbe dirci che l'immagine hello world non 
esiste, abbiamo dovuto usare sudo, per poter avviare docker con 
altri utenti dobbiamo aggiungerli al gruppo docker, con:

```sh
 sudo usermod -aG docker giuseppe
```

now we logout and relogin, in order make the changes take effect.

To look at the docker version we do:

```sh
 docker version
```

## Images vs Containers

Let's see the difference between images and containers.

Images:
 * read only template used to create containers
 * built by me or other docker users
 * stored in the docker hub or my local registry

Containers:
 * isolated application platform
 * contains everything needed to run my app
 * based on one or more images

we can browse images on **dockerhub**, and the one called "library/java" or 
"library/nginx", so in general starting with "library/" are the official ones.

Now it is important to define the difference between an image and 
a live container.

 * An image: is a snapshot which does not change unless we do a 
   commit, in order to instantiate an image we can do a "docker run"
 * A container is a living image, what we do here is persistent, 
   and we can have more containers which were originated from the 
   same image


## Display local images

To display local images we can do:

```sh
 docker images 
 # displays docker images
```

notice that each docker image has "tags" associated to it, these 
tags represent the actual version, for example for java we could 
have tags like "6-jre, 6-jdk, latest, 6b32, etc..." so tags refer 
to various versions, the default tag is "latest". So images are 
specified by "image:tag".

Containers can be specified using their ID or name, there are:

* long ID
* short ID

short ID and name of running containers can be obtained using:

```sh
 docker ps
```
while long ID are obtained by inspecting the container.
To display all the available containers we can do:

```sh
 docker ps -a
```

We can delete all currently running containers shown with `docker ps -a`
by issuing:
```sh
docker rm $(docker ps -a -q -f status=exited)
```

or in more recent versions we can also do:
```sh
docker container prune
```

We can also launch a docker image and automatically remove it when it exits:
```sh
docker run --rm alpine
```

## Creating a Container


To create a container we do:

```sh
 sudo docker run [options] [image] [command] [args]
```
so docker will create the image if it doesn't exist and execute 
the eventual command (if specified), so for example we could do:

```sh
 docker run ubuntu:14.04 echo "Hello World"
```
or

```sh
 docker run ubuntu:14.04 ps -aux
```
notice that docker run = docker create + docker start, hence any 
time we do a docker run, a new container is created, in order to 
have persistance among states, we should launch docker with start 
and attack

## Connecting to the Container


To instantiate an image and connect directly to the container 
with a shell we do:

```sh
 docker run -i -t ubuntu:latest /bin/bash 
```

```sh
docker run -itd --name my-http-container-1 -p 5555:80 my-httpd:latest
```

in this case the "-i" flag tells docker to connect to the STDIN 
on the container, and the flag "-t" flag specifies to get a 
pseudo-terminal. We MUST remember that the docker container is 
alive only when the process is alive and changes are not written 
by default, so we can do whatever we want and changes will not be 
done to our image. Anyway changes will continue to live in the 
container. We can view it with:

```sh
 docker ps -a
```

and we can connect back to it with:
```sh
 docker start <id>
```

```sh
 docker attach <id>
```

or more simply:

```sh
 docker start -ia <id>
```

We can launch in a more advanced manner with:

## Docker Detached Mode


Now we do:

docker run -d centos:7 ping 127.0.0.1 -c 100 in this case the 
docker process detaches itself from the current shell, so the 
operation specified is acting in the background, and only an ID 
is given to us, we can view it with "docker ps", viewing even the 
short ID, now with the short ID we can see what's printing on the 
standard output our process, with:

docker logs 62ba075bee18 in this case we see the ping output, 
since it was the command we gave. 

We can even attach ourself to the log file with:

docker logs -f 62ba075bee18

in this case we see the stdout in real time.

## Practical Example: A Web Application Container


Now we want to run a web application inside a container, we'll 
use the "-P" flag to map contiainer ports to host ports, so we 
do:

```sh
docker run -d -P --name tomcat tomcat:latest
# with -d we start it in detached mode
# with -P will expose all public ports to random ports
# with --name we assign a name to the container
```

We can check which are the assigned random ports by doing:
```sh
docker port static-site
```

now we can do:

docker ps here we see the short ID of our applications and the 
portsgiven to our system, indeed we'll see a string under "PORTS" 
with for example: "0.0.0.0:49153->8080/tcp" this means that the 
port 8080 of the container has been mapped to the port 49153 on 
the host system.

To kill an existing docker process, once we have seen its ID, we 
can do:

docker kill 62ba075bee18

now if we do something, like creating a new file or modifying an 
existing file, we can exit, and once we exit in order to get back 
we should launch docker with start and attach

## Practical Example: Giving Graphics (Xorg) and Sound to an 

  application

Before giving sound and video to a container, we have to enable 
the host machine to accept connections to the Xorg display from 
the containers, we can do this by typing (before starting the 
docker container):

```sh
 xhost local:localuser 
```
```sh
 xhost + 
 # it is an alternative to the previous command, 
 # allowing everyone to connect to the Xorg server
```

it's useful to put this in an initialization configuration file, 
such as "/etc/profile" or things of this kind. 

Once we have done this, the following command let us run 
graphical and/or audio application from inside the container:

```sh
docker run \
-v /tmp/.X11-unix:/tmp/.X11-unix \ # mount the X11 socket 
-e DISPLAY=unix$DISPLAY \ # pass the display called unix0
--device /dev/snd \ # sound
```

let's see an analogous example in which I ran a kali linux distro 
docker image:

```sh
docker run -it --device /dev/snd -v /tmp/.X11-unix:/tmp/.X11-unix 
-e DISPLAY=unix$DISPLAY kalilinux/kali-linux-docker /bin/bash
```

so we just have to remember that each time we have to run a 
graphical application we must provide:

 * necessary directories (-v)
 * necessary devices (--device)
 * necessary environment variables (-e)

notice that on SELinux (e.g., Fedora and friends), systems there 
are additional policies that have to be respected, in order to 
bypass them temporarily we just have to run:

```sh
 su -c "setenforce 0"
```
this is unsafe, to ensure safeness, look at SELinux, and how to 
add/modify policies, probably a command like:

```sh
 chcon -Rt svirt_sandbox_file_t /path/to/volume 
 # check command
```

## Image Layers

Images are comprised of multiple layers, a layer is also just 
another image, but everyimage contains a base layer, layers are 
read only. When we launch a container, docker creates a top 
writable layer for containers, parent images are read only, so 
all changes are made at the writeable layer. To save changes in a 
container as a new image we do:

```sh
docker commit [options] [container ID] [repository:tag]
```

repository name should be based on username/application, we can 
reference the container with the container name instead of ID, 
let's see an example, we can do:

```sh
docker commit 984d25f537c5 johnnyty/myapplycation:1.0
```

if we don't specify the tag, docker uses the default one "latest".
Let's see a scenario, so we do:

```sh
 docker run -it ubuntu:15.05 bash
```
then we do:

```sh
 sudo apt install curl
```

now we can exit the docker image, and see its ID with:

```sh
 docker ps -a 
 # view docker ID of the exited docker image we 
 # want to save the state
```

from here we copy its short ID and do:

```sh
 docker commit 984d25f537c5 johnnyty/myapp:1.0 
 # here the code is the short ID
```

now we can run 

```sh
 docker images 
 # this will let us view the last created image
```
and now we can run our image as we did before but with the new 
name, so with:

```sh
 docker run -it johnnyty/myapp:1.0
```

## Dockerfile

A dockerfile is a configuration file that contains instructions 
for buildinf a Docker image. Basically we have:

 * FROM instructions: who specify the base image to use, such as:
  -- FROM ubuntu:14.04

 * RUN instructions: who specify commands to execute, such as:
  -- RUN apt-get install vim
  -- RUN apt-get install curl

so an example dockerfile can be built as the following:

```docker
#Example of a comment

FROM ubuntu:14.04
RUN apt-get install vim
RUN apt-get install curl
```

the fact is that if we have 10 RUN instructions we do 10 commits, 
to avoid this we can use the "&&" shell operator to aggregate RUN 
instructions together, so we can build dockerfile with:

```docker
FROM ubuntu:14.04

RUN apt-get update && apt-get install -y \
	curl \
	vim \
	openjdk-7-jdk
```

now to build an image following this dockerfile we do:

```sh
 docker build -t myrepo/myapp:1.0 path/to/folder/containing/theDockerfile
```

let's see another example:

```sh
 docker build -t myrepo2/mywebapp:latest . 
 # the build context 
 # here is the current directory
```

Notice that the dockerfile should be named "Dockerfile", we can 
even choose another name, but in this case we should mention the 
filename with the flag "-f"

In the Dockerfiles we can even specify commands that should be 
executed once the container is executed, this is done through the 
"CMD" directive, let's see an example:

```docker
FROM ubuntu:14.04

RUN apt-get update && apt-get install -y \
	curl \
	vim
CMD ping -c 10 127.0.0.1
```

these commands anyway can be overridden by specifying a command 
in docker run, as we did in the first examples.

We can even specify the "ENTRYPOINT" directive which executes the 
image as an executable for example:

```docker
ENTRYPOINT ["ping"]
```

once we have put this at the end of our Dockerfile, when we run 
the docker image the commands do not override the ping command 
specified, instead they are taken as argument, this docker image 
acts exactly like an executable.

### Start and Stop Containers or exec other processes


We can list all containers with:

```sh
 docker ps -a
```
now we can start a container in background as nginx with:

```sh
 docker run -d nginx
```
we can stop this container with:

```sh
 docker stop <containerIDviewedWithPs-a>
```
we can resume the container with:

```sh
 docker start <containerIDviewedWithPs-a>
```
To start other processes within the same container (assuming the 
container has already started) we can do:

```sh
 docker exec -i -t <containerID> /bin/bash 
 # this opens another shell on the container
```
In order to create a container we can do:

```sh
 docker create 
 # creates a writeable container from the image 
 # and prepares it for running.
```
```sh
 docker run 
 # creates the container (same as docker create) and 
 # runs it. 
```

### Practical example: Tomcat

We can do:

```sh
 docker run -d tomcat:7 
 # this starts (and downloads if it # doesn't exist) 
 # an image of tomcat version 7 in background
```

then

```sh
 docker ps 
 # here we see the short ID of the tomcat image
```
now we can attach to the container with another process by doing:

```sh
 docker exec -it <containerID> /bin/bash 
 # this starts a shell 
 # on the container
```
now here for example we can run:

```sh
 ps -ef 
 # shows the active processes inside the container, here 
 # we'll see all the processes attached to the container
```
from here if we execute:

```sh
 exit
```

we won't close the container since the bash was not the process 
with PID 1, which it was actually the tomcat background initial 
execution.

Now in order to resume the image we can do:

```sh
 docker images
```
```sh
 docker start f357e2faab77 
 # restart it in the background
```
```sh
 docker attach f357e2faab77 
 # reattach the terminal & stdin
```
and we are back to our machine, we can also be faster and 
execute:

```sh
 docker start `docker ps -q -l`
```
```sh
 docker attach `docker ps -q -l`
```

## Delete Containers

To remove a container we first have to stop it and then run:

```sh
 docker rm <nameOfTheContainer>
```
or we can execute:

```sh
 docker rmi myrepo/myapp:1.0
```
```sh
 docker rmi -f myrepo/myapp:1.0 
 # in this case we delete the image in a forced way
```
we can verify the deletion of an image with:

```sh
 docker images
```

## Tagging Images

We can tag images or rename a local image repo with:

```sh
 docker tag imageID repo:tag
```
or:

```sh
 docker tag localRepo:tag anotherRepo:tag
```

for example:

```sh
 docker tag edfc1234je32 trainingteam/testexample:1.0
```

or

```sh
 docker tag johnny/testimage:1.5 trainingteam/testexample
```

## Copying data into a Container


In order to copy data in a container we can do:

```sh
 docker cp /path/to/myfile.txt name_of_container:/dest/path
```
```sh
 docker cp name_of_container:/dest/path  /path/to/myfile.txt
```
## Pushing Image to Remote Repo


We can push an image with:

```sh
 docker push johnnytu/testimage:1.0 
 # this will begin the push once we specify the credentials
```

note that this psh will give us errors if on our account there 
isn't yet any repo called "johnnytu/testimage" so we first have 
to create it if it doesn't exist.

## Volumes


A volume is a designated directory in a container, which is 
designed to persist data, independent of the container's life 
cycle. These volumes:

 * can be shared between containers
 * can be mapped to a host directory
 * persist when a container is deleted
 * volume changes are excluded when updating an image

Volumes are mounted when creating or executing a container, 
volume path must be absolute, let's see some example:

```sh
 docker run -d -P -v /myvolume nginx:1.7 
 # in this case we mount 
 # the dir /myvolume into the filesystem of the specified image
```
or another interesting example is:

```sh
 docker run -it -v /data/src:/test/src nginx:1.7 
 # in this case 
 # we map the /data/src directory from the host into the /test/src 
 # directory in the container
```
We can specify volumes even into Dockerfiles, let's see some 
example:

```docker
VOLUME /myvol
```

or even multiple volumes like:

```docker
VOLUME /www/website1 /www/website2 /myvol
```

or with a JSON notation like:

```docker
VOLUME ["myvol", "myvol2"]
```

Mounting folders from the host is food for testing purposes but 
generally not recommended for production use, indeed it is not 
possible to do the mappings inside the Dockerfiles.

## Mapping of ports and services


We don't always need mapping of ports, once we have runned our 
system it will have its IP address, let's say we have a shell, we 
can inspect the IP with the common command "ifconfig", once we 
have the IP address of the dockered machine, if we launch a 
service (e.g. apache) on the dockered machine we can find it at 
ip address: port, just like a machine which is in LAN. Notice 
that by default on most systems docker will configure the 
dockered machine inside a NATted network inside the host machine.

Ok, now let's see a scenario where we want to mirror/map a 
service on the host machine, like for example running apache on 
the guest dockered machine and running it like it was running on 
the host machine. Let's see an example of manual mapping of 
ports:

```sh
 docker run -d -p 8080:80 nginx:1.7 
 # maps port 80 on the 
 # container to port 8080 on the host
```
instead if we would like to do the automatic mapping we do:

```sh
 docker run -d -P nginx:1.7 
 # this will do the automatic port 
 # mapping, but this only works for ports defined in the "EXPOSE" 
 # instructions inside the Dockerfile
```
indeed to let our image to support automatic port mappings we 
should add the ports to enable for automatic mappings in the 
Dockerfile:

EXPOSE 80 443 in this case we are enabling automatic mapping for 
port 80 and 443 (HTTP and HTTPS).

## Linking Containers


To create a link, we first have to create the source containeir 
and the create the recipient container and then use the "--link", 
let's see an example: (BEST PRACTICE: give the containers 
meaningful names)

We first create the source container using the postgres:

```sh
 docker run -d --name database postgres
```
then we create the recipient container and link it:

```sh
 docker run -d -P --name website --link database:db nginx:1.7 
 # here, after --link we put the name of the source folowed by its alias
```

let's see another example:

```sh
 docker run -d --name dbms postgres:latest
```
```sh
 docker run -it --name website --link dbms:db ubuntu:14.04 bash 
```
now inside our ubuntu container if we cat /etc/hosts, we can see 
clearly an entry for our alias name called "db" with its own IP 
address. 

We could check this ip address even from outside the container 
with:

```sh
 docker inspect dbms | grep IPAddress
```
as we can see the two IPs will match.

## We can automate Build Repos!!


On dockerhub there is the possibility to automate the building of 
the repo, let's assume we have a java source called 
JavaHelloWorld.java, we then create a Dockerfile like this:

```docker
FROM java:7
COPY JavaHelloWorld.java .
RUN javac JavaHelloWorld.java

CMD["java", "JavaHelloWorld"]
```

now if we have configured our Dockerhub repo correctly when we 
commit and push from git to github or the configured git repo on 
Dockerhub Dockerhub will notice the commit and create a new 
Docker image. We can even download the image in a second moment 
with:

```sh
 docker pull myrepo/myjavapp
```
and then execute with:

```sh
 docker run myrepo/myjavapp
```
this will run my java application; if we change the code and 
commit+push with git a new image will automatically be created. 
This process is called "CI" (Continous Integration).

  Container Logging

We can show whatever PID 1 writes to stdout

```sh
 docker logs <containerName>
```
or we can view and follow the output with:

```sh
 docker logs -f <containerName>
```
for example we can start a container with tomcat with:

```sh
 docker run -d tomcat
```
then with

```sh
 docker ps 
```

we view the currently running containers with their name, now we 
can do:

```sh
 docker logs containerName 
 # it opens the log for the process ID 1 of that container
```
we can follow the output with:

```sh
 docker logs -f containerName
```

we can even map a a directory on our host to the application log 
directory, so that we can see and operate locally on the 
container logs, for example with:

```sh
 docker run -d -P -v /nginxlogs:/var/log/nginx nginx 
 # now we can see logs in our directory /nginxlogs
```

## Inspecting a Container


We can inspect a container by executing the command:

```sh
 docker inspect containerName 
 # this will display all the details of the container in a JSON array
```

we can use grep to filter for a specific detail, for example:

```sh
 docker inspect containerName | grep IPAddress 
 # this will show the IP address of the specified container
```

## Configurazione del demone docker

Il file di configurazione è localizzato in "/etc/default/docker", 
possiamo usare DOCKER_OPTS per controllare le opzioni di startup 
del demone quando runna come servizio, dobbiamo riavviare il 
servizio per fare in modo che le modifiche abbiano effetto con:

```sh
 sudo service docker restart 
 # riavvia il demone, è possibile 
 # anche con sudo systemctl restart docker
```

possiamo invece ad esempio debuggare runtime avviando docker col 
comando:

```sh
 sudo docker -d --log-level=debug
```

## Docker Security

Docker helps make applications safer as it provides a reduced set 
of default privileges and capabilities, namespaces provide an 
isolated view of the system, each container has its own set of:

 * IPC
 * network stack
 * rootfs
 * etc...

Processes running in one container cannot see and effect 
processes in another container, the technology at the base of all 
this, is "Control Groups" or CGroups which isolate resource usage 
per containers, anyway we must ensure that a compromised 
container won't bring down the entire host by exhausting 
resources. Quick security considerations are:

 * docker daemon service must be run only as root
 * watch who we add to the docker group
 * if binding the daemon to a TCP socket, secure it with TLS
 * use linux hardening solution, such as:
  * apparmor
  * SELinux
  * GRSEC


## Private Registry (alternative to DockerHub)

We can run a new container using the registry image with:

```sh
 docker run -d -p 5000:5000 registry:2.0 
 # we must run an image 
 # from dockerhub to make our own registry
```
once we have downloaded it, we can verify that it is running with 
"docker ps"

now to push and pull from our private registry we can do:

```sh
 docker tag <imageID> myserver.net:5000/my-app:1.0
```
then

```sh
 docker push myserver.net:5000/my-app:1.0 
 # where instead of 
 # myserver.net we can even put an IP address
```
while to pull an image from our registry we do:

```sh
 docker pull myserver.net:5000/my-app:1.0
```
let's see another example (done on localhost):

we first rename an image with:

```sh
 docker tag 91jnu21e9122 localhost:5000/myhello-world:1.0
```
then we do:

```sh
 docker push localhost:5000/myhello-world:1.0
```
notice that when we pull from an IP address we'll get an error if 
we are not using SSL/TLS, so we must modify docker options, but 
before we stop the docker daemon, with "sudo systemctl stop 
docker" and we do:

```sh
 sudo vim /etc/default/docker
```
and we modify the line:

```text
DOCKER_OPTS="--insecure-registry 104.131.142.17:5000"
```

and then we restart the docker daemon.


## Docker Compose

Docker compose is a tool for creating and managing multi 
container applications, containers are all defined in a single 
file colled "docker-compose.yml", and each container runs a 
particulart component/service of our application, for example:

 * web front end
 * user authentication
 * payments
 * database

container links should be defined and compose will spin up all 
our containers in a single command. We know that we can start 
different containers and link them together, but the number of 
components grows, this is very impractical, so compose helps us 
in this.

## Docker Useful Things

### Removing all the intermediate layers by exporting a container into a new image

Let's say we have an image which have committed several times, 
this image will contain all the history of the commissions which 
have been done, we first have to run the container in which we 
are interested in and then export this snapshot with:

```sh
 docker export my-running-image-id > ~/my_image.tar 
 # the 
 # provided id must be the one we see in "docker ps", N.B.: The 
 # container must be running, export works with running containers
```
then now we can delete all the containers we have in the docker 
directory "/var/lib/docker/containers/", or first delete the easy 
ones with:

```sh
 docker rmi -f <container ID>
```
once we have removed even the images contained in the docker 
directory we have to import our tarball archive, we can do this 
with:

```sh
 docker import ~/my_image.tar
```
our image if imported in this way will have name and repo name 
set to "<none>", now we can rename the image with:

```sh
 docker tag imageID repoName:imageName
```

### Docker Fast Track


Docker has the concept of images (which can be thought as blueprints/models/starting points
of virtualization environments) and containers which are instances of these
images.

Once docker is installed we can check installed images with:
```sh
docker images
# or
docker image -ls 
```

we can check living containers with:
```sh
docker container ls
# or
docker ps
# or 
docker container ls -a # to also check for not currently running containers
# or
docker ps -a # to also check for not currently running containers
```

We can download a new image from docker hub by doing:
```sh
docker pull <nameoftheimage>
```

At this point we can run this image in various ways, for example:
1. We can run it by attaching a live interactive shell

```sh
docker run -it -P --name <nameofthecontainer> <nameoftheimage>
```
2. We can run it in detached mode

```sh
docker run -d -P --name <nameofthecontainer> <nameoftheimage>
```
3. We can run it by just executing the commands it automatically executes and
   exit:
```sh
docker run --name <nameofthecontainer> <nameoftheimage>
```

We can also delete the container once the execution is stopped by appending the
`--rm` option to these commands.

We can also be specific on the mapping of ports by launching containers like
this:

```sh
docker run -it -p 8888:80 --name <nameofthecontainer> <nameoftheimage>
```


Once a container is running we can check its exposed ports with:
```sh
docker port <nameofthecontainer>
```

We can at any time delete containers with:
```sh
docker rm <idofcontainer>
# or to remove all exited containers, we can do:
docker rm $(docker ps -a -q -f status=exited)
```

