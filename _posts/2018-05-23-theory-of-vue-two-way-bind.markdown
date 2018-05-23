---
layout: post
title: "vue双向绑定原理"
date: 2018-05-23 20:22:00 +0800
categories: post
---


这篇文章主要是来扒一扒VUE的双向绑定原理。

vue是属于MVVM（modal-view-viewmodal）模型，它其中一大特点就是view和viewmodal的双向绑定。

vue的双向绑定的做法主要是数据劫持和发布者-订阅者模式结合。

### 一、主要用到的知识点

#### documentFragment：

documentFragment主要就是用来数据劫持。用白话说就是：根据id将一个根节点下的所有子孙节点都劫持到一个新建的documentFragment中，然后再将documentFragment整体挂载到根节点下。劫持的过程中可以做一些操作，比如编译HTML（后面细说）。

```
var element = document.createDocumentFragment();
var node = document.getElementById("app");
var child;
while(child = node.firstChild) {
    //do sth compile
    element.appendChild(child); 
    //appendChild()会将child移到element，而不是复制，所以这个while不是死循环哈
}
//循环完之后，node已经没有子孙节点了
document.getElementById("app").appendChild(element);
```

#### Object.defineProperty(obj, val, des):

设置data中的数据的访问器属性，访问器属性是一种特殊的属性，它不能直接在对象中设置，必须通过Object.defineProperty()来实现。
``` 
    var data = {
        a: 'test',
    };
    Object.defineProperty(data, 'a', {
        get: function() {
        //do sth observe
         return sth;
        },
        set: function(val) {
            //do sth observe
        }
    });
```

get和set内部的this都指向data。

当读取data.a的时候就会调用get函数；当给data.a赋值的时候就会调用set函数。因此当熟读data.a或者复制data.a的时候可以监听（observe）到。

#### 发布者-订阅者模式：发布者publisher和订阅者subscriber是一对多的关系。

### 二、双向绑定的实现

#### viewmodal -> view:

上面知识点中有说到数据劫持，并且在数据劫持的过程中，可以做一些编译（compile），主要是两个任务：

1）数据初始化绑定：页面首次打开之后将data渲染到HTML；

2）数据响应式绑定：用defineProperty()给data设置访问器属性，当data中的属性被赋值时，就会触发set方法，对data进行监听。

```
function Vue(option) {
    //数据响应式绑定，observe监听
    this.data = option.data;
    var data = this.data;
    observe(data, this);
    //数据初始化绑定，编译compile
    var id = option.el;
    var node = document.getElementById(id);
    nodeToFragment(node, this);
}

//数据初始化绑定
function nodeToFragment(node, vm) {
    var element = document.createDocumentFragment();
    var node = document.getElementById("app");
    var child;
    while(child = node.firstChild) {
        compile(child, vm);
        element.appendChild(child); 
        //appendChild()会将child移到element，而不是复制，所以这个while不是死循环哈
    }
    //循环完之后，node已经没有子孙节点了
    document.getElementById("app").appendChild(element);
}
function compile(node, vm) {
    if(node.nodeType == 1) {//表示是element
        let attrs = node.attributes;//获取node的属性
        for(let i = 0; i < attrs.length; i ++) {
            if(attrs[i].nodeName == 'v-model') {
                let name = attrs[i].nodeValue;
                node.value = vm.data[name];      //渲染element
                node.removeAttribute('v-model');
            }
        }
    }
    if(node.nodeType == 3) {//表示是text
        let reg = /\{\{(.*)\}\}/;
        if(reg.test(node.nodeValue)) {
            let name = RegExp.$1;      //获取reg第一个括号中匹配到数据
            name = name.trim();
            node.nodeValue = vm.data[name];  //渲染text

        }
    }
}

//数据响应式绑定
function observe(obj, vm) {
    Object.keys(obj).forEach(function(key) {
        defineReactive(vm.data, key, obj[key]);
    })
}
function defineReactive(obj, key, val) {
    Object.defineProperty(obj, key, {
        //...
        set: function(newVal) {
            if(newVal == val) 
                return;
            val = newVal;
            console.log(obj + ':' + key + '被更新了');
            //通知更新试图
        }
    })
}
```

<font color=red>触发set方法之后，如何响应到文本节点？</font>

这里用到了上文所讲的知识点之一————发布者-订阅者模式。set方法触发后，第二件事就是作为发布者发通知，告知订阅者“我变了”，文本节点就是订阅者，收到消息之后，更新试图。

发布者和订阅者是一对多的关系。

```
function Dep() {
    this.subs = [];//订阅者数组
}

Dep.prototype = {
    addSub: function(sub) {  //添加订阅者
        this.subs.push(sub);
    },
    notify: function() {         //发布者通知订阅者更新
        this.subs.forEach(function(sub) {
            sub.update();
        })
    }
}
```

接下来要做的就是在合适的时候添加订阅者，合适的时候自然是get方法中；然后在set方法中添加notify方法，当set被触发之后通知订阅者进行更新试图。

```
function defineReactive(obj, key, val) {
    var dep = new Dep();
    Object.defineProperty(obj, key, {
        //...
        get: function() {
            if(Dep.target) {//Dep.target是全局变量，当前订阅者
                dep.addSub(Dep.target);  //添加订阅者
            }
            return val;
        },
        set: function(newVal) {
            if(newVal == val) 
                return;
            val = newVal;
            dep.notify();//发布者发出通知，dep对象收到通知后推送给订阅者
        }
    })
}
```

