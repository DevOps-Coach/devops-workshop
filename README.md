# DevOps 概念导入研讨
课程《 DevOps 概念导入研讨》 的相关文档 ，课程介绍 http://devopscoach.org/course/devops-workshop/

## 实验环境需求

### 硬件 - Laptop

* CPU Intel i5 或者更高
* RAM 8GB  或者更高
* 磁盘 SSD 50GB 至少可用空间

### 软件

* Vagrant 最新版，电脑操作系统Linux,Mac, Windows 都可以
* 下载并且导入下列 box 镜像
  * bento/centos-7.1 (virtualbox, 0)
  * ubuntu/trusty64  (virtualbox, 0)
  * ubuntu/wily64    (virtualbox, 0)
  

### 网络
本机可以访问外网，本机有VPN可以访问墙外，虚拟机可以访问外网和墙外。 否则镜像构建会失败，实验将无法彻底完成。网速越快越好。

## 环境准备
初始化一个Ubuntu虚拟机并且登陆进去，确保能访问外网。

''‘sh

mkdir test
cd test
vagrant init ubuntu/trusty64
vagrant up
vagrant ssh
ping qq.com

'''
