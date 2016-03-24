###可变的行内容
目前为止，我们所使用的所有的项都只是展示。最多，它们可能反馈下点击事件，和点击一个Listview和GridView的单元一样。

但是，选择模式是怎么样的呢？

ListView和GridView－以及它们共同的祖先AbsListView-有着选择模式的概念，用户可以选中或取消选中项，并且它们的列表或网格会保存这些状态。

因为有很多事情到涉及在RecyclerView中，RecyclerView并不提供选择模式...尽管如此，你可以自己实现它。RecyclerView/ChoiceList
样例代码就把我们列表样式的RecyclerView转化成一个核对列表，每一行都有一个CheckBox,RecyclerView.Adapter会为我们保存CheckBox的选中状态。

首先，我们需要在行中添加一个CheckBox：

  <?xml version="1.0" encoding="utf-8?"
  <android.support.v7.widget.CardView
    xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:cardview="http//shcemas.android.com/apk/res-auto"
    android:layout_width="match_parent"
    android:layout_height="wrap_content"
    android:layout_margin="4dp"
    cardview:cardCornerRadius＝"4dp">
    <LinearLayout
      android:id="@+id/row_content"
      android:layout_width="match_partent"
      android:layout_height="wrap_content"
      android:orientation="horizontal"
      android:background="?android:attr/selectableItemBackground">
      <ImageView
       android:id="@+id/icon"
       android:layout_width="wrap_parent"
       android:layout_height="wrap_content"
       android:padding="2dip"
       android:src="@drawable/ok"
       android:contentDescription="@string/icon"/>
       <LinearLayout
         android:layout_width="0dp"
         android:layout_height="wrap_content"
         android:layout_weight="1"
         android:orientation="vertical">
         <TextView
          android:id="@+id/label"
          android:layout_width="wrap_content"
          android:layout_height="wrap_content"
          android:textSize="25sp"
          android:textStyle="bold"/>
          <TextView 
           android:id="@+id/size"
           android:layout_width="wrap_content"
           android:layout_height="wrap_content"
           android:textSize="15sp"/>
        </LinearLayout>
        <CheckBox
          android:layout_width="wrap_content"
          android:layout_height="wrap_content"
          android:id="@+id/cb"
          android:layout_gravity="center_vertical"/>
        </LinearLayout>   
  </android.support.v7.widget.CardView>
    
我们的IconicAdapter 跟之前的只有细微的不同：

* 它从一个ChoiceCapableAdapter继承而来，我们稍后机会过以下，并且
* 它提供给ChoiceCapableAdapter一个MultiChoiceMode实例作为链接到ChoiceCapableAdapter构造器的一部分

   class IconicAdapter extends ChoiceCapableAdapter<RowController>{
    IconicAdapter(){
       super(new MultiChoiceMode());
    }
    
    @Override
    public RowController onCreateViewHolder(ViewGroup parent,int viewType){
      return(new RowController(this,getLayoutInfalter().infalte(R.layout.row,parent,false)));
    }

    @Override
    public void onBindViewHolder(RowController holder,int position){
       holder.bindModel(items[positoin]);
    }

    @Override
    public int getItemCount(){
       return(items.length);
    }

   }

ChoiceCapableAdapter只不过是一个知道如何处理选择模式的RecyclerView.Adapter，以实现ChocedMode接口的方式:

   	package com.commonsware.android.recyclerview.chocelist;

  	 import android.os.Bundle;

  	 public interface ChoiceMode{
      void setChecked(int position,boolean isChecked);
      boolean isChecked(int position);
      void onSaveInstanceState(Bundle state);
      void onRestoreInstanceState(Bundle state);
 	  }  

  ChoiceMode 是一个有效的策略类，负责记录选择状态，不只是当前的ChoiceCapableAdapter实例，还包括将来产生作为配置改变一部分的。它需要四个方法：

  * setChecked()和isChecked()是给定位置是否选中的getter和setter。
  * onSaveInstanceState()和onRestoreInstanceState()管理存储和恢复这些来自活动或者碎片的保存的实例状态Bundle.

