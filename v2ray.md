---
uuid: a0c21bb0-cabc-4bab-8f11-1e32c65e5b11
title: Linux下安装v2ray
categories:
- Linux
tags:
- v2ray
---

## v2ray简介
v2ray是一个强大的科学上网工具，详细介绍和使用请参见[官网](https://www.v2ray.com/)（已被墙），这里我主要介绍一下怎么在线以及离线安装v2ray.

## 在线一键安装
当你在能够科学上网的vps或者linux上时推荐使用官网的一键安装脚本
		bash <(curl -L -s https://install.direct/go.sh)
		#如果没有安装curl，先安装curl

## 手动离线安装


+ 2021/1/2补充：可以下载对应平台的zip包和go.sh放在同一文件夹下，执行`./go.sh --local ./v2ray.zip`

当你需要在不能科学上网的vps或者linux上安装v2ray时，需要先去[github](https://github.com/v2ray/v2ray-core/releases)下载对应版本的压缩包，然后解压你会得到如下的文件
![](https://i.loli.net/2020/07/16/wTODncjhpdK96qF.jpg)

有脚本安装可以得到以下文件路径
+ /usr/bin/v2ray/v2ray
+ /usr/bin/v2ray/v2ctl
+ /etc/v2ray/config.json
+ /usr/bin/v2ray/geoip.dat
+ /usr/bin/v2ray/geosite.dat

那我们就将解压的文件放到对应的路径，并启动
~~~bash
sudo mkdir /usr/bin/v2ray
sudo cp v2ray /usr/bin/v2ray/v2ray  #主程序
sudo cp v2ctl /usr/bin/v2ray/v2ctl  #工具
sudo cp geoip.dat /usr/bin/v2ray/geoip.dat
sudo cp geosite.dat /usr/bin/v2ray/geosite.dat
sudo mkdir /etc/v2ray/
sudo cp vpoint_vmess_freedom.json /etc/v2ray/config.json #配置文件随便，后面要编辑
sudo mkdir -p /var/log/v2ray	#创建日志文件夹，不然v2ray启动不了
sudo /usr/bin/v2ray/v2ray -config /etc/v2ray/config.json #启动v2ray

~~~
## 配置文件编辑
v2ary 不区分客户端和服务端，二者的不同主要体现在config.json 文件
+ 服务端配置
~~~
{
  "log": {
    "loglevel": "info",
    "access": "/var/log/v2ray/access.log",
    "error": "/var/log/v2ray/error.log"
  },
  "inbounds": [
{
    "port": #端口,
    "protocol": "vmess",
    "settings": {
      "clients": [
        {
          "id": "#uuid，可用v2ctl生成",
          "level": 0,
          "alterId": 71,
	  "email": "lhd@v2ray.com"
        }
      ]
    }
     
  }],
  "outbounds": [{
    "protocol": "freedom",
    "settings": {}
  },{
    "protocol": "blackhole",
    "settings": {},
    "tag": "blocked"
  }],
  "routing": {
      "domainStrategy": "IPIfNonMatch",
	"rules": [
           {
        "type": "field",
        "ip": ["geoip:private"],
        "outboundTag": "blocked"
          }
             
        ]
  }
}

~~~
+ 客户端配置
~~~
{
  "log": {
    "loglevel": "info",
    "access": "/var/log/v2ray/access.log",
    "error": "/var/log/v2ray/error.log"
  },
  "inbounds": [
{
    "port": 1080,
    "listen": "127.0.0.1",
    "protocol": "socks",
    "settings": {
      "udp": true
    },
    "tag": "local"
}],
  "outbounds": [
{
    "protocol": "vmess",
    "settings": {
      "vnext": [{
        "address": "#服务端ip",
        "port": #服务端对应的端口,
        "users": [{ "id": "#服务端对应的id",
                    "alterId": #服务端对应的alterid,
                    "security": "auto",
                    "level": 0
         }]
      }]
    },
     "tag": "localproxy"
  },
    {
      "protocol": "freedom",
      "settings": {
        "domainStrategy": "UseIP"
      },
      "tag": "direct"
    }
],"routing": {
    "domainStrategy": "IPIfNonMatch",
    "rules": [
	{
	"inboundTag": ["local"],
        "type": "field",
        "outboundTag": "localproxy"

	}  
    ]
  }
}

~~~

## 其他配置

+ 如果服务器没有开启端口，需要开启端口，下面以centos为例
~~~bash
	firewall-cmd --zone=public --add-port=XXXX/tcp --permanent
	firewall-cmd --zone=public --add-port=XXXX/udp --permanent
	firewall-cmd --reload	//重启防火墙

~~~

+ proxychains4编译安装
ProxyChains4 是linux平台的一个代理切换软件，主要用于命令行。
先从[GitHub](https://github.com/rofl0r/proxychains-ng)上下载压缩包并解压，进入解压后的文件夹
~~~bash
./configure --prefix=/usr --sysconfdir=/etc 
make
sudo make install
sudo make install-config
#编辑配置文件,将最后一行改成127.0.0.1 1080
sudo vim /etc/proxychains.conf
~~~

+ 设置v2ray开机自启
~~~bash
sudo vim /lib/systemd/system/v2ray.service
[Unit]                                                                       
Description=V2Ray Service

[Service]
User=root 
#该服务进程具体的shell执行文件，xxx 是文件名，不是文件夹名
ExecStart=/usr/bin/v2ray/v2ray -config /etc/v2ray/config.json
#以下这些不用改，照旧就行
SuccessExitStatus=143
TimeoutStopSec=10
Restart=on-failure
RestartSec=5

[Install]
WantedBy=multi-user.target

#服务的基本用法
sudo systemctl enable v2ray.service	#开机自启
sudo systemctl start v2ray.service	#开始服务
sudo systemctl stop v2ray.service	
sudo systemctl restart v2ray.service
sudo systemctl status v2ray.service
~~~





