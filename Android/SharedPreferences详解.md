# SharedPreferences
SharedPreferences是Android平台上轻量级的存储类，用来保存App的各种配置信息，其本质是一个以键值对（key-value）的方式保存数据的xml文件，其保存在/data/data/shared_prefs目录下。这样，每次读取数据时，通过解析xml文件，得到指定key对应的value；每次更新数据，也通过文件中key更新对应的value。

## 使用方法
### 创建SharedPreferences
直接调用Context类的getSharedPreferences(String name, int mode)方法即可。
SharedPreferences仅提供了几个读取的API，而写入的部分需要靠内部类Editor来实现，Editor的获取是通过调用SharedPreferences的edit()方法。

|SharedPreferences|SharedPreferences.Editor|
|---|---|
|getAll()||
|getBoolean(String key, boolean defValue)|putBoolean(String key, boolean value)|
|getFloat(String key, float defValue)|putFloat(String key, float value)|
|getInt(String key, int defValue)|putInt(String key, int value)|
|getLong(String key, long defValue)|putLong(String key, long value)|
|getString(String key, String defValue)|putString(String key, String value)|
|getStringSet(String key, Set<String> defValues)|putStringSet(String key, Set<String> values)|
|contains(String key)||
||remove(String key) //删除一条数据|
||clear() //删除所有数据|
||commit() //有返回boolean，true即为写入成功|
||apply() //无返回|

### 四种操作模式
- Context.MODE_PRIVATE：默认操作模式，代表该文件是私有数据,只能被应用本身访问，在该模式下，写入的内容会覆盖原文件的内容。
- Context.MODE_APPEND：会检查文件是否存在，存在就往文件追加内容，否则就创建新文件。
- Context.MODE_WORLD_READABLE：用来控制其他应用是否有权限读该文件。
- Context.MODE_WORLD_WRITEABLE：用来控制其他应用是否有权限写该文件。

### 跨进程使用SharedPreferences
假设A应用包名为com.android.myapplication，在A应用写入SharedPreferences，并指定模式为Context.MODE_WORLD_READABLE，这时，在B应用就可以通过Context来获取这个SharedPreferences
``` java
Context otherAppsContext = createPackageContext("com.android.myapplication", Context.CONTEXT_IGNORE_SECURITY);
SharedPreferences sharedPreferences = otherAppsContext.getSharedPreferences("sp", Context.MODE_WORLD_READABLE);
```
值得注意的是，AB两个应用需要有相同的用户ID与签名，用户ID在程序的AndroidManifest.xml文件的manifest标签中指定，格式为android:shareUserId="*****"。通常不建议这么使用，更多情况下会使用ContentProvider等来保存多个程序交互使用的共享数据。

## SharedPreferences的问题
SharedPreferences有着严重的性能问题，甚至会导致ANR，


https://mp.weixin.qq.com/s?__biz=MzI1MzYzMjE0MQ==&mid=2247484387&idx=1&sn=e3c8d6ef52520c51b5e07306d9750e70&scene=21#wechat_redirect


https://blog.csdn.net/mq2553299/article/details/109134825?utm_medium=distribute.pc_feed.none-task-blog-personrec_tag-7.nonecase&depth_1-utm_source=distribute.pc_feed.none-task-blog-personrec_tag-7.nonecase&request_id=5f8cc07e98fa0775e7fa90a6
