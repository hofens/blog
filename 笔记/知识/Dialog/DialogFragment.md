
#### 导航栏覆盖弹窗问题

如果弹窗是全屏的 MATCH_PARENT 或者 WRAP，导航栏不会遮挡弹窗；
若是固定高度弹窗，则会被导航栏所覆盖。

**解决方式**
根据导航栏高度动态调整弹窗高度，注意 `return v.onApplyWindowInsets(insets); `，如果只 `return insets; ` 会导致 VIVO 手机上导航栏不绘制。丢失弹窗导航栏、状态栏设置?

```java
window.getDecorView().setOnApplyWindowInsetsListener(new View.OnApplyWindowInsetsListener() {  
    @Override  
    public WindowInsets onApplyWindowInsets(View v, WindowInsets insets) {  
        WindowManager.LayoutParams attributes = window.getAttributes();  
        if (attributes == null) {  
            return insets;  
        }  
        int fixNavigationDialogHeight = getFixNavigationDialogHeight(builder);  
        if (attributes.height != fixNavigationDialogHeight) {  
            attributes.height = builder.height;  
            window.setAttributes(attributes);  
        }  
        return v.onApplyWindowInsets(insets);  
    }  
});

/**  
 * 借助{@link WindowInsets}来获取设备当前的导航栏高度，竖屏获取的是底边，横屏获取的是右边（因为弹窗一般都是底边或右边）  
 * 原理：View在Attach之后会获得一个{@link AttachInfo}，里面包含了WindowInsets数据，其中WindowInsets记录了系统UI的相关数据（如状态栏高度、导航栏高度等等），  
 *      会比原来通过访问navigation_bar_height资源拿到的会准确点（主要是兼容性问题）。  
 * 注意：1. 广泛的说用任何一个View都可以获取到WindowInsets数据，但前提是这个View已经Attached，否则是拿不到的。  
 *      2. 不要在{@link Activity#onCreate(Bundle)}时用当前Activity来获取导航栏高度（因为未Attached），可以用上一个Activity（如果能拿到的话）。  
 * 用途：主要是用于解决{@link DeviceInfo#getHeightOfNavigationBar(Context)}在不同设备下因兼容不到位使得获取的导航栏高度有问题，导致底部DialogFragment被导航栏遮挡的问题。  
 */  
public static int getSystemNavigationBarHeight(Activity activity) {  
    if (activity != null) {  
        Window window = activity.getWindow();  
        if (window != null) {  
            View view = window.getDecorView();  
            if (view != null) {  
                WindowInsets insets = view.getRootWindowInsets();  
                if (insets != null) {  
                    return DeviceInfo.isPortraitOrientation(activity) ? insets.getSystemWindowInsetBottom() : insets.getSystemWindowInsetRight();  
                }  
            }  
        }  
    }  
    // View没有Attach，用原来的方法兜底  
    return DeviceInfo.getHeightOfNavigationBar(activity);  
}
```


#### 样式设置
##### windowIsFloating
`<item name="android:windowIsFloating">true</item>`
true: 弹窗独立为一个窗口，弹窗会跟随软键盘上移; 弹窗会被设置成 WRAP_CONTENT
false: 非全屏情况下，导航栏覆盖在弹窗上面

具体原因可查看 PhoneWindow 中的 generateLayout ()方法
```cpp
protected ViewGroup generateLayout(DecorView decor) {
      //....
        mIsFloating = a.getBoolean(R.styleable.Window_windowIsFloating, false);
        int flagsToUpdate = (FLAG_LAYOUT_IN_SCREEN|FLAG_LAYOUT_INSET_DECOR)
                & (~getForcedWindowFlags());
        if (mIsFloating) {
            setLayout(WRAP_CONTENT, WRAP_CONTENT);
            setFlags(0, flagsToUpdate);
        } else {
            setFlags(FLAG_LAYOUT_IN_SCREEN|FLAG_LAYOUT_INSET_DECOR, flagsToUpdate);
        }
}
```



##### windowAnimationStyle
弹出动画
如果弹窗会有底部回弹效果，可能是设置了；不想有的话不要设置动画
`<item name="android:windowAnimationStyle">@style/FadePopWin</item>`

##### windowIsTranslucent
会让导航栏变透明，但按钮不会变透明
等同 `window.addFlags(WindowManager.LayoutParams.FLAG_TRANSLUCENT_STATUS); `

