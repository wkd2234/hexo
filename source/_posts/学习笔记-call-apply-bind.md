layout: '[post]'
title: '学习笔记--bind'
date: 2016-11-05 12:45:30
tags:
---
    在接触bind前,我经常使用类似传参相似的call,apply。而我对call,apply的理解是:用某个方法代理执行call里指定的作用域与参数。
    比较常见的代码就是
{% codeblock lang:javascript %}
    var SLICE = Array.prototype.slice;
    var str = "abc";

    console.log(SLICE.call(str,0)) // ["a","b","c"]
{% endcodeblock %}
    很直白,原本String不能调用属于Array的slice方法,但是我们知道String类型可以像Array一样通过下标来取值,所以可以让slice来代理操作String。
    然而我一直都没有去使用过bind,只知道它的语法与call,apply类似
{% codeblock lang:javascript %}
    fun.bind(thisArg[, arg1[, arg2[, ...]]])
{% endcodeblock %}
    然而在第一次使用时发现,bind其实是在对某个方法bind了作用域与参数后的返回,也就是说bind方法返回一个作用域与参数被绑定的方法
{% codeblock lang:javascript %}
    var add = function(a,b,c){
        return a+b+c
    }
    var fn = add.bind(this,1) //这里可以理解成fn就变成了 function(b,c){
                              //                           return 1+b+c
                              //                       }
{% endcodeblock %}
像这样的话,我们不就可以写出像这样的代码:
{% codeblock lang:javascript %}
    var add = function(a,b,c){
        return a+b+c
    }
    var curry = function(fn){       //函数式编程中的库里化
        //...
    }
    var fn = curry(add);
    fn(1)(2)(3) // =>6
{% endcodeblock %}
所以bind在函数式编程中还是经常需要使用,以上也都是我在nodeschool的项目中学习functional的收获。
简单curry的实现:
{% codeblock lang:javascript %}
    var curry = function(fn,index){
        var len = index || fn.length;     //fn的形参个数
        return function(num){
            if(len === 1)
                return fn(num);        //当传入最后一个参数时,输出
            return curry(fn.bind(this,num),len-1) //对fn进行绑定
        }
    }
{% endcodeblock %}
