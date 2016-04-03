####现在要做什么呢？

在网络访问那一章中，我们查看了一些从StackOverflow获取最新问题并展示在ListView的样例应用的变种。我们的QuestionsFragment有着一个填充ListView的ItemsAdapter，有着一个典型的getView()实现：

		class ItemAdapter extends ArrayAdapter<Item>{
		  ItemsAdapter(List<Item> items){
		    super(getActivity(),R.layout.row,R.id.title,items);
		  }
		  
		  @Override
		public View getView(int position,View convertView,
		    ViewGroup parent){
			View row=super.getView(position,convertView,parent);
			Item item=getItem(position);
			ImageView icon=(ImageView)row.findViewById(R.id.icon);

			Picasso.with(getActivity()).load(item.owner.profileImage)
			.fit().centerCrop()
			.placeholder(R.drawable.owner_profileImage)
			.error(R.drawable.owner_error).into(icon);

			TextView title=(TextView)row.findViewById(R.id.title);

			title.setText(Html.fromHtml(getItem(positoin).title));

			return(row);
		}
    }		  
 getView()中的一些部分是明显不同的，特别是使用Picasso来下载提问者的头像以及在标题中使用 Html.fromHtml()处理Html样式实体。
 
 但是在getView中使用的一般步骤是相当机械化的:
 
* 从行中获取控件
* 填充数据到行的控件中
* 作为数据绑定到行的一部分做以上两步进行更新

数据绑定,作为一个常规技术，旨在减少声明告诉framework如何从模型对象拉取数据的声明的机械化代码(例如，Item实例)并且把数据注入控件（例如，ImageView和TextView）。





