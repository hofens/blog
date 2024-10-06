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
分支: release_v2.4.8
publishTime: ""
---


### 公屏动效
使用 setItemAnimator (new DefaultItemAnimator ())可以让数据有个自带的插入后往上移动的效果


方案一：
使用 setStackFromEnd=true 可以使数据从 Recycleview 底部开始往上插

表现：再 scrollToPosition 到底部就能实现 从底部往上插入后移动的效果，但会有单条抖动情况

抖动原因：应该是由于 setStackFromEnd 使数据从底部插入，当 holder 高度变化时，是往上扩展，调用 scrollToPosition 往下滑就会有个抖动效果

方案二：
去除 setStackFromEnd，数据从顶部插入

表现：会有个滑动到顶部再滑到底部的效果
解决办法：
按照前人的处理是用以下代码
```java
//用这个诡异的方法先滑动可以防止，列表里消息数量不多时，动画时候出现的异常抖动  
//原因是找anchorView时候 getViewPosition 一直为-1，导致默认的anchorView为0  
// 使得消息列表会有明显的先上滑到顶端再下滑，猜测是某种神奇的时序问题  
//offset可以使得列表的position被重新分配，消除消息加入后的抖动影响
if (linearLayoutManager != null && adapter != null && adapter.getItemCount() -2 >=0) {  
    linearLayoutManager.scrollToPositionWithOffset(adapter.getItemCount() - 2, Integer.MIN_VALUE);  
}  
// post会导致smooth失效
recyclerView.post(() -> {  
    if (linearLayoutManager != null && adapter != null && adapter.getItemCount() > 0) {  
        recyclerView.smoothScrollToPosition(adapter.getItemCount() - 1);  
    }  
});
```

这可以解决问题，但 recyclerView. post 其实是会导致 smoothScroll 失效的


