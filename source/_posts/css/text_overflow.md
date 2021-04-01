title: css文本溢出省略号处理
permalink: css-text-overflow/
tags: [css3, css]
date: "2016-07-22 09:31"
---

# css单行文本溢出省略号
```css
.single-line-ellipsis {
  -o-text-overflow: ellipsis;
  text-overflow: ellipsis;
  overflow: hidden;
  white-space: nowrap;
}
```

# css多行文本溢出省略号

```css
.multi-line-ellipsis {
  display: -webkit-box;
  -webkit-box-orient: vertical;
  -webkit-line-clamp: 2; /*最大行数*/
  overflow: hidden;
}
```
