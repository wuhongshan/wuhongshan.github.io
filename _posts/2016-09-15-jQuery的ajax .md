---
layout: post
title: 模拟jQuery的ajax
categories: [jQuery]
tags: [jQuery]
---


每次通过jQuery的AJAX获取接口，再渲染到页面上时，都暗自感慨AJAX的方便、强大。于是又复习了一遍AJAX的原理，模拟了一下jQuery的AJAX。代码如下：
```js
/*定义一个全局变量 $ */
window.$ = {};
/*声明一个ajax方法*/
/**
 * 方法需求分析：
 * =====请求=====
 * 1. 请求方式 type  get|post   默认 get
 * 2. 请求地址 url              默认 当前页面的地址
 * 3. 是否异步 async true|false 默认 true
 * 4. 传输数据 data  {}         默认 空对象{}
 *
 * =====响应=====
 * 5. 成功回调函数 success function 异步通讯成功做的事情
 * 6. 失败回调函数 error   function 异步通讯失败做的事情
 *
 * @param options
 * @returns {boolean}
 */
/*对象形式的传参方式*/
$.ajax = function (options) {
    /*参数判断*/
    if (!options || typeof options != 'object') {
        /*程序停止执行*/
        return false;
    }
    /*参数的默认设置*/
    /*请求方式*/
    var type = options.type || 'get';
    /*请求地址*/
    /*BOM window.location.pathname 当前页面的地址*/
    var url = options.url || location.pathname;
    /*是否异步*/
    var async = options.async === false ? false : true;
    /*传输数据*/
    var data = options.data || {};

    /*ajax原生 键值对 以&拼接*/
    var dataStr = '';
    /*遍历对象*/
    for (var key in data) {
        /*key=value*/
        dataStr += key + '=' + data[key] + '&';
    }
    /*去掉最后面的&*/
    /*dataStr.slice(0, -1) === dataStr.slice(0, dataStr.length - 1)*/
    dataStr = dataStr && dataStr.slice(0, -1);//截掉最后一个字符

    /*ajax编程*/
    var xhr = new XMLHttpRequest();

    /*设置请求行*/
    /*get url 拼接*/
    xhr.open(type, type == 'get' ? url + '?' + dataStr : url, async);
    /*设置请求头*/
    /*post 才需要设置*/
    if (type == 'post') {
        xhr.setRequestHeader('Content-Type', 'application/x-www-form-urlencoded');
    }
    /*设置请求内容*/
    xhr.send(type == 'post' ? dataStr : null);

    /*响应*/
    xhr.onreadystatechange = function () {
        /*判断通讯有没有完成*/
        if (xhr.readyState == 4) {
            /*判断请求的状态*/
            if (xhr.status == 200) {
                /*异步通讯成功做的事情*/
                /*按照约定过滤返回数据*/
                var contentType = xhr.getResponseHeader('Content-Type');
                /*返回数据*/
                var response = null;
                /*判断是不是xml*/
                if (contentType.indexOf('xml') > -1) {
                    /*xml 对象*/
                    response = xhr.responseXML;
                } else if (contentType.indexOf('json') > -1) {
                    /*json 对象*/
                    response = JSON.parse(xhr.responseText);
                } else {
                    response = xhr.responseText;
                }
                /*返回数据准备好了 返回给回掉函数*/
                options.success && options.success(response);

            } else {
                /*异步通讯失败做的事情*/
                var status = xhr.status;
                var statusText = xhr.statusText;
                options.error && options.error({code: status, message: statusText});
            }
        }
    };
};

/*get请求的ajax*/
$.get = function (options) {
    options.type = 'get';
    $.ajax(options);
};

/*post请求的ajax*/
$.post = function (options) {
    options.type = 'post';
    $.ajax(options);
};
```




