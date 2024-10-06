---
需求: "#769868 CCSDK-2期 https://icc.pm.netease.com/v6/issues/769868"
tags: 
需求文档: https://docs.popo.netease.com/team/pc/uezznlm9/pageDetail/9b3fd13a05b343d9b63f2599b6047245?popo_locale=zh&xyz=1718788471606&appVersion=4.18.0&deviceType=0&popolocale=zh-CN&popo_hidenativebar=1&popo_noindicator=1&disposable_login_token=1#edit
交互: https://mastergo.netease.com/file/126598495998293?page_id=1%3A07305&shareId=126598495998293&devMode=true
UI:
  - https://mastergo.netease.com/file/122248773012537?page_id=597%3A04814&shareId=122248773012537&devMode=true
wiki: 
排期:
  - https://docs.popo.netease.com/lingxi/af483e448fbb4c56a970941ab84e13bc?xyz=1715571300391&tab=0
日志: https://docs.popo.netease.com/team/pc/uezznlm9/pageDetail/194984bc5ca841e8b25ee145383d3571?appVersion=4.17.0&deviceType=0&popolocale=zh-CN&popo_hidenativebar=1&popo_noindicator=1&disposable_login_token=1
耗时: 
备注: 
分支: 766878_CCSDK_1
publishTime: 2024-05-28
---


红点相关逻辑
```
com.netease.cc.roomplay.playentrance.moreentrance.PlayEntranceMoreController
```
服务端接口返回：42198-1 协议获取需要显示红点的玩法列表，然后调用 `com.netease.cc.roomplay.redpoint.GamePlayRedPointController#updateAllEntranceRedPoint` 显示红点

数字提醒逻辑
web 活动玩法 在这里设置
```
com.netease.cc.roomplay.playentrance.base.BaseEntranceModel#showRedPointNum
```




### 房间导航栏

RoomViewStateConstantsApi 相关迁移
ob 位及对应面板实现
tab 框架及对应面板实现


配置地址： https://webadmin.dev.cc.163.com/mongoadmin/mobile_common_config/GameRoomNavigationTabConfig/

需要检查下后台配置的活动，可能存在 CC 配置的活动会执行在 sdk 上不支持的 cclink




### 赛事房间背景
三端赛事直播间重构-移动端： https://km.netease.com/wiki/27/page/381734
赛事直播间改版 mixmatchentrycontrol(43846)： https://km.netease.com/wiki/27/page/381316



43846-3 协议增加赛事房间切换视图背景色





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


