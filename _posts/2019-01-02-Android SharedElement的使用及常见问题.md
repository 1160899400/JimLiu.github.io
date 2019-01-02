---
layout: post
title:  "Android SharedElement的使用及常见问题"
date:   2019-01-02
tags:

- Android
- UI
- Jetpack
- Leaks
---

## Android SharedElement 简介
&emsp;&emsp;SharedElement是Android中常用于实现跳转时动画的一个API，他能使多个activity的某个view互相关联，在activity跳转的时候展现该view的动画。效果如下：  

![Android共享元素实现动画](https://urt1rsliu.github.io/images/post/Android/androidSharedElement.gif)

&emsp;&emsp;它的具体使用一般如下（参考[官方文档](https://developer.android.com/training/transitions/start-activity#java)）：  

&emsp;&emsp;比如说现在要实现从Activity A跳转到另一个Activity B，A中具备动画的view为ViewA，对应B中的View为ViewB。首先要给View设置**相同**的TransitionName，代码如下：  
~~~java
	//或者调用ViewCompat.setTransitionName(viewA, "transition")；
	//也可以直接在xml中声明android:transitionName
	//共享的View的TransitionName必须相同
	VeiwA.setTransitionName("transition");
~~~
&emsp;&emsp;接着在activity A中将intent绑定一个Bundle，Bundle的创建和使用如下：  

~~~java
	Intent intent = new Intent(A.this, B.class);
	Bundle bundle = ActivityOptionsCompat.makeSceneTransitionAnimation(A.this, ViewA, ViewA.getTransitionName()).toBundle();
	startActiviy(intent, bundle);
~~~

&emsp;&emsp;这样，activity从A到B，就会现ViewA到ViewB的动画，下面如何监听这个过程呢？这里提供了两个回调：  
~~~java
//在activity A中监听跳转动画的开始(SharedElement exit)
setExitSharedElementCallback(new SharedElementCallback() {
	@Override
	public void onMapSharedElements(List<String> names, Map<String, View> sharedElements) {
	}
});

...

//在activity B中监听跳转动画的结束(SharedElement enter)
setExitSharedElementCallback(new SharedElementCallback() {
	@Override
	public void onMapSharedElements(List<String> names, Map<String, View> sharedElements) {
	}
});
~~~

## SharedElement造成的leak
&emsp;&emsp;在使用SharedElement时，经常会通过LeakCanary发现activity内存泄漏的情况。这是由于动画在定义时，TransitionManager类内部的静态成员sRunningTransitions（ThreadLocal对象）维护着这个Transition和它所在的DecorView，而DecorView又是activity的成员（activity的根view），由于这里的DecorView的引用是静态的，所以acitivy也不会释放。  

&emsp;&emsp;知道了内存泄漏的原因，那么解决就挺容易的了，只要把sRunningTransitions中对应的DecorView和Transition删掉就ok了。但是sRunningTransitions是private，所以得用下反射机制。代码如下：
~~~java
    if (Build.VERSION.SDK_INT < 21) {
        return;
    }
    Class transitionManagerClass = TransitionManager.class;
    try {
        Field runningTransitionsField = transitionManagerClass.getDeclaredField("sRunningTransitions");
            runningTransitionsField.setAccessible(true);
        //noinspection unchecked
        ThreadLocal<WeakReference<ArrayMap<ViewGroup, ArrayList<Transition>>>> runningTransitions
                = (ThreadLocal<WeakReference<ArrayMap<ViewGroup, ArrayList<Transition>>>>)
                runningTransitionsField.get(transitionManagerClass);
        if (runningTransitions.get() == null || runningTransitions.get().get() == null) {
            return;
        }
        ArrayMap map = runningTransitions.get().get();
        View decorView = activity.getWindow().getDecorView();
        if (map.containsKey(decorView)) {
            map.remove(decorView);
        }
    } catch (NoSuchFieldException e) {
        e.printStackTrace();
    } catch (IllegalAccessException e) {
        e.printStackTrace();
    }
~~~