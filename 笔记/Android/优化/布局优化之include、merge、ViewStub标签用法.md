include用于嵌入布局，merge用于减少嵌入布局层级，ViewStub用于动态加载View


## include使用
它将一个布局文件嵌入到另一个布局文件中。
优点：提高代码的重用性，避免重复编写相同的布局。  
缺点：因为它是嵌入，所以会增加一层级，增加布局的层次深度。  

```xml
<androidx.constraintlayout.widget.ConstraintLayout xmlns:android="http://schemas.android.com/apk/res/android"  
    xmlns:app="http://schemas.android.com/apk/res-auto"  
    xmlns:tools="http://schemas.android.com/tools"  
    android:layout_height="wrap_content"  
    android:layout_width="wrap_content" >  
  
    <ImageView  
        android:id="@+id/iv_bg"  
        android:layout_width="261dp"  
        android:layout_height="60dp"  
        android:scaleType="centerCrop"  
        app:riv_corner_radius="6dp"/>  
  
</androidx.constraintlayout.widget.ConstraintLayout >

<include
    layout="@layout/layout_include" />
```


## merge
用于优化布局的层次结构。当多个视图组件需要放在同一个容器内时，可以用来减少冗余的视图层次。
优点：相对于include来讲，减少嵌入其他布局带来的层级。
缺点：不能单独使用，它必须嵌套在其他布局文件中；也无法在自身布局文件中给定尺寸。

如果布局宽高都固定，建议在merge里给个固定尺寸，如ImageView这里的设置，这样可以省去外部再设置尺寸。
```xml
<?xml version="1.0" encoding="utf-8"?>  
<merge xmlns:android="http://schemas.android.com/apk/res/android"  
    xmlns:app="http://schemas.android.com/apk/res-auto"  
    xmlns:tools="http://schemas.android.com/tools"  
    android:layout_height="wrap_content"  
    android:layout_width="wrap_content"  
    tools:parentTag="androidx.constraintlayout.widget.ConstraintLayout">  
  
    <ImageView  
        android:id="@+id/iv_bg"  
        android:layout_width="261dp"  
        android:layout_height="60dp"  
        android:scaleType="centerCrop"  
        app:riv_corner_radius="6dp"/>  
  
</merge>
```
在merge标签内的设置是不生效的，只能预览，所以一般要在引用的地方再次声明；
``` xml
<include
	android:layout_height="wrap_content"  
    android:layout_width="wrap_content"  
    layout="@layout/layout_merge" />
```


### 使用场景
如果View需要在多处复用时，一般会对其进行封装，这个时候就可以用merge将布局inflate到当前view中
```kotlin
class CustomMergeView : ConstraintLayout {  
  
    constructor(context: Context) : super(context)  
  
    constructor(context: Context, attrs: AttributeSet?) : super(context, attrs)  
  
    constructor(context: Context, attrs: AttributeSet?, defStyleAttr: Int) : super(context, attrs, defStyleAttr)  
  
    init {  
        val inflater = LayoutInflater.from(context)  
        inflater.inflate(R.layout.layout_merge, this, true)  
        initView()  
    }  
  
  
    fun initView() {  
	    // 可以直接用id设置
        btn_copy.setOnClickListener {  
            
        }  
        // 这个ViewGroup只作为布局使用，而非容器，所以不需要draw
        setWillNotDraw(true)  
    }  

	fun loadData() {
	
	}
  
}
```

嵌入布局
```xml
<CustomMergeView  
    android:layout_height="wrap_content"  
    android:layout_width="wrap_content"  
 />
```



## ViewStub
ViewStub是一个占位符，用于延迟加载布局。它在布局加载时不占用内存，直到 ViewStub 被调用才会加载和显示其内容。
优点：延迟加载减少了初始布局的复杂度，优化了启动时间和性能。  
缺点：动态加载需要规避多次inflate情况，可能会小小提升代码复杂度。
```xml
<!-- layout_stub.xml -->
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent">

    <ViewStub
        android:id="@+id/stub"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:layout="@layout/layout_stub" />

    <!-- 其他视图组件 -->
</LinearLayout>
```
在代码中动态加载 ViewStub：
``` java
ViewStub stub = findViewById(R.id.stub);
View inflated = stub.inflate(); // 动态加载布局
```


### 异步inflate ViewStub
甚至可以考虑异步inflate vs
``` kotlin
class AsyncViewStub(context: Context, val lifecycleOwner: LifecycleOwner) : FrameLayout(context) {  
  
    companion object {  
        const val TAG = "AsyncViewStub"  
    }  
  
    private val mInflater: LayoutInflater by lazy { LayoutInflater.from(context) }  
  
    var realView: View? = null  
        private set  
  
    fun isViewReady() = realView != null  
  
    /**  
     * 异步加载控件  
     * @param parent  
     * @param layoutRes  
     * @param inflateListener 布局加载完成后的回调  
     *  
     */    
     fun inflate(parent: ViewGroup?, layoutRes: Int, inflateListener: ((View?) -> Unit)?) {  
        // 异常处理  
        val handler = CoroutineExceptionHandler { _, exp ->  
            exp.printStackTrace()  
            inflateListener?.invoke(null)  
        }  
        lifecycleOwner.lifecycleScope.launch(handler) {  
            val view = withContext(Dispatchers.IO) { inflateViewNoAdd(parent, layoutRes) }  
            realView = view  
            if (view == null) {  
                throw IllegalStateException("AsyncViewStub inflate view is null")  
            } else {  
                inflateListener?.invoke(view)  
                Log.d(TAG, "inflate: AsyncViewStub inflate success")  
            }  
        }  
    }  
  
    private fun inflateViewNoAdd(parent: ViewGroup?, layoutRes: Int): View? {  
        return mInflater.inflate(layoutRes, parent, false)  
    }  
  
  
    private fun replaceWithView(view: View) {  
        // TODO 可以添加占位图  
        addView(view)  
    }  
  
  
}
```

以下是使用例子
```kotlin
class XXXView @JvmOverloads constructor(  
    context: Context,  
    attrs: AttributeSet? = null,  
    defStyleAttr: Int = 0  
) : LinearLayout(context, attrs, defStyleAttr), VisibleRule {  
  
    private var rootView: View? = null  
    private var asyncViewStub: AsyncViewStub? = null  

  
    private fun inflateView() {  
        if (this.asyncViewStub == null) {  
            this.asyncViewStub = AsyncViewStub(context, mFragment!!)  
        }  
        if (rootView == null) {  
            if (!isInflating) {  
                isInflating = true  
                this.asyncViewStub!!.inflate(this, R.layout.layout_video_friend_behavior) { view ->  
                    rootView = view  
                    removeAllViews()  
                    addView(rootView)  
                    bindViewData()  
                }  
            }  
        } else {  
            bindViewData()  
        }  
    }  
  
}
```