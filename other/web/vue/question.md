# 问题

vue中的babel-loader在默认情况下会忽略所有node_modules中的文件。但是有时候我们安装的依赖并不是编译好了的，所以就需要Babel显式转译一个依赖。
做法：
```js
//vue.config.js

transpileDependencies:[
    /[/\\]node_modules[/\\]screenfull[/\\]/
]

```