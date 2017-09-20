# Docker Cheat Sheet

## Basic concepts

An **image** is a lightweight, stand-alone, executable package that includes everything needed to run a piece of software, including the code, a runtime, libraries, environment variables, and config files.

A **container** is a runtime instance of an image—what the image becomes in memory when actually executed. It runs completely isolated from the host environment by default, only accessing host files and ports if configured to do so.

### Containers

* Create Docker image with tag:

    `docker build -t friendlyhello .`
    
* List docker images:
    
    `docker images`
    
    ```
    REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
    friendlyhello       latest              16bdb5cba626        56 seconds ago      195MB
    python              2.7-slim            426d65ab9a72        5 days ago          183MB
    hello-world         latest              05a3bd381fc2        7 days ago          1.84kB
    ```
    
* Run an application:
    
    `docker run -p 4000:80 friendlyhello`
    
    `-d` detached mode(background)
    
    `-p` specify port preferences
    
### Sharing images

* Login:

    `docker login`
    
* Tag the image:

    `docker tag 16bdb5cba626 arturjoshi/get-started:part1`
    
* List of images:

    `docker images`
    
    ```
    REPOSITORY               TAG                 IMAGE ID            CREATED             SIZE
    arturjoshi/get-started   part1               16bdb5cba626        7 minutes ago       195MB
    friendlyhello            latest              16bdb5cba626        7 minutes ago       195MB
    python                   2.7-slim            426d65ab9a72        5 days ago          183MB
    hello-world              latest              05a3bd381fc2        7 days ago          1.84kB
    ```
    
* Publish an image:

    `docker push arturjoshi/get-started:part1`
    
* Remove image:

    `docker rmi 16bdb5cba626 -f`
    
* Run application from remote repository:

    `docker run -p 4000:80 arturjoshi/get-started:part1`
    
### Services

Services are really just “containers in production.” A service only runs one image, but it codifies the way that image runs—what ports it should use, how many replicas of the container should run so the service has the capacity it needs, and so on. Scaling a service changes the number of container instances running that piece of software, assigning more computing resources to the service in the process.

* Get list of services:

    `docker service ls`
    
    ```
    ID                  NAME                MODE                REPLICAS            IMAGE                          PORTS
    3dk1n2jc2o5o        getstartedlab_web   replicated          5/5                 arturjoshi/get-started:part1   *:80->80/tcp
    ```

* Get list of tasks:
 
    `docker service ps 3dk1n2jc2o5o`
     
     ```
     ID                  NAME                  IMAGE                          NODE                DESIRED STATE       CURRENT STATE           ERROR               PORTS
     5ih5def6pj5c        getstartedlab_web.1   arturjoshi/get-started:part1   ajoshi-ubnt         Running             Running 6 minutes ago                       
     q5oct19o3x5s        getstartedlab_web.2   arturjoshi/get-started:part1   ajoshi-ubnt         Running             Running 6 minutes ago                       
     yqnr9fhlsjgq        getstartedlab_web.3   arturjoshi/get-started:part1   ajoshi-ubnt         Running             Running 6 minutes ago                       
     dd8pq4a9ba0h        getstartedlab_web.4   arturjoshi/get-started:part1   ajoshi-ubnt         Running             Running 6 minutes ago                       
     z19g7dge1mtn        getstartedlab_web.5   arturjoshi/get-started:part1   ajoshi-ubnt         Running             Running 6 minutes ago                       
     ```
     
### Swarms

Multi-container, multi-machine applications are made possible by joining multiple machines into a “Dockerized” cluster called a swarm.

A swarm is a group of machines that are running Docker and joined into a cluster. After that has happened, you continue to run the Docker commands you’re used to, but now they are executed on a cluster by a swarm manager. The machines in a swarm can be physical or virtual. After joining a swarm, they are referred to as nodes.

Swarm managers can use several strategies to run containers, such as “emptiest node” – which fills the least utilized machines with containers. Or “global”, which ensures that each machine gets exactly one instance of the specified container. You instruct the swarm manager to use these strategies in the Compose file, just like the one you have already been using.

Swarm managers are the only machines in a swarm that can execute your commands, or authorize other machines to join the swarm as workers. Workers are just there to provide capacity and do not have the authority to tell any other machine what it can and cannot do.

Up until now, you have been using Docker in a single-host mode on your local machine. But Docker also can be switched into swarm mode, and that’s what enables the use of swarms. Enabling swarm mode instantly makes the current machine a swarm manager. From then on, Docker will run the commands you execute on the swarm you’re managing, rather than just on the current machine.

* Initialize swarm:

    `docker swarm init`
    
* Leave swarm:

    `docker swarm leave --force`
    
* Create docker-machine via VM driver:

    `docker-machine create --driver virtualbox myvm1`
    
* Initialize manager on node:

    `docker-machine ssh myvm1 "docker swarm init --advertise-addr <myvm1 ip>"`
    
* Add node as worker:

    `docker-machine ssh myvm2 " docker swarm join --token SWMTKN-1-5padthsogzzsfktb8xemboe1deyh3wrvduswih5ttypg7x0i0x-4pfz5fe2evetgd9v8da35q9i2 192.168.99.100:2377"`

* Get list of nodes:

    `docker node ls`
    
    ```
    ID                            HOSTNAME            STATUS              AVAILABILITY        MANAGER STATUS
    1gjn72hqn8qrbktf9f3c7kpug     myvm2               Ready               Active              
    zgf5upp2u7grg2eu34nxze4tu *   myvm1               Ready               Active              Leader
    ```

* Copy file on node:

    `docker-machine scp docker-compose.yml myvm1:~`
    
### Stacks

A stack is a group of interrelated services that share dependencies, and can be orchestrated and scaled together. A single stack is capable of defining and coordinating the functionality of an entire application (though very complex applications may want to use multiple stacks).
 
* Deploy app in a stack:
    
    `docker stack deploy -c docker-compose.yml getstartedlab`
    
* Take down app:

    `docker stack rm getstartedlab`
 