# HTML、CSS、JavaScipt及Vue踩坑记录及总结

最近相对有空，把这2年多记的工作学习总结及踩坑日志搬出来的几个word文档搬出来重新整理归纳一波，可能这些注意点平常到不能再平常，但工作久了可能还是会偶尔地脑筋一懵就踩了进去，现在markdown到github上来方便查看，如果有前端新人刚入门也可以提前注意下，sure~前端大神就可以略过啦，我还想找机会向你们学习补充知识点呢(#^.^#)

## ◆ HTML

1. Img 标签的 src 为空时浏览器会默认加上边框，可通过设置以下CSS样式避免：  

&emsp;&emsp;img[src=""],img:not([src]){  
&emsp;&emsp;&emsp;&emsp;opacity: 0;  
&emsp;&emsp;}

## ◆ CSS

1. 样式选择器不要嵌套太多层，少用标签选择器

2. id选择器也少用点,特别是以JQuery为技术栈的时候，id主要用在js的元素选择，样式匹配多用class

3. 列表序号可以灵活使用css的自增属性+伪元素:before实现自定义自增序号

4. flex布局很强悍，但如果需求要适配到一些古董机去的话就要注意下兼容性问题了

5. CSS中像伪元素的content属性等少用中文，容易出现乱码

6. rem布局对横屏的自适应比较乏力，需自己再去完善下

7. position: sticky   
CSS3新增的一个布局定位属性，可以尝试使用，如果对兼容性要求比较高的话不建议使用，通过指定top、bottom、left、right四个值之一，使像导航栏这些元素滑动到距视口指定距离时将浮窗固定住  
有关学习链接：[https://www.jianshu.com/p/b72f504121f5](https://www.jianshu.com/p/b72f504121f5)

8. animation属性的steps功能好好学习下，比较好地控制帧数变化  
有关学习链接：[https://www.zhangxinxu.com/wordpress/2018/06/css3-animation-steps-step-start-end/](https://www.zhangxinxu.com/wordpress/2018/06/css3-animation-steps-step-start-end/)

9. background-image背景图片并不像img标签被display:none时图片仍会被加载进来，图片加载的时机是该类首次被用到时

10. background-image不能作为CSS animation动画变化的属性，W3C标准中已经明确background-image属性是不支持动画的

11. transform实现居中样式时存在元素高度为奇数而导致字体模糊的现象（目前在MacOS上发现这种情况），可以用-webkit-font-smoothing: antialiased解决

12. 给父元素加上 transform:translate(0,0) ，子元素 fixed 定位会根据父容器进行定位，即如果父级元素设置了 transform 属性，子元素设置 position:relative/absolute/fixed 定位时会基于此父级元素定位，但 transform 跟 fixed 属性结合使用容易引起一些历史遗留问题，最好少用

13. (针对小白萌新) CSS类命名要规范，用英文、通用术语等别用拼音，还有class命名不能数字开头...

14. 表格td宽高的设置，若想根据自己宽度来计算的话设置 table-layout: fixed  
    |&emsp;&emsp;值&emsp;&emsp;|&emsp;&emsp;描述&emsp;&emsp;|  
    | automatic	| 列宽度由单元格内容设定(default) |  
    | fixed	| 列宽由表格宽度和列宽度设定 |  
    | inherit |	规定应该从父元素继承 table-layout 属性的值 |

15. 如果你给元素设置了CSS3动画效果，不要使用 display:none 进行隐藏，用display隐藏好比动画属性找不到你指定的元素，所以会导致设置的动画不生效，可改用 visibility: hidden 去实现一个动画元素的隐藏

16. iOS 移动端存在 input 输入框聚焦时软键盘会把 fixed 定位的控件撑上去的问题，同时整个 body 的高度也会发生变化（感知现象就是 body 也被顶上去了），在恢复时即软键盘收起会发生光标停留在原位而导致错位的问题，而安卓部分机型也存在软键盘把 body 顶上去的情况，而且还把 body 高度直接给裁切了，从而会导致诸如 输入光标错位、 控件错位、 聚焦失败、rem布局下字体大小变化等问题的出现。  
解决方法： 
    - 针对iOS的光标错位，监听输入框失焦事件，在失去焦点时，即软键盘收起，把body手动滚至顶部，重新对好位置；
    
    - 针对部分安卓机子软键盘弹起把body高度裁切，监听输入框聚焦事件，在软键盘弹起时，把body手动滚至底部，重新对好位置；

    - 针对控件错位，rem字体大小变化等，可以监听窗口大小变化的 resize 事件，计算窗口大小变化前后高度差，跟主流手机软键盘的平均高度进行比较，符合软键盘高度范围内时认为是软键盘弹出事件，进行相关的样式操作

17. 子元素有 transform 属性时，父级元素需设置 transform：rotate(0deg)，避免部分移动机型出现设置圆角属性 border-radius 且用 overflow: hidden 进行裁切时出现 overflow: hidden 失效的问题

18. 文字垂直居中受不同手机字体行高的影响会有1-2px偏移的误差，目前已知的解决方案只有切图可以完全解决，当然上次看了一篇阿里大神的知乎文章，针对安卓7.0以上的设备，可以通过在html标签上设置 lang 属性：\<html lang="zh-cmn-Hans"\>，同时 font-family 不指定英文，如  font-family: sans-serif   
参考学习链接：[https://www.zhihu.com/question/39516424](https://www.zhihu.com/question/39516424)

19. 移动端rem布局容易出现设置 1px 边框后有 1px 空白间隙的情况，大致推断是rem计算误差导致的，改用 box-shadow 或者利用伪元素扩大后缩放实现 1px 边框效果解决

20. 移动端rem布局时容易出现图标被裁切了 1px 或者 元素背景设置渐变时少渲染 1px 的问题，导致这种现象的原因是浏览器的渲染机制。  
参考链接：[https://blog.csdn.net/tomihaohao/article/details/49781807](https://blog.csdn.net/tomihaohao/article/details/49781807)  
解决方案：
    - 图标的话切图时可以四周预留多至少 1px 透明；
    - 背景渐变的话目前还没有较好的解决方法，建议元素的宽度多设置为偶数，当然直接点的话背景渐变用切图来实现的话也可以解决。  
      
21. 移动端IOS8系统存在 input 输入框 readonly 属性无法生效的问题，即点击 readonly 属性的input输入框还是会出现光标和软键盘，添加 user-select:none 属性后光标和软键盘主体消失，但软键盘头部仍在，UI和测试大佬不通过这种方案，排除。后通过监听 readonly 属性输入框的 focus 事件，通过执行 document.activeElement.blur() 令其聚焦的同时失去焦点，成功解决此问题，但仍存在一点瑕疵，使用此方法后点击指定输入框页面会自动将此输入框滚动至原软键盘弹起后输入框被顶起的位置，只不过现在软键盘和光标未出现，最后采取了此种方案。

22. 移动端IOS系统下 input 输入框用 input 事件限制只能输入数字时IOS的原生软键盘的集合符号按钮( [, . ; :] 那个集合按钮)会变成执行回退删除的逻辑，目前还没有找到比较好的解决方法，有大佬有比较好的兼容方法的话望留言，谢谢！

## ◆ JavaScript（叫ECMAScript是不是好一点）

1. 懒加载最好提前一点加载，也就是判断跟底部的距离加长一点，别真的滑到底部再加载，体验不够顺滑

2. JQuery技术栈下哪些属性用attr()访问哪些用prop()访问？
    - 第一个原则：只添加属性名的属性就会生效应该使用prop
    - 第二个原则：只存在true/false的属性应该使用prop()，例如disabled,checked这些，其他可用attr()

3. 减少不必要的 DOM 操作，减少性能的损耗

4. 如果页面考虑多种浏览器版本的兼容性要注意插件或脚本库什么的不一定最新就是最好的，有些属性方法是不兼容的

5. 函数提升优先级比变量提升要高，同名时不会被变量声明覆盖，但是会被变量赋值覆盖

6. 像下拉加载这种有用到onscroll记得做好节流和防抖

7. iOS 11.3后添加 fastclick 后 input 点击聚焦会出现不灵敏的问题，解决方法为重构 faskclick 原型上的 focus 函数：  
	let deviceIsWindowsPhone = navigator.userAgent.indexOf("Windows Phone") >= 0;  
	let deviceIsIOS = /iP(ad|hone|od)/.test(navigator.userAgent) && !deviceIsWindowsPhone;

	/** 解决加了fastclick后ios11.3出现的点击input聚焦不灵敏的问题 */  
	FastClick.prototype.focus = function(targetElement) {

	    &emsp;&emsp;let length;

	/** 兼容处理:在iOS7中，有一些元素（如date、datetime、month等）在setSelectionRange会出现TypeError */

	/** 这是因为这些元素并没有selectionStart和selectionEnd的整型数字属性，所以一旦引用就会报错，因此排除这些属性才使用setSelectionRange方法 */

	&emsp;&emsp;if (deviceIsIOS && targetElement.setSelectionRange && targetElement.type.indexOf('date') !== 0 && targetElement.type !== 'time' && targetElement.type !== 'month' && targetElement.type !== 'email') {

	&emsp;&emsp;&emsp;&emsp;length = targetElement.value.length;

	&emsp;&emsp;&emsp;&emsp;targetElement.setSelectionRange(length, length);

	/** 修复bug ios 11.3不弹出键盘，这里加上聚焦代码，让其强制聚焦弹出键盘 */

	&emsp;&emsp;&emsp;&emsp;targetElement.focus();

	&emsp;&emsp;} else {

	&emsp;&emsp;&emsp;&emsp;targetElement.focus();

	&emsp;&emsp;}

	};  

8. iOS 会出现聚焦input后软键盘弹出会导致绝对定位或固定定位的input框光标错位的问题，监听聚焦（即软键盘弹出与否）事件，手动调整文档位置或动态调整定位样式

9. ES6的yield理解  
    - 只能在Generator(遍历器)函数(function* funcname(){ yourcode })内部使用；
    - 运行一次.next()，执行一个yield命令；
    - .next()返回的是一个状态对象{value,done}，yield命令是没返回值的；
    - 再次运行.next()，从之前暂停的那个yield [表达式]后继续运行，类似于断点；
    - 当.next()传参的时候，yield [表达式] = 被传入的参数；
    - .next()遇到return时退出遍历，即{value,done}中done的状态变为true

10. 移动端浏览器用location.href跳下载链接安装下载包时会阻塞原网页未加载完毕的资源的加载，虽说视觉上停留在当前页，但其时url已经跳去访问下载链接了

11. 日期格式化的坑：  
在IOS设备中不支持 new Date("2018-04-27 11:11") 这种字符串格式，仅支持 new Date("2018/04/27 11:11") 这种格式，所以为了兼容IOS和Android端，将日期字符串格式化后再传进new Date()中，即 new Date("2018-04-27 11:11".replace(/-/g, "/"));

12. history.replaceState 只能替换同源的url，而且要基于当前的链接进行正则修改，格式化为同源的指定url

13. 现在浏览器中用 history.replaceState(null,””,null) 清除浏览记录由于浏览器的安全策略已不起效

14. ( ES6 ) Async/await 函数本身就是个异步函数，只不过函数里面是同步的，比如函数里的await后的操作要等待await操作执行完才能执行

## ◆ Vue + Webpack

1. Vue防止组件缓存而不再执行有关逻辑时可以使用时间钩子activated

2. keep-alive 可以缓存组件，但 keep-alive 标签下必须只包含一个元素，多个元素的话只缓存第一个元素，如果想缓存多个元素，可以把 keep-alive 放在 v-for 里面

3. Vue单元测试：Mocha的学习

    - 测试脚本与所要测试的源码脚本同名，但是后缀名为.test.js（表示测试）或者.spec.js（表示规格）

    - 测试脚本里面应该包括一个或多个describe块，每个describe块应该包括一个或多个it块。

    - describe块称为"测试套件"（test suite），表示一组相关的测试。它是一个函数，第一个参数是测试套件的名称（"加法函数的测试"），第二个参数是一个实际执行的函数。

    - it块称为"测试用例"（test case），表示一个单独的测试，是测试的最小单位。它也是一个函数，第一个参数是测试用例的名称（"1 加 1 应该等于 2"），第二个参数是一个实际执行的函数。

4. 调试工具查看 Object 输出时，如果这个 Object 是异步获取的，你在异步获取完成前输出了控制台显示的是 Object{}，但当你点展开时控制台会再去内存拉一次当前 Object 最新数据

5. Webpack打包时记得将第三方依赖单独抽出来，vendor.js、 manifest.js ，抽出后记得在入口处加chunks指定名称引入


## ◆ Fisp 、Fis3 与 Fis3-smarty (公司以前用的构建工具)

1. Fis3的发布是增量式发布，所以要发布到服务器上时要特别的慎重，因为发错的话只能让后台人员去删改，很麻烦，所以一定要慎重

2. Fis3-smarty设置配置用fis.set，而Fisp设置配置用fis.config.set

3. $smarty.foreach 遍历中 .index 索引从 0 开始， .iteration 索引从 1 开始

## ◆ Fis 与 Webpack 之间的区别(个人理解)
1. Fis 相对 Webpack 来说对npm开源库社区的兼容性比较差，对 node 版本要求也比较苛刻，可能升级 node 版本某个 npm 包就用不了了，而且 Fis 使用 npm 包需要先 fis.hook('node_modules') 把 node_modules 路径引进来，才能再直接去 require 插件，webpack 可以直接引入插件，默认会去node_modules 里去寻找；

2. Fis 的 require 引入依赖于自家提供的 mod.js；

3. Fis 可以生成静态资源依赖表，webpack没有，当然你可以用 webpack-bundle-analyzer 来分析打包依赖，但这插件主要还是用来一个打包性能的分析，个人感觉是没Fis的静态资源依赖表直观的；

4. 因为自己是从 Fis 转 Webpack，相对 Webpack 来说 Fis的配置比较直观粗暴点，而且每个项目都不比较大同小异，按需求来说像一些比较容易的纯前端静态页面什么的我可能还是会直接用Fis 构建一个前端工程去做，比较直接快捷，但因为现在 Fis 已经停止维护了，所以最好还是统一 Webpack吧
