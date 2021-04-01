title: flow学习笔记-从0搭建一个完整的前端开发技术栈(3)
permalink: setup-your-front-end-develop-stack-from-zero-3-flow/
tags: [fe-stack, flow]
date: 2016-08-02 13:56:36
---

flow是facebook开发的一款JavaScript类型推导工具，对于typescript的使用者来说给JavaScript添加类型已经不陌生了，有了静态类型，代码提示更智能，写起JavaScript也有类似java的体验了，bug少了，腰不酸了，腿不疼了，走路都有精神了，仿佛明天就走向人生巅峰了。。快醒醒，PM叫你改bug了～额，，

回到正题，为啥facebook不愿使用微软的typescript呢？原因很简单，因为标准。。flow仅仅是对es6增加类型而已，可以轻松移除，并不是开发了一种全新的编程语言，如果哪天不需要了直接移除就是，但如果你使用coffeescript，现在就已经略显过时了。。

flow项目主页： https://flowtype.org

## 安装

mac: `brew install flow`
linux: `sudo apt-get install flow` or 源码编译
windows: 最新版貌似有支持windows的了 https://github.com/facebook/flow/releases/ ,下载下来把二进制包放入环境变量中就行了

## 使用
在项目根目录下运行`flow init`新建配置文件
然后在你的代码中第一行写上:

```js
@flow

var str: number = 'hello world!';
console.log(str);
```

运行`flow check`,然后你就能看到华丽的报错了。。


## 基本语法
```js
var a:string = 'aa' // string 类型
var b:string|number = 'aa' // string或number类型
b = 1

type I = {a: number} & {b: number}; //类型的并集
var x: I = {a: 1, b: 2};
x = {a: 1, b: 2, c: "three"};

var a:Array<string|number> = [1, 'aaa'] // 带泛型的数组类型

function add(a: number, b: number): number { // 定义函数的参数和返回值的类型
  return a + b
}

```

## flow的类型

### 内置类型
boolean, number, string, null & void(即undefined，不过为了避免undefined被覆盖就用void了), any, mixed

其中any和mixed都可以用来描述任意类型，区别是

* any既是任意类型的父类型，也是任意类型的子类型
* mixed只是任意类型的父类型，但是却不是任何类型的子类型。

从这个角度看any完全相当于JavaScript中的变量类型，官网上对于mixed有个很容易懂的关于函数的例子
<https://flowtype.org/docs/builtins.html#mixed>

还有array/object呢? 其实flow也支持引用类型，因此这些类型都可以用引用的概念来代替了。。

### 引用类型

其实引用类型是和值类型相区分的，只不过any和mixed也放入上一节了，所以和官方文档一致，用内置类型表示。

