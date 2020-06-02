---
title: 前端相关学习笔记
Author: tzcqupt
categories: [前端]
tags: [微信小程序,前端]
comments: true
toc: true
date: 2020-06-02 00:00:00
---

# 前端其他知识

## 水平居中

1. 如果被设置元素为文本、图片等行内元素时，水平居中是通过给父元素设置` text-align:center` 来实现的

2. 定宽块状元素

   设置左右margin 值为auto来实现居中

   ~~~html
   <!DOCTYPE HTML>
   <html>
   <head>
   <meta http-equiv="Content-Type" content="text/html; charset=utf-8">
   <title>定宽块状元素水平居中</title>
   <style>
   div{
       border:1px solid red;
   	
   	width:200px;
   	margin:20px auto;
   }
   
   </style>
   </head>
   
   <body>
   <div>我是定宽块状元素，我要水平居中显示。</div>
   </body>
   </html>
   ~~~

3. 块级元素

   把块级元素放在一排显示:display: flex

   横轴显示参数justify-content

   纵轴显示参数aligin-items

## 层布局模型

绝对定位:`position:absolute` 相对于其父元素来说,没有就是body

`position:relative`相对于自己的以前位置

`position:fixed`固定不动

相对其他元素定位:

~~~html
<div id="box1"> 父元素==>参照父元素定位 position:relative
    <div id="box2">
        子元素===>position:absolute
    </div>
</div>

~~~

## Flex布局

