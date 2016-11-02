# Service Discovery Labs

## Set up etcd manually

vagrant up cd serv-disc-01 --provision

vagrant up  serv-disc-01 --provision

vagrant ssh serv-disc-01

curl -L https://github.com/coreos/etcd/releases/download/v2.1.2/etcd-v2.1.2-linux-amd64.tar.gz \
-o etcd-v2.1.2-linux-amd64.tar.gz

tar xzf etcd-v2.1.2-linux-amd64.tar.gz

sudo mv etcd-v2.1.2-linux-amd64/etcd* /usr/local/bin

rm -rf etcd-v2.1.2-linux-amd64*

etcd >/tmp/etcd.log 2>&1 &


### test etcd service

etcdctl set myService/port "1234"
etcdctl set myService/ip "1.2.3.4"
etcdctl get myService/port
etcdctl get myService/ip

etcdctl ls myService
etcdctl rm myService/port
etcdctl ls myService

sudo apt-get install -y jq


curl http://localhost:2379/v2/keys/myService/newPort \
-X PUT \
-d value="4321" | jq '.'

curl http://localhost:2379/v2/keys/myService/newPort \
| jq '.'


## Confige etcd with Ansible

vagrant up cd serv-disc-01 serv-disc-02 serv-disc-03


read roles/etcd/defaults/main.yml


vagrant ssh cd

ansible-playbook /vagrant/ansible/etcd.yml -i /vagrant/ansible/hosts/serv-disc

curl http://serv-disc-01:2379/v2/keys/test \
-X PUT \
-d value="works" | jq '.'

curl http://serv-disc-03:2379/v2/keys/test \
| jq '.'


read roles/registrator/tasks/main.yml

read registrator-etcd.yml



ansible-playbook \
/vagrant/ansible/registrator-etcd.yml \
-i /vagrant/ansible/hosts/serv-disc

export DOCKER_HOST=tcp://serv-disc-02:2375

docker run -d --name nginx \
--env SERVICE_NAME=nginx \
--env SERVICE_ID=nginx \
-p 1234:80 \
10.100.198.200:5000/nginx

docker logs registrator

curl http://serv-disc-01:2379/v2/keys/ | jq '.'
curl http://serv-disc-01:2379/v2/keys/nginx/ | jq '.'
curl http://serv-disc-01:2379/v2/keys/nginx/nginx | jq '.'

docker rm -f nginx
docker logs registrator

curl http://serv-disc-01:2379/v2/keys/nginx/nginx | jq '.'

read roles/confd/tasks/main.yml

read confd.yml

ansible-playbook \
/vagrant/ansible/confd.yml \
-i /vagrant/ansible/hosts/serv-disc


export DOCKER_HOST=tcp://serv-disc-01:2375

docker run -d --name nginx \
--env SERVICE_NAME=nginx \
--env SERVICE_ID=nginx \
-p 4321:80 \
10.100.198.200:5000/nginx

confd -onetime -backend etcd -node 10.100.194.203:2379

cat /tmp/example.conf


## Configure console

### On CD node

#### install consul command

sudo apt-get install -y unzip
wget https://releases.hashicorp.com/consul/0.6.3/consul_0.6.3_linux_amd64.zip
wget https://releases.hashicorp.com/consul/0.6.0/consul_0.6.0_linux_amd64.zip
unzip consul_0.6.0_linux_amd64.zip
sudo mv consul /usr/local/bin/consul
rm -f consul_0.6.0_linux_amd64.zip
sudo mkdir -p /data/consul/{data,config,ui}

#### test run consul web UI

wget https://releases.hashicorp.com/consul/0.6.0/consul_0.6.0_web_ui.zip
wget https://releases.hashicorp.com/consul/0.7.0/consul_0.7.0_web_ui.zip
unzip consul_0.7.0_web_ui.zip
sudo mv index.html static /data/consul/ui/

sudo consul agent \
-server \
-bootstrap-expect 1 \
-ui-dir /data/consul/ui \
-data-dir /data/consul/data \
-config-dir /data/consul/config \
-node=cd \
-bind=10.100.198.200 \
-client=0.0.0.0 \
>/tmp/consul.log &


