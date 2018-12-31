---
layout: post
title:  "Android Launcher长按icon呼出菜单"
date:   2018-12-27
tags:

- Android
- Jetpack
- Architecture Components
---

&emsp;&emsp;作为Android 7.0的新特性，app支持在Launcher界面长按icon呼出菜单（shortcuts），例如图：  

<img src = "https://urt1rsliu.github.io/images/post/Android/android shortcuts.jpg"  style="zoom:25%" algin = center/>


&emsp;&emsp;整个菜单被官方称为shortcuts，菜单里的选项称为shortcut，参照[Android官方shortcuts Guide介绍](https://developer.android.com/guide/topics/ui/shortcuts/)。下面根据官方推荐的做法介绍下shortcuts的基本使用:  

#### 如何创建Shortcuts:
&emsp;&emsp;首先得知道每个shortcut是对应着一个或多个intent的，通过启动这些intent来实现跳转，shortcut的类型有3种: Static ，Dynamic以及Pinned，官方对于三者的描述如下：  

- Static shortcuts are best for apps that link to content using a consistent structure throughout the lifetime of a user's interaction with the app. Because most launchers can only display four shortcuts at once, static shortcuts are useful for common activities. For example, if the user wants to view their calendar or email in a specific way, using a static shortcut ensures that their experience in performing a routine task is consistent.
- Dynamic shortcuts are used for actions in apps that are context-sensitive. For instance, if you build a game that allows the user to start from their current level on launch, the shortcut will need to be updated frequently. Using a dynamic shortcut allows the shortcut to be updated each time the user clears a level.
- Pinned shortcuts are used for specific, user-driven actions. For example, a user might want to pin a specific website to the launcher. This is beneficial because it allows the user to perform a custom action, like navigating to the website in one step, more quickly than using a default instance of a browser.

&emsp;&emsp;总结大概意思，就是说static shortcuts适合使用在整个app生命周期内都保持不变的操作，例如绝大多数常见的activity，而且支持同时显示的最大个数为4个。  

&emsp;&emsp;而Dynamic shortcuts适用于对context敏感的操作，例如游戏app中进入当前等级的shortcut，当用户清除等级后，就需要更新这个shortcut，这个时候Dynamic shortcuts就有用处。  

&emsp;&emsp;Pinned shortcuts可以将一些特定操作，比如将某些网站固定到Shortcuts，让用户能一步导航到网站，这种方式比直接使用浏览器打开要快。(Android 8.0支持)  

&emsp;&emsp;废话不多说，创建shortcuts的步骤如下:  

##### 创建static shorcuts
1. 在AndroidManifest.xml下找到你的启动activity，在\<activity\>\</activity\>下添加\<meta-data\>如下：

~~~ruby
	<meta-data android:name="android.app.shortcuts"
               android:resource="@xml/shortcuts" /> 
~~~

2.然后创建一个资源文件shortcuts.xml，放在res/xml/ 目录下 
3.往这个资源文件的\<shortcuts\>根节点添加\<shortcut\>节点，\<shortcut\>节点内可以指定一个或多个intent，含有多个intent时，显示最后一个intent对应的内容，shortcut 节点示例如下：

~~~ruby
<shortcuts xmlns:android="http://schemas.android.com/apk/res/android">
  <shortcut
    android:shortcutId="compose"
    android:enabled="true"
    android:icon="@drawable/compose_icon"
    android:shortcutShortLabel="@string/compose_shortcut_short_label1"
    android:shortcutLongLabel="@string/compose_shortcut_long_label1"
    android:shortcutDisabledMessage="@string/compose_disabled_message1">
    <intent
      android:action="android.intent.action.VIEW"
      android:targetPackage="com.example.myapplication"
      android:targetClass="com.example.myapplication.ComposeActivity" />
    <!-- If your shortcut is associated with multiple intents, include them
         here. The last intent in the list determines what the user sees when
         they launch this shortcut. -->
    <categories android:name="android.shortcut.conversation" />
  </shortcut>
  <!-- Specify more shortcuts here. -->
</shortcuts>
~~~

&emsp;&emsp;关于shortcut节点，有很多属性可以设置，下面是一些属性的作用：

| attribute                       | neccesary        | 意义                                                         |
| ------------------------------- | ---------------- | :----------------------------------------------------------- |
| android:shortcutId              | **must provide** | id                                                           |
| android:shortcutShortLabel      | **must provide** | 大小不超过10个字符，shortcut的文字显示                       |
| android:shortcutLongLabel       | optional         | 大小不超过25个字符，当shortcut空间足够时，显示这个而不是android:shortcutShortLabel |
| android:shortcutDisabledMessage | optional         | 当用户点击一个disabled的shortcut时弹出的消息提示             |
| android:enabled                 | optional         | 是否可用，默认为enable                                       |
| android:icon                    | optional         | shortcut显示的图标，位于文字旁边                             |

##### 创建dynamic shortcuts
&emsp;&emsp;上面介绍的是以静态的方式创建的shortcuts，下面介绍动态的创建方式：  

&emsp;&emsp;Google提供了**ShortcutManager**的API用于的动态的创建shortcuts，下面是ShortcutManager的几个重要功能的实现（更多函数可以看源码）：  

1. **Publish**：`setDynamicShortcuts()`，用来设置或重新设置shortcuts；`addDynamicShortcuts()`向已有的shortcuts添加shortcuts 。

2. **Update**：更新shortcuts。

3. **Remove**:`removeDynamicShortcuts()`移除一系列shortcuts，`removeAllDynamicShortcuts()`移除所有shortcuts。

&emsp;&emsp;下面是具体使用实例：  
~~~ruby
//ShortcutManager shortcutManager = getSystemService(Context.SHORTCUT_SERVICE);
ShortcutManager shortcutManager = getSystemService(ShortcutManager.class);

ShortcutInfo shortcut = new ShortcutInfo.Builder(context, "id1")
    .setShortLabel("Website")
    .setLongLabel("Open the website")
    .setIcon(Icon.createWithResource(context, R.drawable.icon_website))
    .setIntent(new Intent(Intent.ACTION_VIEW,
                   Uri.parse("https://www.mysite.example.com/")))
    .build();

shortcutManager.setDynamicShortcuts(Arrays.asList(shortcut));
~~~

##### 创建pinned shortcuts
&emsp;&emsp;步骤如下：  

1. 调用 `isRequestPinShortcutSupported()` 确认设备是否支持pinned shortcuts
2. 创建**ShortcutInfo**对象，根据该shortcut是否存在选择其中一种方式创建：
    a. 如果Shortcut已经存在，创建仅设置和该shorcut的id相同的id的shorcut
    b. 如果Shortcut不存在，创建设置了id，intent，label的ShortcutInfo
3. 调用 `requestPinShortcut()`将该shortcut固定到launcher页面的shorcuts下。这个函数可以通过传入一个PendingIntent对象来回调通知添加完成
4. 调用`updateShortcuts()`更新。

&emsp;&emsp;实例如下：
~~~ruby
ShortcutManager mShortcutManager =
        context.getSystemService(ShortcutManager.class);

if (mShortcutManager.isRequestPinShortcutSupported()) {
    // Assumes there's already a shortcut with the ID "my-shortcut".
    // The shortcut must be enabled.
    ShortcutInfo pinShortcutInfo =
            new ShortcutInfo.Builder(context, "my-shortcut").build();

    // Create the PendingIntent object only if your app needs to be notified
    // that the user allowed the shortcut to be pinned. Note that, if the
    // pinning operation fails, your app isn't notified. We assume here that the
    // app has implemented a method called createShortcutResultIntent() that
    // returns a broadcast intent.
    Intent pinnedShortcutCallbackIntent =
            mShortcutManager.createShortcutResultIntent(pinShortcutInfo);

    // Configure the intent so that your app's broadcast receiver gets
    // the callback successfully.For details, see PendingIntent.getBroadcast().
    PendingIntent successCallback = PendingIntent.getBroadcast(context, /* request code */ 0,
            pinnedShortcutCallbackIntent, /* flags */ 0);

    mShortcutManager.requestPinShortcut(pinShortcutInfo,
            successCallback.getIntentSender());
}
~~~