这个项目使用了ChoiceMode的多选实现：
  
  	 package com.commonsware.android.recyclerview.choicelist;

  	 import android.os.Bundle;

 	 public class MultiChoiceMode implements ChoiceMode{
     private static final String STATE_CHECK_STATES="checkstates";
     private ParcelabelSparseBooleanArray checkStates=new ParcelableSparseBooleanArray();

     @Override
     public void setChecked(int position,boolean isChecked){
       checkStates.put(position,isChecked);
     }

     @Override
     public boolean isChecked(int position){
      return(checkStates.get(position,false));
     }

     @Override
     public void onSaveInstanceState(Bundle state){
       state.putParcelable(STATE_CHECK_STATES,checkStates);
     }

     @Override
     public void onRestoreInstanceState(Bundle state){
      checkStates=state.getParcelabel(STATE_CHECK_STATES);
     }
  }

MultiChoiceMode,反过来，大部分是通过一个ParcelabelSparseBooleanArray进行处理的。SparseBooleanArray是一个由AndroidSDK提供的类，是一个在空间方面十分高效对
int值到boolean值的映射，对应着使用HashMap把这些基本类型转换为Integer和Boolean对象。但是，由于令人费解的原因，SparseBooleanArray没有实现Parcelable,因此它不能
存在Bundle中。ParcelableSparseBooleanArray是SparseBooleanArray的子类，它可以处理Parcelable方面的东西：

 	 package com.commonsware.android.recyclerview.choicelist;

  	 import android.os.Parcel;
  	 import android.os.Parcelable;
  	 import android.util.SparseBooleanArray;

 	 public class ParcelableSparseBooleanArray extends SparseBooleanArray
       implements Parcelable{
        public static Parcelabel.Creator<ParcelableSparseBooleanArray> CREATOR
        =new Parcelable.Creator<ParcelableSparseBooleanArray>(){
          @Override
          public ParcelableSparseBooleanArray createFromParcel(Parcel source){
            return(new ParcelableSparseBooleanArray(source));
          }
          @Override
          public ParcelableSparseBooleanArray[] newArray(int size){
            return(new ParcelbleSparseBooleanArray[size]);
          }
        };

        public ParcelableSpaseBooleanArray(){
          super();
        }

        private ParcelableSparseBooleanArray(Parcel source){
          int size=source.readInt();

          for(int i=0;i<size;i++){
            put(source.readInt(),(Boolean)source.readValue(null));
          }
        }

        @Override
        public int describeContents(){
          return(0);
        }

        @Override
        public void writeToParcel(Parcel dest,int falgs){
          dest.writeInt(size());

          for(int i=0;i<size();i++){
            dest.writeInt(keyAt(i));
            dest.writeValue(valueAt(i));
          }
        }
        
       }

实际结果是多选模式可以借助ParcelableSparseBooleanArray,记录特定位置的选择状态。

ChoiceCapaleAdapter，然后就是一个可以显现选择模式实现的RecyclerView.ViewHolder:
  
 	 package com.commonsware.android.recyclerview.choicelist;

 	 import android.os.Bundle;
 	 import android.support.v7.widget.RecyclerView;

 	 abstract public class
        ChoiceCapableAdapter<T extends RecyclerView.ViewHolder>
        extends RecyclerView.Adapter<T>{
          private final ChoiceMode choiceMode;

          public ChoiceCapableAdapter(ChoiceMode choice){
            super();
            this.choice=choiceMode;
          }

          void onChecked(int position,boolean isChecked){
            choiceMode.setChecked(position,isChecked);
          }

          boolean isChecked(int position){
            return (choiceMode.isChecked(position));
          }

          void onSaveInstanceState(Bundle state){
            choiceMode.onSaveInstanceState(state);
          }

          void onRestoreInstanceState(Bundle state){
            choiceMode.onRestoreInstance(state);
          }
        }

