
> LiveData 数据持有类。有如下特性点：
> 
	1、数据可以被观察者订阅
	2、能够感知实现 LifecycleOwner 接口组件的生命周期 (如 Fragment、Activity、Service)，减少内存泄漏。
	3、可以设定组件处于激活状态（STARTED、RESUMED）才会通知观察者有数据更新，且组件前后台切换后能自动从最新数据开始刷新
	4、粘性
	5、生命周期发生变化，是会监听到最新值的


# 消费
1.observe (LifecycleOwner owner, Observer \<? super T> observer)
注册具有生命周期监听的 Observer，它可以安全地观察 LiveData 而不用担心泄漏，它的生命周期特性表现在

仅当 owner 处于 `Lifecycle. StateSTARTED` 或 `Lifecycle. StateRESUMED` 状态时，观察者才会收到事件。
如果 owner 处于 Lifecycle. StateDESTROYED 状态，则 observer 将被自动删除。
在 owner 未激活的情况下更改数据时，它将不会收到任何更新。如果再次激活，它将自动接收最新的可用数据。(如 Activity 在 `onPause 状态无法接收数据`，变换为 onRresume 状态时可以接收数据)
如果 Observer 已注册过，但是在次注册时前后的 owner 不一致，则会抛 IllegalArgumentException

2.observeForever (Observer\<? super T> observer)

永久性注册方法，其特性为：

将给定的观察者添加到观察者列表中。此调用类似于带有生命周期所有者的{@link LiveData observe（LifecycleOwner，Observer）}，该生命周期所有者始终处于活动状态。
给定的观察者将一直接收所有事件，并且永远不会被自动删除。除非 `手动调用 removeObserver (Observer）方法停止`
当 LiveData 具有此类观察者之一，则它将被视为 active
如果在此之前已将观察者和所有者绑定的 ObserverWrapper 添加进入 LiveData，此次在通过 observeForever (Observer)方式注册观察者则抛 IllegalArgumentException。


3.removeObserver (Observer\<? super T> observer)

从观察者列表中删除给定的观察者，该方法应该与 observeForever (Observer\<? super T> observer)方法成对出现，它一般在 onCreate 和 onDestory 中调用


>如果需要在宿主 onPause 的时候也要监听到数据变化，需要用 observeForever 和 removeObserver



# 生产

数据只会在主线程更新

**postValue**: 将任务发布到主线程以设置给定值。可以在非 UI 线程中执行。如果在主线程执行发布的任务之前多次调用此方法，则只会分派最后一个值。

**setValue**: 修改需要发射分发的值，每个值都有对应 Version 作为唯一 id,此处 mVersion++后进行值的发布。该方法只能运行在主线程中。

```java
如果先后调用

1. liveData.postValue(“a”);
2. liveData.setValue(“b”)；

首先将设置值“ b”，然后主线程将使用值“ a”覆盖它。
```


#### considerNotify (ObserverWrapper observer)
执行数据更新操作，通过多条件判断最终执行了我们熟悉的 Observer.onChanged (T)。
```java
    private void considerNotify(ObserverWrapper observer) {
        //确保ObserverWrapper是mActive=ture才能继续执行
        if (!observer.mActive) {
            return;
        }
        //确保当前生命周期处于允许状态，可继续执行后续代码
        if (!observer.shouldBeActive()) {
            observer.activeStateChanged(false);
            return;
        }

        //数据未发射处理过方可进行后续操作
        if (observer.mLastVersion >= mVersion) {
            return;
        }
        observer.mLastVersion = mVersion;
        //进行数据刷新
        observer.mObserver.onChanged((T) mData);
    }

```

被观察者的每次数据变动都会有一个 id 标识 mVersion 会自增。每个观察者在监听到数据变动进行处理之前都会进行判断，当 mLastVersion>=mVersion 时，即此消息已经响应过了，不再进行响应。

如果符合条件会先赋值 observer. mLastVersion = mVersion，确保 `同一个消息变动不会多次处理`。

#### dispatchingValue (@Nullable ObserverWrapper initiator)

数据变更后的分发发射方法。当某一个 ObserverWrapper 第一次在 LiveData 中注册时，会自动发布最新的数据。所以要注意在 Observer.onChanged (T)中做好 NullPointerException 预防



```java
    @SuppressWarnings("WeakerAccess") /* synthetic access */
    void dispatchingValue(@Nullable ObserverWrapper initiator) {

        ......
        if (initiator != null) {
            considerNotify(initiator);
            initiator = null;
        } else {
            for (Iterator<Map.Entry<Observer<? super T>, ObserverWrapper>> iterator =
                 mObservers.iteratorWithAdditions(); iterator.hasNext(); ) {
                considerNotify(iterator.next().getValue());
                if (mDispatchInvalidated) {
                    break;
                }
            }
        }
        ......

    }

```