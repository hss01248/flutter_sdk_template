




# override pkg:

flutter3.0.5/packages/flutter_tools/template

# 额外的操作

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



# flutter sdk里flutter tool会删除.gradle缓存文件夹的bug

https://github.com/flutter/flutter/issues/89959

> 非常恶心的bug, 一不小心,你pc上的全局gradle缓存就被flutter tool干掉了.谷歌哪招的水货程序员!!!

解决方式: 改flutter sdk源码,注掉里面的几行代码:(来自上面issues里的)

I think I have partially solved this problem.

The solution:

1. Go to the folder where the sdk is located, for example: `~/flutter`
2. Locate the following file: `~/flutter/packages/flutter_tools/lib/src/android/gradle_errors.dart` and open it for editing.
3. Locate the definition of `networkErrorHandler`. In my case I found it on line 210. You can also do the following search: `The Gradle cache directory must be deleted`.
4. Comment out all code related to deleting the `.gradle` folder

```
@visibleForTesting
final GradleHandledError networkErrorHandler = GradleHandledError(
  test: _lineMatcher(const <String>[
    'java.io.FileNotFoundException: https://downloads.gradle.org',
    'java.io.IOException: Unable to tunnel through proxy',
    'java.lang.RuntimeException: Timeout of',
    'java.util.zip.ZipException: error in opening zip file',
    'javax.net.ssl.SSLHandshakeException: Remote host closed connection during handshake',
    'java.net.SocketException: Connection reset',
    'java.io.FileNotFoundException',
    "> Could not get resource 'http",
  ]),
  handler: ({
    required String line,
    required FlutterProject project,
    required bool usesAndroidX,
    required bool multidexEnabled,
  }) async {
    globals.printError(
      '${globals.logger.terminal.warningMark} Gradle threw an error while downloading artifacts from the network. '
      'Retrying to download...'
    );
    // try {
    //   final String? homeDir = globals.platform.environment['HOME'];
    //   if (homeDir != null) {
    //     final Directory directory = globals.fs.directory(globals.fs.path.join(homeDir, '.gradle'));
    //     ErrorHandlingFileSystem.deleteIfExists(directory, recursive: true);
    //   }
    // } on FileSystemException catch (err) {
    //   globals.printTrace('Failed to delete Gradle cache: $err');
    // }
    return GradleBuildStatus.retry;
  },
  eventLabel: 'network',
);
```

Now you need the flutter_tools package to be recompiled. To do this go to the `~/flutter/bin/cache` folder and delete the following files: `flutter_tools.stamp` and `flutter_tools.snapshot`

Then run the `flutter doctor` command. You will see a message: `Building flutter tool...`. With this the `flutter_tools` package has been recompiled and this time it should not delete the `.gradle` folder when it detects a connection failure.

I assume that every time you update the SDK the same operation should be performed, so you should keep this in mind when you update to a new version of the SDK.
