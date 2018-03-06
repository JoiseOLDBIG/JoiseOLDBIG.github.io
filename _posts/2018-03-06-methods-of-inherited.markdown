---
layout: post
title: "浅谈js继承方法"
date: 2018-03-06 17:43:00 +0800
categories: post
---

js的继承可以分成两大类：构造函数的继承和非构造函数的继承。

### 构造函数的继承

比如，现在有一个"动物"对象的构造函数。

    　　function Animal(){
    　　　　this.species = "动物";
    　　}

还有一个"猫"对象的构造函数。

    　　function Cat(name,color){
    　　　　this.name = name;
    　　　　this.color = color;
    　　}

怎样才能使"猫"继承"动物"呢？

#### 1.构造函数绑定

一般是用call和apply，将父对象的构造函数绑定到子对象上：

        function Cat(name,color) {
            `Animal.apply(this, arguments);`
            this.name = name;
            this.color = color;
        }


#### 2.prototype模式

        Cat.prototype = new Animal();
        //将Cat的原型指向Animal实例。
        Cat.prototype.constructor = Cat;

如果不写 Cat.prototype.constructor = Cat 这一行，Cat.prototype的constructor将会指向Animal。

当新建一个cat1,cat1的constructor也指向Animal。
    
        var cat1 = new Cat();
        console.log(cat1.constructor == Animal); //true

#### 3.直接继承prototype

        Cat.prototype = Animal.prototype;
        //优点是不用再创建Animal实例
        Cat.prototype.constructor = Cat;
        //缺点是，上一句把Animal.prototype的constructor也指向了Cat

#### 4.利用空对象作为中介

        var F = function(){};
        F.prototype = Animal.prototype;
        Cat.prototype = new F(); //F是空对象，所以几乎不占库存
        Cat.prototype.constructor = Cat; 
        //这时修改Cat的prototype对象不会影响到Animal

封装方法：

        function extend(Child, Parent) {
            var F = function(){};
            F.prototype = Parent.prototype;
            Child.prototype = new F();
            Child.prototype.constructor = Cat;
            Child.uber = Parent.prototype;
        }

#### 5.拷贝继承

封装方法：

        function extend2(Child, Parent) {
            var p = Parent.prototype;
            var c = Child.prototype;
            for (let i in p) {
                c[i] = p[i];
            }
            c.uber = p;
        }

&nbsp;
### 非构造函数的继承

比如，现在有一个对象，叫做"中国人"。

    　　var Chinese = {
    　　　　nation:'中国'
    　　};

还有一个对象，叫做"医生"。

    　　var Doctor ={
    　　　　career:'医生'
    　　}

#### 1.object()方法

        function object(o) {
            funtion F() {}
            F.prototype = o;//子对象的prototype指向父对象
            return new F();
        }

        var Docotor = object(Chinese);
        Docotor.career = 'docotor';

#### 2.浅拷贝

        function extendCopy(p) {
            var c = {};
            for(let i in p) {
                c[i] = p[i];
            }
            c.uber = p;
            return c;
        }

如果p中有object或者array，那子对象copy的只是父对象的引用地址。如果修改子对象中的某个对象属性，会把父对象的这个属性一起修改。

#### 3.深拷贝

        function deepCopy(p, c) {
            c = c || {};
            for(let i in p) {
                if(typeof p[i] === 'object') {
                    c[i] = p[i] instanceof Array ? [] : {};
                    deepCopy(p[i], c[i]);
                }else {
                    c[i] = p[i]
                }
            }
        }