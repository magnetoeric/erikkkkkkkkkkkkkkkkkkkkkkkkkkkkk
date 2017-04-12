title: nginx笔记
tags:
  - C
category: nginx
date: 2017-02-10 22:01:00

---

### nginx deny all
仅允许某个网段访问，其他全部拒绝
```
server {
    […]
    allow 192.168.0.0/16;
    deny all;
    error_page 403 /error403.html;
}
```

尽管允许某个网段访问，依然无法访问，原因是/error403.html页面也被deny了，正确的如下

```
server {
    […]
    location / {
        error_page 403 /error403.html;
        allow 192.168.0.0/16;
        deny all;
    }
    location = /error403.html {
        allow all;
    }
}

```

如果想配置多个错误页

```
server {
    […]
    location / {
        error_page 403 /error403.html;
        error_page 404 /error404.html;
        allow 192.168.0.0/16;
        deny all;
    }
    location ~ "^/error[0-9]{3}\.html$" {
        allow all;
    }
}
```


### nginx 400 bad request 客户端发送cookies过大
调整large_client_header_buffers大小


### 多条件rewrite
nginx 只有if指令没有else指令， 如果存在多重条件，可以通过多个变量实现
