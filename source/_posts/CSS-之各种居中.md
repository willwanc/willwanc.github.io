---
title: CSS 之各种居中
date: 2021-01-19 19:49:55
tags: ['元素居中', 'CSS居中']
categories: ['CSS布局', 'CSS']
---

# CSS 之各种居中

## 水平居中

- 块元素水平居中（对浮动元素和绝对定位元素无效）

```css
/* 子元素 */
.childEle {
    margin:0 auto;
}
```

- 行内元素或文本水平居中（包括行内块元素）

```css
/* 父元素 */
.parentEle {
    text-align: center;
}
```

## 垂直居中

- 单行文本垂直居中

```css
/* 父元素 */
.parentEle {
    line-height: parentHeight;
}
```


- 块状元素垂直居中（未知元素高度，兼容IE8+）

```css
/* 父元素 */
.parentEle {
    display: table-cell;
    width: 100px;
    vertical-align: middle; /* 兼容IE8+ */
}
```

**注意：** 由于display:table-cell的元素默认宽度可能为0，所以需要为父元素或子元素设定固定宽度值。

如果需要按百分比设置父元素的宽度，可以按如下方式实现：

```css
/* 父元素的父元素 */
.parentEle {
    display: table;
    width: 100%;
}
```

```css
/* 父元素 */
.parentEle {
    display: table-cell;
    width: 100%;
    vertical-align: middle;
}
```

## 水平垂直居中

## 行内元素水平垂直居中（包括行内块元素，兼容IE8+）

```css
/* 父元素 */
.parentEle {
    display: table-cell;
    text-align: center;
    vertical-align: middle;
}
```

说明：因为text-align: center只能居中行内元素和文本，所以此方法对块元素无效

### 块元素水平垂直居中

- 已知元素宽高

方法一：絕對定位+负margin實現水平垂直居中(對各種元素都有效，兼容IE6+)

```css
/* 父元素 */
.parentEle {
    position: relative;
}

/* 子元素 */
.childEle {
    position: absolute;
    top: 50%;
    left: 50%;
    margin-top: -.5height;
    margin-left: -.5width;
}
```

方法二：絕對定位+calc實現水平垂直居中(對各種元素都有效，兼容IE9+)

```css
/* 父元素 */
.parentEle {
    position: relative;
}

/* 子元素 */
.childEle {
    position: absolute;
    top: calc(50% - .5height);
    left: calc(50% - .5width);
}
```

- 未知元素宽高

方法一：利用table-cell和margin实现水平垂直居中（兼容IE8+）

```css
/* 父元素 */
.parentEle {
    display: table-cell;
    vertical-align: middle;
}

```css
/* 子元素 */
.childEle {
    margin-left: auto;
    margin-right: auto;
}
```


方法二：绝对定位+margin实现水平垂直居中（兼容IE8+）

```css
/* 父元素 */
.parentEle {
    position: relative;
}

/* 子元素 */
.childEle {
    position: absolute;
    top: 0;
    bottom: 0;
    left: 0;
    right: 0;
    margin: auto; /* 兼容IE8+ */
}
```

方法三：绝对定位＋translate实现（兼容IE9+）

```css
.parentEle {
    position: relative;
}

.childEle {
    position: absolute;
    top: 50%;
    left: 50%;
    transform: translate(-50%, -50%);
}
```

方法四：flex实现（兼容IE10+）

```css
.parentEle {
    display: flex;
    justify-content: center;
    align-items: center;
}
```
