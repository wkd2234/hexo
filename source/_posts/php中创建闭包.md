layout: '[post]'
title: php中创建闭包
date: 2016-11-24 15:49:52
tags:
---
{% codeblock lang:php %}
<?php
function createClosure(){
    $i = 1;
    return function() use ($i){
        echo ++$i;
    };
};
$closure = createClosure();
$closure(); //echo 2
{% endcodeblock %}