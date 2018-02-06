
##### java print Stack
Log.i(TAG,Log.getStackTraceString(new Throwable()));

##### c++ print the call stack traces for a single thread
#include <utils/CallStack.h>
CallStack stack;
stack.update();
stack.log(LOG_TAG);

##### c++ print the call stack traces for all threads in a process
#include <utils/ProcessCallStack.h>
ProcessCallStack stack;
stack.update();
stack.log(LOG_TAG);

##### View SurfaceView GLSurfaceView 区别

View：显示视图，内置画布，提供图形绘制函数、触屏事件、按键事件函数等；必须在UI主线程内更新画面，速度较慢。
SurfaceView：基于view视图进行拓展的视图类，更适合2D游戏的开发；是view的子类，类似使用双缓机制，在新的线程中更新画面所以刷新界面速度比view快。
GLSurfaceView：基于SurfaceView视图再次进行拓展的视图类，专用于3D游戏开发的视图；是SurfaceView的子类，openGL专用。

##### audio output sampling rate
primary输出类先根据audio_policy.conf中的配置,选取不大于48000的最大sampling rate，然后通过audio hal打开output时，hal层可以修改samping rate


##### stack symbolizes native crash dumps

. build/envsetup.sh && lunch && stack
[develoment/script/stack]

### find a activity
dumpsys window w | grep name

### start a activity
am start package.activity

### compile error
"Exception in thread "main" java.lang.UnsupportedClassVersionError: com/android/signapk/SignApk : Unsupported major.minor version 52.0"
android 7.1 need jdk 1.8
