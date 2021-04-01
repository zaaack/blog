title: 使用svg图标
permalink: svg-spirit-note/
tags: [svg, icon]
date: "2016-07-25 16:35"
---

## css图标演化

图片  ->  雪碧图  ->  font icon  -> svg -> svg spirit

## svg图标

### 优点
svg不仅具有iconfont的优点，诸如矢量特性，设置颜色外，还有更多的优点，比如可以用css改变样式。


## svg雪碧图

* 使用symbol

symbol的好处是可以在symbol标签上设置一些属性，比如设置viewBox实现自动缩放，而这在defs的g标签中是无法做到的。。

其中viewBox是原图像缩放区域(x,y,width,height),对原图像进行裁剪后再缩放到svg的viewport，可以实现svg图标随css`svg{ width: 100px; height: 100px;}`进行缩放。

```svg
<svg>
    <symbol viewBox="0,0,26,24">
        <!-- 第1个图标路径形状之类代码 -->
    </symbol>
    <symbol viewBox="0,0,26,24">
        <!-- 第2个图标路径形状之类代码 -->
    </symbol>
    <symbol viewBox="0,0,26,24">
        <!-- 第3个图标路径形状之类代码 -->
    </symbol>
</svg>
```

* 使用defs

## 引用

假设有一个svg链接http://www.w3school.com.cn/svg/a_1.svg ，那么使用方式如下,和html一样，#后面跟着的是图像的id。

```svg
<svg>
  <use xlink:href="http://www.w3school.com.cn/svg/a_1.svg#Fill-1"></use>
</svg>
```

若想设置 svg的颜色，可以设置css，需要注意的是若svg的子元素中已有的设置则不会生效，因此在svg文件中最好不要有设置。
```css
svg {
  fill: red;
  stroke: green;
}
```
