# Docker Swarm Tutorial 

> Container Orchestration systems is where the next action is likely to be in the movement towards Building →Shipping → Running containers at scale. The list of software that currently provides a solution for this are Kubernetes, Docker Swarm, Apache Mesos and other.


## Why container orchestration ? 

* Health Checks on the Containers
* Launching a fixed set of Containers for a particular Docker image
* Scaling the number of Containers up and down depending on the load
* Performing rolling update of software across containers

## Running docker in swarm mode 


### Pre-requisite 

* Two VMs - master & slave 

### Installation steps 

* Log in to master and get the ip address. 

```
ip addr
```

* Initialize swarm on master 

```
docker swarm init --advertise-addr MANAGER_IP
```

* Output will be as below on your own screen 

```
To add a worker to this swarm, run the following command:
docker swarm join \
 — token SWMTKN-1–5mgyf6ehuc5pfbmar00njd3oxv8nmjhteejaald3yzbef7osl1-ad7b1k8k3bl3aa3k3q13zivqd \
 192.168.1.8:2377

```

* Login to Slave 

* Copy the output from master and execute that command on the slave machine 

```
docker swarm join \
 — token SWMTKN-1–5mgyf6ehuc5pfbmar00njd3oxv8nmjhteejaald3yzbef7osl1-ad7b1k8k3bl3aa3k3q13zivqd \
 192.168.1.8:2377
```

* You should get the below output

```
This node joined a swarm as a worker.

```

* Verify slave has joined master. Log in to master - 

```
docker node ls 
```

* To get the join command again 

```
docker swarm join-token manager
```

* Verify swarm mode 

```
docker info
```

### Deploying containers to a swarm 

> Now that we have our swarm up and running, it is time to schedule our containers on it. This is the whole beauty of the orchestration layer. We are going to focus on the app and not worry about where the application is going to run. 

> Docker swarm now becomes our primary cluster management endpoint. You only focus on the application and let docker swarm take care of scheduling. 

* Create your first service - nginx 

```
docker service create --replicas 5 -p 80:80 --name web nginx
```

* Output should be as below 

```
overall progress: 5 out of 5 tasks 
1/5: running   [==================================================>] 
2/5: running   [==================================================>] 
3/5: running   [==================================================>] 
4/5: running   [==================================================>] 
5/5: running   [==================================================>] 

```

> The above command creates 5 replicas of nginx and the service will now loadbalance between the 5 endpoints

* Check your service 

```
docker service ls
ID                  NAME                MODE                REPLICAS            IMAGE               PORTS
zrd9t74rze2h        web                 replicated          5/5                 nginx:latest        *:80->80/tcp
```

* Get all processes for your web service 

```
docker service ps web
ID                  NAME                IMAGE               NODE                DESIRED STATE       CURRENT STATE                ERROR               PORTS
bot2nt3gwjn6        web.1               nginx:latest        instance-1          Running             Running about a minute ago                       
vqmgnpo5g4s8        web.2               nginx:latest        instance-2          Running             Running about a minute ago                       
yqcqm7eme97j        web.3               nginx:latest        instance-2          Running             Running about a minute ago                       
fkj194q1xiem        web.4               nginx:latest        instance-1          Running             Running about a minute ago                       
ysorr05csowg        web.5               nginx:latest        instance-2          Running             Running about a minute ago          

```

* Verify processes on each docker node 

```
docker ps
```

### Access the service 

> You can access the service by hitting any of the manager or worker nodes. It does not matter if the particular node does not have a container scheduled on it. That is the whole idea of the swarm.

```
curl master

curl slave
```


### Scaling 

* Scale your service to 7 instances

```
docker service scale web=7
docker service ls
```

### Draining nodes

```
docker service ps web 

docker node ls 

docker node ps master

docker node ps slave

docker node inspect slave 

docker node update --availability drain slave

docker node ls

docker service ps web
```

### Rolling update of service 

```
docker service update --image nginx:latest web
```

### Remove service 

```
docker service rm web
```























