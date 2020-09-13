---
title: 使用IntentFlag来启动Activity
date: 2020-07-22 19:05
tags: 
categories: Android
thumbnail: /thumbnail/sebastian-pichler-20070-unsplash.jpg
---

虽然有`Android`有四大启动模式，但是实际上开发中，如果仅仅依靠在`xml`指定`launchMode`的形式来启动`Activity`，不太够用并且不太灵活。所以这篇文章探究下在一些常见的场景如何仅仅利用`IntentFlag`的形式来启动`Activity`。

<!-- more -->

### 场景一:在Service中启动Activity

我们在`MainActivity`中放置一个`Button`，点击`Button`会开启一个`Service`，在`Service`的`onStartCommand`中，我们反过来启动`MainActivity`。

`MainActivity`的代码：

```kotlin
override fun onCreate(savedInstanceState: Bundle?) {
    super.onCreate(savedInstanceState)
    setContentView(R.layout.activity_main)
    
    bt_main_activity.setOnClickListener {
        startService(Intent(this,MyService::class.java))
    }
}
```

`MyService`中的代码：

```kotlin
   override fun onStartCommand(intent: Intent?, flags: Int, startId: Int): Int {
        startActivity(Intent(this,MainActivity::class.java))
        return super.onStartCommand(intent, flags, startId)
    }
```

接着我们启动`App`，点击`MainActivity`中的`Button`，猜怎么着？在我的`Android 7.0`设备上正常启动了`MainActivity`，而在我的模拟器上（`Andorid 10`）上面却崩溃了。错误如下:

> Caused by: android.util.AndroidRuntimeException: Calling startActivity() from outside of an Activity  context requires the FLAG_ACTIVITY_NEW_TASK flag. Is this really what you want?

查阅官方文档：

