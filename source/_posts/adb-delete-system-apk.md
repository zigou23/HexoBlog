---
title: 免root ADB命令卸载系统预装应用
date: 2020-07-24 10:45:01
categories: 
- android
tags:
- android
- adb
- system
---

# 免root ADB命令卸载/停用系统预装应用

首先要知道系统预装的应用分3种：

1. **不可卸载**
    这部分没有 root 权限，是不能卸载的。比如 Phone、Message、Calendar 等。
    - `com.android.xxx`是系统包，千万别卸载，不然重启会开不了
    - 其他为厂商强行植入的软件，涉及设置 框架等部分权限的...自己看着办吧
2. **可卸载，可恢复**
    (国内流氓厂商刷机包自带的apk)这部分没有 root 权限，用户也可以卸载，但恢复出厂后又会回来。很多第三方的 APP。
3. **可卸载，不恢复**
    (自己安装的apk)这部分预置在 data 分区，data 分区是用户存储数据的分区，恢复出厂清空数据时也会清理掉这部分 APP。这种情况一般作特殊用途，比如出厂前测试硬件好坏的部分工具 APP，测完恢复出厂，用户也就感觉不到了。

好了，废话不多说，步骤如下：

## 1、激活开发者模式，打开 USB debug(usb调试)。

1. 打开手机设置，点击“系统”->“关于手机”连续快速点击“版本号”，即会出现 “开发者”提示

2. 打开手机设置，点击“更多设置”->“开发人员选项”，找到调试选项卡，激活“USB调试”，下边能开的尽量开

## 2、连接电脑，打开 cmd 窗口。

1. 记得先下载adb工具包 (我用的Google的，比这全：[连接在这](https://dl.google.com/android/repository/platform-tools-latest-windows.zip))

2. 解压到文件夹中，打开文件夹(里面有adb.exe)

3. 点击上面输入框并输入cmd然后回车并连接电脑

## 3、`adb shell` 进入 shell 模式。

1. 输入`adb devices`（判断你有没有下载adb程序）

   第一次使用adb命令，手机会弹出提示是否允许USB调试，同时CMD屏幕上显示unauthorized,即为未授权，手机上点允许，再输入上面所述命令即可见到屏幕显示device了。
   
   ~~~cmd&PowerShell
   C:\Users\xxx> adb devices
   List of devices attached
   a86bc****04    device
	~~~
	


2. 看标题 `adb shell` 进入 shell 模式
   ```cmd&PowerShell
   C:\Users\xxx> adb shell
   ****:/ $
   ```

## 4、获取要卸载的应用的包名，方法如下：

- 系统自带更好

- [下载apk](https://go.zzzzx.ml/0:/tools/android/APK%E6%8F%90%E5%8F%96%E5%99%A8_DZH-1.1.1.apk)  [蓝奏云](https://zig.lanzoux.com/imlztex9v1i) [副本](https://p.zzza.ml/imlztex9v1i) 打开右上三点，显示系统应用(安卓7-10测试均可使用)

- 或者先将 APP 打开，然后使用 ADB 命令查看当前界面的信息：（没试过）

```cmd&PowerShell
  xxx:/ $ dumpsys window | grep mCurrentFocus
    mCurrentFocus=Window{33613e4 u0 com.baidu.haokan/com.baidu.haokan.app.activity.HomeActivity}
```

- 输入以下命令可查询所有系统应用(如果已经`adb shell`了就不要输了啦)

  ```
  (adb shell) pm list packages -s
  ```

##   5、拿到包名之后，接下来就是卸载/停用应用了，命令如下：

  ```
(adb shell) pm uninstall -k --user 0 应用包名(卸载)
  ```
> `-k` 表示保存数据，如不需要，可去掉 `-k`。
> `--user` 指定用户 `id`，Android 系统支持多个用户，默认用户只有一个，`id=0`。

```
(adb shell) pm disable-user 应用包名(停用)
(adb shell) pm enable 应用包名(启用)
```

**记住，千万别**卸载**`com.android.xxx`和系统相关组件文件，你会后悔的(开不了机)** 建议要会刷机

~~~cmd&PowerShell
mido:/ $ pm uninstall -k --user 0 com.google.android.youtube
Success 成功删除
mido:/ $ pm uninstall -k --user 0 com.mi.android.globalpersoalassistant
Failure [not installed for 0] 没有找到应用，检查重试
~~~

> 至此，系统预置的应用就被卸载了。部分情况下，有可能在`设置 > 应用列表`中看到“`未针对此用户安装`”的字样，这个没有影响，重启一下就没有了。


```
mido:/ $ pm disable-user com.android.browser
Package com.android.browser new state: disabled-user
停用软件
```


---

内容转自 [ShawnXiaFei](https://www.jianshu.com/p/e9434e7f86ea) [玄青月](https://zhuanlan.zhihu.com/p/107371855) [Hanada](https://hanada.info/4376.html) 稍加修改
