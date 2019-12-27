# FileProvider
FileProvider是Android为了提高私有文件的安全性而诞生的，在Android 22.0.0，也就是Android 5.0版本加入进Android系统，该组件是ContentProvider的子类，功能就是用来提供文件在跨进程间的访问能力。从Android 7.0中正式使用，因为从Android 7.0开始，严格模式更加的严格。


## 使用方法
- 在AndroidManifest.xml中配置FileProvider，${applicationId}通常是应用的包名。meta-data是以键值对的方式保存，android.support.FILE_PROVIDER_PATHS作为meta-data的键（key），@xml/file_paths作为meta-data的值。在FileProvider中会读取meta-data中的android.support.FILE_PROVIDER_PATHS对应的值。

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
