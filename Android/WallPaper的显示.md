# Android壁纸
在Android中，壁纸分为静态与动态两种。静态壁纸是一张图片，而动态壁纸则以动画为表现形式，或者可以对用户的操作作出反应。这两种形式看似差异很大，其实二者的本质是统一的。它们都以一个Service的形式运行在系统后台，并在一个类型为TYPE_WALLPAPER的窗口上绘制内容。本篇代码部分主要以Android8.1为主，Android其他版本在代码上也大同小异。

# 涉及的源代码
```shell
frameworks/base/services/java/com/android/server/WallpaperManagerService.java
frameworks/base/core/java/android/service/wallpaper/WallpaperService.java
frameworks/base/core/java/android/app/WallpaperManager.java
frameworks/base/packages/SystemUI/src/com/android/systemui/ImageWallpaper.java
frameworks/base/services/java/com/android/server/wm/WindowManagerService.java
frameworks/base/services/java/com/android/server/wm/WindowStateAnimator.java
frameworks/base/services/java/com/android/server/wm/WindowAnimator.java
```

## 静态壁纸服务
静态壁纸服务在SystemUI里面，会随着系统的启动而启动，实现路径在：
```java
frameworks/base/packages/SystemUI/src/com/android/systemui/ImageWallpaper.java
/**
 * Default built-in wallpaper that simply shows a static image.
 */
public class ImageWallpaper extends WallpaperService {
    @Override
    public void onCreate() {
        super.onCreate();
        mWallpaperManager = (WallpaperManager) getSystemService(WALLPAPER_SERVICE);
    }
}

frameworks/base/core/res/res/values/config.xml
<string name="default_wallpaper_component" translatable="false">@null</string>
<string name="image_wallpaper_component" translatable="false">com.android.systemui/com.android.systemui.ImageWallpaper</string>

frameworks/base/core/java/android/app/WallpaperManager.java
public static ComponentName getDefaultWallpaperComponent(Context context) {
    String flat = SystemProperties.get(PROP_WALLPAPER_COMPONENT);
    if (!TextUtils.isEmpty(flat)) {
        final ComponentName cn = ComponentName.unflattenFromString(flat);
        if (cn != null) {
            return cn;
        }
    }
    flat = context.getString(com.android.internal.R.string.default_wallpaper_component);
    if (!TextUtils.isEmpty(flat)) {
        final ComponentName cn = ComponentName.unflattenFromString(flat);
        if (cn != null) {
            return cn;
        }
    }
    return null;
}

frameworks/base/services/core/java/com/android/server/wallpaper/WallpaperManagerService.java
final ComponentName mImageWallpaper;
public WallpaperManagerService(Context context) {
    ......
    mImageWallpaper = ComponentName.unflattenFromString(
            context.getResources().getString(R.string.image_wallpaper_component));
    mDefaultWallpaperComponent = WallpaperManager.getDefaultWallpaperComponent(context);
    ......
}

boolean bindWallpaperComponentLocked(ComponentName componentName, boolean force,
        boolean fromUser, WallpaperData wallpaper, IRemoteCallback reply) {
    ......
    try {
        if (componentName == null) {
            componentName = mDefaultWallpaperComponent;
            if (componentName == null) {
                // Fall back to static image wallpaper
                componentName = mImageWallpaper;
            }
        }
        ......
        Intent intent = new Intent(WallpaperService.SERVICE_INTERFACE);
        if (componentName != null && !componentName.equals(mImageWallpaper)) {
            // Make sure the selected service is actually a wallpaper service.
            ......
        }
        // Bind the service!
        ......
    }
    ......
}

```
根据ImageWallpaper的注释，我们可以得知，ImageWallpaper是用来展示一张静态的图片作为壁纸。可以看到ImageWallpaper实际继承的是WallpaperService，并且在onCreate()方法中获取WallpaperManager对象，用于和WallpaperManagerService通信。另外WallpaperManagerService是随着系统的启动而启动，会根据当前壁纸使用的服务的包名来进行bind，默认情况下是使用SystemUi的壁纸服务。同样的，如果是动态壁纸，也需要去继承WallpaperService来实现相应的效果，例如Android5.1中动态壁纸MagicSmoke就有如下代码：
```java
public abstract class RenderScriptWallpaper<T extends RenderScriptScene> extends WallpaperService {
}
```

