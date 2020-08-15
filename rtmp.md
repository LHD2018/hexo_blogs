---
uuid: 6de37c35-4f6e-6483-3c9e-0ac28cf911e8
title: 网络图传配置
categories:
- Linux
tags:
- linux
- web
---
因为学习需要搭建远程的图像传输，最后决定采用nginx + rtmp模块+ ffmpeg+web的方式进行

## 服务器端配置
+ 首先服务器端要安装nginx和nginx-rtmp模块，如果都没有安装可以一起编译安装，如果已经安装nginx，则需要单独安装模块。下面简单说一下
1. 下载nginx-rtmp-module GitHub地址<https://github.com/arut/nginx-rtmp-module>
~~~bash
cd /usr/local
git clone https://github.com/arut/nginx-rtmp-module.git
~~~

2. 查看nginx版本号与配置，记下版本号和复制configure arguments的内容。
~~~bash
nginx -V
~~~
3. 下载对应版本号的nginx并解压
~~~bash
wget http://nginx.org/download/nginx-1.12.1.tar.gz 
tar -zxvf nginx-1.12.1.tar.gz 
cd nginx-1.12.1
~~~
4. 编译
~~~bash
./configure --user=www --group=www --prefix=/usr/local/nginx --with-openssl=/usr/local/nginx/src/openssl --add-module=/usr/local/nginx/src/ngx_devel_kit --add-module=/usr/local/nginx/src/lua_nginx_module --add-module=/nginx-rtmp-module 
//configure 后面加你复制的内容并且在第一个module前面加--add-module=/usr/local/nginx-rtmp-module 如果报错就删除报错的模块直到通过
make //编译
~~~
5. 替换文件
~~~bash
cp ./objs/nginx /usr/sbin/
systemctl restart nginx
~~~
+ 修改nginx配置并实现web播放
~~~
vim /etc/nginx/nginx.conf
//在http前面加上
rtmp {
 
    server {
         
        listen 8090;  #//服务端口 
         
        chunk_size 4096;   #//数据传输块的大小
 
        application vod{
            play /home/videos;
}
        application live { 
            live on;
            hls on;
            wait_key on;
            hls_path /home/videos/hls;
            hls_fragment 600s;
            hls_playlist_length 10m;
            hls_continuous on;
            hls_cleanup on;
            hls_nested on;

        }
    }
}
~~~
~~~
//域名绑定

server{
        listen 80;
        server_name video.cloudlhd.cn;

        location /stat {
                rtmp_stat all;
                rtmp_stat_stylesheet stat.xsl;
        }
    
        location /stat.xsl {
                root /usr/local/nginx-rtmp-module/;
        }
    
        location / {
                root /usr/share/nginx/video_live;
                index index.html index.htm;
        }
    
        location /live {
                types {
                  application/vnd.apple.mpegurl m3u8;
                  videos/mp2t ts;
                }
                alias /home/videos/hls;
                expires -1;
                add_header Cache-Control no-cache;
                add_header Access-Control-Allow-Origin *;
        }
        error_page 500 502 503 504 /50x.html;
        location = /50x.html {
                root html;
        }
}
~~~
~~~
//网站根目录html文件

<html>
<head>
    <title>远程图传</title>
    <meta charset="utf-8">
    <link href="http://vjs.zencdn.net/5.5.3/video-js.css" rel="stylesheet">
    <!-- If you'd like to support IE8 -->
    <script src="http://vjs.zencdn.net/ie8/1.1.1/videojs-ie8.min.js"></script>
    <script src="http://vjs.zencdn.net/5.5.3/video.js"></script>
</head>
<body>
	<center>
		<font size="30" style="line-height:400%;">远程图传测试页面</font>
		<br>
		<font size="5">欢迎访问:</font>
		<font size="8" color="red" style="line-height:200%;">video.cloudlhd.cn</font>
		<video id="my-video" class="video-js" controls preload="auto" width="1080" height="720"
       poster="http://ppt.downhot.com/d/file/p/2014/08/12/9d92575b4962a981bd9af247ef142449.jpg" data-setup="{}">
    <source src="rtmp://xxx.xxx.xx.x:8090/live/" type="rtmp/flv"> //服务器ip：端口
    </p>
</video>
	</center>

</body>
</html>
~~~

## 推流
这里以windows端为例，先去f'fmpeg下载文件，然后在有bin目录打开终端
~~~
.\ffmpeg.exe -re -i test.mp4 -f flv rtmp://xxx.xxx.xxx.x:8090/live
~~~
