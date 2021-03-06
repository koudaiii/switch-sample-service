# switch-sample-service
sample application by Consul

### Usage

* download consul 0.5

```
 $ git clone https://github.com/koudaiii/switch-sample-service.git
 $ cd switch-sample-service
 $ wget https://dl.bintray.com/mitchellh/consul/0.5.0_linux_amd64.zip
 $ unzip 0.5.0_linux_amd64.zip
 $ wget https://github.com/hashicorp/consul-template/releases/download/v0.7.0/consul-template_0.7.0_linux_amd64.tar.gz
 $ tar zxvf consul-template_0.7.0_linux_amd64.tar.gz
 $ cp consul-template_0.7.0_linux_amd64/consul-template .
```

* install nginx

```
 $ vagrant up
 $ ssh vagrant@192.168.33.[10 |11 | 12 | 13 ]
 $ sudo su -
 # apt-get -y install nginx
 # sed -i -e "s/nginx/$HOSTNAME/g" /usr/share/nginx/html/index.html
 # service nginx restart
 [LB] # rm /etc/nginx/sites-enabled/default

 # vim /etc/resolv.conf
```

* set up resolv.conf

```
# Dynamic resolv.conf(5) file for glibc resolver(3) generated by resolvconf(8)
#     DO NOT EDIT THIS FILE BY HAND -- YOUR CHANGES WILL BE OVERWRITTEN
#nameserver 10.0.2.3
 nameserver 127.0.0.1
 search lan
```

* start consul

```
root@node#  /vagrant/consul agent -server -dc=local -config-dir=/vagrant/consul.d -bootstrap-expect=2 -data-dir=/tmp/consul -node=node -bind=192.168.33.10 &

root@LB#  /vagrant/consul agent -server -dc=local -config-dir=/vagrant/consul.d -data-dir=/tmp/consul -node=LB -bind=192.168.33.11 -join 192.168.33.10&

root@blue# /vagrant/consul agent -dc=local -config-dir=/vagrant/consul-blue.d -data-dir=/tmp/consul -node=blue -bind=192.168.33.12 -join 192.168.33.10 &

root@green# /vagrant/consul agent -dc=local -config-dir=/vagrant/consul-green.d -data-dir=/tmp/consul -node=green -bind=192.168.33.13 -join 192.168.33.10 &
```

* set event

```
root@LB# /vagrant/consul watch -type=event -name="switch-green" /vagrant/green.service &
root@LB# /vagrant/consul watch -type=event -name="switch-blue" /vagrant/blue.service &
```

* send switch event

```
@all # /vagrant/consul event -name="switch-green"
@all # /vagrant/consul event -name="switch-blue"
```

* jsonを確認できるのに役に立つ

```
# wget http://stedolan.github.io/jq/download/linux64/jq
# chmod +x jq
# mv jq /usr/bin
# curl -s http://127.0.0.1:8500/v1/health/checks/blue | jq .
```

* Use docker

* https://github.com/stlhrt/docker-consul/blob/master/Dockerfile
