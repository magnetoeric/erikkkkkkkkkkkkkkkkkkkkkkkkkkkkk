title: STOMP
category: other
date: 2017-04-05 14:17:00

---
## STOMP 概述
STOMP即Simple (or Streaming) Text Orientated Messaging Protocol，简单(流)文本定向消息协议，它提供了一个可互操作的连接格式，允许STOMP客户端与任意STOMP消息代理（Broker）进行交互。STOMP协议由于设计简单，易于开发客户端，因此在多种语言和多种平台上得到广泛地应用。

STOMP协议的前身是TTMP协议（一个简单的基于文本的协议），专为消息中间件设计。

STOMP是一个非常简单和容易实现的协议，其设计灵感源自于HTTP的简单性。尽管STOMP协议在服务器端的实现可能有一定的难度，但客户端的实现却很容易。例如，可以使用Telnet登录到任何的STOMP代理，并与STOMP代理进行交互。

## STOMP的实现

业界已经有很多优秀的STOMP的服务器/客户端的开源实现，下面就介绍一下这方面的情况。

### STOMP服务器
<table><tr>
            <th>项目名</th>
            <th>兼容STOMP的版本	</th>
            <th>描述</th>
        </tr> <tr>
            <th>Apache Apollo	</th>
            <th>1.0 1.1 1.2</th>
            <th><a href="http://activemq.apache.org/apollo">ActiveMQ</a>的继承者 </th>
        </tr>
        <tr>
            <th>Apache ActiveMQ	</th>
            <th>1.0 1.1	</th>
            <th>流行的开源消息服务器 </th>
        </tr>
         <tr>
            <th>RabbitMQ	</th>
            <th>1.0 1.1 1.2	</th>
            <th>基于Erlang、支持多种协议的消息Broker，通过<a href="http://www.rabbitmq.complugins.html#rabbitmq-stomp">STOMP插件</a>支持STOMP协议 
 </th>
        </tr>
</table> 

### STOMP客户端库
<table>
	<tr>
		<th> 项目名 </th>
		<th> 兼容STOMP的版本	 </th>
		<th> 描述 </th>
	</tr>
	<tr>
		<th> Gozirra </th>
		<th> 1.0	 </th>
		<th> <a href="http://www.germane-software.com/software/Java/Gozirra/">Java客户端库]</a></th>
	</tr>
	<tr>
		<th> stompest </th>
		<th> 1.0 1.1 1.2		 </th>
		<th> <a href="https://github.com/nikipore/stompest">Python客户端库，全功能实现，包括同步和异步</a> </th>
	</tr>
</table>

## STOMP协议分析

STOMP协议与HTTP协议很相似，它基于TCP协议，使用了以下命令：

CONNECT
SEND
SUBSCRIBE
UNSUBSCRIBE
BEGIN
COMMIT
ABORT
ACK
NACK
DISCONNECT

STOMP的客户端和服务器之间的通信是通过“帧”（Frame）实现的，每个帧由多“行”（Line）组成。
第一行包含了命令，然后紧跟键值对形式的Header内容。
第二行必须是空行。
第三行开始就是Body内容，末尾都以空字符结尾。
STOMP的客户端和服务器之间的通信是通过MESSAGE帧、RECEIPT帧或ERROR帧实现的，它们的格式相似。

## 使用
### ActiveMQ 安装
从[官网](http://activemq.apache.org)下载最新版本,解压
配置conf目录下的activemq.xml 打开stomp端口
具体可以看[这里](http://activemq.apache.org/getting-started.html#GettingStarted-TestingtheInstallationTestingtheInstallation)

```

./bin/activemq start

```

### PHP STOMP
安装[STOMP扩展](http://php.net/manual/en/stomp.installation.php)

### php连接
贴个官网的代码

```

<?php

$queue  = '/queue/foo';
$msg    = 'bar';

/* connection */
try {
    $stomp = new Stomp('tcp://localhost:61613');
} catch(StompException $e) {
    die('Connection failed: ' . $e->getMessage());
}

/* send a message to the queue 'foo' */
$stomp->send($queue, $msg);

/* subscribe to messages from the queue 'foo' */
$stomp->subscribe($queue);

/* read a frame */
$frame = $stomp->readFrame();

if ($frame->body === $msg) {
    var_dump($frame);

    /* acknowledge that the frame was received */
    $stomp->ack($frame);
}

/* close connection */
unset($stomp);

?>

```

### 使用telnet
这类基于tcp的简单协议 都可以使用telnet模拟，甚至是查找问题
CONNECT 

```

➜  ~ telnet 10.207.26.112 61613
Trying 10.207.26.112...
Connected to ericwang.com.
Escape character is '^]'.
CONNECT

^@
CONNECTED
server:ActiveMQ/5.14.4
heart-beat:0,0
session:ID:ericwang-37951-1491805812122-3:11
version:1.0


```

SEND

```
SEND
destination:testSTOMP
receipt:123
hello

^@
RECEIPT
receipt-id:123

```

DISCONNECT

```
DISCONNECT
receipt:123

^@
RECEIPT
receipt-id:123

```

其余的可以参考[这里](http://www.edc4it.com/blog/java/stomp-1-2-activemq-using-telnet.html)
