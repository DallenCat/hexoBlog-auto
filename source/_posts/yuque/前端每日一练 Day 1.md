
---

title: 前端每日一练 Day 1

urlname: rllc4g

date: 2020-02-24 23:40:17 +0800

tags: []

---
<a name="lECY4"></a>
### ![IMG_7527(20200224-233828).JPG](https://cdn.nlark.com/yuque/0/2020/jpeg/933518/1582559045888-daa9142a-a1b1-4dfc-8d5b-ccb3eb07ce37.jpeg#align=left&display=inline&height=608&name=IMG_7527%2820200224-233828%29.JPG&originHeight=608&originWidth=1080&size=332267&status=done&style=none&width=1080)
<!--more-->
<a name="vCEAZ"></a>
### HTML
问：<image>标签上title属性与alt属性的区别是什么？<br />答：

- alt属性是当用户鼠标停留在<image>上时显示的文字说明。
- title属性为设置该属性的元素提供建议性的信息。使用title属性提供非本质的额外信息。

---


<a name="rWyUu"></a>
### CSS
问：CSS布局有多少种？

- 弹性布局：Flex-box，通过设置display:flex;盒子分为垂直、水平两个方向，默认水平row，起始位置为左侧;column为垂直方向，起始位置为上方; 
- 固定布局：盒子模型的宽、高、margin、padding、border都是固定的。
- 流式布局：盒子模型的宽、高、margin、padding、border属性都是用百分比来，是我们平常说的自适应布局，在所有分辨率上看到的效果都是相同的。
- 浮动布局：Float布局，是float:left; float:right; 属性，可以在盒子内最后一个元素clear:both;清除浮动。
- 定位布局：分为 绝对定位（absolute）、相对定位(relative)、固定定位(fixed)。
- 响应式布局：媒体查询布局，通过设置CSS3 [@media](#) screen限制分辨率定制样式。

---


<a name="MSV11"></a>
### JS

- typeof，可以判断出`string` `number` `boolean` `undefined` `symbol` ，typeof(null/array/object)=object
- instanceof，原理是 构造函数的 prototype 属性是否出现在对象的原型链中的任何位置

```javascript
function A() {}
let a = new A();
a instanceof A     //true,因为 Object.getPrototypeOf(a) === A.prototype;
```

- Object.prototype.toString.call()，常用于判断浏览器内置对象,对于所有基本的数据类型都能进行判断，即使是 null 和 undefined
- Array.isArray()，用于判断是否为数组



