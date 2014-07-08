docker-consul-ec2
=================

awscli
------
* requires pip
```
sudo easy-install pip
sudo pip install awscli
aws configure
```

Install consul on each ec2 node
--------------
* Make sure that security group permits inbound TCP/UDP from same sg.
```
#!/bin/bash

apt-get update
apt-get -y install unzip

# Install latest docker
curl -s https://get.docker.io/ubuntu/ | sudo sh

# Install docker & consul
mkdir -p /tmp/consul
pushd /tmp/consul
wget https://dl.bintray.com/mitchellh/consul/0.3.0_linux_amd64.zip
unzip *.zip
mv -f consul /usr/bin
popd
rm -rf /tmp/consul

mkdir -p /tmp/consul
cat > /etc/init/consul.conf << CONSUL_CONF_END
start on started tty1
respawn
script
   consul agent -data-dir /tmp/consul -join 172.31.27.98
end script
CONSUL_CONF_END

start consul
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
