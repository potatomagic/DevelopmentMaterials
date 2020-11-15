## APP启动的三种方式
### 冷启动
程序从头开始，系统没有为该程序创建进程。一般场景：程序安装后的第一次启动；应用程序被系统完全终止后再打开。
### 热启动
此时程序仍然驻留在内存中，只是被系统从后台带到前台，因此程序可以避免重复对象初始化，加载布局和渲染。需要注意的是，如果某些程序的某些对象被系统清除，比如调用了onTrimMemory方法，则需要重新创建这些对象以响应热启动事件。
### 暖启动
它包含热启动和冷启动一系列的操作子集，比热启动的消耗稍微多一些。它与热启动最大的区别在于，它必须通过调用onCreate方法开始重复创建活动，也可以从传递给onCreate方法中保存的实例状态中获得某些对象的恢复。

### APP启动流程
- 加载并启动APP
- 启动后立即为该APP显示一个空白启动窗口
- 创建APP进程
- 创建主Activity
- 加载布局、绘制

所以从APP被调用，再到第一个页面渲染到手机屏幕，我们通常只需要关注Application中的onCreate方法，第一个Activity中onCreate、onStart、onResume方法。

## void startActivity(Intent intent)

这个方法在Android开发中尤为重要，一句代码，可以开启一个页面，但其中究竟走了哪些内容？走到了哪里？这就是这篇文章的核心内容。（以下内容是根据Android 9来展开研究的）

