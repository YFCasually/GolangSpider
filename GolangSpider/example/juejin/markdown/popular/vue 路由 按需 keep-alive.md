# vue 路由 按需 keep-alive #

![banner](https://user-gold-cdn.xitu.io/2019/5/19/16acfee7031de2c7?imageslim)

# situation #

一个常见的的场景，

主页 --> 前进 列表页 --> 前进 详情页，详情页 --> 返回 主页 --> 返回 列表页

我们希望，

从 详情页 --> 返回 列表页 的时候页面的状态是缓存，不用重新请求数据，提升用户体验。

从 列表页 --> 返回 主页 的时候页面，注销掉列表页，以在进入不同的列表页的时候，获取最新的数据。

# task #

今天 让我们来实现这个需求。

在 代码的世界里 解决问题的 方法 从来都不止一种。

比如，从数组中找到一个的值索引：

可以用最基础的 for循环 遍历，也可以用indexOf这种工具函数，还可以用findIndex forEach等更语义化的高阶函数来遍历。

我参考了大量的文章，爬了官方的文档，之后，找到了好几种办法，下面，让我们来一个个分析。

**let's begin** ， 我们开始吧。🙌

* 

解决掉提需求的人

😂 emm... ，相对于我们的人生，可能代码会更好掌控吧。

> 
> 
> 
> 程序员活在自己想象的王国里
> 
> 
> 
> 我刚接触电脑就发现电脑的妙处，电脑远没有人那么复杂。如果你的程序写得好，你就可以和电脑处好关系，就可以指挥电脑干你
> 想干的事。这个时候你是十足的主宰。 每每你坐在电脑面前，你就是在你的王国里巡行，这样的日子简直就是天堂般的日子。
> 电脑里的世界很大，编程人是活在自己想象的王国里。你可以想象到电脑里细微到每一个字节、每一个比特的东西。
> 
> 

> 
> 
> 
> -- 雷军(没错就是小米那个雷军)
> 
> 

我很认同雷军的说法，

人生能找到一个 热爱 的东西很难得，你 热爱代码 并 专注于它的话，

终有一天，

它会以各种各样的方式回报给你：工资，认同感，成就感，和你一起进步，心里共鸣的人。

你若盛开，蝴蝶自来。诸君共勉 (能骗一个相信，就少一个沙雕和我抢妹子🤣)

![沙雕](https://user-gold-cdn.xitu.io/2019/5/19/16acfef91567de63?imageslim)

* 

睡服♂提需求的人

😂emm..., 看着镜子里那张的脸，这辈子应该是没办法从脸上得到任何的便利了...

![富婆](https://user-gold-cdn.xitu.io/2019/5/19/16acff5d30ff80d6?imageView2/0/w/1280/h/960/ignore-error/1)

* 

keep-alive

` keep-alive` 是 ` Vue` 提供的一个抽象组件，主要用于保留组件状态或避免重新渲染。

![](https://user-gold-cdn.xitu.io/2019/5/19/16acff709701145a?imageView2/0/w/1280/h/960/ignore-error/1)

但是 ` keep-alive` 会把其包裹的所有组件都缓存起来。

# action #

我们把需求分解成2步来实现

## 1.把需要缓存和不需要缓存的视图组件区分开 ##

思路如下图:

![router区分](https://user-gold-cdn.xitu.io/2019/5/19/16acffc9192eb757?imageView2/0/w/1280/h/960/ignore-error/1)

* 写2个 ` router-view` 出口
` < keep-alive > <!-- 需要缓存的视图组件 --> < router-view v-if = "$route.meta.keepAlive" > </ router-view > </ keep-alive > <!-- 不需要缓存的视图组件 --> < router-view v-if = "!$route.meta.keepAlive" > </ router-view > 复制代码` * 在 ` Router` 里定义好需要缓存的视图组件
` new Router({ routes : [ { path : '/' , name : 'index' , component : () => import ( './views/keep-alive/index.vue' ), meta : { deepth : 0.5 } }, { path : '/list' , name : 'list' , component : () => import ( './views/keep-alive/list.vue' ), meta : { deepth : 1 keepAlive: true //需要被缓存 } }, { path : '/detail' , name : 'detail' , component : () => import ( './views/keep-alive/detail.vue' ), meta : { deepth : 2 } } ] }) 复制代码`

## 2.按需keep-alive ##

![keep-alive](https://user-gold-cdn.xitu.io/2019/5/19/16ad00649bb73773?imageView2/0/w/1280/h/960/ignore-error/1)

![include](https://user-gold-cdn.xitu.io/2019/5/19/16ad008732ddf825?imageView2/0/w/1280/h/960/ignore-error/1)

我们从官方文档提供的api入手,

` keep-alive` 组件如果设置了 ` include` ，就只有和 ` include` 匹配的组件会被缓存，

所以思路就是，动态修改 ` include` 数组来实现按需缓存。

![include](https://user-gold-cdn.xitu.io/2019/5/19/16ad009cfdc4879b?imageView2/0/w/1280/h/960/ignore-error/1)

` < keep-alive :include = "include" > <!-- 需要缓存的视图组件 --> < router-view v-if = "$route.meta.keepAlive" > </ router-view > </ keep-alive > <!-- 不需要缓存的视图组件 --> < router-view v-if = "!$route.meta.keepAlive" > </ router-view > 复制代码`

让我们在app.vue里监听路由的变化,

` export default { name : "app" , data : () => ({ include : [] }), watch : { $route(to, from ) { //如果 要 to(进入) 的页面是需要 keepAlive 缓存的，把 name push 进 include数组 if (to.meta.keepAlive) { ! this.include.includes(to.name) && this.include.push(to.name); } //如果 要 form(离开) 的页面是 keepAlive缓存的， //再根据 deepth 来判断是前进还是后退 //如果是后退 if ( from.meta.keepAlive && to.meta.deepth < from.meta.deepth) { var index = this.include.indexOf( from.name); index !== -1 && this.include.splice(index, 1 ); } } } }; 复制代码`

需要注意的是

![](https://user-gold-cdn.xitu.io/2019/5/19/16ad018aed5b0a02?imageView2/0/w/1280/h/960/ignore-error/1) ` keep-alive` 需要其包裹的组件有name属性，

我们上面的代码中的 ` push` 和 ` splice` 的是 ` router` 的 ` name` 。

所以建议大家把 ` route` 的 ` name` 属性设置和 ` route` 对应 ` component` 的 ` name` 设成一样的。

![就是这个意思](https://user-gold-cdn.xitu.io/2019/5/19/16ad01ec0d57e21d?imageView2/0/w/1280/h/960/ignore-error/1)

# result #

让我们验证一下我们的成果

![banner](https://user-gold-cdn.xitu.io/2019/5/19/16acfee7031de2c7?imageslim)

![](https://user-gold-cdn.xitu.io/2019/5/19/16ad0240ea84a2e8?imageView2/0/w/1280/h/960/ignore-error/1)

在 [vue-devtool]( https://link.juejin.im?target=https%3A%2F%2Fgithub.com%2Fvuejs%2Fvue-devtools ) 里，灰色的组件，代表是缓存状态的组件，注意观察 ` list` 的变化。

# 写在最后 #

* 实现按需 ` keep-alive` ，网上有方法，通过修改 ` route` 配置里的 ` meta` 里的 ` keepAlive` 值来实现。

![](https://user-gold-cdn.xitu.io/2019/5/19/16ad02b3cd17da4f?imageView2/0/w/1280/h/960/ignore-error/1)

直接修改 ` meta` 的值，可能会出现上图的情况，keep-alive里有一直有一个缓存的 list,正常的 ` rotuer-view` 里也有一个,

复现这个问题需要很长得篇幅，感兴趣的朋友可以自己去爬一下坑。

* 还有得方法是 通过在keep-alive 的视图组件在退出 rotuer 的时候，调用this.$destory() 直接摧毁组件，这会导致组件没法在缓存，这个bug ，在官方issue有提到。

# 参考资料 #

> 
> 
> 
> [官方issue](
> https://link.juejin.im?target=https%3A%2F%2Fgithub.com%2Fvuejs%2Fvue%2Fissues%2F6509
> )
> 
> 

> 
> 
> 
> [Vue项目全局配置页面缓存，实现按需读取缓存
> ]( https://juejin.im/post/5b5eb85d6fb9a04fb745e8f8
> )
> 
> 

# 关于动画 #

谢谢大家的点赞和回复，我看好多人想要demo的源码这里分享一下。

因为我公司目前还没有,vue的项目给我施展拳脚。我当时写demo的时候，想的也不是动画方面的封装和优化，

所以分享的demo里的代码里，关于动画，会有很多重复的代码和丑陋的实现。还请多见谅

[源码链接]( https://link.juejin.im?target=https%3A%2F%2Fshare.weiyun.com%2F5XiSQDy )

关于demo动画的一些分析，可以看我的前2篇文章