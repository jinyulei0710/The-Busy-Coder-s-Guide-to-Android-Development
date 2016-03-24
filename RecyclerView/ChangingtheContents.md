###改变内容

之前样例的显而易见的问题是我们为没有响应用户在action mode item上的点击做任何事情。我们真的应该大写或移除单词。但是，这
就包含了修改模型数据和模型数据被RecyclerView展现的方式。


不是很明显的问题在action mode消失的时候我们调用了notifyDataSetChanged(),强制全部重新绘制RecyclerView的内容。虽然这可行，
但是有点小题大做，因为只有部分可见的项被选中了。理想的情况是，在actionmode完成且为未选中的时候，我们应该只更新我们当前选中的某些位置。
就像在single-choice list样例中一样，我们可以使用findViewHolderByPosition（）,找到收到影响的RowController实例。但是实际上，
更新选中状体是另一种大写或移除单词造成相同问题的表现形式，我们需要确保RecyclerView按理想的最小工作量去描述当前的模型状态。

####更新已存在的内容

使用基于BaseAdapte的AdapterView和Adapter,位置告诉AdapterView模型数据改变的方法是notifyDataSetChanged().这会触发整个
AdapterView的重建，是缓慢且昂贵的。

虽然RecyclerView.Adapter也有它自己的notifyDataSetChanged()，那只是用于重新加载全部模型数据的，例如从数据库获取新的游标或者不明确改变是什么的时候。
当你深入UI内部的变化，并且特别是如果你使用的模型数据是像ArrayList这样的模型对象的时候，你可以使用在RecyclerView上使用比notifyDataSetChanged()更具细粒度的方法。

如果某一项被更新了-例如单词大写-你可以在RecyclerView上使用notifyItemChanged()来指出特定位置已经改变了。可选方案包括:

* notifyItemMoved（）,来只是一个项还在模型数据中但是处于一个新的位置
* notifyItemRangeChanged(),指示一个范围内的位置已经修改了，而不是重复调用notifyItemChanged()

当用户大写单词的时候，ActionModeList2使用notifyItemChanged()，如果需要的话让这些项重新绘制。如果一个或多个当前在RecyclerView中不可见的话，可能不是立即需要。

但是目前位置，我们的模型数据一致是静态的字符串数组，并且我们需要一个可变的模型。所以使用和ListView action mode样例相同的方法，把静态数组转化为ArrayList。

MainActivity现在的成员变量是字符串数组，使用的是ORIGINAL_ITEMS这个静态数组：

    private static final String[] ORIGINAL_ITEMS={"lorem","ipsum","dolor",
     "sit","amet",
     "consectetuer","adipiscing","elit","morbi","vel",
     "ligula","vitae","arcu","aliquet","mollis",
     "emtiam","vel","erat","placerat","ante",
     "porttitor","sodales","pellentesque","augue","purus"
     };
    private ArrayList<String> items;

使用引用的项现在使用私有的getItems()方法来懒实例化列表：

    private ArrayList<String> getItems(){
        if(items==null){
          items=new ArrayList<String>();
          for(String s: ORIGINAL_ITEMS){
            items.add(s);
          }
        }
        return(items);
    }

我们同样需要在配置改变的时候持有这些项，因为这些项可能被用户改变。除了原本的让ChoiceCapableAdapter持久化选中状态之外    
，在MainActivity中的我们的onSaveInStanceState()和onRestoreInstanceState()方法来处理这个工作。

    @Override
    public void onSaveInstanceState(Bundle state){
      adapter.onSaveInstanceState(state);
      state.putStringArrayList(STATE_ITEMS,items)；
    }

    @Override
    public void OnRestoreInstance(Bundle state){
      adapter.onRestoreInstance(state);
      items=state.getStringArrayList(STATE_ITEMS);
    }

(这里，STATE_ITEM是一个静态的成员变量，作为Bundle入口的常量键值)

