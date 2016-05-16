###选择正确的资源

假定你有一个有着N个不同定义的资源，Android是如何选择一个来使用的呢？

首先，Android弃用掉那些明确非法的。所以，例如，如果设备的语言是-ru,Android会忽略
指定为其它语言(例如，-zh)的资源集。这个弃用有个例外-密度限定和屏幕尺寸限定——我们稍后
会了解这些例外。

然后，Android选择那个有着要求的资源和最重要唯一限定符的资源集。这里，通过”最重要“
我们意味着基于以上所讨论的目录命名规则，出现在目录名最左侧的那个。并且，通过“唯一”，
我们意味着没有其它资源有这个限定符。

如果没有对应的特定资源集，Android会选择默认那套——那套资源的目录名上没有后缀(例如，res/layout/)。

记住了这些规则，让我们看看一些场景，除了基本情况之外还涵盖上述的例外。

####场景#1:一些简单的
让我们假定我们有一个main.xml文件存在于：
* res/layout-land/
* res/layout/

当我们调用setContentView(R.layout.main),如果设备是处于横向模式的话，Android会选择
位于res/layout-land/的main.xml。在此例中这个特定的资源集是合法的，并且它有着最重要的
唯一限定符(-land)。如果设备是处于纵向模式，res/layout-land/资源集就不能胜任了,并且所以
它就被丢弃了。这就给我们剩下了res/layout/,所以Android使用了这个版本的main.xml。

####场景#2:毫不相干的资源集种类
尽管很奇怪，有可能main.xml位于：

* res/layout-en/
* res/layout-land/
* res/layout/

在此例中，如果设备的地域被设置成了英语，不管设备的方向是什么Android会选择
res/layout-en/。这是因为-en是一个更为重要的资源集限定符——在来自Android文档的“表2.配置限定符名称”中
”语言和地域“比"屏幕方向"的优先级要来的高。不过如果设备没有设置成英语，Android会丢弃掉这个
资源集，就这一点来说决策过程是和以上的场景#1一样的。

####场景#3:多样的限定符

让我们设想有一个项目，其中main.xml存在于：

* res/layout-en/
* res/layout-land-v11/
* res/layout/

你可能认为res/layout-land-v11/是所选择的，因为它更加明确，对比其他资源集
的一个或没有来说，它对应两个资源限定符。

尽管如此，在此例中，语言是比屏幕方向或者Android API 等级更为重要的，所以
决策过程是和以上的场景#2类似的：Android对于英语会选择res/layout-en/,
对于水平的API等级11以上的设备会选择res/layout-land-v11,又或者
对于所有其它的会选择res/layout/。

####场景#4:再访多样的限定符
让我们改变资源的混搭，现在我们有一个项目，其中带有main.xml存在于:

* res/layout-land-night/
* res/layout-land-v11/
* res/layout/

这里，虽然-land是最重要的资源限定符，但不是唯一的——我们有着
不止一个带有-land的资源集。因此，我们需要去核对次重要的资源集限定符。
在此例中，次重要的就是-night,因为夜间模式是比AndroidAPI等级更为重要的类别，
所以如果设备是处于夜间模式的时候,Android会选择res/layout-land-night。
否则的话，如果设备运行在API等级11或更高之上它会选择res/layout-land-v11/。
如果设备不是处于夜间模式并且也没有运行在API等级11或者更高的版本之上，
Android会选择res/layout/。

####场景#5:屏幕密度
现在，让我们看下规则的首个例外:屏幕密度。

Android总是会接收包含屏幕密度的资源集，即使它与设备的密度不相配。当然，
如果有一个精确的密度匹配，Android会使用它。否则的话，它会基于它与设备的实际密度以及
是否有其它比设备的实际密度高或低的设备来使用它觉得次好的匹配。

原因是对于drawable资源来说，Android会向下或者向上自动采样图片，
所以drawable会是正确的大小，即使你没有在某一密度中提供图片。

隐情是双方面的：

1.Android应用这个逻辑到了所有的资源，而不仅仅是drawable,所以
即使没有精确匹配的密度，比如说,一个layout,Android仍旧会从另一个密度的
存储桶中来为layout进行选择

2.作为上一步的副作用，如果你包含了一个密度资源集限定符，Android会忽略
所有低优先级的资源集限定符

所以，现在让我们假装我们的项目的main.xml位于:

* res/layout-mdpi/
* res/layout-nonav/
* res/layout/

Android会选择res/layout-mdpi/,即使对于没有“非接触式导航方法”的-hdpi设备。
虽然-mdpi并不和-hdpi匹配，Android仍旧会选择-mdpi。如果我们要对
drawable资源进行处理，Android会向上采样-mdpi图片。

####场景#6:屏幕尺寸

如果你有着和屏幕尺寸绑定的资源集，Android会选择最接近实际屏幕尺寸而依然比
实际尺寸小的。比实际尺寸大的屏幕尺寸的资源集是被忽略的。

对于所有设备来说，这适用于-swNNNdp,-wNNNdp,以及-hNNNdp。在-large或者
-xlarge的设备上，Android对传统的屏幕尺寸限定符(-small,-normal,-large,
-xlarge)应用了相同的逻辑。但是对于-small或-normal设备Android没有应用相同
的逻辑——一个-normal设备不会加载一个-small资源。

现在让我们假装我们的项目有main.xml位于:

* res/layout-normal/
* res/layout-land/
* res/layout/

如果设备不是-small的，Android会选择res/layout-normal/。否则的话，
如果设备是横向的，Android会选择res/layout-land。如果所有其它的都失败了，
Android会选择res/layout/。

相似的，如果我们有：

* res/layout-w320dp/
* res/layout-land/
* res/layout/

对于当前屏幕宽度是320dp或者更高的设备，Android会选择res/layout-w320dp/。
否则的话，如果设备是横向的，Android会选择res/layout-land。如果所有其它的都失败了，
Android会选择res/layout/。
