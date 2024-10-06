---
需求: "#749932 【IM模块】第2期群互动送礼/红包"
tags: 
需求文档: https://docs.popo.netease.com/team/pc/east/pageDetail/7ecbeb785381429d93f754745be2e8dd
交互: https://g.126.fm/00e3dXc
UI:
  - https://g.126.fm/03l7J7Q
wiki:
  - 礼物
  - https://km.netease.com/team/cc_studio/wiki/page/34845#41894-7-%E7%BE%A4%E8%81%8A%E6%A8%AA%E5%B9%85%E7%89%B9%E6%95%88%E5%B9%BF%E6%92%AD
  - 家族消息
  - https://km.netease.com/team/cc_studio/wiki/page/418612
排期: 
日志: 
耗时: 
备注: 礼物、红包
分支: 745102_family_group2
publishTime: 2024-01-22
---


私聊送礼横幅：
只有自己送的礼物以及对方送自己的礼物触发
如果是连送的就加到同个通道，否则交替添加到两个通道


每次进入家族界面请求 41894/1 获取版本号和 url
- 版本号变化
	- 请求 url，更新 礼物缓存与版本号缓存
- 版本号不变
	- 使用礼物缓存




#### 家族 Activity 多实例问题
家族 Activity 由 Activity+Fragment 实现
**问题**：同一个 Activity 由于绑定数据不同，所以允许存在多个 Activity 实例，但数据相同的 Activity 只能有一个，否则同一个广播协议可能会被多个实例处理。
**处理**：在打开新 Activity 实例后销毁已存在相同数据的 Activity

