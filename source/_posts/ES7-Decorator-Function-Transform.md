title: ES7 Decorator 与函数变换
date: 2016-03-29 13:27:04
tags:
---

参考 [ES7 Decorator 与函数变换](https://mp.weixin.qq.com/s?__biz=MzA4NjE3MDg4OQ==&mid=403162136&idx=1&sn=f0aeab0902d162bc98d8d8ad7ee54eea&scene=1&srcid=03268Ti8r6XeNBGPBxy39syx&key=710a5d99946419d99cbbb6cc99d9c7bc24e8ce6d4b121c9f69356e50711854f238101bd28ac34ee76fa549b9a41949b7&ascene=0&uin=NTkwMTI4NTM1&devicetype=iMac+MacBookPro10%2C2+OSX+OSX+10.11.1+build(15B42)&version=11020201&pass_ticket=O19NczYz2X7rkObDsncE5KRTY9TsXZEHMhIhCu1ABDd3ggb25Ip3wYlMn%2FZEP1pF)

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

