---
layout: post
title:  "Swarm Cluster on ARM SBCs"
date:   2025-05-11 19:49:28 -0300
categories: jekyll update
image: /assets/managernode.png
---


Now based on this great post: https://www.docker.com/blog/getting-started-with-docker-for-arm-on-linux/, we are going to install Docker on the 4 devices. That can be done by following that post, or the package manager of the distro already has the Docker package, or you can use, in the case of Armbian, the config tool called armbian-config. After that, use the command:

sudo usermod -aG docker username

a means add, G means group — add the user to the group docker. You may need to log out and back in to take effect.

Now the Docker versions listing for now:

    Banana Pi: 27.1.1

    Raspberry Pi W: 20.10.24+dfsg1

    Raspberry Pi 3B: 20.10.24+dfsg1

    Orange Pi: 28.0.4

Now let’s see how to create the actual cluster and then use it with Docker Swarm. A good reference is: https://docs.docker.com/engine/swarm/

It goes into detail about redundancy, number of management nodes, number of worker nodes, load balancing, task reassigning. Since the setup is literally a bunch of USB-connected devices to the same power supply, none of the redundancy talk matters much — if the power supply goes down, everything goes down. But in other scenarios, that's a very special thing to think about. No-breaks (UPS) can sometimes be a headache — either it's not properly set up, the batteries are bad, or there's some kind of circuit problem… but anyway, jumping jumping jumping…

When setting up the 3 workers and 1 manager, they all need static IP addresses. That can be done either by setting a static IP address on the computer or setting that up in the router, which in my case was done quickly in OPNsense's DHCP server. After that, verify if the ports are open. They are:

    Port 2377/TCP for communication with and between manager nodes

    Port 7946/TCP/UDP for overlay network node discovery

    Port 4789/UDP (configurable) for overlay network traffic

Commands to open all ports with iptables:

iptables -A INPUT -p tcp --dport 2377 -j ACCEPT
iptables -A INPUT -p tcp --dport 7946 -j ACCEPT
iptables -A INPUT -p udp --dport 7946 -j ACCEPT
iptables -A INPUT -p udp --dport 4789 -j ACCEPT

What does each option mean?
-A appends a rule to the INPUT chain, -p specifies the protocol (TCP or UDP), --dport is the destination port, and -j means the action — it jumps to ACCEPT if matched.

For more details: https://www.linux.co.cr/distributions/review/2002/red-hat-8.0/rhl-rg/s1-iptables-options.html
connection refused can’t connect — iptables?

To start the manager node:

docker swarm init --advertise-addr manager_ip_address

After that, run:

docker swarm join --token SWMTKN-1-4tkqpfwvmidz3ip3eibkkf03fhynhtsu4kky1iwuzl74kjwdoy-b3vz3te6txomfh7rkr52gjzvr 192.168.0.39:2377

Well, probably better to make a script to pass this command through all the hosts. Sending that command can be done in Bash using a list of IPs, or in a more sophisticated way using Ansible. For this setup, Bash is better since I don’t want to use vaults and the more complex setup of Ansible — especially because I’d have to use a different combination of user and password for each node (so more config to go on each of the nodes).

A simple Bash script:

```bash
#!/bin/bash
file=ips.txt
for i in $(cat $file)
do
  ssh user@$i "docker swarm join --token SWMTKN-1-4tkqpfwvmidz3ip3eibkkf03fhynhtsu4kky1iwuzl74kjwdoy-b3vz3te6txomfh7rkr52gjzvr 192.168.0.39:2377"
done
```


After that we have:

ID                          HOSTNAME        STATUS   AVAILABILITY  MANAGER STATUS  ENGINE VERSION
0lqoz9l31vr49qn2ucf2nqmra   bananapim2zero  Ready    Active                      27.1.1  
ou2gmpss8t2c1yn8c7l9d1h80   clusterZero0    Ready    Active                      20.10.24+dfsg1  
v3ujlsa2nac5b7ptzh5lcqo9s   orangepizero2w  Ready    Active       Leader         28.0.4  
kzq1fxgl7yo6lgbqdunk7dym7   pi3b            Ready    Active                      20.10.24+dfsg1  

Now let’s try to deploy a simple test service…  
The test service that Docker suggests is:  

docker service create --replicas 1 --name helloworld alpine ping docker.com

Well, that worked:

REPLICAS  IMAGE         PORTS  
1/1       alpine:latest  
Better try to inspect that. Need to see where that is, etc.  
docker service inspect --pretty helloworld  
Inspecting does not give us the location, only details about the service itself:  
ID:             gj257idwkgshjn41qcrzyrdwr  
Name:           helloworld  
Service Mode:   Replicated  
Replicas:       1  
UpdateConfig:  
  Parallelism:           1  
  On failure:            pause  
  Monitoring Period:     5s  
  Max failure ratio:     0  
  Update order:          stop-first  
RollbackConfig:  
  Parallelism:           1  
  On failure:            pause  
  Monitoring Period:     5s  
  Max failure ratio:     0  
  Rollback order:        stop-first  
ContainerSpec:  
  Image:         alpine:latest@sha256:a8560b36e8b8210634f77d9f7f9efd7ffa463e380b75e2e74aff4511df3ef88c  
  Args:          ping docker.com  
  Init:          false  
Resources:  
Endpoint Mode:   vip  

Now to the location:

docker service ps helloworld  
Now we have it:  
t7w0atmqnpqo  helloworld.1  alpine:latest  orangepizero2w  Running  10 minutes ago  
Well, let’s try more replicas… HEHEHEH  
docker service create --replicas 4 --name helloworld2 alpine ping docker.com  

Results in one replica for each:  
ID            NAME              IMAGE          NODE             DESIRED STATE  CURRENT STATE  
llen0o8vnd0a  helloworld2.1     alpine:latest  clusterZero0     Running        35 seconds ago  
yrcbn4p6e60q  helloworld2.2     alpine:latest  orangepizero2w   Running        58 seconds ago  
lh48dbcvfhru  helloworld2.3     alpine:latest  bananapim2zero   Running        44 seconds ago  
9582ztkzdqsn  helloworld2.4     alpine:latest  pi3b             Running        48 seconds ago  

Well, I can also alter the scale of it — didn't know that. Cool stuff.

docker service scale helloworld=5

Five replicas with only four nodes results in… normal stuff — it just duplicates one:

t7w0atmqnpqo  helloworld.1  alpine:latest  orangepizero2w  Running  17 minutes ago  
al97od3mn5q0  helloworld.2  alpine:latest  clusterZero0    Running  10 seconds ago  
6q90nb3wyg8q  helloworld.3  alpine:latest  clusterZero0    Running  10 seconds ago  
uz8nlmefc4el  helloworld.4  alpine:latest  pi3b            Running  20 seconds ago  
mev0hauj66pr  helloworld.5  alpine:latest  bananapim2zero  Running  20 seconds ago  

Well, now let’s delete all that :)
docker service ps helloworld  
docker service ps helloworld2  

Well, that seems good for now. Next time: going over updates, draining a node (meaning putting it into maintenance), and then trying some of my services to do a stress test and load balancing.
