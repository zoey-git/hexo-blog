---
title: Vue响应原理
date: 2019-07-06 23:32:29
tags:
---

### Copy自
[官方文档](https://cn.vuejs.org/v2/guide/reactivity.html)

### 如何追踪变化
Vue会遍历data选项中的所有属性，并使用`Object.defineProperty`把这些属性全部转化为`getter/setter`。
`Object.defineProperty`是ES5中一个**无法优雅降级**的特性，这也是Vue不支持IE8及更低版本浏览器的原因。

这些`getter/setter`对用户来说是不可见的，但是它们让Vue能够追踪依赖，在属性被访问和修改时通过`getter/setter`来通知变更。

每个组件实例都对应一个**watcher**实例，它会在组件渲染的过程中把“接触”过的数据属性记录为依赖。之后当依赖项的setter触发时，会通知watcher，从而使它关联的组件重新渲染。



### 检测变化的注意事项
受现代JavaScript的限制，Vue**无法检测到对象属性的添加或删除**。
Vue会在**初始化实例时**对属性执行`getter/setter`转化，所以属性必须一开始就得在`data`对象上存在才能让Vue用`Object.defineProperty`将它转化为响应式的。列如：
```
<div>
    <span v-for="(item, index) in obj" :key="index">{{item}}</span>
    <button @click="addObj">添加属性</button> // 
</div>
data() {
    return {
        obj: {
            a:1 // 响应式属性
        }
    }
},
methods: {
    addObj() {
        this.obj.b = 2 // 非响应式属性
    }
},
```
上面的代码希望调用addObj方法向`this.obj`添加属性并且视图能够更新。失败！需要向响应式对象添加属性应该用Vue.set()方法才能将添加的属性变为响应式的。
对于已经创建的实例，Vue不允许动态添加根级别的响应式属性。但是，Vue提供了`Vue.set(obj, propertyName, value)`方法向嵌套对象添加响应式属性。列如：
```
this.$set(this.obj, 'b', 2)
```
需要为已有对象赋值多个新属性时应该用原对象与要混合进去的对象的新属性一起创建一个新对象：
```
this.obj = Object.assign({}, this.obj, {c: 3, d: 4})
```

### 异步更新队列
Vue在更新DOM时是**异步**执行的。只要侦听到数据的变化，Vue将开启一个队列，并缓冲 在同一事件循环中发生的所有数据变更。如果同一个watcher被多次触发，只会被推入到队列中一次。这种在缓冲时去除重复数据对于避免不必要的计算和DOM操作非常重要。然后，在下一个事件循环tick中，Vue刷新队列并执行实际工作。Vue在内部对异步队列尝试使用原生的`Promise.then`、`MutationObserver`和`setTimeOut`，如果执行环境不支持，则会采用`setTimeOut(fn, 0)`代替。
为了在数据变化之后等待Vue完成更新DOM，可以在数据变化后立即使用`Vue.nextTick(callback)`。这样回调函数将在DOM更新完后被调用。
`nextTick()`返回一个`Promise`对象，所以可以使用`async/await`语法：
```
async function () {
    <!-- DOM未更新 -->
    await this.$nextTick()
    <!-- DOM已更新 -->
}
```