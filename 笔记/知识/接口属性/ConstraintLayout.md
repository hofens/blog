
layout_constraintWidth_max 限制 View 最大值

com.netease.cc.widget.cornerclip.CornerClipConstraintLayout
会裁剪，所以如果子元素需要超出父布局，不要用这个



### 解决布局超出问题

可以使 TextView 和 ImageView 在同一行中

> [!NOTE] layout_constrainedWidth
> TextView 的 layout_constrainedWidth 属性设置为 true。这意味着如果文本内容超出了可用的宽度, TextView 的宽度将受到约束, 可能会被压缩以适应约束条件。
> 如果将 layout_constrainedWidth 设置为 false, 则 TextView 的宽度将根据其文本内容的实际长度来确定, 即使超出了可用的宽度, 也不会受到约束。



layout_constraintHorizontal_chainStyle 可能可以不用

```xml
app:layout_constrainedWidth="true"  
app:layout_constraintHorizontal_bias="0"  
app:layout_constraintHorizontal_chainStyle="packed" 

```



``` xml
<TextView  
    android:id="@+id/tv_h2"  
    android:layout_width="wrap_content"  
    android:layout_height="wrap_content"  
    android:maxLines="1"  
    android:ellipsize="end"  
    android:textColor="@color/color_18181A"  
    android:textSize="@dimen/general_font_size_14"  
    android:text="@{data.h2Span}"  
    isVisible="@{data.h2Span != null}"  
    app:movementMethod="@{data.movementMethod}"  
    android:layout_marginStart="6dp"  
    bind:layout_constrainedWidth="true"  
    bind:layout_constraintHorizontal_bias="0"  
    app:layout_constraintHorizontal_chainStyle="packed"  
    bind:layout_constraintTop_toTopOf="parent"  
    bind:layout_constraintStart_toEndOf="@id/iv_icon"  
    bind:layout_constraintEnd_toStartOf="@id/flTag"  
    tools:text="像我像我像我像我像我" />  
  
<ImageView  
    android:id="@+id/flTag"  
    android:layout_width="wrap_content"  
    android:layout_height="wrap_content"  
    android:gravity="center_vertical"  
    bind:layout_constraintEnd_toEndOf="parent"  
    bind:layout_constraintTop_toTopOf="@id/tv_h2"  
    bind:layout_constraintBottom_toBottomOf="@id/tv_h2"  
    bind:layout_constraintStart_toEndOf="@id/tv_h2">
```


ImageView 跟随 TextView：`layout_constraintStart_toEndOf="@id/tv_h2"`
ImageView 固定右边：去除 `layout_constraintStart_toEndOf="@id/tv_h2"`




