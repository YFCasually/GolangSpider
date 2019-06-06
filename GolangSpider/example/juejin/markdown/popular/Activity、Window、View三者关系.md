# Activity、Window、View三者关系 #

#### 目录介绍 ####

* 01.Window，View，子Window
* 02.什么是Activity
* 03.什么是Window
* 04.什么是DecorView
* 05.什么是View
* 06.关系结构图
* 07.Window创建过程
* 08.创建机制分析

* 8.1 Activity实例的创建
* 8.2 Activity中Window的创建
* 8.3 DecorView的创建

### 弹窗系列博客 ###

* [01.Activity、Window、View三者关系]( https://link.juejin.im?target=https%3A%2F%2Fgithub.com%2Fyangchong211%2FYCBlogs%2Fblob%2Fmaster%2Fandroid%2FWindow%2F01.Activity%25E3%2580%2581Window%25E3%2580%2581View%25E4%25B8%2589%25E8%2580%2585%25E5%2585%25B3%25E7%25B3%25BB.md )

* 深入分析Activity、Window、View三者之间的关系

* [02.Toast源码深度分析]( https://link.juejin.im?target=https%3A%2F%2Fgithub.com%2Fyangchong211%2FYCBlogs%2Fblob%2Fmaster%2Fandroid%2FWindow%2F02.Toast%25E6%25BA%2590%25E7%25A0%2581%25E6%25B7%25B1%25E5%25BA%25A6%25E5%2588%2586%25E6%259E%2590.md )

* 最简单的创建，简单改造避免重复创建，show()方法源码分析，scheduleTimeoutLocked吐司如何自动销毁的，TN类中的消息机制是如何执行的，普通应用的Toast显示数量是有限制的，用代码解释为何Activity销毁后Toast仍会显示，Toast偶尔报错Unable to add window是如何产生的，Toast运行在子线程问题，Toast如何添加系统窗口的权限等等

* [03.DialogFragment源码分析]( https://link.juejin.im?target=https%3A%2F%2Fgithub.com%2Fyangchong211%2FYCBlogs%2Fblob%2Fmaster%2Fandroid%2FWindow%2F03.DialogFragment%25E6%25BA%2590%25E7%25A0%2581%25E5%2588%2586%25E6%259E%2590.md )

* 最简单的使用方法，onCreate(@Nullable Bundle savedInstanceState)源码分析，重点分析弹窗展示和销毁源码，使用中show()方法遇到的IllegalStateException分析

* [04.Dialog源码分析]( https://link.juejin.im?target=https%3A%2F%2Fgithub.com%2Fyangchong211%2FYCBlogs%2Fblob%2Fmaster%2Fandroid%2FWindow%2F04.Dialog%25E6%25BA%2590%25E7%25A0%2581%25E5%2588%2586%25E6%259E%2590.md )

* AlertDialog源码分析，通过AlertDialog.Builder对象设置属性，Dialog生命周期，Dialog中show方法展示弹窗分析，Dialog的dismiss销毁弹窗，Dialog弹窗问题分析等等

* [05.PopupWindow源码分析]( https://link.juejin.im?target=https%3A%2F%2Fgithub.com%2Fyangchong211%2FYCBlogs%2Fblob%2Fmaster%2Fandroid%2FWindow%2F05.PopupWindow%25E6%25BA%2590%25E7%25A0%2581%25E5%2588%2586%25E6%259E%2590.md )

* 显示PopupWindow，注意问题宽和高属性，showAsDropDown()源码，dismiss()源码分析，PopupWindow和Dialog有什么区别？为何弹窗点击一下就dismiss呢？

* [06.Snackbar源码分析]( https://link.juejin.im?target=https%3A%2F%2Fgithub.com%2Fyangchong211%2FYCBlogs%2Fblob%2Fmaster%2Fandroid%2FWindow%2F06.Snackbar%25E6%25BA%2590%25E7%25A0%2581%25E5%2588%2586%25E6%259E%2590.md )

* 最简单的创建，Snackbar的make方法源码分析，Snackbar的show显示与点击消失源码分析，显示和隐藏中动画源码分析，Snackbar的设计思路，为什么Snackbar总是显示在最下面

* [07.弹窗常见问题]( https://link.juejin.im?target=https%3A%2F%2Fgithub.com%2Fyangchong211%2FYCBlogs%2Fblob%2Fmaster%2Fandroid%2FWindow%2F07.%25E5%25BC%25B9%25E7%25AA%2597%25E5%25B8%25B8%25E8%25A7%2581%25E9%2597%25AE%25E9%25A2%2598.md )

* DialogFragment使用中show()方法遇到的IllegalStateException,什么常见产生的？Toast偶尔报错Unable to add window，Toast运行在子线程导致崩溃如何解决？

* [09.onAttachedToWindow和onDetachedFromWindow]( https://link.juejin.im?target=https%3A%2F%2Fgithub.com%2Fyangchong211%2FYCBlogs%2Fblob%2Fmaster%2Fandroid%2FWindow%2F09.onAttachedToWindow%25E5%2592%258ConDetachedFromWindow.md )

* onAttachedToWindow的调用过程，onDetachedFromWindow可以做什么？

* [10.DecorView介绍]( https://link.juejin.im?target=https%3A%2F%2Fgithub.com%2Fyangchong211%2FYCBlogs%2Fblob%2Fmaster%2Fandroid%2FWindow%2F10.DecorView%25E4%25BB%258B%25E7%25BB%258D.md )

* 什么是DecorView，DecorView的创建，DecorView的显示，深度解析

### 01.Window，View，子Window ###

* 弹窗有哪些类型

* 使用子窗口：在 Android 进程内，我们可以直接使用类型为子窗口类型的窗口。在 Android 代码中的直接应用是 PopupWindow 或者是 Dialog 。这当然可以，不过这种窗口依赖于它的宿主窗口，它可用的条件是你的宿主窗口可用
* 采用View系统：使用 View 系统去模拟一个窗口行为，且能更加快速的实现动画效果，比如SnackBar 就是采用这套方案
* 使用系统窗口：比如吐司Toast

### 02.什么是Activity ###

* Activity并不负责视图控制，它只是控制生命周期和处理事件。真正控制视图的是Window。一个Activity包含了一个Window，Window才是真正代表一个窗口。
* **Activity就像一个控制器，统筹视图的添加与显示，以及通过其他回调方法，来与Window、以及View进行交互。**

### 03.什么是Window ###

* Window是什么？

* 表示一个窗口的概念，是所有View的直接管理者，任何视图都通过Window呈现(点击事件由Window->DecorView->View; Activity的setContentView底层通过Window完成)
* Window是一个抽象类，具体实现是PhoneWindow。PhoneWindow中有个内部类DecorView，通过创建DecorView来加载Activity中设置的布局 ` R.layout.activity_main` 。
* 创建Window需要通过WindowManager创建，通过WindowManager将DecorView加载其中，并将DecorView交给ViewRoot，进行视图绘制以及其他交互。
* WindowManager是外界访问Window的入口
* Window具体实现位于WindowManagerService中
* WindowManager和WindowManagerService的交互是通过IPC完成

* 如何通过WindowManager添加Window(代码实现)？

* 如下所示 ` //1. 控件 Button button = new Button(this); button.setText( "Window Button" ); //2. 布局参数 WindowManager.LayoutParams layoutParams = new WindowManager.LayoutParams(WindowManager.LayoutParams.WRAP_CONTENT, WindowManager.LayoutParams.WRAP_CONTENT, 0, 0, PixelFormat.TRANSPARENT); layoutParams.flags = WindowManager.LayoutParams.FLAG_NOT_TOUCH_MODAL | WindowManager.LayoutParams.FLAG_NOT_FOCUSABLE | WindowManager.LayoutParams.FLAG_SHOW_WHEN_LOCKED; layoutParams.gravity = Gravity.LEFT | Gravity.TOP; layoutParams.x = 100; layoutParams.y = 300; // 必须要有 type 不然会异常: the specified window type 0 is not valid layoutParams.type = WindowManager.LayoutParams.TYPE_SYSTEM_ERROR; //3. 获取WindowManager并添加控件到Window中 WindowManager windowManager = getWindowManager(); windowManager.addView(button, layoutParams); 复制代码`

* WindowManager的主要功能是什么？

* 添加、更新、删除View ` public interface ViewManager{ public void addView(View view, ViewGroup.LayoutParams params); //添加View public void updateViewLayout(View view, ViewGroup.LayoutParams params); //更新View public void removeView(View view); //删除View } 复制代码`

### 04.什么是DecorView ###

* DecorView是FrameLayout的子类，它可以被认为是Android视图树的根节点视图。

* DecorView作为顶级View，一般情况下它内部包含一个竖直方向的LinearLayout， **在这个LinearLayout里面有上下三个部分，上面是个ViewStub，延迟加载的视图（应该是设置ActionBar，根据Theme设置），中间的是标题栏(根据Theme设置，有的布局没有)，下面的是内容栏。**
* 具体情况和Android版本及主体有关，以其中一个布局为例，如下所示：

` <LinearLayout xmlns:android= "http://schemas.android.com/apk/res/android" android:fitsSystemWindows= "true" android:orientation= "vertical" > <!-- Popout bar for action modes --> <ViewStub android:id= "@+id/action_mode_bar_stub" android:layout_width= "match_parent" android:layout_height= "wrap_content" android:inflatedId= "@+id/action_mode_bar" android:layout= "@layout/action_mode_bar" android:theme= "?attr/actionBarTheme" /> <FrameLayout style= "?android:attr/windowTitleBackgroundStyle" android:layout_width= "match_parent" android:layout_height= "?android:attr/windowTitleSize" > <TextView android:id= "@android:id/title" style= "?android:attr/windowTitleStyle" android:layout_width= "match_parent" android:layout_height= "match_parent" android:background= "@null" android:fadingEdge= "horizontal" android:gravity= "center_vertical" /> </FrameLayout> <FrameLayout android:id= "@android:id/content" android:layout_width= "match_parent" android:layout_height= "0dip" android:layout_weight= "1" android:foreground= "?android:attr/windowContentOverlay" android:foregroundGravity= "fill_horizontal|top" /> </LinearLayout> 复制代码`
* 在Activity中通过setContentView所设置的布局文件其实就是被加到内容栏之中的，成为其唯一子View，就是上面的id为content的FrameLayout中，在代码中可以通过content来得到对应加载的布局。 ` ViewGroup content = (ViewGroup)findViewById(android.R.id.content); ViewGroup rootView = (ViewGroup) content.getChildAt(0); 复制代码`

### 06.关系结构图 ###

* Activity 与 PhoneWindow 与 DecorView 关系图

* ![image](https://user-gold-cdn.xitu.io/2019/5/29/16b0201fe3dc60ec?imageView2/0/w/1280/h/960/ignore-error/1)

### 07.Window创建过程 ###

* App点击桌面图片启动过程

* ![image](https://user-gold-cdn.xitu.io/2019/5/29/16b0201fe3eabd7e?imageView2/0/w/1280/h/960/ignore-error/1)

* window启动流程

* ![image](https://user-gold-cdn.xitu.io/2019/5/29/16b0201fe3c92a7a?imageView2/0/w/1280/h/960/ignore-error/1)

* Activity 与 PhoneWindow 与 DecorView 之间什么关系？

* 一个 Activity 对应一个 Window 也就是 PhoneWindow，一个 PhoneWindow 持有一个 DecorView 的实例，DecorView 本身是一个 FrameLayout。

### 08.创建机制分析 ###

#### 8.1 Activity实例的创建 ####

* ActivityThread中执行performLaunchActivity，从而生成了Activity的实例。源码如下所示，ActivityThread类中源码 ` private Activity performLaunchActivity(ActivityClientRecord r, Intent customIntent) { ... Activity activity = null; try { java.lang.ClassLoader cl = r.packageInfo.getClassLoader(); activity = mInstrumentation.newActivity( cl, component.getClassName(), r.intent); ... } catch (Exception e) { ... } try { ... if (activity != null) { ... activity.attach(appContext, this, getInstrumentation(), r.token, r.ident, app, r.intent, r.activityInfo, title, r.parent, r.embeddedID, r.lastNonConfigurationInstances, config, r.referrer, r.voiceInteractor); ... } ... } catch (SuperNotCalledException e) { throw e; } catch (Exception e) { ... } return activity; } 复制代码`

#### 8.2 Activity中Window的创建 ####

* 从上面的performLaunchActivity可以看出，在创建Activity实例的同时，会调用Activity的内部方法attach
* 在attach该方法中完成window的初始化。源码如下所示，Activity类中源码 ` final void attach(Context context, ActivityThread aThread, Instrumentation instr, IBinder token, int ident, Application application, Intent intent, ActivityInfo info, CharSequence title, Activity parent, String id, NonConfigurationInstances lastNonConfigurationInstances, Configuration config, String referrer, IVoiceInteractor voiceInteractor, Window window, ActivityConfigCallback activityConfigCallback) { mWindow = new PhoneWindow(this, window, activityConfigCallback); mWindow.setWindowControllerCallback(this); mWindow.setCallback(this); mWindow.setOnWindowDismissedCallback(this); mWindow.getLayoutInflater().setPrivateFactory(this); if (info.softInputMode != WindowManager.LayoutParams.SOFT_INPUT_STATE_UNSPECIFIED) { mWindow.setSoftInputMode(info.softInputMode); } if (info.uiOptions != 0) { mWindow.setUiOptions(info.uiOptions); } } 复制代码`

#### 8.3 DecorView的创建 ####

* 用户执行Activity的setContentView方法，内部是调用PhoneWindow的setContentView方法，在PhoneWindow中完成DecorView的创建。流程

* 1.Activity中的setContentView
* 2.PhoneWindow中的setContentView
* 3.PhoneWindow中的installDecor

` public void set ContentView(@LayoutRes int layoutResID) { getWindow().setContentView(layoutResID); initWindowDecorActionBar(); } @Override public void set ContentView(int layoutResID) { ... if (mContentParent == null) { installDecor(); } else if (!hasFeature(FEATURE_CONTENT_TRANSITIONS)) { mContentParent.removeAllViews(); } ... } private void installDecor () { if (mDecor == null) { mDecor = generateDecor(); mDecor.setDescendantFocusability(ViewGroup.FOCUS_AFTER_DESCENDANTS); mDecor.setIsRootNamespace( true ); if (!mInvalidatePanelMenuPosted && mInvalidatePanelMenuFeatures != 0) { mDecor.postOnAnimation(mInvalidatePanelMenuRunnable); } } ... } 复制代码`

### 关于其他内容介绍 ###

#### 01.关于博客汇总链接 ####

* 1. [技术博客汇总]( https://link.juejin.im?target=https%3A%2F%2Fwww.jianshu.com%2Fp%2F614cb839182c )
* 2. [开源项目汇总]( https://link.juejin.im?target=https%3A%2F%2Fblog.csdn.net%2Fm0_37700275%2Farticle%2Fdetails%2F80863574 )
* 3. [生活博客汇总]( https://link.juejin.im?target=https%3A%2F%2Fblog.csdn.net%2Fm0_37700275%2Farticle%2Fdetails%2F79832978 )
* 4. [喜马拉雅音频汇总]( https://link.juejin.im?target=https%3A%2F%2Fwww.jianshu.com%2Fp%2Ff665de16d1eb )
* 5. [其他汇总]( https://link.juejin.im?target=https%3A%2F%2Fwww.jianshu.com%2Fp%2F53017c3fc75d )

#### 02.关于我的博客 ####

* 我的个人站点：www.yczbj.org， www.ycbjie.cn
* github： [github.com/yangchong21…]( https://link.juejin.im?target=https%3A%2F%2Fgithub.com%2Fyangchong211 )
* 知乎： [www.zhihu.com/people/yczb…]( https://link.juejin.im?target=https%3A%2F%2Fwww.zhihu.com%2Fpeople%2Fyczbj%2Factivities )
* 简书： [www.jianshu.com/u/b7b2c6ed9…]( https://link.juejin.im?target=http%3A%2F%2Fwww.jianshu.com%2Fu%2Fb7b2c6ed9284 )
* csdn： [my.csdn.net/m0_37700275]( https://link.juejin.im?target=http%3A%2F%2Fmy.csdn.net%2Fm0_37700275 )
* 喜马拉雅听书： [www.ximalaya.com/zhubo/71989…]( https://link.juejin.im?target=http%3A%2F%2Fwww.ximalaya.com%2Fzhubo%2F71989305%2F )
* 开源中国： [my.oschina.net/zbj1618/blo…]( https://link.juejin.im?target=https%3A%2F%2Fmy.oschina.net%2Fzbj1618%2Fblog )
* 泡在网上的日子： [www.jcodecraeer.com/member/cont…]( https://link.juejin.im?target=http%3A%2F%2Fwww.jcodecraeer.com%2Fmember%2Fcontent_list.php%3Fchannelid%3D1 )
* 邮箱：yangchong211@163.com
* 阿里云博客： [yq.aliyun.com/users/artic…]( https://link.juejin.im?target=https%3A%2F%2Fyq.aliyun.com%2Fusers%2Farticle%3Fspm%3D5176.100- ) 239.headeruserinfo.3.dT4bcV
* segmentfault头条： [segmentfault.com/u/xiangjian…]( https://link.juejin.im?target=https%3A%2F%2Fsegmentfault.com%2Fu%2Fxiangjianyu%2Farticles )
* 掘金： [juejin.im/user/593943…]( https://juejin.im/user/5939433efe88c2006afa0c6e )

### GitHub链接： [github.com/yangchong21…]( https://link.juejin.im?target=https%3A%2F%2Fgithub.com%2Fyangchong211 ) ###