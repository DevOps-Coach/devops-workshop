```
vagrant up cd prod
vagrant ssh cd
ansible-playbook /vagrant/ansible/prod.yml \
-i /vagrant/ansible/hosts/prod

wget https://raw.githubusercontent.com/vfarcic/books-ms/master/docker-compose.yml


read docker-compose.yml file

base:
image: 10.100.198.200:5000/books-ms
ports:
- 8080
environment:
- SERVICE_NAME=books-ms
app:
extends:
service: base
links:
- db:db
db:
image: mongo

export DOCKER_HOST=tcp://prod:2375

docker-compose up -d app


/etc/default/docker
DOCKER_OPTS="$DOCKER_OPTS --insecure-registry 10.100.198.200:5000 -H tcp://0.0.0\
.0:2375 -H unix:///var/run/docker.sock"


docker-compose ps

docker inspect dl_app_1

PORT=$(docker inspect \
--format='{{(index (index .NetworkSettings.Ports "8080/tcp") 0).HostPort}}' \
dl_app_1)
echo $PORT
```
