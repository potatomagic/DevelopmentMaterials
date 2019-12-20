# FileProvider
FileProvider是Android为了提高私有文件的安全性而诞生的，在Android 22.0.0，也就是Android 5.0版本加入进Android系统，该组件是ContentProvider的子类，功能就是用来提供文件在跨进程间的访问能力。从Android 7.0中正式使用，因为从Android 7.0


## 使用方法
- 在AndroidManifest.xml中配置FileProvider
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
- 在res下创建xml文件夹，并创建file_paths文件
```xml
<?xml version="1.0" encoding="utf-8"?>
<paths xmlns:android="http://schemas.android.com/apk/res/android">
    <root-path
        name="ext_sd_files"
        path="." />
    <files-path
        name="int_files"
        path="." />
    <external-path
        name="ext_files"
        path="." />
    <cache-path
        name="cache_files"
        path="." />
    <external-files-path
        name="ext_int_files"
        path="." />
</paths>
```
- 使用FileProvider提供的API来生成相应的Uri
```java
File file = new File("path");
Uri uri;
if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.N) {
    uri = FileProvider.getUriForFile(context, context.getApplicationContext().getPackageName() + ".fileprovider", file);
} else {
    uri = Uri.fromFile(file);
}
```
