title: php socket
tags:
  - php
category: php
date: 2015-12-24 19:11:00

---
闲了很多天，又不知道写点什么，看书看到了socket这块，还挺有意思，照着书上的模仿了一下，用socket模拟smtp协议发送邮件，邮件服务器是qq的。不过书有点老，qq的邮件服务器必须用https了，所以自己摸索着实现了一下
```
<?php
/**
 * Created by PhpStorm.
 * User: eric
 * Date: 15/12/24
 * Time: 上午10:54
 */
class Mail{
    private $sock;
    private $errorStr;
    private $errorNo;
    private $timeout=30;
    private $host;
    private $port;
    private $user;
    private $password;
    function __construct($host,$port,$user,$pass,$format=1,$debug=0){
        $this->host = $host;
        $this->port = $port;
        $this->user = base64_encode($user);
        $this->password = base64_encode($pass);
        $this->sock = fsockopen($this->host,$this->port,$this->errorNo,$this->errorStr,$this->timeout);
        if(!$this->sock){
            exit("oops");
        }
        $response  = fgets($this->sock);
        if(strstr($response,'220') === false){
            exit("server error");
        }
    }
    function showDebug($response,$commond){
        echo $commond;
        echo $response.PHP_EOL;
    }
    function doCommond($commond,$return_code){
        fwrite($this->sock,$commond);
        $response = fgets($this->sock);
        if(stristr($response,"$return_code") == false){
            $this->showDebug($response,$commond);
            return false;
        }
        return true;
    }
    function sendMail($from,$to,$sub,$body){
        $detail = "FROM:".$from."\r\n";
        $detail.="To:".$to."\r\n";
        $detail.="Subject:".$sub."\r\n";
        $detail.="Content-type:text/plain;\r\n\r\n";
        $detail.=$body;
        $this->doCommond("HELO smtp.qq.com\r\n",250);
        $this->doCommond("AUTH LOGIN\r\n",334);
        $this->doCommond($this->user."\r\n",334);
        $this->doCommond($this->password."\r\n",235);
        $this->doCommond("MAIL FROM:".$from."\r\n",250);
        $this->doCommond("RCPT TO:".$to."\r\n",250);
        $this->doCommond("DATA\r\n",354);
        $this->doCommond($detail."\r\n.\r\n",250);
        $this->doCommond("QUIT\r\n",221);
        return true;

    }
}

$mail  =  new Mail('ssl://smtp.qq.com',465,'xxx@qq.com','特殊的密码，因为我有邮箱独立密码，所以只能使用qq邮箱额外提供的');
$mail->sendMail('xxx@qq.com','xxxx@qq.com','hi','嘿嘿嘿');
```
因为smtp可以在telnet中模拟，所以想着去模拟一个请求去访问redis也应该是可以的，在没有装redis扩展或者composer依赖的时候，简单的请求可以直接用socket实现
```
<?php
/**
 * Created by PhpStorm.
 * User: eric
 * Date: 15/12/24
 * Time: 下午2:15
 */
$socket = fsockopen('127.0.0.1',6379);
fwrite($socket,"ping\r\n");
$response = fgets($socket);
echo "<pre>";print_r($response);
fwrite($socket,"set a aaa\r\n ");
$response = fgets($socket);
echo "<pre>";print_r($response);
fwrite($socket,"get a\r\n");
$response = fgets($socket);
echo "<pre>";print_r($response);

```
接下来尝试下用php socket搭建一个煎蛋代理服务(因为之前转过了一篇node)





