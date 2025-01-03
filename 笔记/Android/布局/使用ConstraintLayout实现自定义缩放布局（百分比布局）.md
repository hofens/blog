


一个复杂的布局或自定义View如何在添加到其他不同大小的ViewGroup中按比例去缩放自己的布局内容呢？我尝试使用ConstraintLayout解决了这个问题。

### 1. 简单的布局
大家先看一个简单的布局，由上下两个view组成，都是16:9的比例。![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/5b86f0457c419e3358c3bac57c91eb05.png)



左边是设置 `android:layout_width="match_parent"` 的情况，右边是将layout_width设为了200dp，模拟缩小到宽为200dp的View。大家可以发现的是他们实现等比缩小了。实现原理也很简单，就是通过ConstraintLayout将View按照比例去控制大小就好。
核心代码就是下面几行，设置width、height为0dp，app:layout_constraintDimensionRatio设置比例
```xml
  <ImageView
  		...
        android:layout_width="0dp"
        android:layout_height="0dp"
        app:layout_constraintDimensionRatio="16:9"
        ...
 />
```
完整代码：
```xml
<?xml version="1.0" encoding="utf-8"?>
<androidx.constraintlayout.widget.ConstraintLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="wrap_content"
    tools:context=".ScaleLayout">


    <ImageView
        android:id="@+id/imageView1"
        android:layout_width="0dp"
        android:layout_height="0dp"
        android:background="#f5f5f5"
        app:layout_constraintDimensionRatio="16:9"
        app:layout_constraintEnd_toEndOf="parent"
        app:layout_constraintStart_toStartOf="parent"
        app:layout_constraintTop_toTopOf="parent"
        tools:srcCompat="@tools:sample/avatars" />


    <ImageView
        android:id="@+id/imageView2"
        android:layout_width="0dp"
        android:layout_height="0dp"
        android:background="#f0eaff"
        app:layout_constraintDimensionRatio="16:9"
        app:layout_constraintEnd_toEndOf="parent"
        app:layout_constraintStart_toStartOf="parent"
        app:layout_constraintTop_toBottomOf="@+id/imageView1"
        tools:srcCompat="@tools:sample/avatars" />
    
</androidx.constraintlayout.widget.ConstraintLayout>

```


### 2. 稍微复杂点的布局
刚刚很简单的就做到了两个ImageView随viewgroup进行缩放，但需要满足其中一个条件不知道大家发现没有，width或者height需要有一个 `layout_width="0dp"`，那如果我们的子view不需要match_parent呢？比如需要在布局中间位置加一个40\*40大小的头像，总不能也设置成 `layout_width="0dp"` 吧。这时候就需要guideline的帮助了。先给大家看下效果。
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/aef039ecd5523bf85b65b9d19fe7d435.png)

可以看到底下的的布局照常保持16:9，头像部分也按比例进行了缩小。**核心思想就是用guideline去限制View的大小，可以将guideline理解成想parent的边一样，那只要用三条guideline，把View的top、right、left各自约束到guideline就行了**，记得guideline的约束要用百分比哦。


```xml
<?xml version="1.0" encoding="utf-8"?>
<androidx.constraintlayout.widget.ConstraintLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="wrap_content"
    tools:context=".ScaleLayout">


    <androidx.constraintlayout.widget.ConstraintLayout
        android:id="@+id/layout_img"
        android:layout_width="0dp"
        android:layout_height="0dp"
        app:layout_constraintDimensionRatio="16:9"
        android:background="#f5f5f5"
        app:layout_constraintEnd_toEndOf="parent"
        app:layout_constraintStart_toStartOf="parent"
        app:layout_constraintTop_toTopOf="parent">


        <androidx.constraintlayout.widget.Guideline
            android:id="@+id/guideline1"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:orientation="vertical"
            app:layout_constraintGuide_percent="0.3" />

        <androidx.constraintlayout.widget.Guideline
            android:id="@+id/guideline2"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:orientation="vertical"
            app:layout_constraintGuide_percent="0.7" />

        <androidx.constraintlayout.widget.Guideline
            android:id="@+id/guideline3"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:orientation="horizontal"
            app:layout_constraintGuide_percent="0.2" />

        <ImageView
            android:id="@+id/imageView1"
            android:layout_width="0dp"
            android:layout_height="0dp"
            app:layout_constraintDimensionRatio="1:1"
            app:layout_constraintEnd_toStartOf="@+id/guideline2"
            app:layout_constraintStart_toStartOf="@+id/guideline1"
            app:layout_constraintTop_toTopOf="@+id/guideline3"
            tools:srcCompat="@tools:sample/avatars" />
    </androidx.constraintlayout.widget.ConstraintLayout>


    <ImageView
        android:id="@+id/imageView2"
        android:layout_width="0dp"
        android:layout_height="0dp"
        android:background="#f0eaff"
        app:layout_constraintDimensionRatio="h,16:9"
        app:layout_constraintEnd_toEndOf="parent"
        app:layout_constraintStart_toStartOf="parent"
        app:layout_constraintTop_toBottomOf="@+id/layout_img" />

</androidx.constraintlayout.widget.ConstraintLayout>
```

如果大家有更好的方法，欢迎指教。



