docker-consul-ec2
=================

awscli
------
* requires pip
```
sudo easy-install pip
sudo pip install awscli
```

Install consul on each ec2 node
--------------
* Make sure that security group permits inbound TCP/UDP from same sg.
```
sudo apt-get install unzip
curl -OL https://dl.bintray.com/mitchellh/consul/0.2.1_linux_amd64.zip
unzip *
chmod +x consul
cp consul /usr/local/bin
```

Consul quorum
--------------------
  - deploy 3 t1 instances
 - for leader node
```
consul agent -server -bootstrap -data-dir=/tmp/consul
```
 - for follower nodes
```
consul agent -server -data-dir=/tmp/consul -join=$(ANY_OTHER_SERVER_IP)
```
 - and then kill leader node, re-start the node as an ordinary follower node.

Working node
-------------------
```
nohup consul agent -data-dir=/tmp/consul -config-dir=/etc/consul.d > /var/log/consul.log 2>&1 &
```
- ```kill -1 $(CONSUL_PID)``` to refresh consul service definitions

Checking nginx health
------------------
```curl -sS -o /dev/null http://localhost:80```

Utilizing docker net-host mode(>=0.11.0)
--------------------------------
```docker run --net=host configuration...```

Consul as DNS
-------------
```
apt-get install dnsmasq
echo "server=/consul/127.0.0.1#8600" > /etc/dnsmasq.d/10-consul
service dnsmasq force-reload
```
