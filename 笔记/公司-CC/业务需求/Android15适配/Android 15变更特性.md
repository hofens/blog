
每次 Android 版本升级都会有几个部分内容，如 
1、必须处理：影响 Android 15上所有应用的行为变更
2、需要留意：影响以 Android 15为目标平台应用的行为变更（即 targetSdkVersion>=Android 15 的 sdkVersion）
3、可以了解：新功能和 API


## 版本特性介绍

[developer.android.com/about/versi…](https://link.juejin.cn?target=https%3A%2F%2Fdeveloper.android.com%2Fabout%2Fversions%2F15%3Fhl%3Dzh-cn "https://developer.android.com/about/versions/15?hl=zh-cn")

## 所有功能和 API 概览

[developer.android.com/about/versi…](https://link.juejin.cn?target=https%3A%2F%2Fdeveloper.android.com%2Fabout%2Fversions%2F15%2Ffeatures%3Fhl%3Dzh-cn%23language-switching "https://developer.android.com/about/versions/15/features?hl=zh-cn#language-switching")

## 发布时间
![|800](assets/Pasted%20image%2020240826162729.png)


## 适配指南

Google开发平台：[developer.android.com/about/versi…](https://link.juejin.cn?target=https%3A%2F%2Fdeveloper.android.com%2Fabout%2Fversions%2F15%2Fmigration%3Fhl%3Dzh-cn "https://developer.android.com/about/versions/15/migration?hl=zh-cn")
小米开放平台：[dev.mi.com/distribute/…](https://link.juejin.cn?target=https%3A%2F%2Fdev.mi.com%2Fdistribute%2Fdoc%2Fdetails%3FpId%3D1826 "https://dev.mi.com/distribute/doc/details?pId=1826")
OPPO开放平台：[open.oppomobile.com/new/develop…](https://link.juejin.cn?target=https%3A%2F%2Fopen.oppomobile.com%2Fnew%2FdevelopmentDoc%2Finfo%3Fid%3D13047 "https://open.oppomobile.com/new/developmentDoc/info?id=13047")
其它Android版本适配指南：[github.com/getActivity…](https://link.juejin.cn?target=https%3A%2F%2Fgithub.com%2FgetActivity%2FAndroidVersionAdapter "https://github.com/getActivity/AndroidVersionAdapter")

## 所有应用都会受影响，所以也是CC 需要留意的变更

### [音频播放](https://dev.mi.com/distribute/doc/details?pId=1826#_18)
Android15之前，如果应用请求使用 direct playback 或者 offload playback 播放模式时，另一个应用正在播放音频并且资源达到限制，则请求的应用无法创建新的 AudioTrack 。

Android15 中当应用请求使用 direct playback 或者 offload playback 播放模式时资源达到限制后，系统会让已经创建使用的 AudioTrack 对象也失效。

处理：暂时未复现，可能涉及 开播 SDK

### [支持16KB Page Size](https://dev.mi.com/distribute/doc/details?pId=1826#_22)
随着设备制造商不断打造具有更大物理内存 (RAM) 的设备，这些设备中的许多可能会配置 16 KB（最终更大）的页面大小，以优化设备的性能。添加对 16 KB 设备的支持可让应用在这些设备上运行，并帮助应用从相关性能改进中受益。
今年携带 Android 15 的设备可能尚不会使用 16 KB 页面，但国内厂商最终将跟随 Google，因此建议应用适配。

性能提升：
- 在系统面临内存压力时缩短应用启动时间：平均降低了 3.16%
- 降低应用启动时的功耗：平均降低 4.56%
- 相机启动速度更快：平均热启动速度加快 4.48%，冷启动速度平均加快 6.60%
- 缩短了系统启动时间：平均缩短了 1.5%（约 0.8 秒）

兼容性影响：
- 含有so库的应用需要重新构建支持 16KB 设备的应用，否则在16KB设备上很可能会crash。

处理：暂时未复现，可能涉及 开播 SDK 等引入的其它开源库


### [默认开启预测性返回动画](https://dev.mi.com/distribute/doc/details?pId=1826#_27)
从 Android 15 开始，移除了预测性返回动画的开发者选项。现在，对于已完全或在 activity 级别选择启用预测性返回手势的应用，系统现在会显示“返回主屏幕”“跨任务”和“跨 activity”等系统动画

处理：可以忽略


## 影响以Android 15为目标平台应用的行为变更

CCtargetSdkVersion 为 30，暂不考虑升级到 Android 15 version=35，所以这部分变更暂时不会影响到 CC

以下给出未来升级 targetSdkVersion 后 CC可能遇到的问题

### [对请求音频焦点的限制](https://dev.mi.com/distribute/doc/details?pId=1826#_43)

#### 特性背景

Android15之前， 应用可以无任何限制的申请音频焦点。Android 15针对此做出了规范。

#### 特性内容

面向 Android 15 的应用必须是前台应用或运行与音频相关的前台服务（ [mediaPlayback](https://developer.android.com/develop/background-work/services/fg-service-types#media), [camera](https://developer.android.com/develop/background-work/services/fg-service-types#camera),[microphone](https://developer.android.com/develop/background-work/services/fg-service-types#microphone), 或 [phoneCall](https://developer.android.com/develop/background-work/services/fg-service-types#phone-call)） 才能请求音频焦点。 如果应用程序在不满足这些要求之一时尝试请求焦点，则调用将返回 AUDIOFOCUS_REQUEST_FAILED。

#### 应用适配

如果需要播放音频或者录制音频，那么必须考虑当前是否为前台应用（**在前台**或者**有和音频相关的前台服务**）

### [Edge-to-edge（边到边）强制执行](https://dev.mi.com/distribute/doc/details?pId=1826#_53)

#### 特性背景

在Android 15设备上，targetSDK>=Android15的应用将强制进行全屏展示，并且状态和导航栏将保持透明化。  
targetSDK<Android 15的应用程序默认不会允许边到边的特性，即仍然保持用户层的View在状态栏和导航栏之间。  
当在Android15平台上之前使用的设置系统栏颜色的API将被弃用，包括[setNavigationBarColor](https://developer.android.com/reference/android/view/Window#setNavigationBarColor(int))，[setNavigationBarColor](https://developer.android.com/reference/android/view/Window#setNavigationBarColor(int))，即便使用这些方法设置了，系统也将默认保持沉浸式的体验。


#### 应用适配

如果应用targetSdk>=Android 15，那么针对布局需要设置android:fitsSystemWindows="true"



## 新功能和API

### ApplicationStartInfo API
Android 15提供了ApplicationStartInfo API供开发者获取应用启动的相关信息。
- getDefiningUid(): 大多数情况下与通常认为的 UID 相同，使用了 android: useAppZygote 属性和 Context.BIND_EXTERNAL_SERVICE 标志位的 service 可能会导致这个字段有所不同。
- getIntent(): 就是引发这个应用启动的intent。
- getLaunchMode(): 启动模式，本例中为0，意为LAUNCH_MODE_STANDARD。
- getPackageUid(): 即App安装时分配到的UID。
- getPid()、getProcessName(): 即进程Pid、进程名。
- getRealUid(): 大多数情况下与PackageUid相同，在涉及应用分身、多用户等情况下可能会不同。
- getReason(): 应用启动的原因。
- getStartType(): App 自动的类型
- getStartupState(): 启动状态
- getStartupTimestamps(): 得到不同启动阶段的时间戳
- wasForceStopped(): 应用是否是被 forcestop 结束的


### 音量控制
Android 15 引入了对 [CTA-2075](https://shop.cta.tech/products/loudness-standard-for-over-the-top-television-and-online-video-distribution-for-mobile-and-fixed-devices-ansi-cta-2075) 音量标准的支持，这一标准有助于确保用户在切换不同音频内容时，不会遇到显著的音量变化，例如从电影中的安静对话场景切换到响亮的动作场面，或者在不同的媒体应用之间切换。系统利用输出设备（耳机和扬声器）的已知特征以及 AAC 音频内容中提供的音量元数据来智能地调整音频音量和动态范围压缩级别。  
[CTA-2075](https://shop.cta.tech/products/loudness-standard-for-over-the-top-television-and-online-video-distribution-for-mobile-and-fixed-devices-ansi-cta-2075) 是一个关于音频响度标准的规范，由消费技术协会（Consumer Technology Association，简称 CTA）制定。这个标准旨在处理和改善音频内容播放时的响度一致性问题，尤其是在不同设备和内容之间切换时用户经常遇到的音量波动问题。


### 详细的应用大小信息
自 Android 8.0（API 级别 26）起，Android 就一直包含 [StorageStats.getAppBytes](https://developer.android.com/reference/android/app/usage/StorageStats?hl=zh-cn#getAppBytes()) API，该 API 将应用的安装大小汇总为一个字节，这些字节是 APK 大小、从 APK 中提取的文件的大小以及设备上生成的文件（例如预先 (AOT) 编译代码）的总和。就应用的存储空间使用情况而言，此数据不够详细。


### 封面屏幕支持
Android15为应用提供了充分利用android屏幕的技术支持，包含大屏幕、可翻转和可折叠。应用通过声明一个属性，借助该属性可让应用在Android15设备上的折叠屏或小屏幕上运行。

### 更顺滑的PIP进入
为解决pip进出动画不够连贯的问题，Android 15引入了新的API。
从 Android15 开始，PictureInPictureUiState 类中引入了一个新的状态 IsTransitioningToPip。当 Pip 进入动画开始时，系统将调用 onPictureInPictureUiStateChanged 回调，并且应用程序可以通过判断 isTransitioningToPip() = true 来隐藏叠加在主 UI 之上的 UI 元素，如视频播放中的布局，评分，标题等。





