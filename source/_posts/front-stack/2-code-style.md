title: 代码风格-从0搭建一个完整的前端开发技术栈(2)
permalink: setup-your-front-end-stack-from-zero-2-code-style/
tags: [fe-stack, code-style]
date: 2016-08-02 11:56:36
---

为了充分体现代码的逼格，良好统一的代码风格是必不可少的。而lint工具更是团队规范开发的必备利器。这里我们直接使用最流行的airbnb的lint配置，使用已有的配置比自己一个个查文档去配也方便快捷许多。

## airbnb/javascript

介绍：https://github.com/airbnb/javascript
eslint主页： http://eslint.org/

### 安装lint包



```sh
npm install --save-dev eslint-config-airbnb eslint@^2.9.0 eslint-plugin-jsx-a11y@^1.2.0 eslint-plugin-import@^1.7.0 eslint-plugin-react@^5.0.1
```
### 添加配置文件
然后直接在项目目录下添加.eslintrc文件就可以了

```js
// Use this file as a starting point for your project's .eslintrc.
// Copy this file, and add rule overrides as needed.
{
  "extends": "airbnb"
}
```

或者将配置写入package.json里面, airbnb的代码风格居然需要写分号，对于无分号党果断不能忍，于是在rules字段中添加`semi: ["error", "never"]`覆盖掉了。更多eslint的配置可以参考<http://eslint.org/docs/user-guide>
```js
{
  //...
  "eslintConfig": {
    "extends": "airbnb",
    "rules": {
      "semi": [
        "error",
        "never"
      ]
    }
  }
  //...
}

```

### 编辑器插件

我用的是atom，因为后面还要配flow，所有推荐nuclide facebook全家桶。

安装插件：

`apm install linter linter-eslint`

### 一些细节问题


#### 关闭某段代码的eslint

关闭全部规则:
```js
/* eslint-disable */
alert(1) // 一般alert是会提醒的
/* eslint-enable */  //如果整个文件都关闭则不需要最后的enable了
```
关闭部分规则：
```js
/* eslint-disable no-alert, no-console */

alert('foo');
console.log('bar');

/* eslint-enable no-alert, no-console */

```
#### 忽略某些目录或文件

使用`.eslintignore`文件，和`.gitignore`差不多

#### 全局变量

http://eslint.org/docs/user-guide/configuring#specifying-environments

```js
/* global var1, var2 */

// or

/* global var1:false, var2:false */

// or
{
    "env": {
        "browser": true,
        "node": true
    }
}

```

## airbnb/css

airbnb的css/sass风格: https://github.com/airbnb/css
scss-lint.yml配置文件：https://github.com/airbnb/css/blob/master/.scss-lint.yml

## EditorConfig

EditorConfig是一种通用编辑器配置文件，目前主流编辑器和IDE应该都对其有所支持，目的是保持项目的代码风格统一，诸如缩进，最后一行。。。这里直接抄了一个[react-boilerplate](https://github.com/mxstbr/react-boilerplate)的

更多配置细节请看EditorConfig项目主页： http://editorconfig.org/

```ini
root = true

[*]
end_of_line = lf
insert_final_newline = false
indent_style = space
indent_size = 2

```
