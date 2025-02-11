## 1. 常用 dump
``` shell
adb shell dumpsys activity service com.android.systemui/.SystemUIService > SystemUIService.txt

# nms dump
adb shell dumpsys notification --noredact

adb shell dumpsys SurfaceFlinger > SurfaceFlinger.txt

adb shell dumpsys window > window.txt

adb logcat -b events 
```

## 2. adb  shell settings get

可以用来获取 Android 设备上的**系统设置（配置）**。

命令格式：`adb shell settings get <namespace> <key>`

其中 namespace 是设置所在的命名空间，key 是设置的键名(字段)。

<namespace> 参数设置为 `system`、`secure` 或 `global`

```Java
system    // 包含了一些系统级别的设置，如屏幕亮度、音量等
secure    // 包含了一些安全相关的设置，如锁屏密码、Wi-Fi 密码等
global    // 包含了一些全局的设置，如时区、语言等。
```

<Key> 定义在      `frameworks/base/core/java/android/provider/Settings.java

栗子：

```Plain
adb shell settings list    // 返回所有设置的键名和值


adb shell settings get system battery_indicator_style      // 获取电池图标样式

adb shell settings put secure user_setup_complete 1        // 跳过开机向导
adb shell settings put global device_provisioned 1

adb shell settings delete global http_proxy                // 移除代理
```

## 3. adb shell getprop/setprop

从系统的各种配置文件中读取一些设备的信息，用来获取 Android 设备上的**系统属性**。

这些信息可以在手机设备中找到

```Shell
init.rc
default.prop
/system/build.prop
```

命令：

```Shell
adb shell getprop [key]

adb shell setprop [key] [value]
```

## 4. 快速查看手机中设置项settings

1. 先list现有的settings值：

```Shell
adb shell settings list secure >s1 && adb shell settings list global>g1 && adb shell settings list system>sys1
```

2. 操作更改设置项
3. list 修改后的settings值

```Shell
adb shell settings list secure >s2 && adb shell settings list global>g2 && adb shell settings list system>sys2
```

4. Diff

```Shell
diff s1 s2 && diff g1 g2 && diff sys1 sys2
```


## 5.  开启 bugger log

```Shell
adb shell cmd statusbar echo -t <tagName>:<level>
adb shell cmd statusbar echo -b <bufferName>:<level>

# eg: 
adb shell cmd statusbar echo -t NotifCollection:V
```

详细：

【参考类】com.android.systemui.log.LogBuffer

LogBuffer 是 SystemUI 记录日志的工具，正常情况下，只有在生成 BugReport 的时候，才会全部转储

【在 logcat 中使用】

正常只有大于等于 warn 级别的，才会回显到 logcat 中；但是它可以通过相关的命令实现回显到 logcat 中

LogBuffer 在构造的时候会传入 name 参数，这个就是 Buffer 的名称

LogBuffer 在记录的时候会传入 tag 和 level 参数，这个是日志的 tag 和级别

在进行回显的时候，可以针对 Buffer 或者针对 Tag 设置 level 级别的回显

1. 针对Buffer级别
   1. ```Shell
      adb shell cmd statusbar echo -b <bufferName>:<level>
      ```

   2.   bufferName 在子类构造的时候便会传入，如果是通过注解生成的，大概率是该类名称

   3.   也可以通过BugReport去查询下

   4.   以NotificationPanelViewController为例，它使用的是ShadeLog，它的Buffer名称就是“ShadeLog”

   5.   如果需要将整个ShadeLog全部回显到logcat中，可以执行以下命令

   6.   adb shell cmd statusbar echo -b ShadeLog:verbose
2. 针对 tag 级别
   1. ```Shell
      adb shell cmd statusbar echo -t <tagName>:<level>
      ```

 道理同上，以 ShadeLog 为例，`systemui.shade`即为 tag

```Shell
adb shell cmd statusbar echo -t systemui.shade:verbose
```

V 上可以用上述命令打开 buffer log，这样可以直接 logcat 实时抓


## 6.  查找手机内某资源属性
查找当前 dimen -- notification_panel_width
``` shell
adb shell cmd overlay lookup --verbose com.android.systemui com.android.systemui:dimen/notification_panel_width
```