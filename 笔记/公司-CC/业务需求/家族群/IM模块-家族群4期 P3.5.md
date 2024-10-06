---
需求: 
tags: []
需求文档: https://docs.popo.netease.com/team/pc/east/pageDetail/18352bd100ba47d4b60ff0089797b014
交互: https://g.126.fm/00e3dXc
UI:
  - https://mastergo.netease.com/file/120791024225119?fileOpenFrom=notification&page_id=0%3A122341
wiki:
  - 家族消息
  - https://km.netease.com/team/cc_studio/wiki/page/418612
  - 群代办
  - https://km.netease.com/team/cc_studio/wiki/page/418612#%E7%BE%A4%E4%BB%A3%E5%8A%9E%E5%8C%BA
排期:
  - https://url.163.com/HfzY
日志: 
耗时: 
备注: 大插件、@全体
分支: family_group_v5
publishTime: 2024-04-09
---


### 本期内容


1、分享弹窗家族上方加 tag 
`com.netease.cc.message.share.adapter.ShareToFriendAdapter.ShareToFriendViewHolder#bindData`
没区分直播间，可能非直播间拉起的分享弹窗也有家族 tag
分享渠道修改角标样式逻辑

2、大插件
`com.netease.cc.message.family.plugin.PluginBigViController`
待前端处理

3、家族门槛
（1）客户端通过公告栏加入是请求 44177/12 加群，目前已有对 code!=0 加了 toast。
（2）用的是 44177/11 ， code!=0 有加了 toast，复用 code=4，弹窗文案修改为服务端下发
（3）同 1，44177/12 回包中 code!=0 我们就会弹 toast
![](assets/Pasted%20image%2020240328173606.png)

被踢提示弹窗文案修改，采用服务端下发 msg

4、待办区
```
com/netease/cc/message/family/todo/FamilyTodoRepository.kt:116
com.netease.cc.message.family.todo.FamilyTodoAreaController#initAtAllLayout
com.netease.cc.message.family.todo.FamilyTodoAreaController#initWebPageLayout
```
待办区增加 H 5、待办区@所有人类型
接收到@所有人消息需要高亮、艾特消息提醒：富文本支持@用户解析、区分主客态；增加@全体采用富文本方式发送
https://km.netease.com/team/cc_studio/wiki/page/418612#%E7%BE%A4%E4%BB%A3%E5%8A%9E%E5%8C%BA

5、消息翻页逻辑修改
https://km.netease.com/team/cc_studio/wiki/page/418612#%E8%8E%B7%E5%8F%96%E7%BE%A4%E6%9C%80%E8%BF%91%E8%81%8A%E5%A4%A9%E6%B6%88%E6%81%AF

服务端要求使用接口返回的 beforeId 替代目前客户端自己计算 beforeId 逻辑，若 records 为空则根据 beforeId 继续查询，直到 hasMore=false

