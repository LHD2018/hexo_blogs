---
uuid: 16aab646-2492-1215-25d3-c53a0c73ebda
title: Ubuntu初始设置
categories:
- Linux
tags:
- linux
- 美化
---

+ ubuntu安装分区
>单独安装ubuntu时，/boot分区设为主分区，给2G，其他都是逻辑分区，交换分区（swap）给8G，根目录 / 给40G以上，其他的都给/ home，将boot安装在/boot分区。

>安装Windows和ubuntu双系统时，设置U盘启动时要选择有uefi前缀的U盘，注意不要分/boot分区，交换分区（swap）设为主分区，给8G，其他都是逻辑分区，根目录 / 给40G以上，其他给 /home，将boot安装在Windows boot manager。

+ 一般ubuntu装好后，分辨率都会有问题，这是因为显卡驱动没有安装，参考[教程](https://zhuanlan.zhihu.com/p/59618999)进行安装 
~~~bash
sudo add-apt-repository ppa:graphics-drivers/ppa //添加ppa仓库
sudo apt update
sudo ubuntu-drivers autoinstall //自动安装
~~~

+ 美化 
~~~bash
//安装主题
sudo add-apt-repository ppa:noobslab/themes
sudo apt-get update
sudo apt-get install flatabulous-theme
	
//安装图标
sudo add-apt-repository ppa:noobslab/icons
sudo apt-get update
sudo apt-get install ultra-flat-icons
	
//安装字体
sudo apt-get install fonts-wqy-microhei
	
安装gnome-tweak-tool配置
sudo apt-get install gnome-tweak-tool
~~~

+ 切换阿里源
	lsb_release -a	#查看系统codename
	cd /etc/apt
	sudo mv sources.list sources.list_bak	#备份原来的源

~~~
sudo vim sources.list	#编辑源文件，复制以下内容
# deb cdrom:[Ubuntu 16.04 LTS _Xenial Xerus_ - Release amd64 (20160420.1)]/ xenial main restricted
deb-src http://archive.ubuntu.com/ubuntu xenial main restricted #Added by software-properties
deb http://mirrors.aliyun.com/ubuntu/ xenial main restricted
deb-src http://mirrors.aliyun.com/ubuntu/ xenial main restricted multiverse universe #Added by software-properties
deb http://mirrors.aliyun.com/ubuntu/ xenial-updates main restricted
deb-src http://mirrors.aliyun.com/ubuntu/ xenial-updates main restricted multiverse universe #Added by software-properties
deb http://mirrors.aliyun.com/ubuntu/ xenial universe
deb http://mirrors.aliyun.com/ubuntu/ xenial-updates universe
deb http://mirrors.aliyun.com/ubuntu/ xenial multiverse
deb http://mirrors.aliyun.com/ubuntu/ xenial-updates multiverse
deb http://mirrors.aliyun.com/ubuntu/ xenial-backports main restricted universe multiverse
deb-src http://mirrors.aliyun.com/ubuntu/ xenial-backports main restricted universe multiverse #Added by software-properties
deb http://archive.canonical.com/ubuntu xenial partner
deb-src http://archive.canonical.com/ubuntu xenial partner
deb http://mirrors.aliyun.com/ubuntu/ xenial-security main restricted
deb-src http://mirrors.aliyun.com/ubuntu/ xenial-security main restricted multiverse universe #Added by software-properties
deb http://mirrors.aliyun.com/ubuntu/ xenial-security universe
deb http://mirrors.aliyun.com/ubuntu/ xenial-security multiverse
~~~

```bash
sudo apt-get update
```

+ 安装米聊,chrome,输入法等软件