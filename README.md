docker-consul-ec2
=================

consul quorum
--------------------
  - deploy 3 t1 instances
```
sudo apt-get install unzip
curl -OL https://dl.bintray.com/mitchellh/consul/0.2.1_linux_amd64.zip
unzip *
chmod +x consul
cp consul /usr/local/bin
```
 - for leader node
```
consul agent -server -bootstrap -data-dir=/tmp/consul
```
 - for follower nodes
```
consul agent -server -data-dir=/tmp/consul -join=$(ANY_OTHER_SERVER_IP)
```
 - and then kill leader node, re-start the node as an ordinary follower node.
