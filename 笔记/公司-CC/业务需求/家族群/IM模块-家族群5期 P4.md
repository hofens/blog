---
需求: 
tags: 
需求文档: https://docs.popo.netease.com/team/pc/east/pageDetail/18352bd100ba47d4b60ff0089797b014
交互: https://mastergo.netease.com/file/121772849512462?page_id=871%3A000880&shareId=121772849512462
UI:
  - https://mastergo.netease.com/file/120791024225119?fileOpenFrom=notification&page_id=0%3A122341
wiki:
  - 家族消息
  - https://km.netease.com/team/cc_studio/wiki/page/418612
  - 顶部列表
  - https://km.netease.com/wiki/27/page/425195?parent_id=292085&wiki=27&has_child=false#%E8%8E%B7%E5%8F%96%E5%AE%B6%E6%97%8F%E6%88%90%E5%91%98%E7%8A%B6%E6%80%81%E5%88%97%E8%A1%A8
  - 消息处头像
  - https://km.netease.com/wiki/27/page/418612?parent_id=292085&wiki=27&has_child=false#%E8%8E%B7%E5%8F%96%E7%BE%A4%E7%94%A8%E6%88%B7%E4%BF%A1%E6%81%AF
  - 历史消息增加quoteInfo
  - https://km.netease.com/wiki/27/page/418612?parent_id=292085&wiki=27&has_child=false#%E5%93%8D%E5%BA%94
  - 发言增加quoteMsgId
  - https://km.netease.com/wiki/27/page/418612?parent_id=292085&wiki=27&has_child=false#%E7%BE%A4%E5%8F%91%E8%A8%80
排期:
  - https://docs.popo.netease.com/team/pc/uezznlm9/pageDetail/a70f6377617c4b9cb06aef27168b89f9
日志: 
耗时: 
备注: 状态、回复、表情、视频卡片、禁止行为
分支: 761672_family_group_5
publishTime: 2024-04-23
---


### 本期内容
![|775](assets/Pasted%20image%2020240409163550.png)



举报 
协议：40/10

> [!note] 
> 举报 
协议：40/10




状态列表
- 顶部列表
- 消息列表、头像 UI
- 通告栏从盖在消息列表上 改为 和消息列表同层级

回复
- 各类型消息提供引用预览内容
- 富文本、文本消息增加 引用 UI
- 引用定位、点击
- 输入框限制 200 字
- 不可见消息、已删除消息显示

表情
- 富文本、文本消息解析
- 家族表情入口开关 family_group_emotion_switch

 ---

1、之前当消息区域高度发生变化时可能底部导致截断，优化了这块逻辑
2、各消息类型提供被引用内容
3、增加表情，需要看下表情入口显隐 UI

1、通告栏/状态列表从无到显示时、迎新表情包出现时、互动头像出现时、观察底部消息是否截断
2、各类型消息表现


### [[ObservableField]]
使用 ObservableField 替代 viewholder 的 payload 做局部刷新

### 状态列表
```
com.netease.cc.message.family.member.FamilyMemberStateController
```
顶部列表状态与消息列表状态处理


### 家族表情入口开关
key: family_group_emotion_switch
value: 0/1


### 数据解析中的类型转化
```kotlin
private var jModel2ViewBeanConvert: AdapterTypeConvert? = null
setViewBeanConvert(viewBeanConvertList)
```

设置转换接口 bean
```kotlin
interface ChatMsg2ViewBeanConvert {  
    fun filter(chatMsg: FamilyChatMsgJModel): Boolean  
  
  
    fun convert(chatMsg: FamilyChatMsgJModel): FamilyChatMessage  
  
}
```

可能有多个转换规则，设置转换接口 bean 集合
```kotlin
val viewBeanConvertList: LinkedHashSet<ChatMsg2ViewBeanConvert> by lazy {  
    val set = LinkedHashSet<ChatMsg2ViewBeanConvert>()  
    var unSupportConvert: ChatMsg2ViewBeanConvert? = null  
    supportTypes.onEach { type ->  
        val holder = getViewHolderAdapterByType(type) ?: return@onEach  
        val vhAdapter = (holder as ChatVhAdapter<*, *>)  
        if (type == FAMILY_CHAT_UN_SUPPORT) {  
            unSupportConvert = vhAdapter.provideConvert()  
        } else {  
            set.add(vhAdapter.provideConvert())  
        }  
    }  
    unSupportConvert?.let { set.add(it) }  
    set  
}
```


设置 `jModel2ViewBeanConvert`，调用 `ChatMsg2ViewBeanConvert` 转换数据
```kotlin
fun setViewBeanConvert(convertList: LinkedHashSet<ChatMsg2ViewBeanConvert>?) {  
    if (convertList == null) {  
        this.jModel2ViewBeanConvert = null  
        return    }  
    this.jModel2ViewBeanConvert = { msgJModel ->  
        var ret: FamilyChatMessage? = null  
        var quoteFamilyChatMessage: FamilyChatMessage? = null  
  
        msgJModel.quoteMsgInfo?.let { quoteMsg ->  
            for (convert in convertList) {  
                if (convert.filter(quoteMsg)) {  
                    quoteFamilyChatMessage = convert.convert(quoteMsg)  
                    // todo hofe  
  
                    break  
                }  
            }  
        }  
        for (convert in convertList) {  
            if (convert.filter(msgJModel)) {  
                ret = convert.convert(msgJModel)  
                ret.quoteFamilyChatMessage = quoteFamilyChatMessage  
                break  
            }  
        }  
  
        ret  
    }  
}
```

调用 `jModel2ViewBeanConvert` 输入数据进行转化
```kotlin
jModel2ViewBeanConvert!!.invoke(chatBean)?.apply {  
    dmBean = this@FamilyChatViewModel.dmBean  
    afterInstance()  
    if (msgIsDeleted(msgJModel.msgId ?: msgJModel.clientMsgId)) {  
        return@onEachIndexed  
    }  
    addNewChatMsgProcessorList.onEach {  
        if (it.shouldProcessMessage(this)) {  
            it.processMessage(this)  
        }  
    }  
    if (index - 1 >= 0) {  
        commonViewBean.previousMsgTime = msgList[index - 1].msgTime ?: 0L  
    }  
    //获取红包消息  
    retList.add(this)  
}
```



### 引用实现
消息引用前一条消息，由于消息是 model 转成 viewbean 的，所以只要在消息 model 中带上引用消息的 model 就可以把引用也转成 viewbean，看上述 quoteMsgInfo 处理



