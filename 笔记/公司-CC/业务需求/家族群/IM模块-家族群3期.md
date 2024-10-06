---
需求: "#755868 IM三期开发及二期验收调整 https://icc.pm.netease.com/v6/issues/755868"
tags: []
需求文档: https://docs.popo.netease.com/team/pc/east/pageDetail/07a86d6d3b5a4e81876f3b816601a73e?popo_locale=zh&xyz=1702880053384&tab=1&appVersion=4.6.1&deviceType=0&popolocale=zh-CN&popo_hidenativebar=1&popo_noindicator=1&disposable_login_token=1&xyz=1708311493763
交互: https://g.126.fm/00e3dXc
UI:
  - https://g.126.fm/03l7J7Q
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
备注: 
分支: family_group_v3
publishTime: 2024-02-19
---



1.Recycleview 实现上滑移除 Item 效果可以使用 `ItemTouchHelper`，分页效果可以使用 `PagerSnapHelper`
```kotlin
// 实现分页效果  
PagerSnapHelper().attachToRecyclerView(this)  
// 实现滑动删除效果  
ItemTouchHelper(object : ItemTouchHelper.SimpleCallback(0, ItemTouchHelper.UP) {  
    override fun onMove(  
        @NonNull recyclerView: RecyclerView,  
        @NonNull viewHolder: RecyclerView.ViewHolder,  
        @NonNull target: RecyclerView.ViewHolder  
    ): Boolean {  
        return true  
    }  
  
    override fun onSwiped(@NonNull viewHolder: RecyclerView.ViewHolder, direction: Int) {  
        // 处理滑动操作  
        redPacketAdapter.removeItem(viewHolder.absoluteAdapterPosition)  
    }  
}).attachToRecyclerView(this)
```

2.一次性打开多个 h 5的情况下，如果 h 5 请求了 tcp 但没有限定响应的 h 5，可能会导致多个 h 5 都发生修改


3、kotlin 中 MutableList 是一个可变的集合类型，当你将一个 MutableList 对象赋值给另一个变量时，它们实际上引用的是同一个集合实例。因此，对其中一个变量所引用的集合进行的更改也会影响另一个变量所引用的集合，因为它们指向同一个内存位置。
```kotlin
fun submitList(newList: MutableList<RedPacketInfo>) {  
	this.redPacketList = newList  
}  
这样写，修改 redPacketList 中的数据，传入 submitList 的对象源数据也会被修改
```

4、
`val familyNoticeListLD: MutableLiveData<FamilyNoticeList> = MutableLiveData()`
由于 LiveData 在 onPause 后是不活跃的，observe 不会监听，再加上 postValue 是后台线程执行