curl -X PUT -d 'this is a test' \
http://localhost:8500/v1/kv/msg1
curl -X PUT -d 'this is another test' \
http://localhost:8500/v1/kv/messages/msg2
curl -X PUT -d 'this is a test with flags' \
http://localhost:8500/v1/kv/messages/msg3?flags=1234

curl http://localhost:8500/v1/kv/?recurse \
| jq '.'

curl http://localhost:8500/v1/kv/msg1 \
| jq '.'

curl http://localhost:8500/v1/kv/msg1?raw


http://10.100.198.200:8500/ui/

curl -X DELETE http://localhost:8500/v1/kv/messages/msg2

curl -X DELETE http://localhost:8500/v1/kv/?recurse

### On cluster nodes

Read roles/consul/tasks/main.yml

Read roles/consul/defaults/main.yml

Read consul.yml

ansible-playbook \
/vagrant/ansible/consul.yml \
-i /vagrant/ansible/hosts/serv-disc


curl serv-disc-01:8500/v1/catalog/nodes \
| jq '.'

#### Set up Registrator working with Consul

export DOCKER_HOST=tcp://serv-disc-01:2375

docker run -d --name registrator-consul-kv \
-v /var/run/docker.sock:/tmp/docker.sock \
-h serv-disc-01 \
10.100.198.200:5000/registrator \
-ip 10.100.194.201 consulkv://10.100.194.201:8500/services



docker logs registrator-consul-kv


curl http://serv-disc-01:8500/v1/kv/services/nginx-80/nginx?raw


docker run -d --name registrator-consul \
-v /var/run/docker.sock:/tmp/docker.sock \
-h serv-disc-01 \
10.100.198.200:5000/registrator \
-ip 10.100.194.201 consul://10.100.194.201:8500

curl http://serv-disc-01:8500/v1/catalog/service/nginx-80 | jq '.'


docker run -d --name nginx2 \
--env "SERVICE_ID=nginx2" \
--env "SERVICE_NAME=nginx" \
--env "SERVICE_TAGS=balancer,proxy,www" \
-p 1111:80 \
10.100.198.200:5000/nginx

curl http://serv-disc-01:8500/v1/catalog/service/nginx-80 | jq '.'

Read registrator.yml

ansible-playbook \
/vagrant/ansible/registrator.yml \
-i /vagrant/ansible/hosts/serv-disc

Setting Up Consul Template

wget https://releases.hashicorp.com/consul-template/0.12.0/\
consul-template_0.12.0_linux_amd64.zip
sudo apt-get install -y unzip
unzip consul-template_0.12.0_linux_amd64.zip
sudo mv consul-template /usr/local/bin
rm -rf consul-template_0.12.0_linux_amd64*

echo '
{{range service "nginx-80"}}
The address is {{.Address}}:{{.Port}}
{{end}}
' >/tmp/nginx.ctmpl

curl http://serv-disc-01:8500/v1/catalog/service/nginx-80 | jq '.'

consul-template \
-consul serv-disc-01:8500 \
-template "/tmp/nginx.ctmpl:/tmp/nginx.conf" \
-once
cat /tmp/nginx.conf

Read roles/consul-template/tasks/main.yml

Read consul-template.yml

ansible-playbook \
/vagrant/ansible/consul-template.yml \
-i /vagrant/ansible/hosts/serv-disc

http://10.100.194.201:8500/ui/#/dc1/services


export DOCKER_HOST=tcp://serv-disc-03:2375

docker run -d --name nginx2 \
--env "SERVICE_ID=nginx2" \
--env "SERVICE_NAME=nginx" \
--env "SERVICE_TAGS=balancer,proxy,www" \
-p 1111:80 \
10.100.198.200:5000/nginx

docker run -d --name nginx3 \
--env "SERVICE_ID=nginx3" \
--env "SERVICE_NAME=nginx" \
--env "SERVICE_TAGS=balancer,proxy,www" \
-p 1113:80 \
10.100.198.200:5000/nginx
