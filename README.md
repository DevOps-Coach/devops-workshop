# DevOps Toolkit Workshop
课程《 DevOps 概念导入研讨》 的相关文档 ，课程介绍 http://devopscoach.org/course/devops-workshop/

## 实验环境需求

### 硬件 - 笔记本电脑/台式机

* CPU Intel i5 或者更高
* RAM 8GB  或者更高
* 磁盘 SSD 50+GB可用空间

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

```
mkdir test
cd test
vagrant init ubuntu/trusty64
vagrant up
vagrant ssh
ping qq.com
ping google.com
```
## 环境初始化
下载本页面相关的所有代码到本机，进入 'ms-lifecycle' 目录。 用vagrant up 命令把所有需要的虚拟机都创建一次，然后依次关机，保留待用，不要作进一步其它操作。

```
git clone https://github.com/DevOps-Coach/devops-workshop.git
cd ms-lifecycle
vagrant up 
#等待所有虚拟机都创建完成，至少需要8GB内存
vagrant halt
```
