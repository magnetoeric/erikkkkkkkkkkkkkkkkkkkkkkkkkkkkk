title: 单例模式 
tags:
  - singleton
category: patterns
date: 2015-07-18 19:02:19

---
## 单例模式
经过良好设计的系统一般通过方法调用传递对象实例。通常单例使用与如下背景环境：  
1.对象会被系统中的任何对象使用  
2.对象不应该储存在会被修改的全局变量中  
3.系统中不应该超过一个该对象，也就是说Y设置了X对象的属性，Z可以不通过其他对象，直接获取到该属性的值  

因此，单例对象的构造方法应该是私有的  

```
class Singleton{
	private $props = array();
	private function __construct(){
	}
	public function setProperty($key,$value){
	$this->props[$key] = $value;
	}
	public function getProperty($key){
	return $this->pros[$key];
	}
}
```

但是这样是不能用的，因为构造方法是私有的。所以需要静态方法和静态属性来间接完成实例化。  
 
```
class Singleton{
	private $props = array();
	private  static $instance;
	private function __construct(){
	}
	public function setProperty($key,$value){
	    $this->props[$key] = $value;
	}
	public function getProperty($key){
	    return $this->props[$key];
	}
	public static function getInstance(){
	if(empty(self::$instance)){
		self::$instance = new Singleton();
	}
	return self::$instance;
	}
}
```

单例模式和全局变量依然会被误用，因为它可以在任何地方被使用，所以很难调试它的依赖关系（比如公司的框架，对pdo进行了封装，而在对某些语句进行跟踪调试时，就变的困难了，因为不只是一处去调用该模块）
关于单例模式，java中有很多种实现方式，每一种都有自己的优缺点，另外java中有时候单例模式有时候会失效，这些都是语言层级了，就不多写了