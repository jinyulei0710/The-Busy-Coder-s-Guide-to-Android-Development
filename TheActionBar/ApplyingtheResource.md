###应用资源

从你的activity中，你通过重载一个onCreateOptionsMenu()方法教导这些动作栏项，例如这个来自ActionBar/ActionBarDemoNative样例项目的ActionBarDemoActivity:

	@Override
	public boolean onCreateOptionsMenu(Menu menu){
	    getMenuInflater().inflate(R.menu.actions,menu);
	    
	    return(super.onCreateOptionsMenu(menu));
	}

这里，我们会创建一个MenuInflater然后告诉他inflate我们的菜单XML资源(R.menu.actions)然后把它们注入到提供的Menu对象中。我们然后链接到父类，返回它的结果。

	
	