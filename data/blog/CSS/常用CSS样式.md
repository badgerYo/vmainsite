---
title: 常用CSS收录
date: '2022-04-07'
tags: ['CSS']
summary: 滚动条样式、单行多行文本超出、placeholder颜色，绝对定位点击...
---

### 去除滚动条样式

scrollbar-width 宽度设置 0，加上一个伪元素。 scss/less 写法：

```css
scrollbar-width: none;
&::-webkit-scrollbar {
  display: none;
}
```

### 文本超出显示...

文本域需要指定指定范围，block 指定宽度, flex 撑开等。

单行文本：

```css
width: 100px;
display: block;
overflow: hidden;
white-space: nowrap;
text-overflow: ellipsis;
```

多行文本：

```css
width: 200px;
display: -webkit-box;
overflow: hidden;
-webkit-line-clamp: 2;
text-overflow: ellipsis;
-webkit-box-orient: vertical;
```

### 绝对定位不抢占点击

绝对定位布局覆盖了相对布局的元素之后，绝对定位里的点击事件生效，被覆盖的相对定位的元素就点击失效了。

需要进行处理：在绝对定位布局上添加样式` pointer-events: none;` 在绝对定位里需要保留点击的子元素添加样式`pointer-events: auto;`

这样相对布局和绝对布局里的子元素的点击事件都可以触发了，但两个元素 Z 向重叠时，默认只触发绝对定位元素的点击。

> MDN: 父元素值`none`表示鼠标事件“穿透”该元素， 子元素的`pointer-events`属性指定其他值时，鼠标事件可以指向该子元素

```html
<div class="rela">
  <button onclick="alert('1')">相对定位的按钮</button>
  <div class="abso">
    <button class="abso_btn" onclick="alert('2')">绝对定位层里的按钮</button>
  </div>
</div>
<style>
  .rela {
    width: 300px;
    height: 100px;
    border: 1px solid black;
    position: relative;
  }
  .abso {
    position: absolute;
    background-color: #a00;
    opacity: 0.3;
    top: 0;
    left: 0;
    width: 300px;
    height: 100px;
    pointer-events: none;
  }
  .abso_btn {
    margin-top: 50px;
    pointer-events: auto;
  }
</style>
```
