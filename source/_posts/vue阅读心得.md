---
title: vue阅读笔记
date: 2019-03-15 09:48:37
tags: vue、vuex
---

## vue基础知识

### vue实例

**实例被创建时data中存在的属性才是相应式的**，
通过使用Object.freeze()会阻止修改现有的属性，也意味着响应系统无法再追踪变化。

#### 指令：是带有v-前缀的特殊特性。指令的指责是当表达式的值改变时，将其产生的连带影响响应式作用于DOM

#### 常用指令缩写

```
v-bind 的缩写
<a v-bind:href="url">...</a>
<a :href="url">...</a>
v-on 的缩写
<a v-on:click="doSomething">...</a>
<a @click="doSomething">...</a>
```

#### 计算属性
计算属性缓存法的区别
```
vm = new Vue({
  el: '#example',
  data: 'message',
  computed: {
     reversedMessage: function() {
         return this.message.split('').reverse.join('')
     } 
  } 
  methods: {
      reversedMesaage: function() {
          return this.message.split('').reverse.join('') 
      }
  }
})
```
我们可以将同一个函数定义为一个方法而不是一个计算属性，两种方式的最终结果确实是完全相同的。不同的是**计算属性是基于他们的依赖进行缓存的**，只要相关依赖发生改变时它们才会重新求值。

#### 侦听器  
虽然计算属性在大多数情况下更合适，但有时也需要一个自定义的侦听器，这就是为什么通过watch选项提供了一个更通用的方法来响应数据的变化。当需要在数据变化时执行异步或开销较大的操作时，这个方式时最有用的。

### 条件渲染
- 不推荐同时使用v-if和v-for。
- 当v-if与v-for一起使用时，v-for具有比v-if更高的优先级。

### 列表渲染

```
var vm = new Vue({
  data: {
    items: ['a', 'b', 'c']
  }
})
vm.items[1] = 'x' // 不是响应性的
vm.items.length = 2 // 不是响应性的
```
可以通过Vue.set方法`Vue.set(vm.items, indexOfItem, newValue)`也可以使用vm.$set实例方法`vm.$set(vm.items, indexOfItem, newValue)`。对象方法同样如此。

### 事件处理

有时也需要在内联语句处理器中访问原始的DOM事件。可以用特殊变量$event把它传入方法
`<button v-on:click="warn('Form cannot be submitted yet.', $event)">
  Submit
</button>`

- 事件修饰符
```
<!-- 阻止单击事件继续传播 -->
<a v-on:click.stop="doThis"></a>

<!-- 提交事件不再重载页面 -->
<form v-on:submit.prevent="onSubmit"></form>

<!-- 修饰符可以串联 -->
<a v-on:click.stop.prevent="doThat"></a>

<!-- 只有修饰符 -->
<form v-on:submit.prevent></form>

<!-- 添加事件监听器时使用事件捕获模式 -->
<!-- 即元素自身触发的事件先在此处理，然后才交由内部元素进行处理 -->
<div v-on:click.capture="doThis">...</div>

<!-- 只当在 event.target 是当前元素自身时触发处理函数 -->
<!-- 即事件不是从内部元素触发的 -->
<div v-on:click.self="doThat">...</div>

<!-- 滚动事件的默认行为 (即滚动行为) 将会立即触发 -->
<!-- 而不会等待 `onScroll` 完成  -->
<!-- 这其中包含 `event.preventDefault()` 的情况 -->
<div v-on:scroll.passive="onScroll">...</div>
```

### 插槽

- 插槽可以类比与react的this.props.children属性。
- 插槽的name属性相当于一个key用来匹配填写插槽内容
插槽的作用域限制： 父组件模版的所有东西都会在父级作用域内编译；子组件模版的所有东西都会在子级作用域内编译。

### 处理边界情况

#### 依赖注入
provide 和 inject类比react的context属性，组件跨级之间的通信。

```
// 顶级组件声明
provide: function() {
    return {
        getMap: this.getMap
    }
}
// 后代组件使用
inject: ['getMap']
```
相比$parent属性，这个用法可以让我们在任意后代组件中访问getMap属性，而不需要暴露整个组件实例。可以把依赖注入看作一部分大范围有效的prop。

依赖注入是存在负面影响的，将目前的组件耦合了起来，使重构变得更加困难，**同时所提供的属性是非响应式的。**

## vuex数据管理

### state

vuex的状态存储是响应式的，从store实例中读取状态最简单的就是在计算属性中返回某个状态。  
组件中获取vuex的状态：vuex通过store选项，提供了一种机制将状态从根组件“注入”到每一个子组件中（Vue.use(Vuex),子组件通过this.$store访问。

- mapState辅助函数 

当一个组件需要获取多个状态时候，将这些状态都声明为计算属性会有些重复和冗余，可以使用mapState辅助函数帮我我们生成计算属性。

### getter

getter会暴露为store.getters对象，getter可以通过属性和方法访问, ** **
```
// 属性访问
getters: {
    donTodoCount: (state, getters) => {
        return getters.doneTodos.length
    }
}

// 组件中使用
computed: {
   donTodoCount() {
       return this.$store.getters.doneTodosCount
   } 
}

------------------------------
// 方法访问
getters: {
    getTodoById: (state) => (id) => {
        return state.todos.find(todo => todo.id === id)
    }
}

store.getters.getTodoById(2)
```

#### mapGetters 辅助函数

mapGetters辅助函数仅仅是将store中的getter映射到局部计算属性
```
computed: {
    ...mapGetters([
        'doneTodoCount',
        'anotherGetter',
    ])
}
```

### mutation

更改Vuex的store中的状态的唯一方法是提交mutation，每个mutation都有一个字符串类型的事件类型（type）和一个回调函数（handler）。

**mutation必须是同步函数**

### action

- action提交的是mutation，而不是直接变更状态
- action可以包含任意异步操作。

store.dispatch可以处理被触发的action的处理函数返回的Promise,并且store.dispatch仍旧返回Promise.

### module

由于使用单一状态树，应用的所有状态会集中到一个比较大的对象，store对象变得臃肿，因为vuex允许我们将store分割成模块，每个模块有自己的state、mutation、action、getter。

```
const moduleA = {
    state: {},
    mutations: {},
    actions: {},
    getters: {},
}
const moduleB = {
    state: {},
    mutations: {},
    actions: {},
    getters: {},
}
const store = new Vuex.store({
    modules: {
        a: moduleA,
        b: moduleB,
    }
})
```