```js

// @flow
// 假如你定义了一个类，那么这个类的实例的类型就是这个类，例如：
var A = function(){
  this.aa = 'aaa'
}
var a1:A = new A()

// 除此之外数组还有泛型：

var a2: Array<string> = ['aaa', 'bbb']

// 数组还能这样表示：
var a3: string[] = ['aaa', 'bbb']
var a4: (string|number)[] = ['aaa', 'bbb'] //不加括号由于优先级问题则表示为string或number[]类型

//函数泛型，学过 C++ || Java || C# 的不必多说
function reversed<T>(array: T[]): T[] {
  let ret = [];
  let i = array.length;
  while (i--)
    ret.push(array[i]);
  return ret;
}
// 扩展运算符应该算作数组类型，不多说，大家都懂的。。
function sum(...xs: number[]): number {
  return xs.reduce((a,b) => a + b);
}
// 还支持lambda表达式：
const flip = <A,B>([a,b]: [A,B]): [B,A] => [b,a];


// 类就不介绍了，一样的，这里放上类泛型的demo：
class Box<T> {
  _value: T;

  constructor(value: T) {
    this._value = value;
  }

  get(): T {
    return this._value;
  }
}
// 对象类型，感觉其实就是类类型，只是不需要继承父类罢了，咦，这不就是接口吗？其实和接口还是有差异的，对象类型直接作用于对象，是Java里面没有的，后面提到的接口类型作用于类，和Java的接口更类似
// type可以用来定义类型， 至于为啥上面的不用定义，因为JavaScript已经有可以用来描述的对象了啊～
type Person = { //这是对象类型，规范对象每个属性的类型
  name: string,
  age: number, //字段
  (param1: string): number //定义了参数和返回值的函数，感觉是不是和java的接口很像？感觉JavaScript越来越强大了: )
};

// 函数类型，和接口中的一样
type TimesTwo = (value: number) => number;

// 数组类型
var array_of_num: number[] = [];
var array_of_num_alt: Array<number> = [];
var optional_array_of_num: ?number[] = null; //问号可以快捷定义可空(null/void)类型
var array_of_optional_num: Array<?number> = [null, 0];

// 元组类型，参考python的元组
var tuple_of_str_and_num: [string, number] = ["Hi", 42];

//接口类型，额，前面貌似有个对象类型，其实这两者还是有差异的，对象类型是作用于对象上，而接口是作用于类上的
interface Comparable<T> {
  compare(a: T, b: T): number;
}

//type类型支持es6语法：
//导入和导出类型
// foo.js
export type Foo = string

//other.js
import type { Foo } from "./foo"
var foo: Foo = "Hello"

// 解构赋值
var {a, b: {c}}: {a: string, b: {c: number}} = {a: "", b: {c: 0}}


// 类型转换，懒得写了，从官网贴代码了：
(1+1: number); // this is fine
(1+1: string); // but this is is an error
[(0: ?number)]; // Flow will infer the type Array<?number>
[0];            // Without the typecast, Flow infers the type Array<number>

class Base {}
class Child extends Base {}
var child: Child = new Child();

// Upcast from Child to Base, a more general type: OK
var base: Base = new Child();

// Upcast from Child to Base, a more general type: OK
(child: Base);

// Downcast from Base to Child: unsafe, ERROR
(base: Child);

// Upcast base to any then downcast any to Child.
// Unsafe downcasting from any is allowed: OK
((base: any): Child);

```

## 进阶

### 配置文件

记得我们最开始用`flow init`生成的配置文件吗？

官网介绍： <https://flowtype.org/docs/advanced-configuration.html#flowconfig>

* [include]: 包含.flowconfig 所在目录之外的目录或文件。
* [ignore]: 忽略某些目录或文件
* [libs]: 需要注意的是这个配置项不是你的项目的库，而是flow的类型库所在的目录，里面用来定义类型，比如`declare var DEBUG: bool;` 详情请看 <https://flowtype.org/docs/declarations.html>
* [options]: 参数配置 https://flowtype.org/docs/advanced-configuration.html#options

### 第三方库

flow出于性能考虑默认会忽略node_modules（引入模块自动解析成any），但是项目中一般都会用到第三方库，咋办？用React不要怕，facebook全家桶不是白叫的，早就内置支持了。。但是如果是其他的，就得自己定义了：
```yml
[libs]
interfaces/
```

```js
// interfaces/underscore.js
declare class Underscore {
  findWhere<T>(list: Array<T>, properties: {}): T;
}

declare var _: Underscore;
```

其实这才是完整的静态语言支持的关键，typescript有typings包来管理三方依赖的类型定义，flow生态其实也有: https://github.com/flowtype/flow-typed
```sh
npm install -g flow-typed

cd /path/to/my/project
flow-typed install -f 0.27 rxjs@5.0.0 # `-f 0.27` specifies the Flow version we're using for this project
# 'rxjs_v5.0.x.js' installed at /path/to/my/project/flow-typed/npm/rxjs_v5.0.xjs
```
支持的第三方包和相关版本有：
<https://github.com/flowtype/flow-typed/tree/master/definitions/npm>

不知道有没有直接从第三方包生成类型定义或者从typescript的typings转换过来的工具。。


### 工具

