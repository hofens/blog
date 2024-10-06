---
需求: "#755868 IM三期开发及二期验收调整 https://icc.pm.netease.com/v6/issues/755868"
tags: 
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
备注: 互动
分支: 758397_family_group_4
publishTime: 2024-03-13
---

回归：
新人礼包移动至小插件定位

com. netease. cc. message. share. adapter. ShareToFriendAdapter. ShareToFriendViewHolder#bindData
家族上方加 tag



#### wiki
用户状态【P 2】：头像会展示各状态【开播中】【开黑中】【在线】[离线]： https://km.netease.com/team/cc_studio/wiki/page/418612#%E8%8E%B7%E5%8F%96%E7%BE%A4%E7%94%A8%E6%88%B7%E4%BF%A1%E6%81%AF




---


### 业务逻辑

#### 互动
目前互动形式有多种：
1. 服务端下发系统富文本消息，点击高亮部分，执行通用 cc://或互动协议 cc://family-chat-group-interact
2. 若为 cc://family-chat-group-interact，则解析后会请求 44243/3，参数为 cc://携带的参数；服务端收到后根据参数对应操作下发欢迎消息/显示打招呼面板/ 互动消息
	- 互动协议接口： https://km.netease.com/team/cc_studio/wiki/page/424947#%E4%BA%92%E5%8A%A8
	- 打招呼引导： https://km.netease.com/team/cc_studio/wiki/page/424947#%E6%89%93%E6%8B%9B%E5%91%BC%E8%A1%A8%E6%83%85%E5%BC%95%E5%AF%BC

#### 互动消息
1. 历史消息（`v1/mixfamilygroup/group-msg/getRecentMsg`）：会带 interactTools 和 interactInfos 
	-  https://km.netease.com/team/cc_studio/wiki/page/418612#%E8%8E%B7%E5%8F%96%E7%BE%A4%E6%9C%80%E8%BF%91%E8%81%8A%E5%A4%A9%E6%B6%88%E6%81%AF
2. 新消息广播（44178-1）：只有 interactTools 没有 interactInfos
	- https://km.netease.com/team/cc_studio/wiki/page/418612#%E6%96%B0%E6%B6%88%E6%81%AF%E9%80%9A%E7%9F%A5 
3. 互动数据变更广播（44243-5）：会带 UpdateInteractInfo ，需要用这个更新 interactInfos，两个数据结构不一样
	- https://km.netease.com/team/cc_studio/wiki/page/424947#%E4%BA%92%E5%8A%A8%E4%BF%A1%E6%81%AF%E5%B9%BF%E6%92%AD

流程： https://km.netease.com/team/cc_studio/wiki/page/424947#%E5%8F%91%E8%A8%80%E5%BF%AB%E6%8D%B7%E4%BA%92%E5%8A%A8%E7%BB%84%E4%BB%B6%E6%B5%81%E7%A8%8B


由于存在**互动数据变更广播**的数据要更新**新消息广播**中的 interactInfos，所以在遍历消息时，会给 interactInfos 为 null 的数据构造个对象，便于在**互动数据变更广播**中更新 interactInfos


#### 待办
点击后请求对应接口，服务端下发对应消息，SVGA 显示；新消息会先播放一次 SVGA 后显示静态图；历史消息显示静态图


消息格式： https://km.netease.com/team/cc_studio/wiki/page/418603#骰子

群内增加互动入口，支持点击随机发出骰子及石头剪子布：

群待办区增加骰子及石头剪子布： https://km.netease.com/team/cc_studio/wiki/page/418612#%E7%BE%A4%E4%BB%A3%E5%8A%9E%E5%8C%BA

发出骰子： https://km.netease.com/team/cc_studio/wiki/page/424947#%E6%8A%95%E6%8E%B7%E9%AA%B0%E5%AD%90

石头剪子布： https://km.netease.com/team/cc_studio/wiki/page/424947#%E7%9F%B3%E5%A4%B4%E5%89%AA%E5%88%80%E5%B8%83



---

### 记录
#### 用浏览器打开了 cc://，会打开 `data:text\html` 这个页面

```kotlin

if (CcSchemeHelper.isCCScheme(link)) {  
	CcSchemeHelper.dealWhenSupport(link, {  
		CCSchemeParseHelper.parseSchemeUrl(ApplicationWrapper.getTopActivity(), link)  
		
	}) {  
		CLog.i("jumpCCOrOtherLink", "不支持该协议, 无法跳转。$link")  
	}  
	return  
}  


反例，留意return@dealWhenSupport

if (CcSchemeHelper.isCCScheme(link)) {  
	CcSchemeHelper.dealWhenSupport(link, {  
		CCSchemeParseHelper.parseSchemeUrl(ApplicationWrapper.getTopActivity(), link)  
		return@dealWhenSupport
	}) {  
		CLog.i("jumpCCOrOtherLink", "不支持该协议, 无法跳转。$link")  
		return@dealWhenSupport
	}   
}  

@dealWhenSupport不会结束这个function

```



历史消息有 interactTools 和 interactInfos 展示底部互动区

更新消息，监听互动信息广播（44243/5）, 通过 msgId 更新消息 interactInfos

新消息，只有 interactTools 信息

changeUserInfos：执行操作的用户，根据 bcType 判断是执行还是取消执行
showUserInfos：前几个展示的头像


收到加群成功通知，请求打招呼协议，判断下是否在群里

#### layoutDirection
父容器镜像，通过 `layoutDirection` 禁止该子布局镜像后，如果是 layout_marginStart 就还是会受镜像影响，layout_marginLeft 不会
``` xml
<com.netease.cc.widget.dragflowlayout.FlowLayout  
    android:id="@+id/layout_interact_info"  
    android:layout_width="wrap_content"  
    android:layout_height="wrap_content"  
    android:layoutDirection="ltr"  
    android:layout_marginLeft="8dp"  
    android:layout_below="@+id/tv_chat_msg"  
    android:layout_alignStart="@+id/tv_chat_msg"/>
```

---
### 待办

1、分享弹窗家族上方加 tag 
`com. netease. cc. message. share. adapter. ShareToFriendAdapter. ShareToFriendViewHolder #bindData`
未处理

2、大插件
`com.netease.cc.message.family.plugin.PluginBigViController`
已处理：容器加入大插件
未处理：剩下位置没调整、url 替换

3、待办区
```
com/netease/cc/message/family/todo/FamilyTodoRepository.kt:116
com.netease.cc.message.family.todo.FamilyTodoAreaController#initAtAllLayout
com.netease.cc.message.family.todo.FamilyTodoAreaController#initWebPageLayout
```
已处理：增加 H 5、@所有人类型
未处理：接收到@所有人消息需要高亮、艾特消息提醒
https://km.netease.com/team/cc_studio/wiki/page/418612#%E7%BE%A4%E4%BB%A3%E5%8A%9E%E5%8C%BA

4、消息翻页逻辑修改
https://km.netease.com/team/cc_studio/wiki/page/418612#%E8%8E%B7%E5%8F%96%E7%BE%A4%E6%9C%80%E8%BF%91%E8%81%8A%E5%A4%A9%E6%B6%88%E6%81%AF

服务端要求使用接口返回的 beforeId 替代目前客户端自己计算 beforeId 逻辑，若 records 为空则根据 beforeId 继续查询，直到 hasMore=false