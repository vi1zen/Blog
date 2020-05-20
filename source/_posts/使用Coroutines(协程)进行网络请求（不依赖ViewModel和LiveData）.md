# 使用Coroutines(协程)进行网络请求（不依赖ViewModel和LiveData）

---
title: 使用Coroutines(协程)进行网络请求（不依赖ViewModel和LiveData）
date: 2020-05-20 11:16
tags: 

 - Android
---

   目前大多数使用协程进行网络请求都需要依赖ViewModel和LiveData，但是这样会引入新的ViewModel类，使用起来总是不够简便，能不能像普通的网络请求一样直接进行接口请求，数据处理呢？答案肯定是可行的。

首先需要添加依赖

```groovy
implementation 'androidx.lifecycle:lifecycle-runtime-ktx:2.2.0'
```
   由于本文是基于Coroutines+retrofit+okhttp等框架进行网络请求的，网络相关的依赖如下
```groovy
implementation 'com.squareup.okhttp3:okhttp:4.2.1'
implementation 'com.squareup.retrofit2:retrofit:2.6.2'
```
   需要注意的是retrofit的版本需要2.6.0以上，不然使用方式会不一样

<!-- more -->

其他代码由于与普通的网络请求写法一致，我这里不再赘述，区别在于需要用到LifecycleScope，关于LifecycleScope的详细介绍可以去[官网](https://developer.android.com/topic/libraries/architecture/coroutines)查看，简单描述就是带有生命周期的CoroutineScope,它可以在界面销毁的时候自动取消协程，防止内存泄漏。

创建网络请求工具类：

```kotlin
object HttpUtil{
    fun <T> request(lifecycleScope:LifecycleCoroutineScope,
                    requestBlock: suspend () -> T,
                    onSuccess:(result:T) -> Unit,
                    onFailure:(e:Exception) -> Unit
                    = { ToastUtils.showShort(it.message) }){
                lifecycleScope.launch {
                    try {
                        val result = requestBlock.invoke()
                        val exception = ServiceErrorHandler.getServiceError(result)
                        if(exception != null){
                            onFailure.invoke(exception)
                        }else{
                            onSuccess.invoke(result)
                        }
                    } catch (e: Exception) {
                        ServiceErrorHandler.handlerServiceError(e,onFailure)
                    }
                }
            }
}

```

使用方法如下：

```kotlin
HttpUtil.request(lifecycleScope,{ apiService.getUserInfo(1) },
                onSuccess = { },
                onFailure = { })
```

以上基本能满足使用了，但是每次都需要传一个lifecycleScope感觉不是那么优雅，这里使用kotlin的扩展方法处理一下

```kotlin
fun <T> LifecycleCoroutineScope.request(requestBlock: suspend () -> T,
                                        onSuccess:(result:T) -> Unit,
                                        onFailure:(e:Exception) -> Unit
                                        = { ToastUtils.showShort(it.errorMessage) }){
    launch {
        try {
            val result = requestBlock.invoke()
            val exception = ServiceErrorHandler.getServiceError(result)
            if(exception != null){
                onFailure.invoke(exception)
            }else{
                onSuccess.invoke(result)
            }
        } catch (e: Exception) {
            ServiceErrorHandler.handlerServiceError(e,onFailure)
        }
    }
}
```

使用方法如下：

```kotlin
lifecycleScope.request({ apiService.getUserInfo(1) } ,
	onSuccess = {
    	showToast(it.data.toString())
    })
```

以上就是不依赖ViewModel和LiveData使用协程进行请求的全部内容，最后附上使用ViewModel和LiveData进行网络请求的代码

ViewModel定义如下：

```kotlin
class BaseViewModel : ViewModel(){

    protected val apiService: ApiService by lazy { ApiHelper.getInstance().apiService }

    protected fun <T> request(block: suspend () -> T): LiveData<Result<T>> = liveData {
        try {
            emit(Result.success(block.invoke()))
        } catch (e: Exception) {
            emit(Result.failure<T>(e))
        }
    }

}
```

```kotlin
class DemoViewModel : BaseViewModel(){
    fun getData() = request { apiService.getUserInfo(1) }
}
```

LiveData扩展方法：

```kotlin
fun <T> LiveData<Result<T>>.observer(owner: LifecycleOwner, 
    onSuccess:(t:T) -> Unit, 
    onFailure:(requestException: RequestException) -> Unit){
    this.observe(owner, Observer {
        it.onSuccess { t ->
            val exception = ServiceErrorHandler.getServiceError(t)
            if(exception != null){
                onFailure.invoke(exception)
            }else{
                onSuccess.invoke(t)
            }
        }.onFailure { exception ->
            ServiceErrorHandler.handlerServiceError(exception,onFailure)
        }
    })
}
```
使用方法如下：

```kotlin
class DemoActivity : BaseActivity(){
    private val model by lazy { ViewModelProvider(this)[DemoViewModel::class.java] }
	......
    fun getData(){
    	model.getData().observer(this, onSuccess = {}, onFailure = {})
    }
}
```