##### backgroundDimEnabled

默认 true，**背景渐变，会导致导航栏也一起变暗**




#### 几个例子


如果导航栏有颜色覆盖，可能是 Window 的背景，可以用这个来覆盖

```
<style name="TransparentBottomDialog" parent="ShareDialog">  
    <item name="android:windowBackground">@android:color/transparent</item>  
    <item name="android:backgroundDimEnabled">false</item>  
    <item name="android:windowIsTranslucent">true</item>  
</style>
```


底部弹窗-导航栏不变色
``` xml
<style name="BottomConfirmDialog" parent="@android:style/Theme.Dialog">  
    <item name="windowActionModeOverlay">true</item>  
    <item name="android:windowBackground">@color/transparent</item>  
    <item name="android:windowFrame">@null</item>  
    <item name="android:windowNoTitle">true</item>  
    <item name="android:windowIsTranslucent">false</item>  
    <item name="android:windowContentOverlay">@null</item>  
    <item name="android:background">@null</item>  
    <item name="android:backgroundDimEnabled">false</item>  
    <item name="android:windowAnimationStyle">@style/Popupwindow_Anim_login</item>  
    <item name="android:windowMinWidthMajor">100%</item>  
    <item name="android:windowMinWidthMinor">100%</item>  
</style>
```

导航栏变色
``` xml
<!-- 目前用于仿ios风格弹窗的样式 -->  
<style name="DrawerStyleDialog" parent="VerticalScreenDialogStyle">  
    <item name="android:windowAnimationStyle">@style/Popupwindow_Anim_login</item> <!-- 以200ms间隔自下往上匀速弹出 -->  
    <item name="android:windowBackground">@color/transparent</item> <!--设置窗口背景透明-->  
    <item name="android:backgroundDimEnabled">true</item> <!--启用背景暗淡效果-->  
    <item name="android:backgroundDimAmount">0.3</item> <!--暗淡效果程度 范围0.0~1.0 值越大越暗-->  
    <item name="android:windowFrame">@null</item>   <!--取消默认Dialog的边框-->  
    <item name="android:windowNoTitle">true</item>  <!--设置Dialog无标题-->  
    <item name="android:windowIsFloating">false</item>  <!-- 设置非悬浮window，不允许悬浮在其它窗口上方 -->  
    <item name="android:windowIsTranslucent">true</item>    <!-- 设置窗口半透明 -->  
    <item name="android:windowTranslucentStatus">true</item>    <!--设置状态栏透明-->  
    <item name="android:windowContentOverlay">@null</item>      <!--不设置drawable覆盖在窗口上-->  
    <item name="android:background">@android:color/transparent</item> <!--设置控件背景透明-->  
</style>
```





## 导航栏兼容

最主要让前端背景做纯色填充，即使是半屏，背景部分也超出 UI 高度；
客户端稍微调高弹窗高度，这样即使遮挡也不会挡住内容区域

方式一：弹窗全屏，页面自身兼容
方式二：弹窗半屏，页面全屏，超出 UI 高度部分纯色填充，弹窗高度加上导航栏高度，这样即使有小部分机型导航栏遮挡网页，也不会挡住内容区域

方式三、设置 style
不行 X
```java
<item name="android:windowTranslucentNavigation">false</item>
或者
根布局设置fitsystemwindow=true
fitsystemwindow会对上下左右都做padding，也就是说除了不会延申到导航栏之外，也不会延申到状态栏，如果美术要求状态栏沉浸式的话，fitsystemwindow就不适合（或者外面再包一层放背景）
```

方式四、动态调整布局
windowInsets
![](assets/Pasted%20image%2020240202151752.png)



导航栏颜色设置

```java
/**  
 * 设置导航栏颜色  
 */  
public static void setNavigationBarColor(Window window, @ColorInt int color) {  
    if (window == null || !navigationBarExist(window)) {  
        return;  
    }  
    if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.LOLLIPOP) {  
        window.addFlags(WindowManager.LayoutParams.FLAG_DRAWS_SYSTEM_BAR_BACKGROUNDS);  
        window.clearFlags(WindowManager.LayoutParams.FLAG_TRANSLUCENT_STATUS);  
        window.setNavigationBarColor(color);  
    }  
}
```

