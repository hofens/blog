
## 原理
是在 Activity 中放了一个继承自 FrameLayout 的 ReactRootView，这个 View 收到的触摸、点击、手势以及 Activity 的 Bundle 数据都会通过 JSBridge 传给





## 搭建 demo

公司电脑又 McAfee，需要调用 `taskkill /f /t /im macmnsvc.exe` 关闭




# 网易云音乐 RN 新架构升级实践

https://zhuanlan.zhihu.com/p/672232841

### 新架构
RN 新架构的核心主要有三方面的优化 —— Fabric、TurboModule 和 Hermes，分别对应组件渲染、信息通信和执行引擎，三项优化都可以独立开启和关闭，接入复杂度上 Hermes 接入适配成本相对最低；Fabric 和 TurboModule 都需要进行代码改造适配后才能启用，TurboModule 开启后 NativeModule 仍然可以使用，改造成本适中，Fabric 的开启最为复杂，由于 Fabric 开启后只支持渲染 FabricComponent，所以需要将原来的 NativeComponent 全部改造为 FabricComponent 才能使用，Fabric 在三者中适配成本最高。