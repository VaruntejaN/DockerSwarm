# Getting started with swarm mode - Docker
The tutorial guides you through the following activities:

* initializing a cluster of Docker Engines in swarm mode
* adding nodes to the swarm
* deploying application services to the swarm
* managing the swarm once you have everything running

## Initializing a cluster of Docker Engines in swarm mode in Windows 
### Prerequisites:
Docker installed on the system.\
Open HyperV Mananger and create a Virtual Switch using Virtual Switch Mananger with type "external".

### Create Docker Machines:
The first step is to create a set of Docker machines that will act as nodes in our Docker Swarm.\
Run the Gitbash as Administrator and use the standard command to create a Docker Machine named manager1.

* docker-machine create -d hyperv --hyperv-virtual-switch "Virtual Switch Name" machine_name

![creating machines](https://user-images.githubusercontent.com/91135247/210503990-d1090b1f-c1b2-4404-9db5-4795da8d5692.png)

Similarly, create the other worker nodes.\
After creating, fire the following command to check on the status of all the Docker machines.

* docker-machine ls

![machine status](https://user-images.githubusercontent.com/91135247/210504735-3bd31835-426c-4e46-85c6-28321c70e8cf.png)

Note down the IP Address of the manager1. One way to get the IP address of the manager1 machine is
* docker-machine ip manager1\
10.9.6.13

## Adding nodes to the swarm
Now our machines are setup, we can proceed with setting up the Swarm.\
The first thing to do is initialize the Swarm. We will SSH into the manager1 machine and initialize the swarm in there.
* docker-machine ssh manager1

This will initialize the SSH session.\
Perform the following steps:
* docker swarm init --advertise-addr MANAGER_IP

![swarm initialization](https://user-images.githubusercontent.com/91135247/210517371-bf76dc7b-8ab7-4b71-a981-e41d99bfb17b.png)

Note down the command from the output to join other nodes as workers.

## Adding worker nodes to our Swarm
Now we can do a SSH into each of the worker Docker machines and then fire the respective join command in them.\
SSH into the worker1 machine. Then fire the respective command got for joining as a worker.
*  docker swarm join --token SWMTKN-1-48imud4fdm2ads248daun4ld2lmefbxnngrq67swis6coqe9nm-eh4uyylvja64hxii4cf8nsfeh 10.9.6.13:2377

![worker nodes](https://user-images.githubusercontent.com/91135247/210519266-5b7e9645-f500-418d-a56d-3d46c31c820a.png)

Do the same thing by launching SSH sessions for worker2/3 and then pasting the same command.\
Go back to manager1 SSH session and fire the following command to check on the status of Swarm i.e. see the nodes participating in it:
* docker node ls

![node status](https://user-images.githubusercontent.com/91135247/210520298-cb305f6e-124d-4e09-a832-1a616fb61fc8.png)

You can see there are 4 nodes, one as the manager (manager1) and the other 3 as workers.\
Also execute the "docker info" command here to check out the details for Swarm.

![swarm info](https://user-images.githubusercontent.com/91135247/210521902-48f32553-3de4-4af5-8dc8-01a3cfe5c408.png)

## Creating a Service
Now we tell the manager to run the containers for us and it will take care of scheduling out the containers, sending the commands to the nodes and distributing it.\
To do that, I am again in the SSH session for my manager1 node. And give the following command:

* docker service create --replicas 4 -p 3000:3000 --name service_name image_name

![service](https://user-images.githubusercontent.com/91135247/210529575-ef35a922-0013-44ad-a324-99115bde5903.png)
Here I launched 4 replicas of a simple frontend application.

You can find out the status of the service, by giving the following command:

* docker service ls

![service status](https://user-images.githubusercontent.com/91135247/210530007-d5329287-0e87-4f26-9947-5a600d0756ed.png)

You can also see how it is getting orchestrated to the different nodes by using the following command:

* docker service ps service_name

![service orchestration](https://user-images.githubusercontent.com/91135247/210530224-7c6d09e1-747d-41a4-983b-e450ce04643c.png)

## Accessing the service
You can access the service by hitting any of the manager or worker nodes.\
Hit the URL (http://machine-ip) in the browser. You should be able to get a simple frontend application.

![Sample Application](https://user-images.githubusercontent.com/91135247/210532259-aafb0385-459e-44d6-b53b-79b32b28cdac.png)

## Scaling up
We currently have 4 containers running. Let us scale it up to 8 by executing the command on the manager1 node.

* docker service scale service_name=8

![scale](https://user-images.githubusercontent.com/91135247/210533710-af7bb2ec-db5f-44ce-899c-1dbcecb96921.png)

## Draining a node
If the node is ACTIVE, it is ready to accept tasks from the Manager.\
Now, let us set the Availability to DRAIN so that Manager will stop tasks running on that node and launches the replicas on other nodes with ACTIVE availability.

* docker node update --availability drain node_name

![drain](https://user-images.githubusercontent.com/91135247/210537203-41f50730-fb8a-4fbf-94c3-d8018ce634dc.png)

## Worker node goes down
If any worker node goes down, then Manager launches the replicas running on that node on other nodes.

![worker down](https://user-images.githubusercontent.com/91135247/210538444-d6044908-6605-4dd7-b01a-97791aef30b5.png)

## Promoting Worker to Manager
Currently we have 4 nodes. Let's add 2 more worker nodes to our swarm.\
We can promote workers to become Managers by using the command
* docker node promote worker_node

![promote worker](https://user-images.githubusercontent.com/91135247/210542912-431d8102-dbb2-4a7a-9cbe-b3d33264ddd2.png)

We can see there are 3 managers where 1 is leader and other 2 are "reachable".

## Manager node goes down

To take advantage of swarm mode’s fault-tolerance features, Docker recommends you implement an odd number of nodes so that you can recover from the failure of a manager node without downtime.
* A three-manager swarm tolerates a maximum loss of one manager.
* A five-manager swarm tolerates a maximum simultaneous loss of two manager nodes.
* An odd number N of manager nodes in the cluster tolerates the loss of at most 
(N-1)/2 managers. Docker recommends a maximum of seven manager nodes for a swarm.

We have a three-manager swarm, so if Manager1 node goes down, then any one of the remaining two managers become leader and starts handling the cluster.

![manager down](https://user-images.githubusercontent.com/91135247/210760595-eb7ec9b6-114f-4852-b4d6-99fb47658c06.png)

![checking status after manager went down](https://user-images.githubusercontent.com/91135247/210761010-e195e77b-8c48-44b5-ad6c-55baef1a62f0.png)

NOTE: In a swarm cluster, more than 50% of the managers should be online to manage the cluster.

In our case if two of the managers goes down simultaneously, then our swarm has less than 50% of the managers online, so we lose the control over the cluster and the cluster run into a disaster.

![swarm disaster](https://user-images.githubusercontent.com/91135247/210764002-a009751c-939f-4e04-bc13-2e773b676485.png)

## Restore from disaster
SSH into the last active manager node and run the command:

* docker swarm init --force-new-cluster

![restore from disaster](https://user-images.githubusercontent.com/91135247/210774749-dc7695fa-f5f4-4878-b92d-be87083d6225.png)

This re-initializes the cluster with last active manager as leader and starts handling the cluster.

![new cluster](https://user-images.githubusercontent.com/91135247/210775015-44db9bf6-33f6-4014-810b-c91ea3057aa0.png) 

## References

https://docker-docs.netlify.app/machine/drivers/hyper-v/

https://docker-docs.netlify.app/machine/drivers/hyper-v/#example

https://rominirani.com/docker-swarm-tutorial-b67470cf8872
