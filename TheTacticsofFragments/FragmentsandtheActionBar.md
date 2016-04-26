###碎片和动作栏

碎片能通过从onCreate()调用setHasOptionsMenu(true)把项添加到动作栏中。这向activity指示它需要在fragment上
调用onCreateOptionsMenu()和onOptionsItemSelected()。

Fragment/ActionBarNative样例应用展示了这个。这和来自动作栏那章的ActionBar/ActionBarDemoNative样例有着相同的功能。

在onCreate()中，我们调用了了setHasOptionsMenu(true)，指示我们有兴趣参与到动作栏中去：

    @Override
    public void onCreate(Bundle savedInstance){
        super.onCreate(savedInstanceState);
        
        setRetainInstance(true);
        setHasOptionsMenu(true);
    }
    
  (我们将会稍后一章讨论setRetainInstance(true)的调用）
  
这会触发我们fragment的onCreateOptionsMenu()和onOptionsItemSelected()方法在适当的时间呗调用：

	 @Override
    public void onCreateOptionsMenu(Menu menu, MenuInflater inflater) {
    inflater.inflate(R.menu.actions, menu);

    super.onCreateOptionsMenu(menu, inflater);
    }

    @Override
    public boolean onOptionsItemSelected(MenuItem item) {
     switch(item.getItemId()) {
       case R.id.add:
        addWord();

        return(true);

      case R.id.reset:
        initAdapter();

        return(true);
     }

     return(super.onOptionsItemSelected(item));
    }  
    
这里，我们从R.menu.action菜单XML资源初始化了我们的动作栏，包括设置我们的EditText控件，外加
响应重置动作栏溢出项的逻辑。

我们的activity不用做任何特殊的事情来允许fragment为动作栏出力-它只是设置了动态碎片:
		
		package com.commonsware.android.abf;

        import android.app.Activity;
        import android.os.Bundle;

         public class ActionBarFragmentActivity extends Activity {
            @Override
            public void onCreate(Bundle savedInstanceState) {
                 super.onCreate(savedInstanceState);

             if (getFragmentManager().findFragmentById(android.R.id.content) == null) {
             getFragmentManager().beginTransaction()
                             .add(android.R.id.content,
                                 new ActionBarFragment()).commit();
            }
          }
       }    