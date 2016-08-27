# Docker swarm test environment

## Requirements

* Install Vagrant

## Running the environment

```bash
vagrant up

# One terminal -- this is the server node that manages the test swarm environment
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

#ID                           HOSTNAME  STATUS  AVAILABILITY  MANAGER STATUS
#9lq0quun52e3x4bpcve0kwjfe *  manager   Ready   Active        Leader

# Swarm manager is now ready

```


```bash
# Second terminal -- this is the client1 node in the swarm
$ vagrant ssh client1
$ sudo -i
```

```bash
# Third terminal -- this is the client2 node in the swarm
$ vagrant ssh client2
$ sudo -i
```
