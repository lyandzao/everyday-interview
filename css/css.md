## 1.介绍下BFC及其应用

BFC 就是块级格式上下文，是页面盒模型布局中的一种 CSS 渲染模式，相当于一个独立的容器，里面的元素和外部的元素相互不影响。创建 BFC 的方式有：

1. html 根元素
2. float 浮动
3. 绝对定位
4. overflow 不为 visiable
5. display 为表格布局或者弹性布局

BFC 主要的作用是：

1. 清除浮动

2. 防止同一 BFC 容器中的相邻元素间的外边距重叠问题

   [网络链接](https://zhuanlan.zhihu.com/p/25321647)
   
   [本地链接](./详细/BFC.md)
   

---

## 2.怎么让一个 div 水平垂直居中

```html
<div class="parent">
  <div class="child"></div>
</div>
```

1. 

```css
div.parent {
    display: flex;
    justify-content: center;
    align-items: center;
}
```

2. 这里也可以用计算calc属性

```css
div.parent {
    position: relative; 
}
div.child {
    position: absolute; 
    top: 50%;
    left: 50%;
    transform: translate(-50%, -50%);  
}
/* 或者 */
div.child {
    width: 50px;
    height: 10px;
    position: absolute;
    top: 50%;
    left: 50%;
    margin-left: -25px;
    margin-top: -5px;
}
/* 或 */
div.child {
    width: 50px;
    height: 10px;
    position: absolute;
    left: 0;
    top: 0;
    right: 0;
    bottom: 0;
    margin: auto;
}
```

3. 

```css
div.parent {
    display: grid;
}
div.child {
    justify-self: center;
    align-self: center;
}
```

4. 

```js
div.parent {
    font-size: 0;
    text-align: center;
    &::before {
        content: "";
        display: inline-block;
        width: 0;
        height: 100%;
        vertical-align: middle;
    }
}
div.child{
  display: inline-block;
  vertical-align: middle;
}
```

5. 

```css
div.parent{
  display:flex;
}
div.child{
  margin:auto;
}
```