# windows ubuntu nginx+ffmpeg+nginx-rtmp-module 实现 点播+直播

网上有很多关于使用nginx搭建视屏服务器的案列，我今天主要是搭建了一个hls视频服务器，主要目的是使用ffmpeg将视屏分片上传，方便前端拉取视频流，期间实现了rtmp的点播功能和直播功能。

其实 不论是在windows上面还是在linux上面安装 这几个软件都听简单的

在windows上面编译挺麻烦的，但是有人已经替我们编译好了：[github windows编译版本](https://github.com/illuspas/nginx-rtmp-win32)

在linux下面安装 还是蛮简单的（本来服务器一般也就是linux的，windows只是用来本地测试一下）：[ubuntu nginx+nginx-rtmp-module 安装](https://www.jianshu.com/p/a210eab31f45)


## 解释一下配置文件和使用方法

> 原始的配置文件是这样的

```
worker_processes  1;
events {
   worker_connections  1024;
}
http {
   include       mime.types;
   default_type  application/octet-stream;
   sendfile        on;
   keepalive_timeout  65;
   server {
       listen       80;
       server_name  localhost;
       location / {
           root   html;
           index  index.html index.htm;
       }
       error_page   500 502 503 504  /50x.html;
       location = /50x.html {
           root   html;
       }
   }
}

```

> 第一个 我们配置 rtmp 点播

```
worker_processes  1;
events {
   worker_connections  1024;
}
rtmp {                #RTMP服务
   server {
       listen 1935;  #//服务端口
       chunk_size 4096;   #//数据传输块的大小

       application vod { # vod  -> vedio on demand 点播的意思
           play /home/ubuntu/vedio; #视频文件存放位置。
       }
   }
}
http {
   include       mime.types;
   default_type  application/octet-stream;
   sendfile        on;
   keepalive_timeout  65;
   server {
       listen       80;
       server_name  localhost;
       location / {
           root   html;
           index  index.html index.htm;
       }
       error_page   500 502 503 504  /50x.html;
       location = /50x.html {
           root   html;
       }
   }
}

```
> 这样就完成了点播的配置，在 /home/ubuntu/vedio 文件夹下面存放你的视频文件，然后使用 rtmp://ip/vod/vedio.mp4 即可在 vlc 中播放

> 不加多扣的原因是 rtmp 协议的默认端口就是 1935

> 接下来实现 rtmp 直播，在这里就需要用到 ffmpeg 了， 自行安装

```
worker_processes  1;
events {
   worker_connections  1024;
}
rtmp {                #RTMP服务
   server {
       listen 1935;  #//服务端口
       chunk_size 4096;   #//数据传输块的大小

       //1
       application vod { # vod  -> vedio on demand 点播的意思
           play /home/ubuntu/vedio; #视频文件存放位置。
       }
       //2
       application tv{
           live on; //这么一行设置皆可以了
       }

   }
}
http {
   include       mime.types;
   default_type  application/octet-stream;
   sendfile        on;
   keepalive_timeout  65;
   server {
       listen       80;
       server_name  localhost;
       location / {
           root   html;
           index  index.html index.htm;
       }
       error_page   500 502 503 504  /50x.html;
       location = /50x.html {
           root   html;
       }
   }
}
```