由ChoiceCapableAdapter暴露出来的方法可以在外部使用。具体的，主活动把onSaveInstance()和onRestoreInstanceState()委托给ChoiceCapableAdapter。所以选中
状态可以掌握配置改变等等。此外，RowController可以勾住OncheckedChangedListener并更新基于checkbox状态改变的ChoiceCapableAdapter:

  	package com.commonsware.android.recyclerview.choicelist;

 	 import android.annotation.TargetApi;
 	 import android.os.Build;
 	 import android.support.v7.widget.RecyclerView;
 	 import android.view.MotionEvent;
  	 import android.view.View;
  	 import android.widget.CheckBox;
  	 import android.widget.CompoundButton;
  	 import android.widget.ImageView;
  	 import android.widget.TextView;
  	 import android.widget.Toast;

 	 class RowController extends RecyclerView.ViewHolder
       implements View.onClickListener,CompoundButton.OnCheckedChangeListener{
        private ChoiceCapableAdapter adapter;
        private TextView label=null;
        private TextView size=null;
        private ImageView icon=null;
        private String template=null;
        private CheckBox cb=null;

        RowController(ChoiceCapableAdapter adapter,view row){
          super(row);

          this.adapter=adapter;
          label=(TextView)row.findViewbyId(R.id.label);
          size=(TextView)row.findViewById(R.id.size);
          icon=(ImageView)row.findViewById(R.id.icon);
          
          template=size.getContext().getString(R.string.size_template);

          row.setOnclickListener(this);

          if(Build.VERSION.SDK_INT>=Build.VERSION_CODES.LOLLOPOP){
            row.setOnTouchListener(new View.onTouchListener){
              @TargetApi(Build.VERSION_CODES_LOLLIPOP)
              @Override
              public boolean onTouch(View v,MotionEvent event){
                v
                 .findViewById(R.id.row_content);
                 .getBackgound()
                 .setHotspot(event.getX(),event.getY());

                 return(false);
              }
            });
          }

          cb.setOnCheckedChangeListener(this);
        }

        @Override
        public void onClick(View v){
          Toast.makeText(v.getContext(),String.format("clicked on position %d",getAdapterPosition()),
          Toast.LENGTH_SHORT).show();
        }

        @Override
        public void onCheckedChanged(CompoundButton buttonView,boolean isChecked){
          adapter.onChecked(String.format(template,item.length()));
        }


        void bindModel(String item){
          label.setText(item);
          size.setText(String.format(template,item.length()));

          if(item.length()>4){
            icon.setImageResource(R.drawable.delete);
          }
          else{
            icon.setImageResource(R.drawable.ok);
          }

          cb.setChecked(adapter.isChecked(getAdapterPosition()));
        }
       }

这里，bindModel（）是基于ChoiceCapableAdapter给Recyclerview.ViewHolder的某一位置的isChecked()的值来更新CheckBox的（通过getPositon（）获取）。
onCheckedChanged()更新了ChoiceCapableAdapter来记录这个位置是否被选中，来处理行回收，配置改变等情况。

用户可以点击checkbox的时候checkbox会改变状态，这个状态在行回收，配置改变的时候都能存在。除此之外，用户界面没有什么变化。

