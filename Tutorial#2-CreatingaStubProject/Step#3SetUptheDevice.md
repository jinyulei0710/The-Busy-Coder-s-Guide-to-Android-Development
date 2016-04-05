###步骤＃3-设置设备
你在入门开发Android应用开发的时候你是不需要Android设备的。在你尝试发布一个应用之前，拥有一个则是一个很好的主意。（例如，把它上传到市场）。并且，可能你已经有一个设备了－可能就是它激励你开发Android的兴趣。

如果你没有一个Android设备的话，跳过这一步骤。

让设备准备好被开发使用的第一步是进入设置上的应用设置。现在发生的些许取决于你的Android版本：

* 在Android 1.x/2.x上，进入应用，然后进入开发
* 在Android 3.0 到 Android 4.1上，从设置页面的主页进入“开发者选项”。
* 在Android 4.2以及更高版本上，进入关于，点击版本号7次，然后按下返回，进入开发者选项(之前被隐藏了)

你可能要滑动右上角的switch按钮到“打开”的位置来修改这屏上的值。

通常，你会要打开USB调试，然后你能让你的设备使用Android构建工具。如果你希望的话你可以把其它设置先放到一边，即使你可能很方便找到“保持唤醒”选项，它节省了你的手机在插入USB的时候所有解锁的时间。

注意，在Android 4.2.2以及更高版本的设备上，在你能使用你刚刚切换的设置之前，你以通过对话框的方式被提示允许USB在你的特定的开发机器上进行调试：

这发生在你插好USB线并已经正确设置驱动的时候。这一过程因你的开发机器的操作系统而不同，如接下来章节所涵盖的。

####Windows
当你首次插入你的Android设备的时候，Windows会尝试找到与之对应的驱动。如果可能的话，由于其它你已经安装的软件，设备已经能使用。如果它发现了一个驱动，那么你就可能已经准备好了。

如果驱动没有找到的话，这有获取驱动的一些选项。

#####Windows 更新

一些Windows版本（例如，Vista）会提示你用Windows更新搜索驱动。这的确值得一试，尽管不是每一个设备都会有供给微软的驱动。

#####标准Android驱动

在你安装Android SDK的过程中，如果你选择安装通过sdk管理器安装“Google USB驱动”，你会发现一个extras/google/usb_driver/目录，它包含了Android设备的通用Windows驱动。你可以尝试点击这个目录中驱动安装向导来看看驱动是否是合适你的设备的。这通常会在Nexus设置上有效。

#####供应厂商驱动

如果你没有驱动，开发者文档中的OEM UBS Drivers可能会对你从你的设备产商一个可下载的驱动有帮助。注意你可能需要知道你的设备的机型编号，而不是用于市场的机型名（例如GT-P3113而不是“Samsung Galaxy Tab 2.7.0”）。

####OS X和linux

最有可能的事只要插上你的设备就可以了。你可以通过在shell中(例如，OS X终端)运行adb devices看到Android有没有识别你的设备。adb在你的sdk目录中的platform-tools目录。如果你得到了与如下类似的输出，构建工具侦测到了你的设备:

	List of devices attached
    HT9CPP809576

如果你运行的是Ubuntu(或这其它Linux变种)，这个命令不会有效，你可能要添加一些udev规则。例如，这就有一个51-android.rules文件，它会处理来自一个厂商的设备。

	SUBSYSTEM=="usb",SYSFS{idVendor}=="0bb4",MODE="0666"
	SUBSYSTEM=="usb",SYSFD{idVendor}=="22b8",MODE="0666"
	SUBSYSTEM=="usb",SYSFD{idVendor}=="18d1",MODE="0666"
	SUBSYSTEM=="usb",ATTRS{idVendor}=="18d1",ATTRS{idVendor}	=="0c01",MODE="0666",OWNER="[me]"
	SUBSYSTEM=="usb",SYSFS{idVendor}=="19d2",SYSFS{idProduct}	=="1354",MODE="0666"
	SUBSYSTEM="usb",SYSFS{idVendor}=="04e8",SYSFS{idProduct}=="681c",MODE="0666"    
    
清除Ubuntu上你的/etc/udev/rules.d下的内容，然后要么重启电脑要么重新加载udev规则（例如,sudo service udev reload）。

CyanogenMod项目维护了一个wiki页面，上面有这些udev规则，其中包含了来自大陆产商和设备的规则。   

