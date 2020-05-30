水平居中

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

层布局模型: 绝对定位:position:absolute 相对于其父元素来说,没有就是body

position:relative:相对于自己的以前位置

position:fixed:固定不动

相对其他元素定位:

~~~html
<div id="box1"> 父元素==>参照父元素定位 position:relative
    <div id="box2">
        子元素===>position:absolute
    </div>
</div>

~~~



<div>