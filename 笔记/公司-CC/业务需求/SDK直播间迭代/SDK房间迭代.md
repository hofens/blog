---
需求: "#767354 CCSDK-1期-Android https://icc.pm.netease.com/v6/issues/767354"
tags: 
需求文档: https://docs.popo.netease.com/team/pc/uezznlm9/pageDetail/7d06adb44c754d47be1f04e0c7f3918a
交互: https://mastergo.netease.com/file/126598495998293?page_id=1%3A07305&shareId=126598495998293&devMode=true
UI:
  - https://mastergo.netease.com/file/122248773012537?page_id=597%3A04814&shareId=122248773012537&devMode=true
wiki: 
排期:
  - https://docs.popo.netease.com/lingxi/af483e448fbb4c56a970941ab84e13bc?xyz=1715571300391&tab=0
日志: https://docs.popo.netease.com/team/pc/uezznlm9/pageDetail/194984bc5ca841e8b25ee145383d3571?appVersion=4.17.0&deviceType=0&popolocale=zh-CN&popo_hidenativebar=1&popo_noindicator=1&disposable_login_token=1
耗时: 
备注: 答题
分支: 766878_CCSDK_1
publishTime: 2024-05-28
---



## 礼物架

语音厅礼物架
```java
com.netease.cc.gift.manager.ShowGiftShelfManager#showGiftDialog(int, int, int, java.lang.String)

调用入口
com.netease.cc.gift.old.view.GiftLogoView#init


把入口变成giftView
//礼物icon container  
ViewGroup giftLogoViewContainer = view.findViewById(R.id.btn_gift_logo);  
IGiftService giftService = ServiceController.getService(IGiftService.class);  
if (giftLogoViewContainer != null && giftService != null && ConfigHelper.enableAudioHallSendGift()) {  
    giftService.initGiftLogoView(giftLogoViewContainer, false);  
}

```

视频房间礼物架
```java
com.netease.ccdsroomsdk.activity.roomcontrollers.RoomGiftController#showGiftDialog

礼物架数据源
com.netease.cc.gift.controller.RoomGiftShelfController#loadGiftData(boolean, boolean)
```


旧的视频房间礼物架
``` java
数据源
com.netease.ccdsroomsdk.activity.gift.fragment.GiftBaseFragment#loadGiftData
```



## 功能



### 改动范围
1、礼物架
视频房间基于语音厅 礼物架修改，可能有影响到
视频房间礼物架可能会比 ios 多一些功能，可以让我去除


2、功能配置外部入口与更多弹窗中的系统功能
没区分房间语音厅也会显示




### 备忘
```
db46fbf95edc1444c3c8cae1fd41ddf52040baab
隐藏礼物特效

340a56abc3e1e9f28e65ef5d406f963aa85ce4b7
d25b18557e4d116c22099a28751287a0eb6c374d
隐藏礼物架提醒、包裹等

3cf68bd409a8dc7e5f764d49502eb1a4b12a55c7
隐藏红点


```


