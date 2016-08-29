# Docker swarm test environment

No bullshit instructions to running Docker swarm with a manager node and two client nodes for testing, learning and just seeing what the fuzz Docker 1.12 features are all about. You can thank me later.

All the examples below are in "just run" format, essentially you just can copy paste away and forget your worries.

## Requirements

* [Install Vagrant](https://www.vagrantup.com/docs/installation/)

## Running the environment

Open three terminal windows as it will make controlling the environment a breeze.

### Setup environment

#### Terminal 1
```bash
$ vagrant up

# Server/Manager:
# First terminal -- this is the server node that manages the test swarm environment
$ vagrant ssh server
$ sudo -i

# Create swarm manager and tell it to advertise on this specific IP
# Nodes need to be able to connect to this IP -- This IP for this environment is set in Vagrantfile
$ docker swarm init --advertise-addr 10.0.15.30

# Swarm initialized: current node (9lq0quun52e3x4bpcve0kwjfe) is now a manager.
# To add a worker to this swarm, run the following command:

#    docker swarm join \
#    --token SWMTKN-1-2cria3pj176nydbj6cdapf2bi7ity8ogvhs20ynkaek0bh78aq-1eei6k8nhmdouyki0qz1gw0bs \
#    10.0.15.30:2377

# To add a manager to this swarm, run 'docker swarm join-token manager' and follow the instructions.

# View information about nodes connected
$ docker node ls

# ID                           HOSTNAME  STATUS  AVAILABILITY  MANAGER STATUS
# 9lq0quun52e3x4bpcve0kwjfe *  manager   Ready   Active        Leader

# Swarm manager is now ready. Connect clients (nodes) into the manager.

```

#### Terminal 2

```bash
# client1:
$ vagrant ssh client1
$ sudo -i
# NOTE copy the --toke from your own Terminal 1 as it'll be a bit different from this example
$ docker swarm join --token SWMTKN-1-2cria3pj176nydbj6cdapf2bi7ity8ogvhs20ynkaek0bh78aq-1eei6k8nhmdouyki0qz1gw0bs 10.0.15.30:2377
```

#### Terminal 3

```bash
# client2:
$ vagrant ssh client2
$ sudo -i
# NOTE copy the --toke from your own Terminal 1 as it'll be a bit different from this example
$ docker swarm join --token SWMTKN-1-2cria3pj176nydbj6cdapf2bi7ity8ogvhs20ynkaek0bh78aq-1eei6k8nhmdouyki0qz1gw0bs 10.0.15.30:2377
```

#### Terminal 1

```bash
# Terminal: Server/Manager
$ docker node ls

# ID                           HOSTNAME       STATUS  AVAILABILITY  MANAGER STATUS
# 2p85633kx19q0quuobm8dstoa    swarm-client2  Ready   Active
# 6ogb4iz9zhsr9iea9p3m5top5    swarm-client1  Ready   Active
# 9lq0quun52e3x4bpcve0kwjfe *  manager        Ready   Active        Leader
```

That's it, you now have a docker swarm with one manager node and two clients connected to it. You drive the environment through the manager and the nodes follow the orchestration. The world is your oyster.

## What next?

You're essentially set for a fully configured environment and you can start learning the docker swarm features or test out your deployments the way you want before you decide to take everything to production. Go ahead, play with it! Fiddle it like no tomorrow.

## No, really, what next?

Alright, if you feel like adventuring with me, keep following and we'll look into other features of Docker Swarm.

## Services

This is the heard and soul of Docker swarm, services are made out of N containers spread over N nodes based on requirements and configuration.

### Deploy service to the swarm

#### Terminal 1

```bash
# Note: --replicas describe the desired state of running the instances (how many)
$ docker service create --replicas 1 --name helloworld alpine ping docker.com
# 4ynua3ewbg4ff74ftlf9b3ami

# List the services to see what's going on
$ docker service ls
# ID            NAME        REPLICAS  IMAGE   COMMAND
# 4ynua3ewbg4f  helloworld  1/1       alpine  ping docker.com
```

### Inspecting the service

#### Terminal 1

```bash
$ docker service inspect --pretty helloworld
# Note the pretty flag, if it's not used, you will get a raw json output - super helpful for cli / scripts.

# ID:				4ynua3ewbg4ff74ftlf9b3ami
# Name:				helloworld
# Mode:				Replicated
#  Replicas:		1
# Placement:
# UpdateConfig:
#  Parallelism:		1
#  On failure:		pause
# ContainerSpec:
#  Image:			alpine
#  Args:			ping docker.com
# Resources:

# NOTE after running the below command, there is a change that your service is runnin on some other node than the master, you'll need to ssh to that node if you want to tinker the specific container

$ docker service ps helloworld
# ID                         NAME          IMAGE   NODE     DESIRED STATE  CURRENT STATE           ERROR
# e3jezu4e2ek9fg9egc3e0piz7  helloworld.1  alpine  manager  Running        Running 26 minutes ago

# Open the correct terminal depending on what NODE it is running on - in this example it's on manager
$ docker ps
# CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS               NAMES
# 52c4e3b1041f        alpine:latest       "ping docker.com"   30 minutes ago      Up 30 minutes                           helloworld.1.e3jezu4e2ek9fg9egc3e0piz7
```

### Scaling services

#### Terminal 1

```bash
$ docker service ls
# ID            NAME        REPLICAS  IMAGE   COMMAND
# 4ynua3ewbg4f  helloworld  1/1       alpine  ping docker.com

$ docker service scale helloworld=5 # NOTE you can also use the ID if you want, I prefer name
# helloworld scaled to 5

$ docker service ps helloworld
# ID                         NAME              IMAGE   NODE           DESIRED STATE  CURRENT STATE            ERROR
# 1lz0jvkqjkd3sfxd67g1tgd0d  helloworld.1      alpine  manager        Running        Running 24 seconds ago
# 8jf1g3foszgz9qk97tncmm6mu  helloworld.2      alpine  swarm-client2  Running        Running 25 seconds ago
# 2cz8571hs0bz8luz42ngg8jin  helloworld.3      alpine  swarm-client1  Running        Running 50 seconds ago
# 4bvxuppbaw4l292sgvqd697xm  helloworld.4      alpine  swarm-client2  Running        Running 25 seconds ago
# 3myydfyduut9mgelf9zgncxkl  helloworld.5      alpine  swarm-client1  Running        Running 25 seconds ago

# To see the containers of this service running on this node used
$ docker ps
# CONTAINER ID        IMAGE               COMMAND             CREATED              STATUS              PORTS               NAMES
# 63d9242100ba        alpine:latest       "ping docker.com"   About a minute ago   Up About a minute                       helloworld.1.1lz0jvkqjkd3sfxd67g1tgd0d
```

### Deleting services

#### Terminal 1

```bash
$ docker service rm helloworld
# helloworld
$ docker service ls
# ID  NAME  REPLICAS  IMAGE  COMMAND
$ docker service inspect helloworld
# []
# Error: no such service: helloworld
```

### Apply rolling updates

Update configuration (UpdateConfig) for a swarm describes how the services should be updated; order, parallelism and such.

#### Terminal 1

Deploying `redis` server and later updating the services and making sure the service is up the whole time when we roll the updates in one at a time.

```bash
$ docker service create --replicas 3 --name redis --update-delay 10s redis:3.0.6
# e7ikenqcvvd1qmw21d5bu02v5

$ docker service inspect --pretty redis
# ID:				e7ikenqcvvd1qmw21d5bu02v5
# Name:				redis
# Mode:				Replicated
#  Replicas:		3
# Placement:
# UpdateConfig:
#  Parallelism:		1
#  Delay:			10s
#  On failure:		pause
# ContainerSpec:
#  Image:			redis:3.0.6
# Resources:

# Running docker service update is bound by UpdateConfig above and updates the services the way UpdateConfig describes
$ docker service update --image redis:3.0.7 redis
# redis

# Let's inspect the service when it's updating
$ docker service inspect --pretty redis
# ID:		e7ikenqcvvd1qmw21d5bu02v5
# Name:		redis
# Mode:		Replicated
#  Replicas:	3
# Update status:
#  State:		updating
#  Started:	about a minute ago
#  Message:	update in progress
# Placement:
# UpdateConfig:
#  Parallelism:	1
#  Delay:		10s
#  On failure:	pause
# ContainerSpec:
#  Image:		redis:3.0.7
# Resources:

$ docker service ls
# ID            NAME   REPLICAS  IMAGE        COMMAND
# e7ikenqcvvd1  redis  2/3       redis:3.0.7

# There is always a possibility that an update can fail, you can see the failure with the inspect command and you can easily enough restart the service update and try again with
$ docker service update redis

# To avoid repeating update failures, you can reconfigure the service with docker service flags.

# Using the command service ps you can view the update process in a bit more detailed manner and see where it's going
$ docker service ps redis
cvyeh5qru4l4om5iuf7bdez80  redis.1      redis:3.0.7  swarm-client1  Running        Running 4 minutes ago
3wnysnxm0t6af1oiwjbzpuhzk   \_ redis.1  redis:3.0.6  manager        Shutdown       Shutdown 5 minutes ago
889pyle30zr35eymskdqw9ice  redis.2      redis:3.0.7  manager        Running        Running 3 minutes ago
c64vb6hscrz9f9fg2toq0tl0g   \_ redis.2  redis:3.0.6  swarm-client2  Shutdown       Shutdown 4 minutes ago
0vdzc1qp9t4zlp4gxsq67y09f  redis.3      redis:3.0.7  swarm-client2  Running        Running 25 seconds ago
sefe2343243f983f9fu39f38f   \_ redis.2  redis:3.0.6  swarm-client2  Shutdown       Shutdown 4 minutes ago

# Before swarm updates all of the tasks, you can see that some are running the older version and some the never one. Example above shows that all have updated.
```

### Drain a node

For example for planned maintenance times, you need to drain a node of services so it can be safely booted. `DRAIN` availability prevents a node from receiving new tasks from the swarm manager. It also means the manager stops tasks running on the node and launches replica tasks on a node with `ACTIVE` availability.

#### Terminal 1

```bash
$ docker node ls
# ID                           HOSTNAME       STATUS  AVAILABILITY  MANAGER STATUS
# 2p85633kx19q0quuobm8dstoa    swarm-client2  Ready   Active
# 6ogb4iz9zhsr9iea9p3m5top5    swarm-client1  Ready   Active
# 9lq0quun52e3x4bpcve0kwjfe *  manager        Ready   Active        Leader

# Let's do a bit of cleaning for the sake of this README.md.
# We just remove the redis service and recreate so we can play around with it even further
$ docker service rm redis
$ docker service create --replicas 3 --name redis --update-delay 10s redis:3.0.6

# See how swarm manages the service across all the nodes
$ docker service ps redis
# ID                         NAME     IMAGE        NODE           DESIRED STATE  CURRENT STATE                   ERROR
# 2xzjzugarko48ii9nrfrdy2ac  redis.1  redis:3.0.6  swarm-client2  Running        Running 1 seconds ago
# 5e3x61ekdjdabcbpg2zsuy9nu  redis.2  redis:3.0.6  swarm-client1  Running        Running less than a second ago
# 2fqxio89ch5rskc1v4ro3zazr  redis.3  redis:3.0.6  manager        Running        Preparing 4 seconds ago

# Pick a node you want to drain, we'll go with swarm-client2 - notice that on your setup swarm might have decided to distribute the service differently and this is completely normal.

$ docker node update --availability drain swarm-client2
# swarm-client2
# Check the node for availability
$ docker node inspect --pretty swarm-client2
# ID:			    	2p85633kx19q0quuobm8dstoa
#  Hostname:			swarm-client2
# Joined at:			2016-08-27 09:03:59.128142932 +0000 utc
# Status:
#  State:				Ready
#  Availability:		Drain
# Platform:
#  Operating System:	linux
#  Architecture:		x86_64
# Resources:
#  CPUs:				1
#  Memory:				238 MiB
# Plugins:
#   Network:			bridge, host, null, overlay
#   Volume:				local
# Engine Version:		1.12.1

# Node shows availability as Drain. We know it's working the way it should after the drain command.
# Run service ps to see how swarm manager redistributed the containers for the service to keep it runnin as smooth as possible.
$ docker service ps redis
# ID                         NAME          IMAGE        NODE      DESIRED STATE  CURRENT STATE           ERROR
# 7q92v0nr1hcgts2amcjyqg3pq  redis.1       redis:3.0.6  manager1  Running        Running 4 minutes
# b4hovzed7id8irg1to42egue8  redis.2       redis:3.0.6  worker2   Running        Running About a minute
# 7h2l8h3q3wqy5f66hlv9ddmi6   \_ redis.2   redis:3.0.6  worker1   Shutdown       Shutdown 2 minutes ago
# 9bg7cezvedmkgg6c8yzvbhwsd  redis.3       redis:3.0.6  worker2   Running        Running 4 minutes

# After you have run all the updates, planned maintenance tasks and such for the swarm-client2, we bring it back online with:
$ docker node update --availability active swarm-client2
# Running the inspect and service ps again you can see the node is in active availability state and the swarm manager starts using it again. Easy, no downtime updates are a go!

```

# Further reading

The start essentially is a concatenated fast way of getting up and running with swarm from Dockers own documentation with all the virtual machine configurations and that included.

At this point you've learned to manage the swarm and you want to look into further details as needed of the swarm api and swarm features themselves. They are really well documented at Docker docs.
