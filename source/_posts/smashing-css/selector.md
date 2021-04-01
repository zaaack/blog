title: CSS 选择器
permalink: css-selector/
tags: [book-note, smashing-css, css]
date: 2016-06-05 09:32:53
layout: false
---

### 选择器类型

#### 元素选择器
语法: eltname
#### 类选择器
语法: .classname
#### ID选择器
语法: #idname
#### 通用选择器
语法:
格式       | 含义
---       |---
ns\|*     |命名空间ns下的所有元素
\*\|\*    |所有命名空间下的所有元素
\|*       |没有申明命名空间的元素

*ps: 命令空间见<http://www.zhangxinxu.com/wordpress/2012/02/css-namespaces-module/>, 不过貌似Chrome都不支持*

#### 属性选择器

格式           | 匹配
---           |---
[attr]        | 拥有属性attr的元素
[attr=value]  | 属性值等于value的元素
[attr~=value] | 属性值等于value或用空白字符(`/\s/`)分割后包含value
[attr\|=value] | 属性值等于value或由value开始且后面紧跟连字符'-'
[attr^=value] | 属性值由value开始(参考正则表达式)
[attr$=value] | 属性值由value结束(参考正则表达式)
[attr*=value] | 属性值包含value
[attr operator value i] | 在上述表达式的最后加上空格和i(或I)表示属性`值`忽略大小写

<iframe height='265' scrolling='no' src='//codepen.io/zaaack/embed/YWZzXq/?height=265&theme-id=0&default-tab=html,result&embed-version=2' frameborder='no' allowtransparency='true' allowfullscreen='true' style='width: 100%;'>See the Pen <a href='https://codepen.io/zaaack/pen/YWZzXq/'>selector demo</a> by Zack Young (<a href='http://codepen.io/zaaack'>@zaaack</a>) on <a href='http://codepen.io'>CodePen</a>.
</iframe>
