对比Android8多窗口，OPENTHOS不光在原有的五部分进行修改，还需要针对桌面化增加新的内容，Android 8 多窗口解析链接如下：

[Android8 多窗口解析](https://github.com/potatomagic/GoodGoodStudy_DayDayUp/blob/master/Android/Android%E5%A4%9A%E7%AA%97%E5%8F%A3/Android7%20Android8%20%E5%A4%9A%E7%AA%97%E5%8F%A3%E8%A7%A3%E6%9E%90.md)

||AOSP 8|OPENTHOS 8|
|---|---|---|
|Stack ID|定义Stack ID，根据需求来指定Activity的样式，默认FullScreen Mode，且Freeform默认不打开，|增加最小化Stack，BACKGROUD Stack ID，同样根据需求来显示Activity，默认Freeform mode|
|标题栏layout|控制窗口最大化，关闭以及拖动|增加回退，最小化，窗口样式选择菜单等操作|
|窗口初始位置和大小|需要先打开应用，然后在近期任务中切换，或者安装TaskBar来直接进入Freeform，原理是在启动Activity的时候设置了Activity的Bounds|默认给出初始化的Bounds，并做好窗口尺寸数据的保存|
|窗口移动和缩放|默认支持|增加缩放大小框，节约性能；增加窗口停靠自动变化窗口尺寸|
|Focus Activity|默认支持,但多窗口状态下，Launcher不可见|Launcher焦点问题，并将Launcher从Home Stack移动到Freeform Stack|
|新增||Alt + Tab快捷键|
|新增||应用屏幕旋转的处理|
|新增||兼容模式|

## Stack ID
