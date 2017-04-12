title: 0mq的一些尝试
category: other
date: 2017-01-11 20:30:00

---
## 安装
http://zeromq.org/bindings:php
需要libzmq-dev包

## 简单模式
### server 
```
<?php
/*
*  Hello World server
*  Binds REP socket to tcp://*:5555
*  Expects "Hello" from client, replies with "World"
* @author Ian Barber <ian(dot)barber(at)gmail(dot)com>
*/

$context = new ZMQContext(1);

//  Socket to talk to clients
$responder = new ZMQSocket($context, ZMQ::SOCKET_REP);
$responder->bind("tcp://*:5555");

while (true) {
    //  Wait for next request from client
    $request = $responder->recv();
    printf ("Received request: [%s]\n", $request);

    //  Do some 'work'
    sleep (1);

    //  Send reply back to client
    $responder->send("World");
}
```

### client 

```
<?php
/*
*  Hello World client
*  Connects REQ socket to tcp://localhost:5555
*  Sends "Hello" to server, expects "World" back
* @author Ian Barber <ian(dot)barber(at)gmail(dot)com>
*/

$context = new ZMQContext();

//  Socket to talk to server
echo "Connecting to hello world server…\n";
$requester = new ZMQSocket($context, ZMQ::SOCKET_REQ);
$requester->connect("tcp://localhost:5555");

for ($request_nbr = 0; $request_nbr != 10; $request_nbr++) {
    printf ("Sending request %d…\n", $request_nbr);
    $requester->send("Hello");

    $reply = $requester->recv();
    printf ("Received reply %d: [%s]\n", $request_nbr, $reply);
}
```

## 获取0mq版本号 
```
<?php
/* Report 0MQ version
*
* @author Ian Barber <ian(dot)barber(at)gmail(dot)com>
*/

if (class_exists("ZMQ") && defined("ZMQ::LIBZMQ_VER")) {
    echo ZMQ::LIBZMQ_VER, PHP_EOL;
}
```


## 发布-订阅模式

### server 

```
<?php
/*
*  Weather update server
*  Binds PUB socket to tcp://*:5556
*  Publishes random weather updates
* @author Ian Barber <ian(dot)barber(at)gmail(dot)com>
*/

//  Prepare our context and publisher
$context = new ZMQContext();
$publisher = $context->getSocket(ZMQ::SOCKET_PUB);
$publisher->bind("tcp://*:5556");
$publisher->bind("ipc://weather.ipc");

while (true) {
    //  Get values that will fool the boss
    $zipcode     = mt_rand(0, 100000);
    $temperature = mt_rand(-80, 135);
    $relhumidity = mt_rand(10, 60);

    //  Send message to all subscribers
    $update = sprintf ("%05d %d %d", $zipcode, $temperature, $relhumidity);
    $publisher->send($update);
}
```


### client
```
<?php
/*
*  Weather update client
*  Connects SUB socket to tcp://localhost:5556
*  Collects weather updates and finds avg temp in zipcode
* @author Ian Barber <ian(dot)barber(at)gmail(dot)com>
*/

$context = new ZMQContext();

//  Socket to talk to server
echo "Collecting updates from weather server…", PHP_EOL;
$subscriber = new ZMQSocket($context, ZMQ::SOCKET_SUB);
$subscriber->connect("tcp://localhost:5556");

//  Subscribe to zipcode, default is NYC, 10001
$filter = $_SERVER['argc'] > 1 ? $_SERVER['argv'][1] : "10001";
$subscriber->setSockOpt(ZMQ::SOCKOPT_SUBSCRIBE, $filter);

//  Process 100 updates
$total_temp = 0;
for ($update_nbr = 0; $update_nbr < 100; $update_nbr++) {
    $string = $subscriber->recv();
    sscanf ($string, "%d %d %d", $zipcode, $temperature, $relhumidity);
    $total_temp += $temperature;
}
printf ("Average temperature for zipcode '%s' was %dF\n",
    $filter, (int) ($total_temp / $update_nbr));
```

## 总结
速度最快的消息中间件,支持各种客户端，缺点是不能保证消息丢失

