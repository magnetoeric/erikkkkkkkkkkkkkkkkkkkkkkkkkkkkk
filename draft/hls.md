title: hls相关
category: other
date: 2017-01-10 20:59:00

---
## 概念
HTTP Live Streaming（简称 HLS）是一个基于 HTTP 的视频流协议，由 Apple 公司实现，Mac OS 上的 QuickTime、Safari 以及 iOS 上的 Safari 都能很好的支持 HLS，高版本 Android 也增加了对 HLS 的支持。一些常见的客户端如：MPlayerX、VLC 也都支持 HLS 协议。
HLS 协议基于 HTTP，非常简单。一个提供 HLS 的服务器需要做两件事：
编码：以 H.263 格式对图像进行编码，以 MP3 或者 HE-AAC 对声音进行编码，最终打包到 MPEG-2 TS（Transport Stream）容器之中；
分割：把编码好的 TS 文件等长切分成后缀为 ts 的小文件，并生成一个 .m3u8 的纯文本索引文件；
浏览器使用的是 m3u8 文件。m3u8 跟音频列表格式 m3u 很像，可以简单的认为 m3u8 就是包含多个 ts 文件的播放列表。播放器按顺序逐个播放，全部放完再请求一下 m3u8 文件，获得包含最新 ts 文件的播放列表继续播，周而复始。整个直播过程就是依靠一个不断更新的 m3u8 和一堆小的 ts 文件组成，m3u8 必须动态更新，ts 可以走 CDN。一个典型的 m3u8 文件格式如下：

```
#EXTM3U
#EXT-X-STREAM-INF:PROGRAM-ID=1, BANDWIDTH=200000
gear1/prog_index.m3u8
#EXT-X-STREAM-INF:PROGRAM-ID=1, BANDWIDTH=311111
gear2/prog_index.m3u8
#EXT-X-STREAM-INF:PROGRAM-ID=1, BANDWIDTH=484444
gear3/prog_index.m3u8
#EXT-X-STREAM-INF:PROGRAM-ID=1, BANDWIDTH=737777
gear4/prog_index.m3u8
```

可以看到 HLS 协议本质还是一个个的 HTTP 请求 / 响应，所以适应性很好，不会受到防火墙影响。但它也有一个致命的弱点：延迟现象非常明显。如果每个 ts 按照 5 秒来切分，一个 m3u8 放 6 个 ts 索引，那么至少就会带来 30 秒的延迟。如果减少每个 ts 的长度，减少 m3u8 中的索引数，延时确实会减少，但会带来更频繁的缓冲，对服务端的请求压力也会成倍增加。所以只能根据实际情况找到一个折中的点。
对于支持 HLS 的浏览器来说，直接这样写就能播放了：
<video src="*.m3u8" height="300" width="400"></video>

对于不支持 m3u8 的浏览器，比如 PC / Mac 上的 Chrome，需要借助 Flash 来实现了。网上有一些较为成熟的方案可以直接用，如：Sewise Player（免费）、JW Player（收费）。
Real Time Messaging Protocol
Real Time Messaging Protocol（简称 RTMP）是 Macromedia 开发的一套视频直播协议，现在属于 Adobe。这套方案需要搭建专门的 RTMP 流媒体服务如 Adobe Media Server，并且在浏览器中只能使用 Flash 实现播放器。它的实时性非常好，延迟很小，但无法支持移动端 WEB 播放是它的硬伤。
JW Player 能很好的播放采用 RTMP 协议的直播服务。


## 搭建简单的视频直播服务
nginx 

nginx rtmp 模块 
https://github.com/arut/nginx-rtmp-module


添加配置

```
rtmp {
    server {
        listen 1935;
        chunk_size 4000;
        application hls {
            live on;
            hls on;
            hls_path /usr/local/nginx/html/hls;
            hls_fragment 5s;
        }
        application mytv {
            live on;
            record off;
            allow publish all;
            allow play all;
        }
    }
}

http {
    server {
        listen  8080;
        location /hls {
            types {
                application/vnd.apple.mpegurl m3u8;
                video/mp2t ts;
            }
            root html;
            expires -1;
        }
    }
}
```

rtmp是服务器推流
端口是1935 hls aplication 这个directive会把 rtmp流写入到hls_path目录下的ts文件和m3u8文件
这个目录下的文件会动态刷新，前面写入的ts片段会被后面的ts片段覆盖。当这个直播流播完后，所有相关的ts片段和m3u8文件都将被删除。


监控 

在nginx.conf中加入stat模块的相关配置信息

```
location /stat {  
rtmp_stat all;  
rtmp_stat_stylesheet stat.xsl;  
}  

location /stat.xsl {  
root /usr/local/src/nginx-rtmp-module/;  
}

注意为了得到统计信息，还需要开启下面的控制模块

location /control {  
rtmp_control all;  
}  

```









