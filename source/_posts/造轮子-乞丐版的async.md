layout: '[post]'
title: 造轮子--乞丐版的async
date: 2016-11-14 14:48:36
tags:
---
{% codeblock lang:javascript %}
var myAsync = {};
module.exports = myAsync;
{% endcodeblock %}
{% codeblock lang:javascript mapLimit%}
/**
 * Example:
 * '''
 * function once(){
 * //....
 * }
 * myAsync.mapLimit(arr, num, function(item, callback){
 *     once(item, callback)
 *   },cb) 
 * '''
 */
myAsync.mapLimit = function(list, limit, iteratorFunction, callback){
    var _index = count = 0,
        result = [],
        sum    = list.length;

    function callbackOnce(err, res){
        if(err) return callback(err, result);
        if(result) result.push(res)
        
        if(_index >= sum) return callback(null, result)        
        if(--count > limit) return;

        ++count;
        return iteratorFunction(list[_index++], callbackOnce)
    }
    for(var i = 0; i < limit; i++){
        iteratorFunction(list[_index++], callbackOnce)
        count ++;
    }
}
{% endcodeblock %}