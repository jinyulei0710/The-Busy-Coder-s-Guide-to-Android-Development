###API版本控制的资源

就像之前在这章中指出的，-vNNN系列后缀指示了这个目录中的资源是针对规定的API等级或者更高的等级的。
所以,例如，res/values-v21/指示的是这个目录中的资源应该只能在API等级21(Android 5.0)和更高的上使用。
运行较老Android版本的设备会忽略这些资源。

这对于处理主要的Android版本变更是一个重要的系列后缀。固有的Android应用在API等级11(Android 3.0)
和API等级21(Android 5.0)的时候，观感上发生了巨大的变化。你可能发现在这些API等级分界点你开始有了
不同的资源，所以你的UI在所支持的所有的Android版本上看起来都是恰如其分的。

####用例:API等级的主题
这个特性的一个重大的用例是以API等级区分不同主题。

即使你的`minSdkVersion`是11或者更高的版本，你的应用中可能存在两个不同的主题:

* 一个是基于`Theme.Holo`的，被使用在API等级11-20
* 另一个是基于`Theme.Material`的，从API等级21起开始使用

你粗暴的替代方案是使用`action bar`的`appcompat-v7`向后兼容方案和一些`Material Design`审美。
对于高度程式化的应用来说，或者你确定要在Android 5.0之前的设备上使用`Material Design`,`appcompat-v7`
是值得考虑的。但是如果你想要更好地在每个主要的UI变种上进行混合，你会想要在Android 3.x和4.x上支持
Theme.Holo，并且在之后的版本上使用Theme.Material。

这里的繁重工作是设置你的主题，就像`action bar`那章概述的那样。让它们同时可用，基于
于设备版本来说，仅仅是一个把资源放进合适的目录中的问题。

例如，看下ActionBar/VersionedColor这个样例项目。这是HoColor和MaterialColor样例项目
的混合，基于API等级决定使用哪个主题。

在res/values/目录中，我们有一个和在HoloColor样例中一样的styles.xml文件，指示
名字标准化成了styles.xml。它使用了一个由Action Bar样式生成器生成的自定义主题
(Theme.Apptheme)。

也存在一个指示values资源在API等级和更高版本上使用的res/values-v21/目录。
它有着最初在MaterialColor样例中看到的主题，其中主题资源被重命名成了Theme.Apptheme,
来对应定义在res/values/的那个。

然后，<application>引用Theme.Apptheme,我们就在正确的设备上得到了正确的
`action bar`。

这里，让主题资源名称保持一致是很重要的，因为我们在<application>元素中引用了这个名称。
为了能拉取正确的那个，我们需要它们都是同样的名称。但是，至于这些主题所引用的资源，例如颜色
或者drawable资源，进不进版本控制的目录，就随你瞧着办了。如果你想要多个版本，其中API等级
选择使用哪个版本的话，它们必须到版本控制的目录中并且要有相同的名称。

例如，定义在res/values-v21/styles.xml的基于Theme.Material的主题引用了三个颜色资源。
这些资源的文件(colors.xml)恰好也在res/values-v21/中。但是，因为我们没有基于API等级
去放置这些颜色，放置在res/values/的colors.xml也能很好的工作。并且，如果我们不想要
因API等级的不同而有不同的颜色，我们需要这些颜色都定义在相关的资源集中，例如在res/values和
res/values-v21/。
