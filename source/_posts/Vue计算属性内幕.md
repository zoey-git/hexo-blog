---
title: Vue计算属性内幕
date: 2019-07-06 16:51:12
tags:
---

先从案例开始。现在有一个input框用来输入数值，另有一个span标签会把货币指加上￥符号。
```
<div>
    <input v-model="money" />
    <span>{{RMB}}</span>
</div>

<script>
    data() {
        return {
            money: 10
        }
    },
    computed: {
        RMB() {
            return '￥' + this.money
        }
    }
</script>
```
这里的RMB就是一个计算属性，依赖于数据this.money，伴随后者的变化而变化。


### 响应式属性的概念
通过Object.defineProperty，可以创建一个带有getter和setter方法的属性：
```
var bank = {moneyNornal: 1}

Object.defineProperty(bank, 'money', {
    get: function () {
        console.log('get money')
        return 1
    }
})

console.log("money: ", bank.money, +","+, bank.moneyNornal) // money：1 , 1
```
尽管使用起来一样，但实际上每次访问money属性会首先经过getter函数。vue会把所有定义在data中所返回的属性做一次definProperty处理，所以每次访问属性，vue就能监听到。

计算属性是基于它们的响应式依赖进行缓存的。只在相关响应式依赖发生改变时才会重新求值。这就意味着只要`money`没有发生改变，多次访问`RMB`计算属性会立即返回之前的计算结果，而不必再次执行计算函数。

这也意味着下面的计算属性不会更新，因为Date.mow()不是响应式依赖：
```
computed: {
    now() {
        return Date.now();
    }
}
```

#### 为什么需要缓存？
假设有一个性能开销比较大的计算属性getMoney，它需要遍历一个巨大的数组并做计算。然后我们可能有其他的计算属性依赖于getMoney。如果没有缓存，我们将不可避免的多次执行A的getter！
```
computed: {
    // 在第一次执行getMoney后这个值将会被缓存，之后再获取this.getMoney时将直接取缓存，而不会执行getMoney()
    getMoney () {
        ...巨大开销
        console.log('执行了moneyAdd'); //只会打印一次
        return this.money * 10
    },
    RMB() {
        console.log('执行了RMB');
        return '￥' + this.getMoney
    },
    USD() {
        console.log('执行了USA');
        return '$' + this.getMoney
    }
},
```
如果不希望有缓存，请用方法来替代：
```
mounted: {
    // 不可避免的将多次执行getMoney()
    getMoney () {
        ...巨大开销
        console.log('执行了moneyAdd');
        return this.money * 10
    },
    RMB() {
        console.log('执行了RMB');
        return '￥' + this.getMoney()
    },
    USD() {
        console.log('执行了USA');
        return '$' + this.getMoney()
    }
},
```