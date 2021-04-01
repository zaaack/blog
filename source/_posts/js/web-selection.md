layout: "post"
title: "web-selection"
slug: "web-selection"
date: "2016-08-20 09:28"
---

## js操作光标

### textarea 或者 input的情况
```js
function getCursorPosition (ctrl) {//获取光标位置函数
    var pos = 0;    // IE Support
    if (document.selection) {
      ctrl.focus ();
      var Sel = document.selection.createRange ();
      Sel.moveStart ('character', -ctrl.value.length);
      pos = Sel.text.length;
    }
    // Firefox support
    else if (selectionStart in ctrl)
      pos = ctrl.selectionStart;
    return (pos);
}
function setCursorPosition(ctrl, pos){//设置光标位置函数
    if (ctrl.setSelectionRange) {
      ctrl.focus();
      ctrl.setSelectionRange(pos,pos);
    }
    else if (ctrl.createTextRange) {
      var range = ctrl.createTextRange();
      range.collapse(true);
      range.moveEnd('character', pos);
      range.moveStart('character', pos);
      range.select();
    }
}
```

### div+contenteditable 的情况（其实就是和不能输入的普通节点的情况是一样的）

```js

var sel = window.getSelection()

// 获取全部range
var ranges = []
for(var i = 0; i < sel.rangeCount; i++) {
 ranges[i] = sel.getRangeAt(i);
}

// 获取第一个range，一般也只有一个
var range = sel.getRangeAt(0)

```

range的属性:
```coffee
startContainer: Node # 开始位置所在节点
startOffset: number # 开始位置在所在节点的偏移
endContainer: Node # 结束位置在所在节点的偏移
endOffset: number # 结束位置在所在节点的偏移
```

需要注意的是这里的节点是Node而不是Element， 区别请看：
http://www.cnblogs.com/jscode/archive/2012/09/04/2670819.html
https://developer.mozilla.org/en-US/docs/Web/API/Node/nodeType?redirectlocale=en-US&redirectslug=nodeType

简单讲Element就是我们平时所看到的标签节点，它是Node的子集，Node还包括

Constant	| Value |	Description
----|--|--
Node.ELEMENT_NODE |	1	| 一个dom节点如 c or &lt;div&gt;.
Node.TEXT_NODE |	3	| 文本节点
Node.PROCESSING_INSTRUCTION_NODE |	7 |	Axml文档解析指示节点 如 &lt;?xml-stylesheet ... ?&gt; .
Node.COMMENT_NODE	| 8 | 	注释节点，如&lt;!-- 我是注释~~ --&gt;.
Node.DOCUMENT_NODE	| 9 |	document节点， &lt;html&gt;.
Node.DOCUMENT_TYPE_NODE	| 10 | DocumentType 节点，如 HTML5 文档的 &lt;!DOCTYPE html&gt;.
Node.DOCUMENT_FRAGMENT_NODE	| 11 | DocumentFragment 节点.

参考：
https://developer.mozilla.org/en-US/docs/Web/API/Range
