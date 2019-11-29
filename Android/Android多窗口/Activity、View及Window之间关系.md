# View
View是Android中最基本的UI组件，是所有的控件类的父类。

# Window
Window表示一个窗口，一般来说，Window大小取值为屏幕大小。但对话框、Toast等就不是整个屏幕大小。我们可以指定Window的大小。Window包含一个View Tree和窗口的layout参数。简单的说，Window相当于一个容器，里面“盛放”着很多View，这些View是以树状结构组织起来的。在Android中，Window是一个抽象类，它的唯一实现类是PhoneWindow。

# Activity
一个Activity就“相当于”一个页面，通常会在Activity里处理事件，如onKeyEvent，onTouchEvent等，并可以通过Activity维护应用程序的生命周期。