![wizard](/the_busy_coder's_guide/img/figure_442.png)

注意，这个例子使用的Android5.0以上设备的Material主题，因为截屏是来自一个Android5.0的模拟器，CheckBox的样式是基于主色调的，这里的主色调是明黄色。

####转移到激活样式

也需要注意到ChoiceCapableAdpter,MultiChoiceMode及其亲属对用户是如何被通知什么选中了什么没被选中是茫然的。前一个例子中的RowController恰好使用了一个CheckBox。
RowController也可以使用其它空间，比如说Switch。

另一种方式就是使用激活状态。再一次的，这种事情也是由ListView及其选择模式自动处理的，但是只要一点调整，我们就能让我们的RowController使用这个方法。这会在RecyclerView／activatedList这个样例项目中展示。

首先，我们需要给我们的行的背景有一个支持激活状态的StateListDrawble。最简单的方法－也是Listview使用的传统方法－用固有的主题背景drawable设置一个激活样式，然后把样式应用到行上。

所以呢，这个样例应用就在res/values/styles.xml定义了activated：

	<?xml version="1.0" encoding="utf-8"
	<resource>

		<style name="Theme.AppTheme parent="@android:style/Theme.Holo.Light.DarkActionBar">
		</style>

		<style name="activated" parent="Theme.Apptheme">
		   <item name="android:background">?android:attr/activatedBackgroundIndicator</item>
		</style>   

	</resource>

注意，activated是从Theme.AppTheme继承而来的。这就意味着通常会得到Theme.Holo风格的背景，但是在API等级21以上，我们得到了Theme.Material风格的背景，由覆盖Theme.Apptheme的一个res/value-v21／styles.xml文件提供：

	<?xml version="1.0" encoding="utf-8"?>
	<resources>
		<style name="Theme.AppTheme" parent="android:Theme.Material.Light.DarkActionBar"
		<item name="android:colorPrimary">@color/primary</item>
		<item name="android:colorPrimaryDark">@color/primary_dark</item>
		<item name="android:colorAccent">@color/accent</item>
	    </style>
	<resources>    			

我们的行布局现在弃用了CardView（它的背景可能和激活样式背景冲突）并应用激活样式到根LinearLayout上：

	<?xml version="1.0" encoding="utf-8"?>
	<LinearLayout
	 xmlns:android:"http://shcemas.android.com/apk/res/android"
	 android:layout_width="match_parent"
	 android:layout_height="wrap_content"
	 android:orientation="horizontal"
	 style="@style/activated">
     
     <ImageView
      android:id="@+id/icon"
      android:layout_width="wrap_content"
      android:layout_height="wrap_content"
      android:layout_gravity="center_vertical"
      android:padding="2dip"
      android:src="@drawable/ok"
      android:contentDescription="@String/icon"/>
    
    	<LinearLayout
     	 android:layout_width="0dp"
     	 android:layout_height="wrap_content"
      	android:layout_weight="1"
      	android:orientation="vertical">

     	 <TextView
      	 android:id="@+id/label"
      	 android:layout_width="wrap_content"
      	 android:layout_height="wrap_content"
      	 android:textSize="25sp"
       	 andorid:textStyle="bold"/>
    
      	<TextView
       	 android:id="@+id/size"
       	 android:layout_width="wrap_content"
       	 android:layout_height="wrap_content"
      	 android:textSize="15sp"/>
	 	</LinearLayout>
	 </LinearLayout>

这个行也没有CheckBox了，因为它不再需要。

RowController 现在使用OnClicklistener接口来响应点击并使用它来切换行的激活状态：

	@Override
	public void onClick(View v){
		boolean isCheckedNow=adapter.isChecked(GetAdapterPosition());

		adapter.onChecked(getAdapterPosition,!isCheckedNow);
		row.setActivated(!isCheckedNow);
	}

setActivated(),应用到视图，指示它有没有激活，影响所有依赖于状态的所有东西，例如背景。

类似的，bindModel()使用setActivated()更新绑定数据时的激活状态：

	void bindModel(String item){
		label.setText(item);
		size.setText(String.format(template,item.length()));

		if(item.length()>4){
           icon.setImageResource(R.drawable.delete);
		}
		else{
			icon.setImageResource(R.drawable.ok);
		}

		row.setActivated(adapter.isChecked(getAdapterPosition()));
	}


其余都跟最初的checkbox样例一样。只不过，选中状态用激活高亮指示。

![wizard](/the_busy_coder's_guide/img/figure_445.png)

并且,因为这个demo是在Android 5.o上运行的，激活的高亮色时主色调，而这里是被设置成黄色的。

####但是，单选呢？

两个之前的例子阐述的都是多选的行为。尽管如此，单选行为是更好的选择。例如,在主从结构中，在双窗格模式中(例如，平板，主页和详细页都是可见的)，你可能就要一个
单选模式了。

这肯定是可能的，尽管如此，再一次的RecyclerView没有提供。它也带来了个小问题，在选中其它项的时候，如何反选之前选中的项。就像RadioGroup中的RadioButton控件一样，
我们需要确保同一时间只有一个选中的，这就需要我们更新之前选中的和现在没选中的项的用户界面。

更改一些地方之后，我们使用多选激活状态列表的样例项目，可以改成单选。这些修改会在RecyclerView/SingleActivatedList样例项目中阐释。

ChoiceMode接口现在有了两个新的方法：

* 1.isSingleChoice()在单选模式的时候返回true，其余返回false
* 2.getCheckedPosition()会返回当前选中项的位置

		package com.commonsware.android.recyclerview.singleactivatedlist;
		import android.os.Bundle;

		public interface ChoiceMode{
			boolean isSingleChoice();
			int getCheckedPosition();
			void setChecked(int position,boolean isChecked);
			boolean isChecked(int position);
			void onSaveInstanceState(Bundle state);
			void onRestoreInstanceState(Bundle state);
		}


SingleChoiceMode 是现在我们的ChoiceMode的实现:

		package com.commonsware.android.recyclerview.singleactivatedlist;

		import android.os.Bundle;

		public class SingleChoiceMode implements ChoiceMode{
			private static final String STATE_CHECKED="checkedPosition";
			private int checkedPosition=-1;

			@Override
			public boolean isSingleChoice(){
				return(true);
			}

			@Override
			public int getCheckedPosition(){
				return(checkedPosition);
			}

			@Override
			public void setChecked(int position,boolean isChecked){
				if(isChecked){
					checkedPosition=position;
				}
				else if(isChecked(position)){
					checkedPosition=-1;
				}
			}

            @Override
            public boolean isChecked(int position){
            	return(checkedPosition==position);
            }
           
			@Override 
			public void onSaveInstanceState(Bundle state){
				state.putInt(STATE_CHECKED,checkedPosition);
			}

			@Override
			public void onRestoreInstanceState(Bundle state){
				checkPosition=state.getInt(STATE_CHECKED,-1);
			}
		}

单选模式记录了当前选中的位置，用－1指示没有位置选中。需要注意的是，如果一个位置被选中了，使用setChecked()进行反选，SingleChoiceMode就会返回－1指示当前没有选中。

ChoiceCapableAdapter也有两处修改，首先它接受RecyclerView本身作为构造器参数，以一个rv成员变量就像持有。并且，onChecked()需要进行修改负责在新的项选中时移除之前选中的激活状态：

		package com.commonsware.android.recyclerview.singleactivatedlist;

		import android.os.Bundle;
		import android.support.v7.widget.RecyclerView;

		abstract public class ChoiceCapableAdapter<T extends RecyclerView.ViewHolder>
		      extends RecyclerView.Adapter<T>{
		      	private final ChoiceMode choiceMode;
		      	private final RecyclerView rv;

		      	public ChoiceCapableAdapter(ChoiceMode choiceMode,ChoiceMode choiceMode){
		      		super();
		      		this.rv=rv;
		      		this.choiceMode=choiceMode();
		      	}

		      	void onChecked(int position,boolean isChecked){
		      		if(choiceMode.isSingleChoice()){
		      			int checked=choiceMode.getCheckedPosition();

		      			if(checked>=0){
		      				RowController row=(RowController)rv.findViewHolderForAdapterPosition(checked);
		      				if(row!=null){
		      					row.setChecked(false);
		      				}
		      			}
		      		}
		      	}

		      	boolean isChecked(int position){
		      		return(ChoiceMode.isChecked(position));
		      	}

		      	void OnSaveInstanceState(Bundle state){
		      		choiceMode.onSaveInstanceState(state);
		      	}

		      	void onRestoreInstanceState(Bundle state){
		      		choiceMode.onRestoreInstanceState(state);
		      	}

		      	int getCheckedCount(){
		      		return(choiceMode.getCheckedCount());
		      	}
                 
                void clearChecks(){
                	choiceMode.clearChecks();
                }

                @Override
                public void onViewAttachedToWindow(T holder){
                	super.onViewAttachedToWindow(holder);

                	if(holder.getAdapterPosition!=choiceMode.getCheckPosition()){
                		((RowController)holder).setChecked(false);
                	}
                }

		      }
		
为了这么做，onChecked()问ChoiceMode它是不是单选的。如果是，它就获取了最后一个选中的。如果位置是合理的，它就通过findViewHolderForAdapterPosition()获取
RecyclerView.ViewHolder。如果它返回的不是null的话，它就是一个RowController，所以调用setChecked(false)的时候会移除激活状态。


findViewHolderForAdapterPosition()和findViewHolderForLayoutPosition（）代替了现在不赞成的findViewHolderForPosition（）方法。三个方法都做着相同基本的
一件事：给定一个位置，如果有的话返回这个位置的ViewHolder。至少目前来说，findViewHolderForAdapter()和findViewHolderForLayoutPosition()有着相同的实现。
findviewHolderForAdapterPosition最大的不同是在数据改变的时候总是返回空（例如，notifyDataSetChanged()被调用了。）但是这些改变还没有铺放。在这个样例中，差别是学理上的,但是在大都数用例中findViewHolderForAdapter()可能是更安全的选择。

但是呢，这些find...（）方法有个问题，它们只在行可见的时候返回ViewHolder。如果ViewHolder被缓存了，但不可见的话，find...（）仍旧不会返回它。这就导致了一个问题，
当我们需要反选一个不可见行的时候,是不使用onBindViewHolder重绑定的。（因为ViewHolder已经设置好了).这就需要我们实现onViewAttachedToWindow（）-在ViewHolder内容真正作为子视图副驾到RelativeLayout上是调用，并且更新选择状态作为反馈。


setChecked()在之前的样例中不存在，因为激活状态完全由RowController内部处理。所以呢，现在RowController现在有个setChecked()方法来切换激活状态：

	void setChecked(boolean isChecked){
		row.setActivated(isChecked);
	}

MainActivity现在必须在onCreate()中把RecyclerView供给IconicAdapter:

	IconicAdapter(RecyclerView rv){
		super(rv,new SingleChoiceMode());
	}


视觉上来说，除了一次只能选中一个之外，结果是跟之前的样例一模一样的。关键的是至多这个短语，这个实现允许用户轻触一个选中的并进行反选。这可能很好，因为你的应用在这个情形中
简单的隐藏了细节，仍旧允许用户和action bar项交互。(例如一个“创建新的模型”项)。如果你想阻止这个，当用户轻触之情选中的项不要让SingleChoiceMode把checkedPosition
设为－1，保持当前选中的位置不同。

####Action Modes

ListView给予我们还有就是支持action modes。特别的，“multiple－choice modal"设置会自动开始和结束一个action mode。

再一次的，RecyclerView没有action modes的钩子。尽管如此，你可以自己去实现。我们要手动启动和销毁aciton mode，此外还要反馈aciton mode上用户的交互。（轻触项，或者手动退去action mode)

