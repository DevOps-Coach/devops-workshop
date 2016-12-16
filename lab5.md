```
cd ..
git clone https://github.com/vfarcic/ms-lifecycle.git
cd ms-lifecycle
vagrant up cd
vagrant ssh cd

git clone https://github.com/vfarcic/books-ms.git
cd books-ms

docker build \
-f Dockerfile.test \
-t 10.100.198.200:5000/books-ms-tests \
.

docker-compose \
-f docker-compose-dev.yml \
run --rm tests

ll target/scala-2.10/

read Dockerfile

docker build -t 10.100.198.200:5000/books-ms .

docker run -d --name books-db mongo
docker run -d --name books-ms \
-p 8080:8080 \
--link books-db:db \
10.100.198.200:5000/books-ms

docker exec -it books-ms bash
env | grep DB

docker ps -a
docker logs books-ms

docker rm -f books-ms books-db
docker ps -a

docker-compose -f docker-compose-dev.yml up -d app

read docker-compose-dev.yml file

docker-compose ps

docker-compose logs

curl -H 'Content-Type: application/json' -X PUT -d \
'{"_id": 1,
"title": "My First Book",
"author": "John Doe",
"description": "Not a very good book"}' \
http://localhost:8080/api/v1/books | jq '.'
curl -H 'Content-Type: application/json' -X PUT -d \
'{"_id": 2,
"title": "My Second Book",
"author": "John Doe",
"description": "Not a bad as the first book"}' \
http://localhost:8080/api/v1/books | jq '.'
curl -H 'Content-Type: application/json' -X PUT -d \
'{"_id": 3,
"title": "My Third Book",
"author": "John Doe",
"description": "Failed writers club"}' \
http://localhost:8080/api/v1/books | jq '.'
curl http://localhost:8080/api/v1/books | jq '.'
curl http://localhost:8080/api/v1/books/_id/1 | jq '.


docker-compose stop
docker-compose rm -f

docker push 10.100.198.200:5000/books-ms

docker push 10.100.198.200:5000/books-ms-tests
```
