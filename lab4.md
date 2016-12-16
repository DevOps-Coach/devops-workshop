```
git clone https://github.com/vfarcic/books-ms.git
cd books-ms
vagrant plugin install vagrant-cachier

read Vagrantfile

vagrant up dev

vagrant ssh dev

sudo sh bootstrap.sh

PYTHONUNBUFFERED=1 ansible-playbook  /vagrant/ansible/dev.yml -c local

ansible --version
docker --version
docker-compose --version
cd /vagrant
ll

sudo docker run -it --rm \
-v $PWD/client/components:/source/client/components \
-v $PWD/client/test:/source/client/test \
-v $PWD/src:/source/src \
-v $PWD/target:/source/target \
-p 8080:8080 \
--env TEST_TYPE=watch-front \
vfarcic/books-ms-tests


sudo docker-compose -f docker-compose-dev.yml run feTestsLocal

sudo docker-compose -f docker-compose-dev.yml run testsLocal
```
