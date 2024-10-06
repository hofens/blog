## WindowInsets 在View下的的分发（一）

[![](https://upload.jianshu.io/users/upload_avatars/5580200/a04b37aa-5845-48e1-ad16-830e1d61af19.jpg?imageMogr2/auto-orient/strip|imageView2/1/w/96/h/96/format/webp)](https://www.jianshu.com/u/87f5a0907e38)

0.3452017.09.02 17:59:05字数 833阅读 8,468

## 绪论

Android 4.4后，可以通过将StatusBar和NavigationBar的背景设置为透明或者通过 getWindow().getDecorView().setSystemUiVisibility 的方式，使得 contentView 可以铺满整个DecorView。然而大多数情况下，我们并不希望有实质性的内容被StatusBar或者NavigationBar给覆盖掉，那么Android是如何处理这些看似额外的空间的分发呢，这就涉及到了WindowInsets了。

[WindowInsets 在View下的的分发（二）](https://www.jianshu.com/p/986e2f8c98ff)

> **什么是WindowInsets？**

在Android源码的注释中解释为 window content 的一系列插入集合，final 型，不可修改，但后期可能继续扩展。其主要成员包括 mSystemWindowInsets， mWindowDecorInsets， mStableInsets。

-   mSystemWindowInsets

表示全窗口下，被StatusBar, NavigationBar, IME 或者其它系统窗口部分或者全部覆盖的区域。

-   mWindowDecorInsets

表示内容窗口下，被Android FrameWork提供的窗体，诸如ActionBar, TitleBar, ToolBar，部分或全部覆盖区域。

-   mStableInsets

表示全窗口下，被系统UI部分或者全部覆盖的区域。

> **如何理解WindowInsets**

以 mSystemWindowInsets 为例：

```cpp
private Rect mSystemWindowInsets; public int getSystemWindowInsetLeft() { return mSystemWindowInsets.left; } public int getSystemWindowInsetTop() { return mSystemWindowInsets.top; } public int getSystemWindowInsetRight() { return mSystemWindowInsets.right; } public int getSystemWindowInsetBottom() { return mSystemWindowInsets.bottom; }
```

这里的Rect的概念已经区别于View的Rect了，它的四个点已经不再表示围成矩形的坐标，而表示的是insets需要的左右的宽度，顶部和底部需要的高度。

> **消费Windowinsets**

以 mSystemWindowInsets 为例：

```java
private boolean mSystemWindowInsetsConsumed = false; public WindowInsets consumeSystemWindowInsets() { final WindowInsets result = new WindowInsets(this); result.mSystemWindowInsets = EMPTY_RECT; result.mSystemWindowInsetsConsumed = true; return result; } public WindowInsets consumeSystemWindowInsets(boolean left, boolean top, boolean right, boolean bottom) { if (left || top || right || bottom) { final WindowInsets result = new WindowInsets(this); result.mSystemWindowInsets = new Rect( left ? 0 : mSystemWindowInsets.left, top ? 0 : mSystemWindowInsets.top, right ? 0 : mSystemWindowInsets.right, bottom ? 0 : mSystemWindowInsets.bottom); return result; } return this; }
```

从上可以看出，mSystemWindowInsets的消费分为全部消费和部分消费，如果不存在消费，则返回对象本身，如果消费了，则返回将消费部分置为0的对象copy。

```java
//判断WindowInsets是否被消费掉 public boolean isConsumed() { return mSystemWindowInsetsConsumed && mWindowDecorInsetsConsumed && mStableInsetsConsumed; }
```

可见要消费掉WindowInsets，需要同时消耗掉 mSystemWindowInsets， mWindowDecorInsets， mStableInsets。

> **谁在消费WindowInsets**

从WindowInsets的注释中，可以发现两个函数

```php
/** * @see View.OnApplyWindowInsetsListener * @see View#onApplyWindowInsets(WindowInsets) */
```

深入View的源码可以发现，如果View设定了OnApplyWindowInsetsListener后，会采用OnApplyWindowInsetsListener的实现来处理WindowInsets，否则才会使用onApplyWindowInsets(WindowInsets)方法来处理WindowInsets，在dispatchApplyWindowInsets(WindowInsets)中进行分发处理，代码如下：

```kotlin
public WindowInsets dispatchApplyWindowInsets(WindowInsets insets) { try { mPrivateFlags3 |= PFLAG3_APPLYING_INSETS; if (mListenerInfo != null && mListenerInfo.mOnApplyWindowInsetsListener != null) { return mListenerInfo.mOnApplyWindowInsetsListener.onApplyWindowInsets(this, insets); } else { return onApplyWindowInsets(insets); } } finally { mPrivateFlags3 &= ~PFLAG3_APPLYING_INSETS; } }
```

可以发现，对于View而言，会在dispatchApplyWindowInset的过程中Apply WindowInsets。对应的ViewGroup代码如下：

```java
@Override public WindowInsets dispatchApplyWindowInsets(WindowInsets insets) { insets = super.dispatchApplyWindowInsets(insets); if (!insets.isConsumed()) { final int count = getChildCount(); for (int i = 0; i < count; i++) { insets = getChildAt(i).dispatchApplyWindowInsets(insets); if (insets.isConsumed()) { break; } } } return insets; }
```

ViewGroup自身也会Apply WindowInsets，如果该过程中没有消耗掉WindowInsets，则会继续传递给 child 处理WindwInsets，如果child中消耗了WindowInsets, 则会退出分发循环。

再看一下，View自身是如何处理WindowInsets的

```kotlin
public WindowInsets onApplyWindowInsets(WindowInsets insets) { if ((mPrivateFlags3 & PFLAG3_FITTING_SYSTEM_WINDOWS) == 0) { // We weren't called from within a direct call to fitSystemWindows, // call into it as a fallback in case we're in a class that overrides it // and has logic to perform. if (fitSystemWindows(insets.getSystemWindowInsets())) { return insets.consumeSystemWindowInsets(); } } else { // We were called from within a direct call to fitSystemWindows. if (fitSystemWindowsInt(insets.getSystemWindowInsets())) { return insets.consumeSystemWindowInsets(); } } return insets; }
```

查看fitSystemWindows(Rect insets)方法

```java
//当if条件成立时，会进入 dispatchApply逻辑，不成立则进入实际的处理逻辑fitSystemWindowsInt(insets) //如此设置的原因在于，不直接调用fitSystemWindowsInt(insets)方法，而是要经过dispatchApply后再调用 protected boolean fitSystemWindows(Rect insets) { if ((mPrivateFlags3 & PFLAG3_APPLYING_INSETS) == 0) { if (insets == null) { // Null insets by definition have already been consumed. // This call cannot apply insets since there are none to apply, // so return false. return false; } // If we're not in the process of dispatching the newer apply insets call, // that means we're not in the compatibility path. Dispatch into the newer // apply insets path and take things from there. try { mPrivateFlags3 |= PFLAG3_FITTING_SYSTEM_WINDOWS; return dispatchApplyWindowInsets(new WindowInsets(insets)).isConsumed(); } finally { mPrivateFlags3 &= ~PFLAG3_FITTING_SYSTEM_WINDOWS; } } else { // We're being called from the newer apply insets path. // Perform the standard fallback behavior. return fitSystemWindowsInt(insets); } }
```

再看fitSystemWindowsInt(Rect insets)方法

```swift
//当View设置了fitSystemWindow = true 后， 才会处理 WindowInsets，否则，直接返回false。 // private boolean fitSystemWindowsInt(Rect insets) { if ((mViewFlags & FITS_SYSTEM_WINDOWS) == FITS_SYSTEM_WINDOWS) { mUserPaddingStart = UNDEFINED_PADDING; mUserPaddingEnd = UNDEFINED_PADDING; Rect localInsets = sThreadLocal.get(); if (localInsets == null) { localInsets = new Rect(); sThreadLocal.set(localInsets); } boolean res = computeFitSystemWindows(insets, localInsets); mUserPaddingLeftInitial = localInsets.left; mUserPaddingRightInitial = localInsets.right; internalSetPadding(localInsets.left, localInsets.top, localInsets.right, localInsets.bottom); return res; } return false; } //计算是否消费WindowInsets protected boolean computeFitSystemWindows(Rect inoutInsets, Rect outLocalInsets) { if ((mViewFlags & OPTIONAL_FITS_SYSTEM_WINDOWS) == 0 || mAttachInfo == null || ((mAttachInfo.mSystemUiVisibility & SYSTEM_UI_LAYOUT_FLAGS) == 0 && !mAttachInfo.mOverscanRequested)) { outLocalInsets.set(inoutInsets); inoutInsets.set(0, 0, 0, 0); return true; } else { // The application wants to take care of fitting system window for // the content... however we still need to take care of any overscan here. final Rect overscan = mAttachInfo.mOverscanInsets; outLocalInsets.set(overscan); inoutInsets.left -= overscan.left; inoutInsets.top -= overscan.top; inoutInsets.right -= overscan.right; inoutInsets.bottom -= overscan.bottom; return false; } } //重新调整View的padding值 protected void internalSetPadding(int left, int top, int right, int bottom) { //省略部分非关键代码 ... if (mPaddingLeft != left) { changed = true; mPaddingLeft = left; } if (mPaddingTop != top) { changed = true; mPaddingTop = top; } if (mPaddingRight != right) { changed = true; mPaddingRight = right; } if (mPaddingBottom != bottom) { changed = true; mPaddingBottom = bottom; } if (changed) { requestLayout(); invalidateOutline(); } }
```

以上便是WindowInsets在View和ViewGroup中处理WindowInsets的过程，ViewGroup与View之间处理WindowInsets的区别在于dispatchApplyWindowInsets(...)函数，网上大数文章将其当成一个分发逻辑看待，其实更准确的说法应该是一个消费或者分发逻辑。

> **总结：**

WindowInsets是一个描述了屏幕上的各个插入空间的一个类，其在后期中可以扩展，WindowInsets在消耗后将不再继续传递。对于普通的View而言，要消耗WindowInsets必须先设置View的 fitsSystemWindows 的属性为true。这也是为什么对普通View层级设置fitsSystemWindows属性为true却只有一个顶层的生效而已。单对于一些特殊的View而言，则是另外一番情况了，具体将在下篇中说明。

最后编辑于

：2017.12.10 05:41:47

更多精彩内容，就在简书APP

![](https://upload.jianshu.io/images/js-qrc.png)

"小礼物走一走，来简书关注我"

还没有人赞赏，支持一下

[![  ](https://upload.jianshu.io/users/upload_avatars/5580200/a04b37aa-5845-48e1-ad16-830e1d61af19.jpg?imageMogr2/auto-orient/strip|imageView2/1/w/100/h/100/format/webp)](https://www.jianshu.com/u/87f5a0907e38)

-   序言：七十年代末，一起剥皮案震惊了整个滨河市，随后出现的几起案子，更是在滨河造成了极大的恐慌，老刑警刘岩，带你破解...
    
    [![](https://upload.jianshu.io/users/upload_avatars/15878160/783c64db-45e5-48d7-82e4-95736f50533e.jpg?imageMogr2/auto-orient/strip|imageView2/1/w/48/h/48/format/webp)沈念sama](https://www.jianshu.com/u/dcd395522934)阅读 145,047评论 1赞 306
    
-   序言：滨河连续发生了三起死亡事件，死亡现场离奇诡异，居然都是意外死亡，警方通过查阅死者的电脑和手机，发现死者居然都...
    
-   文/潘晓璐 我一进店门，熙熙楼的掌柜王于贵愁眉苦脸地迎上来，“玉大人，你说我怎么就摊上这事。” “怎么了？”我有些...
    
-   文/不坏的土叔 我叫张陵，是天一观的道长。 经常有香客问我，道长，这世上最难降的妖魔是什么？ 我笑而不...
    
-   正文 为了忘掉前任，我火速办了婚礼，结果婚礼上，老公的妹妹穿的比我还像新娘。我一直安慰自己，他们只是感情好，可当我...
    
    [![](https://upload.jianshu.io/users/upload_avatars/4790772/388e473c-fe2f-40e0-9301-e357ae8f1b41.jpeg?imageMogr2/auto-orient/strip|imageView2/1/w/48/h/48/format/webp)茶点故事](https://www.jianshu.com/u/0f438ff0a55f)阅读 49,312评论 1赞 262
    
-   文/花漫 我一把揭开白布。 她就那样静静地躺着，像睡着了一般。 火红的嫁衣衬着肌肤如雪。 梳的纹丝不乱的头发上，一...
    
-   那天，我揣着相机与录音，去河边找鬼。 笑死，一个胖子当着我的面吹牛，可吹牛的内容都是我干的。 我是一名探鬼主播，决...
    
-   文/苍兰香墨 我猛地睁开眼，长吁一口气：“原来是场噩梦啊……” “哼！你这毒妇竟也来了？” 一声冷哼从身侧响起，我...
    
-   序言：老挝万荣一对情侣失踪，失踪者是张志新（化名）和其女友刘颖，没想到半个月后，有当地人在树林里发现了一具尸体，经...
    
-   正文 独居荒郊野岭守林人离奇死亡，尸身上长有42处带血的脓包…… 初始之章·张勋 以下内容为张勋视角 年9月15日...
    
    [![](https://upload.jianshu.io/users/upload_avatars/4790772/388e473c-fe2f-40e0-9301-e357ae8f1b41.jpeg?imageMogr2/auto-orient/strip|imageView2/1/w/48/h/48/format/webp)茶点故事](https://www.jianshu.com/u/0f438ff0a55f)阅读 29,435评论 2赞 218
    
-   正文 我和宋清朗相恋三年，在试婚纱的时候发现自己被绿了。 大学时的朋友给我发了我未婚夫和他白月光在一起吃饭的照片。...
    
    [![](https://upload.jianshu.io/users/upload_avatars/4790772/388e473c-fe2f-40e0-9301-e357ae8f1b41.jpeg?imageMogr2/auto-orient/strip|imageView2/1/w/48/h/48/format/webp)茶点故事](https://www.jianshu.com/u/0f438ff0a55f)阅读 30,790评论 1赞 232
    
-   序言：一个原本活蹦乱跳的男人离奇死亡，死状恐怖，灵堂内的尸体忽然破棺而出，到底是诈尸还是另有隐情，我是刑警宁泽，带...
    
-   正文 年R本政府宣布，位于F岛的核电站，受9级特大地震影响，放射性物质发生泄漏。R本人自食恶果不足惜，却给世界环境...
    
    [![](https://upload.jianshu.io/users/upload_avatars/4790772/388e473c-fe2f-40e0-9301-e357ae8f1b41.jpeg?imageMogr2/auto-orient/strip|imageView2/1/w/48/h/48/format/webp)茶点故事](https://www.jianshu.com/u/0f438ff0a55f)阅读 31,734评论 3赞 214
    
-   文/蒙蒙 一、第九天 我趴在偏房一处隐蔽的房顶上张望。 院中可真热闹，春花似锦、人声如沸。这庄子的主人今日做“春日...
    
-   文/苍兰香墨 我抬头看了看天上的太阳。三九已至，却和暖如春，着一层夹袄步出监牢的瞬间，已是汗流浃背。 一阵脚步声响...
    
-   我被黑心中介骗来泰国打工， 没想到刚下飞机就差点儿被人妖公主榨干…… 1. 我叫王不留，地道东北人。 一个月前我还...
    
-   正文 我出身青楼，却偏偏与公主长得像，于是被迫代替她去往敌国和亲。 传闻我的和亲对象是个残疾皇子，可洞房花烛夜当晚...
    
    [![](https://upload.jianshu.io/users/upload_avatars/4790772/388e473c-fe2f-40e0-9301-e357ae8f1b41.jpeg?imageMogr2/auto-orient/strip|imageView2/1/w/48/h/48/format/webp)茶点故事](https://www.jianshu.com/u/0f438ff0a55f)阅读 33,875评论 2赞 238
    

### 被以下专题收入，发现更多相似内容

### 推荐阅读[更多精彩内容](https://www.jianshu.com/)

-   Android 自定义View的各种姿势1 Activity的显示之ViewRootImpl详解 Activity...
    
-   1\. 概述   作为Android开发中最常见的一个控件,个人觉得有必要谈谈了。我们刚开始接触Android的时候...
    
    [![](https://upload.jianshu.io/users/upload_avatars/1869441/30ef0ee0-eb94-433f-8ffb-28934103f2d5.jpg?imageMogr2/auto-orient/strip|imageView2/1/w/48/h/48/format/webp)忆念成风](https://www.jianshu.com/u/fa19fba4ef18)阅读 3,086评论 2赞 16
    
-   原文链接 在 Android Translucent Status Bar 系列中，我基于“给哪个View设置fi...
    
-   Android 事件分发和滑动冲突都是开发中经常遇到的难点问题，遇到问题时可能会通过 Google 或者 Stac...
    
    [![](https://cdn2.jianshu.io/default_avatar/10.jpg)任教主来也](https://www.jianshu.com/u/e027c09aa4dd)阅读 2,632评论 0赞 24
    
-   大龄剩女的四面楚歌 文/胡大小姐 前段时间，写了篇《大龄剩女和离婚女人，你更能接受哪个标签？》，后台的评论异常火爆...