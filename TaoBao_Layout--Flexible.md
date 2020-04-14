# 手淘移动端适配及dpr学习心得

挺久前写的心得了，17年左右刚入职实习的时候，由于要开发适配移动端的H5页面，当时采用的策略就是手淘的移动端适配布局了，换句通俗的话说就是现在常说的rem布局了，最近刚好时不时抽时间整理以前存放在本地的学习笔记，就顺带把这篇学习心得也一起重新整理一下发上来。

## ◆ 比较不错的几篇Guide articles

可以先看看、学习学习

- Viewport的解读：[http://www.w3cplus.com/css/viewports.html](http://www.w3cplus.com/css/viewports.html)

- 手淘flexible方案自适应教程：[https://www.w3cplus.com/mobile/lib-flexible-for-html5-layout.html](https://www.w3cplus.com/mobile/lib-flexible-for-html5-layout.html)

- 手淘移动端自适应新方案教程（使用vw）：[http://www.w3cplus.com/css/vw-for-layout.html](https://www.w3cplus.com/mobile/lib-flexible-for-html5-layout.html)  
PS: 当然如果公司项目需兼容低版本内核手机的话vw和vh这些CSS属性兼容性依旧不是很好

## ◆ 自己的一些学习见解

&emsp;&emsp;目前已知各互联网公司来说UI设计师给的设计稿通常都是基于640px、750px以及1125px宽度为准，640px是基于iphone5，750px则基于iphone6、7，淘宝的Flexible方案是基于iphone6、7来的，也就是750px，这个可根据自己公司的需求进行修改，当然我觉得现在大多数公司的移动端页面设计稿大概率都是基于750px来的了。

&emsp;&emsp;Flexible布局方案会将视觉稿分成100份（开发人员的目的意在为了以后能更好的兼容vh和vw属性），即一份 7.5px，我们把每一份称为一个单位a，同时设定 1rem 单位为 10a，也就是 1rem 为 75px。用算式表达出来的话就是：  
- 1a   = 7.5px
- 1rem = 75px   

&emsp;&emsp;那么我们这个示例的设计稿按rem单位来分的话就分成了10份，也就是整个宽度为 10rem，<html>对应的  font-size 为 75px，那么CSS里的rem基准就是根据html的font-size属性大小来进行换算的，比如我有个 width: 2rem 换算到视口中就是宽为 150px , 就是说你设置完根 (html) 的相对字体单位大小 (font-size) 后，要在CSS中将相关的尺寸 (px) 根据你规定的 rem 计算倍率换算为 rem 单位就可以了。当然技术发展是飞快的，现在前端开发普遍都是用 webpack 打包发布，webpack 插件很多，像 postcss-pxtorem-include、px2rem 都能帮你在打包过程中自动将 px 转化为 rem 单位，免去自己繁琐的换算，你只要设置好换算比例，直接用 px 照着设计稿开工就行了。 

&emsp;&emsp;接着再说下我对dpr（设备像素比）的简单理解，主要是针对IOS端，因为他们用了 Retina 视网膜屏，在 Retina 屏幕上进行设计，文字尺寸、空间大小等等都应该遵照逻辑像素（设备独立像素）进行。按我们通常思维对屏幕的理解来说，一个设备独立像素是由一个物理像素组成的，即 dpr 为 1 的情况，也就是设备独立像素就等于物理像素，但实际并不是这样，特别是IOS的 Retina 屏，一个设备独立像素是由几个物理像素组成的，像iphone5、6是 2，iphone6 plus是 3，拿iphone6来说，那么其设备宽为 375 设备独立像素，dpr 为 2，实际物理像素就是 375 * 2 = 750 像素，即UI设计稿宽要定为 750px ,因为实际物理像素为 750px。而设备显示是按设备独立像素来的，所以也就是说flexible方案的font-size是按750物理像素设计的话设备渲染相当于渲染了750设备独立像素的网页（Retina屏的1像素是指1设备独立像素，相当于翻倍了），那么 dpr 折算后也就是实际物理像素是 1500 像素，为了避免这种情况，那么这时就需要设置meta标签进行0.5倍的缩放，才能让网页实际物理像素大小等于 750 像素。  

![avatar](https://github.com/XKF/github_Blog/blob/master/img/taoBao_flexible/screenshot_1.gif)  

![avatar](https://github.com/XKF/github_Blog/blob/master/img/taoBao_flexible/screenshot_2.jpg)  

&emsp;&emsp;这也会导致一个问题，由于图片是原比例导进去的，为防止缩放时图片失真模糊，可以准备2倍大（dpr为 2）或3倍大（dpr为 3）的图片，也就是现在切图中常看见的图片名称后缀带有 @2x 或 @3x 标志，通俗点说就是几倍大的图片，毕竟术语理解起来还是有点抽象。这里你可能会说是不是漏了安卓，安卓怎么能说，有点杂，绝大多数的安卓机 dpr 普遍还是 1，不过现在全名屏什么开始流行，dpr 为 2 的也越来越多，像我当年的小米Note顶配版的2K屏幕的 dpr 直接到 4 了，我记得有几款APP的内置网页没兼容好，许多图片被裁了，所以目前安卓也就没管太多。  

&emsp;&emsp;对于图片在 Retina 屏下失真需提供2倍图，可参考下如下解释：

&emsp;&emsp;理论上，1个位图像素对应于1个物理像素，图片才能得到完美清晰的展示。

&emsp;&emsp;在普通屏幕下是没有问题的，但是在Retina屏幕下就会出现位图像素点不够，从而导致图片模糊的情况。

![avatar](https://github.com/XKF/github_Blog/blob/master/img/taoBao_flexible/screenshot_3.jpg) 

&emsp;&emsp;如上图：对于 dpr=2 的 Retina 屏幕而言，1个位图像素对应于4个物理像素，由于单个位图像素不可以再进一步分割，所以只能就近取色，从而导致图片模糊(注意上述的几个颜色值)。

&emsp;&emsp;所以，对于图片高清问题，比较好的方案就是两倍图片(@2x)。

&emsp;&emsp;如：200×300(css pixel)img标签，就需要提供400×600的图片。

&emsp;&emsp;如此一来，位图像素点个数就是原来的4倍，在Retina 屏幕下，位图像素点个数就可以跟物理像素点个数形成 1 : 1的比例，图片自然就清晰了(这也解释了之前留下的一个问题，为啥视觉稿的画布大小要×2？)。

&emsp;&emsp;这里就还有另一个问题，如果普通屏幕下，也用了两倍图片，会怎样呢？

&emsp;&emsp;我们换算一下，在普通屏幕下，200×300(css pixel)的img标签，所对应的物理像素个数就是200×300个，而两倍图片的位图像素个数则是 200x2×300x2 => 200x300x4个，所以就会出现一个物理像素点对应4个位图像素点，那么它的取色也只能通过一定的算法(显示结果就是一张只有原图像素总数四分之一，专业术语称这个过程叫做downsampling)，肉眼看上去虽然图片不会模糊，但是会觉得图片缺少一些锐利度，或者是有点色差(但还是可以接受的)。

用一张图片来表示：

![avatar](https://github.com/XKF/github_Blog/blob/master/img/taoBao_flexible/screenshot_4.jpg) 

&emsp;&emsp;然后，rem主要用于那些需要大小随屏幕伸缩而适配尺寸的元素，像正文文字大小、细边框等不宜随屏幕伸缩的尺寸的则最好还是用 px 写死，这也是手淘为何字体，边框不用rem进行设计的原因。

&emsp;&emsp;考虑到字体的点阵信息，一般文字尺寸多会采用 16px 20px 24px等值，若以rem指定文字尺寸，会产生诸如21px，19px这样的奇数值，会导致字形难看，毛刺，甚至黑块，故大部分文字应该以px写死设置，当然按自己公司实际需求来，字体需要自适应大小的话也不是不可以用rem，像一般标题类文字，可能也有要求随屏幕缩放，且考虑到这类文字一般都比较大，超过30px的话，也可以用rem设置字体。而边框可能会出现折算后低于0.5 css像素的 border 而导致边框消失不见（部分dpr为 1 的设备），所以特别是 1像素 边框最好别用rem，写死 1px 是可以，但在 Retina 屏下回变粗，主要受 dpr 影响，实际视觉效果可能会渲染为2px或3px等，这里就涉及到 1px 边框处理问题，这个自己不清楚的话就去百度啦~

&emsp;&emsp;最后这是自己认为解读得最好的学者，本心得部分地方引用到此大神的理解：[http://div.io/topic/1092](http://div.io/topic/1092)



