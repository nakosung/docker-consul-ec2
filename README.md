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

Create AMI Role to retrieve tag information within ec2-instances
-------------
```
{
  "Version": "2012-10-17",
  "Statement": [
    {    
      "Effect": "Allow",
      "Action": [ "ec2:DescribeTags"],
      "Resource": ["*"]
    }
  ]
}
```

Install docker and consul on each ec2 node (ubuntu 14.04 LTS)
--------------
* Make sure that security group permits inbound TCP/UDP from same sg.
* Let an instance have IAM role created above.
```
#!/bin/bash

apt-get update
apt-get -y install unzip

# Install latest docker
curl -s https://get.docker.io/ubuntu/ | sudo sh

# Install consul
mkdir -p /tmp/consul
pushd /tmp/consul
wget https://dl.bintray.com/mitchellh/consul/0.3.0_linux_amd64.zip
unzip *.zip
mv -f consul /usr/bin
popd
rm -rf /tmp/consul

# Install AWS-CLI
apt-get install python-pip -y
pip install awscli

# ec2_tag (http://stackoverflow.com/questions/3883315/query-ec2-tags-from-within-instance)
cat > /usr/bin/ec2_tag << 'EC2_TAG_END'
#!/bin/sh
TAG_NAME=$1
INSTANCE_ID="`wget -qO- http://instance-data/latest/meta-data/instance-id`"
REGION="`wget -qO- http://instance-data/latest/meta-data/placement/availability-zone | sed -e 's:\([0-9][0-9]*\)[a-z]*\$:\\1:'`"
TAG_VALUE="`aws ec2 describe-tags --filters "Name=resource-id,Values=$INSTANCE_ID" "Name=key,Values=$TAG_NAME" --region $REGION --output=text | cut -f5`"
echo $TAG_VALUE
EC2_TAG_END
chmod +x /usr/bin/ec2_tag

# consul_ip
cat > /usr/bin/consul_ip << 'CONSUL_IP_END'
dig @127.0.0.1 -p 8600 +short $1
CONSUL_IP_END
chmod +x /usr/bin/consul_ip

# consul_port
cat > /usr/bin/consul_port << 'CONSUL_PORT_END'
dig @127.0.0.1 -p 8600 SRV +short $1 | awk '{print $3}'
CONSUL_PORT_END
chmod +x /usr/bin/consul_port
```

```
# For clients
mkdir -p /tmp/consul
mkdir -p /etc/consul.d
mkdir -p /var/consul
cat > /etc/init/consul.conf << 'CONSUL_CONF_END'
start on started tty1
respawn
script
   consul agent -data-dir /var/consul -config-dir /etc/consul.d -join $(ec2_tag CONSUL_JOIN)
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