Dep.target是什么呢？Dep.target是一个全局变量，会把当前订阅者复制给Dep.target，以便在get方法中调用。

订阅者Watcher

```
function Watcher(vm, node, name) {
    Dep.target = this; //将自己复制给Dep.target;
    this.vm = vm;
    this.node = node;
    this.name = name;
    this.update();//添加订阅者和更新订阅者value
    Dep.target = null; //将全局变量Dep.target赋值为空
}
Watcher.prototype = {
    update: function() {
        this.get();
        this.node.nodeValue = this.value;//
    },
    get: function() {
        this.value = this.vm.data[this.name];//触发get方法
    }
}

function compile(node, vm) {
    //...
    if(node.nodeType == 3) {//表示是text
        let reg = /\{\{(.*)\}\}/;
        if(reg.test(node.nodeValue)) {
            let name = RegExp.$1;      //获取reg第一个括号中匹配到数据
            name = name.trim();
            //node.nodeValue = vm.data[name];  //渲染text

            new Watcher(vm, node, name);

        }
    }
}
```

#### view -> viewmodal

指的是input，select，textarea文本框内的值被改变之后，也要同步到data中的数据。

解决办法就是在数据劫持的时候，给每个节点添加监听，这一步也是在compile中实现。

```
function compile(node, vm) {
    //...
    if(node.nodeType == 1) {//表示是element
        let attrs = node.attributes;//获取node的属性
        for(let i = 0; i < attrs.length; i ++) {
            if(attrs[i].nodeName == 'v-model') {
                let name = attrs[i].nodeValue;
                //node.value = vm.data[name];      //渲染element

                new Watcher(vm, node, name);
                node.removeAttribute('v-model');

                node.addEventListener('keydown', function(e) {
                    vm.data[name] = node.value;
                })
            }
        }
    }
}
```

这样就实现了view <-> viewmodal的双向绑定了。

整体代码↓↓↓亲测可行，不过这只是简易版的双向绑定，实际会复杂很多。
```
<html>
<head>
</head>
<body>
<div id="app">
<input type="text" v-model="text">
{{text}}
</div>
<script type="text/javascript">
function Dep() {
    this.subs = [];//订阅者数组
}
Dep.prototype = {
    addSub: function(sub) {  //添加订阅者
        this.subs.push(sub);
    },
    notify: function() {         //发布者通知订阅者更新
        this.subs.forEach(function(sub) {
            sub.update();
        })
    }
}

function Watcher(vm, node, name) {
    Dep.target = this; //将自己复制给Dep.target;
    this.vm = vm;
    this.node = node;
    this.name = name;
    this.update();//添加订阅者和更新订阅者value
    Dep.target = null; //将全局变量Dep.target赋值为空
}
Watcher.prototype = {
    update: function() {
        this.get();
        this.node.nodeValue = this.value;//
    },
    get: function() {
        this.value = this.vm.data[this.name];//触发get方法
    }
}

function Vue(option) {
    //数据响应式绑定，observe监听
    this.data = option.data;
    var data = this.data;
    observe(data, this);
    //数据初始化绑定，编译compile
    var id = option.el;
    var node = document.getElementById(id);
    nodeToFragment(node, this);
}

//数据初始化绑定
function nodeToFragment(node, vm) {
    var element = document.createDocumentFragment();
    var node = document.getElementById("app");
    var child;
    while(child = node.firstChild) {
        compile(child, vm);
        element.appendChild(child); 
        //appendChild()会将child移到element，而不是复制，所以这个while不是死循环哈
    }
    //循环完之后，node已经没有子孙节点了
    document.getElementById("app").appendChild(element);
}
function compile(node, vm) {
    if(node.nodeType == 1) {//表示是element
        let attrs = node.attributes;//获取node的属性
        for(let i = 0; i < attrs.length; i ++) {
            if(attrs[i].nodeName == 'v-model') {
                let name = attrs[i].nodeValue;
                node.value = vm.data[name];      //渲染element
                node.removeAttribute('v-model');

                node.addEventListener('keyup', function(e) {//对文本节点进行更新
                    vm.data[name] = node.value;
                })
            }
        }
    }
    if(node.nodeType == 3) {//表示是text
        let reg = /\{\{(.*)\}\}/;
        if(reg.test(node.nodeValue)) {
            let name = RegExp.$1;      //获取reg第一个括号中匹配到数据
            name = name.trim();
            //node.nodeValue = vm.data[name];  //渲染text

            new Watcher(vm, node, name);

        }
    }
}

//数据响应式绑定
function observe(obj, vm) {
    Object.keys(obj).forEach(function(key) {
        defineReactive(vm.data, key, obj[key]);
    })
}
function defineReactive(obj, key, val) {
    var dep = new Dep();
    Object.defineProperty(obj, key, {
        //...
        get: function() {
            if(Dep.target) {//Dep.target是全局变量，当前订阅者
                dep.addSub(Dep.target);  //添加订阅者
            }
            return val;
        },
        set: function(newVal) {
            if(newVal == val) 
                return;
            val = newVal;
            console.log('我被更新了');
            dep.notify();//发布者发出通知，dep对象收到通知后推送给订阅者
        }
    })
}


var vm = new Vue({
    el: 'app',
    data: {
        text: 'aaa',
    }
});


</script>
</body>
</html>
```

[参考文章](http://www.cnblogs.com/kidney/p/6052935.html?utm_source=gold_browser_extension)



























