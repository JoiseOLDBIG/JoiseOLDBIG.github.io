---
layout: post
title:  "js如何区分变量类型"
date:   2018-03-05 16:13:00 +0800
categories: post
---

javascript的变量类型大致分两种，一种是基本数据类型，包括Undefined、Null、String、Number、Boolean这5中；另一种是复杂数据类型，Object，当然Object还细分很多具体类型，比如Array、Function、Data等。

首先定义以下变量：

        var num = 123;
        var str = 'abcdef';
        var bool = true;
        var arr = [1, 2, 3, 4];
        var json = {name:'wenzi', age:25};
        var func = function(){ console.log('this is function'); }
        var und = undefined;
        var nul = null;
        var date = new Date();
        var reg = /^[a-zA-Z]{5,20}$/;
        var error= new Error();

区分这些变量类型主要有以下几个方法：

### 1.typeof

测试代码如下：

        console.log(
            typeof num, 
            typeof str, 
            typeof bool, 
            typeof arr, 
            typeof json, 
            typeof func, 
            typeof und, 
            typeof nul, 
            typeof date, 
            typeof reg, 
            typeof error
        );
        //number string boolean object object function undefined object object object object

可以看出typeof只能区分Number、String、Boolean和Undefined，而其他类型的变量则都被认为是object。

### 2.instanceof

测试代码如下： 

        console.log(
            num instanceof Number,    //false
            str instanceof String,   //false
            bool instanceof Boolean,   //false
            arr instanceof Array,   //true
            json instanceof Object,   //true
            func instanceof Function,   //true
            und instanceof Object,   //false
            nul instanceof Null,   //false
            date instanceof Date,   //true
            reg instanceof RegExp,   //true
            error instanceof Error   //true
        )
        //false false false true true true false false true true true


可以看出instanceof可以区分Array、Json、Function、Date、RegExp、Error；
无法区分Undefined和Null，以及Number、String、Boolean。
假设用new的方法创建Number、String、Boolean，再看看看是否能被检查出来。

        var num = new Number('123');
        var str = new String('abcdef');
        var bool = new Boolean(true);
        
        console.log(
            num instanceof Number,   //true
            str instanceof String,   //true
            bool instanceof Boolean   //true
        );
        
        //如果用typeOf， 返回的则都为object
        console.log(
            typeof num, 
            typeof str, 
            typeof bool, 
        );
        // object object object

由此可知instanceof可以识别new出来的Number、String、Boolean，而不能识别字面量写法的Number、String、Boolean。typeof则相反。

### 3.constructor

constructor本来是原型对象上的属性，指向构造函数。但是根据实例对象寻找属性的顺序，若实例对象上没有属性或者方法时，就去原型链上寻找。因此，实例对象也是能使用constructor属性的。

    function Person(){
     
    }
    var Tom = new Person();
     
    // undefined和null没有constructor属性
    console.log(
        Tom.constructor==Person,
        num.constructor==Number,
        str.constructor==String,
        bool.constructor==Boolean,
        arr.constructor==Array,
        json.constructor==Object,
        func.constructor==Function,
        date.constructor==Date,
        reg.constructor==RegExp,
        error.constructor==Error
    );
    // true true true true true true true true true true

通过constructor方法，我们可以区分Number、String、Boolean、Array、Object、Function、Date、RegExp和Error。Undefined和Null因为没有constructor属性所以不能用constructor进行区分。

不过使用constructor也不是保险的，因为constructor属性是可以被修改的，会导致检测出的结果不正确，例如：

        function Person(){
         
        }
        function Student(){
         
        }
        Student.prototype = new Person();
        var John = new Student();
        console.log(John.constructor==Student); // false
        console.log(John.constructor==Person); // true

上述例子中，由于Student继承了Person，constructor被修改指向了Person，导致检测不是John的真实构造函数。

同时，使用instaceof和construcor,被判断的array必须是在当前页面声明的！比如，一个页面（父页面）有一个框架，框架中引用了一个页面（子页面），在子页面中声明了一个array，并将其赋值给父页面的一个变量，这时判断该变量，Array == object.constructor;会返回false； 原因：

1、array属于引用型数据，在传递过程中，仅仅是引用地址的传递。

2、每个页面的Array原生对象所引用的地址是不一样的，在子页面声明的array，所对应的构造函数，是子页面的Array对象；父页面来进行判断，使用的Array并不等于子页面的Array；切记，不然很难跟踪问题！

### 4.Object.prototype.toString.call

测试代码如下：

        console.log(
            Object.prototype.toString.call(num),
            Object.prototype.toString.call(str),
            Object.prototype.toString.call(bool),
            Object.prototype.toString.call(arr),
            Object.prototype.toString.call(json),
            Object.prototype.toString.call(func),
            Object.prototype.toString.call(und),
            Object.prototype.toString.call(nul),
            Object.prototype.toString.call(date),
            Object.prototype.toString.call(reg),
            Object.prototype.toString.call(error)
        );
        //[object Number] [object String] [object Boolean] [object Array] [object Object] [object Function] [object Undefined] [object Null] [object Date] [object RegExp] [object Error]

可以看出Object.prototype.toString.call完美得区分了Number、Str、Boolean、Array、Object、Function、Undefined、Null、Date、RegExp和Error。

我们现在再来看看ECMA里是是怎么定义Object.prototype.toString.call的：

代码如下:

        Object.prototype.toString( ) When the toString method is called, the following steps are taken: 
        1. Get the [[Class]] property of this object. 
        2. Compute a string value by concatenating the three strings “[object “, Result (1), and “]”. 
        3. Return Result (2)


上面的规范定义了Object.prototype.toString的行为：

首先，取得对象的一个内部属性[[Class]]，

然后依据这个属性，返回一个类似于”[object Array]”的字符串作为结果（看过ECMA标准的应该都知道，[[]]用来表示语言内部用到的、外部不可直接访问的属性，称为“内部属性”）。

利用这个方法，再配合call，我们可以取得任何对象的内部属性[[Class]]，然后把类型检测转化为字符串比较，以达到我们的目的。


### 四种方法比较

| 类型\判断方法   | typeof | instanceof | constructor | Object.prototype.toString.call  |
|:-------------   | :----- |  :-------- | : --------- | : ---------                     |
| Number(字面量)  |  number  |  false     | true        |  ture             |
| String(字面量)  |  string  |  false     | true        |  ture             |
| Boolean(字面量) |  boolean  |  false     | true        |  ture             |
| Number(new方法) |  object  |  true     | true        |  ture             |
| String(new方法) |  object  |  true     | true        |  ture             |
| Boolean(new方法)|  object  |  true     | true        |  ture             |
| Array           |  object  |  true     | true        |  ture             |
| Object          |  object  |  true     | true        |  ture             |
| Fucntion        |  function  |  true   | true        |  ture             |
| Undefined       |  undefined |  -     | -       |  ture             |
| Null            |  object  |  -        | -       |  ture             |
| Date            |  object  |  true     | true        |  ture             |
| RegExp          |  object  |  true     | true        |  ture             |
| Error           |  object  |  true     | true        |  ture             |
| 优点            |  使用简单  |  能检测出复杂的类型     | 能检测出复杂的类型        |  能检测出所有类型             |
| 缺点            |  检测出的类型比较少  |  字面量基本类型检测不出，不能跨ifram     | 不能跨iframe，且constructor容易被修改        |  IE6下undefined和null均为object             |