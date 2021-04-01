layout: "post"
title: "react笔记"
slug: "react-note"
date: "2016-08-20 08:43"
---

## react笔记

最近项目中用到了react，记录些自己的思考。。

### focus是使用属性还是方法

原先使用属性，但是问题是focus、value等都是可能由用户改变的，而属性是不可变的，父级组件不知道组件的状态发生了改变，则此时的focus还是原来的，下次更新属性时可能会“误”改变focus状态。解决办法主要是通过`onBlur/onFocus`实现双向绑定，保证父级组件的状态中的focus和子级组件的实际状态保持一致。。这样一不小心就多了很多方法。。

事实上我们看dom的设计，虽然dom的api大部分都是由属性构成的，但是focus恰好就是一个函数。。于是最终我的解决办法是给组件增加一个focus函数，调用时通过设置ref标签，通过`this.refs.myInput.focus()`来改变组件的focus状态...

```js

var Input = React.createClass({
  render: function () {
    return <input type="text" ref="input" />
  },
  focus: function () {
    this.refs.input.focus()
  }
})

var Parent =  React.createClass({
  render: function () {
    return (
      <div>
        <button onClick={ this.onClick }>聚焦</button>
        <Input ref="myInput" />
      </div>
    )
  },
  onClick: function () {
    this.refs.myInput.focus()
  }
})

```

### react实现模态窗

实现模态窗有个坑就是挂载点一般需要在body最后append一个节点，这样可以实现全局位置。但是react的组件是分层封装的，如何实现一个子组件挂载在别的挂载点呢?看下已有的优秀实践，发现一般都是用 `unstable_renderSubtreeIntoContainer` 这个不稳定的接口实现的（如：https://github.com/reactjs/react-modal) 这个方法和render的区别是增加了一个组件实例的参数用来指定父级组件。借此原理写了一个工具函数，用来对组件进行一个包装，返回一个包装组件，不仅可以将属性传递给真实组件，还可以调用真实组件的成员方法。开始时由于项目太老还通过render+setProps模拟，不过现在升级了还是用新方法吧。。写这个函数的个中曲折（踩坑）一言难述，看注释吧。。

```js
/**
 * 在body最后面单独append一个挂载点来挂载组件，用来制作弹窗等。
 * @param  {React.Component} Component
 * @param  {object} config
 * @return {React.Component}
 */
function makeSingletonDom(Component, config) {

  config = {
    containerId: config && config.containerId || null
  }

  var SingletonDomWrapper
  var Cls = SingletonDomWrapper = React.createClass({
    componentWillMount: function () {
      if (!Cls.isRealMount) {
        Cls.container = document.createElement('div')
        if (config.containerId) Cls.container.id = config.containerId
        document.body.appendChild(Cls.container)
        Cls.isRealMount = true
      }
      Cls.mountNum = Cls.mountNum || 0
      Cls.mountNum++
      this.renderSubtree()
    },
    componentDidMount: function() {
      this.copyMethods()
    },
    renderSubtree: function (props, callback) {
      var that = this
      ReactDOM.unstable_renderSubtreeIntoContainer(
        this, <Component {...this.props}/>, Cls.container, function () {
          Cls.instance = this
          callback && callback.call(this)
        })
    },
    copyMethods: function () {
      if (this.real) return
      var instance = Cls.instance
      var that = this
      this.real = instance

      for (var key in instance) {
        if (instance.hasOwnProperty(key)
            && (instance[key] instanceof Function)
            && ! (key in this)) {
          this[key] = (function (instanceMethod) {
              return function() {
                var args = arguments
                // 避免调用组件方法和设置组件属性同时进行时发生冲突导致组件设置属性失败
                // 记得文档上说render函数可能以后会做成异步的，因此需要通过callback
                // 得到返回的实例，因此试着加上setTimeout就成功了。后来想了想，可能是因为
                // dom api毕竟都是同步的，所谓的异步实现可能也是用setTimeout，因此这里用
                // setTimeout实际上是在上次setTimeout之后顺序执行的。。做了下测试也符合猜想，见下图
                setTimeout(function () {
                  that.renderSubtree(that.props, function () {
                    instanceMethod.apply(instance, [].slice.call(args))
                  })
                })
              }
            }(instance[key]))
        }
      }
    },
    componentWillReceiveProps: function(nextProps) {
      this.renderSubtree(nextProps)
    },
    render: function () {
      return false
    },
    componentWillUnmount: function() {
      Cls.mountNum--
      if (Cls.mountNum === 0) {
        this.clearContainer()
      }
    },
    clearContainer: function () {
      ReactDOM.unmountComponentAtNode(Cls.container)
      document.body.removeChild(Cls.container)
      Cls.instance = null
      Cls.container = null
    }
  })

  return SingletonDomWrapper
}

```

setTimeout测试:

![setTimeout测试](/images/2016/08/settimeout测试.png)
