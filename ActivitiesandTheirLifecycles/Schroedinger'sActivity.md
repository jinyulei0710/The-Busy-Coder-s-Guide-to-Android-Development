###薛定谔的Activity

一个activity，通常来说，在任何时间点会处于以下四种状态中的一种:

1.Active: activity被用户启动了，正在运行，并且处于前台。这是你用来思考你activity的操作的时候。

2.Paused: activity被用户启动了，正在运行，并且是可见的，但另外有个activity覆盖了部分屏幕。
在这个期间，用户可以看见你的activity但是可能不能和它进行交互。这是一个相对不常见的状态，因为大都数activity
是被设置成全屏的，没有一个让它们看起来像某种对话框的主题。

3.Stopped:activity被用户启动了，正在运行，但是它被其他启动的或切换到的activity挡住了。

4.Dead:活动被杀死了，可能是因为用户按下了返回按钮。