隐藏状态栏
```java
public static void hideStatusBar(@Nullable Dialog dialog) {  
    if (null == dialog || dialog.getWindow() == null) {  
        return;  
    }  
    WindowManager.LayoutParams wmlp = dialog.getWindow().getAttributes();  
    wmlp.systemUiVisibility = View.SYSTEM_UI_FLAG_FULLSCREEN;  
    dialog.getWindow().setAttributes(wmlp);  
}
```



## 软键盘遮挡弹窗

设置 SOFT_INPUT_ADJUST_RESIZE 以及 布局中要套一层 FrameLayout ，SOFT_INPUT_ADJUST_RESIZE 调整的 是最外层容器高度，所以只要最外层高度给够了就可以调整，比如 container 我给的高度是 match_parent，也可以给个尽可能大的值比如 500. dp 这样能确保 container 有足够高度容纳键盘与自己的布局，如果希望自己布局靠底部，子父容器就得有 `android:layout_gravity="bottom"`


``` xml
<?xml version="1.0" encoding="utf-8"?>  
<layout xmlns:android="http://schemas.android.com/apk/res/android"  
    xmlns:app="http://schemas.android.com/apk/res-auto">  
  
    <data>  
  
    </data>  
  
    <FrameLayout  
	    android:id="@+id/container"
        android:layout_width="match_parent"  
        android:layout_height="match_parent">  
  
        <com.netease.cc.widget.cornerclip.CornerClipConstraintLayout
	        android:id="@+id/container_input"
            android:layout_width="match_parent"  
            android:layout_height="68dp"  
            android:layout_gravity="bottom"  
            android:background="#161616"  
            android:orientation="horizontal">  
  
            <EditText  
                android:id="@+id/input_content"  
                android:inputType="number"  
                android:layout_width="0dp"  
                android:layout_height="wrap_content"  
                android:layout_marginStart="12dp"  
                android:layout_marginEnd="8dp"  
                android:paddingHorizontal="10dp"  
                android:paddingVertical="11dp"  
                android:scrollbars="none"  
                android:focusable="true"  
                android:background="@drawable/bg_0affffff_8r"  
                android:focusableInTouchMode="true"  
                android:textColor="#4dFFFFFF"  
                android:textSize="13sp"  
                android:maxLines="1"  
                android:text="1"  
                android:maxLength="4"  
                android:textCursorDrawable="@drawable/bg_6b53ff_1r"  
                android:hint="输入赠送的礼物数量，最多为1000个"  
                android:textColorHint="#4dFFFFFF"  
                app:layout_constraintStart_toStartOf="parent"  
                app:layout_constraintEnd_toStartOf="@id/sendBtn"  
                app:layout_constraintTop_toTopOf="parent"  
                app:layout_constraintBottom_toBottomOf="parent" />  
  
            <com.netease.cc.widget.cornerclip.CornerClipTextView  
                android:id="@+id/sendBtn"  
                android:layout_width="60dp"  
                android:layout_height="40dp"  
                android:layout_marginEnd="12dp"  
                app:allCornerRadius="8dp"  
                android:background="#6F57FF"  
                android:text="发送"  
                android:gravity="center"  
                android:textColor="@color/color_ffffff"  
                android:layout_gravity="center_vertical"  
                app:layout_constraintEnd_toEndOf="parent"  
                app:layout_constraintTop_toTopOf="parent"  
                app:layout_constraintBottom_toBottomOf="parent"/>  
  
        </com.netease.cc.widget.cornerclip.CornerClipConstraintLayout>  
    </FrameLayout>  
  
  
</layout>
```


``` java
setSoftInputMode(WindowManager.LayoutParams.SOFT_INPUT_ADJUST_RESIZE)
```

``` kotlin
class GiftShelfNumInputDialog(private val initText: String?,  
                              private val externalWatcher: TextWatcher): BaseRxDialogFragment() {  
  

  
    override fun onCreateDialog(savedInstanceState: Bundle?): Dialog {  
        return PluginFrameworkDialog.Builder()  
            .activity(activity)  
            .style(R.style.TransparentPopUpDownDialog)  
            .width(ViewGroup.LayoutParams.MATCH_PARENT)
            .height(800.dp)
            .build()  
    }  
  

    override fun onStart() {  
        super.onStart()  
        dialog?.window?.apply {  
            setSoftInputMode(WindowManager.LayoutParams.SOFT_INPUT_ADJUST_RESIZE)  
        }  
    }  
  

}
```