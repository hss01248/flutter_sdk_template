



# 额外的操作

host里的baseApp无法自动生成,需要手动粘贴:

```java
package com.xxx.xx.flutter.xxx_flutter.host;

import androidx.multidex.MultiDexApplication;

public class BaseApp extends MultiDexApplication {
}
```

修改Android里flutter模块脚本的sdk版本:

```java
"$flutterRoot/packages/flutter_tools/gradle/flutter.gradle"

/** For apps only. Provides the flutter extension used in app/build.gradle. */
class FlutterExtension {
    /** Sets the compileSdkVersion used by default in Flutter app projects. */
    static int compileSdkVersion = 33

    /** Sets the minSdkVersion used by default in Flutter app projects. */
    static int minSdkVersion = 16

    /** Sets the targetSdkVersion used by default in Flutter app projects. */
    static int targetSdkVersion = 33

```