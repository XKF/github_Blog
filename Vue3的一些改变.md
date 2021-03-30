# Vue3的一些改变

前端发展确实快到不行，我已经快跟不上，尤大提出Vue3也已经有一段时间，并且稳定版也已经出来，听说Vue的项目组现在主要是在进行Vue2迁移Vue3阶段的一些完善。那么，有时间还是需要赶紧研究一波Vue3，虽说现在Vue2加Vue-cli脚手架足够使用，但随着时间的迁移，肯定都会往Vue3去迁移的。对于没有Vue基础的人，建议还是去看下官方文档从零学习一波。
- 官方文档：https://vue3js.cn/docs/zh/guide/migration/introduction.html

如果有Vue基础的，我下面将列一些平常用得比较多的比较重要的变化，其他的话大家就自行看文档啦，而且是全新项目的变化，如果是Vue2迁移Vue3的话建议看下迁移文档，毕竟有些Vue2的旧语法在Vue3是兼容的。
- v2迁移文档：https://vue3js.cn/docs/zh/guide/migration/introduction.html#devtools-extension

## Vue3主要变化
1、Vue3 官方推荐一款新的打包工具Vite，打包构建 so fast，可以去了解一下，但目前还不够成熟，新生力量，向 Webpack 和 Rollup 发起挑战；
- Vite官网：https://vitejs.dev/

2、新建Vue实例方式改变，为了更好地按需加载，进一步缩小打包后Vue包的大小
```js
import Vue from 'vue'

new Vue({
  el: '#app',
  router,
  store,
  
  render(h) {
    return h(App)
  }
}).$mount('#app')

//转为

import { createApp } from 'vue'

createApp(App).mount('#app')
```
**3、组件不再需要一个块级容器包裹，可以并行多个便签，类似于React的<></>；**

4、布尔值attribute绑定的变更，例如disabled属性，当绑定变量值为null或undefined时相应的布尔值attribute属性消失；

**5、数据劫持底层原理更换，由Object.defineProperty更换为new Proxy，现在能够监听数组、对象里层赋值的变化以及新增响应式属性**；  
Proxy跟Object.defineProperty区别就是Proxy是劫持整个对象而不是属性

6、v-for 与 v-if  的优先级更换，v-if 现在优先级比 v-for 高，但仍不建议这俩个一起使用；

**7、.sync语法糖写法更换，.sync被废弃，更换为v-model；**
```js
<my-component :foo="bar" @update:foo="xxxxx"></my-component>
// => 等同
<my-component foo.sync="bar"></my-component>
// => 更换为
<my-component v-model:foo="bar"></my-component>
// 多个绑定的话
<user-name
  v-model:first-name="firstName"
  v-model:last-name="lastName"
></user-name>
```

8、可自定义v-model修饰符，定义后修饰符当组件的 created 生命周期钩子触发时，组件props的modelModifiers将有你定义的修饰符的值，其值为true，那你子组件修改数据时就可以根据修饰符状态进行对应的逻辑操作
```js
<div id="app">
  <my-component v-model.capitalize="myText"></my-component>
  {{ myText }}
</div>

const app = Vue.createApp({
  data() {
    return {
      myText: ''
    }
  }
})

app.component('my-component', {
  props: {
    modelValue: String,
    modelModifiers: {
      default: () => ({}) //created后会变为 { capitalize: true }
    }
  },
  methods: {
    emitValue(e) {
      let value = e.target.value
      // 根据修饰符做相应的操作
      if (this.modelModifiers.capitalize) {
        value = value.charAt(0).toUpperCase() + value.slice(1)
      }
      this.$emit('update:modelValue', value)
    }
  },
  template: `<input
    type="text"
    :value="modelValue"
    @input="emitValue">`
})

app.mount('#app')
```

9、事件绑定去除.native修饰符，现在组件的根元素可以直接继承attribute，如果你不希望直接继承attribute，你可以在组件选项中设置 **inheritAttrs: false** ，然后自己通过$attrs去分配属性继承。**PS：如果根节点是多节点的情况你需要显示地去绑定$attrs到某个根节点元素上，不然会报错。**

10、组件可以通过emits定义所有发出事件的名称，如果里面定义了click等原生事件，将使用组件中事件替代原生事件侦听器，即你自己去触发自定义的click等原生事件。同时，可以利用emits做抛出事件的验证，不过好像就验证而已，拦截不了，顶多验证完做些toast提示操作等。
```js
app.component('custom-form', {
  emits: {
    // 没有验证
    click: null,

    // 验证submit 事件
    submit: ({ email, password }) => {
      if (email && password) {
        return true
      } else {
        console.warn('Invalid submit event payload!')
        return false //拦截了，没抛出事件
      }
    }
  },
  methods: {
    submitForm() {
      this.$emit('submit', { email, password })
    }
  }
})
```
11、v-slot可以缩写为#，例如 v-slot:header === #header，想使用缩写就必须采用具名插槽，默认插槽要写为#default

