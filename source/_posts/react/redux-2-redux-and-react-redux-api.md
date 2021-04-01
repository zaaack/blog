title: redux和react-redux api学习--redux学习笔记(2)
permalink: redux-note-2/
date: 2016-07-24 09:32:53
tags: [redux, js]
---

>看redux的demo，总是会弹出一些难以从名字理解的函数，其实这主要是因为redux借鉴了大量函数式编程的概念，而大部分程序猿如我只有面向对象的编程经验，因此很难像一般框架一样稍微看看文档再看看demo就能理解个七七八八。。

### configureStore

在redux的demo中经常可以看到configureStore.js这个文件，从外部看，这个文件一般用来构建store，构建完后传入Provider就基本没它啥事了，但是打开一看，里面到底干了些啥，怎么改动还是不懂。。

从redux的examples中的[real-world](https://github.com/reactjs/redux/blob/master/examples/real-world/store/configureStore.dev.js)中贴段代码：
```js
import { createStore, applyMiddleware, compose } from 'redux'
import thunk from 'redux-thunk'
import createLogger from 'redux-logger'
import api from '../middleware/api'
import rootReducer from '../reducers'
import DevTools from '../containers/DevTools'

export default function configureStore(preloadedState) {
  const store = createStore(
    rootReducer,
    preloadedState,
    compose(
      applyMiddleware(thunk, api, createLogger()),
      DevTools.instrument()
    )
  )

  if (module.hot) {
    // Enable Webpack hot module replacement for reducers
    module.hot.accept('../reducers', () => {
      const nextRootReducer = require('../reducers').default
      store.replaceReducer(nextRootReducer)
    })
  }

  return store
}

```

粗看下来感觉都看得懂，先用createStore创建store，然后根据module.hot判断是不是热更新，是的话就根据reducers的改动替换rootReducer, 但是什么compose，applyMiddleware是什么鬼？

看下文档：
* createStore(reducer, [initialState])：创建一个redux store，即redux中唯一的那个状态树，接受两个参数，分别是reducer和可选的初始化状态initialState，
  * 参数
    * reducer (Function): 接收两个参数，分别是当前的 state 树和要处理的 action，返回新的 state 树。
    * [initialState] (any): 初始时的 state。 在同构应用中，你可以决定是否把服务端传来的 state 水合（hydrate）后传给它，或者从之前保存的用户会话中恢复一个传给它。如果你使用 combineReducers 创建 reducer，它必须是一个普通对象，与传入的 keys 保持同样的结构。否则，你可以自由传入任何 reducer 可理解的内容。
  * 返回值:
    * (Store): 保存了应用所有 state 的对象。改变 state 的惟一方法是 dispatch action。你也可以 subscribe 监听 state 的变化，然后更新 UI


* compose：这是一个这是函数式编程中的方法，作用是从右到左来组合多个函数，其实就是让你少写几个函数嵌套。。没有函数式编程基础的话光靠“看”肯定是看不出其作用的。。
  ```js
  compose(funcA, funcB, funcC)

  // 等效于 ==>
  funcA(funcB(funcC))
  ```

* applyMiddleware(...middlewares)
  * 参数
    * ...middlewares (arguments): 遵循 Redux middleware API 的函数。每个 middleware 接受 Store 的 dispatch 和 getState 函数作为命名参数，并返回一个函数。该函数会被传入 被称为 next 的下一个 middleware 的 dispatch 方法，并返回一个接收 action 的新函数，这个函数可以直接调用 next(action)，或者在其他需要的时刻调用，甚至根本不去调用它。调用链中最后一个 middleware 会接受真实的 store 的 dispatch 方法作为 next 参数，并借此结束调用链。所以，middleware 的函数签名是 ({ getState, dispatch }) => next => action。
  * 返回值
    * (Function) 一个应用了 middleware 后的 store enhancer。这个 store enhancer 就是一个函数，并且需要应用到 createStore。它会返回一个应用了 middleware 的新的 createStore。

贴下官网的middleware的demo：
```js
function logger({ getState }) {
  return (next) => (action) => {
    console.log('will dispatch', action)

    // 调用 middleware 链中下一个 middleware 的 dispatch。
    let returnValue = next(action)

    console.log('state after dispatch', getState())

    // 一般会是 action 本身，除非
    // 后面的 middleware 修改了它。
    return returnValue
  }
}
```

其实所谓的middlewares就是用来管理 dispatch 的状态的中间件，next是下一个middleware, 最后一个middleware的next直接传入dispatch，返回的函数是一个包装后的dispatch函数。 之所以applyMiddleware最后会传入createStore, 根据函数式思维，肯定不会取修改createStore，而是对于进行包装，返回一个新的createStore函数。

看下源码实现，其实非常简单，只是没有函数式思维略优点跟不上而已
```js
// applyMiddleware源码
export default function applyMiddleware(...middlewares) {
  return (createStore) => (reducer, preloadedState, enhancer) => {//返回一个用来包装createStore的函数，传入createStore然后返回一个新的createStore的包装函数，参数和返回值都和createStore一样
    var store = createStore(reducer, preloadedState, enhancer)
    var dispatch = store.dispatch
    var chain = []

    var middlewareAPI = {
      getState: store.getState,
      dispatch: (action) => dispatch(action)
    }
    chain = middlewares.map(middleware => middleware(middlewareAPI))//执行每个middleware获得dispatch的包装函数
    dispatch = compose(...chain)(store.dispatch)//然后通过compose函数将这些dispatch的包装函数连接起来变成一个dispatch函数，最后传入真的dispatch，就成为包含所有middleware的dispatch的包装函数了

    return {// 返回的就是store了
      ...store,
      dispatch
    }
  }
}
```

而enhancer（增强子）又是什么鬼？直接看源码，十几行的源码都看不懂那么js真是白学了。。
```js
export default function createStore(reducer, preloadedState, enhancer) {
  if (typeof preloadedState === 'function' && typeof enhancer === 'undefined') {
    enhancer = preloadedState
    preloadedState = undefined
  }

  if (typeof enhancer !== 'undefined') {
    if (typeof enhancer !== 'function') {
      throw new Error('Expected the enhancer to be a function.')
    }

    return enhancer(createStore)(reducer, preloadedState) //怎么样？是不是和前面的推测一样，如果有enhancer这个参数且是个参数，则传入createStore并对其包装一层，返回新的createStore并调用它，参数都是一样的。。
  }

  //...

}
```
读了这些源码，是不是感觉对于redux的逻辑有了一些了解？关键就是函数式的思维，像react那种面向对象的框架其实还是很好理解的，各种组件状态方法，和其他的面向对象的框架都大同小异，因为大部分程序猿都接受过大量面向对象的编程思维的训练。但是函数式的思维的逻辑则由于缺少基础，因此粗看很难理解其中奥妙，貌似很复杂，其实很简单，大部分的api都是纯函数，所有的类型都尽量不可变，尽量返回新的类型。

## bindActionCreators

这个函数的作用是把传入的对象的每一个action creator(action工厂函数，只返回纯的action对象)包装一层dispatch

看下源码：
```js
// 绑定action creator和dispatch，相当于不用你自己去写 this.props.dispatch(actionCreator(...args))了， ps: this.props.dispatch是react-redux的connect函数注入的
function bindActionCreator(actionCreator, dispatch) {
  return (...args) => dispatch(actionCreator(...args))
}

export default function bindActionCreators(actionCreators, dispatch) {
  if (typeof actionCreators === 'function') {
    return bindActionCreator(actionCreators, dispatch)
  }

  if (typeof actionCreators !== 'object' || actionCreators === null) {
    throw new Error(
      `bindActionCreators expected an object or a function, instead received ${actionCreators === null ? 'null' : typeof actionCreators}. ` +
      `Did you write "import ActionCreators from" instead of "import * as ActionCreators from"?`
    )
  }
  //actionCreators是一个属性值为action creator的对象
  var keys = Object.keys(actionCreators)
  var boundActionCreators = {}
  for (var i = 0; i < keys.length; i++) {
    var key = keys[i]
    var actionCreator = actionCreators[key]
    if (typeof actionCreator === 'function') {
      boundActionCreators[key] = bindActionCreator(actionCreator, dispatch)
    }
  }
  return boundActionCreators
}
```

## combineReducers(reducers)

把多个reducer合并成一个,代码文档都太长懒得贴了，自己去看文档吧。。

http://cn.redux.js.org/docs/api/combineReducers.html


## connect
然后我们再看connect函数，这个函数的作用是把组件(React.Component)和store连接起来，是react-redux包里的API，
贴文档：
* 参数

  * [mapStateToProps(state, [ownProps]): stateProps] (Function): 如果定义该参数，组件将会监听 Redux store 的变化。任何时候，只要 Redux store 发生改变，mapStateToProps 函数就会被调用。该回调函数必须返回一个纯对象，这个对象会与组件的 props 合并。如果你省略了这个参数，你的组件将不会监听 Redux store。如果指定了该回调函数中的第二个参数 ownProps，则该参数的值为传递到组件的 props，而且只要组件接收到新的 props，mapStateToProps 也会被调用。
    * 个人理解就是store.subscribe的方法的包装，监听到store的变化则将唯一状态树state传入，然后返回你的组件需要的数据来更新你的组件的props。

  * [mapDispatchToProps(dispatch, [ownProps]): dispatchProps] (Object or Function): 如果传递的是一个对象，那么每个定义在该对象的函数都将被当作 Redux action creator，而且这个对象会与 Redux store 绑定在一起，其中所定义的方法名将作为属性名，合并到组件的 props 中。如果传递的是一个函数，该函数将接收一个 dispatch 函数，然后由你来决定如何返回一个对象，这个对象通过 dispatch 函数与 action creator 以某种方式绑定在一起（提示：你也许会用到 Redux 的辅助函数 bindActionCreators()）。如果你省略这个 mapDispatchToProps 参数，默认情况下，dispatch 会注入到你的组件 props 中。如果指定了该回调函数中第二个参数 ownProps，该参数的值为传递到组件的 props，而且只要组件接收到新 props，mapDispatchToProps 也会被调用。
    * 个人的理解就是把dispatch绑定到props上，默认connect会注入`this.props.dispatch`,但是为了写更少的代码，还可以把`this.dispatch(actionCreator(...args))`一起注入到props，
      * 如果传入属性值为actionCreator(action的工厂函数，返回一个action)的object则会这样绑定: `this.props[objectKey] = (...args) => this.dispatch(objectValue(...args))`,
      * 如果是函数则自己定义如何绑定dispatch到props，

      ```js
      connect(state => state.todos, {
        loadTodos
      })(MyTodoListComponent)

      //调用

      render() {
        return <button onClick={this.props.loadTodos}></button>
      }

      //相当于connect中干了这种事：
      bindActionCreators(actionCreators)//然后把返回的绑定了dispatch和action的函数注入到Component中

      ```

  * [mergeProps(stateProps, dispatchProps, ownProps): props] (Function): 如果指定了这个参数，mapStateToProps() 与 mapDispatchToProps() 的执行结果和组件自身的 props 将传入到这个回调函数中。该回调函数返回的对象将作为 props 传递到被包装的组件中。你也许可以用这个回调函数，根据组件的 props 来筛选部分的 state 数据，或者把 props 中的某个特定变量与 action creator 绑定在一起。如果你省略这个参数，默认情况下返回 Object.assign({}, ownProps, stateProps, dispatchProps) 的结果。

  * [options] (Object) 如果指定这个参数，可以定制 connector 的行为。

    * [pure = true] (Boolean): 如果为 true，connector 将执行 shouldComponentUpdate 并且浅对比 mergeProps 的结果，避免不必要的更新，前提是当前组件是一个“纯”组件，它不依赖于任何的输入或 state 而只依赖于 props 和 Redux store 的 state。默认值为 true。
    * [withRef = false] (Boolean): 如果为 true，connector 会保存一个对被包装组件实例的引用，该引用通过 getWrappedInstance() 方法获得。默认值为 false

* 返回值

  * 根据配置信息，返回一个注入了 state 和 action creator 的 React 组件。

从文档和我的注解，可以看出，connect是实redux单一状态树很关键的一个函数，并且它通过四个参数让用户来决定如何在单一状态更新时去更新组件的状态（或者说属性），这样也避免了绑定model(state)到view的性能问题，因为组件的props是否改变都是由组件自身决定的(mapStateToProps)，如果是其他组件的state

## 基本使用

```js
// Root.jsx
import { Provider } from 'react-redux'
import configureStore from './store/configureStore'

const store = configureStore()
const Root = (props) => {
  return (
    <Provider store={store}>
      <App/>
    </Provider>
  )
}

// Components
import actionCreators from '../actions/todoActions'
/*
actionCreators = {
  loadTodos: function(){
    return {
      type: 'LOAD_TODOS',
      todos: ['aaa', 'bbbb']
    }
  },
  // ...
}
 */

class App extends React.Component {

  render() {
    const {loadTodos, todos} = this.props
    return (
      <button onClick={loadTodos}></button>
      <ul>{todos.map(todo => <li>{todo.name}</li>)}</ul>
    )
  }

}

function mapStateToProps(state){
  return {todos: state.todos}
}
export default connect(mapStateToProps, actionCreators)

// todoReducer.js

function todoReducer(state=[], action) {
  switch(action.type) {
  case 'LOAD_TODOS':
    return action.todos
  }

  return state
}


```

## 数据流动

View(JSX)发生变动 => dispatch action(mapDispatchToProps简化绑定) => 调用reducers函数来更新model(state, 单一状态树) => 更新View(mapStateToProps, 根据state来返回需要更新的props)


## 参考文档

* redux中文文档：http://cn.redux.js.org/docs/
* redux-totorial, 一个很简短的教程，可以让你领略 Flux 和 Redux 思想的精髓：https://github.com/react-guide/redux-tutorial-cn
* http://div.io/topic/1309