![](https://i.loli.net/2020/07/22/Wu2VXIFRPilDUoN.png)也就是说在`Android 7.0`以下或者`Android 9`以上`Service`或者在`Application`中启动`Activity`，必须加上`FLAG_ACTIVITY_NEW_TASK`这个`Flag`；而因为一个`bug`，这个要求在`Android 7.0`中不是强制的。无论如何我们在`Service`中加上这个`Flag`就可以了。

`Service`中的代码变成了这样：

```kotlin
override fun onStartCommand(intent: Intent?, flags: Int, startId: Int): Int {
    startActivity(Intent(this,MainActivity::class.java)
        .apply {
            addFlags(Intent.FLAG_ACTIVITY_NEW_TASK)
        })
    return super.onStartCommand(intent, flags, startId)
}
```

修改之后，两个设备都正常启动了`MainActivity`，此时发现任务栈中存在了2个`MainActivity`。

其实这里比较奇怪，按照书上说的，`FLAG_ACTIVITY_NEW_TASK`等于`launchMode`中指定`singleTask`，也就是栈内复用模式。可实际上还是启动了2个`MainActivity`。我以为是在`Service`中启动的问题，可实际上就算是在`MainActivity`中启动，也还是会启动两个`MainActivity`。所以说还是不能全信书本，应该自己多实践。

继续，如果想只启动一个`MainActivity`，可以借助`FLAG_ACTIVITY_SINGLE_TOP`这个`Flag`，表示启用栈顶复用模式，因为`MainActivity`在栈顶，所以不会重新创建。

修改`Service`代码：

```kotlin
override fun onStartCommand(intent: Intent?, flags: Int, startId: Int): Int {
    startActivity(Intent(this,MainActivity::class.java)
        .apply {
            addFlags(Intent.FLAG_ACTIVITY_NEW_TASK)
            addFlags(Intent.FLAG_ACTIVITY_SINGLE_TOP)
        })
    return super.onStartCommand(intent, flags, startId)
}
```

测试结果：加上这个`Flag`之后，`MainActivity`没有重建，其`onNewIntent`得到回调。

此时，考虑一种情况。如果我们是在`SecondActivity`中启动了`Service`，此时`SecondActivity`在栈顶了。如果此时再用`FLAG_ACTIVITY_SINGLE_TOP`这个`Flag`，肯定还是会重建`MainActivity`，测试一下。

我们由`MainActivity`启动`SecondActivity`,在`SecondActivity`再启动`Service`。

经测试，不改变`Service`代码的情况下。`Service`启动后，任务栈有三个`Activity`，分别是`MainActivity`->`SecondActivity`->`MainActivity`。

如果我们只想保留一个`MainActivity`怎么办呢？此时可以使用`FLAG_ACTIVITY_CLEAR_TOP`这个`Flag`。

修改`Service`代码如下：

```kotlin
override fun onStartCommand(intent: Intent?, flags: Int, startId: Int): Int {
    startActivity(Intent(this,MainActivity::class.java)
        .apply {
            addFlags(Intent.FLAG_ACTIVITY_NEW_TASK)
            addFlags(Intent.FLAG_ACTIVITY_CLEAR_TOP)
        })
    return super.onStartCommand(intent, flags, startId)
}
```

此时运行一下，发现任务栈确实只有一个`MainActivity`了，不过该`MainActivity`重建了。如果我们不希望他重建怎么办呢？`FLAG_ACTIVITY_CLEAR_TOP`的文档中有一段话：

> ```
> The currently running instance of activity B in the above example will either receive the new intent you are starting here in its  onNewIntent() method, or be itself finished and restarted with the new intent.  If it has declared its launch mode to be "multiple" (the default) and you have not set {@link #FLAG_ACTIVITY_SINGLE_TOP} in the same intent, then it will be finished and re-created; for all other launch modes or if {@link #FLAG_ACTIVITY_SINGLE_TOP} is set then this Intent will be delivered to the current instance's onNewIntent().
> ```

可以知道，如果想避免重建可以加上`FLAG_ACTIVITY_SINGLE_TOP`。

修改`Service`代码：

```kotlin
override fun onStartCommand(intent: Intent?, flags: Int, startId: Int): Int {
    startActivity(Intent(this,MainActivity::class.java)
        .apply {
            addFlags(Intent.FLAG_ACTIVITY_NEW_TASK)
            addFlags(Intent.FLAG_ACTIVITY_CLEAR_TOP)
            addFlags(Intent.FLAG_ACTIVITY_SINGLE_TOP)
        })
    return super.onStartCommand(intent, flags, startId)
}
```

再次测试，同样的步骤`MainActivity`没有重建。`onNewIntent`方法得到回调。

### 场景二：在Activity中启动Activity。

试想一种场景，你在一个层级比较深的`Activity`，需要返回到`MainActivity`，并且不能销毁`MainActivity`，可以怎么做？其实和场景一一样，只是我们不需要加`FLAG_ACTIVITY_NEW_TASK`这个`Flag`。

修改代码，在`SencondActivity`中跳转到`ThirdActivity`，并由`ThirdActivity`跳转到`MainActivity`。

`ThirdActivity`代码如下：

```kotlin
bt_third_activity.setOnClickListener {
    startActivity(Intent(this,MainActivity::class.java)
        .apply {
            addFlags(Intent.FLAG_ACTIVITY_CLEAR_TOP)
            addFlags(Intent.FLAG_ACTIVITY_SINGLE_TOP)
        })
}
```

经测试，效果达到，任务栈中只有`MainActivity`，并且未重建，`onNewIntent`回调。

### 场景三：从通知中启动Activity。

我们根据场景2修改代码，在`ThirdActivity`中弹出个通知，并且希望点击通知回到`MainActivity`。

代码如下：

```kotlin
bt_third_activity.setOnClickListener {
    val intent = Intent(this, MainActivity::class.java)
        .apply {
            addFlags(Intent.FLAG_ACTIVITY_CLEAR_TOP)
            addFlags(Intent.FLAG_ACTIVITY_SINGLE_TOP)
        }
    val pendingIntent = PendingIntent.getActivity(this, 0, intent, 0)
    val ntf = NotificationCompat.Builder(this, "id")
        .setSmallIcon(R.mipmap.ic_launcher)
        .setContentTitle("title")
        .setContentText("text")
        .setContentIntent(pendingIntent)
        .build()

    val notificationManager =
        getSystemService(Service.NOTIFICATION_SERVICE) as NotificationManager
    notificationManager.notify(2, ntf)
}
```

测试发现`IntentFlag`无效了，在`so`上找到一个答案https://stackoverflow.com/questions/24873131/android-clear-task-flag-not-working-for-pendingintent

发现不能将`PendingIntent`的`requestCode`设置为`0`。修改成不为`0`之后效果达到和场景二一样。

修改代码如下：

```kotlin
    val pendingIntent = PendingIntent.getActivity(this, 1, intent, 0)
```

### 场景四：清除栈内所有Activity

这种情况比较常见的是`token`过期，需要清除栈内所有`Activity`，然后启动登录界面。我们假如`ThirdActivity`是登录界面。在任务栈栈内存在`MainActivity`，`SecondActivity`的情况下，此时`token`过期了。这时我们需要启动`ThirdActivity`。

此时我们需要用到`FLAG_ACTIVITY_CLEAR_TASK` 这个`Flag`，并且，文档表示该`Flag`必须和`FLAG_ACTIVITY_NEW_TASK`一起使用。我们试一下不一起使用会怎么样：

`SecondActivity`代码：

```kotlin
bt_second_activity.setOnClickListener {
    startActivity(Intent(this,ThirdActivity::class.java)
        .apply {
            addFlags(Intent.FLAG_ACTIVITY_CLEAR_TASK)
        })
}
```

经测试，如果不添加`FLAG_ACTIVITY_NEW_TASK`，任务栈并没有被清空，

加上之后效果正常。

### 结语

以上就是上面就是常见的几种使用`Flag`来启动`Activity`的场景。熟练之后肯定是比在`xml`中指定更加灵活的。

