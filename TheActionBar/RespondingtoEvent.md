###响应事件

为了发现什么时候用户点击了这些东西，你会需要重载onOptionsItemSelected(),例如如下展示的ActionBarDemoActivity实现：
    
      @Override
      public boolean onOptionsItemSelcted(Menu Item){
         switch(item.getItemId()){
            case R.id.add:
             addWord();          
             return(true);
            
            case R.id.reset:
              initAdapter();
              return(true);
              
            case R.id.about:
               Toast.makeText(this,R.string.about_toast,Toast,LENGTH_LONG)
                .show();
                
               return(true);
           }
           return(super.onOptionsItemSelcted(item));               
      }
    
你会传入一个MenuItem。你可以在它上面调用getItemId()，并将这个值和来自你菜单XML资源的值进行比对(R.id.add和R.id.reset)。如果你处理了这个事件，让它返回true，否则的话，返回链接到父类的这个方法实现的值。

如果你想要响应动作栏左侧的你的应用图标的点击事件，将getItemId()和android.R.id.home进行比较，因为这个就是特别的动作栏按钮所使用的MenuItem。注意到这一点，如果你把你的android:targetSdkVersion设置成14或者更高的版本，你也会需要在ActionBar(通过调用getActionBar获取)调用setHomeButtonEnabled（true）来开启这个行为。注意，这个图标可能不存在，特别是当你使用的是Android 5.0+的Theme.Material的时候。




    