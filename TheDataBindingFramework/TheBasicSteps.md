###基本步骤

考虑到这一点，让我们查看需要做什么让这个样例转而使用Google的数据绑定系统。

这节展示的样例代码来自[DataBing/Basic](https://github.com/jinyulei0710/cw-omnibus/tree/master/DataBinding/Basic)这个样例项目。

###设置工具链

数据绑定只在1.3及以上的更高的版本能够正常使用并且推荐使用1.5.0或以上的Gradle Android插件。

数据绑定系统由两部分组成：另一个Gradle插件以及一个和我们应用捆绑在一起的库。但是，我们不需要手动设置他们。相反，我们只要告诉Gradle Android插件我们想要数据绑定，那么它就把必要的插件和类库为我们添加了。

所有我们需要添加的是一个小小的dataBinding闭包,在其中我们设置enabled属性为true:

        apply plugin: 'com.android.application'

		dependencies {
 		   compile 'org.greenrobot:eventbus:3.0.0'
 		   compile 'com.squareup.picasso:picasso:2.5.2'
  		   compile 'com.squareup.retrofit2:retrofit:2.0.0'
           compile 'com.squareup.retrofit2:converter-gson:2.0.0'
		}

		android {
    		compileSdkVersion 23
    		buildToolsVersion "23.0.2"

  			defaultConfig {
       			 minSdkVersion 15
       			 targetSdkVersion 23
       			 versionCode 1
       			 versionName "1.0"
    		}

   			 dataBinding {
       			 enabled = true
    		}
		}
		
一旦你做了这个，将来你打开项目的时候Androidstdio可能引起你得到一个“Source folders generated at incorrect location”消息。

这是一个bug,时间充足的话可能被修复。但是这个bug看起来是良性的，并且它们不会造成你应用的任何问题。

####在布局和模型中增加

真正的乐趣从我们ListView行的布局开始。原本的版本的这个布局资源是一个有ImageView和TextView的LinearLayout:

    <?xml version="1.0" encoding="utf-8"?>
    <LinearLayout
    xmlns:android="http://schemas.android.com/apk/res/android"    
    android:layout_width="match_parent"
    android:layout_height="wrap_content"
    android:orientation="horizontal">
    <ImageView
     android:id="@+id/icon"
     android:layout_width="@dimen/icon"
     android:layout_height="@dimen/icon"
     android:layout_gravity="center_vertical"
     android:contentDescription="@string/icon"
     andorid:padding="8dip"/>
     <TextView
      android:id="@+id/title"
      android:layout_width="wrap_content"
      android:layout_height="wrap_content"
      android:textSize="20sp"
      android:layout_gravity="left|center_vertical"/>
    </LinearLayout>  
为了实现数据绑定我们需要做一些改变:

     <?xml version="1.0" encoding="utf-8"?>
     <layout xmlns:android="http://schemas.android.com/apk/res/android”>
     <data>
      
      <variable
       name="item"
       
       type="com.commonsware.android.databind.basic.Item"/>
     </data>
       <LinearLayout
        android:layout_width="match_parent"
        android:layout_weight="wrap_content"
        android:layout_gravity="center_vertical"
        android:contentDescription="@string/icon"
        android:padding="8dip"
       
       <ImageView
        android:id="@+id/icon"
        android:layout_width="@dimen/icon"
        android:layout_height="@dimen/icon"
        android:layout_gravity="center_vertical"
        android:contentDescription="@string/icon"
        android:padding="8dip"/>
        
        <TextView
        android:id="@+id/title"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_gravity="left|center_vertical"
        android:text="@{item.title}"
        android:textSize="20sp"/>
       </LinearLayout>
       </layout> 

首先，整个资源文件是包在一个<layout>元素中的，在这个元素中我们可以放置android命名
空间声明。    

<layout>元素有着两个孩子。第二个孩子是我们的LinearLayout,代表资源的根视图或视图组。首个孩子是一个<data>元素,并且这个是我们配置当这个布局被使用的时候数据绑定如何被处理的地方。特别的，<variable>元素指示我们想要从一个Item对象绑定数据到定义在这个布局的控件。

然后，如果你看下TextView,你会看到它现在有了一个android:text属性，这是原本的布局资源所缺乏的。更重要的，android:text的值是不寻常的:@{item.title｝。@{}表达是指示不要像文本字符串一样进行解析，也不是字符串资源的引用，这个值是真正的一个表达式，用数据绑定表达语言表示的，它应该在运行时吧被计算出来然后赋值给TextView的text。

在这个例子中，表达式是item.value。item是Item对象在<variable>元素中被给定的名字。
任何我们想要从Item对象拉取数据的地方，我们可以使用.标志来引用item表达式语言变量上的东西。

item.value意味着"从item获取value"。在运行时，数据绑定类库会尝试从Item类上通过公开的gettter方法(getValue)或公开的域（值）来获取这个值。原本的项目有一个value域，但是它不是公开的，所以修改项目把Item域标为公开，那么我们就能在数据绑定中使用它们了：

       package com.commonsware.android.databind.basic;
       
       public calss Item{
          public String title;
          public Owner owner;
          public String link;
          
          @Override
          public String toString(){
             return(title);
          }
       }

正如我们在这章所看到的，这里所使用的表达式语言比仅仅引用对象上JavaBean样式的属性复杂多了，但是目前来说，它是满足要求的。

####应用绑定

布局配置了绑定的一个方面:拉取数据到控件。我们仍然需要做一些工作来配置绑定的另一方面：提供数据的来源。在这个样例的情形中，我们需要提供Item对象给这个布局资源。

这个是通过对原本版本ItemsAdapter的getView()方法进行一些修改而被处理的：

	@Override
    public View getView(int position, View convertView, 
      ViewGroup parent) {
      RowBinding rowBinding=
        DataBindingUtil.getBinding(convertView);

      if (rowBinding==null) {
        rowBinding=
          RowBinding.inflate(getActivity().getLayoutInflater(),
            parent, false);
      }

      Item item=getItem(position);
      ImageView icon=rowBinding.icon;

      rowBinding.setItem(item);

      Picasso.with(getActivity()).load(item.owner.profileImage)
             .fit().centerCrop()
             .placeholder(R.drawable.owner_placeholder)
             .error(R.drawable.owner_error).into(icon);

      return(rowBinding.getRoot());
    }		
 这里有四个改变：我们创建了绑定，提供模型（Item）给绑定，从绑定获取其它控件，并且获取布局的根视图。
####创建绑定

当我们在布局资源中使用<layout>并设置数据绑定系统的布局方面，构建系统代码生成了与这个布局文件相关联的Java类。类名是来源于布局名的，其中name_like_this转化成NamesLikeThis并附加了Binding后缀。所以，因为我们的布局文件是row.xml，我们获得了RowBinding。这是代码生成的在mainifest包名下的数据绑定分包下的。因此，完全限定的这个类的导入声明是：

	import com.commonsware.android.databind.basic.databing.RowBinding;

这是ViewDataBinding的子类，是由databing类库提供的。它是通过在你的build.gradle文件中开启数据绑定而添加到你的项目中的。

创建一个binding实例也infalte了相关联的布局文件。你的绑定类有着大量的工厂方法来infalteing布局并创建绑定。这些映射了你在其它地方使用过的其它方法。

* setContentView(),取得一个Activity和布局资源ID作为参数，inflates布局，在Activity传递结果给setContentView()，并创建绑定。
* inflate()，有着多个可选的参数列表，只要使用LayoutInfalter()infaltes布局，并创建绑定

这里我们使用的是三个参数风格的inflate(),它把LayoutInfalter(从宿主activity获得的)，父容器以及false作为参数。这个映射了使用layoutInfalter本身infalte()那种，除此之外它也给了我们绑定。

当然，这是一个ListView,所以我们要处理可能的行回收问题。DataBindingUtil类有着一个getBinding()方法，它会为给定的来自infalted布局的视图返回绑定。在这里案例中，我们的converView。所以，我们先尝试获取已存在的绑定，然后在必要的情况下退回去inflating一个新的。因为getBinding为convertView完全处理了空值，我们不需要自己特地再去做空值检查。

####注入模型到绑定

生成的绑定类有布局中我们的<data>元素中的每个<variable>的setters.Setter名词是使用标准的JavaBean约定生成的，所以我们的item变量变成了setItem()。当我们调用setItem()的时候，数据绑定系统会使用Item对象来填充我们的TextView，应用来自我们android:text属性的绑定表达式。

####从绑定取得控件

但是，我们没有做任何与布局中ImageView相关的数据绑定的事(虽然，之后在这章中我们会)。因此，我们仍旧需要手动去做，获取Picasso来异步获取头像并把它放进ImageView.

但是，这表明我们有了ImageView。正常情况下，我们会在infalted的布局的根视图上来获取。

但是，我们的绑定类有着给布局中的每个有着android:id值的控件（至少对于有@id/...和@+id/....值的）代码生成的公开域。我们的ImageView有着一个值为@+id/icon的android:id,所以RowBinding类有了一个持有我们ImageView的icon域。我们可以仅仅引用它，而不是我们自己做findViewById()查找。

####获取真实的视图

因为getView()是假定返回inflated的布局的根视图的，我们需要某种方式从绑定获取这个。幸运的是，ViewDataBinding有着一个getRoot()方法，这个方法我们的生成类也继承了，所以我们能仅仅调用这个就可以获取值从getView()返回。

####结果

看起来的话，这个应用和之前是一样的(虽然这个版本在兼容的设备上使用的是Theme.Material)。功能上，这个应用是和之前一样的。并且从代码复杂性角度，这个应用可能比之前还差，因为仅仅为了避免多次调用findViewById()和调用setText()一次，我们做了很多工作。

因此，虽然数据绑定系统很好，它真的只为较大的项目增色，特别是那些有着复杂布局的。在这章结尾的时候，你应该对什么时候数据绑定是有益的什么时候是小题大作有了更好的认识。

		
