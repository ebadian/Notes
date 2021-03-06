
# Table of Contents

-   [Docker](#org4ed3a5d)
    -   [Installation](#org28ba25e)
    -   [Creating and using containers](#org269c50d)
    -   [Play](#org7a28880)
    -   [Basic look inside container](#orgfc0205d)
    -   [Running Shell commands in a container](#orgf5cfe6e)
    -   [Networks](#org9e3569c)
    -   [Container images finding and buidling them](#org13a0dc2)
    -   [Building Docker Images](#orgc5db4be)
    -   [Volumes and Bind mounts](#orgf373425)
    -   [Docker Compose](#org0fd9c00)
-   [Swarm](#org2728c19)
    -   [RAFT](#org109406e)
    -   [Manager Node](#org50d1835)
    -   [Worker Node](#org48c3fb9)
-   [Kubernetes](#org58e02b0)
    -   [Architecture](#orgfea2cb6)
    -   [Minikube](#org6418497)
    -   [Helm, packaaging and deploying](#org3d0ad02)
-   [Cheat sheets](#org0d5ae44)



<a id="org4ed3a5d"></a>

# Docker


<a id="org28ba25e"></a>

## Installation


### Auto-compelete

See [docker guide](https://docs.docker.com/docker-for-mac/#install-shell-completion).
You might need to add `autoload -Uz compinit; compinit` to the end `.zshrc`

-   Installing on Linux

    Go to [docker store](http://store.docker.com) it has instructions and dont install via package mamanger.
    This will get you the edge builds, which should be fine as docker moves fast.
    
    A cheat for installing is a script that you find  [here](http://get.docker.com). This gives a script that will install docker with ease.
    Inorder to stop using `sudo` with every command you have to add your user to the docker group:
    
        sudo usermod -aG docker <USERNAME>
    
    Then log out and log back in again.
    This is not recomended bu security so dont do that on prod. Some distros do not allow you to do this.
    
    Docker machine and docker compose have to ne installed manually on Linux boxes.
    [Docker machine](https://github.com/docker/machine/releases)
    [Docker compose installer](https://github.com/docker/compose/releases)
    
    Since these are manually installed you might have manually update them.

-   Shell customization

    [see bret&rsquo;s customization](http://bretfisher.com/shell)


<a id="org269c50d"></a>

## Creating and using containers


### checking config and isntallation

-   `docker version`

-   `docker info`


### Image vs container

-   An image is the application we want to run

-   A container is an instance of that image running as a process

    You can have many instances of an image running at the same time.


### Running a container

`docker container run --publish 80:80 ngix`

-   This downloads &rsquo;ngix&rsquo; image from docker hub.
-   Starts a container from that image.
-   Opened port 80 on the host IP
-   Routes that traffic to the container IP, port 80.

You can use the option `--detach` to run the above command in the background. The command will return a unique ID hack.
To see list the containers running use `docker container ls`. 
To stop a running container use `docker container stop <CONTAINER ID>`.
By default docker will give a random name that you use to address the container. You can set the name of the container by using `--name <NAME>` when you start the container.
`docker container logs <CONTAI"NER NAME>` will show you the logs produced by the container.
To clean up the stoped container you can issue the command `docker container rm <CONTAINER ID> <CONTAINER ID> ..`

What happens in `docker container run`
   1.Looks for that image locally in image cache, doesn&rsquo;t find anything
   2.Then looks in remote image repository (defaults to Docker Hub) 
   3.Downloads the latest version (nginx:latest by default) 
   4.Creates new container based on that image and prepares to start 
   5.Gives it a virtual IP on a private network inside docker engine 
   6.Opens up port 80 on host and forwards to port 80 in container 
   7.Starts container by using the CMD in the image Dockerfile

Docker is a procerss.


<a id="org7a28880"></a>

## Play

    docker container run --detach --publish 80:80  --name nginx  nginx
    docker container run --detach --publish 3306:3306  --name sql  --env MYSQL_RANDOM_ROOT_PASSWORD=yes mysql
    docker container logs sql | grep "PASSWORD"
    docker container run --detach --publish 8080:80  --name server httpd
    docker container ls -a
    docker container stop server sql nginx


<a id="orgfc0205d"></a>

## Basic look inside container

`docker container top`     - process list in one container   
`docker container inspect` - details of one container config    
`docker container stats`   - performance stats for all containers   


<a id="orgf5cfe6e"></a>

## Running Shell commands in a container

`docker container run -it` - Starts the new container interactivly
 e.g:

    docker container run -it --name proxy nginx bash

The above example what we did was to override the default command that runs with a `bash` command.

`docker container exec -it` - Run command in container currently running
e.g:

    docker container exec -it mysql bash

The above example opens a terminal in the running container `mysql` 
`-it` is two seprate options:
`-t` also `-tty` this is a pseudo-tty -Simulates a real terminal like SSH.   
`-i`, `--interactive` Keeps the session open to receive terminal intput.


<a id="org9e3569c"></a>

## TODO Networks

`--publish` (`-p`) format in HOST:CONTAINER


### Networks CLI

-   TODO What is NAT in networking

-   `docker container inspect` This shows you the container configuration such as specific container like Networks, IP address, mounts, and current status. It can also be modified to only return a specific piece of data using the —format flag.


<a id="org13a0dc2"></a>

## Container images finding and buidling them


### What is an images?

Just App binaries and dependencies with metadata about the image and how to run it.
There is no kernal or any drivers involved, those are provided by the host. (This is different to Virtual Machines)


<a id="orgc5db4be"></a>

## Building Docker Images

Dockerfiles are layered, each layer is cached so in the next build if the layer has not changed (its hash hasn&rsquo;t changed) it will not rebuild it from scratch.

    FROM //Typical start of docker file. Normally something small like Alpine linux
    ENV  //Sets enviroment vars
    ARGS //Set Variables
    RUN  // Command to run for set up such as apt-get
    WORKDIR // this is prefered way of changing directory over RUN cd
    EXPOSE //This opens up a port so that you can 'map' to it when running the image using -p option (e.g 80:3000)
    CMD     //The last command to run, such as /bash. This is inhereted from base image if not included.

For variables that are needed to build an image only its best to use `ARG` and inorder to set variables that are needed as *Enviroment Variables* in the ruuning container `ENV` is best suited.


### TODO prunning images


<a id="orgf373425"></a>

## Volumes and Bind mounts


### persistance data

Containers are immutable and ephermal, this means data will disapear as soon as its removed. So data such databases might lose data. To keep the data docker in two main ways:

-   Data Volumes

    Makes a special location outside of container UFS. Volumes live after the container and are only removed manually.
    
    -   Creating Volume
    
        To Create a volume create an entry in the docker file:
        
            Volume /var/lib/mysql
        
        This is setting the file path within the container file system which we want to persist on the host.
        To see current volume and thier set up
        
            docker image inspect 
        
        Here you can see the image config including `volumes` and `mounts`. In the mount section you can see where the volume is mounted, i.e how/where the host file is mapped to the container path. Under `"Mounts"` the field `source` gives the location of the host that the volume is writting to.
        The volumes currently mounted can be listed: 
        
            docker volume ls
        
        The `volume name` from above command then can be passed into
        
            docker volume inspect <volume name>
        
        this will give all the details about the volume, including `mount point` and `scope`.
        Inorder to use have more meaningful name for the volume we can use the `-v` option when we run the image.
        
            docker container run -d --name mysql -e MYSQL_ALLOW_EMPTY_PASSWORD=True -v mysql-db:/var/lib/mysql mysql
        
        This is called the named volume. The syntax is as follows `volumename:volume location`.

-   TODO Bind Mounts

    Link container path to host path, i.e. maps hots file/directory to the container file/directory.
    When the file in the host changes the effect is seen in the running container. Bind mounts are used to speed up developments.
    Bind mounts can&rsquo;t be defined in the Dockerfile, they must be defined at `container run`. 
    The syntax is similar to named volumes, the difference is the bind mounts start with `/` .
    
        docker container run -d --name ngix -p 80:80 -v $(pwd):/user/share/nginx/html nginx
    
    The `$(pwd)` saves us from typing the full path of current directory.


<a id="org0fd9c00"></a>

## Docker Compose

Docker compose is used to configure relationships between containers. It can also be used to save our container run settings in an easy to read file. It helps to create a one liner run command to bing up a whole stack. 


### docker-compse.yaml

    In this file, the solution options for container, networks and volumes are defined.
    `compose` YAML format has its own versions, with each version more features have been added to docker.
    These files can be used locally by `docker-compose` or directly in production with Swarm by `docker` command.
The default name for these files is `docker-compose.yaml` however they can be anything by using `docker-compose -f`
Full reference for [is here](https://docs.docker.com/compose/compose-file/). However here is quick example of a `docker-compose.yaml` :

-   TODO &#x2013; comment on each line like in videos @ 3:46

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
    
    -   docker-compose CLI
    
        seperate binary (bundled with mac and windows but linux have download/install sepreatly)
        suited for local not prod.
        
        docker-compose up
        docker-compose down


<a id="org2728c19"></a>

# Swarm

Swarm has to be enabled. Whether swarm is enabled

    docker info | grep -i "swarm"


<a id="org109406e"></a>

## TODO RAFT


<a id="org50d1835"></a>

## TODO Manager Node


<a id="org48c3fb9"></a>

## TODO Worker Node

\*\*


<a id="org58e02b0"></a>

# Kubernetes

It is a popular orchestrator released by Google and maintained by comminuty. It runs on top of docker as a set of APIs in conatiners which provides API/CLI to manage conayiners across servers. Most cloud providers will have thier own distrobution of it available, like the EKS from aws.


<a id="orgfea2cb6"></a>

## TODO Architecture


### Control panel / Master

-   TODO CoreDNS

-   ETCD

    It is a distributed rliable key-value store that is simple secure and fast.
    Can be installed on any OS and once its running (`./etcd`) it listens to port `2379`.
    Crude analogy, its like a hashmap.
    Basic usage:
    
        ./etcdctl set key1 value1
        ./etcdctl get key1
    
    It&rsquo;s role in Kubernetes is to manage all information about:
    
    -   Nodes
    -   PODs
    -   Configs
    -   Accounts
    -   Roles
    -   Binding
    
    Whithin Kubernetes ETCD is resposible for everything that is seen when `kubectl get` command is issued. Any changes to Kubernetes is only considered done once the `etcd` is upated.
    
    -   TODO Manual Kubernetes setup
    
        ETCD has to be downloaded and installed manually
    
    -   TODO Using kubeadm to install
    
        It will install etcd as a pod
        To explore what keys are stored on the etcd server run
        
            kubectl exec etcd-master -n kube-system etcdctl get / --prefix -keys-only

-   Kube API

    This is the primaray managment component, all communcations/interactions in and to kubernetes, even between components, are done via the api server.
    When a command or a post resquest is sent API, it will carry the following steps:
    
    -   Authenticate User
    -   Validates the request
    -   Retrive the data
    -   Updates the ETCD and related component such as Schedular and Kubelets
    
    If the kubernetes is installed via `kubeadm` then it is deployed as a pod on the master node and can be views using:
    
        kubectl get pods -n kube-system
    
    And its options can be viewed in `/etc/kubernetes/manifests/kube-apiserver.yaml`
    In a none-kubeadm setup the options are located in `/etc/systemd/system/kube-apiserver.service`
    The current running options can also be viewed via:
    
        ps -aux | grep kube-apiserver

-   Kube Controller manager

    It&rsquo;s primary task is to monitor the state of the cluster and reconcile the current state if and when current state is not aligned with desired state. It does this by running multiple controller processes. These controllers include:
    
    -   replication controller
    -   endpoints controller
    -   node controller
    -   token controller
    -   service controller
    
    If the kubernetes has been installed with kubeadmn, it&rsquo;s installed as a pod in `kube-system` namespace. With such installation the options for this component can be found in `/etc/kubernetes/manifests/kube-controller-manager.yaml`
    In a non-kubeadmn install these options maybe found in `/etc/systemd/system/kube-controller-manager.service`
    Again the current effective options can also be viewed using:
    
        ps -aux | grep kube-controller-manager

-   Schedular

    It decides which pods go on to which nodes. It doesn&rsquo;t place the pods on the nodes, that is the job of the `kubelet`.
    Schedular decides on which notes are best to place the pods based on criterias such as resources.
    It goes through two phases in order to make this decision.
        1 - Filtering Nodes
          First it will filter out nodes which do not meet the resources criteria for the pod. For example if the pods needs 5 cpus, then all the pods with less CPUs available are filtered out.
        2 - Ranking Nodes
            It uses a priority function to assign a score on a scale 0 - 10. For example the schedular will calculate the resources that will be free after it places the pod on them.


### Worker Nodes

They host Applications as containers.
They have

-   Kubelet

    This service runs on every worker nodes. Their job is to manage containers. It will makes sures that the containers are running and are healthy. They also manage the resources on the nodes which they are running on. They are also the only only point of contact between API server on the master and the worker nodes.
    
    Kubelets register the Node with the API server and once it&rsquo;s instucted to do so it will pull the image and create the pods. It will then monitor the node and the pods and report back their health checks to the kube server api (on a timely maner, by default every 5 mins)
    
    One major differece between kubelets and other components is that kubeadm does not install this component, they have to be installed manually.
    
    -   kube-proxy
    -   Container runtime engine


<a id="org6418497"></a>

## Minikube


### TODO Install minikube


### TODO Run minikube

`minikube start`


### Test minikube

mini kube inserts configs into `~/.kube/config`
At the end of the file see

    
    - name: minikube
      user:
        client-certificate: /Users/ebadiana/.minikube/profiles/minikube/client.crt
        client-key: /Users/ebadiana/.minikube/profiles/minikube/client.key

    
    kubectl create deployment hello-node --image=k8s.gcr.io/echoserver:1.4
    kubectl expose deployment hello-minikube --type=NodePort --port=8080
    minikube service hello-node

The last command will open the browser to dispaly config:

    CLIENT VALUES:
    client_address=172.17.0.3
    command=GET
    real path=/
    query=nil
    request_version=1.1
    request_uri=http://127.0.0.1:8080/
    
    SERVER VALUES:
    server_version=nginx: 1.10.0 - lua: 10001
    
    HEADERS RECEIVED:
    accept=text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,*/*;q=0.8
    accept-encoding=gzip, deflate
    accept-language=en-GB,en;q=0.5
    connection=keep-alive
    cookie=PGADMIN_KEY=f521540f-4134-4b44-b185-5073d3ad8ff6; PGADMIN_LANGUAGE=en
    dnt=1
    host=127.0.0.1:65481
    upgrade-insecure-requests=1
    user-agent=Mozilla/5.0 (Macintosh; Intel Mac OS X 10.14; rv:79.0) Gecko/20100101 Firefox/79.0
    BODY:
    -no body in request-


### TODO Stop minikube

`minikube stop`


### TODO Delete

`minikube delete`


<a id="org3d0ad02"></a>

## Helm, packaaging and deploying

Helm is a package manager for kubernetes.


### Charts

Charts is a collection of files that describe a set Kubernetes resources.  A single chart can deploy an app, a piece of software or a database. it can also have dependencies.
Charts use templates that will generate yaml files.


### TODO What is tiller?


### common Helm commands

<table border="2" cellspacing="0" cellpadding="6" rules="groups" frame="hsides">


<colgroup>
<col  class="org-left" />

<col  class="org-left" />
</colgroup>
<thead>
<tr>
<th scope="col" class="org-left">Command</th>
<th scope="col" class="org-left">Description</th>
</tr>
</thead>

<tbody>
<tr>
<td class="org-left"><code>helm init</code></td>
<td class="org-left">Install <b>tiller</b> on the cluster</td>
</tr>


<tr>
<td class="org-left"><code>helm reset</code></td>
<td class="org-left">Remove <b>tiller</b> from cluster</td>
</tr>


<tr>
<td class="org-left"><code>helm install</code></td>
<td class="org-left">Install a helm chart</td>
</tr>


<tr>
<td class="org-left"><code>helm search</code></td>
<td class="org-left">search for a chart</td>
</tr>


<tr>
<td class="org-left"><code>helm list</code></td>
<td class="org-left">list releases</td>
</tr>


<tr>
<td class="org-left"><code>helm upgrade</code></td>
<td class="org-left">upgrade a release</td>
</tr>


<tr>
<td class="org-left"><code>helm rollback</code></td>
<td class="org-left">rollback a release</td>
</tr>


<tr>
<td class="org-left"><code>helm uninstall &lt;pod name&gt; -n &lt;namespace&gt;</code></td>
<td class="org-left">&#xa0;</td>
</tr>
</tbody>
</table>


### TODO Templates

These are Kubernetes mainifest files that describe the resources, which have to be deployed on the cluster.


### TODO `values.yaml`

This file provides default configureation data for the templates in a structured foramt.


<a id="org0d5ae44"></a>

# Cheat sheets

<table border="2" cellspacing="0" cellpadding="6" rules="groups" frame="hsides">


<colgroup>
<col  class="org-left" />

<col  class="org-left" />

<col  class="org-left" />
</colgroup>
<thead>
<tr>
<th scope="col" class="org-left">Tool</th>
<th scope="col" class="org-left">Command</th>
<th scope="col" class="org-left">Description</th>
</tr>
</thead>

<tbody>
<tr>
<td class="org-left">kubectl</td>
<td class="org-left">config get-contetexs</td>
<td class="org-left">Displays all contetexs/clusters</td>
</tr>


<tr>
<td class="org-left">&#xa0;</td>
<td class="org-left">config use context &lt;cluster-name&gt;</td>
<td class="org-left">switches the context/cluster</td>
</tr>


<tr>
<td class="org-left">&#xa0;</td>
<td class="org-left">logs -f -n &lt;namespace&gt; &lt;podname&gt;</td>
<td class="org-left">view logs from pod. -f Tails the logs</td>
</tr>


<tr>
<td class="org-left">&#xa0;</td>
<td class="org-left">exec &#x2013;stdin &#x2013;tty &lt;pod-name&gt; -n &lt;namespace&gt; &#x2013; /bin/bash</td>
<td class="org-left">ssh into running pod</td>
</tr>


<tr>
<td class="org-left">&#xa0;</td>
<td class="org-left">get namespaces</td>
<td class="org-left">Lists all namespaces in the cluster</td>
</tr>


<tr>
<td class="org-left">&#xa0;</td>
<td class="org-left">get pod &lt;pod-id&gt; -o yaml</td>
<td class="org-left">Get a pods yaml ( might have to use -n &lt;namespace&gt; )</td>
</tr>


<tr>
<td class="org-left">&#xa0;</td>
<td class="org-left">get pods -n &lt;namespace&gt;</td>
<td class="org-left">Lists all the pods in the namespace</td>
</tr>


<tr>
<td class="org-left">&#xa0;</td>
<td class="org-left">get pods -n kube-system</td>
<td class="org-left">will return all kubernetes system pods</td>
</tr>
</tbody>

<tbody>
<tr>
<td class="org-left">minikube</td>
<td class="org-left">start / stop /delete</td>
<td class="org-left">&#xa0;</td>
</tr>
</tbody>
</table>


### commands ran for play

kubectl create deployment hello-node &#x2013;image=k8s.gcr.io/echoserver:1.4
kubectl expose deployment hello-minikube &#x2013;type=NodePort &#x2013;port=8080
minikube service hello-node


### TO READ

-   Vagrant