可以看到AOSP中动态壁纸使用了WallpaperService，并且使用了RenderScript来实现动态壁纸效果。

## 壁纸加载Engine
壁纸的加载使用的是Engine，Engine是WallpaperService中的一个内部类，实现了壁纸窗口的创建以及Surface的维护工作。另外，Engine提供了可供子类重写的一系列回调，用于通知关于壁纸的生命周期、Surface状态的变化以及对用户的输入事件进行响应。可以说，Engine类是壁纸实现的核心所在。壁纸开发者需要继承Engine类，并重写其提供的回调以完成壁纸的开发。ImageWallpaper自然也会对Engine进行定制处理，可以看到ImageWallpaper的内部类DrawableEngine继承自Engine：
```java
frameworks/base/packages/SystemUI/src/com/android/systemui/ImageWallpaper.java
class DrawableEngine extends Engine {

    @Override
    public void onCreate(SurfaceHolder surfaceHolder) {
        ......
        updateSurfaceSize(surfaceHolder, getDefaultDisplayInfo(), false /* forDraw */);
    }

    int mBackgroundWidth = -1, mBackgroundHeight = -1;

    boolean updateSurfaceSize(SurfaceHolder surfaceHolder, DisplayInfo displayInfo,boolean forDraw) {
        boolean hasWallpaper = true;
        // Load background image dimensions, if we haven't saved them yet
        if (mBackgroundWidth <= 0 || mBackgroundHeight <= 0) {
            // Need to load the image to get dimensions
            mWallpaperManager.forgetLoadedWallpaper();
            loadWallpaper(forDraw, false /* needsReset */);
            hasWallpaper = false;
        }
        // Force the wallpaper to cover the screen in both dimensions
        int surfaceWidth = Math.max(displayInfo.logicalWidth, mBackgroundWidth);
        int surfaceHeight = Math.max(displayInfo.logicalHeight, mBackgroundHeight);

        if (FIXED_SIZED_SURFACE) {
            // Used a fixed size surface, because we are special.  We can do
            // this because we know the current design of window animations doesn't
            // cause this to break.
            surfaceHolder.setFixedSize(surfaceWidth, surfaceHeight);
            ......
        } else {
            surfaceHolder.setSizeFromLayout();
        }
        return hasWallpaper;
    }

    private AsyncTask<Void, Void, Bitmap> mLoader;

    private void loadWallpaper(boolean needsDraw, boolean needsReset) {
        ......
        mLoader = new AsyncTask<Void, Void, Bitmap>() {
                @Override
                protected Bitmap doInBackground(Void... params) {
                    try {
                        ......
                        return mWallpaperManager.getBitmap();
                    }
                    ......
                    return null;
                }

                @Override
                protected void onPostExecute(Bitmap b) {
                    mBackground = null;
                    mBackgroundWidth = -1;
                    mBackgroundHeight = -1;
                    if (b != null) {
                        mBackground = b;
                        mBackgroundWidth = mBackground.getWidth();
                        mBackgroundHeight = mBackground.getHeight();
                    }
                    updateSurfaceSize(getSurfaceHolder(), getDefaultDisplayInfo(),
                            false /* forDraw */);
                    if (mNeedsDrawAfterLoadingWallpaper) {
                        drawFrame();
                    }
                    mLoader = null;
                    mNeedsDrawAfterLoadingWallpaper = false;
                }
        }.executeOnExecutor(AsyncTask.THREAD_POOL_EXECUTOR);
    }

    void drawFrame() {
        ......
        SurfaceHolder sh = getSurfaceHolder();
        final Rect frame = sh.getSurfaceFrame();
        final int dw = frame.width();
        final int dh = frame.height();
        ......

        // Load bitmap if it is not yet loaded
        if (mBackground == null) {
            loadWallpaper(true /* needDraw */, true /* needReset */);
            return;
        }

        // Center the scaled image
        mScale = Math.max(1f, Math.max(dw / (float) mBackground.getWidth(),
                dh / (float) mBackground.getHeight()));
        final int availw = dw - (int) (mBackground.getWidth() * mScale);
        final int availh = dh - (int) (mBackground.getHeight() * mScale);
        int xPixels = availw / 2;
        int yPixels = availh / 2;
        .....
        // draw wallpaper
        if (ActivityManager.isHighEndGfx()) {
            if (!drawWallpaperWithOpenGL(sh, availw, availh, xPixels, yPixels)) {
                drawWallpaperWithCanvas(sh, availw, availh, xPixels, yPixels);
            }
        } else {
            drawWallpaperWithCanvas(sh, availw, availh, xPixels, yPixels);
        }
        ......
    }
}
```
FIXED_SIZED_SURFACE值如果配置为true则采用最大尺寸设置为壁纸surface的尺寸即图片大就用图片的尺寸，屏幕分辨率大就使用屏幕的尺寸。所以如果图片大就可以滑动桌面时壁纸随着移动。如果配置为false则以屏幕视图尺寸为准。图片大的话会遭到剪切，即使图片大也不会随着桌面滑动。