变得有趣的是选中项和action mode之间联系。这里有两个用户体验规定：

1. 没有项选中的时候，应该没有action mode

2.没有action mode的时候，应该没有选中项

你可能认为，这两个规定是一样的，在某种程度上来说它们是的。这样措辞是为了强调状态改变的复杂性：

* 当用户选中一个项，action mode应该出现
* 当用户反选最后一个选中的项，因此没有更多选中的项了，aciton mode应该消失
* 当用户退去action mode(例如，按下返回键)，所有的项应该反选

处理这些转变是需要花费一点时间的，正如在RecyclerView／ActionModeList中所展示的那样。这是早前ChoiceList样例的一个复制版，当有一个项选中的时候就出现
action mode。action mode的逻辑大部分是从这本书的action 磨的样例中复制过来的，在这个样例中，我们允许用户大写或移除选中的项。

再一次的，我们对ChoiceMode做了一些修改，添加了两个方法：

1. getCheckedCount（）,返回选中的项的数量，被用于作为action mode的副标题

2.clearChecks()，反选所有选中的项

	package com.commonsware.android.recyclerview.actionmodelist;

	import android.os.Bundle

	public interface ChoiceMode{
		void setChecked(int position,boolean isChecked);
		boolean isChecked(int position);
		void onSaveInstance(Bundle state);
		void onRestoreInstanceState(Bundle state);
		int getCheckedCount();
		void clearChecks();
	}

