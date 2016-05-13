###你的配置改变选项

注意，配置改变在你的activity上是相当侵入性的，会用全新的内容进行替换(虽然可能一些来自旧activity
控件的信息会搬运到新的activity的控件上)。

因此，在任意给定的activity中，你有多种处理配置改变的方法。

####什么也不做

当然，最容易做的事情，是什么也不做。如果所有你的状态是交由Android自动处理的
除了默认的之外你不需要在做任何事。

例如来自前一章回的ViewPager/Fragment样例能很好地开箱即用。我们所有的状态
都是被绑定在EditText控件上的，其中是Android自动处理的。所以，我们能在一堆这些
控件上输入东西，然后选择屏幕(例如，通过Windows或linux PC上模拟器中的<Ctrl>-<F11>)，
并且我们输入的文本被保留了。

不幸的，有很多情形中内置的行为要么是不完整的要么直接是错误的，并且我们需要做更多地工作来确保
我们的配置改变被很好地处理了。

####保留你的fragment

一个处理这些类型配置改变的方法是让Android保留一个动态的fragment。

这里，“retain”意味着在配置改变是Android会保有相同的fragment实例，把它从
最初的宿主activity上分离开来，然后附加到新的宿主activity上去。因为它是相同的fragment实例，
所有包含在实例中的东西都由它自己保留了，因此当activity销毁和重建的时候没有丢失。

为了在行动中去看，来看下ConfigChange/Fragments这个样例项目。

这个demo的业务逻辑(以及对于这章中所有其它的demo来说)，我们要的是允许用户从他们设备的默认器的
联系人清单中选择一个联系人。用户按下“选择”按钮，我们会展示一个会让用户选择联系人并返回结果
给我们。然后，我们会启用一个”View"button,让用户浏览所选联系人的细节。关键是我们所选的联系人
在配置改变的时候会被保留-否则的话当用户旋转屏幕的时候activity看起来会忘记所选的联系人。

activity本身仅仅是加载动态的fragment，以下是之前在这本书出现过的代码片段:

      package com.commonsware.android.rotation.frag;

      import android.app.Activity;
      import android.os.Bundle;

      public class RotationFragmentDemo extends Activity {
        @Override
          public void onCreate(Bundle savedInstanceState) {
            super.onCreate(savedInstanceState);

            if (getFragmentManager().findFragmentById(android.R.id.content)==null) {
              getFragmentManager().beginTransaction()
                                 .add(android.R.id.content,
                                      new RotationFragment()).commit();
                    }
            }
      }    

检查fragment存在与否的原因现在应该清楚一些了。因为在配置改变的时候Android会自动重建(或保留)
我们的fragment，当我们已经有了拷贝的时候我们不会想要去创建相同fragment的第二个拷贝。

fragment要使用有着两个实现方式的R.layout.main的布局资源。其中一个在res/layout-land/中，
会在横屏时被使用：

      <?xml version="1.0" encoding="utf-8"?>
      <LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
        android:orientation="horizontal"
        android:layout_width="match_parent"
       android:layout_height="match_parent"
        >
        <Button android:id="@+id/pick"
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        android:layout_weight="1"
        android:text="@string/pick"
        android:enabled="true"
        />
        <Button android:id="@+id/view"
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        android:layout_weight="1"
        android:text="@string/view"
        android:enabled="false"
        />
        </LinearLayout>

竖屏版本，是在res/layout/中,和LinearLayout的方向完全相同:

        <?xml version="1.0" encoding="utf-8"?>
        <LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
        android:orientation="vertical"
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        >       
        <Button android:id="@+id/pick"
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        android:layout_weight="1"
        android:text="@string/pick"
        android:enabled="true"
        />
        <Button android:id="@+id/view"
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        android:layout_weight="1"
        android:text="@string/view"
        android:enabled="false"
        />
        </LinearLayout>

