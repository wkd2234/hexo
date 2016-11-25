layout: '[post]'
title: php的单例模式
date: 2016-11-25 09:30:27
tags:
---
php的单例模式依靠用<b>类的静态变量</b>来存储类本身的一个<b>实例</b>来做到:
* 这个对象能被系统中的任何对象使用且不存在于<b>全局作用域</b>
* 这个类最多只能实例<b>一个</b>对象
{% codeblock lang:php %}
<?php
class Single{
    private $name;
    private static $single;
    
    private function __construct(){}
    
    public function getSingle(){
        if(empty($single)){
            self::$single = new Single();
        }
        return self::$single;
    }
    
    public function setName($name){
        $this->name = $name;
    }
    
    public function getName(){
        return $this->name;
    }
}

$sing = Single::getSingle();
$sing->setName('wkd');
echo $sing->getName()."\n";  //wkd
$sing2 = Single::getSingle();
echo $sing->getName()."\n"; //wkd
{% endcodeblock %}