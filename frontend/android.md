# Index

- [Index](#index)
- [Overview](#overview)
- [Android Studio](#android-studio)
  - [Android Debug Bridge (adb)](#android-debug-bridge-adb)
    - [Command](#command)
- [Android App](#android-app)
  - [Configuration](#configuration)
    - [Change app name](#change-app-name)
    - [Change package name](#change-package-name)
    - [Change the app icon](#change-the-app-icon)
  - [Development](#development)
    - [Build Release apk](#build-release-apk)
- [Android Resource](#android-resource)
  - [Dimension](#dimension)
  - [Drawable Folder](#drawable-folder)

# Overview

# Android Studio

## Android Debug Bridge (adb)

Android Debug Bridge (`adb`) is a versatile command-line tool that lets you communicate with a device.

### Command

**Show devices attached**

```sh
$ adb devices

List of devices attached
R28M406DDWD     device
```

If the device is `unauthorized`, try to unplug the device, turn off the USB debugging, and revoke USB debugging authorisations, and then replug the device.

**Show all system services**

```sh
# First, list out all services
$ adb shell dumpsys -l

# Then, choose the service that you want to inspect on, e.g. window
$ adb shell dumpsys window
```

- For dpi (dot per inch), you may check `window` service, and search `mDisplayInfo`.

**Show all installed apps**

```sh
$ adb shell pm list packages com.mf

package:com.mfinance.success.dev
package:com.mfinance.android.CFJ
package:com.mfinance.android.SUI
```

**Get the apk path of an installed app**

```sh
$ adb shell pm path com.mfinance.copymaster.react

package:/data/app/~~cfK8tYdcWWTasJoZEfj6RA==/com.mfinance.copymaster.react-vgNWZJoXS48tAyWuTsiEdg==/base.apk
```

**Pull file from device to computer**

```sh
$ adb pull /data/app/~~cfK8tYdcWWTasJoZEfj6RA==/com.mfinance.copymaster.react-vgNWZJoXS48tAyWuTsiEdg==/base.apk ./

/data/app/~~cfK8tYdcWWTasJoZEfj6RA==/com.mfinance.copymaster.react-vgNWZJoXS48tAyWuTsiEdg==/base.apk: 1 file pulled, 0 skipped. 37.8 MB/s (62537446 bytes in 1.579s)
```

# Android App

## Configuration

### Change app name

1. https://stackoverflow.com/questions/5443304/how-to-change-an-android-apps-name
2. Go to `<project_dir>\android\app\src\main\AndroidManifest.xml`.
3. Update `android:label`.

### Change package name

1. Go to `<project_dir>\android\app\build.gradle`
2. Change `applicationId`.

### Change the app icon

- https://developer.android.com/codelabs/basic-android-kotlin-compose-training-change-app-icon#4

## Development

### Build Release apk

- https://reactnative.dev/docs/signed-apk-android?package-manager=npm
- seems this work: https://medium.com/geekculture/react-native-generate-apk-debug-and-release-apk-4e9981a2ea51

```
cd android; ./gradlew assembleRelease
```

You can find the generated APK at `<project_dir>\android\app\build\outputs\apk\release\`.

# Android Resource

## Dimension

- https://developer.android.com/guide/topics/resources/more-resources.html#Dimension

Unit | Description
---- | -----------
`dp` | Density-independent pixels: an abstract unit that is based on the physical density of the screen. <br><br> These units are relative to a 160 dpi (dots per inch) screen, on which 1 dp is roughly equal to 1 px. For example, in a 320 dpi screen, 1 dp will become 2 px.

## Drawable Folder

- [Support different pixel densities - Android Developer](https://developer.android.com/training/multiscreen/screendensities)

Density Qualifier | Description | Scale | Example
-------- | ----------- | ----- | -------
drawable | For fallback if no specific drawable folder is matched. |  |
drawable-mdpi | Resources for medium-density screens (~160 dpi) | 1 | 375 px
drawable-hdpi | Resources for high-density screens (~240 dpi) | 1.5 | 563 px
drawable-xhdpi | Resources for extra-high-density screens (~320 dpi) | 2 | 750 px
drawable-xxhdpi | Resources for extra-extra-high-density screens (~480 dpi) | 3 | 1125 px
drawable-xxxhdpi | Resources for extra-extra-extra-high-density screens (~640 dpi) | 4 | 1500 px

Based on the device's screen density, Android selects the resource at the closest larger density bucket and then scales it down.

For example, on a phone of 420 dpi, Android decides that the app will use `xxhdpi`, so it will use the image of 1125 px. Then, if you set `android:layout_width` or `android:layout_height` as `375dp`, the image will be scaled to 984 px.
