# FileProvider
FileProvider是Android为了提高私有文件的安全性而诞生的，在Android 22.0.0，也就是Android 5.0版本加入进Android系统，FileProvider是ContentProvider的子类，功能就是用来提供文件在跨进程间的访问能力。从Android 7.0开始，严格模式更加的严格。为了进一步提高私有文件的安全性，Android不再由开发者放宽私有文件的访问权限，之前我们一直使用"file:///"绝对路径来传递文件地址的方式，应用间的文件共享需要使用content://类型的URI分享，并且需要为其提供临时的文件访问权限
(Intent.FLAG_GRANT_READ_URI_PERMISSION和Intent.FLAG_GRANT_WRITE_URI_PERMISSION)。frameworks具体判断Uri合法性位置如下：
```java
frameworks/base/core/java/android/content/Intent.java
public void prepareToLeaveProcess(boolean leavingPackage) {
    ......
    if (mAction != null && mData != null && StrictMode.vmFileUriExposureEnabled() && leavingPackage) {
        ......
        mData.checkFileUriExposed("Intent.getData()");
        ......
    }
}

frameworks/base/core/java/android/net/Uri.java
public void checkFileUriExposed(String location) {
    if ("file".equals(getScheme()) && (getPath() != null) && !getPath().startsWith("/system/")) {
        StrictMode.onFileUriExposed(this, location);
    }
}

frameworks/base/core/java/android/os/StrictMode.java
public static void onFileUriExposed(Uri uri, String location) {
    final String message = uri + " exposed beyond app through " + location;
    if ((sVmPolicyMask & PENALTY_DEATH_ON_FILE_URI_EXPOSURE) != 0) {
        throw new FileUriExposedException(message);
    } else {
        onVmPolicyViolation(null, new Throwable(message));
    }
}
```

## FileProvider使用方法
- 在AndroidManifest.xml中配置FileProvider，${applicationId}通常是应用的包名。meta-data是以键值对的方式保存，android.support.FILE_PROVIDER_PATHS作为meta-data的键，@xml/file_paths作为meta-data的值。在FileProvider中会读取meta-data中的android.support.FILE_PROVIDER_PATHS对应的值。
```xml
<provider
    android:name="android.support.v4.content.FileProvider"
    android:authorities="${applicationId}.fileprovider"
    android:exported="false"
    android:grantUriPermissions="true">
    <meta-data
        android:name="android.support.FILE_PROVIDER_PATHS"
        android:resource="@xml/file_paths"/>
</provider>
```

- 在res下创建xml文件夹，并创建file_paths文件，该文件名随意，需要与清单配置文件匹配
```xml
<?xml version="1.0" encoding="utf-8"?>
<paths xmlns:android="http://schemas.android.com/apk/res/android">
    <root-path
        name="root-path"
        path="." />
    <files-path
        name="files-path"
        path="." />
    <cache-path
        name="cache-path"
        path="." />
    <external-path
        name="external-path"
        path="." />
    <external-files-path
        name="external_file_path"
        path="." />
    <external-cache-path
        name="external_cache_path"
        path="." />
</paths>
```

- 使用FileProvider提供的API来生成相应的Uri，并且赋予相应的权限
```java
File file = new File("path");
Uri uri;
if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.N) {
    uri = FileProvider.getUriForFile(context, context.getApplicationContext().getPackageName() + ".fileprovider", file);
} else {
    uri = Uri.fromFile(file);
}
Intent intent = new Intent();
intent.setData(uri);
intent.addFlags(Intent.FLAG_GRANT_READ_URI_PERMISSION | Intent.FLAG_GRANT_WRITE_URI_PERMISSION);
```

- 多个FileProvider共存处理，这种情况主要出现在使用第三方SDK，第三方SDK同样使用了FileProvider，如果不考虑自定义FileProvider，就需要把第三方sdk中的路径配置copy到路径配置文件中
```xml
<?xml version="1.0" encoding="utf-8"?>
<paths xmlns:android="http://schemas.android.com/apk/res/android">
    <!-- /storage/emulated/0/Download/${applicationId}/.beta/apk-->
    <external-path name="beta_external_path" path="Download/"/>
    <!--/storage/emulated/0/Android/data/${applicationId}/files/apk/-->
    <external-path name="beta_external_files_path" path="Android/data/"/>

    <external-path name="external_storage_root" path="."/>
    <files-path name="files" path="."/>
</paths>
```

- 2018年，Android推出了新的扩展库AndroidX，用于替换原来的Android扩展库，这时候就需要将android.support.v4.content.FileProvider更改为androidx.core.content.FileProvider。清单配置文件，导包等问题需要根据targetVersion来确定，避免发生FileProvider找不到的问题。

## Uri的解析
根据版本的不同，传递来的Uri主要有两种模式，一种是"file:///"，一种是"content:///"。
- file:/// 后跟的是绝对路径，直接获取即可
- content:/// 这种需要使用ContentResolver来查询，通常需要知道相应的字段来查询。或者无须知道具体路径，以流的形式直接来读取加载即可，例如ContentResolver提供了通过加载Uri获取BitMap的借口，MediaPlayer提供了通过Uri创建实例的接口。值得注意的是，FileProvider获取的URI是没有路径参数的，只有文件名和文件大小两个参数，主要原因是提高安全性，具体代码如下：

```java
private static final String[] COLUMNS = {OpenableColumns.DISPLAY_NAME, OpenableColumns.SIZE };

public Cursor query(@NonNull Uri uri, @Nullable String[] projection, @Nullable String selection, @Nullable String[] selectionArgs, @Nullable String sortOrder) {
   final File file = mStrategy.getFileForUri(uri);
   if (projection == null) {
       projection = COLUMNS;
   }
   String[] cols = new String[projection.length];
   Object[] values = new Object[projection.length];
   int i = 0;
   for (String col : projection) {
       if (OpenableColumns.DISPLAY_NAME.equals(col)) {
           cols[i] = OpenableColumns.DISPLAY_NAME;
           values[i++] = file.getName();
       } else if (OpenableColumns.SIZE.equals(col)) {
           cols[i] = OpenableColumns.SIZE;
           values[i++] = file.length();
       }
   }
   cols = copyOf(cols, i);
   values = copyOf(values, i);
   final MatrixCursor cursor = new MatrixCursor(cols, 1);
   cursor.addRow(values);
   return cursor;
}
```
