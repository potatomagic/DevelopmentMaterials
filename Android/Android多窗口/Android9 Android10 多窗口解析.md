Android9开始，多窗口架构有较大的变化，与Android7和Android8有很大的不同。首先，多窗口依然有三种模式：

- Split-Screen Mode: 分屏模式。
- Freeform Mode 自由模式：类似于Windows的窗口模式。
- Picture In Picture Mode：画中画模式（PIP）。

变化较为明显的是Freeform自由模式。在Android7和Android8中，Freeform模式主要有五部分构成：
- 定义特有的Freeform Stack ID，所有的自由模式的Activity会在特定的Freeform Stack启动
- 为Freeform窗口添加特殊的layout, 用于控制窗口最大化，关闭以及拖动等功能
- 为Freeform提供一个控制窗口初始化位置和大小的类
- 为Freeform提供可以控制窗口移动和缩放处理的类
- 为Freeform添加一个Focus Activity

在Android9中，多窗口第一个核心变化是Stack ID取消，使用WINDOWING MODE来代替，具体代码在frameworks/base/core/java/android/app/WindowConfiguration.java中
```java
/** Windowing mode is currently not defined. */
public static final int WINDOWING_MODE_UNDEFINED = 0;
/** Occupies the full area of the screen or the parent container. */
public static final int WINDOWING_MODE_FULLSCREEN = 1;
/** Always on-top (always visible). of other siblings in its parent container. */
public static final int WINDOWING_MODE_PINNED = 2;
/** The primary container driving the screen to be in split-screen mode. */
public static final int WINDOWING_MODE_SPLIT_SCREEN_PRIMARY = 3;
/**
* The containers adjacent to the {@link #WINDOWING_MODE_SPLIT_SCREEN_PRIMARY} container in
* split-screen mode.
* @see #WINDOWING_MODE_FULLSCREEN_OR_SPLIT_SCREEN_SECONDARY
*/
public static final int WINDOWING_MODE_SPLIT_SCREEN_SECONDARY = 4;
/**
* Alias for {@link #WINDOWING_MODE_SPLIT_SCREEN_SECONDARY} that makes it clear that the usage
* points for APIs like {@link ActivityOptions#setLaunchWindowingMode(int)} that the container
* will launch into fullscreen or split-screen secondary depending on if the device is currently
* in fullscreen mode or split-screen mode.
*/
public static final int WINDOWING_MODE_FULLSCREEN_OR_SPLIT_SCREEN_SECONDARY =
       WINDOWING_MODE_SPLIT_SCREEN_SECONDARY;
/** Can be freely resized within its parent container. */
public static final int WINDOWING_MODE_FREEFORM = 5;
```
这种设计模式的好处是可以很明显的看到多种WINDIWING MODE的窗口显示在同一屏幕，不再由AMS来管理窗口的显示，将窗口的显示部分全部都交给WMS，这一点在Android 10的源码中体现的较为明显，Google将原来在frameworks/base/services/core/java/com/android/server/am/目录下的很多类都移动到frameworks/base/services/core/java/com/android/server/wm/目录中。
