[QGC GitHub PR #11753](https://github.com/mavlink/qgroundcontrol/pull/11753)

## Background

We have been facing recurring issues when attempting to set up the QGC build for Android, despite strictly following the official build documentation. This report details the challenges encountered, the troubleshooting steps undertaken, and the solutions implemented to resolve these issues.

## Problem Statement

Despite following the build documentation precisely, the QGC build process on Android consistently resulted in errors that were difficult to diagnose. The primary error encountered was as follows:

```
FAILURE: Build failed with an exception.

What went wrong:
Execution failed for task ':processDebugResources'.
A failure occurred while executing com.android.build.gradle.internal.res.LinkApplicationAndroidResourcesTask$TaskAction
Android resource linking failed
aapt2 E 07-28 13:26:58 45293 6627245 LoadedArsc.cpp:94] RES_TABLE_TYPE_TYPE entry offsets overlap actual entry data.
aapt2 E 07-28 13:26:58 45293 6627245 ApkAssets.cpp:149] Failed to load resources table in APK '/Users/kaustuvpokharel/Library/Android/sdk/platforms/android-35/android.jar'.
error: failed to load include path /Users/kaustuvpokharel/Library/Android/sdk/platforms/android-35/android.jar.

Try:
Run with --stacktrace option to get the stack trace. Run with --info or --debug option to get more log output.
Task :desugarDebugFileDependencies
Run with --scan to get full insights.
Get more help at <https://help.gradle.org/>.

```

There were a few other random errors that led to dead ends, making it harder to comprehend the root cause.

## Investigation and Findings

Initial troubleshooting efforts involved inquiring on various QGC community platforms to understand the root cause of the errors. Through extensive trial and error, it was discovered that the stable variant of Qt 6.6.3 was compatible with Java Development Kit (JDK) versions 17, 19, and 20. Adopting JDK 17 significantly reduced the number of build errors.

Further investigation revealed that the outdated instructions in the official documentation were misleading. The recommendation of JDK 11 was suitable for Qt 5.15.2. However, with the transition from Qt 5 to Qt 6, the appropriate JDK version should have been updated to JDK 17 for Qt 6.6.3, particularly for Android 13 and later versions. Additionally, the specific version of the Native Development Kit (NDK) used also played a crucial role. The compatible NDK version for our setup was 25.1.8937393.

## Correct Configuration

The successful configuration that resolved the build issues is as follows:

```yaml
name: Setup Java Environment
uses: actions/setup-java@v4
with:
  distribution: temurin
  java-version: 17

- name: Setup Android Environment
  uses: android-actions/setup-android@v3
  with:
    cmdline-tools-version: 11076708
    packages: 'platform-tools platforms;android-34 build-tools;34.0.0' # ndk;25.1.8937393'

- name: Install Qt6 for Android (arm64_v8a)
  uses: jurplel/install-qt-action@v4
  if: contains(env.QT_ANDROID_ABIS, 'arm64-v8a')
  with:
    version: ${{ env.QT_VERSION }}
    aqtversion: ==3.1.*
    host: macOS
    target: android
    arch: android_arm64_v8a
    dir: ${{ runner.temp }}
    modules: qtcharts qtlocation qtpositioning qtspeech qt5compat qtmultimedia qtserialport qtimageformats qtshadertools qtconnectivity qtquick3d qtsensors
```

## Resolution and Contribution

Upon identifying the issues and correcting the configuration, we submitted a Pull Request (PR) to the QGC GitHub repository. The PR was reviewed and merged into the master branch by the project owners, acknowledging the necessity of these updates for general users to build successfully. The approved PR can be found below:

[QGC GitHub PR #11753](https://github.com/mavlink/qgroundcontrol/pull/11753)

## Conclusion

The primary cause of the build errors was the outdated configuration instructions in the official QGC documentation. Updating the JDK version to 17 and ensuring compatibility with specific NDK versions resolved the issues. The updated configuration has been successfully merged into the QGC master branch, improving the build process for the entire community.
