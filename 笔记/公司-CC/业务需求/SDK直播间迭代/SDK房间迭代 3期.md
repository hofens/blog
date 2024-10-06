---
需求: "#769868 CCSDK-2期 https://icc.pm.netease.com/v6/issues/769868"
tags: 
需求文档: https://docs.popo.netease.com/team/pc/uezznlm9/pageDetail/9b3fd13a05b343d9b63f2599b6047245?popo_locale=zh&xyz=1718788471606&appVersion=4.18.0&deviceType=0&popolocale=zh-CN&popo_hidenativebar=1&popo_noindicator=1&disposable_login_token=1#edit
交互: https://mastergo.netease.com/file/126598495998293?page_id=1%3A07305&shareId=126598495998293&devMode=true
UI:
  - https://mastergo.netease.com/file/122248773012537?page_id=597%3A04814&shareId=122248773012537&devMode=true
wiki: 
排期:
  - https://docs.popo.netease.com/lingxi/2c5ffbd46e4d47a3add169a5ce49bc98?appVersion=4.21.0&deviceType=0&popo_hidenativebar=1&popo_noindicator=1&disposable_login_token=1&xyz=1722499837312&tab=3
日志: https://docs.popo.netease.com/team/pc/uezznlm9/pageDetail/194984bc5ca841e8b25ee145383d3571?appVersion=4.17.0&deviceType=0&popolocale=zh-CN&popo_hidenativebar=1&popo_noindicator=1&disposable_login_token=1
耗时: 
备注: 
分支: release_v2.4.7
publishTime: 2024-08-07
---


### 播放暂停等

确认下这个作用

```
ccgroomsdk__layout_room_video_topbar.xml
layout/ccgroomsdk__layout_room_video.xml:51
```


### 赛事房间背景
三端赛事直播间重构-移动端： https://km.netease.com/wiki/27/page/381734
赛事直播间改版 mixmatchentrycontrol(43846)： https://km.netease.com/wiki/27/page/381316
43846-3 协议增加赛事房间切换视图背景色

需要隐藏背景的话，可以调用
`com.netease.ccdsroomsdk.activity.roomcontrollers.wallpaper.BaseRoomSkinController#updateWallPaperVisible`
内部也提供业务过滤的接口

### viewpager
在使用 viewpager 的时候，需要设置 FragmentPagerAdapter，如果 有多个 viewpager，且在创建 FragmentPagerAdapter 的时候，给这个 adapter 传入的是同一个  fragment 的 fragmentManager，那么在 notifydatasetchange 的时候，可能会因为其它 viewpager 已存在 fragment，所以在当前 viewpager 就不再创建重复 fragment



### 播放器
要播放视频都需要先获取到 主播的ccid
```
com.netease.ccdsroomsdk.activity.roomcontrollers.RoomVideoController#startFastLiveVideo
会先尝试获取快播地址#startFastLiveVideo()，若有快播则调用startPlayByVideoPath()，无快播的情况下使用mMobileUrl拼接ccid获取播放链接#startPlayByMobileUrl();

不管用哪种方式，播放后都会赋值mVConfig;

当6144_15协议获取直播信息后，会得到mobileurl以及通知播放事件VideoController.MSG_VIDEO；如果先前未获取到主播ccid进行播放(mVConfig=null)，则在收到这个事件通知后会使用mobileurl兜底播放;

时移会调用startPlayByVideoPath()，因为这种播放方式带进度

```







