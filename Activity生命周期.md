# Activity生命周期

---

## 为什么要了解Activity的生命周期
**了解Activity的生命周期的根本目的就是为了设计用户体验更加良好的应用。**因为Activity就相当于MVC中的View层，是为了更好的向用户展现数据，并与之交互。了解Activity的生命周期和各回调方法的触发时机，我们可以更好的在合适的地方向用户展现数据（因为每个应用每个Activity的作用不同，所以具体每个回调方法的最佳实践不好把握，但是只要遵循最基本的原则即可），保证数据的完整性和程序的良好运行。

下面是官方文档的原话：
> Managing the lifecycle of your activities by implementing callback methods is crucial to developing a strong and flexible application. The lifecycle of an activity is directly affected by its association with other activities, its task and back stack.  

翻译：对于开发一个强大和灵活的应用程序，实现Activity的回调方法来管理Activity的生命周期至关重要。一个Activity的生命周期直接影响与它结合的其他Activity，它的任务和回退栈。  

## Activity生命周期的表现
除了我们自行启动（start）或者结束（finish）一个Activity，我们并不能直接控制一个Activity 的生命状态，我们只能通过实现Activity生命状态的表现 —— 即回调方法来达到管理Activity生命周期的变化。

具体如下图所示：

![](http://onmer39jj.bkt.clouddn.com/image/060009291302389.png)

### entire lifetime （整个生命周期）
Activity实例是由系统自动创建，并在不同的状态期间回调相应的方法。一个最简单的完整的Activity生命周期会按照如下顺序回调：onCreate -> onStart -> onResume -> onPause -> onStop -> onDestroy。**Activity应该在onCreate()方法里执行设置“全局”状态(如定义布局)，并在onDestroy()方法里释放所有剩余资源。**例如，如果你的活动有一个线程在后台运行下载网络数据，它可以在onCreate()中创建该线程，然后在onDestroy()中停止线程。

### visible lifetime（可见生命周期）
当执行onStart回调方法时，Activity开始被用户所见（也就是说，onCreate时用户是看不到此Activity的，那用户看到的是哪个？当然是此Activity之前的那个Activity），一直到onStop之前，此阶段Activity都是被用户可见。**在这两个方法，你可以保持该Activity需要展示给用户的资源。**例如，您可以在onStart()方法里注册一个BroadcastReceiver来监控你的UI的变化，并在onStop()方法里注销它。在整个生命周期的活动中，系统可能会调用onStart()和onStop()多次，因为活动之间交替进行隐藏或显示给用户。

### foreground lifetime（前台生命周期）
当执行到onResume回调方法时，Activity可以响应用户交互，一直到onPause方法之前。在这段时间里，这个Activity在其他所有Activity的前面，拥有用户输入焦点。一个Activity可以经常在前台状态发生转换—比如，当设备休眠或者弹出了个对话框。因为经常会发生转换，所以**在这两个方法之间的代码应该是轻量级的**，防止导致其他转换变慢使得用户需要等待。

一个Activity本质上只有三种状态：
Resumed（运行）、Paused（暂停）、Stopped（停止），因为从Activity被创建之后，它只可能在这三种状态保持长久的停留，其他的回调方法结束后的状态都只能称之为过渡状态。比如进入到onStart方法后，执行完该方法，会立即进入到OnResume方法。（这里所说的状态都是指对应的某个方法返回之后）

即使一个Activity进入到Paused或者Stopped方法，它仍然是存在的，被保存在任务回退栈中。它仍然保持着自身的所有实例和状态，所以根本不用担心它在返回到onResume方法时，实例会变为null，或者控件的事件监听不了。唯一需要考虑的就是，系统在内存不足的情况下，杀死在Paused或者Stopped状态下的Activity。
当一个Activity在Resumed状态下，它是不会因内存不够而被系统直接杀死（在极端的情况下也有可能被杀死，但是一般不会考虑这种情况）。只有进入Paused或者Stopped状态才会，而且可能根本就不会去调用onStop()和onDestory()方法，所以**onPause()方法是我们最大程度上保证Activity在销毁之前能够执行到的方法**。因此，如果你的某个Activity需要保存某些数据到数据库，您应该在onPause()里编写持久化数据的代码。但要注意，你应该选择哪些信息必须保留在onPause()，因为这个方法任何阻塞程序都会阻止过渡到下一个Activity，这样给用户体验就感觉十分缓慢。

### 应用场景

在实际应用场景中，假设A Activity位于栈顶，此时用户操作，从A Activity跳转到B Activity。那么对AB来说，具体会回调哪些生命周期中的方法呢？回调方法的具体回调顺序又是怎么样的呢？

开始时，A被实例化，执行的回调有A:onCreate -> A:onStart -> A:onResume。

**当用户点击A中按钮来到B时，假设B全部遮挡住了A，将依次执行A:onPause -> B:onCreate -> B:onStart -> B:onResume -> A:onStop。**

**此时如果点击Back键，将依次执行B:onPause -> A:onRestart -> A:onStart -> A:onResume -> B:onStop -> B:onDestroy。**

至此，Activity栈中只有A。在Android中，有两个按键在影响Activity生命周期这块需要格外区分下，即Back键和Home键。我们先直接看下实验结果：

**此时如果按下Back键，系统返回到桌面，并依次执行A:onPause -> A:onStop -> A:onDestroy。**

**此时如果按下Home键（非长按），系统返回到桌面，并依次执行A:onPause -> A:onStop。由此可见，Back键和Home键主要区别在于是否会执行onDestroy。**

**此时如果长按Home键，不同手机可能弹出不同内容，Activity生命周期未发生变化。**

由于Android本身的特性，使得现在不少应用都没有直接退出应用程序的功能，按照一般的逻辑，当Activity栈中有且只有一个Activity时，当按下Back键此Activity会执行onDestroy，那么下次点击此应用程图标将从重新启动，因此，当前不少应用程序都是采取如Home键的效果，当点击了Back键，系统返回到桌面，然后点击应用程序图标，直接回到之前的Activity界面，这种效果是怎么实现的呢？

通过重写按下Back键的回调函数，转成Home键的效果即可。
```
@Override
public void onBackPressed() {
    Intent home = new Intent(Intent.ACTION_MAIN);
    home.addCategory(Intent.CATEGORY_HOME);
    startActivity(home);
}
```
当然，此种方式通过Home键效果强行影响到Back键对Activity生命周期的影响。注意，此方法只是针对按Back键需要退回到桌面时的Activity且达到Home效果才重写。

或者，为达到此类效果，Activity实际上提供了直接的方法。
```
activity.moveTaskToBack(true);
```
moveTaskToBack()此方法直接将当前Activity所在的Task移到后台，同时保留activity顺序和状态。

在之前的项目开发过程中，当时遇到一个很奇怪的问题：手机上的“开发者选项”中有一个“不保留活动”的设置，当开启此设置，手机上的设置提示是“用户离开后即销毁每个活动”，开启后，对于其他的应用程序是从A Acticity到B Activity，然后Back键回到A，此时，其他应用程序只是先白屏（有可能黑屏等，取决于主题设置）一下，然后A开始可见，但是我的应用程序中出现的一个结果却是直接返回到了桌面。一开始百思不得其解。最后终于定位出问题。首先，我们需要明确开启此设置项后对Activity生命周期的影响。开启此设置项后，当A到B时，假设B全部遮挡住了A，将依次执行A:onPause -> B:onCreate -> B:onStart -> B:onResume -> A:onStop -> A:onDestroy。是的，A在系统原本的生命周期回调中增加了onDestroy。此即“用户离开后即销毁每个活动”的含义。但此时需要注意的是，只要没有认为的调用A的finish()方法，虽然A执行了onDestroy，但Activity栈中依然保留有A，此时B处于栈顶。那么在B中按Back键回到A时，将依次执行：B:onPause -> A:onCreate -> A:onStart -> A:onResume -> B:onStop -> B:onDestroy。没错，A从onCreate开始执行了。此处也就解释了为什么A可能会出现白屏（或黑屏等）一下的原因了。

那么为什么我的应用程序会跟其他应用程序出现不一样呢?最后定为出问题在于当时我的应用程序中为了做到完全退出应用程序效果，专门使用了一个Activity栈去维护Activity（当时是借鉴了网上的此类实现方案，现在想想，实在没必要，且不说Android本身特性决定了没必要通过如此方法去达到退出效果，仅仅是此方法本身也存在很大的问题，现在在网上依然能见到有不少文章说到应用程序退出可以使用此方法，哎。。），在onCreate中入栈，onDestroy出栈，调用了如下方法：

```
// 结束Activity&从堆栈中移除
AppManager.getAppManager().finishActivity(this);
```

其中，AppManager中finishActivity函数具体定义是：

```
/**
 * 结束指定的Activity
 */
public void finishActivity(Activity activity) {
	if (activity != null) {
   		activityStack.remove(activity);
        activity.finish();
        activity = null;
	}
}
```

至此，相信大家应该看出问题的所在了吧。

没错，问题在于执行了activity的finish()方法！！ activity的finish()方法至少有两个层面含义，1.将此Activity从Activity栈中移除，2.调用了此Activity的onDestroy方法。对于不开启“不保留活动”的设置项，实际上也没什么影响，但是一旦开启此设置，问题显露无疑。开启此此设置后，正常情况下离开A，即使执行了A的onDestroy，Activity栈中还是有A的，但是我这样写后，finish()方法一执行，Activity栈中就没有A了，因此，当点击Back键时，Activity栈中已经没有此应用的任何Activity了，直接来到了手机桌面。

可能，有些人会说，我就是要通过此种方法想去完全退出应用程序，同时希望自己的Activity栈和系统中Activity栈保持一致，怎么办呢？

在此，可以通过如下改写去实现：

```
/**
* 结束指定的Activity
 */
public void finishActivity(Activity activity) {
    if (activity != null) {
    // 为与系统Activity栈保持一致，且考虑到手机设置项里的"不保留活动"选项引起的Activity生命周期调用onDestroy()方法所带来的问题,此处需要作出如下修正
    if(activity.isFinishing()){
        activityStack.remove(activity);
        //activity.finish();
        activity = null;
    }
    }
}
```

### 横竖屏切换

- 不设置Activity的android:configChanges时，切屏会重新调用各个生命周期，切横屏时会执行一次，切竖屏时会执行两次
	1. 切换到横屏：onSaveInstanceState -> onPause -> onStop -> onDestroy -> onCreate -> onStart -> onRestoreInstanceState -> onResume

	2. 切换回竖屏，销毁了两次：onSaveInstanceState -> onPause -> onStop -> onDestroy -> onCreate -> onStart -> onRestoreInstanceState -> onResume -> onSaveInstanceState -> onPause -> onStop -> onDestroy -> onCreate -> onStart -> onRestoreInstanceState -> onResume

- 设置Activity的android:configChanges="orientation"时，切屏还是会重新调用各个生命周期，切横、竖屏时只会执行一次
	1. 修改AndroidManifest.xml，把该Activity添加 android:configChanges="orientation"，切横屏，只销毁一次：onSaveInstanceState -> onPause -> onStop -> onDestroy -> onCreate -> onStart -> onRestoreInstanceState -> onResume

	2. 再切回竖屏，只销毁一次，但调用了onConfigurationChanged：onSaveInstanceState -> onPause -> onStop -> onDestroy -> onCreate -> onStart -> onRestoreInstanceState -> onResume -> onConfigurationChanged

- 设置Activity的android:configChanges="orientation|keyboardHidden"时，切屏不会重新调用各个生命周期，只会执行onConfigurationChanged方法。（**注意：自从Android 3.2（API 13），在设置Activity的android:configChanges="orientation|keyboardHidden"后，还是一样会重新调用各个生命周期的。因为screen size也开始跟着设备的横竖切换而改变。因此，阻止程序在运行时重新加载Activity，除了设置"orientation"，你还必须加上"screenSize"。**）
	1. 把 android:configChanges="orientation" 改成 android:configChanges="orientation|keyboardHidden"，切横屏：onConfigurationChanged

	2. 再切回竖屏：onConfigurationChanged