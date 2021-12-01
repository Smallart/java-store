# Router

## 嵌套的组件导入

```js
/**
*文件结构
* *layout/
* ** index.vue
* ** components/
*/
import Layout from '@/layout'

[
    {
        path: '',
        componnet: Layout,
        redirect: 'index',
        children:[
            {
                path: 'index',
                component: ()=> require('@/views/index'),
                name: 'Index',
                meta:{title:'首页',icon:'dashboard',affix: true}
            }
        ]
    }
]
```

Vue使用import...from...来导入组件，库，变量等。而from后的来源可以是`js、vue、json`。可以在webpack.base.conf.js中设置的：

```js
module.exports={
    resolve:{
        extendsions:['.js','.vue','.json'],
        alias:{
            '@':resolve('src')
        }
    }
}
```
以上定义之后，则对应的文件后缀可以省略：

```text
import test from './test.vue' => import test from './test'

import test from './test.js' => import test from './test'

import test from './test.json'不能省略
```

当与这几类文件同时出现在一个文件夹下，则import的导入优先级为：如果package.json存在且设置正确情况下，会默认加载package.json，若不满足，则加载index.js；若不满足，则加载index.vue。


