# Go的包管理工具（四）：Go Module Proxy #

在前面的文章，我们介绍了 ` Go Modules` ( https://link.juejin.im?target=http%3A%2F%2Fblueskykong.com%2F2019%2F03%2F01%2Fgo-dep-3%2F ) 。Go module支持了Versioned Go，并初步解决了包依赖管理的问题。

新的工作模式也带来了一些问题，在大陆地区我们无法直接通过 ` go get` 命令获取到一些第三方包，最常见的就是 ` golang.org/x` 下面的各种优秀的包。一旦工作在模块下， ` go build` 将不再关心 GOPATH 或是 vendor 下的包，而是到 ` GOPATH/pkg/mod` 查询是否有cache，如果没有，则会去下载某个版本的 module，而对于某些包的 module，在大陆地区往往会失败。本文将重点介绍 go module 的 proxy 配置实现，包括如下两种的代理配置：

* GOPROXY
* Athens

## GOPROXY ##

goproxy 是一个开源项目，当用户请求一个依赖库时，如果它发现本地没有这份代码就会自动请求源，然后缓存到本地，用户就可以从 goproxy.io 请求到数据。当然，这些都是在一个请求中完成的。goproxy.io 只支持 go module 模式。当用户执行 go get 命令时，会去检查 ` $GOPROXY//@v/list` 这个文件中是否有用户想要获取的版本，如果有，就依次获取 ` $GOPROXY//@v/.info` 、 ` $GOPROXY//@v/.mod` 、 ` $GOPROXY//@v/.zip` 等文件，如果没有就直接从源码库中去下载。

通过命令：

` export GOPROXY=https://goproxy.io 复制代码`

设置了这个环境变量，一旦设置生效，后续 go 命令会通过 ` go module download protocol` 与 ` proxy` 交互下载特定版本的 ` module` 。当然，我们还可以置空 ` GOPROXY` 变量，来关闭 ` GOPROXY` 代理。

详见： [github.com/goproxyio/g…]( https://link.juejin.im?target=https%3A%2F%2Fgithub.com%2Fgoproxyio%2Fgoproxy )

## Athens ##

![](https://user-gold-cdn.xitu.io/2019/3/18/169910a7a5c77085?imageView2/0/w/1280/h/960/ignore-error/1) Athens 是一个建立在 vgo（或者是1.11 +）之上的项目，试图让依赖关系更接近你，即使在 VCS 关闭时你也可以依赖可重复的构建。

依赖关系是来自 Github 的不可变的代码块和相关的元数据。 他们存储在 Athens 控制的仓库里。

您可能已经知道“不可变”意味着什么，但让我再次指出它，因为它对整个系统非常重要。当人们改变他们的包，迭代，实验或其他任何东西时，Athens 的代码都不会改变。如果软件包作者发布了一个新版本，Athens 将把它拉下来。比如某个项目依赖于包 M 版 ` v1.2.3` ，它将永远不会改变 Athens 上的包。

### 安装 ###

Athens 支持多种方式的安装，docker 容器、k8s 和二进制安装包，本文将会介绍如何通过二进制包安装。

` git clone https://github.com/gomods/athens cd athens make build-ver VERSION= "0.2.0" 复制代码` `./athens -version 复制代码`

### 获取私有仓库 ###

Athens 获取私有仓库中的 module，这也是一个企业级的需求。通常企业私有仓库都是需要身份验证的，因此我们需要在 Athens 中配置访问私有仓库的账号和凭证信息。目前 Athens 官方文档中提供了通过 `.netrc` 方式访问带有身份验证的私有仓库的功能。

通过创建 `.netrc` 文件，进行私有仓库身份验证。

` //.netrc machine github.com login MY_USERNAME password MY_PASSWORD 复制代码`

### 本地应用 ###

` export GO111MODULE=on export GOPROXY=http://127.0.0.1:3000 复制代码`

我们可以使用 Athens 提供的 ` walkthrough` 。

` git clone https://github.com/athens-artifacts/walkthrough.git 复制代码` ` $ cd../walkthrough $ go run . go: finding github.com/athens-artifacts/samplelib v1.0.0 handler: GET /github.com/athens-artifacts/samplelib/@v/v1.0.0.info [200] handler: GET /github.com/athens-artifacts/samplelib/@v/v1.0.0.mod [200] go: downloading github.com/athens-artifacts/samplelib v1.0.0 handler: GET /github.com/athens-artifacts/samplelib/@v/v1.0.0.zip [200] The 🦁 says rawr! 复制代码`

` go run .` 的输出包括尝试查找 ` github.com/athens-artifacts/samplelib` 依赖项。由于代理是在后台运行的，因此还可以看到来自 Athens 的输出，表明它正在处理对依赖项的请求。

### global public proxy ###

Athens 还提供了一个试验性的 proxy： ` athens.azurefd.net` 供全球gopher 使用。导入环境变量：

` export GOPROXY= "https://athens.azurefd.net" 复制代码`

当然虽然可以使用这些公有的代理，但是包的源并不是很全，最稳妥的方法还是自建 Athens 服务。官方issue中所说：

> 
> 
> 
> ` athens.azurefd.net` 处于 alpha 的版本，目前基础设施还没有很好地维护，所以看到超时并不感到惊讶。自建 ` Athens`
> 或使用 GoCenter（目前唯一的托管Go模块存储库）。
> 
> 

#### 推荐阅读 ####

[Go的包管理工具]( https://link.juejin.im?target=http%3A%2F%2Fblueskykong.com%2Ftags%2F%25E5%258C%2585%25E7%25AE%25A1%25E7%2590%2586%2F )

#### 订阅最新文章，欢迎关注我的公众号 ####

![微信公众号](https://user-gold-cdn.xitu.io/2019/3/18/169910879f2ba8e3?imageView2/0/w/1280/h/960/ignore-error/1)

#### 参考 ####

* [Athens docs]( https://link.juejin.im?target=https%3A%2F%2Fdocs.gomods.io )
* [再探go modules：使用与细节]( https://link.juejin.im?target=https%3A%2F%2Fwww.cnblogs.com%2Fapocelipes%2Fp%2F10295096.html )