MultiChoiceMode 实现了这些，外加对setChecked（）的细微修改。之前版本的MultiChoceMode只是简单地把状态的布尔值放在ParcelableSparseBooleanArray中，默认为false。
现在我们移除了没有选中的项，所以ParcelableSparseBooleanArray中只有选中的项。这使得getCheckedCount()和clearChecks()实现变得很简单：

	package com.commonsware.android.recyclerview.actionmodelist;

	import android.os.Bundle;

	public class MultiChoiceMode implements ChoiceMode{
		private static final String STATE_CHECKS_STATES="checkStates";
		private ParcelableSparseBooleanArray checkStates=new ParcelableSparseBooleanArray();
        
        @Override
        public void setChecked(int position,boolean isChecked){
        	if(isChecked){
        		checkStates.put(position,isChecked);
        	}
        	else{
        		checkStates.delete(position);
        	}
        }

        @Override
        public boolean isChecked(int position){
        	return(checkStates.get(position,false));
        }

        @Override
        public void onSaveInstanceState(Bundle state){
        	state.putParcelable(STATE_CHECK_STATES,checkStates);
        }


        @Override
        public void onRestoreInstanceState(Bundle state){
        	checkStates=state.getParcelable(STATE_CHECK_STATES);
        }

        @Override
        public int getCheckedCount(){
        	return(checkStates.size());
        }

        @Override
        public void clearChecks(){
        	checkStates.clear();
        }

	}


