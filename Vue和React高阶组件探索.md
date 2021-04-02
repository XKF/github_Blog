# Vue和React高阶组件探索

## 什么是高阶组件
高阶组件即指组件通过一个高阶函数（对组件进行包装加工）生成出来的新组件，高阶处理的过程不改变原组件，相当于在原组件的基础上添加一些属性，或根据一些属性做一些逻辑操作，然后返回一个被包装好的组件。这也是组件代码复用的推荐做法。高阶组件在React里听得比较多，Vue几乎很少听到，但mixins混入总归知道吧，对，mixins也是代码复用的一种方式，只不过高阶组件弥补了mixins很多的不足，但为啥Vue还是以mixins为主，主要是Vue和React的设计思想不同，也不是说mixins不好，技术方面常听到这么一句话，没有不好的，只有最适合的，其实React之前版本也有mixins来的，所以那我们就先从混入mixins说起吧。

## mixins混入
Vue的mixins想必都用过，当你某个函数或者时间钩子操作需要复用，你很容易就想到了用mixins，然后在需要的组件中进行引用，实现复用，旧版的react其实也有mixins来的，在createClass里面配置
```js
const PureRenderMixin = require('react-addons-pure-render-mixin')
const MyComponent = React.createClass({
  mixins: [PureRenderMixin]
})
```
但是，用起来爽是爽，只要用到我就引入mixins，如果代码多了，必然会引起许多问题，所以React认为mixins模式不适合他们清爽的生态，主要是以下几点
- mixins 带来了隐式依赖，耦合度比较高
- 引用多个mixins时，代码一多就容易出现命名冲突
- mixins是侵入式，相当于并到你的组件中，那当代码量一上来的话，就会越来越乱，滚雪球式发展

所以，React新版本废除了mixins，大力倡导高阶组件HOC，当然也不是说HOC就是最好的，只是说最适合React当前框架，你看Vue还是mixins，但笔者不建议Vue也使用过多的mixins，原因也跟React团队提的差不多：
- 耦合性大
- 代码量多的时候杂乱  
笔者是建议多提取成工具库函数，按需引入，方便tree-shaking

那我接着就说说高阶组件HOC，刚好最近开始学React，就浅浅的表述下

## 高阶组件 HOC
1、高阶组件是个纯函数，一个包装函数，接收一个组件作为参数，通过给组件进行props等属性赋值或者做一些逻辑操作后将包装后的组件返回，当然这个过程不应该修改到原组件，即在原组件基础上修改

2、高阶组件不关心你的props传了什么，里面的被包装组件也不关心传给他的props是哪里来的，所以在返回被包装组件时，要将包装组件上的props，除了那几个规定用来进行逻辑操作的属性外的props，传给被包装组件，而不应该做删减

3、高阶组件不应该在render函数中穿件，高阶函数用来修饰类组件或者包裹函数式组件。

PS：某些情况下高阶组件的ref有可能指向的是外层容器组件而不是被包装组件，可通过传一个函数指定ref的指向解决此问题。

## 那Vue究竟有没高阶组件
答案是有的，但我们Vue是模板组件怎么办，没事，模板组件经过编译也会变成函数组件，但是，那是最终的结果，在我们的组件引用中，我们组件export default的是什么啊，对！是个选项对象，那我们做高阶函数肯定只能去修改对象啊，是的，Vue的高阶就是要通过接收一个选项对象，包装返回一个新的选项对象，进而实现高阶组件。看个例子
```js
export default function WithConsole (WrappedComponent) {
  return {
    //$listeners和$attrs好比React的高阶组件把组件的props和事件传回被包装组件
    template: '<wrapped v-on="$listeners" v-bind="$attrs"/>',
    components: {
      wrapped: WrappedComponent
    },
    //包装多了这个输出逻辑
    mounted () {
      console.log('I have already mounted')
    }
  }
}
```
但是没这么简单，其一，我们是不是漏了props属性呀，Vue的props可是要声明的，attrs可没包含声明的props属性；其二，template只能在完整版Vue包中使用，如果使用的运行版则不支持。所以我们还要传入多个$props以及将template转为我们的render函数。
```js
export default function WithConsole (WrappedComponent) {
  return {
    mounted () {
      console.log('I have already mounted')
    },
    render (h) {
      return h(WrappedComponent, {
        on: this.$listeners,
        attrs: this.$attrs,
        props: this.$props
      })
    }
  }
}
```
如果简单的话一个高阶组件这样就算完了，但是如果你的组件复用度还想高一点的话，是不是少了个什么东西？

想一想

对就是插槽，那插槽怎么引入呢，我这里就直接上代码了
```js
function WithConsole (WrappedComponent) {
  return {
    mounted () {
      console.log('I have already mounted')
    },
    props: WrappedComponent.props,
    render (h) {
      const slots = Object.keys(this.$slots)
        .reduce((arr, key) => arr.concat(this.$slots[key]), [])
        .map(vnode => {
          //这里为啥要这么做？
          vnode.context = this._self
          //我们可以通过this.$vnode中的 VNode来获取父实例插槽内容，因为内容是在父组件中被创建的，所以 this.$vnode 中的 context 引用着父实例；
          //因为我们包装成了WithConsole组件，而被包装组件的$slot是被我们高阶组件上的$slot直接透传下去的,
          //所以现在this.$vnode.context指向成了该高阶组件，而$slot的$vnode.componentOptions.children[0].context指向父实例；
          //Vue规定this.$vnode.context === this.$vnode.componentOptions.children[0].context不相等时具名插槽无效，当默认插槽处理;
          //所以我们要把vnode.context指回去之前的父实例避免具名插槽失效
          return vnode
        })

      return h(WrappedComponent, {
        on: this.$listeners,
        props: this.$props,
        // 透传 scopedSlots
        scopedSlots: this.$scopedSlots,
        attrs: this.$attrs
      }, slots)
    }
  }
}
```
## 总结
好了，Vue的高阶组件也就完成，当然肯定还有诸多细节问题，但简单的例子就是这样子了，看到这你可能会说卧槽Vue写个高阶这么麻烦，难道平常几乎没看到，怎么说呢，主要是俩个框架的设计思想不同吧，React相对来说更清爽、简洁，封装的功能没那么多，耍起来舒畅但是逻辑功能多靠自己原生去实现，有点像我们一加的氢系统，Vue的话封装得相对比较多，因此功能多，上手容易，比较友好，抄起满篇的API就是撸，但也因此失去了相应的灵活性，有点像小米手机MIUI系统的生态，所以各有各优缺点，没有最好只有最适合。

## Hooks
如今，React最新版推出**Hooks**
1、它可以让你在class以外使用state和其他React特性。  
噢！想到了什么？对React函数式组件也可以有state了。

2、以在将含有state的逻辑从组件中抽象出来，这将可以让这些逻辑容易被测试。

3、Hooks可以帮助你在不重写组件结构的情况下复用这些逻辑。所以，它也可以作为一种实现状态逻辑复用的方案。

好了由于React才刚入门，没怎么用到Hooks，这里就不细说暴露渣渣水平了，具体大家就去搜博客学习下吧。

## 参考文章
- http://hcysun.me/2018/01/05/%E6%8E%A2%E7%B4%A2Vue%E9%AB%98%E9%98%B6%E7%BB%84%E4%BB%B6/
- https://juejin.cn/post/6844903815762673671#heading-35
