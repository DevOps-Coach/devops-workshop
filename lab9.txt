# 代理服务 - Proxy Services

## 手工配额代理服务器

vagrant up cd proxy

Read nginx.yml

Read roles/nginx/tasks/main.yml

Read roles/nginx/files/services.conf

vagrant ssh cd

ansible-playbook /vagrant/ansible/nginx.yml \
-i /vagrant/ansible/hosts/proxy

When gpg error, run those commands in proxy vm.

sudo gpg --keyserver keyserver.ubuntu.com --recv 3E5C1192
gpg --export --armor  3E5C1192 | sudo apt-key add -
sudo apt-key adv --keyserver hkp://pgp.mit.edu:80 --recv-keys 58118E89F3A912897C070ADBF76221572C52609D

Downlaod compose to dl folder.
wget https://raw.githubusercontent.com/vfarcic\
/books-ms/master/docker-compose.yml

cd dl

export DOCKER_HOST=tcp://proxy:2375
docker-compose up -d app
docker-compose ps
curl http://proxy/api/v1/books

curl http://10.100.193.200:8500/v1/catalog/service/books-ms | jq '.'

PORT=$(docker inspect \
--format='{{(index (index .NetworkSettings.Ports "8080/tcp") 0).HostPort}}' \
vagrant_app_1)
echo $PORT
curl http://proxy:$PORT/api/v1/books | jq '.'
There is no book. you will get nothing. That is correct.

echo "
location /api/v1/books {
proxy_pass http://10.100.193.200:$PORT/api/v1/books;
}
" | tee books-ms.conf

scp books-ms.conf \
proxy:/data/nginx/includes/books-ms.conf

# pass: vagrant

docker kill -s HUP nginx

curl -H 'Content-Type: application/json' -X PUT -d \
"{\"_id\": 1,
\"title\": \"My First Book\",
\"author\": \"John Doe\",
\"description\": \"Not a very good book\"}" \
http://proxy/api/v1/books | jq '.'
curl http://proxy/api/v1/books | jq '.'


## 自动化配置代理服务器

docker-compose scale app=2
docker-compose ps

Read nginx-includes.conf

Read nginx-upstreams.ctmpl

consul-template \
-consul proxy:8500 \
-template "nginx-upstreams.ctmpl:nginx-upstreams.conf" \
-once
cat nginx-upstreams.conf

scp nginx-includes.conf \
proxy:/data/nginx/includes/books-ms.conf

scp nginx-upstreams.conf \
proxy:/data/nginx/upstreams/books-ms.conf

docker kill -s HUP nginx

curl http://proxy/api/v1/books | jq '.'
repeat above 4 times

docker logs nginx

docker stop dl_app_2

docker stop nginx

## 测试 HAproxy

### 手动

Read haproxy roles file

ansible-playbook /vagrant/ansible/haproxy.yml \
-i /vagrant/ansible/hosts/proxy

export DOCKER_HOST=tcp://proxy:2375
docker ps -a
docker logs haproxy


PORT=$(docker inspect \
--format='{{(index (index .NetworkSettings.Ports "8080/tcp") 0).HostPort}}' \
dl_app_1)

echo $PORT

echo "
frontend books-ms-fe
bind *:80
option http-server-close
acl url_books-ms path_beg /api/v1/books
use_backend books-ms-be if url_books-ms
backend books-ms-be
server books-ms-1 10.100.193.200:$PORT check
" | tee books-ms.service.cfg

cat /vagrant/ansible/roles/haproxy/files/haproxy.cfg.orig \
*.service.cfg | tee haproxy.cfg

scp haproxy.cfg proxy:/data/haproxy/config/haproxy.cfg

curl http://proxy/api/v1/books | jq '.'
docker start haproxy
docker logs haproxy
curl http://proxy/api/v1/books | jq '.'

### 自动化

docker-compose scale app=2
docker-compose ps

wget http://raw.githubusercontent.com/vfarcic\
/books-ms/master/haproxy.ctmpl \
-O haproxy.ctmpl

sudo consul-template \
-consul proxy:8500 \
-template "haproxy.ctmpl:books-ms.service.cfg" \
-once

cat books-ms.service.cfg


cat /vagrant/ansible/roles/haproxy/files/haproxy.cfg.orig \
*.service.cfg | tee haproxy.cfg

scp haproxy.cfg proxy:/data/haproxy/config/haproxy.cfg

docker logs haproxy
curl http://proxy/api/v1/books | jq '.'


docker stop dl_app_1
curl http://proxy/api/v1/books | jq '.'
curl http://proxy/api/v1/books | jq '.'
curl http://proxy/api/v1/books | jq '.'
curl http://proxy/api/v1/books | jq '.'
