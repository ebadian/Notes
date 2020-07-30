- [Docker](#org5be78d4)
  - [Installation](#org2a72ca3)
    - [Auto-compelete](#org84c219e)
  - [Creating and using containers](#org2895281)
    - [checking config and isntallation](#orgbbc131a)
    - [Image vs container](#org056667b)
    - [Running a container](#orga853b0b)
  - [Play](#orgb27e120)
  - [Basic look inside container](#orgda89eae)
  - [Running Shell commands in a container](#orgcdef464)
  - [Networks](#org2af5361)
    - [Networks CLI](#org98c8329)
  - [Container images finding and buidling them](#org70e178f)
    - [What is an images?](#orgfd6049f)
  - [Building Docker Images](#org9df7676)
    - [prunning images](#org2799011)
  - [Volumes and Bind mounts](#org44b3012)
    - [persistance data](#org07c2d54)
  - [Docker Compose](#org28e5501)
    - [docker-compse.yaml](#org5e40a84)
    - [docker-compose CLI](#org89834b5)



<a id="org5be78d4"></a>

# Docker


<a id="org2a72ca3"></a>

## Installation


<a id="org84c219e"></a>

### Auto-compelete

See [docker guide](https://docs.docker.com/docker-for-mac/#install-shell-completion). You might need to add `autoload -Uz compinit; compinit` to the end `.zshrc`

-   Installing on Linux

    Go to [docker store](http://store.docker.com) it has instructions and dont install via package mamanger. This will get you the edge builds, which should be fine as docker moves fast.
    
    A cheat for installing is a script that you find [here](http://get.docker.com). This gives a script that will install docker with ease. Inorder to stop using `sudo` with every command you have to add your user to the docker group:
    
    ```
    sudo usermod -aG docker <USERNAME>
    ```
    
    Then log out and log back in again. This is not recomended bu security so dont do that on prod. Some distros do not allow you to do this.
    
    Docker machine and docker compose have to ne installed manually on Linux boxes. [Docker machine](https://github.com/docker/machine/releases) [Docker compose installer](https://github.com/docker/compose/releases)
    
    Since these are manually installed you might have manually update them.

-   Shell customization

    [see bret's customization](http://bretfisher.com/shell)


<a id="org2895281"></a>

## Creating and using containers


<a id="orgbbc131a"></a>

### checking config and isntallation

-   `docker version`

-   `docker info`


<a id="org056667b"></a>

### Image vs container

-   An image is the application we want to run

-   A container is an instance of that image running as a process

    You can have many instances of an image running at the same time.


<a id="orga853b0b"></a>

### Running a container

`docker container run --publish 80:80 ngix`

-   This downloads 'ngix' image from docker hub.
-   Starts a container from that image.
-   Opened port 80 on the host IP
-   Routes that traffic to the container IP, port 80.

You can use the option `--detach` to run the above command in the background. The command will return a unique ID hack. To see list the containers running use `docker container ls`. To stop a running container use `docker container stop <CONTAINER ID>`. By default docker will give a random name that you use to address the container. You can set the name of the container by using `--name <NAME>` when you start the container. `docker container logs <CONTAI"NER NAME>` will show you the logs produced by the container. To clean up the stoped container you can issue the command `docker container rm <CONTAINER ID> <CONTAINER ID> ..`

What happens in `docker container run` 1.Looks for that image locally in image cache, doesn't find anything 2.Then looks in remote image repository (defaults to Docker Hub) 3.Downloads the latest version (nginx:latest by default) 4.Creates new container based on that image and prepares to start 5.Gives it a virtual IP on a private network inside docker engine 6.Opens up port 80 on host and forwards to port 80 in container 7.Starts container by using the CMD in the image Dockerfile

Docker is a procerss.


<a id="orgb27e120"></a>

## Play

```bash
docker container run --detach --publish 80:80  --name nginx  nginx
docker container run --detach --publish 3306:3306  --name sql  --env MYSQL_RANDOM_ROOT_PASSWORD=yes mysql
docker container logs sql | grep "PASSWORD"
docker container run --detach --publish 8080:80  --name server httpd
docker container ls -a
docker container stop server sql nginx
```


<a id="orgda89eae"></a>

## Basic look inside container

`docker container top` - process list in one container `docker container inspect` - details of one container config `docker container stats` - performance stats for all containers


<a id="orgcdef464"></a>

## Running Shell commands in a container

`docker container run -it` - Starts the new container interactivly e.g:

```
docker container run -it --name proxy nginx bash
```

The above example what we did was to override the default command that runs with a `bash` command.

`docker container exec -it` - Run command in container currently running e.g:

```
docker container exec -it mysql bash
```

The above example opens a terminal in the running container `mysql` `-it` is two seprate options: `-t` also `-tty` this is a pseudo-tty -Simulates a real terminal like SSH. `-i`, `--interactive` Keeps the session open to receive terminal intput.


<a id="org2af5361"></a>

## TODO Networks

`--publish` (`-p`) format in HOST:CONTAINER


<a id="org98c8329"></a>

### Networks CLI

-   TODO What is NAT in networking

-   `docker container inspect` This shows you the container configuration such as specific container like Networks, IP address, mounts, and current status. It can also be modified to only return a specific piece of data using the —format flag.


<a id="org70e178f"></a>

## Container images finding and buidling them


<a id="orgfd6049f"></a>

### What is an images?

Just App binaries and dependencies with metadata about the image and how to run it. There is no kernal or any drivers involved, those are provided by the host. (This is different to Virtual Machines)


<a id="org9df7676"></a>

## Building Docker Images

Dockerfiles are layered, each layer is cached so in the next build if the layer has not changed (its hash hasn't changed) it will not rebuild it from scratch.

```docker
FROM //Typical start of docker file. Normally something small like Alpine linux
ENV  //Sets enviroment vars
ARGS //Set Variables
RUN  // Command to run for set up such as apt-get
WORKDIR // this is prefered way of changing directory over RUN cd
EXPOSE //This opens up a port so that you can 'map' to it when running the image using -p option (e.g 80:3000)
CMD     //The last command to run, such as /bash. This is inhereted from base image if not included.
```

For variables that are needed to build an image only its best to use `ARG` and inorder to set variables that are needed as *Enviroment Variables* in the ruuning container `ENV` is best suited.


<a id="org2799011"></a>

### TODO prunning images


<a id="org44b3012"></a>

## Volumes and Bind mounts


<a id="org07c2d54"></a>

### persistance data

Containers are immutable and ephermal, this means data will disapear as soon as its removed. So data such databases might lose data. To keep the data docker in two main ways:

-   Data Volumes

    Makes a special location outside of container UFS. Volumes live after the container and are only removed manually.
    
    -   Creating Volume
    
        To Create a volume create an entry in the docker file:
        
        ```Dockerfile
        Volume /var/lib/mysql
        ```
        
        This is setting the file path within the container file system which we want to persist on the host. To see current volume and thier set up
        
        ```bash
        docker image inspect 
        ```
        
        Here you can see the image config including `volumes` and `mounts`. In the mount section you can see where the volume is mounted, i.e how/where the host file is mapped to the container path. Under `"Mounts"` the field `source` gives the location of the host that the volume is writting to. The volumes currently mounted can be listed:
        
        ```bash
        docker volume ls
        ```
        
        The `volume name` from above command then can be passed into
        
        ```
        docker volume inspect <volume name>
        ```
        
        this will give all the details about the volume, including `mount point` and `scope`. Inorder to use have more meaningful name for the volume we can use the `-v` option when we run the image.
        
        ```bash
        docker container run -d --name mysql -e MYSQL_ALLOW_EMPTY_PASSWORD=True -v mysql-db:/var/lib/mysql mysql
        ```
        
        This is called the named volume. The syntax is as follows `volumename:volume location`.

-   TODO Bind Mounts

    Link container path to host path, i.e. maps hots file/directory to the container file/directory. When the file in the host changes the effect is seen in the running container. Bind mounts are used to speed up developments. Bind mounts can't be defined in the Dockerfile, they must be defined at `container run`. The syntax is similar to named volumes, the difference is the bind mounts start with `/` .
    
    ```bash
    docker container run -d --name ngix -p 80:80 -v $(pwd):/user/share/nginx/html nginx
    ```
    
    The `$(pwd)` saves us from typing the full path of current directory.


<a id="org28e5501"></a>

## Docker Compose

Docker compose is used to configure relationships between containers. It can also be used to save our container run settings in an easy to read file. It helps to create a one liner run command to bing up a whole stack.


<a id="org5e40a84"></a>

### docker-compse.yaml

In this file, the solution options for container, networks and volumes are defined. `compose` YAML format has its own versions, with each version more features have been added to docker. These files can be used locally by `docker-compose` or directly in production with Swarm by `docker` command. The default name for these files is `docker-compose.yaml` however they can be anything by using `docker-compose -f` Full reference for [is here](https://docs.docker.com/compose/compose-file/). However here is quick example of a `docker-compose.yaml` :

-   TODO &#x2013; comment on each line like in videos @ 3:46

    ```yaml
    version: '3.1'  # if no version is specified then v1 is assumed. Recommend v2 minimum
    
    services:  # containers. same as docker run
     servicename: # a friendly name. this is also DNS name inside network
       image: # Optional if you use build:
       command: # Optional, replace the default CMD specified by the image
       environment: # Optional, same as -e in docker run
       volumes: # Optional, same as -v in docker run
     servicename2:
    
    volumes: # Optional, same as docker volume create
    
    networks: # Optional, same as docker network create
    
    
    
    
    ```


<a id="org89834b5"></a>

### docker-compose CLI

seperate binary (bundled with mac and windows but linux have download/install sepreatly) suited for local not prod.

docker-compose up docker-compose down
