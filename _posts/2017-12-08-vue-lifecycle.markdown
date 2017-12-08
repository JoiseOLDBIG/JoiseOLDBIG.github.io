---
layout: post
title:  "vue的生命周期"
date:   2017-12-08 10:05:00 +0800
categories: post
---

#### **vue生命周期**

首先来看下摘自VUE中文官方文档的生命周期图（vue2.x）。

![png1]

然后在项目node_modules/vue/core/config.js文件中会发现以下这个生命周期钩子函数。

![png2]

我们将会对这些钩子函数进行测试，来检验这些函数是在什么情况下运行。
测试代码：

```javascript
  var vue = new Vue({
    data() {
        return {
            msg: 'data',
            arr: ['apple', 'banana', 'orange'],
        } 
    },
    methods: {
        test() {
            return 'methods';
        }
    },
    beforeCreate() {
        console.log('beforeCreate-start');
        console.log(this.msg, this.test);
        console.log(document.querySelectorAll('li').length);
        console.log('beforeCreate-end');
    },
    created() {
        console.log('created-start');
        const _self = this;
        console.log(this.msg, this.test);
        console.log(document.querySelectorAll('li').length);
        setTimeout(() => {
          _self.arr = ['12','14'];
          _self.$nextTick(() => {
            console.log('nextTick', document.querySelectorAll('li').length);
          })
        }, 0);
        setTimeout(() => {
          _self.arr = ['12','14','12','14','12','14','12','14'];
          _self.$nextTick(() => {
            console.log('nextTick', document.querySelectorAll('li').length);
          })
        }, 1000);
        console.log('created-end');
    },
    beforeMount() {
        console.log('beforeMount-start');
        console.log(this.msg, this.test);
        console.log(document.querySelectorAll('li').length);
        console.log('beforeMount-end');
    },
    mounted() {
        console.log('mounted-start');
        console.log(this.msg, this.test);
        console.log(document.querySelectorAll('li').length);
        console.log('mounted-end');
    },
    beforeUpdate() {
        console.log('beforeUpdate-start');
        console.log(this.msg, this.test);
        console.log(document.querySelectorAll('li').length);
        console.log('beforeUpdate-end');
    },
    updated() {
        console.log('updated-start');
        console.log(this.msg, this.test);
        console.log(document.querySelectorAll('li').length);
        console.log('updated-end');
    },
    beforeDestroy() {
        console.log('beforeDestroy-start');
        console.log(this.msg, this.test);
        console.log(document.querySelectorAll('li').length);
        console.log('beforeDestroy-end');
    },
    destroyed() {
        console.log('destroyed-start');
        console.log(this.msg, this.test);
        console.log(document.querySelectorAll('li').length);
        console.log('destroyed-end');
    },
    activated() {
        console.log('activated-start');
        console.log(this.msg, this.test);
        console.log(document.querySelectorAll('li').length);
        console.log('activated-end');
    },
    deactivated() {
        console.log('deactivated-start');
        console.log(this.msg, this.test);
        console.log('deactivated-end');
    }
  }).$mount('#root'); 
```

代码有点啰嗦哈，运行后在console中的输出为：

![png3]

![png4]

从而我们得出下表：

<table>
  <tr>
    <th>vue 2.x</th>
    <th>Description </th>
  </tr>
  <tr>
    <td>beforeCreate</td>
    <td style="color:red">data、method未初始化，dom未渲染</td>
  </tr>
  <tr>
    <td>created</td>
    <td style="color:red">data、method初始化，dom未渲染。 适合初始化异步任务，异步数据请求</td>
  </tr>
  <tr>
    <td>beforeMount</td>
    <td>data、method初始化，dom未渲染</td>
  </tr>
  <tr>
    <td>mounted</td>
    <td style="color:red">data、method初始化，dom已经渲染。 初始数据的dom渲染</td>
  </tr>
  <tr>
    <td>beforeUpdate</td>
    <td></td>
  </tr>
  <tr>
    <td>updated</td>
    <td>数据更新完毕，统一重新渲染。 如果是分别渲染，就在异步函数使用nextTick，nextTick是在更新数据后立即操作。</td>
  </tr>
  <tr>
    <td>activated</td>
    <td>组件被激活时调用</td>
  </tr>
  <tr>
    <td>deactivated</td>
    <td>组件被移除时调用</td>
  </tr>
  <tr>
    <td>beforeDestroy</td>
    <td>组件销毁前调用。 全局</td>
  </tr>
  <tr>
    <td>destroyed</td>
    <td>组件销毁后调用。 全局</td>
  </tr>
</table>


本文中只对beforeCreate,created,beforeMount,mounted,beforeUpdate,updated进行了分析。

#### **vue的优缺点**

优点：

  简单：官方文档很清晰，比 Angular 简单易学。
  
  快速：异步批处理方式更新 DOM。
  
  组合：用解耦的、可复用的组件组合你的应用程序。
  
  紧凑：~18kb min+gzip，且无依赖。
  
  强大：表达式 & 无需声明依赖的可推导属性 (computed properties)。
  
  对模块友好：可以通过 NPM、Bower 或 Duo 安装，不强迫你所有的代码都遵循 Angular 的各种规定，使用场景更加灵活。

缺点：

  大项目的时候还要配合其他框架或者库来使用，比较麻烦。
  
  不支持IE8。

从vue的优缺点也可以看出，vue偏轻量，很适合移动端。




[png1]: {{ site.baseurl }}/assets/images/20171208-1.png
[png2]: {{ site.baseurl }}/assets/images/20171208-2.png
[png3]: {{ site.baseurl }}/assets/images/20171208-3.png
[png4]: {{ site.baseurl }}/assets/images/20171208-4.png
