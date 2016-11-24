layout: '[post]'
title: 使用node.js爬取中关村壁纸站图片
date: 2016-11-13 15:26:18
tags:
---
练手项目是一个模仿中关村壁纸站的壁纸站，所以需要去原网站爬取图片。

首先就是先浏览下中关村壁纸站的文档结构：
{% img /img/crawel1.jpg %}
这里我就先只按着壁纸分类来爬取图片,点击分类栏内任意链接就会转到PC壁纸页,同时在选择区内多了壁纸子类这一分类,点击下面的壁纸集就跳转进详细页:
{% img /img/crawel2.jpg %}
{% img /img/crawel3.jpg %}
再点击下去就可以得出这样的数据结构:
{% codeblock %}
分类
-+-+子类
-+-+-+-+单个壁纸集
-+-+-+-+-+-+单个壁纸
-+-+-+-+-+-+-+-+不同分辨率
{% endcodeblock %}


{% codeblock 那我就把数据库的表字段大致设置成: %}
壁纸集 | 壁纸 | 分辨率 |
{% endcodeblock %} 

模拟一个http请求,这里我使用的是superagent,一个轻量易学的封装了各种http操作的模块

取得了分类列表的信息后,就要再取得每个分类下的子类,这里就要遍历一遍分类列表,而因为http请求是异步的,所以设置一个计数器来控制流程。

到了这一步,已经获取许多许多的壁纸集信息,接下里就要取得壁纸集下每张图片的信息,再来按照前面设置计数器的方法来做的就不能很好控制自己的并发数量,一次并发1000多条请求必然是不合理的。所以这里又引入了大名鼎鼎的流程控制模块async。

完整代码如下:
{% codeblock lang:javascript main.js %}
var zol_c = require('./crawel')
var events      = require('events')
var util       = require('util');
var doInsert = require('./db_function').doInsert;

//使用事件来控制流程
function Crawel(){
    events.EventEmitter.call(this)
}
util.inherits(Crawel, events.EventEmitter);

/**
 * 设置事件的响应
 * @param {Function} fn 执行函数
 * @param {Array}  args 执行函数所需的参数, 最后一位为下一步事件名称
 */
function emitEvent(fn, args){
    var _len = args.length,
        self = this;
    return (function bindEventOnce(i, fn){
        if(i === _len){
            return fn.bind(this,function(){
                return self.emit(args[i])
            })
        }
        bindEventOnce(i+1, fn.bind(this, task[i]))
    })(0, fn)
}

Crawel.prototype.startPC       = emitEvent(zol_c, ['pc', 'endPC']);
Crawel.prototype.startMobile   = emitEvent(zol_c, ['mobile', 'endMobile']);
Crawel.prototype.startIpad     = emitEvent(zol_c, ['ipad', 'endIpad']);
Crawel.prototype.startInsertZj = emitEvent(doInsert, ['bz_crawel', 'zj', 'zj', 'endInsertZj']);
Crawel.prototype.startInsertBz = emitEvent(doInsert, ['bz_crawel', 'bz', 'bz', 'endInsertBz']);
Crawel.prototype.startInsertWj = emitEvent(doInsert, ['bz_crawel', 'all', 'wj', 'endInsertWj']);

var crawel = new Crawel();

crawel.on('endPC',function(){
    crawel.startMobile()
})
crawel.on('endMobile',function(){
    crawel.startIpad()
})
crawel.on('endIpad',function(){
    crawel.startInsertZj()
})
crawel.on('endInsertZj',function(){
    crawel.startInsertBz()
})
crawel.on('endInsertBz',function(){
    crawel.startInsertWj()
})
crawel.on('endInsertWj', function(){
    process.exit()   
})

crawel.startInsertWj();
{% endcodeblock %}