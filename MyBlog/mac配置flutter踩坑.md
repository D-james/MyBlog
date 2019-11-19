按照官网的教程：[[https://flutterchina.club/setup-macos/](https://flutterchina.club/setup-macos/)](https://flutterchina.club/setup-macos/)

一步步来，到flutter doctor报错，根据错误提示修改错误。
###我的错误记录
用Android studio新建flutter项目报如下错误

```
error installing flutter null returned 1
```

Android studio在创建flutter项目选择SDK path时，提示的错误。

其中SDK path就是从官网下载的SDK，flutter文件夹地址。当flutter doctor命令没有验证通过时，选择SDK path会报这个错误。需要执行flutter doctor查看的报错


```

 ✗ libimobiledevice and ideviceinstaller are not installed. To install with

 Brew, run:

 brew update

 brew install --HEAD usbmuxd

 brew link usbmuxd

 brew install --HEAD libimobiledevice

 brew install ideviceinstaller

```

依次执行上述提示的brew命令（上面的命令有问题），再次flutter doctor

当mac插上手机设备报如下错误：

```

Unhandled exception:

Exception: idevice_id returned an error:

#0 IMobileDevice.getInfoForDevice (package:flutter_tools/src/ios/mac.dart:122:9)

<asynchronous suspension>

#1 IOSDevice.getAttachedDevices (package:flutter_tools/src/ios/devices.dart:152:53)

<asynchronous suspension>

#2 IOSDevices.pollingGetDevices (package:flutter_tools/src/ios/devices.dart:112:57)

#3 PollingDeviceDiscovery.devices (package:flutter_tools/src/device.dart:163:52)

<asynchronous suspension>

#4 DeviceManager.getAllConnectedDevices (package:flutter_tools/src/device.dart:91:46)

<asynchronous suspension>

#5 DeviceValidator.validate (package:flutter_tools/src/doctor.dart:677:54)

<asynchronous suspension>

#6 Doctor.startValidatorTasks (package:flutter_tools/src/doctor.dart:107:52)

#7 Doctor.diagnose (package:flutter_tools/src/doctor.dart:174:41)

#8 _AsyncAwaitCompleter.start (dart:async/runtime/libasync_patch.dart:49:6)

#9 Doctor.diagnose (package:flutter_tools/src/doctor.dart:164:24)

#10  DoctorCommand.runCommand (package:flutter_tools/src/commands/doctor.dart:29:39)

#11  _AsyncAwaitCompleter.start (dart:async/runtime/libasync_patch.dart:49:6)

#12  DoctorCommand.runCommand (package:flutter_tools/src/commands/doctor.dart:28:42)

#13  FlutterCommand.verifyThenRunCommand (package:flutter_tools/src/runner/flutter_command.dart:383:18)

#14  _asyncThenWrapperHelper.<anonymous closure> (dart:async/runtime/libasync_patch.dart:77:64)

#15  _rootRunUnary (dart:async/zone.dart:1132:38)

#16  _CustomZone.runUnary (dart:async/zone.dart:1029:19)

#17  _FutureListener.handleValue (dart:async/future_impl.dart:129:18)

#18  Future._propagateToListeners.handleValueCallback (dart:async/future_impl.dart:642:45)

#19  Future._propagateToListeners (dart:async/future_impl.dart:671:32)

#20  Future._complete (dart:async/future_impl.dart:476:7)

#21  _SyncCompleter.complete (dart:async/future_impl.dart:51:12)

#22  _AsyncAwaitCompleter.complete.<anonymous closure> (dart:async/runtime/libasync_patch.dart:33:20)

#23  _rootRun (dart:async/zone.dart:1124:13)

#24  _CustomZone.run (dart:async/zone.dart:1021:19)

#25  _CustomZone.bindCallback.<anonymous closure> (dart:async/zone.dart:947:23)

#26  _microtaskLoop (dart:async/schedule_microtask.dart:41:21)

#27  _startMicrotaskLoop (dart:async/schedule_microtask.dart:50:5)

#28  _runPendingImmediateCallback (dart:isolate/runtime/libisolate_patch.dart:115:13)

#29  _RawReceivePortImpl._handleMessage (dart:isolate/runtime/libisolate_patch.dart:172:5)

```

其实是上面的brew命令有问题，解决办法：

```

brew update

brew uninstall --ignore-dependencies libimobiledevice

brew uninstall --ignore-dependencies usbmuxd

brew install --HEAD usbmuxd

brew unlink usbmuxd

brew link usbmuxd

brew install --HEAD libimobiledevice

```

再次flutter doctor

这次只有一个错误

```

[!] Android toolchain - develop for Android devices (Android SDK version 28.0.3)

 ✗ Android license status unknown

```

执行下面命令，来接受安卓许可证协议

```

flutter doctor --android-licenses

```

报错

```

A newer version of the Android SDK is required. To update, run:

/Users/duan/Library/Android/sdk/tools/bin/sdkmanager --update

```

解决方法是使用`SDKMANAGER_OPTS`环境变量

```

export SDKMANAGER_OPTS="--add-modules java.se.ee"

```

之后再执行

```

flutter doctor --android-licenses

```

依次y回车，来接收许可证

再次flutter doctor，结果：
```

[✓] Flutter (Channel stable, v1.2.1, on Mac OS X 10.14.2 18C54, locale zh-Hans-CN)

[✓] Android toolchain - develop for Android devices (Android SDK version 28.0.3)

[✓] iOS toolchain - develop for iOS devices (Xcode 10.1)

[✓] Android Studio (version 3.3)

[✓] VS Code (version 1.31.1)

[✓] Connected device (2 available)

• No issues found!

```

成功~



最后记录下运行后的错误处理：
```
ERROR: Could not connect to lockdownd, error code -17
```
这是设备和计算机没有建立信任关系，手机和电脑互相信任就可以了