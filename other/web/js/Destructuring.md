# 解构

**解构赋值：**将数组或对象“拆包”为到一系列变量中，这是为了使用变量更加方便。将结构中的各元素复制到变量中来达到“解构”的目的。

## 数组解构
等号右边可以是任何可迭代对象一起使用。等号左边可以是任何内容。

```js
let arr = ["Ilya","Knator"];
let [firstName,surname] = arr;
let [firstName,surname] = "Ilya Kantor".split(' ');
// --可以通过添加额外的逗号来把它丢弃
let [firstName,,title]= ["Julius","Caesar","Consul"];
console.log(title); // Consul
// --可以在等号左侧使用人任何“可以被赋值的”东西
let user = {}
[user.name,user.username] = "Ilya Kantor".split(' ');
// --entries
let user={
    name:"john",
    age:30
}
for(let [key,value] of Obect.entries(user)){

}
// --交换变量值
[guest,admin] = [admin,guest]
// --剩余的'...' 不只要获得第一个值，还要将后续的所有元素都收集起来
let [name1,name2,...rest]= ["Julius","Caesar","Consul","of the Roman"]
console.log(rest[0]) // consul
// --默认值 该默认值可以是表达式或是函数
let [name="Guest",surname="Anonymous"]=["Julius"]
```
## 对象解构

```js
let {var1,var2}={var1:...,var2:...}
console.log(var1);
console.log(var2);
// --把一个属性赋值给另一个名字的变量
let options={
    title: "Menu",
    width: 100,
    height: 100
}
let {width: w,height: h,title: t} = options;
// 默认值 赋值赋值可以是任何表达式甚至是函数调用
let {width=10,height=10,title} = options;

let {width: w=100,height: h=100,title} = options;
// --剩余模式 ...
let {title,...rest} = options;
console.log(rest.height);
// --对于使用以声明的变量
let title, width, height;
({title,width,height}={title:"Menu",width: 100,height: 100});
```

## 嵌套解构
如果一个对象或数组嵌套其他的对象和数组，则如何解构

```js
let options = {
    size:{
        width: 100,
        height: 200
    },
    items: ["Cake","Donut"],
    extra: true
}

let {
    size:{
        width:,
        height
    },
    items: [item1,item2]
} = options;
```

## 智能函数参数

一个函数有很多参数，其中大部分的参数都是可选的，如何对应赋值。
```js
function showMenu({title="Menu",width=100,height=100}){

}
showMenu({});
```