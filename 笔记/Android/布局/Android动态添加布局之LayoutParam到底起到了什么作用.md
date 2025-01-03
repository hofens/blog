

> 在需求开发过程中，可能会遇到需要动态添加View的场景，那到底如何去添加View以及怎样像在XML中写布局一样指定View摆放的位置呢。在初学时也去翻阅了很多文章，但发现都没讲清楚怎么去控制View的位置这件事，所以这篇文章侧重详细解释下LayoutParam的作用。


### 一、静态布局我们是怎么写的
首先知道一点，安卓的布局是通过父ViewGroup里添加子View或者子ViewGroup实现的，那对于父容器来讲，我得知道子View的大小、位置才能准确绘制View的位置对吧，因此最基础的信息就是子View大小、位置信息。
先来看这样一段代码，ConstraintLayout里包裹着LinearLayout，LinearLayout里包裹着ImageView。

我们先看ConstraintLayout和LinearLayout，他们之间的关系就是父容器与子布局的关系，对于子容器LinearLayout来讲，它告诉爸爸我的宽度和你一样，我的高度由我的儿子们决定，这是LinearLayout的大小信息；位置信息呢则是 `app:layout_constraintTop_toTopOf="parent"`，也就是顶部和你顶部对齐，这就是LinearLayout的位置信息。因此就可以绘制出LinearLayout的位置了。

再来，先忽略爷爷级ConstraintLayout，只看LinearLayout和ImageView这一块。对于ImageView来说LinearLayout就是它的父容器，所以ImageView也必须告诉他爸爸大小、位置。这时候有人就问了，你这只有ImageView的大小，没有位置呀？这是因为LinearLayout只能在水平或者垂直方向上添加View，你不可能有多个View重叠的情况，也就是说你的位置不用告诉LinearLayout也知道。
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/a1ad91e0e2553cb9bd1ea2f2d55b90b2.png)
而如果我们把LinearLayout换成ConstraintLayout呢，首先你可以看到ImageView多了topTotop和startTostart，也就是至少得把我的坐标告诉ConstraintLayout才能让其知道我在哪。
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/f0de831f7535672eaf37aa8dfb7ac3c5.png)
所以从这个例子来看，我们知道子View要提供什么信息是要看父容器需要什么信息，毕竟子View是要交给父容器去绘制的。ConstraintLayout因为支持子View摆放在任何位置，相当于子View是相对parent或者其他View进行偏移，所以你得告知ConstraintLayout相对谁、偏移多少。而对于LinearLayout来讲，它的子View只有可能在水平或者垂直方向上添加，你从没见过LinearLayout中有View相互重叠这种情况吧。


### 二、动态布局该如何添加View及其位置、大小
#### LinearLayout中添加子View
上面的布局改一改，首先模拟下在parent_linear里动态添加ImageView的场景。要在XML中添加一个ImageView且距离左边30dp，距离上边20dp，得像这样写。
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/36bb51fbdc3effc26fa1dc96a177a2be.png)
动态添加就得这样写。
```java
@Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_layout);

        // 先找到要添加ImageView的爸爸容器
        LinearLayout parentLinear = findViewById(R.id.parent_linear);
        // new一个ImageView
        ImageView imageView = new ImageView(this);
        imageView.setImageDrawable(getDrawable(R.mipmap.ic_launcher));
        // 因为子View要添加到LinearLayout中，所以是new 一个LinearLayout.LayoutParams，设定子view的大小为50,50
        LinearLayout.LayoutParams linearParam = new LinearLayout.LayoutParams(dpToPx(50), dpToPx(50));
        // 设定边距
        linearParam.leftMargin = dpToPx(30);
        linearParam.topMargin = dpToPx(20);
        // 相当于把ImageView以linearParam参数添加到LinearLayout中
        parentLinear.addView(imageView, linearParam);
    }

    private int dpToPx(int dp){
        float scale=getResources().getDisplayMetrics().density;
        return (int)(dp*scale+0.5f);
    }

```
我想大家比较疑惑的点应该在 `LinearLayout.LayoutParams linearParam = new LinearLayout.LayoutParams(dpToPx(50), dpToPx(50))` 这里，我自己刚学的时候也很迷惑，`LinearLayout.LayoutParams` 容易误解成是父容器的参数，和我ImageView有什么关系呢？但其实要理解成 **我的View要动态添加到哪种容器里，就得用哪种 `ViewGroup.LayoutParams`。** 如果你爸爸是ConstraintLayout那就用 `ConstraintLayout.LayoutParams`，记住这个就好。
那parent_linear也就是我们动态添加ImageView的爸爸的这个LinearLayout，它也有相对于它爸爸ConstraintLayout的布局参数吧？有的！

