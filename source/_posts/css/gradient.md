title: css3渐变笔记
permalink: css3-gradient-note/
tags: [css3, css]
date: "2016-07-17 16:35"
---

css3中增加了很多新的特性，然而自己一直没有系统的学习过，只能在项目里边用边记录了。。

最近项目中需要在背景图片上加上渐变的效果，像以往这种效果一般需要图片来实现，但是移动端由于网速和高昂流量费等原因，太多的图片无疑会极大的降低体验。由于现在css3的兼容性已经越来越好了，通过[Can I Use](http://caniuse.com/#search=gradient)网站可以看出，即使是没有前缀的渐变函数的支持率也已经很高了，因此我们可以通过css3的渐变效果来模拟。

css3的渐变函数有4种分别是

属性|含义
---|---
linear-gradient|线性渐变
radial-gradient|环形渐变
repeating-linear-gradient|重复线性渐变
repeating-radial-gradient|重复环形渐变

需要注意的是，渐变函数的返回是一个图片，因此你不能在background-color中使用，只能在background-image中使用。

## linear-gradient

语法：
```
linear-gradient(
  [ <angle> | to <side-or-corner> ,]? <color-stop> [, <color-stop>]+ )
  \---------------------------------/ \----------------------------/
    Definition of the gradient line        List of color stops

where <side-or-corner> = [left | right] || [top | bottom]
  and <color-stop>     = <color> [ <percentage> | <length> ]?
```

其中第一个参数是角度或关键字，MDN上的例子中角度的单位都是角度deg(degree)，经过测试发现也可以是弧度rad(radian)

需要注意的是这个角度的计算方式和数学里的坐标系不同，是从Y轴的正半轴沿顺时针计算的，因此
to top = 0deg = 0rad
to right = 90deg = 1.57rad
to bottom = 180deg = 3.14rad
to left = 270deg = 4.71rad



第二个参数是颜色帧，包含颜色和位置(可以是百分比或长度)，颜色可以使用rgba支持透明度，再配合css3的多背景的特性就可以实现强大的渐变遮罩效果了。

具体操作上有几个问题:
* css3 multi background特性是第一个图片在最上面，后面的依次在下面，因此图片得放在最后，渐变遮罩得放在前面
* 渐变遮罩默认是全部铺满的，因此得使用透明度使下面的图片显露出来
* 其他背景属性也是通过逗号分开，分别对应背景图片

<p data-height="265" data-theme-id="0" data-slug-hash="XKkAmy" data-default-tab="css,result" data-user="zaaack" data-embed-version="2" class="codepen">See the Pen <a href="http://codepen.io/zaaack/pen/XKkAmy/">linear-gradient demo</a> by Zack Young (<a href="http://codepen.io/zaaack">@zaaack</a>) on <a href="http://codepen.io">CodePen</a>.</p>





<script async src="//assets.codepen.io/assets/embed/ei.js"></script>


## radial-gradient
语法：

```
radial-gradient(   [ circle || <length> ] [ at <position> ]? ,
                 | [ ellipse || [<length> | <percentage> ]{2}] [ at <position> ]? ,
                 | [ [ circle | ellipse ] || <extent-keyword> ] [ at <position> ]? ,
                 | at <position> ,
                 <color-stop> [ , <color-stop> ]+ )
where <extent-keyword> = closest-corner | closest-side | farthest-corner | farthest-side
  and <color-stop> = <color> [ <percentage> | <length> ]?
```
其中大小可以是关键字也可以是长度，不过貌似只有椭圆的支持百分比，分别是元素的宽高为基准。

常量| 含义
---|---
closest-side	| 圆或椭圆的边在最近的边
closest-corner	| 圆或椭圆的边在最近的角
farthest-side		| 圆或椭圆的边在最远的边
farthest-corner		| 圆或椭圆的边在最远的角

<p data-height="265" data-theme-id="0" data-slug-hash="BzwrPG" data-default-tab="css,result" data-user="zaaack" data-embed-version="2" class="codepen">See the Pen <a href="http://codepen.io/zaaack/pen/BzwrPG/">radial-gradient demo</a> by Zack Young (<a href="http://codepen.io/zaaack">@zaaack</a>) on <a href="http://codepen.io">CodePen</a>.</p>

## repeating-linear-gradient

语法：
```
repeating-linear-gradient(  [ <angle> | to <side-or-corner> ,]? <color-stop> [, <color-stop>]+ )
                            \---------------------------------/ \----------------------------/
                              Definition of the gradient line         List of color stops

where <side-or-corner> = [left | right] || [top | bottom]
   and <color-stop>     = <color> [ <percentage> | <length> ]?
```

与linear-gradient基本一致，只不过如果颜色帧没有设置的部分按设置的在两个方向上重复，比如只设置了40%和50%，那么0%到40%和50%到100%均按0%到10%的部分重复。


## repeating-radial-gradient
语法：
```

repeating-radial-gradient(
       [[ circle  || <length> ]                     [at <position>]? , |
        [ ellipse || [<length> | <percentage> ]{2}] [at <position>]? , |
        [[ circle | ellipse ] || <extent-keyword> ] [at <position>]? , |
                                                     at <position>   ,    <color-stop> [ , <color-stop> ]+ )
        \---------------------------------------------------------------/\--------------------------------/
                  Contour, size and position of the ending shape               List of color stops

where <extent-keyword> = closest-corner | closest-side | farthest-corner | farthest-side
  and <color-stop> = <color> [ <percentage> | <length> ]?
```
同上。

## 渐变叠加计算

多个带透明度的背景叠加时遵循以下公式：

假设rgb颜色A,B,C，B带不透明度alpha且叠加在A上面，C为叠加后效果
则

$$A*(1-alpha)+B*alpha=C$$

即C等于A乘以B的透明度(1-alpha)加上B乘以B的不透明度alpha

可以写个求渐变遮罩颜色值的函数如下：
```js
function getMask(A,C, alpha) {
  var rgbA = A.slice(0,3)
  var rgbC = C.slice(0,3)

  var B
  B = rgbA.map((a, i)=>
    (rgbC[i] - a*(1-alpha))/alpha
  );
  return B
}

console.log(getMask([0, 60, 60], [0, 77, 68], 0.3));
```
如果我们想得到叠加后的颜色为rgb(0, 77, 68)的话，可以这样写：
```css
background-color: rgb(0, 60, 60);
background: linear-gradient(to left, rgba(0, 117, 87, 0.3) 0%, rgba(0, 117, 87, 0.3) 100%);
```