flow只是静态检查工具，要想真正发挥强类型的优势，没有配套的lint和代码提示工具是远远不够的。。在这里就不得不推荐facebook全家桶之一的[nuclide.io](https://nuclide.io) IDE了，该项目建立在 [Atom](https://atom.io) 编辑器上，对flow有着近乎完美的支持（除了偶尔出现各种bug，笔者写这篇文章时还给nulide提了最新版atom(v1.9)上flow失效的bug）..

如图就是nuclide的flow lint效果，据说和atom的`linter`包冲突，需要disable`linter`，不过依赖linter的包也可以依赖nuclide，并不影响。
![](/images/2016/08/587ee75d-0d08-4de2-8e15-bc647647090e.png)

此外还有自动完成功能，但是略坑的是貌似不支持node或browser的api, 也许以后会有第三方flow定义的type库吧，现在还是配合[ternjs](https://atom.io/packages/atom-ternjs)比较合理。。


but, 最新版的npm已经把所有的依赖包扁平化了，这意味着每个包目录下都不包括依赖包，这样的话如果只单独引入某个模块并不会引入依赖的模块（依然自动解析成any），但是我们却可以享受到引入模块的提示功能，比如flow-typed里面并没有包含react，因此如果想提示react可以直接这样：

```ini
[include]
./node_modules/react

```
然后就可以在`React.PropTypes.`时就会有提示了:
![](/images/2016/08/ca0859af-fceb-44cc-9570-df14fd80030a.png)

### 和webpack/babel整合

毫无疑问这种语法是不被浏览器和node支持的，因此也需要工具在编译时移除所有flow语法，官方推荐是`babel-plugin-transform-flow-strip-types`

#### 安装:

`npm i -D babel-plugin-transform-flow-strip-types`

#### webpack配置：

```js
module.exports = {
  // ...
  //
  modules: {
    loaders: [{
        test: /\.(js|jsx)$/,
        exclude: /(node_modules|bower_components)/,
        loader: 'babel',
        query: {
          presets: ['es2015', 'react'],
          plugins: ['transform-runtime', 'transform-flow-strip-types'],
        },
      },
    ]
  }
  // ...
}
```


### 和eslint整合

需要注意的是eslint是不支持flow语法的，不过还好已经有了非常完善的插件可以支持[eslint-plugin-flowtype](https://github.com/gajus/eslint-plugin-flowtype)

#### 安装
`npm i -D eslint babel-eslint eslint-plugin-flowtype`

#### 配置

然后在你的.eslintrc文件或者package.json的eslintConfig字段中配置如下就行，而且和airbnb的配置也不冲突。。
```js
{
    "parser": "babel-eslint",
    "extends": "airbnb", // 如果安装前面的配置下来这里是有airbnb的。。
    "plugins": [
        "flowtype"
    ],
    "rules": {
        "semi": [ // 无分号党果断关闭了分号
          "error",
          "never"
        ],
        "flowtype/define-flow-type": 1, // 避免flow的type定义却未使用的error
        "flowtype/require-parameter-type": 0, // 每个函数的参数都要写类型，太严格了，匿名函数也是如此，我关了。。
        "flowtype/require-return-type": [ //要求有返回类型，这个不错，开着
            1,
            "always",
            {
                "annotateUndefined": "never"
            }
        ],
        "flowtype/space-after-type-colon": [ // 类型的冒号后面加空格，为了美观果断得开
            1,
            "always"
        ],
        "flowtype/space-before-type-colon": [ // 类型的冒号前加空格，太变态了，关
            1,
            "never"
        ],
        "flowtype/type-id-match": [ // 自定义的type的名称后面都得加上Type, 如type fooType = {}, 感觉很不错的样子，暂时用不上，不管。。
            1,
            "^([A-Z][a-z0-9]+)+Type$"
        ],
        "flowtype/use-flow-type": 1 // 避免 flow type 的别名的 no-unused-vars 错误
    },
    "settings": {
        "flowtype": {
            "onlyFilesWithFlowAnnotation": false
        }
    }
}
```
