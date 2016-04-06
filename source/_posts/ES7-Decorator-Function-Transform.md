title: ES7 Decorator 与函数变换
date: 2016-03-29 13:27:04
tags:
---

参考 [ES7 Decorator 与函数变换](https://mp.weixin.qq.com/s?__biz=MzA4NjE3MDg4OQ==&mid=403162136&idx=1&sn=f0aeab0902d162bc98d8d8ad7ee54eea)

这里面有很多的例子非常好。

```js
function add(x, y){
    return x + y;
}
function mul(x, y){
    return x * y;
}
function concat(arr1, arr2){
    return arr1.concat(arr2);
}
```

现在是 2 个数做 +，*，数组连接。

支持多位数，直接侵入式改写：

```js
function add_many(...args){
    return args.reduce((a,b) => a+b));
}
function mul_many(...args){
    return args.reduce((a,b) => a*b));
}
function concat(arr1, arr2){
    return args.reduce((a,b) => a.concat(b));
}
```

更聪明的方法，抽出高阶函数。

```js
function reducer(target){
    return (...args) => args.reduce(target);
}
 
let add_many = reducer(add);
let mul_many = reducer(mul);
let concat_many = reducer(concat);
```

