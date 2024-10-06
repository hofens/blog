#### maxOfNull



``` kotlin 
var use4RowLayout = false  
run loop@ {  
    data.optionBean?.forEach { option ->  
        DataBindingUtil.bind<LayoutFamilySubjectOptionItemBinding>(  
            host.layoutInflater.inflate(R.layout.layout_family_subject_option_item, null)  
        )?.let {  
            it.tvOption.text = option.text  
            UITools.measureView(it.root)  
            if (it.root.measuredWidth > singleOptionItemMinWidth) {  
                use4RowLayout = true  
                return@loop  
            }  
        }  
    }}
```

有误！不一定比 forEach 好，内部也是会遍历每个 item
```kotlin
val maxOptionWidth = data.optionBean?.maxOfOrNull { option ->  
    DataBindingUtil.bind<LayoutFamilySubjectOptionItemBinding>(  
        host.layoutInflater.inflate(R.layout.layout_family_subject_option_item, null)  
    )?.apply {  
        tvOption.text = option.text  
        UITools.measureView(root)  
        CLog.d(tag, "题目内容 ${option.text} 测量宽度 ${root.measuredWidth}, 切换布局宽度阈值 $singleOptionItemMinWidth")  
    }?.root?.measuredWidth ?: 0  
} ?: 0
```

any 会在满足条件的时候结束循环
```kotlin
use4RowLayout = data.optionBean?.any { option ->  
    tvOption.text = option.text  
    CLog.d(TAG, "测量宽度 $measuredWidth, 切换布局宽度阈值 $singleOptionItemMinWidth, 题目内容 ${option.text} ")  
    measuredWidth > singleOptionItemMinWidth  
} ?: false
```
