## 黑白屏
由于系统在APP启动的第一时间启动一个空白页面，这个页面通常是白色的，与APP内部整体页面设计有冲突，可能会导致黑白屏的现象，该问题解决方法有
- 修改默认主题AppTheme，视觉效果可以解决黑白屏，但会感觉点击图标有一种卡顿的感觉
  - 在应用默认的AppTheme中，设置系统“取消预览（空白窗体）”为true
  ```xml
  <style name="AppTheme" parent="Theme.AppCompat.Light.DarkActionBar">
      <item name="android:windowDisablePreview">true</item>
  </style>
  ```
  - 在应用默认的AppTheme中，设置空白窗体背景色为透明
  ```xml
  <style name="AppTheme" parent="Theme.AppCompat.Light.DarkActionBar">
      <item name="android:windowIsTranslucent">true</item>
  </style>
  ```
- 使用自定义主题，从设计和使用体验上优化App启动时的空白窗体，例如SplashActivity，或称为过渡页面、启动页面，如使用layer-list。

## 代码优化
在构建App时，我们经常需要引用一些第三方的sdk，而项目业务越多，引用的第三方也越多，有些第三方会要求我们在Application的onCreate方法中对其初始化。这意味着：在Application的onCreate方法中执行时间会被越长，首个Activity布局的渲染时间也会相应的拉长。同理，如果我们在Activity的onCreate，onStart，onResume方法中执行的任务时间过长，同样也会导致布局被渲染的时间拉长。这样直接导致的问题就是，用户会感觉页面迟迟没有加载出来，用户体验极差。

## 启动时间检测
### App启动时间检测
adb shell am start -W com.android。application/.MainActivity 运行结果解释如下：
- ThisTime:最后一个Activity启动时间
- TotalTime:一系列Activity启动时间
- WaitTime:总启动时间，包含系统在冷启动时，需要加载app信息到内存的时间

### 代码执行时间检测
```java
public void onCreate() {
    //trace文件位置 /storage/emulated/0/app.trace
    File file=new File(Environment.getExternalstorageDirectory(), "app.trace");
    Debug.startMethodTracing(file.getAbsolutePath());
    //业务逻辑代码
    init();
    Debug.stopMethodTracing();
}
```
