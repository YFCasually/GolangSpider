# Flutter完整开发实战详解(十四、混合开发打包 Android 篇) #

本篇将带你深入了解 Flutter 中打包和插件安装等原理，帮你快速完成 Flutter 集成到现有 Android 项目，实现混合开发支持。

> 
> 
> 
> 前文：
> 
> 
> 
> * [一、  Dart语言和Flutter基础]( https://juejin.im/post/5b631d326fb9a04fce524db2
> )
> * [二、  快速开发实战篇]( https://juejin.im/post/5b685a2a5188251ac22b71c0 )
> * [三、  打包与填坑篇]( https://juejin.im/post/5b6fd4dc6fb9a0099e711162 )
> * [四、  Redux、主题、国际化]( https://juejin.im/post/5b79767ff265da435450a873 )
> * [五、  深入探索]( https://juejin.im/post/5bc450dff265da0a951f032b )
> * [六、  深入Widget原理]( https://juejin.im/post/5c7e853151882549664b0543 )
> * [七、  深入布局原理]( https://juejin.im/post/5c8c6ef7e51d450ba7233f51 )
> * [八、  实用技巧与填坑]( https://juejin.im/post/5c9e328251882567b91e1cfb )
> * [九、  深入绘制原理]( https://juejin.im/post/5ca0e0aff265da309728659a )
> * [十、  深入图片加载流程]( https://juejin.im/post/5cb1896ce51d456e63760449 )
> * [十一、全面深入理解Stream]( https://juejin.im/post/5cc2acf86fb9a0321f042041 )
> * [十二、全面深入理解状态管理设计]( https://juejin.im/post/5cc816866fb9a03231209c7c )
> * [十三、全面深入触摸和滑动原理](
> https://juejin.im/editor/posts/5cd54839f265da03b2044c32 )
> 
> 
> 

## 一、前言 ##

随着各种跨平台框架的不断涌现， **很多时候我们会选择混合开发模式作为脚手架** ，因为企业一般不会把业务都压在一个框架上，同时除非是全新项目，不然出于对原有业务重构的 **成本和风险** 考虑，都会选择混合开发去尝试入坑。

但是混合开发会对 **打包、构建和启动等流程熟悉度要求较高** ，同时遇到的问题也更多，以前我在 ` React Native` 也写过类似的文章 ： [《从Android到React Native开发（四、打包流程解析和发布为Maven库）》]( https://juejin.im/post/5b2116466fb9a01e3128359f ) ，而这方面是有很多经验可以通用的， **所以适当的混开模式有利于避免一些问题，同时只有了解 ` Flutter` 整体项目的构建思路，才有可能更舒适的躺坑。**

> 
> 
> 
> 额外唠叨一句，跨平台的意义更多在于解决多端逻辑的统一 ，至少避免了逻辑重复实现，所以企业刚开始，一般会选择一些轻量级业务进行尝试。
> 
> 

## 二、打包 ##

一般跨平台混合开发会有两种选择：

* 1、将 ` Flutter` 整体框架依赖和打包脚本都集成到主项目中。
* 2、以 ` aar` 的完整库集成形式添加到主项目。

两种实现方法各有利弊：

* 

第一种方式可以 **更方便运行时修改问题，但是对主项目“污染”会比较高，同时改动会大一些。**

* 

第二种方式 **需要单独调试后，更新 ` aar` 文件再集成到项目中调试，但是这类集成方式更干净，同时 ` Flutter` 相关代码可独立运行测试，且改动较小。**

一般而言，对于普通项目我是建议以 **第二种方式集成到项目中的** ，通过新建一个 ` Flutter` 工程，然后对工程进行组件化脚本处理，让它 **既能以 apk形式单独运行调试，又能打包为aar形式对外提供支持。**

相信对于原生平台熟悉的应该知道，我们可以通过简单修改项目 ` gradle` 脚本，让它快速支持这个能力，如下图片所示，图片中为省略的部分脚本代码，完整版可见 [flutter_app_lib]( https://link.juejin.im?target=https%3A%2F%2Fgithub.com%2FCarGuo%2Fflutter_app_lib ) 。

![](https://user-gold-cdn.xitu.io/2019/5/24/16ae9bb0802c5e4d?imageView2/0/w/1280/h/960/ignore-error/1)

我们通过了 ` isLib` 标记为去简单实现了项目的打包判断，当项目作为 ` lib` 发布时， **设置 ` isLib` 为 true，之后执行 `./gradlew assembleRelease` 即可** ，剩下的工作依旧是 ` Flutter` 自身的打包流程，而对于打包后的 ` aar` 文件直接在原生项目里引入即可完成依赖。

而一般接入时， **如果需要 ` token` 、用户数据等信息，推荐提供定义好原生接口，如 ` init(String token, String userInfo)` 等，然后通过 ` MethodChannel` 将信息同步到 ` Flutter` 中。**

对于原生主工程，只需要接入 ` aar` 文件，完成初始化并打开页面，而无需关心其内部实现，和引入普通依赖并无区别。

> 
> 
> 
> 你可能需要修改的还有 ` AndroidManifset` 中的启动 ` MainActivity` 移除，然后添加一个自定义 ` Activity`
> 去继承 ` FlutterActivity` 完成自定义。
> 
> 

## 三、插件 ##

如果普通情况下，到上面就可以完成 ` Flutter` 的集成工作了，但是往往事与愿违， **一些 ` Flutter` 插件在提供功能时，往往是通过原生层代码实现的，如 ` flutter_webview` 、 ` android_intent` 、 ` device_info` 等等，那这些代码是怎么被引用的呢？**

> 
> 
> 
> 这里稍微提一下，用过 ` React Native` 的应该知道，带有原生代码的 ` React Native` 插件，在 ` npm` 安装以后，需要通过
> ` react-native link` 命令完成安装处理。 **这个命令会触发脚本修改原生代码，从而修改 ` gradle`
> 脚本增加对插件项目的引用，同时修改 ` java` 代码实现插件的模版引入，这使得项目在一定程度被插件“污染”。**
> 
> 

在 ` React Native` 中带有原生代码的插件，会被以本地 ` Module` 工程的方式引入，那 ` Flutter` 呢？

其实原理上 **` Flutter` 带有原生代码的插件，在插件安装后，也是会以本地 ` Module Project` 的形式引入** ，但是它整个过程更加巧妙，让开发中对这个过程几乎无感。

如下图所示，不知道你注意过没有，在插件安装之后，所有带原生代码的插件，都会以路径和插件名的 ` key=value` 形式 **存在 `.flutter-plugins` 文件中。**

![](https://user-gold-cdn.xitu.io/2019/5/24/16ae9bb08d3db483?imageView2/0/w/1280/h/960/ignore-error/1)

而在 ` android` 工程的 ` settings.gradle` 里，如下图所示， **会通过读取该文件将 `.flutter-plugins` 文件中的项目一个个 ` include` 到主工程里。**

![](https://user-gold-cdn.xitu.io/2019/5/24/16ae9bb0950abfa5?imageView2/0/w/1280/h/960/ignore-error/1)

之后就是主工程里的 ` apply from: "$flutterRoot/packages/flutter_tools/gradle/flutter.gradle"` 脚本的引入了，这个脚本一般在于 ` flutterSDK/packages/flutter_tools/gradle/` 目录下，如下代码所示，其中最关键的部分同样是 **读取 `.flutter-plugins` 文件中的项目，然后一个一个再 ` implementation` 到主工程里完成依赖。**

![](https://user-gold-cdn.xitu.io/2019/5/24/16ae9bb09e233e7c?imageView2/0/w/1280/h/960/ignore-error/1)

**自此所有原生代码的 ` Flutter` 插件，都被作为本地 ` Module Project` 的形式引入主工程了** ，最后脚本会自动生成一个 **` GeneratedPluginRegistrant.java`** 文件，实现原生代码的引用注册， 而这个过程对你完全是无感的。

**说了那么多就是为了说明，既然插件是被当作本地 ` Module Project` 的形式引入，那么这时候按照原来直接打包 aar 是会有问题的：**

> 
> 
> 
> `Android` 默认 `gradle` 脚本打包时，对于 `project` 和远程依赖只会打包引用而不会打包源码和资源。
> 
> 

所以这时候就需要 ` fat-aar` 的加持了，关于 ` fat-aar` 的详细概念可见 ： [《从Android到React Native开发（四、打包流程解析和发布为Maven库）》]( https://juejin.im/post/5b2116466fb9a01e3128359f ) ，这里可以简单理解为， **这是一个支持将引用代码和资源到合并到一个 aar 的插件。**

如下代码所示，我们在原本的组件化脚本上， **通过增加 ` apply plugin: 'com.kezong.fat-aar'` 引入插件，然后参考 ` Flutter` 脚本对 `.flutter-plugins` 文件中的项目进行 ` embed` 依赖引用即可** ，这时候再打包出的 ` aar` 文件即为完整 ` Flutter` 项目代码。

![](https://user-gold-cdn.xitu.io/2019/5/24/16ae9bb09e6c305f?imageView2/0/w/1280/h/960/ignore-error/1)

> 
> 
> 
> 完整版可见 [flutter_app_lib](
> https://link.juejin.im?target=https%3A%2F%2Fgithub.com%2FCarGuo%2Fflutter_app_lib
> ) 。
> 
> 

## 四、堆栈 ##

最后需要说的问题就是堆栈了。

如果说混合开发中最难处理的是什么，那一定是各平台之间的堆栈管理， **一般情况下我们都会避免混合堆栈的相互调用** ，但是面对不得不如此为之的情况下，闲鱼给出了他们的答案： **` fluttet_boost` 。**

我们知道 ` Flutter` 整个项目都是绘制在一个 ` Surface` 画布上，而 ` fluttet_boost` 将堆栈统一到了原生层，通过一个单例的 ` flutter engine` 进行绘制。

每个 ` FlutterFragment` 和 ` FlutterActivity` 都是一个 ` Surface` 承载容器，切换页面时就是切换 ` Surface` 渲染显示，而对于不渲染的页面通过 ` Surface` 截图缓存画面显示。

![](https://user-gold-cdn.xitu.io/2019/5/24/16ae9bb0a080a9cb?imageView2/0/w/1280/h/960/ignore-error/1)

**这样整个 ` Flutter` 的路由就被映射到原生堆栈中，统一由原生页面堆栈管理， ` Flutter` 内每 ` push` 一个页面就是打开一个 ` Activity` 。**

> 
> 
> 
> ` flutter_boost` 截止到我测试的时间 2019-05-16, 只支持 1.2之前的版本。 ` flutter_boost` 的整体流程相对复杂，同时对于
> ` Dialog` 的支持并不好，且业务跳转深度太深时会出现黑屏问题。
> 
> 

![](https://user-gold-cdn.xitu.io/2019/6/3/16b1da061958fcae?imageView2/0/w/1280/h/960/ignore-error/1)

> 
> 
> 
> 自此，第十四篇终于结束了！(///▽///)
> 
> 

### 资源推荐 ###

* Github ： [github.com/CarGuo]( https://link.juejin.im?target=https%3A%2F%2Fgithub.com%2FCarGuo )
* 本文Demo ： [github.com/CarGuo/flut…]( https://link.juejin.im?target=https%3A%2F%2Fgithub.com%2FCarGuo%2Fflutter_app_lib )
* 推荐完整代码 ： [github.com/CarGuo/GSYG…]( https://link.juejin.im?target=https%3A%2F%2Fgithub.com%2FCarGuo%2FGSYGithubAppFlutter )

##### 完整开源项目推荐： #####

* [GSYGithubApp Flutter]( https://link.juejin.im?target=https%3A%2F%2Fgithub.com%2FCarGuo%2FGSYGithubAppFlutter )
* [GSYGithubApp React Native]( https://link.juejin.im?target=https%3A%2F%2Fgithub.com%2FCarGuo%2FGSYGithubApp )
* [GSYGithubAppWeex]( https://link.juejin.im?target=https%3A%2F%2Fgithub.com%2FCarGuo%2FGSYGithubAppWeex )

##### 文章 #####

[《Flutter完整开发实战详解(一、Dart语言和Flutter基础)》]( https://juejin.im/post/5b631d326fb9a04fce524db2 )

[《Flutter完整开发实战详解(二、 快速开发实战篇)》]( https://juejin.im/post/5b685a2a5188251ac22b71c0 )

[《Flutter完整开发实战详解(三、 打包与填坑篇)》]( https://juejin.im/post/5b6fd4dc6fb9a0099e711162 )

[《Flutter完整开发实战详解(四、Redux、主题、国际化)》]( https://juejin.im/post/5b79767ff265da435450a873 )

[《Flutter完整开发实战详解(五、 深入探索)》]( https://juejin.im/post/5bc450dff265da0a951f032b )

[《Flutter完整开发实战详解(六、 深入Widget原理)》]( https://juejin.im/post/5c7e853151882549664b0543 )

[《Flutter完整开发实战详解(七、 深入布局原理)》]( https://juejin.im/post/5c8c6ef7e51d450ba7233f51 )

[《Flutter完整开发实战详解(八、 实用技巧与填坑)》]( https://juejin.im/post/5c9e328251882567b91e1cfb )

[《Flutter完整开发实战详解(九、 深入绘制原理)》]( https://juejin.im/post/5ca0e0aff265da309728659a )

[《Flutter完整开发实战详解(十、 深入图片加载流程)》]( https://juejin.im/post/5cb1896ce51d456e63760449 )

[《Flutter完整开发实战详解(十一、全面深入理解Stream)》]( https://juejin.im/post/5cc2acf86fb9a0321f042041 )

[《Flutter完整开发实战详解(十二、全面深入理解状态管理设计)》]( https://juejin.im/post/5cc816866fb9a03231209c7c )

[《Flutter完整开发实战详解(十三、全面深入触摸和滑动原理)》]( https://juejin.im/editor/posts/5cd54839f265da03b2044c32 )

[《跨平台项目开源项目推荐》]( https://juejin.im/post/5b6064a0f265da0f8b2fc89d )

[《移动端跨平台开发的深度解析》]( https://juejin.im/post/5b395eb96fb9a00e556123ef )

![我们还会再见吗？](https://user-gold-cdn.xitu.io/2019/5/10/16aa1221cbe49af2?imageView2/0/w/1280/h/960/ignore-error/1)