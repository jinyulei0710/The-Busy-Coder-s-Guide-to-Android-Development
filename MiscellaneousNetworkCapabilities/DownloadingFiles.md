###下载文件

Android 2.3引进了一个DownloadManager,是为处理大量下载较大文件的复杂性问题而设计的,例如：

1. 获取用户使用的是WiFi还是移动蜂窝数据信息，知道之后决定下载该不该发生

2. 处理用户，之前在Wifi环境，然后移动到无限访问节点范围之外之后，在移动蜂窝数据上不成功的情况

3. 确保设备在下载过程中保持唤醒状态

DownloadManager 本身比你自身写代码去处理以上这些问题会简单点。但是，它有一些挑战性。在这节中，我们会
检视[Internet/Download](https://github.com/commonsguy/cw-omnibus/tree/master/Internet/Download)这个使用DownloadManager的样例项目。

####权限

为了使用DownloadManager，你会需要持有INTERNET权限。你也会需要WRITE_EXTERNAL_STOREAGE权限，因为
DownloadManager只能下载到外部存储。即使你尝试让DownloadManager在某个可能不需要权限的地方写入（例如，
Android 4.4＋设备上的getExternalFilesDir()）。DownloadManager需要你持有权限，Android framework更是如此，
并且目前，在所有API等级上DownloadManager都需要权限。

例如，这里是Internet/Download 应用的manifest,在里面呢我们请求了这两个权限:

####TODO