12、**提供与注入(provide and inject)**  
深层的子组件可以直接注入顶层父组件的属性，而不用再用prop一层一层繁琐地传下去了。
```js
const app = Vue.createApp({})

app.component('todo-list', {
  data() {
    return {
      todos: ['Feed a cat', 'Buy tickets']
    }
  },
  //建议函数定义provide，才能访问this，不然只能传静态值
  provide(){
    return {
        user: 'John Doe' //父组件提供可注入属性值
    }
  },
  template: `
    <div>
      {{ todos.length }}
      <!-- 模板的其余部分 -->
    </div>
  `
})

app.component('todo-list-statistics', {
  inject: ['user'], //孙子组件直接获取，不用再prop一层一层传下来
  created() {
    console.log(`Injected property: ${this.user}`) // > 注入 property: John Doe
  }
})
```
当然提供与注入不是响应式更新，如果你想做成响应性，可通过ref property 或 reactive 对象传递给 provide 来更改此行为，或者给对应的值分配一个计算属性。例如：
```js
app.component('todo-list', {
  // ...
  provide() {
    return {
      todoLength: Vue.computed(() => this.todos.length)
    }
  }
})
```
同时，为了防止provide被inject的组件修改，建议用readonly函数处理下属性传递。

13、**异步组件引入方式更换**
需要包裹多一层Vue.defineAsyncComponent，当然你可以将{ defineAsyncComponent }解构出来
```js
//ago
const AsyncComp = () => import('@/views/Index.vue')

//now 
const AsyncComp = defineAsyncComponent(
    () => import('@/views/Index.vue')
)
```

14、Teleport
通过teleport标签的to属性将HTML渲染到指定标签，而不用为了在不同位置渲染而将组件进行拆分（。。这又是学的React吧）
```js
app.component('modal-button', {
  template: `
    <button @click="modalOpen = true">
        Open full screen modal! (With teleport!)
    </button>

    //.modal类div渲染到body去了，to可以为标签名、.类名、#ID名
    <teleport to="body">
      <div v-if="modalOpen" class="modal">
        <div>
          I'm a teleported modal! 
          (My parent is "body")
          <button @click="modalOpen = false">
            Close
          </button>
        </div>
      </div>
    </teleport>
  `,
  data() {
    return { 
      modalOpen: false
    }
  }
})
```

15、**组合式API**  
将相似的逻辑关注点进行提取整合，比如获取列表和赋值列表这一套操作就可以组合。有点类似于函数式编程的函数组合，将相似的逻辑关注点组合成一个功能模块（包括定义响应式属性、绑定时间钩子等），预定义好各个逻辑功能，在执行完毕后暴露所需要的值，直接取用就OK了，这样的代码逻辑块十分清晰明朗，可能小项目你没啥感觉，但遇到特大项目就很有用了，做过稍大一点的项目你是不是会觉得代码很杂，变量啊、方法啊贼多，找来找去，这就是组合式API诞生的原因。组合式API的一个大概构建过程：  

1、将某些逻辑关注点相似的变量、操作进行整合，放到setup函数中。  
PS：
- setup函数运行在beforeCreate前，也就是vue还没实例，因此没有this，只有props和context可以使用，props只能拿默认值，setup通过return一个对象供后续Vue实例选项式API使用，会将这个对象并入到Vue实例中去，通过this调用对应属性，有点提预定义一些data属性、method方法的味道。  
- setup返回的对象中的refs和reactive对象会被自动解开
```js
setup() {
  const readersNumber = ref(0)
  const book = reactive({ title: 'Vue 3 Guide' })

  // expose to template
  return {
    readersNumber,
    book
  }
  /*{
      readersNumber:0 //不能通过value取值了
      book：{
        title:'Vue3 Guide'
      }hi 
  }*/
}
```

2、设置响应式变量，通过ref或者reactive函数自己手动定义，ref相当于一个包装函数，主要是为了将Number和String这类基本类型的值包装为引用类型，从而能够实现响应式，如果是引用类型建议用reactive函数。  

3、定义方法以及绑定时间钩子执行方法等，setup里的时间钩子绑定有点像事件监听，onMounted、onUpdated等,因为其就在beforeCreate时间钩子前，所以没提供onBeforeCreate和onCreated,也可以设置watch监听，形式为
```js
watch(变量名，(newValue,oldValue)=>{})
```
4、如果你想解构props，通过 toRefs(props) 将props的解构行为变为响应式，否则解构使用响应式会失效

5、将统一逻辑关注点的代码抽离成一个单独JS文件，并暴露所需要的相关属性供后面选项式API使用

6、众观一看，一个个逻辑功能块，清晰明朗，方便多人员开发。

详细请自行查看组合式API章节
- https://vue3js.cn/docs/zh/guide/composition-api-introduction.html#%E4%BB%80%E4%B9%88%E6%98%AF%E7%BB%84%E5%90%88%E5%BC%8F-api