```java
ConstraintLayout.LayoutParams parentParam = (ConstraintLayout.LayoutParams) parentLinear.getLayoutParams();
```
看见了吧，它的LayoutParams 是ConstraintLayout的，因为parentLinear是添加在ConstraintLayout中的。同时因为parentLinear是我们写在静态布局里也就是XML文件里的，所以它已经有LayoutParams了，直接通过 `getLayoutParams()` 获取到。


#### ConstraintLayout中添加子View
直接来个难的，parent_constrain相对于它的爸爸向下偏移40dp，内部动态添加ImageView下偏10dp，左偏100dp。
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/4e9e8848af98aee57b44da6a95db6020.png)
效果如上图，我把ImageView删掉，parent_constrain的marginTop也删掉。然后再动态修改parent_constrain的marginTop以及添加ImageView。
```java
<androidx.constraintlayout.widget.ConstraintLayout
    xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    tools:context=".LayoutActivity">

    <androidx.constraintlayout.widget.ConstraintLayout
        android:id="@+id/parent_constrain"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        app:layout_constraintTop_toTopOf="parent">

    </androidx.constraintlayout.widget.ConstraintLayout>

</androidx.constraintlayout.widget.ConstraintLayout>
```
动态添加代码如下
```java
/* ---- 先在parent_constrain添加下ImageView */
        // 1.先找到要添加ImageView的爸爸容器
        ConstraintLayout parentConstraintLayout = findViewById(R.id.parent_constrain);
        // 2.new一个ImageView
        ImageView imageView = new ImageView(this);
        imageView.setImageDrawable(getDrawable(R.mipmap.ic_launcher));
        // 3.因为子View要添加到ConstraintLayout中，所以是new ConstraintLayout.LayoutParams，设定子view的大小为50,50
        ConstraintLayout.LayoutParams param = new ConstraintLayout.LayoutParams(dpToPx(50), dpToPx(50));
        // 4.设定边距
        param.leftMargin = dpToPx(100);
        param.topMargin = dpToPx(10);
        // 5.因为爸爸是ConstraintLayout噢，所以还得加上相对爸爸
        param.topToTop = ConstraintLayout.LayoutParams.PARENT_ID;
        param.startToStart = ConstraintLayout.LayoutParams.PARENT_ID;
        // 6.相当于把ImageView以param参数添加到ConstraintLayout中
        parentConstraintLayout.addView(imageView, param);

        /* ---- 别忘了还要把爸爸往下偏移40dp */
        // 因为parentConstraintLayout的爸爸也是ConstraintLayout，所以getLayoutParams()获取的类型是ConstraintLayout.LayoutParams
        ConstraintLayout.LayoutParams layoutParams = (ConstraintLayout.LayoutParams) parentConstraintLayout.getLayoutParams();
        // 设置偏移
        layoutParams.topMargin = dpToPx(40);
        // 重新把参数设置给parentConstraintLayout
        parentConstraintLayout.setLayoutParams(layoutParams);
```

可以看到ImageView是动态添加的，parent_constrain的marginTop也动态的加上了
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/bb970519801923b319d17911c68456b1.png)


### 总结
总结来讲，你的View要动态添加到哪个布局里，就new对应的LayoutParam，如果是已经存在的view就getLayoutParam ()。不同布局有不同的参数，你在xml里是怎么写的，动态就怎么写。也要注意不同布局的特性，如子view是否在同一层级，是否支持子View间重叠。




