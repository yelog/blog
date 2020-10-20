---
title: 一文看懂JavaScript中的Promise
permalink: know-javascript-promise
date: 2020-10-20 19:43:44
categories:
- 大前端
tags:
- javascript
---
## 一、Promise 是什么

`Promise` 是 `ES6` 提供的原生对象，用来处理异步操作

它有三种状态
- `pending`: 初始状态，不是成功或失败状态。
- `fulfilled`: 意味着操作成功完成。
- `rejected`: 意味着操作失败。

## 二、使用
### 2.1 创建 Promise
通过 `new Promise` 来实例化，支持链式调用
```javascript
new Promise((resolve, reject)=>{
  // 逻辑
}).then(()=>{
  //当上面"逻辑"中调用 resolve() 时触发此方法
}).catch(()=>{
  //当上面"逻辑"中调用 reject() 时触发此方法
})
```
### 2.2 执行顺序
`Promise`一旦创建就立即执行，并且无法中途取消，执行逻辑和顺序可以从下面的示例中获得

如下，可修改 `if` 条件来改变异步结果，下面打印开始的数字是执行顺序

[在线调试此示例 - jsbin](https://jsbin.com/cijuwakeha/1/edit?js,console)
```javascript
console.log('1.开始创建并执行 Promise')
new Promise(function(resolve, reject) {
  console.log('2.由于创建会立即执行，所以会立即执行到本行')
  setTimeout(()=>{ // 模拟异步请求
    console.log('4. 1s之期已到，开始执行异步操作')
    if (true) {
        // 一般我们符合预期的结果时调用 resolve()，会在 .then 中继续执行
        resolve('成功')
    } else {
        // 不符合预期时调用 reject()，会在 .catch 中继续执行
        reject('不符合预期')
    }
  }, 1000)
}).then((res)=>{
  console.log('5.调用了then，接收数据：' + res)
}).catch((error)=>{
  console.log('5.调用了catch，错误信息：' + error)
})
console.log('3.本行为同步操作，所以先于 Promise 内的异步操作（setTimeout）')
```
执行结果如下
```javascript
"1.开始创建并执行 Promise"
"2.由于创建会立即执行，所以会立即执行到本行"
"3.本行为同步操作，所以先于 Promise 内的异步操作（setTimeout）"
"4. 1s之期已到，开始执行异步操作"
"5.调用了then，接收数据：成功"
```
### 2.3 用函数封装 Promise
这是比较常用的方法，如下用 `setTimeout` 模拟异步请求，封装通用请求函数

[在线调试此示例 - jsbin](https://jsbin.com/figuhohoki/1/edit?js,console)
```javascript
// 这是一个异步方法
function ajax(url){
  return new Promise(resolve=>{
    console.log('异步方法开始执行')
    setTimeout(()=>{
      console.log('异步方法执行完成')
      resolve(url+'的结果集')
    }, 1000)
  })
}
// 调用请求函数，并接受处理返回结果
ajax('/user/list').then((res)=>{
  console.log(res)
})
```
执行结果
```javascript
"异步方法开始执行"
"异步方法执行完成"
"/user/list的结果集"
```


## 三、高级用法
### 3.1 同时支持Callback与Promise
[在线调试此示例 - jsbin](https://jsbin.com/qitewirina/1/edit?js,console)
```javascript
function ajax(url, success, fail) {

  if (typeof success === 'function') {
    setTimeout(() => {
      if (true) {
        success({user: '羊'})
      } else if (typeof fail === 'function') {
        console.log(typeof fail)
        fail('用户不存在')
      }
    }, 1000)
  } else {
    return new Promise((resolve, reject) => {
      this.ajax(url, resolve, reject)
    })
  }

}

// callback 调用方式
ajax('/user/get', (res)=>{
  console.log('Callback请求成功！返回结果:', res)
}, (error)=>{
  console.log('Callback请求失败！错误信息:', error)
})

// Promise 调用方式
ajax('/user/get').then((res)=>{
  console.log('Pormise请求成功！返回结果：', res)
}).catch((error)=>{
  console.log('Promise请求失败！返回结果：', error)
})
```
执行结果
```javascript
Callback请求成功！返回结果: {user: "羊"}
Pormise请求成功！返回结果： {user: "羊"}
```

### 3.2 链式调用
`.then` 支持返回 `Promise` 对象进行链式调用
```javascript
ajax('/user/info').then((res)=>{
  // 用户信息查询成功后，可以根据返回结果查询后续信息
  console.log('用户信息:', res)
  return ajax('/user/score')
}).then((res)=>{
  console.log('用户成绩:', res)
  return ajax('/user/friends')
}).then((res)=>{
  console.log('用户朋友:', res)
})
```

### 3.3 Promise.all
`Promise.all` 方法用于将多个 `Promise` 实例，包装成一个新的 `Promise` 实例。
[在线调试此示例 - jsbin](https://jsbin.com/xojifuzapo/1/edit?js,console)
```javascript
// 生成一个Promise对象的数组
var promises = [2, 3, 5, 7, 11, 13].map(function(id){
  return new Promise((resolve, reject)=>{
    if (id % 3 === 0) {
      resolve(id)
    } else {
      reject(id)
    }
  });
});
Promise.all(promises).then(function(post) {
  console.log('全部通过')
}).catch(function(reason){
  console.log('未全部通过，有问题id：'+reason)
});
```
执行结果
```javascript
未全部通过，有问题id：2
```

### Reference
[mozilla web docs](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Promise)