我们在应用中任意位置调用startActivity，其本质是通过Binder来调用ActivityManagerService的startActivity方法。这里需要对Binder有相应的了解（[Binder机制](https://github.com/potatomagic/DevelopmentMaterials/blob/master/Android/Binder%E6%9C%BA%E5%88%B6.md)）。具体流程如下所示：

```java
android.content.Context
public abstract void startActivity(Intent intent);

android.content.ContextWrapper
Context mBase;
//代理模式，真正实现在ContextImpl中
public void startActivity(Intent intent) {
    mBase.startActivity(intent);
}

android.app.ContextImpl
public void startActivity(Intent intent) {
    ......
    mMainThread.getInstrumentation().execStartActivity(
                getOuterContext(), mMainThread.getApplicationThread(), null,
                (Activity) null, intent, -1, options);
    ......
}

android.app.Instrumentation
public ActivityResult execStartActivity(
        Context who, IBinder contextThread, IBinder token, Activity target,
        Intent intent, int requestCode, Bundle options) {
    ......
    //通过Binder远程调用ActivityManagerService的startActivity
    int result = ActivityManager.getService()
            .startActivity(whoThread, who.getBasePackageName(), intent,
            intent.resolveTypeIfNeeded(who.getContentResolver()),
            token, target != null ? target.mEmbeddedID : null,
            requestCode, 0, null, options);
    ......
}

// 调用实际是编译生成的IActivityManager.java中的对应方法
android.app.IActivityManager.aidl
int startActivity(in IApplicationThread caller, in String callingPackage, in Intent intent,
        in String resolvedType, in IBinder resultTo, in String resultWho, int requestCode,
        int flags, in ProfilerInfo profilerInfo, in Bundle options);

//该类通过IActivityManager.aidl编译产生
android.app.IActivityManager.java
// Client 发送数据
public int startActivity(android.app.IApplicationThread caller, java.lang.String callingPackage,
        android.content.Intent intent, java.lang.String resolvedType,
        android.os.IBinder resultTo, java.lang.String resultWho,
        int requestCode, int flags, android.app.ProfilerInfo profilerInfo,
        android.os.Bundle options) throws android.os.RemoteException {
    // 用来发送数据
    android.os.Parcel _data = android.os.Parcel.obtain();
    // 用来回传数据
    android.os.Parcel _reply = android.os.Parcel.obtain();
    ......
    // 向_data按照参数顺序依次写入参数
    _data.writeStrongBinder((((caller != null)) ? (caller.asBinder()) : (null)));
    ......
    // 将打包好的数据发送出去
    mRemote.transact(Stub.TRANSACTION_startActivity, _data, _reply, 0);
    ......
}

// Server 接收数据
public boolean onTransact(int code, android.os.Parcel data, android.os.Parcel reply, int flags)
        throws android.os.RemoteException {
    ......
    //接收其他进程传递来的startActivity调用
    case TRANSACTION_startActivity: {
        return this.onTransact$startActivity$(data, reply);
    }
    ......  
}

private boolean onTransact$startActivity$(android.os.Parcel data, android.os.Parcel reply)
        throws android.os.RemoteException {
    ......
    // 对Parcel data按照参数顺序进行参数提取，然后调用相应的方法
    android.app.IApplicationThread _arg0;
    _arg0 = android.app.IApplicationThread.Stub.asInterface(data.readStrongBinder());
    ......
    int _result = this.startActivity(_arg0, _arg1, _arg2, _arg3, _arg4, _arg5, _arg6, _arg7, _arg8, _arg9);
    ......
}

com.android.server.am.ActivityManagerService
//真实调用
public final int startActivity(IApplicationThread caller, String callingPackage,
        Intent intent, String resolvedType, IBinder resultTo, String resultWho, int requestCode,
        int startFlags, ProfilerInfo profilerInfo, Bundle bOptions) {
    return startActivityAsUser(caller, callingPackage, intent, resolvedType, resultTo,
            resultWho, requestCode, startFlags, profilerInfo, bOptions,
            UserHandle.getCallingUserId());
}

//根据参数创建ActivityStarter，调用ActivityStarter的execute来启动页面
public final int startActivityAsUser(IApplicationThread caller, String callingPackage,
        Intent intent, String resolvedType, IBinder resultTo, String resultWho, int requestCode,
        int startFlags, ProfilerInfo profilerInfo, Bundle bOptions, int userId,
        boolean validateIncomingUser) {
    enforceNotIsolatedCaller("startActivity");
    userId = mActivityStartController.checkTargetUser(userId, validateIncomingUser,
            Binder.getCallingPid(), Binder.getCallingUid(), "startActivityAsUser");
    // TODO: Switch to user app stacks here.
    return mActivityStartController.obtainStarter(intent, "startActivityAsUser")
            .setCaller(caller)
            .setCallingPackage(callingPackage)
            .setResolvedType(resolvedType)
            .setResultTo(resultTo)
            .setResultWho(resultWho)
            .setRequestCode(requestCode)
            .setStartFlags(startFlags)
            .setProfilerInfo(profilerInfo)
            .setActivityOptions(bOptions)
            .setMayWait(userId)
            .execute();
}

com.android.server.am.ActivityStarter
int execute() {
    try {
        // TODO(b/64750076): Look into passing request directly to these methods to allow
        // for transactional diffs and preprocessing.
        if (mRequest.mayWait) {
            return startActivityMayWait(mRequest.caller, mRequest.callingUid,
                    mRequest.callingPackage, mRequest.intent, mRequest.resolvedType,
                    mRequest.voiceSession, mRequest.voiceInteractor, mRequest.resultTo,
                    mRequest.resultWho, mRequest.requestCode, mRequest.startFlags,
                    mRequest.profilerInfo, mRequest.waitResult, mRequest.globalConfig,
                    mRequest.activityOptions, mRequest.ignoreTargetSecurity, mRequest.userId,
                    mRequest.inTask, mRequest.reason,
                    mRequest.allowPendingRemoteAnimationRegistryLookup);
        } else {
            return startActivity(mRequest.caller, mRequest.intent, mRequest.ephemeralIntent,
                    mRequest.resolvedType, mRequest.activityInfo, mRequest.resolveInfo,
                    mRequest.voiceSession, mRequest.voiceInteractor, mRequest.resultTo,
                    mRequest.resultWho, mRequest.requestCode, mRequest.callingPid,
                    mRequest.callingUid, mRequest.callingPackage, mRequest.realCallingPid,
                    mRequest.realCallingUid, mRequest.startFlags, mRequest.activityOptions,
                    mRequest.ignoreTargetSecurity, mRequest.componentSpecified,
                    mRequest.outActivity, mRequest.inTask, mRequest.reason,
                    mRequest.allowPendingRemoteAnimationRegistryLookup);
        }
    } finally {
        onExecutionComplete();
    }
}
```