这里是RotationFragment的完整实现:

      package com.commonsware.android.rotation.frag;

      import android.app.Activity;
      import android.app.Fragment;
      import android.content.Intent;
      import android.net.Uri;
      import android.os.Bundle;
      import android.provider.ContactsContract;
      import android.view.LayoutInflater;
      import android.view.View;
      import android.view.ViewGroup;

      public class RotationFragment extends Fragment implements
      View.OnClickListener {
        static final int PICK_REQUEST=1337;
        Uri contact=null;

        @Override
        public View onCreateView(LayoutInflater inflater, ViewGroup parent,
                   Bundle savedInstanceState) {
                     setRetainInstance(true);

                     View result=inflater.inflate(R.layout.main, parent, false);

                     result.findViewById(R.id.pick).setOnClickListener(this);

                     View v=result.findViewById(R.id.view);

                     v.setOnClickListener(this);
                     v.setEnabled(contact != null);

                     return(result);
                   }

                   @Override
                   public void onActivityResult(int requestCode, int resultCode,
                       Intent data) {
                         if (requestCode == PICK_REQUEST) {
                           if (resultCode == Activity.RESULT_OK) {
                             contact=data.getData();
                             getView().findViewById(R.id.view).setEnabled(true);
                           }                          
                         }
                       }

                       @Override
                       public void onClick(View v) {
                         if (v.getId() == R.id.pick) {
                           pickContact(v);
                         }
                         else {
                           viewContact(v);
                         }
                       }

                       public void pickContact(View v) {
                         Intent i=
                         new Intent(Intent.ACTION_PICK,
                           ContactsContract.Contacts.CONTENT_URI);

                           startActivityForResult(i, PICK_REQUEST);
                         }

                         public void viewContact(View v) {
                           startActivity(new Intent(Intent.ACTION_VIEW, contact));
                         }
                       }

在onClick()中，我们连接“选择”按钮到了一个pickContact()方法。在那里，我们使用了一个ACTION_VIEW
Intent去调用startActivityForResult()，指示我们想从ContactsContract.Contacts.CONTENT_URI
联系人集合中选择一些东西。我们之后会在这本书中更为详细地讨论ContactsContract。就目前来说，
记住Android有着这样的ACTION_PICK activity，这个activity能给用户展示可用的联系人：

除了ACTION_PICK Intent之外，我们也提供了一个唯一的int值给startActivityForResult()。
这个int应该是个16位的值(0-65535),这对于这个来自activity的特定的startActivityForResult()
调用来说是唯一的。这个值和你的其它activity比较时不需要是唯一的，更不用说整个设备。

如果用户选择了一个联系人，控制权返回到我们的activity，调用我们的onActivityResult。onActivityResult
就通过了：

* 我们提供给startActivityForResult()的唯一ID是为了帮助识别我们可能从其它地方接收这个结果
* 如果用户选择了一个联系人就返回RESULT_OK，如果用户丢弃了选择的activity就返回RESULT_CANCELED
* Intent包含了来自选择activity的结果，在此例中，这个结果包含了一个代表
所选择联系人的Uri，通过getData()获取。

我们保存这个Uri到数据成员中，外加启用了“查看”按钮，当这个按钮按下的时候，
会在所选择的联系人上通过它的Uri唤起一个ACTION_VIEW activity：

在onCreateView()上，我们调用了setRetainInstance(true)。这告诉Android在配置改变的时候
保留这个fragment。因此，我们可以在竖屏模式中选择一个联系人，然后旋转屏幕(例如，通过Windows或
linux PC上模拟器中的<Ctrl>-<F11>)，并且在横屏模式中查看联系人。即使activity和按钮因为旋转
被取代了，fragment没有，并且fragment保留着所选联系人的Uri。

这项技术胜于其他的优点是，你可以保留有任意你想要的数据种类，
任意数据类型，任何大小，等等。但是，这个方法不能再你的进程被终止之后把
状态返回给你，例如用户通过最近任务列表返回你的应用。

#####Model Fragment

在这个主题上的一个变种是“model fragment”。虽然fragment通常来说关注的是提供UI的一部分给
用户，但这不是必要的。model fragment仅仅是一个使用了setRetainInstance(true)来确保
它在配置改变的时候会被保留。fragment然后就持有了它的宿主activity所需的所有模型数据，所以
当activity被销毁和重建的时候，模型数据保留在model fragment中。

