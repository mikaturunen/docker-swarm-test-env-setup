# Docker swarm test environment

## Requirements

* [Install Vagrant](https://www.vagrantup.com/docs/installation/)

## Running the environment

### Terminal 1
```bash
vagrant up

# Server/Manager:
# First terminal -- this is the server node that manages the test swarm environment
$ vagrant ssh server
$ sudo -i

# Create swarm manager and tell it to advertise on this specific IP
# Nodes need to be able to connect to this IP -- This IP for this environment is set in Vagrantfile
$ docker swarm init --advertise-addr 10.0.15.30

#Swarm initialized: current node (9lq0quun52e3x4bpcve0kwjfe) is now a manager.
#To add a worker to this swarm, run the following command:

#    docker swarm join \
#    --token SWMTKN-1-2cria3pj176nydbj6cdapf2bi7ity8ogvhs20ynkaek0bh78aq-1eei6k8nhmdouyki0qz1gw0bs \
#    10.0.15.30:2377

#To add a manager to this swarm, run 'docker swarm join-token manager' and follow the instructions.

# View information about nodes connected
$ docker node ls

# ID                           HOSTNAME  STATUS  AVAILABILITY  MANAGER STATUS
# 9lq0quun52e3x4bpcve0kwjfe *  manager   Ready   Active        Leader

# Swarm manager is now ready. Connect clients (nodes) into the manager.

```

### Terminal 2

```bash
# client1:
$ vagrant ssh client1
$ sudo -i
$ docker swarm join --token SWMTKN-1-2cria3pj176nydbj6cdapf2bi7ity8ogvhs20ynkaek0bh78aq-1eei6k8nhmdouyki0qz1gw0bs 10.0.15.30:2377
```

### Terminal 3

```bash
# client2:
$ vagrant ssh client2
$ sudo -i
$ docker swarm join --token SWMTKN-1-2cria3pj176nydbj6cdapf2bi7ity8ogvhs20ynkaek0bh78aq-1eei6k8nhmdouyki0qz1gw0bs 10.0.15.30:2377
```

### Terminal 1

```bash
# Terminal: Server/Manager
$ docker node ls

# ID                           HOSTNAME       STATUS  AVAILABILITY  MANAGER STATUS
# 2p85633kx19q0quuobm8dstoa    swarm-client2  Ready   Active
# 6ogb4iz9zhsr9iea9p3m5top5    swarm-client1  Ready   Active
# 9lq0quun52e3x4bpcve0kwjfe *  manager        Ready   Active        Leader
```

That's it, you now have a docker swarm with one manager node and two clients connected to it. You drive the environment through the manager and the nodes follow the orchestration. The world if your ouster.
