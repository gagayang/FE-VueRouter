#vue-router的源码分析

vue-router 大家都不陌生了。这个是vue的路由导航。这个作者呢也是vue的作者尤雨溪  Evan You

大神就是厉害啊 我看很多的vue的周边都是他写的。

其实也是废话，如果他只弄一个vue 周边没有扩展的那么好，鬼才用。

那么我们就开始源码分析吧

开始之前先说点题外话

我们看到vue 在官网上号称是渐进式的框架。那么它为什么说是渐进式呢？

其实就是他提供了这种install 机制。其实就是

vue.use()

这个里面就是加入了vue的生命周期。beforecreate 就开始监听事件

这里配置里面有mode 选项 分别对应着 hash 、history、abstract这三种模式

这里要简单先说明下

hash的模式一般是用来适配老的浏览器、这个使用#作为茅点/后面的作为路由路径

主要实现是通过监听hash的change事件来匹配不同的组件完成。

history 这个在react 中也叫做bowerhistoy模式 这个是适应现代新的浏览器。这个就是直接在根下面增加路径配置。这个需要说明下在服务器端的时候需要nginx配合。不然这个就是服务器的文件夹路径。
主要的实现原理是 h5标准的新api pushState

abstratc这种一般是在node里面。这个是通过堆栈来模拟了一套浏览器的histoy。这个是在ssr的时候使用

这里还要说明下

在vue-router 的配置里面可以进行平滑降级。比如你配置使用了history模式 但是你的浏览器是老旧版本不能使用。如果你配置了平滑降级。那么就可以默认降低到 hash模式下。

现在来分析一下里面的实现原理

1、老生常谈 在vue里面的一个重要机制就是install机制。
在这里面我们看看干了啥
	
	这里Evan 给vue里面混入（mixin）了两个生命周期
		1、beforeCreate
			这个里面重要的就是他进行了初始化路由。调用了router.init 进行初始化
			第二个很重要的就是他使用Vue.util.defineReactive这个方法就是定义了vue实例里面的一个响应属性（类似在每一个实例里面的data 增加了一个参数）这样在这个参数改变的时候就有render发生			
			然后就是注册这个实例
			
		2、destoryed 就是销毁之后取消注册实例
		
上面就是混入的内容，接下来 做了两个属性

当我们get $router 和 $route的时候分别返回什么内容

然后就是注册 RouterView 和 RouterLink组件

然后出现了另一个比较重要的东西  Vue.config.optionMergeStrategies

这里它就把我们的一些生命周期的hooks 给合并了。


接下来我们来看看 index.js 做了什么内容

先看看 coustructor方法吧
	这里就我们new的时候调用 来看看
	
	里面主要做了这么几件事
	1、创建了一个history 
		根据选择的模式和是否降级来创建 不同的 HashHistory\HTML5History\abstract 
	2、创建了this.matcher 来创建一个匹配器
	
我们接下来看看init 初始化方法

	这里我们看到是把传入的实例 放入apps里面
	
	然后就是有个transitionTo方法。
	这里就通过不同history 来获取路径
	
	封装成一个router的object 然后这里就出现了一个重要的回调。
	就是transitionTo里面传递的oncomplate这里就完成之后 把每一个app里面的_route的实例都改掉了
	
	这里就呼应了上面的install的 Vue.util.defineReactive就可以从新render每一个了
	
然后我们简单查看下router-view的函数就是找到匹配的路由 然后直接调用里面的component的render 

我们再简单看看 link 组件做了啥


好吧 其实我也没看

如果让我写的话 我就会写一个a标签 然后里面就是样式不同。然后点击的时候根据设置的mode不同然后去replace hash 或者是直接的路径。

看了下 差不多。当然是比我想的全面。（这不废话么，如果你想的比大神全面。你不就大神了么）
				
					
	

