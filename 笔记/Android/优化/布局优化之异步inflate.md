
异步inflate即是将view的inflate过程放到其他线程中执行，不阻塞主线程；基于此还可以考虑提前异步inflate布局，将view放到缓存中，等待需要的时候从缓存中获取。

### 多线程执行
系统默认的AsyncLayoutInflater是单线程执行的，可以改写它为多线程执行。基于官方AsyncLayoutInflater，扩展了线程池并行处理多个请求，提高了处理效率，也可以避免单一线程被阻塞时提交不进去。
```kotlin
class InflateThread : Thread() {  
	private val mQueue = LinkedBlockingQueue<InflateRequest>(50)  
	private val mRequestPool = Pools.SimplePool<InflateRequest>(50)  
	private val executorService by lazy { ThreadPoolExecutor(0, 50, 0L, TimeUnit.MILLISECONDS, LinkedBlockingQueue<Runnable>()) }  

	// Extracted to its own method to ensure locals have a constrained liveness  
	// scope by the GC. This is needed to avoid keeping previous request references        // alive for an indeterminate amount of time, see b/33158143 for details        fun runInner() {  
		val request: InflateRequest  
		try {  
			request = mQueue.take()  
		} catch (ex: InterruptedException) {  
			// Odd, just continue  
			Log.w(TAG, ex)  
			return  
		}  
		executorService.submit {  
			try {  
				request.view = request.inflater!!.mInflater.inflate(  
					request.resid, request.parent, false  
				)  
			} catch (ex: RuntimeException) {  
				// Probably a Looper failure, retry on the UI thread  
				Log.w(  
					TAG, "Failed to inflate resource in the background! Retrying on the UI"  
							+ " thread", ex  
				)  
			}  
			Message.obtain(request.inflater!!.mHandler, 0, request)  
				.sendToTarget()  
		}  
	}  

	override fun run() {  
		while (true) {  
			runInner()  
		}  
	}  

	fun obtainRequest(): InflateRequest {  
		var obj = mRequestPool.acquire()  
		if (obj == null) {  
			obj = InflateRequest()  
		}  
		return obj  
	}  

	fun releaseRequest(obj: InflateRequest) {  
		obj.callback = null  
		obj.inflater = null  
		obj.parent = null  
		obj.resid = 0  
		obj.view = null  
		mRequestPool.release(obj)  
	}  

	fun enqueue(request: InflateRequest) {  
		try {  
			mQueue.put(request)  
		} catch (e: InterruptedException) {  
			throw RuntimeException(  
				"Failed to enqueue async inflate request", e  
			)  
		}  
	}  

	companion object {  
		val instance: InflateThread = InflateThread()  

		init {  
			instance.start()  
		}  
	}  
}  
}
```

### 缓存布局
将异步inflate创建的view放入缓存中
``` kotlin
private class ViewCache {  
    private val mViewPools = SparseArray<LinkedList<SoftReference<View?>>>()  
  
    fun getViewPool(layoutId: Int): LinkedList<SoftReference<View?>> {  
        var views = mViewPools[layoutId]  
        if (views == null) {  
            views = LinkedList()  
            mViewPools.put(layoutId, views)  
        }  
        return views  
    }  
  
    fun getViewPoolAvailableCount(layoutId: Int): Int {  
        val views = getViewPool(layoutId)  
        val it = views.iterator()  
        var count = 0  
        while (it.hasNext()) {  
            if (it.next().get() != null) {  
                count++  
            } else {  
                it.remove()  
            }  
        }  
        return count  
    }  
  
    fun putView(layoutId: Int, view: View?) {  
        if (view == null) {  
            return  
        }  
        getViewPool(layoutId).offer(SoftReference(view))  
    }  
  
    fun getView(layoutId: Int): View? {  
        return getViewFromPool(getViewPool(layoutId))  
    }  
  
    fun getViewFromPool(views: LinkedList<SoftReference<View?>>): View? {  
        if (views.isEmpty()) {  
            return null  
        }  
        val target = views.pop().get() ?: return getViewFromPool(views)  
        return target  
    }  
}
```



```kotlin
fun clear() {  
        this.listener = null  
        refreshDisposable?.dispose()  
        refreshHelper.release()  
    }  
  
    fun getView(parent: ViewGroup, layoutId: Int): View {  
        val view: View? = viewCache.getView(layoutId)  
        if (view != null) {  
            preload(parent, layoutId, 0)  
//            LogUtil.d(TAG, "getView: view from cache, layoutId = $layoutId")  
            return view  
        }  
//        LogUtil.d(TAG, "getView: view from inflate, layoutId = $layoutId")  
        return LayoutInflater.from(parent.context).inflate(layoutId, parent, false)  
    }  
  
    private fun preload(parent: ViewGroup, @LayoutRes layoutResId: Int, forcePreCount: Int) {  
        val viewsAvailableCount = viewCache.getViewPoolAvailableCount(layoutResId)  
        if (viewsAvailableCount >= maxCount) {  
            return  
        }  
        var needPreloadCount: Int = maxCount - viewsAvailableCount  
        if (forcePreCount > 0) {  
            needPreloadCount = Math.min(forcePreCount, needPreloadCount)  
        }  
        for (i in 0 until needPreloadCount) {  
            // 异步加载View  
            if (mInflater == null) {  
                val context = parent.context  
                mInflater = AsyncLayoutInflater(context)  
            }  
            mInflater!!.inflate(layoutResId, parent) { view, layoutId, parent ->  
                viewCache.putView(layoutId, view)  
            }  
        }  
    }
```

### 优化效果
在IM场景，onCreateViewHolder耗时可以从680ms降低到37ms




