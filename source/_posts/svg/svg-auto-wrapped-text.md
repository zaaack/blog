title: svg 文本自动换行 React 组件
permalink: svg-auto-wrapped-text-component-for-react/
date: 2018-08-16 18:41:47
tags: [react, svg]

---

SVG 的 text 标签是不支持自动换行的，但是如果想用 svg 作为一个画图模版还是需要文本换行。这个组件可以自动对 svg 节点进行截断，支持 tspan 单独设置样式，比如改变颜色啥的，但是暂时不支持 tspan 改变文本宽度，因为用的是别人写的测量宽度的包。本来这个作者也有个换行组件的，但是看源码发现不支持中文，所以写了这个组件。

用法:
```js


<AutoText
    width={50 / 375 * window.innerWidth} // width should be 'px'
    x={47} // x, y can be svg coordinate
    y={337}
    fill="#494949"
    fontFamily="SourceHanSerif-Regular, Source Han Serif"
    fontSize={13}
    fontWeight="normal"
    lineSpacing={30} // or line-height
    spans={[
        'Helllo，',
        { tag: 'tspan', text: 'Amy', props: { fill: '#B09148' } },
        ', ',
        'what can I do for you?',
    ]}
/>
```

```js

import 'core-js/fn/set'
import 'core-js/fn/map'

import React from 'react'
import svgTextSize from 'svg-text-size';


const rightSymbols = new Set(
  '“‘[「【﹝〔<‹«{『（(<《'.split('')
)
const leftSymbols = new Set(
  ',，,！!?？”\]」】》>’»﹞〕〗〉})）』'.split('')
)

const toggleSymbols = new Map([
  ['\'', 'right'],
  ['"', 'right'],
])

// Takes a string, and a width (and svg attrs, if they apply), and returns
// an array of lines, representing the break points in the string.
const textWrap = (text, width, attrs, doc = document, {
  _leftSymbols = leftSymbols,
  _rightSymbols = rightSymbols,
  _toggleSymbols = toggleSymbols,
}) => {
  let words = []
  let toggleMap = new Map()
  for (let i = 0; i < text.length; i++) {
    const char = text[i];
    if (_rightSymbols.has(char)) {
      words.push(char + text[i + 1])
      i += 1
    } else if (_leftSymbols.has(char)) {
      words[words.length - 1] += char
    } else if (_toggleSymbols.has(char)) {
      let isRightStart = _toggleSymbols.get(char) === 'right'
      if (!toggleMap.has(char)) {
        toggleMap.set(char, isRightStart ? 'right' : 'left')
      }
      let stickyDirection = toggleMap.get(char)
      if (stickyDirection === 'right') {
        words.push(char + text[i + 1])
        i += 1
      } else {
        words[words.length - 1] += char
      }
      toggleMap.set(char, stickyDirection === 'left' ? 'right' : 'left')
    } else if (/^\w+$/.test(char)) {
      let last = words[words.length - 1]
      if (/^\w+$/.test(last)) {
        words[words.length - 1] += char
      } else {
        words.push(char)
      }
    } else {
      words.push(char)
    }
  }
  let lines = [];
  let currentLine = [];
  words.forEach(word => {
    const newLine = [...currentLine, word];
    const size = svgTextSize(newLine.join(''), attrs, doc);
    if (size.width > width) {
      lines.push(currentLine.join(''));
      currentLine = [word];
    } else {
      currentLine.push(word);
    }
  });
  lines.push(currentLine.join(''));
  if (lines[0] === '') { lines.shift(); }
  return lines;
};

function getReactAttr(attrs) {
  let newAttrs = {}
  for (let key in attrs) {
    let val = attrs[key]
    key = key.replace(/([A-Z])/g, s => '-' + s.toLowerCase())
    switch (key) {
      case 'font-size':
      case 'line-spacing':
      case 'letter-spacing':
        if (typeof val === 'number') {
          val += 'px'
        }
        break;
      default:
        break;
    }
    newAttrs[key] = val
  }
  return newAttrs
}

/**
\```
props: {
 spans: [
   "some text",
   { tag: 'tspan', text: 'some text', props: { } }
 ],
 width: 100,
 x: 0,
 y: 0,
}
\```
 */
export default class SVGAutoText extends React.PureComponent {

  getAttrProps() {
    const {
      spans, width, x, y,
      leftSymbols, rightSymbols, toggleSymbols,
      ...props } = this.props
    return props
  }

  getWrappedSpans() {
    let spanSlices = []
    let startIdx = 0
    let fullText = ''
    let spans = this.props.spans.map(
      span => {
        if (!span.tag) {
          return {
            tag: 'tspan',
            text: span ? span + '' : '',
            props: null,
          }
        }
        span.text += ''
        return span
      }
    )
    for (const span of spans) {
      let end = startIdx + span.text.length
      spanSlices.push({
        ...span,
        start: startIdx,
        end,
      })
      startIdx = end
      fullText += span.text
    }
    let globalProps = this.getAttrProps()
    let lines = textWrap(fullText, this.props.width, getReactAttr(globalProps), this.props)
    let tspans = []
    let lineIdx = 0
    let lineLen = 0
    let sliceIdx = 0
    let sliceLen = 0

    let lastLine = -1
    let { x: offsetX = 0, y: offsetY = 0 } = this.props
    let lineHeight = parseInt(globalProps['lineSpacing'] || globalProps['lineHeight'], 10)
    let fontSize = parseInt(globalProps['fontSize'], 10) || 16
    function makeTspan(text, props, lineIdx) {
      let x = lastLine === lineIdx ? void 0 : offsetX
      lastLine = lineIdx
      return (
        <tspan
          key={tspans.length + text}
          {...props}
          x={x}
          y={lineIdx * (lineHeight ? lineHeight : fontSize * 1.4) + offsetY}
        >
          {text}
        </tspan>
      )
    }
    for (const line of lines) {
      lineLen += line.length
      while (sliceLen < lineLen && sliceIdx < spanSlices.length) {
        let slice = spanSlices[sliceIdx]
        tspans.push(makeTspan(slice.text, slice.props, lineIdx))
        sliceIdx++
        sliceLen += slice.text.length
      }
      if (sliceLen > lineLen) { // break slice
        let lastSlice = spanSlices[sliceIdx - 1]
        let text1 = lastSlice.text.slice(0, lastSlice.text.length - (sliceLen - lineLen))
        let text2 = lastSlice.text.slice(text1.length)

        tspans.pop()
        tspans.push(makeTspan(text1, lastSlice.props, lineIdx))
        tspans.push(makeTspan(text2, lastSlice.props, lineIdx + 1))
      }
      lineIdx++
    }
    return tspans
  }

  render() {
    return (
      <text {...this.getAttrProps()}>
        {this.getWrappedSpans()}
      </text>
    )
  }
}

```
