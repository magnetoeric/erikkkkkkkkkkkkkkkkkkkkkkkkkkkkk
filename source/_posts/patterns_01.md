title: 观察者模式
tags:
  - patterns
categories:
    - patterns
date: 2017-04-14 22:35:00

---
Spl中提供了两个接口来实现观察者模式
Subject需要实现3个具体方法,包括添加观察者，删除观察者，通知观察者
Observer需要实现一个接口，更新

```
class ConcertSubject implements SplSubject{
    private $obeservers;

    private $value;

    public function __construct()
    {
        $this->obeservers = new SplObjectStorage();
    }

    public function attach (SplObserver $observer){
        $this->obeservers->attach($observer);
    }


    public function detach (SplObserver $observer){
        $this->obeservers->detach($observer);
    }

    public function notify (){
        foreach ($this->obeservers as $obeserver){
            $obeserver->update($this);
        }
    }
    public function setValue($value){
        $this->value = $value;
        $this->notify();
    }
    public function getValue(){
        return $this->value;
    }
}
class ConcertObserver implements SplObserver{

    public function update (SplSubject $subject){
            var_dump($subject->getValue());
    }
}

$subject = new ConcertSubject();
$observer =  new ConcertObserver();
$subject->attach($observer);
$subject->setValue(666);

```