[学习教程](https://www.runoob.com/w3cnote/flex-grammar.html)

### 定义

Flex是Flexible Box的缩写，意为”弹性布局”，用来为盒状模型提供最大的灵活性。

任何一个容器都可以指定为Flex布局。

> 行内元素也可以使用Flex布局.
>
> 设置为Flex布局后,元素的`float`、`clear`和`vertical-align`属性将失效。

~~~css
.box1{
  display: flex;
}
.box2{
  display: inline-flex;
}
~~~

### 基本概念

容器:`flex container`

项目 `flex item`

主轴:`main axis`

交叉轴:`cross axis`

主轴起点`main start`

主轴终点 `main end`

交叉轴起点 `cross start`

交叉轴终点 `cross end`

![Flex布局](http://img.tzcqupt.top/img/flex布局.png)



### 容器属性

#### flex-direction

决定主轴的方向,默认row,主轴为水平方向,起点在左端.

~~~css
.box {
  flex-direction: row | row-reverse | column | column-reverse;
}
~~~

#### flex-wrap

默认是一行排列,一行放不下,如何换行.默认nowrap,不换行.

~~~css
.box{
  flex-wrap: nowrap | wrap | wrap-reverse;
}
/*
wrap:换行,第一行在上方
wrap-reverse 换行,第一行在下方
*/
~~~

#### flex-flow

flex-flow属性是flex-direction属性和flex-wrap属性的简写形式，默认值为row nowrap。

```css
.box {
  flex-flow: <flex-direction> <flex-wrap>;
}
```

####  justify-content

justify-content属性定义了项目在主轴上的对齐方式。

~~~css
.box {
  justify-content: flex-start | flex-end | center | space-between | space-around;
}
/*
flex-start左对齐
flex-end 右对齐
center 居中
space-between 两端对齐，项目之间的间隔都相等。
space-around 每个项目两侧的间隔相等。所以，项目之间的间隔比项目与边框的间隔大一倍。
*/
~~~

#### align-items

align-items属性定义项目在交叉轴上`cross axis`如何对齐。

~~~css
.box {
  align-items: flex-start | flex-end | center | baseline | stretch;
}
/*
flex-start：交叉轴的起点对齐。
flex-end：交叉轴的终点对齐。
center：交叉轴的中点对齐。
baseline: 项目的第一行文字的基线对齐。
stretch（默认值）：如果项目未设置高度或设为auto，将占满整个容器的高度.
*/
~~~

#### align-content

align-content属性定义了多根轴线的对齐方式。如果项目只有一根轴线，该属性不起作用。

~~~css
.box {
  align-content: flex-start | flex-end | center | space-between | space-around | stretch;
}
/*
flex-start：与交叉轴的起点对齐。
flex-end：与交叉轴的终点对齐。
center：与交叉轴的中点对齐。
space-between：与交叉轴两端对齐，轴线之间的间隔平均分布。
space-around：每根轴线两侧的间隔都相等。所以，轴线之间的间隔比轴线与边框的间隔大一倍。
stretch（默认值）：轴线占满整个交叉轴。
*/
~~~

### 项目属性

#### order

order属性定义项目的排列顺序。数值越小，排列越靠前，默认为0。

~~~css
.item {
  order: <integer>;
}
~~~

#### flex-grow

flex-grow属性定义项目的放大比例，默认为0，即如果存在剩余空间，也不放大。

~~~css
.item {
  flex-grow: <number>; /* default 0 */
}
~~~

####  flex-shrink

flex-shrink属性定义了项目的缩小比例，默认为1，即如果空间不足，该项目将缩小。

~~~css
.item {
  flex-shrink: <number>; /* default 1 */
}
~~~

#### flex-basis

flex-basis属性定义了在分配多余空间之前，项目占据的主轴空间（main size）。浏览器根据这个属性，计算主轴是否有多余空间。它的默认值为auto，即项目的本来大小。

~~~css
.item {
  flex-basis: <length> | auto; /* default auto */
}
~~~

#### flex

flex属性是flex-grow, flex-shrink 和 flex-basis的简写，默认值为0 1 auto。后两个属性可选。


```css
.item {
  flex: none | [ <'flex-grow'> <'flex-shrink'>? || <'flex-basis'> ]
   /*auto (1 1 auto) 和 none (0 0 auto)。*/
}
```

####  align-self

align-self属性允许单个项目有与其他项目不一样的对齐方式，可覆盖align-items属性。默认值为auto，表示继承父元素的align-items属性，如果没有父元素，则等同于stretch。

```css
.item {
  align-self: auto | flex-start | flex-end | center | baseline | stretch;
}
```

该属性可能取6个值，除了auto，其他都与align-items属性完全一致。



# 微信小程序

## [路由](https://developers.weixin.qq.com/miniprogram/dev/api/)

| 名称                                                         | 功能说明                                         |
| :----------------------------------------------------------- | :----------------------------------------------- |
| [wx.switchTab](https://developers.weixin.qq.com/miniprogram/dev/api/route/wx.switchTab.html) | 跳转到 tabBar 页面，并关闭其他所有非 tabBar 页面 |
| [wx.reLaunch](https://developers.weixin.qq.com/miniprogram/dev/api/route/wx.reLaunch.html) | 关闭所有页面，打开到应用内的某个页面             |
| [wx.redirectTo](https://developers.weixin.qq.com/miniprogram/dev/api/route/wx.redirectTo.html) | 关闭当前页面，跳转到应用内的某个页面             |
| [wx.navigateTo](https://developers.weixin.qq.com/miniprogram/dev/api/route/wx.navigateTo.html) | 保留当前页面，跳转到应用内的某个页面             |
| [wx.navigateBack](https://developers.weixin.qq.com/miniprogram/dev/api/route/wx.navigateBack.html) | 关闭当前页面，返回上一页面或多级页面             |

### `wx.navigateTo`

保留当前页面，跳转到应用内的某个页面。但是不能跳到 tabbar 页面。使用 [wx.navigateBack](https://developers.weixin.qq.com/miniprogram/dev/api/route/wx.navigateBack.html) 可以返回到原页面。小程序中页面栈最多十层。

#### 请求参数

**Object object**

| 属性     | 类型     | 默认值 | 必填 | 说明                                                         |
| :------- | :------- | :----- | :--- | :----------------------------------------------------------- |
| url      | string   |        | 是   | 需要跳转的应用内非 tabBar 的页面的路径 (代码包路径), 路径后可以带参数。参数与路径之间使用 `?` 分隔，参数键与参数值用 `=` 相连，不同参数用 `&` 分隔；如 'path?key=value&key2=value2' |
| events   | Object   |        | 否   | 页面间通信接口，用于监听被打开页面发送到当前页面的数据。基础库 2.7.3 开始支持。 |
| success  | function |        | 否   | 接口调用成功的回调函数                                       |
| fail     | function |        | 否   | 接口调用失败的回调函数                                       |
| complete | function |        | 否   | 接口调用结束的回调函数（调用成功、失败都会执行）             |

#### `object.success` 回调函数

##### 参数

**Object res**

| 属性           | 类型                                                         | 说明                 |
| :------------- | :----------------------------------------------------------- | :------------------- |
| `eventChannel` | [EventChannel](https://developers.weixin.qq.com/miniprogram/dev/api/route/EventChannel.html) | 和被打开页面进行通信 |

#### 示例代码

```js
wx.navigateTo({
  url: 'test?id=1',
  events: {
    // 为指定事件添加一个监听器，获取被打开页面传送到当前页面的数据
    acceptDataFromOpenedPage: function(data) {
      console.log(data)
    },
    someEvent: function(data) {
      console.log(data)
    }
    ...
  },
  success: function(res) {
    // 通过eventChannel向被打开页面传送数据
    res.eventChannel.emit('acceptDataFromOpenerPage', { data: 'test' })
  }
})
//test.js
Page({
  onLoad: function(option){
    console.log(option.query)
    const eventChannel = this.getOpenerEventChannel()
    eventChannel.emit('acceptDataFromOpenedPage', {data: 'test'});
    eventChannel.emit('someEvent', {data: 'test'});
    // 监听acceptDataFromOpenerPage事件，获取上一页面通过eventChannel传送到当前页面的数据
    eventChannel.on('acceptDataFromOpenerPage', function(data) {
      console.log(data)
    })
  }
})
```

## 易错编程规范

### 数据绑定

1. 属性值必须被包裹在双引号中.

   ~~~html
   <!-- 正确的写法 -->
   <text data-test="{{test}}"> hello world</text>
   ~~~

2. 没有被定义的不能输出

   ~~~css
   <!--
   {
     var2: undefined,
     var3: null,
     var4: "var4"
   }
   -->
   
   
   <view>{{var1}}</view>
   <view>{{var2}}</view>
   <view>{{var3}}</view>
   <view>{{var4}}</view>
   ~~~

   