---
title: Vuex基础使用
date: 2019-07-06 17:37:46
tags:
---
> Copy自[Vuex官方文档](https://vuex.vuejs.org/zh/)

Vuex是一个专门为Vue程序开发的*状态管理模式*。采用集中式存储来管理应用的所有组件的状态。

### State
#### 获取State的方式：
1. 通过根实例中注册的store选项：
```
this.$store.state.count
```
2. mapState 辅助函数
```
import { mapState } from 'vuex'
computed: {
    ...mapState()  // mapState函数返回的是一个对象。
    or
    ...mapState(['aount'])  // 可以给mapState传一个数组，数组的项是你要取的值。
}
```

### Getter
*就像计算属性一样，getter的返回值会根据它的依赖被缓存起来，且只有当它的依赖发生了改变时才会被重新计算。*
#### 定义方法：
```
const store = new Vuex.Store({
    state: {
        todos: [
            {id:1, done: true},
            {id: 2, done: false}
        ]
    },
    getters: {
        // Getter 接受state作为其第一个参数
        doneTodos(state) {
            return state.todos.filter(todo => todo.done)
        },
        // Getter 也可以接受其他getter 作为其第二个参数
        doneTodosCount(state, getters) {
            return getters.doneTodos.length // 这里会获取到上面那个定义的getter方法(doneTodos)
        },
        // 通过让getter返回一个函数，来实现给getter传参。在对store里的数组进行查询时非常有用。
        getTodoById(state) {
            return (id) => state.todos.find(todo => todo.id === id) // 这里会通过参数id，从todos列表中返回Id相同的数据
        }
    }
})
```
#### 获取Getter的方式：
1. 通过属性访问
`this.$store.getters.doneTodos`
> getter在通过属性访问时是作为Vue的响应式系统的一部分缓存其中的。
2. 通过方法访问（传参方式）
`this.$store.getters.getTodoById(2)`
> getter在通过方法访问时，每次都会去进行调用，而 *不会缓存结果*。
3. mapGetters 辅助函数
```
import {mapGetters} from 'vuex;
computed: {
    ...mapGetters(['doneTodos']) // 使用对象展开运算符将getter混入computed中
    // 如果想将一个getter属性另取一个名字，使用对象形式：
    ...mapGetters({
        doneTodosCount: 'doneTodos' // 把this.doneTodosCount映射为this.$store.getters.doneTodos
    })
}
```

### Mutation
更改store中的状态的*唯一方法*是提交mutation。
#### 定义方法：
可以给mutation定义很多方法，这些方法就是我们实际进行状态更改的地方，并且它会接受state作为第一个参数。
可以向mutation方法传入额外的参数，大多数情况下参数应该是一个对象，这样可以包含多个字段并且记录的mutation会更易读。
```
const store = new Vuex.Store({
    state: {...},
    mutations: {
        increment(state, payload) {
            state.count += payload.mount  // 直接修改state
        }
    }
})
```
#### 使用方式：
可以在组件中使用 `this.$sotre.commit('xxx')` 提交mutation，或者使用`mapMutations`辅助函数将组件中的methods映射为`store.commit`调用。
1. 普通方法
`
this.$store.commit('increment', { amount: 10 })
`
2. 对象风格的提交方式
```
this.$store.commit({
    type: 'increment',  // 直接使用包含type属性的对象，其他属性为参数。
    amount: 10
})
```
3. mapMutations 辅助函数
```
import {mapMutations} from 'vuex']
methods: {
    ...mapMutations([
        'increment',  // 将 this.increment() 映射为 this.$store.commit('increment')
        'incrementBy'  // 将 this.incrementBy() 映射为 this.$store.commit('incrementBy')
    ]),
    ...mapMutations({
        add: 'increment'  //将 this.add 映射为 this.$store.commit('increment')
    })
}
```
#### 规则：
既然Vuex的store中的状态是响应式的，那么当我们变更状态时，监视状态的Vue组件也会自动更新。
这也意味着Vuex中的mutation也需要与使用Vue一样遵守一些注意事项：
1. 最好提前在你的store中初始化好所需属性；
2. 当需要在对象上添加新属性时，应该：
    * 使用Vue.set(obj, 'newProp', 123)
    * 用新对象替换老对象：state.obj = {...state.obj, newProp: 123}
3. mutation必须是同步函数。

### action
action与mutation，不同在于：
* action提交的是mutaion，而不是直接变更状态
* action可以包含任意异步操作
#### 定义方法：
```
const store = new Vuex.Store({
    state: {
        count: 0
    },
    mutations: {
        increment(state) {
            state.count++
        }
    },
    actions: {
        // action的方法接受一个与store实例具有相同方法和属性的context对象，因此可以调用`context.commit`提交一个mutation。
        increment(context) {
            context.commit('increment')
        }
        or
        // 实践中，会使用解构来简化代码
        increment({commit}) {
            commit('increment')
        }
    }
})
```
#### 使用方式：
1. action通过store.dispatch方法触发：
```
this.$store.dispatch('increment')
```
2. 传递参数方式
```
this.$store.dispatch('increment', {
    amount: 1
})
// 以对象形式
this.$store.dispatch({
    type: 'increment',
    amount: 10
})
```
3. mapActions 辅助函数
```
import {mapActions} from 'vuex'
methods: {
    ...mapActions(['increment']),
    ...mapActions({
        add: 'increment' // 将 `this.add()` 映射为 `this.$store.dispatch('increment')`
    })
}
```


### 项目中使用：
[Vuex在项目中简单模块拆分](https://github.com/zoey-git/vuex-module)