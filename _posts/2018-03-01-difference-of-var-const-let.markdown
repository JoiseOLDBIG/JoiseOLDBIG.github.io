---
layout: post
title:  "var、const、let的区别"
date:   2018-03-01 19:57:11 +0800
categories: post
---

let和const都是ES6中定义的变量声明，通过与var的比较，我们将更了解let和const。

&nbsp;
### 1.作用域

`var只有全局作用域和函数作用域，没有块级作用域。`

`let和const是块级作用域。`

另外，for循环还有一个特别之处，就是设置循环变量的那部分是一个父作用域，而循环体内部是一个单独的子作用域。

        for (let i = 0; i < 3; i++) {
          let i = 'abc';
          console.log(i);
        }
        // abc
        // abc
        // abc

上面代码正确运行，输出了3次abc。这表明函数内部的变量i与循环变量i不在同一个作用域，有各自单独的作用域。

&nbsp;
### 2.变量提升

`var会发生变量提升现象，即变量可以在申明之前使用，值为undefined。`

`let和const不存在变量提升，声明的变量一定要在声明后使用，否则报错。`

&nbsp;
### 3.let、const不能重复声明

&nbsp;
### 4.const声明一个只读常量。一旦申明，就不能修改，切申明时必须初始化。

`const实际上保证的，并不是变量的值不得改动，而是变量指向的那个内存地址不得改动。对于简单类型的数据（数值、字符串、布尔值），值就保存在变量指向的那个内存地址，因此等同于常量。但对于复合类型的数据（主要是对象和数组），变量指向的内存地址，保存的只是一个指针，const只能保证这个指针是固定的，`至于它指向的数据结构是不是可变的，就完全不能控制了。因此，将一个对象声明为常量必须非常小心。

        const foo = {};
        
        // 为 foo 添加一个属性，可以成功
        foo.prop = 123;
        foo.prop // 123
        
        // 将 foo 指向另一个对象，就会报错
        foo = {}; // TypeError: "foo" is read-only

上面代码中，常量foo储存的是一个地址，这个地址指向一个对象。不可变的只是这个地址，即不能把foo指向另一个地址，但对象本身是可变的，所以依然可以为其添加新属性。

下面是另一个例子。

        const a = [];
        a.push('Hello'); // 可执行
        a.length = 0;    // 可执行
        a = ['Dave'];    // 报错

上面代码中，常量a是一个数组，这个数组本身是可写的，但是如果将另一个数组赋值给a，就会报错。

如果真的想将对象冻结，应该使用Object.freeze方法。

        const foo = Object.freeze({});

        // 常规模式时，下面一行不起作用；
        // 严格模式时，该行会报错
        foo.prop = 123;

上面代码中，常量foo指向一个冻结的对象，所以添加新属性不起作用，严格模式时还会报错。

除了将对象本身冻结，对象的属性也应该冻结。下面是一个将对象彻底冻结的函数。

        var constantize = (obj) => {
          Object.freeze(obj);
          Object.keys(obj).forEach( (key, i) => {
            if ( typeof obj[key] === 'object' ) {
              constantize( obj[key] );
            }
          });
        };

&nbsp;
### 5.全局变量

* 在游览器里面，顶层对象是window对象，但Node和Web Worker没有window。
* 在游览器和Web Workder里面，self也指向顶层对象，但是Node没有self。
* 在 Node 里面，顶层对象是global对象，但其他环境都不支持。

综上所述，很难找到一种方法，可以在所有情况下，都取到顶层对象。下面是两种勉强可以使用的方法。

        // 方法一
        (typeof window !== 'undefined'
           ? window
           : (typeof process === 'object' &&
              typeof require === 'function' &&
              typeof global === 'object')
             ? global
             : this);
        
        // 方法二
        var getGlobal = function () {
          if (typeof self !== 'undefined') { return self; }
          if (typeof window !== 'undefined') { return window; }
          if (typeof global !== 'undefined') { return global; }
          throw new Error('unable to locate global object');
        };
 
ES5 之中，顶层对象的属性与全局变量是等价的。

`var命令和function命令声明的全局变量，依旧是顶层对象的属性；`

`另一方面，let命令、const命令、class命令声明的全局变量，不属于顶层对象的属性。`

&nbsp;
### 6.let和const的暂时性死区

只要块级作用域内存在let命令，它所声明的变量就“绑定”（binding）这个区域，不再受外部的影响。

        var tmp = 123;
            
        if (true) {
          tmp = 'abc'; // ReferenceError
          let tmp;
        }

上面代码中，存在全局变量tmp，但是块级作用域内let又声明了一个局部变量tmp，导致后者绑定这个块级作用域，所以在let声明变量前，对tmp赋值会报错。

ES6 明确规定，如果区块中存在let和const命令，这个区块对这些命令声明的变量，从一开始就形成了封闭作用域。凡是在声明之前就使用这些变量，就会报错。

总之，在代码块内，使用let命令声明变量之前，该变量都是不可用的。这在语法上，称为“暂时性死区”（temporal dead zone，简称 TDZ）。

        if (true) {
          // TDZ开始
          tmp = 'abc'; // ReferenceError
          console.log(tmp); // ReferenceError
        
          let tmp; // TDZ结束
          console.log(tmp); // undefined
        
          tmp = 123;
          console.log(tmp); // 123
        }

上面代码中，在let命令声明变量tmp之前，都属于变量tmp的“死区”。

“暂时性死区”也意味着typeof不再是一个百分之百安全的操作。

        typeof x; // ReferenceError
        let x;

上面代码中，变量x使用let命令声明，所以在声明之前，都属于x的“死区”，只要用到该变量就会报错。因此，typeof运行时就会抛出一个ReferenceError。

作为比较，如果一个变量根本没有被声明，使用typeof反而不会报错。

        typeof undeclared_variable // "undefined"

上面代码中，undeclared_variable是一个不存在的变量名，结果返回“undefined”。所以，在没有let之前，typeof运算符是百分之百安全的，永远不会报错。现在这一点不成立了。这样的设计是为了让大家养成良好的编程习惯，变量一定要在声明之后使用，否则就报错。

有些“死区”比较隐蔽，不太容易发现。

        function bar(x = y, y = 2) {
          return [x, y];
        }
        
        bar(); // 报错

上面代码中，调用bar函数之所以报错（某些实现可能不报错），是因为参数x默认值等于另一个参数y，而此时y还没有声明，属于”死区“。如果y的默认值是x，就不会报错，因为此时x已经声明了。

        function bar(x = 2, y = x) {
          return [x, y];
        }
        bar(); // [2, 2]
        另外，下面的代码也会报错，与var的行为不同。
        
        // 不报错
        var x = x;
        
        // 报错
        let x = x;
        // ReferenceError: x is not defined

上面代码报错，也是因为暂时性死区。使用let声明变量时，只要变量在还没有声明完成前使用，就会报错。上面这行就属于这个情况，在变量x的声明语句还没有执行完成前，就去取x的值，导致报错”x 未定义“。

ES6 规定暂时性死区和let、const语句不出现变量提升，主要是为了减少运行时错误，防止在变量声明前就使用这个变量，从而导致意料之外的行为。这样的错误在 ES5 是很常见的，现在有了这种规定，避免此类错误就很容易了。

总之，暂时性死区的本质就是，只要一进入当前作用域，所要使用的变量就已经存在了，但是不可获取，只有等到声明变量的那一行代码出现，才可以获取和使用该变量。