ChoiceCapableAdapter暴露了两个新的ChoiceMode功能给它的子类

    package com.commonsware.android.recyclerview.actionmodelist;

    import android.os.Bundle;
    import android.support.v7.widget.RecyclerView;

    abstract public class 
         ChoiceCapableAdapter<T extends RecyclerView.ViewHolder>
         extends RecyclerView.Adapter<T>{
          private final ChoiceMode choiceMode;

          public ChoiceCapableAdapter(ChoiceMode choiceMode){
            super();
            this.choiceMode=choiceMode;
          }

          void onChecked(int position,boolean isChecked){
            choiceMode.setChecked(position,isChecked);
          }

          boolean isCheck(int position){
            return (choiceMode.isChecked(position));
          }

          void onSaveInstanceState(Bundle state){
            choiceMode.onSaveInstance(state);
          }

          void onRestoreInstanceState(Bundle state){
            choiceMode.onRestoreInstanceState(state);
          }

          int getCheckCount(){
            return(choiceMode.getCheckCount());
          }

          void clearChecks(){
            choiceMode.clearChecks();
          }

         }


IconicAdapter现在不仅继承了ChoiceCapableAdapter,它也实现了ActionMode.Callback接口，负责管理aciton mode:

    class IconicAdapter extends ChoiceCapableAdapter<RowController>
          implements ActionMode.Callback{

IconicAdapter现在重载了onChecked()，通常只由ChoiceCapableAdapter处理。除了链接父类的标准行为，IconicAdapter还负责管理action mode：

  * 如果我们选择一个项（isChecked is true），并且我们没有正在进行的action mode,使用startActionMode
  * 如果我们有选中的项，并且我们已经有了action mode了，只需要更新副标题，因为我们选中项的数据变了。
  * 如果我们没有任何选中项，并且我们有一个激活的action mode,结束这个action mode，因为反选最后的一个选中的时候，action mode 就不需要了。

		@Override
		void onChecked(int position,boolean isChecked){
		  super.onChecked(position,isChecked);

		  if(isChecked){
		     if(activeMode==null){
		       activeMode=startActionMode(this);
		     }
		     else{
		        updateSubTitle(activeMode);
		     }
		  }
		  else if(getCheckedCount()==0&&activeMode!=null){
		      activeMode.finish();
		  }
		} 

因为IconicAdapter实现了ActionMode.Callback,它需要通过接口实现方法。包括:

* onCreateActionMode(),设置action mode
* onPrepareActionMode()，仅仅因为它被接口需要
* onActionItemClicked(),真正需要做工作的地方，但是现在呢先写个TODO注释
* onDestroyActionMode(),确保所有的项不选中(clearChecks())并告诉RecyclerView.Adapter数据已经改变了，强制
  重新绘制所有的可见行，会影响不再选中的。

 		@Override
 		public boolean onCreateActionMode(ActionMode mode,Menu menu){
 		   MenuInfalter infalter=getMenuInflater();

           infalter.infalte(R.menu.context,menu);
           mode.setTitle(R.string.context_title);
           activeMode=mode;
           updateSubtitle(activeMode); 

           return (true);
        } 
        @Override
        public boolean onPrepareActionMode(Action mode,Menu menu){
           return (false);
        }

        @Override
        public boolean onActionItemClicked(ActionMode mode,MenuItem item){
           //TODO:do something based on the action

           updateSubTitle(activeMode);
           return(true);
        }

        @Override
        public void onDestoryActionMode(ActionMode mode){
          if(activeMode!=null){
             activeMode=null;
             clearChecks();
             notifyDataSetChanged();
          }
        }

 updateSubTilte()方法，引用之前的一些方法，只更新了action mode的副标题来反映
 当前选中的项的数量：

        private void updateSubtitle(ActionMode mode{
           mode.setSubTitle("("+getCheckedCount()+")");
        }

应用结果跟原本的ChoiceList样例代码大部分一样，直到我们选中或更多的项的时候，action mode 出现了：


![wizard](/the_busy_coder's_guide/img/figure_446.png)
