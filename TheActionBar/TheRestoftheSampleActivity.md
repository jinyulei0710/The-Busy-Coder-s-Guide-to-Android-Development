###样例Activity的其余部分

那么，什么是我们真正在ActionBarActivity中做的呢？

在很多方面，这让人回想起前一章回的ListActivity样例。我们有着由25个拉丁单词组成的数组，并且
我们想把它们展示在列表中。

但是，在此例中，我们最初的时候只是展示5个单词。一个“add”动作栏项会添加25个清单中的单词的另外部分，直到ListView持有所有的25个。一个“reset”动作栏按钮会返回给我们最初的5个单词。

ActionBarDemoActivity是一个ListActivity。但是，我们把这个工作委托给了一个initAdapter()方法而不是像其它样例做的那样直接写在onCreate()方法中。再者，initAdapter()方法所做的工作和其它样例所做的又有一点不同：

    private void initAdapter(){
      words=new ArrayList<String>();
      
      for(int i=0;i<5;i++){
        words.add(items[i]);
      }
      
      adapter=new ArrayAdapter<String>(this,android.layout.simple_list_item1,
                          words);
                          
      setListAdapter(adapter);                    
    }
    
我们创建了一个新的ArrayList并把5个来自项的元素注入其中，而不是直接使用静态的items数组。然后在ArrayList之上创建ArrayAdapter。这看起来是多余的，但是我们的动作栏项可以很好地利用这个方式。  

当用户点击动作栏上的"reset"项的时候，我们再次调用initAdapter()，这个方法给予我们的ListActivity新的一组用来展示的5个拉丁单词：
    
      @Override
      public boolean onOptionsItemSelected(MenuItem item) {
        switch(item.getItemId()) {
         case R.id.add:
          addWord();

          return(true);

        case R.id.reset:
          initAdapter();

          return(true);

       case R.id.about:
        Toast.makeText(this, R.string.about_toast, Toast.LENGTH_LONG)
             .show();

        return(true);
    }

    return(super.onOptionsItemSelected(item));
    }

当用户点击动作栏上的“add”项的时候，我们可以调用一个私有的addWord()方法，这个方法会添加items数组的接下来的单词并将其附加到ListView上：

      private void addWord(){
        if(adapter.getCount()<items.length){
            adapter.add(items[adapter.getCount()]);
        }
      }  
     
最终结果是我们有了一个带有我们自定义动作栏的activity:

在我们的动作栏中有一个“about”，它会一直处于溢出菜单中。这会有三种视觉结果。

首先，在没有屏幕下方菜单键的设备上，溢出菜单是以"..."按钮展现的，它会在被点击的时候展现溢出菜单：

在有着屏幕下方菜单键的Android 4.x设备上，点击菜单键会使溢出菜单从屏幕底部出现：

Android 4.4+设备应该一直有着"..."按钮，正如接下来这一节描述的。

Android 2.3设备运行这个应用会没有动作栏:

但是，按下菜单按钮的时候回带来旧式的选择按钮，其中会出现我们的动作项:




     