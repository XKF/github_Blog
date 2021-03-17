# 认识Webpack5

## 官方更新的主要方面

1、尝试用持久性缓存来提高构建性能。  
2、尝试用更好的算法和默认值来改进长期缓存。  
3、尝试用更好的 Tree Shaking 和代码生成来改善包大小。  
4、尝试改善与网络平台的兼容性。  
5、尝试在不引入任何破坏性变化的情况下，清理那些在实现 v4 功能时处于奇怪状态的内部结构。  
6、试图通过现在引入突破性的变化来为未来的功能做准备，使其能够尽可能长时间地保持在 v5 版本上。

## 比较大的变更（不兼容）
1、如果项目中还有使用webpack4版本已经废弃但没明确禁止使用的功能，v5版本都将被全面移除和禁止，若仍使用会警告报错（如IgnorePlugin 和 BannerPlugin），
以及一些语法废弃变更，可自行官方文档查看。

2、不兼容node了，在webpack5以前的版本会对node.js模块自动引用polyfill（垫片）以兼容浏览器版本，只要某个模块引用到node相关核心模块，webpack就会自动加垫片，v5 里将全面专注前端模块兼容，不再自动填充垫片，跟我们相关的话应该就是 process 这个对象了


## 打包文件的长期缓存

### 1、ChunkId方面
新长期缓存算法会给模块或分块分配（3或5）数字ID，避免以前版本打包完会有散列的模块影响gzip性能。

### 2、内容哈希[contenthash]

webpack4的哈希是结构哈希，只要结构一变化就会重新生成新的，那么缓存也就失效了，v5 转变为真实内容哈希，根据真正的文件内容，特别像是修改注释或者变量名这些压缩后由于不可见所以哈希值不会变，对长期缓存有积极影响


## 代码命名

以前我们异步加载模块如果不配置/* webpackChunkName: 'xxx' */,默认会帮我们打包出0.js,1.js这种，现在 v5 的新属性让我们不用再配置 webpackChunkName 就可以生成有上下文路径决定的块ID，但如果着实需要还是可以配置的

```js
import("./async.js").then((_)=>{
    console.log(_.data);
})
```
上述会打包出一个 src_async_js.js 在 dist 里

## 新增新特性(只抽了几个常用的)

1、资源模块导入：
旧方式：import url from "./image.png" 和 在module.rule 中设置 type: "asset"  
新方式：new URL("./image.png", import.meta.url)

2、支持异步模块了

## tree-shakin的优化(感受到了生存的压力)

### 1、嵌套的tree-shaking
以前export对象中嵌套属性未使用时未被删除，如今已经优化处理了
![image](B71FC1D977D6486CAD43B46474C01D56)

### 2、内部模块的tree-shaking 
webpack 4 没有分析模块的导出和引用之间的依赖关系。webpack 5 有一个新的选项 optimization.innerGraph，在生产模式下是默认启用的，它可以对模块中的标志进行分析，找出导出和引用之间的依赖关系。
![image](3E3769E298044D7FB39FB5CDCB6A50C5)

### 3、CommonJs Tree Shaking
增加多commonJs和require()的分析，除非webpack检测到不可分析的代码，不然也可以跟踪引用进行tree shaking了

### 4、副作用分析
除了package.json 中手动标记的 "sideEffects" 的无副作用模块外，webpack 5 也会对源码进行静态分析进行无副作用标记，在能shaking掉时shaking

### 5、export * 的优化
细节不必知道了，还有：  
import() 允许通过 /* webpackExports: ["abc", "default"] */ 该魔法注释手动 tree shake 模块。

## 构建优化

1、对于在生产模式下出现的错误，在开发模式下却没有的情况进行优化，以前是package.json的"sideEffects"标记不正确导致的，现在一致性更好了，能更快发现定位问题

2、不再只生成ES5代码，现在也可以生成ES6/ES2015代码

3、只支持现代浏览器，将使用箭头函数生成更短代码，旧浏览器岂不是

4、target配置优化，选项更加全面详细，不再只是“web”和“node”之间粗略的选择

## 性能优化

### 1、持久缓存
![image](E1E482143C844A85942534CE38830BCF)

wepback的缓存类型将开启文件系统来实现长效缓存，将cache的type改为‘filesystem’可以开启持久缓存

一方面，持久缓存忽略node_modules的哈希值和时间戳，这俩个因素不会再让缓存失效

另一方面，持久缓存也回来带来如下问题：  
1、mode之类的变化无法响应，缓存不会变。  
2、如果根据不同的场景，有不同的babel配置等，也同样不会感知，依然会用旧的缓存。  
3、使用DefinePlugin注入的动态内容，全部不会变化。

解决方案：你可以通过更改cache配置的name或者version来让缓存失效  
![image](D52887209C954BF58D3D50252E98DD5E) ![image](0AB415DC62D34258A6DA532F00BECFD3)

**PS：但要注意你的name和version不应该是一直变化的，不然就起不了缓存的作用了**

## 最小支持Node.js >= 10.13.0(LTS)

## 配置变更（常用的）
1、entry: {} 现在可以赋值一个空对象（允许使用插件来修改入口）；

2、target 支持数组，版本及 browserslist，即一个配置对象；

3、移除了 cache: Object：不能再设置内存缓存对象

4、添加了 cache.type：现在可以在 "memory" 和 "filesystem" 间进行选择  
在 cache.type = "filesystem" 时，增加了新配置项：  
- cache.cacheDirectory  
- cache.name  
- cache.version  
- cache.store  
- cache.hashAlgorithm  
- cache.idleTimeout  
- cache.idleTimeoutForIntialStore  
- cache.buildDependencies  

5、resolve.alias 值可以为数组或 false

6、output.filename 可以设置为函数

...太多了其他自己看配置文档去吧

## 默认配置变更
![image](5625EECEF493468F9EF8326278BF0FA4)
...

## 用于缓存的插件
MemoryCachePlugin - 内存缓存  
FileCachePlugin - 持久性（文件系统）缓存

## 其他一堆微小改动，请自行文档

### 学习资源From:
1、++https://webpack.docschina.org/blog/2020-10-10-webpack-5-release/#changes-to-the-defaults++

2、++https://zhuanlan.zhihu.com/p/81122986++

3、++https://www.cnblogs.com/zhouyideboke/p/12058380.html++
