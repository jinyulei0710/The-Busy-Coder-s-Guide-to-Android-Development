###回收activity们

让我们假定我们有三个activity，分别叫做A,B和C.A基于一些用户输入启动了一个B的实例，并且稍后B通过一些更多的
用户操作启动了C的一个实例。

我们的“activity栈”现在是A-B-C，意味着如果我们从C按下返回键，我们就返回到了B，并且如果我们从
B按下了返回键，我们返回到了A。

现在，让我们假定我们想从C导航回到A。例如，可能用户按下我们动作栏左侧的图标，我们想要返回到主界面activity，在我们的情形中就是A。如果C调用了startActivity(),指定了A，我们就以A-B-C-A这个activity栈告终了。

那是因为启动一个activity时，默认会创建一个新的这个activity的实例。那么，现在我们就有了两份独立的A拷贝。

有时候，这是所期望的行为。例如，我们可能有单个ListActivit通过层次数据集向下钻取数据，就像一个目录树。我们可能
选择保持启动同一个ListAcivity,但是带有不同的`extras`,来展示每一个层级。在这种情形中，我们会想要activity的独立实例，那样返回按钮就表现的就和用户所预期的一样。

但是，当我们导航回主界面activity，我们可能不会想要一个单独的A实例。

如何解决这个问题和导航到A之后的activity栈的样子有一点关系。

如果你想要的activity栈是B-C-A-那么已存在的A拷贝被带到了前台，但是B和C的实例就被留下了-然后你可以在使用
startActicity()的时候添加FLAG_ACTIVITY_FEORDER_TO_FRONT到你的Intent。

	Intent i=new Intent(this,HomeActivity.class);
	i.setFlags(Intent.FLAG_ACTIVITY_REORDER_TO_FRONT);
	startActivity(i);

取而代之的，如果你想要的activity站仅仅是A-那么如果用户按下了返回键，他们就离开了你的应用-那么你会添加两个
标记：FLAG_ACTIVITY_CLEAR_TOP和FLAG_ACTIVITY_SINGLE_TOP:

	Intent i=new Intent(this,HomeActivity.class);
	i.setFlags(Intent.FLAG_ACTIVITY_REORDER_TO_FRONT|Intent.FLAG_ACTIVITY_SINGLE_TOP);
	startActivity(i);
	
这会结束栈中位于当前activity和你所启动的activity之前的activity，在我们的情形中，结束了C和B。

	

