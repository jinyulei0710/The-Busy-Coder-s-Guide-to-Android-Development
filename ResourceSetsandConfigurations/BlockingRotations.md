###阻碍旋转

无疑你已经看过一些Android应用只不过会忽略所有旋转屏幕的尝试。许多游戏就是这么做的，完全在横屏
模式下操作，不管设备是如何朝向的。

为了这么做，添加android:screenOrientation="sensorLandscape",或者也许可能是android：
screenOrientation="sensorPortrait"到你的manifest中去。这些名字的“sensor”部分，指示你的
应用能够在常规或者这个方向的旋转版(例如，“常规”横屏是设备从竖直状态逆时针旋转90度，同时翻转版横屏是
从竖屏顺时针旋转90度)。在API等级18以上，你可以换而使用userLandscape或者userPortrait,因为
这些会尊敬用户的系统级别的选择是否要锁定屏幕，如果没有锁定屏幕默认为sensorPortrait或sensorLandscape的
行为。

理想情况下，你选择landscape，因为一些设备(例如Android电视)，只能是水平的。

也要注意到Android仍旧把它当成一个配置改变，不管对用户可不可见。因此，你仍旧需要使用之前所提及到的技术
中的一种去处理这个配置改变和其它改变(例如，底座事件和，区域改变)。
