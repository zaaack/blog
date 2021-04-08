title: vscode 秒变 全功能 所见即所得 markdown 编辑器
permalink: vscode-markdown-editor/
tags: [projects, vscode]
date: 2021-04-08 14:21:04

---

# vscode 秒变 全功能 所见即所得 markdown 编辑器

最近写了一些  markdown， 在 vscode 上感觉一般的纯文本文档写起来还是比较舒服的，但是如果有表格编辑起来就比较麻烦了，排版很困难，于是想起了最近很多基于 electron/nwjs 的所见即所得的 markdown 编辑器，比如 typora, marktext 等等, 如果在 vscode 中实现类似功能编辑起来就很方便了。

于是便写了这个插件:

![]( https://ftp.bmp.ovh/imgs/2021/04/0799ad9b7657907d.gif)

插件地址： https://marketplace.visualstudio.com/items?itemName=zaaack.markdown-editor

源码地址： https://github.com/zaaack/vscode-markdown-editor

基于强大 [vditor]( https://ld246.com/article/1549638745630) 项目，vscode 端并不需要多少代码即可拥有众多 feature:

* 可视化编辑 markdown
* 图片上传 /粘贴
* 黑暗模式, 多种代码高亮样式
* 大纲、数学公式、脑图、图表、流程图、甘特图、时序图、五线谱、多媒体、语音阅读、标题锚点、代码高亮及复制、graphviz 渲染、plantumlUML 图
* ...