这对于没有fragment作为家的数据尤其有用。例如，想象一个activity，它的UI是完全由
一个ViewPager组成的，(像教程应用一样)。即使可能ViewPager会持有fragment，大都数
翻页中都会有很多页。添加一个独立的没有UI的fragment，并且让它为ViewPager持有activity
的模型数据。这仍然允许activity被销毁和重建，甚至允许ViewPager被销毁和重建的同时，仍旧保留有
已经加载的数据。

在使用常规的fragment是，Google途径使用model fragment而不是使用setRetainInstance(true)。
保留的fragment持有的越少，你持有你不应该持有的可能性就越小，例如由于一个可能的
区域改变一个字符串需要重新从字符串资源加载。这就是说，如果你很小心并确保你的数据
成员是被合理占用的，从任意的fragment使用setRetainInstance(true)就是安全的。

####添加到Bundle

如果你坚持不止要维护配置改变的情形，也要维护进程终止的情形，你会要使用onSaveInstanceState()
和onRestoreInstanceState()。

你可以在你的activity中重载onSaveInstanceState()。它传入了一个Bundle,你可以把数据存储到
里面，它能配置改变的时候保存住。美中不足的是虽然Bundle有点像HashMap，它是不能持有任意的数据类型的，
这就限制了你所能通过onSaveInstanceState()保存的信息种类。onSaveInstanceState()大约是在
onPause()和onStop()时被调用的。

控件的状态是由Android通过内置的onSaveInstanceState()自动维护的。如果你自己进行重载，
通常会想要链接到父类得到它的继承行为，除此之外你自己要把东西放到Bundle中去。

Bundle在两个地方传回给你:

* onCreate()
* onRestoreInstanceState()

因为除了由于配置改变之外onCreate()在很多情形下都会被调用，传入的Bundle经常是空的。
另一方面，onRestoreInstanceState()只在Bundle被使用的时候被调用。

要看这个是如何工作的，来看下ConfigChange/Bundle这个样例项目。

这里，rotationBundleDemo是有着和之前样例相同核心业务逻辑的activity。
因为activity会在配置改变的时候销毁和重建，我们重载onSaveInstanceState()和
onRestoreInstanceState()来保存我们的联系人，如果已经有个联系人在配置改变
之前被选中了的话：

    @Override
    protected void onSaveInstanceState(Bundle outState) {
      super.onSaveInstanceState(outState);

      if (contact != null) {
        outState.putParcelable("contact", contact);
      }
    }

    @Override
    protected void onRestoreInstanceState(Bundle state) {
      super.onRestoreInstanceState(state);

      contact=state.getParcelable("contact");
      viewButton.setEnabled(contact != null);
   }

这里我们使用putParcelable()把Uri放到Bundle中去。putParcelable是
一个你能在一个Java类上实现的接口，这个接口能允许你的类实例放入到一个Bundle中去。
Uri恰好实现了Parcelable。Parcelable的全部细节和你如何能在东西实现Parcelable在
本书后面的内容提供了。

这个方法的缺点是不是所有的东西都能到Bundle中去的。Bundle不能持有任意的数据类型，
所以例如你不能传入一个Socket到Bundle中去。同样的，这个Bundle需要是相对较小的，
因为它是跨进程的，所以你不能把大的对象(例如bitmap)放进Bundle。对于这些情形，
你会不得不使用保留的fragment方法解决。

####Fragment和Bundle

Fragment也有一个它们能重载的onSaveInstanceState()方法。使用方法适合activity一样的-
你可以存储数据到提供的Bundle上，这个Bundle后来会提供回给你。最大的不同是不存在onRestoreInstanceState()
方法-你要在其它生命周期方法中处理：

* onCreate()
* onCreateView()
* onViewCreated()
* onActivityCreated()

你可以在ConfigChange/FragmentBundle这个样例项目中看到这个。这是之前两个样例的混合的结果，
fragment但是使用的是onSaveInstanceState()而不是setRetainInstance(true)。

我们的RotationFragment现在有了一个onSaveInstanceState()方法，
和来自ConfigChange/Bundle样例的activity的那个activity很像：

      @Override
      public void onSaveInstanceState(Bundle outState) {
        super.onSaveInstanceState(outState);

        if (contact != null) {
          outState.putParcelable("contact", contact);
        }
        }

