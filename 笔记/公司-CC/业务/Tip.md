```kotlin
在锚点底下，自动消失，tip 上边距3

CTip.Builder()  
    .lifecycleOwner(this)  
    .text("去看看家族奖励吧~")  
    .anchorView(binding.layoutFamilyTop.mainPage)  
    .isOutsideTouchable(true)  
    .showArrow(true)  
    .delayDismiss(30000)  
    .alignEdge(CBaseTip.ALIGN_CENTER)  
    .yOffset(3.dp)  
    .xOffset(-3.dp)  
    .position(CBaseTip.SHOW_BELOW)  
    .build()  
    .show()
```

alignXOffset：整体偏移，不影响箭头位置，<0 往左移，>0 往右移
