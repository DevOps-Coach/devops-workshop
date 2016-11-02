Configuring the Production Environment

vagrant up cd prod --provision

vagrant ssh cd

ansible-playbook /vagrant/ansible/prod.yml -i /vagrant/ansible/hosts/prod

ansible-playbook prod.yml -i hosts/prod

The content of the prod.yml62 Ansible playbook is as follows.
- hosts: prod
remote_user: vagrant
serial: 1
sudo: yes
roles:
- common
- docker
