title: mac 关闭apache开机启动
category: other
date: 2015-11-23 17:20:00

---
虽然mac常以待机状态恢复基本不关机，不过有时候会死机。
现在常用nginx作为php的应用服务器，而且反向代理nginx使用起来比较方便。
所以需要关闭apache，释放80端口，使用nginx

mac os不像linux有/etc/init.d/rc.local以及service的方式可以设置程序随机启动，而是使用plist文件管理。

apache的plist文件在/System/Library/LaunchDaemons目录下,文件名是org.apache.httpd.plist

launchctl 管理OS X的启动脚本，控制启动计算机时需要开启的服务。也可以设置定时执行特定任务的脚本，就像Linux cron一样。

Launchd脚本存储在以下位置：

    ~/Library/LaunchAgents    
    /Library/LaunchAgents          
    /Library/LaunchDaemons
    /System/Library/LaunchAgents
    /System/Library/LaunchDaemons

可以使用 `launchctl list |grep apache` 查看是否开机启动apache
关闭开机启动

    sudo launchctl unload  -w /System/Library/LaunchDaemons/org.apache.httpd.plist

然而这并没有完，使用/usr/sbin/apachectl启动apache的时候，apache依旧会重新开机启动，原因是/usr/sbin/apachectl这个脚本里，使用的是launchctl load来启动的，它会把它加入到启动项里，解决办法就是自己写一个httpd的启动关闭脚本,直接启动httpd