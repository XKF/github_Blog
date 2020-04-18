# 移动端H5开发-iOS常见兼容性问题汇总

移动端H5开发的页面在低版本iOS系统下可能会存在某些兼容性问题或不科学现象，这里汇总一下，部分问题已有解决方法，也有极少部分问题没有比较好的方法去解决，希望有大佬可以指点一波~

## 一、CSS相关

1. iOS 移动端存在 input 输入框聚焦时软键盘会把 fixed 定位的控件撑上去的问题，同时整个 body 的高度也会发生变化（感知现象就是 body 也被顶上去了），在恢复时即软键盘收起会发生光标停留在原位而导致错位的问题，而安卓部分机型也存在软键盘把 body 顶上去的情况，而且还把 body 高度直接给裁切了，从而会导致诸如 输入光标错位、 控件错位、 聚焦失败、rem布局下字体大小变化等问题的出现。  
解决方法： 
    - 针对iOS的光标错位，监听输入框失焦事件，在失去焦点时，即软键盘收起，把body手动滚至顶部，重新对好位置；
    
    - 针对部分安卓机子软键盘弹起把body高度裁切，监听输入框聚焦事件，在软键盘弹起时，把body手动滚至底部，重新对好位置；

    - 针对控件错位，rem字体大小变化等，可以监听窗口大小变化的 resize 事件，计算窗口大小变化前后高度差，跟主流手机软键盘的平均高度进行比较，符合软键盘高度范围内时认为是软键盘弹出事件，进行相关的样式操作

2. 子元素有 transform 属性时，父级元素需设置 transform：rotate(0deg)，避免部分移动机型（之前在iOS低版本机子看到过）出现设置圆角属性 border-radius 且用 overflow: hidden 进行裁切时出现 overflow: hidden 失效的问题。

3. 移动端IOS8系统存在 input 输入框 readonly 属性无法生效的问题，即点击 readonly 属性的input输入框还是会出现光标和软键盘，添加 user-select:none 属性后光标和软键盘主体消失，但软键盘头部仍在，UI和测试大佬不通过这种方案，排除。后通过监听 readonly 属性输入框的 focus 事件，通过执行 document.activeElement.blur() 令其聚焦的同时失去焦点，成功解决此问题，但仍存在一点瑕疵，使用此方法后点击指定输入框页面会自动将此输入框滚动至原软键盘弹起后输入框被顶起的位置，只不过现在软键盘和光标未出现，最后采取了此种方案。

4. 移动端IOS系统下 input 输入框用 input 事件限制只能输入数字时IOS的原生软键盘的集合符号按钮( [, . ; :] 那个集合按钮)会变成执行回退删除的逻辑，目前还没有找到比较好的解决方法，有大佬有比较好的兼容方法的话望留言，谢谢！

5. 注意 iOS 的 Retina 屏的 dpr 问题，特别是 1px 边框，即使你写死 1px ，实际视觉效果会因为 dpr 显示出 1 x dpr 倍数像素的视觉效果

## 二、JS相关

1. iOS 11.3后添加 fastclick 后 input 点击聚焦会出现不灵敏的问题，解决方法为重构 faskclick 原型上的 focus 函数：  
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

2. 日期格式化的坑：  
在IOS设备中不支持 new Date("2018-04-27 11:11") 这种字符串格式，仅支持 new Date("2018/04/27 11:11") 这种格式，所以为了兼容IOS和Android端，将日期字符串格式化后再传进new Date()中，即 new Date("2018-04-27 11:11".replace(/-/g, "/"));