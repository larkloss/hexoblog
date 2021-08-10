---
title: Promise规范和最简实现
date: 2021-08-10 14:47:00
tags:
- 前端
- JavaScript
- Promise

categories:
- 技术
---
# PromiseA+规范

讲解PromiseA+规范前, 咱们先来了解⼀下这些术语, 以便在后续提到的时候有明确且统⼀的概念。

##术语
1. promise 是⼀个有then⽅法的对象或者是函数，⾏为遵循本规范
2. thenable 是⼀个有then⽅法的对象或者是函数
3. value 是promise状态成功时的值，也就是resolve的参数, 包括各种数据类型, 也包括
   undefined/thenable或者是 promise
4. reason 是promise状态失败时的值, 也就是reject的参数, 表示拒绝的原因
5. exception 是⼀个使⽤throw抛出的异常值

##规范
   接下来分⼏部分来讲解PromiseA+规范和实现.
<!--more-->
### Promise States
   Promise应该有三种状态. 要注意他们之间的流转关系.
1. pending
   1.1 初始的状态, 可改变.
   1.2 ⼀个promise在resolve或者reject前都处于这个状态。
   1.3 可以通过 resolve fulfilled 状态;
   1.4 可以通过 reject rejected 状态;
2. fulfilled
   2.1 最终态, 不可变.
   2.2 ⼀个promise被resolve后会变成这个状态.
   2.3 必须拥有⼀个value值
3. rejected
   3.1 最终态, 不可变.
   3.2 ⼀个promise被reject后会变成这个状态
   3.3 必须拥有⼀个reason 
   
Tips: 总结⼀下, 就是promise的状态流转是这样的
   pending resolve(value) fulfilled
   pending reject(reason) rejected
### then
   Promise应该提供⼀个then⽅法, ⽤来访问最终的结果, ⽆论是value还是reason。

`promise.then(onFulfilled, onRejected)`

1. 参数要求
   1.1 onFulfilled 必须是函数类型, 如果不是函数, 应该被忽略.
   1.2 onRejected 必须是函数类型, 如果不是函数, 应该被忽略.
   
2. onFulfilled 特性
   2.1 在promise变成 fulfilled 时，应该调⽤ onFulfilled, 参数是value
   2.2 在promise变成 fulfilled 之前, 不应该被调⽤.
   2.3 只能被调⽤⼀次(所以在实现的时候需要⼀个变量来限制执⾏次数)
   
3. onRejected 特性
   3.1 在promise变成 rejected 时，应该调⽤ onRejected, 参数是reason
   3.2 在promise变成 rejected 之前, 不应该被调⽤.
   3.3 只能被调⽤⼀次(所以在实现的时候需要⼀个变量来限制执⾏次数)
   
4. onFulfilled 和 onRejected 应该是微任务
   这⾥⽤queueMicrotask来实现微任务的调⽤.
   
5. then⽅法可以被调⽤多次
   5.1 promise状态变成 fulfilled 后，所有的 onFulfilled 回调都需要按照then的顺序执⾏, 也就
   是按照注册顺序执⾏(所以在实现的时候需要⼀个数组来存放多个onFulfilled的回调)
   5.2 promise状态变成 rejected 后，所有的 onRejected 回调都需要按照then的顺序执⾏, 也
   就是按照注册顺序执⾏(所以在实现的时候需要⼀个数组来存放多个onRejected的回调)


6. 返回值
   then 应该返回⼀个promise
   
   `promise2 = promise1.then(onFulfilled, onRejected);`
   
   6.1 onFulfilled 或 onRejected 执⾏的结果为x, 调⽤ resolvePromise( 这⾥⼤家可能难以理
   解, 可以先保留疑问, 下⾯详细讲⼀下resolvePromise是什么东⻄ )
   6.2 如果 onFulfilled 或者 onRejected 执⾏时抛出异常e, promise2需要被reject
   6.3 如果 onFulfilled 不是⼀个函数, promise2 以promise1的value 触发fulfilled
   6.4 如果 onRejected 不是⼀个函数, promise2 以promise1的reason 触发rejected
   
7. resolvePromise
`resolvePromise(promise2, x, resolve, reject)`
7.1 如果 promise2 和 x 相等，那么 reject TypeError

7.2 如果 x 是⼀个 promsie
如果x是pending态，那么promise必须要在pending,直到 x 变成 fulfilled or rejected.
如果 x 被 fulfilled, fulfill promise with the same value.
如果 x 被 rejected, reject promise with the same reason.

7.3 如果 x 是⼀个 object 或者 是⼀个 function
let then = x.then.


如果 x.then 这步出错，那么 reject promise with e as the reason.
如果 then 是⼀个函数，then.call(x, resolvePromiseFn, rejectPromise)
resolvePromiseFn 的 ⼊参是 y, 执⾏ resolvePromise(promise2, y, resolve, reject);
rejectPromise 的 ⼊参是 r, reject promise with r.
如果 resolvePromise 和 rejectPromise 都调⽤了，那么第⼀个调⽤优先，后⾯的调⽤忽
略。

如果调⽤then抛出异常e

如果 resolvePromise 或 rejectPromise 已经被调⽤，那么忽略
则，reject promise with e as the reason
如果 then 不是⼀个function. fulfill promise with x.


