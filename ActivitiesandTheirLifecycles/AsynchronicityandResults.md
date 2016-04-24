###同步和结果

注意,startActivity()是同步的。在你的返回应用主线程控制权给Android之后，另一个activity才会显示出来。

通常来说，这不是很大的问题。但是，有些时候一个activity可能启动了另一个，其中第一个activity想知道一些来自
第二个activity的结果。例如，第二个activity可能是某种选择，允许用户选择一个文件或一个联系人或一首歌或这个其他的，
并且第一个activity需要知道用户选择了什么。使用同步的startActivity()，我们明显是不可能从startActivity()本身获取某种类型的返回结果。

为了处理这个情形，还有一个单独的startActivityForResult()方法。虽然它依然是同步的，它允许新启动的activity提供一个结果(通过一个setResult()方法)，这个结果通过onActivityResult()方法传送到了最初的activity。我们会在之后一章中更为详细地分析startActivityForResult()。



