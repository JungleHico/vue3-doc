# Vue 3 笔记

参考官方文档：[从 Vue 2 迁移](https://v3.cn.vuejs.org/guide/migration/introduction.html)

## 组合式 API（Composition API）

### 为什么需要组合式 API？

首先看一个 Vue 2 的组件实例：

```js
export default {
  data() {
    return {
      users: [],
      books: []
    }
  },
  computed: {
    filteredUsers() {
      // ...
    },
    FilterBooks() {
      // ...
    }
  },
  watch: {
    users(newValue, oldValue) {
      // ...
    },
    books(newValue, oldValue) {
      // ...
    }
  }
  methods: {
    getUsers() {
      // ...
    },
    getBooks() {
      // ...
    }
  }
}
```

组件中使用 `data`、`computed`、`watch` 和 `methods` 这些组件选项，每个选项中都涉及 `users` 和 `books` 这两个数据，也就是说这两个数据的逻辑分散在不同的选项中，想象一下，当我们的数据比较多，或者每个函数的内容比较多时，组件的维护就会变得比较困难。看一下 Vue 3 组合式 API 的实例：

```js
import { reactive, computed, watch } from 'vue'

export default {
  setup() {
    // users 相关
    const users = reactive([])
    const filteredUsers = computed(() => {
      // ...
    })
    watch(users, (newValue, oldValue) => {
      // ...
    })
    const getUsers = () => {
      // ...
    }

    // books 相关
    const books = reactive([])
    const FilterBooks = computed(() => {
      // ...
    })
    watch(books, (newValue, oldValue) => {
      // ...
    })
    const getBooks = () => {
      // ...
    }

    return {
      users,
      filteredUsers,
      getUsers,
      books,
      FilterBooks,
      getBooks
    }
  }
}
```

使用组合式 API，有以下好处：
1. 逻辑关注点更加集中，便于维护。例如上面的例子，`users` 和 `books` 的相关业务可以分别集中管理。
2. 有利于代码的复用。例如上面的例子，`users` 和 `books` 的业务逻辑，我们可以进一步封装，便于其他组件复用。

### Setup

`setup` 函数接受两个参数，一个是 props，一个是 context

#### Props

```js
export default {
  props: {
    title: String
  }
  setup(props) {
    console.log(props.title)
  }
}
```

> 注意：`setup` 函数的调用发生在 `data`、`methods` 等这些选项之前，所以 `setup` 函数中的 `this` 并不是组件实例的引用，应当避免 `this.title` 的使用。

> 注意：`setup` 函数中 `props` 参数是响应式的，应该避免使用 ES6 解构，它会消除 `prop` 的响应性。如果需要解构 `prop`，可以在 `setup` 函数中使用 `toRefs` 函数来完成此操作。

```js
import { toRefs } from 'vue'

export default {
  setup(props) {
    const { title } = toRefs(props)

    console.log(title.value)
  }
}

```

#### Context

`context` 是一个普通的 JavaScript 对象，可以被 ES6 安全解构，它暴露了一些其他可用的值。

```js
export default {
  setup(props, context) {
    // Attribute (非响应式对象，等同于 $attrs)
    console.log(context.attrs)

    // 插槽 (非响应式对象，等同于 $slots)
    console.log(context.slots)

    // 触发事件 (方法，等同于 $emit)
    console.log(context.emit)

    // 暴露公共 property (函数)
    console.log(context.expose)
  }
}
```

### Reactive 和 Ref 创建响应式变量

在 `setup` 函数中，如果要让某个变量变成响应式，则需要通过 `reactive` 或 `ref` 创建并返回变量。

- `ref` 用于对基本值（例如，字符串）创建独立响应式值；
- `reactive` 用于对一个对象创建响应式状态。

```html
<template>
  <p>{{ count }}</p>
  <p>{{ person.name }}</p>
  <p>{{ person.age }}</p>
</template>

<script>
import { ref, reactive } from 'vue'

export default {
  setup() {
    const count = ref(1)
    const person = reactive({
      name: 'Tom',
      age: 20
    })

    console.log(count.value)

    return {
      count,
      person
    }
  }
}
</script>
```

> 注意：`ref` 返回的是一个响应式对象，在 `setup` 中，如果要访问或修改变量的值，则需要通过 `.value`

### 生命周期钩子

在 Vue 3 中，可以在 `setup` 中访问生命周期钩子，函数名为生命周期前面加 `on`：

| 选项 API | setup 内生命周期钩子 |
| - | - |
| beforeCreate | - |
| created | - |
| beforeMount | onBeforeMount |
| mounted | onMounted |
| beforeUpdate | onBeforeUpdate |
| updated | onUpdated |
| beforeUnmount | onBeforeUnmount |
| unmounted | onUnmounted |

更多生命周期钩子参考：[生命周期钩子](https://v3.cn.vuejs.org/api/options-lifecycle-hooks.html#beforecreate)

这些函数接受一个回调函数，当钩子被组件调用时将会被执行：

```js
import { onMounted } from 'vue'

export default {
  setup() {
    onMounted(() => {
      console.log('onMounted')
    })
  }
}
```

> 注意：因为 `setup` 是围绕 `beforeCreate` 和 `created` 生命周期钩子运行的，所以不需要显式地定义它们。换句话说，在这些钩子中编写的任何代码都应该直接在 `setup` 函数中编写。

### computed

在 Vue 3 中，我们可以在 `setup` 中使用独立的 `computed` 函数来创建计算属性，`computed` 函数接受一个回调函数，返回一个只读的响应式引用，与 `ref` 类似，我们需要通过 `.value` 访问计算属性的值：

```js
import { ref, computed } from 'vue'

export default {
  setup() {
    const count = ref(1)
    const doubleCount = computed(() => count.value * 2)

    console.log(count.value)
    console.log(doubleCount.value)

    return {
      count,
      doubleCount
    }
  }
}
```

### watch

在 Vue 3 中，我们也可以在 `setup` 中使用 `watch` 属性：

```js
import { ref, watch } from 'vue'

export default {
  setup() {
    const count = ref(1)
    watch(count, (newValue, oldValue) => {
      console.log(`newValue: ${newValue}, oldValue: ${oldValue}`)
    })

    count.value++

    return {
      count
    }
  }
}
```

`watch` 函数接受3个参数：
- 一个想要侦听的响应式引用或 getter 函数
- 一个回调
- 可选的配置选项

更多详细用法参考：[高阶指南-响应式-watch](https://v3.cn.vuejs.org/guide/reactivity-computed-watchers.html#watch)

### 方法

在 Vue 3 中，可以在 `setup` 中创建和返回独立的方法：

```html
<template>
  <p>count: {{ count }}</p>
  <button @click="onIncrease">+</button>
</template>

<script>
import { ref } from 'vue'

export default {
  setup() {
    const count = ref(1)
    const onIncrease = () => {
      count.value++
    }

    return {
      count,
      onIncrease
    }
  }
}
</script>
```

### Mixin

在 Vue 2 中，`mixin` 是创建可复用组件逻辑的主要机制。在 Vue 3 继续支持 `mixin` 的同时，组合式 API 是更推荐的在组件之间共享代码的方式。

### ~~filters~~

Vue 3 已废弃 `filters` 选项，请使用方法或者计算属性代替。

### Provide/inject（setup）

当组件层级较多时，通过逐级 `props` 传递数据是比较麻烦和难以维护的，这种情况可以使用 `provide/inject` 来传递数据。祖先组件通过 `provide` 提供数据，后代组件通过 `inject` 使用这些数据。

在 `setup` 中使用 `provide/inject` 也需要显示导入。

`provide` 函数参数：
1. 属性名（`String`）
2. 属性值

`inject` 函数参数：
1. 需要 inject 的属性名
2. 默认值（可选）

```html
<!-- ParentComponent.vue -->

<template>
  <child-component></child-component>
</template>

<script>
import { provide } from 'vue'

export default {
  setup() {
    provide('user', {
      name: 'Tom',
      age: 20
    })
  }
}
</script>
```

```html
<!-- ChildComponent.vue -->

<template>
  <p>name: {{ user.name }}</p>
  <p>age: {{ user.age }}</p>
</template>

<script>
import { inject } from 'vue'

export default {
  setup() {
    const user = inject('user')

    return {
      user
    }
  }
}
</script>
```

#### 添加响应式

为了增加 `provide` 值和 `inject` 值之间的响应性，我们可以在 `provide` 值时使用 `ref` 或 `reactive`。

```html
<!-- ParentComponent.vue -->

<template>
  <child-component></child-component>
</template>

<script>
import { provide, reactive } from 'vue'

export default {
  setup() {
    const user = reactive({
      name: 'Tom',
      age: 20
    })
    provide('user', user)
    user.name = 'Jacket'
  }
}
</script>
```

#### 修改响应式属性

为了方便维护，建议将对响应式属性的所有修改限制在定义 `provide` 的组件内部。如果需要在注入数据的组件内部更新 `inject` 的数据，建议 `provide` 一个方法来负责改变响应式属性。

```html
<!-- ParentComponent.vue -->

<template>
  <child-component></child-component>
</template>

<script>
import { provide, reactive, readonly } from 'vue'

export default {
  setup() {
    const user = reactive({
      name: 'Tom',
      age: 20
    })
    const setUserName = name => {
      user.name = name
    }
    provide('user', readonly(user))
    provide('setUserName', setUserName)
  }
}
</script>
```

对 `provide` 的属性使用 `readonly`，可以确保数据不会被 `inject` 组件直接修改。

## 片段（fragments）

Vue 3 组件支持片段，即组件支持多根节点，但是，这要求开发者显式定义 `attribute` 应该分布在哪里。

```html
<template>
  <header>...</header>
  <main v-bind="$attrs">...</main>
  <footer>...</footer>
</template>
```

## emits

Vue 3 新增 `emits` 选项，用于自定义事件，官方建议定义所有发出的事件，以便更好地记录组件应该如何工作。

`emits` 支持数组和对象，使用对象语法可以对事件参数进行验证，如果验证不通过，控制台会发出警告。

```html
// 子组件
<script>
export default {
  emits: ['open'], // 数组
  // 对象
  // emits: {
  //   open(value) {
  //     return typeof value === 'boolean'
  //   }
  // },
  setup(props, { emit }) {
    emit('open', true)
  }
}
</script>

// 父组件
<template>
  <child-component @open="onOpen"></child-component>
</template>

<script>
export default {
  setup() {
    const onOpen = value => {
      console.log('onOpen', value)
    }

    return {
      onOpen
    }
  }
}
</script>
```

## 全局 API

### ~~new Vue()~~ => createApp

Vue 2 中通过 `new Vue()` 创建的是一个 Vue 的根实例，而不是一个严格意义上的“app”，Vue 2 的一些全局 API 也会影响每个根实例，在测试期间，全局配置很容易意外地污染其他测试用例。

```js

// 这会影响到所有根实例
Vue.mixin({
  // ...
})

const app1 = new Vue({ el: '#app-1' })
const app2 = new Vue({ el: '#app-2' })
```

例如，上面的例子，通过 `Vue.mixin` 创建的 `mixin` 会响应所有的实例。

Vue 3 通过 `createApp` 创建独立的 app，一些全局 API 也挂载到实例上，不会对所有实例造成污染。

```js

const app = createApp({})

// 只对当前实例生效
app.mixin({
  // ...
})

app.mount('#app')
```

### ~~Vue.prototype~~ => app.config.globalProperties/provide

在 Vue 2 中， `Vue.prototype` 通常用于添加所有组件都能访问的属性。

在 Vue 3 中与之对应的是 `config.globalProperties`。这些属性将被复制到应用中，作为实例化组件的一部分。

```js
// Vue 2
Vue.prototype.$http = () => {}

export default {
  created() {
    this.$http()
  }
}
```

```js
// Vue 3
const app = createApp({})
app.config.globalProperties.$http = () => {}

import { getCurrentInstance } from 'vue'
export default {
  setup() {
    const { proxy } = getCurrentInstance()
    proxy.$http()
  }
}
```

官方不建议在应用的代码中使用 `getCurrentInstance` 作为访问组件实例的 `this` 的替代方案，所以建议使用 `provide` 来代替 `globalProperties`。

```js
const app = createApp({})
app.provide('$http', () => {})

import { inject } from 'vue'
export default {
  setup() {
    const $http = inject('$http')
    $http()
  }
}
```

## 自定义组件 v-model

Vue 3 中自定义组件 `v-model` 的变化：

| Vue 2 | Vue 3 |
| ---- | ---- |
| 默认 prop：`value`，默认事件：`input` | 默认 prop：`modelValue`，默认事件：`update:modelValue` |
| `model` 选项更改 prop 和事件名称 | `v-model` 指定参数 |
| `.sync` 修饰符对 prop 进行“双向绑定” | `v-model` 指定参数（支持多个）| 
| - | 支持自定义 `v-model` 修饰符 |

### 默认 prop 和事件名称

```html
<!-- Vue 2 -->
<child-component v-model="pageTitle" />

<!-- 是以下的简写: -->

<child-component :value="pageTitle" @input="pageTitle = $event" />
```

```html
<!-- Vue 3 -->
<child-component v-model="pageTitle" />

<!-- 是以下的简写: -->

<child-component
  :modelValue="pageTitle"
  @update:modelValue="pageTitle = $event"
/>
```

### 修改 prop 和事件名称

```html
<!-- Vue 2 -->

<!-- ChildComponent.vue -->
<script>
export default {
  // model 选项指定 prop 和事件名称
  model: {
    prop: 'title',
    event: 'change'
  },
  props: {
    title: String
  }
}
</script>

<!-- ParentComponent.vue -->
<child-component v-model="pageTitle" />

<!-- 是以下的简写（变更了绑定的属性和事件名称）: -->

<child-component :title="pageTitle" @change="pageTitle = $event" />
```

```html
<!-- Vue 3 -->

<!-- ParentComponent.vue -->
<child-component v-model="pageTitle" />

<!-- 是以下的简写（变更了绑定的属性和事件名称，不需要修改 model 选项）: -->

<child-component :title="pageTitle" @update:title="pageTitle = $event" />
```

### 替代 .sync 修饰符

Vue 2：

```js
// ChildComponent.vue
// 属性变化时触发事件
this.$emit('update:title', newValue)
```

```html
<!-- ParentComponent.vue -->
<child-component :title="pageTitle" @update:title="pageTitle = $event" />

<!-- 使用 .sync 修饰符简写： -->

<child-component :title.sync="pageTitle" />
```

Vue 3：

```html
<child-component
  :title="pageTitle"
  @update:title="pageTitle = $event"
/>

<!-- 可简写为： -->

<child-component v-model:title="pageTitle" />
```

## ~~.native 修饰符~~ => emits 选项

## ~~$listeners~~