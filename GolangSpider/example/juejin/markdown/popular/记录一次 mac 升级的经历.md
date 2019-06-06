# 记录一次 mac 升级的经历 #

## 记录一次mac 升级的经历 ##

## 前提-顶不住了 ##

* 一直使用一台 ` 17 年 late 的 13 寸 8 g 小水管` 的 mbp，在忍受了一年多 8g 内存和低压 cpu 带来的不便以后。
* 终于在端午节前夕，小 mac 它也顶不住压力倒下了，一周之内重启黑屏数次 ![image](https://user-gold-cdn.xitu.io/2019/6/6/16b2bd6ef406c277?imageView2/0/w/1280/h/960/ignore-error/1)

![image](https://user-gold-cdn.xitu.io/2019/6/6/16b2bd6dcd4ddcec?imageView2/0/w/1280/h/960/ignore-error/1)

### 新电脑到手 ###

* 然后上午找了网管老哥，下午新电脑就到手了，换成了 15 寸的 16g mbp（ps：公司效率真高）。
* google了一下迁移助手的使用方法（ [support.apple.com/zh-cn/HT204…]( https://link.juejin.im?target=https%3A%2F%2Fsupport.apple.com%2Fzh-cn%2FHT204350 )
* 其中提供了可以快速传输数据的方案（使用雷电连接两台电脑）。本想着用迁移助手--雷电线传输两个电脑的数据，结果发现电脑自带的充电线不是雷电的，居然要自己买（官网 200 一根，恩这很库克）。

### 迁移助手 ###

* 只能在回家的时候带两个电脑连接同一个 wifi 传输数据，我以为要用一晚上，结果速度还是非常快的（用的自如的 100M 的小水管）。
* 不到两个小时，新旧 mac 的配置和所有文件都在迁移助手的帮助下完全配置好了。
* 新电脑直接上手就可以开发，所有的开发环境都和旧电脑一样。只需要配置了一下 git 的账户密码以外，没有任何其他的不便。

![image](https://user-gold-cdn.xitu.io/2019/6/6/16b2bd6e7fd88ad3?imageView2/0/w/1280/h/960/ignore-error/1)

### 有一些问题出现 ###

* 桌面背景和保护程序被重置了，没有同步。
* 付费的一些 app 的需要重新输入密钥。
* ` sony wi 1000x` ，新 mac 蓝牙搜索不到，重置了 smc 以后，鼓捣了半天，发现是是附件的旧电脑影响了蓝牙的连接 。旧电脑关机后，可以正常使用。

### 整理桌面 ###

![image](https://user-gold-cdn.xitu.io/2019/6/6/16b2bd6e862baa8d?imageView2/0/w/1280/h/960/ignore-error/1)

* 整理后的桌面，相比以前的桌面，空余面积+30%，放弃了外接键盘，发现 19 年的 mac 键盘手感还可以。正常使用罗技家的max anywhere和外接屏，没有任何问题。

> 
> 
> 
> 老图对比
> 
> 

![image](https://user-gold-cdn.xitu.io/2019/6/6/16b2bd6edbec6641?imageView2/0/w/1280/h/960/ignore-error/1)

## 感受 ##

* 15 寸真爽，4k 再也不会拖屏了，node 打包速度飞起，心情好 code 速度都快了很多，bugs 也少了，我应该早点换电脑😌