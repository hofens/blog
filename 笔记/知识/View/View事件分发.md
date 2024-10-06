
View 的事件分发机制是 Android 中非常核心的一个概念，它负责处理触摸事件（如按下、移动、抬起等）的分发过程。事件分发主要涉及三个方法：`dispatchTouchEvent()`, `onInterceptTouchEvent()`, 和 `onTouchEvent()`。

```java
public boolean dispatchTouchEvent(MotionEvent ev){
    boolean result = false;
    if(onInterceptTouchEvent(ev)){
        result = onTouchEvent(ev);
    }else{
        result = child.dispatchTouchEvent(ev);
    }
    return result;

}

```


### dispartchTouchEvent()

当一个触摸事件发生时，这个事件首先会被传递到顶层的 ViewGroup（比如一个 Activity 的根布局），然后通过 `dispatchTouchEvent()` 方法来决定如何分发这个事件。


