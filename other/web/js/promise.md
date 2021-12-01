# promise
JS提供了一些函数，使得我们可以计划异步行为。并且JS允许函数作为参数参与另一个函数的逻辑，最常用的便是作为某个函数的回调。虽然这样做很方便但是还是会出现一些问题，比如回调地狱。

回调产生的原因，下一步函数的执行需要上一步函数执行的情况，再做下一步的打算。

```js
function f(param,callback){
    try{
        let result = .....;
        callback(null,result);
    }catch(error){
        callback(error);
    }
}
// 当参数一的过程执行成功，则执行参数二的执行过程，如果有多个参数则一直嵌套下去。
function demo(param){
    f(param,(error,param1)=>{
        if(error){
            ...
        }else{
            f(param1,(error,param2)=>{
                ...
            })
        }
    })
}

///////将步骤函数独立出来
function demo(param){
    f(param,step1);
}

function step1(error,param){
    f(param,step2);
}

```

对于某个函数中有过多的嵌套函数，这就称之为回调地狱。这种代码维护难度高。独立出来后虽然每个行为都编写成为了一个独立的顶层函数，但是在代码可读性方面依然很差。

了解了上面的一些情况后，我们就需要一种能够将栈似调用的转化为链式调用的方法，该方法便是`promsie`。

## promise语法

```js
let promise = new Promise(function(resolve,reject){
    // executor
});
```
传递给new Promise的函数被称之为executor，当new Promse被创建，executor会自动运行，它包含最终应产出结果的生产者代码。它的参数resolve和reject是由JavaScript滋生提供的回调。

当executor获得结果，无论是早还是晚都没关系，它应该调用以下回调之一：
* resolve(value) - 如果任务完成并带有结果value
* reject(error) - 如果出现error

也就是：executor会自动运行并尝试执行一项工作，尝试结束后，如果成功则调用resolve，如果出现error则调用reject。并且由new Promise构造函数返回的promise对象具有以下内部属性：
* state - 最初是 pending，然后在resolve被调用时变为“fulfilled”，或则在reject被调用时变为rejected。
* result - 最初是undefined，然后再resolve(value)被调用时变为value，或者再reject时变为error。
* 不论是resolve或是reject调用，该promise都会变为settled状态

消耗此次结果：then、catch、finally
```js
//获取该promise的结果或异常
promise.then((result,error)=>{})
//获取运行时的异常
promise.catch(error=>{})
//fianlly出来程序没有参数，总是再promise为settled状态时运行，一般用来清扫，但是语句不会结束，该函数依然会将结果和error传递给下一个处理程序，也就是后面依然可以加then或是catch
promsie.finally(()=>{})
```
.then(handler)中所使用的处理程序(handler)可以创建并返回一个promise，在这种情况下，其他的处理程序(handler)将等待它settled后再获得其结果。

```js
let promise = new Promsie((resolve,reject)=>{
    setTimeout(()=>resolve(1),1000);
});
// 这样每个result都是1
promise.then(result=>{});
promise.then(result=>{});
promise.then(result=>{});


new Promise((resolve,reject)=>{
    setTimeout(()=>resolve(1),1000);
}).then(result=>{
    return new Promise((resolve,reject)=>{

    });
}).then(result=>{})
```

## 错误处理
Promise链再错误处理中十分强大，当要给promise被reject时，控制权将移交至最近的rejection处理程序（handler）。对于executor或是handler中抛出的异常，都会交由最近的error处理程序。对于在.catch中的throw，那么控制权会被移交到下一个最近的error处理程序，处理完全之后则程序控制会交给离他最近的then处理程序。

如果出现一个error并且在这儿没有.catch，那么unhandledrejection处理程序就会被触发，并获取具有error相关信息的event对象。

## Promise的静态方法

* Promise.all 并行执行多个promise，并等待所有promise都准备就绪。如果任意一个promise被reject，则该方法会立即reject，并带有这个error。
* Promise.race 与Promise.all类似，但只等待第一个settled的promise并获取结果。

Promise的处理程序都是异步的。这是因为ECMA规定了内部队列PromiseJobs（微任务队列）。这个队列有以下特点：
* 队列是先进先出的，先进入队列的任务会首先运行
* 只有在JS引擎中没有其他任务在运行时，才开始执行任务队列中的任务

```js
// promise链会进入队列中，然后在当前代码执行完成并且先前排队的处理程序都完成时才会被执行
let promise = Promise.resolve();
promise.then(()=>alert('promise done!'));
alert("code finished");
// code finished
// promise done
```

## Async/await

以下的语句表示这个被async修饰的函数总是返回一个promise，并且它的返回值会被包装到一个resolved的promise中。

```js
async function f(){
    return 1;
}

f().then(alert)
```
关键字await让js引擎等待知道promise完成（settled）并返回结果。await会暂停函数的执行，直到promise状态变为settled，然后以promise的结果继续执行，这个行为不会消耗任何CPU资源，因为js引擎可以同时处理其他任务：执行其他脚本、处理事件。
```js
// await 这个关键字只能在 async函数中工作。
let value = await promise;

```
### error处理

```js
async fucntion f(){
    try{
        let response = await fetch();
    }catch(error){
        alert();
    }
}
// 或是
f().catch(error=>{})
```