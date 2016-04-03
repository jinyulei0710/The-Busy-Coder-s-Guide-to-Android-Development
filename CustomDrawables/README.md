## 自定义`Drawables`

很多时候，我们的图片可以仅仅是一些PNG或JPEG文件，不同密度资源文件下各不相同。

尽管如此，有时候我们需要的会更多。

除了支持标准的PNG和JPEG文件之外，Android有很多自定义drawable资源格式－大部分是写在
XML中用来处理特定的情形。

例如。你想要自定义一个按钮的“背景",但是一个按钮真的有很多针对不同情形的不同背景图片（normal，
pressed,focused,disabled，等等.)。Android有着特定类型的drawable资源可以集合其它的drawable
资源，指示不同情形下该用什么资源（例如，normal按钮用x，disabled按钮用y)。

在本章中，我们会探索这些非传统类型的drawables并且告诉你如何在你的应用中使用。

目录

* [准备工作](https://github.com/jinyulei0710/The-Busy-Coder-s-Guide-to-Android-Development/blob/master/CustomDrawables/Prerequisites.md)
* 这些东西到哪里去了？
* ColorDrawable
* AnimationDrawable
* ColorDrawable
* AnimationDrawable
* StateListDrawable
* ColorStateList
* LayerDrawable
* TransitionDrawable
* LevelListDrawable
* `ScaleDrawable`和`ClipDrawable`
* InsetDrawable
* 矢量
* ShapeDrawable
* VectorDrawable
* BitmapDrawable
* 合成`Drawables`
* 小洞不补大洞吃苦
