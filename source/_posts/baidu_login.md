title: 百度 模拟登陆
tags:
  - php
categories:
    - php
date: 2017-04-23 11:35:00
---
# 百度 模拟登陆
最近闲着无聊 没事写点东西玩 
其实这种模拟登陆就是体力活，多次寻找，对比发送请求，找出'变量'和'常量',然后代码中对'变量'进行填充,就可以了。难解决的问题是验证码,ip等,这里不做扩展。
百度在首次访问时会随机生成一个uid,这个uid的算法是js生成的,具体可以在js文件看到，不过js文件是通过多层js引用加载出来的
登陆时会使用到rsa算法，也是使用js做的加密,pubkey是通过异步动态获取的,逻辑不少，多抓几次包差不多就了解个大概了，细心点就可以了
直接贴代码吧


```

<?php
/**
 * Created by PhpStorm.
 * User: eric
 * Date: 2017/4/22
 * Time: 下午2:17
 */
use \GuzzleHttp\Client;
require "vendor/autoload.php";
class HttpClient extends Client{
    private static $instance = null;
    public static function getInstance(){
        if(self::$instance ==null){
            self::$instance = new HttpClient(['cookies' => true]);
        }
        return self::$instance;
    }
}
function buildGid(){
    $ss = 'xxxxxxx-xxxx-4xxx-yxxx-xxxxxxxxxxxx';
    $len = strlen($ss);
    $result = '';
    for ($i = 0; $i < $len; $i++) {
        $result .= rep_sp($ss[$i]);
    }
    return $result;
}
function rep_sp($ns){
    $t = rand(0, 16);
    preg_match('/[xy]/',$ns,$result);
    if(empty($result) ){
        return $ns;
    }
    $n = $result ? $t:(3 & $t|8);
    return strtoupper(dechex($n) );
}
function getToken($gid,$tt){
    
    $callback = 'bd__cbs__3vybvq';
    $api = "https://passport.baidu.com/v2/api/?getapi&tpl=pp&apiver=v3&tt=$tt&class=login&gid=$gid&logintype=basicLogin&callback=$callback";
    $result = HttpClient::getInstance()->request('GET',$api);
    $t = substr($result->getBody(),strlen($callback)+1,-1);
    $t = str_replace("'","\"",$t);
    $t = json_decode($t,true);
    $token = $t['data']['token'];
    return $token;

}
function getRsaKey($token,$gid,$tt){
    $callback = 'bd__cbs__51lp3z';
    $api = "https://passport.baidu.com/v2/getpublickey?token=$token&tpl=pp&apiver=v3&tt=$tt&gid=$gid&callback=$callback";
    $result =  HttpClient::getInstance()->request('GET',$api);
    $t = substr($result->getBody(),strlen($callback)+1,-1);
    $t = str_replace("'","\"",$t);
    $t = json_decode($t,true);
    return array($t['key'],$t['pubkey']);
}
function rsaSign($str,$pubKey) {
    openssl_public_encrypt($str,$sign,$pubKey);
    return base64_encode($sign);
 }
function getMicroTime(){
    return intval(microtime(true)*1000);
}

$passort_api = "https://passport.baidu.com/v2/api/?login";
$password = 'xxxx';
$username = 'xxxx';
HttpClient::getInstance()->request('get','https://passport.baidu.com/v2/?login');
$random_time = getMicroTime();
$gid = buildGid();
$token = getToken($gid,$random_time);
$random_time = $random_time+rand(1,1000);
list($key,$pubKey) = getRsaKey($token,$gid,$random_time);
$sign = rsaSign($password,$pubKey);
$postData = array(
    'staticpage'=>'https://passport.baidu.com/static/passpc-account/html/v3Jump.html',
    'charset'=>'UTF-8',
    'token'=>$token,
    'tpl'=>'pp',
    'subpro'=>'',
    'apiver'=>'v3',
    'tt'=>time()*1000+123,
    'codestring'=>'',
    'safeflg'=>"0",
    'u'=>"https://passport.baidu.com/",
    'isPhone'=>0,
    'detect'=>"1",
    'gid'=>$gid,
    'quick_user'=>0,
    'logintype'=>'basicLogin',
     'logLoginType'=>'pc_loginBasic',
    'idc'=>'',
    'loginmerge'=>"true",
    'username'=>$username,
    'password'=>$sign,
    'crypttype'=>12,
     'ppui_logintime'=>rand(1000,10000),
    'countrycode'=>'',
    'rsakey'=>$key,
    'mem_pass'=>'on',
    'callback'=>'parent.bd__pcbs__l3z9y3'

);
$headers = array(
    'Accept'=>'text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8',
    'User-Agent'=> 'Mozilla/5.0 (X11; Ubuntu; Linux x86_64; rv:52.0) Gecko/20100101 Firefox/52.0',
    'Accept-Language'=>'zh-CN,zh;q=0.8,en-US;q=0.5,en;q=0.3',
    'Accept-Encoding'=>' deflate, br',
    'Referer'=> 'https://passport.baidu.com/v2/?login',
    'Content-Type'=>'application/x-www-form-urlencoded',
    'Host'=> 'passport.baidu.com',
    'Connection'=>'keep-alive',
    'Upgrade-Insecure-Requests'=>1,
    );



$aaa = HttpClient::getInstance()->request("POST",$passort_api,['form_params'=>$postData,'headers'=>$headers]);
echo "<pre>";print_r($aaa->headers());exit;

```
获取到BDUSS cookie即表明登陆成功