为了能大写化或移除列表中的选中的单词，我们需要知道哪一个选中了。为了不直接暴露数据，ChoiceMode现在有了一个
visitChecks()方法，通过这个方法给每个选中的位置提供一个Vistor:

    package com.commonsware.android.recyclerview.actionmodelist2;

    import android.os.Bundle;

    public interface ChoiceMode{
      void setChecked(int position,boolean isChecked);
      boolean isChecked(int position);
      void onSaveInstanceState(Bundle state);
      void onRestoreInstanceState(Bundle state);
      int getCheckedCount();
      void clearChecks();
      void visitChecks(Visitor v);
    }
     
MultiChoiceMode 通过遍历checkStates ParcelableSparseBooleanArray实现了visitChecks()。
这样的话，如果visitor修改了选中状态，我们的循环是不受影响的。

    @Override
    public void visitChecks(Vistor v){
      SparseBooleanArray copy=checkStates.clone();
      for(int i=0;i<copy.size();i++){
        v.onCheckedPosition(copy,keyAt(i));
      }
    }

visitChecks()和其它ChoiceMode中的方法一样，visitChecks()也由ChoiceCapableAdapter暴露的。

现在，IconicAdapter可以使用visitChecks（）大写化单词:
    
    case R.id.cap:
        visitChecks(new ChoiceMode.Vistor(){
           @Override
           public void onCheckedPosition(int position){
              String word=getItems().get(position);

              word=word.toUpperCase(Locale.ENGLISH);
              getItems().set(position,word);
              notifyItemChanged(position);
           }
        });
        break;
这里呢，对于每个选中的项，我们都对它进行了大写化，把原来的替换成对应的大写的，并且调用notifyItemChanged()来让RecyclerView知道
这个位置的模型数据变了，如果需要的话进行重新绘制。

在onDestroyActionMode()中我们也使用了visitChecks()，来避免调用notifyDataSetChanged（）:

    @Override
    public void onDestoryActionMode(ActionMode mode){
        if(activeMode!=null){
          activeMode=null;
          visitChecks(new ChoiceMode.Vistor（){
            @Override
            public void onCheckedPosition(int position){
              onChecked(position,false);
              notifyItemChanged(position);
            }
          });
        }
    } 

每个选中的都进行反选，并且使用notifyItemChanged()来确保需要时的重新绘制。
 
现在，选中一些项目并且选择action mode中的“CAPITALIZE”会把这些词大写化：

####添加和移除项

在RecyclerView.Adapter上也有你添加或从adapter移除项的时候调用的方法。不仅会造成RecyclerView的自我更新，如果相关的位置是可见的话也会有改变的动画。

具体的，你可以调用:

* notifyItemInserted(),来指示新的项被插入到特定的位置，表中其它的向后移动一个位置。
* notifyItemRangeInserted(),插入一些项到区间
* notifyItemRemoved(),来指示有一项从表中移除了，之后的项移到之前的位置
* notifyItemRangeRemoved()，移除某一个区间中的项

ActionModeList2样例使用notifyItemRemoved()作为处理action mode项目的一部分:
      
      case R.id.remove:
        final ArrayList<Integer> positions=new ArrayList<Integer>();

        visitChecks(new ChoiceMode.Visitor(){
          @Override
          public void onCheckedPosition(int position){
            positions.add(position);
          }
        });

        Collections.sort(positions,Collections.reverseOrder());

        for(int position:positions){
          getItems().remove(position);
          notifyItemRemoved(position);
        }

        clearChecks();
        activeMode.finish();
        break;

因为滑动的时候，移除项的时候项会补位，先移除最低的项是很重要的，然后自底向上。这就是这份代码写成这样的原因：

* 统计列表中选中的位置
* 对列表进行颠倒排序
* 遍历选中的项目】，从ArrayList中移除并调用notifyItemRemoved()来通知adapter原来位置的项没了
* 清除所有选中的项（因为所有的项都移除了）并且结束action mode(因为没有选中的项了)

结果是当用户移除项的时候，它们很快淡出，之后的向上滑动补位。如果你想使用其它动画，你也可以通过创建你自己的RecyclerView.ItemAnimator
并通过使用setItemAnimator（）附加。

