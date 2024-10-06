---
需求: "#762715 家族IM消息体迭代 https://icc.pm.netease.com/v6/issues/762715"
tags: 
需求文档: https://docs.popo.netease.com/team/pc/uezznlm9/pageDetail/c7b8cb2800ff4d39b92d134a13796ac8
交互: https://mastergo.netease.com/file/121772849512462?page_id=1439%3A89840&shareId=121772849512462&devMode=true
UI:
  - https://mastergo.netease.com/file/120791024225119?fileOpenFrom=notification&page_id=2456%3A47092&devMode=true
wiki:
  - 答题业务数据
  - https://km.netease.com/wiki/27/page/429136
  - 答题消息
  - https://km.netease.com/wiki/27/page/418603#%E7%AD%94%E9%A2%98
  - 表情数据
  - https://km.netease.com/wiki/27/page/424947#%E8%A1%A8%E6%83%85%E5%9B%9E%E5%A4%8D%E5%B9%BF%E6%92%AD
排期:
  - https://docs.popo.netease.com/team/pc/uezznlm9/pageDetail/3d7b48ffbb6a464593817f1fbb8e6c67?popo_locale=zh&xyz=1713165006344&tab=0&appVersion=4.13.0&deviceType=0&popolocale=zh-CN&popo_hidenativebar=1&popo_noindicator=1&disposable_login_token=1&xyz=1713766115011
日志: 
耗时: 
备注: 通用图文卡片、长按表情互动
分支: 764065_family_6
publishTime: 2024-05-15
---
#### 点赞等操作数据

- 获取回复表情列表
	https://km.netease.com/wiki/27/page/424947#%E8%8E%B7%E5%8F%96%E5%9B%9E%E5%A4%8D%E8%A1%A8%E6%83%85%E5%88%97%E8%A1%A8 ]( https://km.netease.com/wiki/27/page/424947#%E8%8E%B7%E5%8F%96%E5%9B%9E%E5%A4%8D%E8%A1%A8%E6%83%85%E5%88%97%E8%A1%A8 )
	
- 表情回复 44243-6
	[https://km.netease.com/wiki/27/page/424947#%E8%A1%A8%E6%83%85%E5%9B%9E%E5%A4%8D](https://km.netease.com/wiki/27/page/424947#%E8%A1%A8%E6%83%85%E5%9B%9E%E5%A4%8D)
	
- 请求表情回复数据
- https://km.netease.com/wiki/27/page/424947#%E6%89%B9%E9%87%8F%E8%8E%B7%E5%8F%96%E6%B6%88%E6%81%AF%E8%A1%A8%E6%83%85%E5%9B%9E%E5%A4%8D%E6%95%B0%E6%8D%AE
	
- 表情回复广播 44243-7
	https://km.netease.com/wiki/27/page/424947#%E8%A1%A8%E6%83%85%E5%9B%9E%E5%A4%8D%E5%B9%BF%E6%92%AD ]( https://km.netease.com/wiki/27/page/424947#%E8%A1%A8%E6%83%85%E5%9B%9E%E5%A4%8D%E5%B9%BF%E6%92%AD )

#### 通用图文消息

-  通用图文消息格式
	https://km.netease.com/wiki/27/page/418603#%E9%80%9A%E7%94%A8%E5%9B%BE%E6%96%87%E6%B6%88%E6%81%AF




不支持展示消息 UI
各类型消息 UI


### 改动范围
将家族从 IM 模块拆出，需要回归下 IM 和家族功能
日志：
组队、图文、分享直播间 之前进房没有 motive，补了；
点击消息卡片 进房 motive 增加了日志要求字段；
点击用户状态 进房 motive 有修改了实现逻辑；
本期日志

