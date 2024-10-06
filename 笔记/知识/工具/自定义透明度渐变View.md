支持设置尾部渐变透明度

![|425](assets/Pasted%20image%2020240802114312.png)

```java
public class GradientTransparencyLayout extends FrameLayout {  
  
    private Paint mPaint;  
    private LinearGradient mGradient;  
    private float mStartPosition = 0.5f;  
    private float mEndPosition = 1.0f;  
    private int mStartColor = 0xFF000000;  
    private int mEndColor = 0x00000000;  
  
    public GradientTransparencyLayout(Context context) {  
        super(context);  
        init(null);  
    }  
  
    public GradientTransparencyLayout(Context context, @Nullable AttributeSet attrs) {  
        super(context, attrs);  
        init(attrs);  
    }  
  
    public GradientTransparencyLayout(Context context, @Nullable AttributeSet attrs, int defStyleAttr) {  
        super(context, attrs, defStyleAttr);  
        init(attrs);  
    }  
  
  
    private void init(@Nullable AttributeSet attrs) {  
        mPaint = new Paint();  
        setWillNotDraw(false);  
  
        if (attrs != null) {  
            TypedArray a = getContext().obtainStyledAttributes(attrs, R.styleable.GradientTransparencyLayout);  
            mStartPosition = a.getFloat(R.styleable.GradientTransparencyLayout_startPosition, mStartPosition);  
            mEndPosition = a.getFloat(R.styleable.GradientTransparencyLayout_endPosition, mEndPosition);  
            mStartColor = a.getColor(R.styleable.GradientTransparencyLayout_startColor, mStartColor);  
            mEndColor = a.getColor(R.styleable.GradientTransparencyLayout_endColor, mEndColor);  
            a.recycle();  
        }  
    }  
  
  
    @Override  
    protected void onSizeChanged(int w, int h, int oldw, int oldh) {  
        super.onSizeChanged(w, h, oldw, oldh);  
        updateGradient();  
    }  
  
    private void updateGradient() {  
        int width = getWidth();  
        mGradient = new LinearGradient(  
            width * mStartPosition, 0,  
            width * mEndPosition, 0,  
            mStartColor, mEndColor,  
            Shader.TileMode.CLAMP  
        );  
    }  
  
  
    @Override  
    protected void dispatchDraw(Canvas canvas) {  
        // 创建离屏缓冲  
        int saveCount = canvas.saveLayer(0, 0, getWidth(), getHeight(), null);  
  
        // 绘制子视图  
        super.dispatchDraw(canvas);  
  
        // 设置渐变和混合模式  
        mPaint.setShader(mGradient);  
        mPaint.setXfermode(new PorterDuffXfermode(PorterDuff.Mode.DST_IN));  
  
        // 绘制渐变遮罩  
        canvas.drawRect(0, 0, getWidth(), getHeight(), mPaint);  
  
        // 恢复画布  
        canvas.restoreToCount(saveCount);  
        mPaint.setXfermode(null);  
    }  
}
```


```xml
<declare-styleable name="GradientTransparencyLayout">  
    <attr name="startPosition" format="float" />  
    <attr name="endPosition" format="float" />  
    <attr name="startColor" format="color" />  
    <attr name="endColor" format="color" />  
</declare-styleable>
```


```xml
<com.netease.cc.widget.widget.GradientTransparencyLayout  
    android:id="@+id/layout_strip_room_tab"  
    android:layout_width="0dp"  
    android:layout_height="wrap_content"  
    android:layout_marginBottom="4dp"  
    app:startPosition="0.85"  
    app:endPosition="1"  
    app:startColor="#FF000000"  
    app:endColor="#00000000"  
    app:layout_constraintStart_toStartOf="parent"  
    app:layout_constraintEnd_toStartOf="@id/layout_switch_room">  
  
    <com.netease.cc.cui.slidingbar.CSlidingTabStrip  
        android:id="@+id/strip_room_tab"  
        android:layout_width="match_parent"  
        android:layout_height="@dimen/ccgroomsdk__navigation_bar_height"  
        android:background="@color/transparent"  
        app:c_slidingTabStrip_edgeGradient="true"  
        app:c_slidingTabStrip_gradientLeftWidth="0dp"  
        app:c_slidingTabStrip_gradientRightWidth="0dp"  
        app:c_slidingTabStrip_gradientWidth="0dp"  
        app:c_slidingTabStrip_fixed_height="true"  
        app:c_slidingTabStrip_showIndicator="true"  
        app:c_slidingTabStrip_tabAlignment="start"  
        app:c_slidingTabStrip_tabPaddingTop="4dp"  
        app:c_slidingTabStrip_tabPaddingBottom="4dp"  
        app:c_slidingTabStrip_tabFirstPaddingLeft="16dp"  
        app:c_slidingTabStrip_tabPaddingLeftRight="10dp"/>  
  
</com.netease.cc.widget.widget.GradientTransparencyLayout>
```
