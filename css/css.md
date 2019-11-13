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

## 3.分析比较 opacity: 0、visibility: hidden、display: none 优劣和适用场景。

> 1. display: none (不占空间，不能点击, 会发生回流)（场景，显示出原来这里不存在的结构）
> 2. visibility: hidden（占据空间，不能点击, 会发生重绘）（场景：显示不会导致页面结构发生变动，不会撑开）
> 3. opacity: 0（占据空间，可以点击）（场景：可以跟transition搭配）

```
补充：株连性
如果祖先元素遭遇某祸害，则其子孙孙无一例外也要遭殃，比如：
opacity:0和display:none，若父节点元素应用了opacity:0和display:none，无论其子孙元素如何挣扎都不会再出现在大众视野；
而若父节点元素应用visibility:hidden，子孙元素应用visibility:visible，那么其就会毫无意外的显现出来。
```

- `display: none;`

1. **DOM 结构**：浏览器不会渲染 `display` 属性为 `none` 的元素，不占据空间；
2. **事件监听**：无法进行 DOM 事件监听；
3. **性能**：动态改变此属性时会引起重排，性能较差；
4. **继承**：不会被子元素继承，毕竟子类也不会被渲染；
5. **transition**：`transition` 不支持 `display`。

- `visibility: hidden;`

1. **DOM 结构**：元素被隐藏，但是会被渲染不会消失，占据空间；
2. **事件监听**：无法进行 DOM 事件监听；
3. **性 能**：动态改变此属性时会引起重绘，性能较高；
4. **继 承**：会被子元素继承，子元素可以通过设置 `visibility: visible;` 来取消隐藏；
5. **transition**：`transition` 支持 `display`。

- opacity: 0;

1. **DOM 结构**：透明度为 100%，元素隐藏，占据空间；
2. **事件监听**：可以进行 DOM 事件监听；
3. **性 能**：提升为合成层，不会触发重绘，性能较高；
4. **继 承**：会被子元素继承,且，子元素并不能通过 `opacity: 1` 来取消隐藏；
5. **transition**：`transition` 支持 `opacity`。