我们的onCreateView()检验了传入的Bundle，并且如果它是非空的
话就尝试从中获取我们的联系人：

      @Override
      public View onCreateView(LayoutInflater inflater, ViewGroup parent,
                  Bundle state) {
                    View result=inflater.inflate(R.layout.main, parent, false);

                    result.findViewById(R.id.pick).setOnClickListener(this);

                    View v=result.findViewById(R.id.view);

                    v.setOnClickListener(this);

                    if (state != null) {
                      contact=(Uri)state.getParcelable("contact");
                    }

                    v.setEnabled(contact != null);

                    return(result);
                  }

这没有像setRetainInstance(true)所做的那样，允许我们的activity持有任意的数据。
但是，使用activity层面的onSaveInstanceState()，onSaveInstanceState()处理了
多个保留的fragment不会处理的场景，例如由于低内存你的进程被终止了，用户之后依然能使用
BACK返回应该处于你的activity(以及它的fragment)的内容。

####DIY

在少数几个情形中，即使保留的fragment也是不够的，因为调动和重新应用状态会变得太繁杂或者太慢。或者，在一些情形中，硬件便会阻碍，例如当尝试使用相机拍照的时候-一个我们之后会在这本书中涵盖到的概念。

如果你是完全孤注一掷的，你会告诉Android不要在配置改变的时候销毁和重建activity...尽管它有着它自己的一系列结果。为了做这个:

* 放一个android:ConfigChanges入口到你们的AndroidManifest文件中去，列出你想要自己处理的配置改变而不是让Android为你进行处理
* 在你的Activity中实现onConfigurationChanged()，当其中一个
你列在android:ConfigChanges中的配置改变发生的时候回被调用

现在，对于所有你要的配置改变，你可以忽略整个activity的析构过程并且仅仅获取一个回调让你知道改变。

例如，让我们看下ConfigChange/DIY这个样例代码。

在AndroidManifest.xml中，我们添加android:ConfigChanges到了<activity>元素中去，指示我们要自己处理一些配置改变：

    <activity
    android:name="RotationDIYDemo"
   android:configChanges="keyboardHidden|orientation|screenSize"
   android:label="@string/app_name">
   <intent-filter>
     <action   android:name="android.intent.action.MAIN"/>

     <category android:name="android.intent.category.LAUNCHER"/>
     </intent-filter>
     </activity>

这个的许多代码片段会让你自己处理orientaion和keyboardHidden。
但是现在，你也需要吹屏幕尺寸(并且，理论上来说，是最小屏幕尺寸),
如果你把你的android:targetSdkVersion设置成13或者更高的版本。记住这需要你的build target设置成13或者更高的版本。

因此，对于这些特定的配置改变来说，Android不会销毁并重建activity，而是会调用onConfigurationChanged()。在RotationDIYDemo实现中，仅仅厨房了LinearLayout的方向来匹配
设备的方向：

       @Override
       public void onConfigurationChanged(Configuration newConfig) {
         super.onConfigurationChanged(newConfig);

         LinearLayout container=(LinearLayout)findViewById(R.id.container);

         if (newConfig.orientation == Configuration.ORIENTATION_LANDSCAPE) {
           container.setOrientation(LinearLayout.HORIZONTAL);
         }
         else {
           container.setOrientation(LinearLayout.VERTICAL);
         }
       }

因为activity没有在配置改变的时候被销毁，我们完全没有必要担心所选的联系的Uri-它没有去任何地方。

这个实现的问题主要有两方面:

1.我们没有处理所有可能的配置改变。如果用户把设备放到汽车充电座上，Android会销毁并重建我们的
activity，并且我们会丢失我们所选中的联系人。

2.我们可能忘记一些由于配置改变需要被改变的资源。例如，如果我们开始翻译被布局所使用的
字符串，并且我们包含locale到android:configChanges中，我们不仅需要更新LinearLayout
也要更新Button控件的说明文字，因为Android不会自动为我们处理。

这就是为什么除非万不得已Google才推荐使用这个技术的两个原因。

也要记住，当你的进程被终止，用户通过最近任务列表返回你的应用，这个方法对于保存状态也是完全
没有帮助的。
