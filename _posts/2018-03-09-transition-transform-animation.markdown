---
layout: post
title: "浅谈transition、transform和animation"
date: 2018-03-09 17:43:00 +0800
categories: post
---

这三个属性都是CSS3中提出的，与动画相关。

其中transform描述了元素静态样式。而transtion和animation都能实现动画效果。



### 用法

#### 2D/3D 转换属性（Transform）

语法

        transform: none|transform-functions;

| 值  |   描述 |
| :---- | :---- |
| none   |   定义不进行转换。 |  
| matrix(n,n,n,n,n,n) |  定义 2D 转换，使用六个值的矩阵。 |
| matrix3d(n,n,n,n,n,n,n,n,n,n,n,n,n,n,n,n) |    定义 3D 转换，使用 16 个值的 4x4 矩阵。  |
| translate(x,y) |   定义 2D 转换。  |
| translate3d(x,y,z)  |  定义 3D 转换。 |  
| translateX(x) |    定义转换，只是用 X 轴的值。|
| translateY(y)  |   定义转换，只是用 Y 轴的值。|
| translateZ(z)  |   定义 3D 转换，只是用 Z 轴的值。 |
| scale(x,y) |   定义 2D 缩放转换。|
| scale3d(x,y,z)  |  定义 3D 缩放转换。 |
| scaleX(x)  |   通过设置 X 轴的值来定义缩放转换。 |
| scaleY(y)  |   通过设置 Y 轴的值来定义缩放转换。 |
| scaleZ(z)  |   通过设置 Z 轴的值来定义 3D 缩放转换。  |
| rotate(angle)  |   定义 2D 旋转，在参数中规定角度。 |
| rotate3d(x,y,z,angle)  |   定义 3D 旋转。   |
| rotateX(angle)  |  定义沿着 X 轴的 3D 旋转。  | 
| rotateY(angle)  |  定义沿着 Y 轴的 3D 旋转。   |
| rotateZ(angle)  |  定义沿着 Z 轴的 3D 旋转。   |
| skew(x-angle,y-angle)  |   定义沿着 X 和 Y 轴的 2D 倾斜转换。 |
| skewX(angle)   |   定义沿着 X 轴的 2D 倾斜转换。 |
| skewY(angle)   |   定义沿着 Y 轴的 2D 倾斜转换。 |
| perspective(n)  |  为 3D 转换元素定义透视视图。   |

&nbsp;

举个栗子： 

        div {
            transform:rotate(7deg);
            -ms-transform:rotate(7deg);     /* IE 9 */
            -moz-transform:rotate(7deg);    /* Firefox */
            -webkit-transform:rotate(7deg); /* Safari 和 Chrome */
            -o-transform:rotate(7deg);  /* Opera */
        }

&nbsp;

#### 过渡属性（Transition）

transition 属性是一个简写属性，用于设置四个过渡属性：

* transition-property
* transition-duration
* transition-timing-function
* transition-delay

语法

        transition: property duration timing-function delay;

|值 |  描述 |
| :---- | : ---- | 
| transition-property | 规定设置过渡效果的 CSS 属性的名称。 |
| transition-duration | 规定完成过渡效果需要多少秒或毫秒。 |
| transition-timing-function |  规定速度效果的速度曲线。 |
| transition-delay |    定义过渡效果何时开始。 |

举个栗子：

        div {
            width:100px;
            height:100px;
            background:blue;
            transition:width 2s ease-in 2s;
            -moz-transition:width 2s ease-in 2s; /* Firefox 4 */
            -webkit-transition:width 2s ease-in 2s; /* Safari and Chrome */
            -o-transition:width 2s ease-in 2s; /* Opera */
        }


&nbsp;

#### CSS3 动画属性（Animation）

animation 属性是一个简写属性，用于设置六个动画属性：

* animation-name
* animation-duration
* animation-timing-function
* animation-delay
* animation-iteration-count
* animation-direction

语法

        animation: name duration timing-function delay iteration-count direction;

| 值  |   描述 |
|:--- | : ----|
| animation-name |   规定需要绑定到选择器的 keyframe 名称。。 |
| animation-duration  |  规定完成动画所花费的时间，以秒或毫秒计。 |
| animation-timing-function |    规定动画的速度曲线。 |
| animation-delay |  规定在动画开始之前的延迟。 |
| animation-iteration-count |    规定动画应该播放的次数。 |
| animation-direction  | 规定是否应该轮流反向播放动画。 |

        div {
            width:100px;
            height:100px;
            background:red;
            position:relative;
            animation:mymove 5s infinite;
            -webkit-animation:mymove 5s infinite; /*Safari and Chrome*/
        }
        
        @keyframes mymove {
            from {left:0px;}
            to {left:200px;}
        }
        
        @-webkit-keyframes mymove /*Safari and Chrome*/ {
            from {left:0px;}
            to {left:200px;}
        }

&nbsp;

### 不同点

1.触发条件不同。transition通常和hover等事件配合使用，由事件触发。animation则和gif动图差不多，立即播放

2.虚幻。animation可以设定循环次数。

3.准确性。animation可以设定每一帧的样式和时间。transition只能设定头尾。animation中可以设置每一帧需要单独变化的样式属性，transition中所有样式属性都要一起变化。

4.与javascript的交互。animation与js的交互不是很紧密。transition和js的结合更强大。js设定要变化的样式，transiton负责动画效果。

&nbsp;

### 结论

1. 如果要灵活定制多个帧以及循环，用animation.

2. 如果要简单的from to 效果，用 transition.

3. 如果要使用js灵活设定动画属性，用transition.