DrawableEngine在初始化时调用了updateSurfaceSize()方法。而在updateSurfaceSize()方法中，初始情况下，mBackgroundWidth和mBackgroundHeight的值都是-1，这时候需要进行loadWallpaper()方法。loadWallpaper()方法主要是通过异步AsyncTask调用mWallpaperManager.getBitmap()去获取壁纸资源，并且一次获取失败就会再去获取一遍，如果还是没有得到就只好返回null。当正常得到了图片资源后会重新调用updateSurfaceSize()方法对长宽尺寸进行简单调整后，调用drawFrame()方法进行壁纸的渲染，渲染的方式有两种，一种是OpenGL，一种是Canvas，渲染静态图片原理就不在这里赘述了。

## 壁纸重要文件信息
- /data/system/users/{userid}/wallpaper_info.xml
- /data/system/users/{userid}/wallpaper

通常情况下，{userid}是0。这两个壁纸信息文件配置在WallpaperManagerService中，代码如下
```java
frameworks/base/services/java/com/android/server/WallpaperManagerService.java
static final String WALLPAPER_CROP = "wallpaper";
static final String WALLPAPER_INFO = "wallpaper_info.xml";

final String base = new File(getWallpaperDir(userId), WALLPAPER_INFO).getAbsolutePath();

File preNWallpaper = new File(getWallpaperDir(0), WALLPAPER_CROP);

private static File getWallpaperDir(int userId) {
    return Environment.getUserSystemDirectory(userId);
}

frameworks/base/core/java/android/os/Environment.java
public static File getUserSystemDirectory(int userId) {
    return new File(new File(getDataSystemDirectory(), "users"), Integer.toString(userId));
}

public static File getDataSystemDirectory() {
    return new File(getDataDirectory(), "system");
}

public static File getDataDirectory() {
    return DIR_ANDROID_DATA;
}

private static final File DIR_ANDROID_DATA = getDirectory(ENV_ANDROID_DATA, "/data");
```

## WallpaperManager
WallpaperManager作为一个应用可以获取的服务，主要方法如下，具体使用不在这里赘述。

|方法|描述|
|--|--|
|setBitmap(Bitmap bitmap)|将壁纸设置为bitmap所代表的位图|
|setResource(int resid)|将壁纸设置为resid资源所代表的图片|
|setStream(InputStream data)|将壁纸设置为data数据所代表的图片|
|clear()|清除壁纸，设置回系统默认的壁纸|
|getDesiredMinimumHeight()|最小壁纸高度|
|getDesiredMinimumWidth()|最小壁纸宽度|
|getDrawable()|获得当前系统壁纸，如果没有设置壁纸，则返回系统默认壁纸|
|getWallpaperInfo()|加入当前壁纸是动态壁纸，返回动态壁纸信息，否则返回null|
|peekDrawable()|获得当前系统壁纸，如果没设置壁纸的话返回null|

## 总结
Android系统壁纸主要分成三部分

|class|描述|
|--|--|
|WallpaperManagerService|系统级壁纸服务|
|WallpaperManager|提供给其他应用的壁纸管理类，通过Binder与WallpaperManagerService通信|
|WallpaperService|壁纸应用需要集成实现的壁纸服务，会由WallpaperManagerService来bind通信|
