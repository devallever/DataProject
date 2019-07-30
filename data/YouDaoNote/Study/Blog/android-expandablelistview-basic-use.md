---
title: Android ExpandableListView基础用法
date: 2017-08-09 16:28:24
tags:
 - Android
 - ExpandableListView
categories: [Android]
---

# ExpandableListView简介
可以折叠的ListView

# 基本使用
使用方法和ListView大同小异,都是需要item的布局,需要adapter
But:
ExpandableListView需要父Item布局,需要子Item布局.Adapter需要的是BaseExpandableListAdapter
下面以一个好友分组的例子总结一下用法

## 布局
分组Item布局
```
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
     android:layout_width="match_parent"
    android:layout_height="wrap_content"
    android:padding="10dp"
    android:background="@color/color_green_50">
    <RelativeLayout
        android:layout_width="match_parent"
        android:layout_height="wrap_content">
        <ImageView
            android:id="@+id/id_friendgroup_item_parent_iv_expand"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:src="@mipmap/ic_unexpanded"/>
        <TextView
            android:id="@+id/id_friendgroup_item_parent_tv_friendgroup_name"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:layout_marginLeft="10dp"
            android:layout_toRightOf="@id/id_friendgroup_item_parent_iv_expand"
            android:text="我的好友"
            android:layout_centerVertical="true"
            android:textSize="18sp"
            android:textColor="@color/colorPrimary"/>
        <TextView
            android:id="@+id/id_friendgroup_item_parent_tv_friend_count"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:text="0"
            android:layout_marginRight="5dp"
            android:textSize="18sp"
            android:textColor="@color/colorGray_500"
            android:layout_centerVertical="true"
            android:layout_alignParentRight="true"/>
    </RelativeLayout>
</LinearLayout>
```
效果图:
![](https://github.com/devallever/DataProject/blob/master/data/expandablelistview/group_item.png?raw=true)

好友Item布局
```
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:orientation="vertical" android:layout_width="match_parent"
    android:layout_height="wrap_content"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    >

    <RelativeLayout
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:padding="10dp">

        <de.hdodenhof.circleimageview.CircleImageView
            android:id="@+id/id_friendgroup_item_child_iv_head"
            android:layout_width="56dp"
            android:layout_height="56dp"
            app:civ_border_width="0.5dp"
            app:civ_border_color="#fff"
            android:src="@mipmap/winchen"/>

        <TextView
            android:id="@+id/id_friendgroup_item_child_tv_nickname"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:text="Nickname"
            android:layout_marginLeft="10dp"
            android:textColor="@color/black_deep"
            android:textSize="18sp"
            android:layout_toRightOf="@id/id_friendgroup_item_child_iv_head"/>
        <TextView
            android:id="@+id/id_friendgroup_item_child_tv_signature"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:text="Signature"
            android:layout_marginLeft="10dp"
            android:layout_marginTop="4dp"
            android:maxLines="1"
            android:ellipsize="end"
            android:layout_below="@id/id_friendgroup_item_child_tv_nickname"
            android:layout_toRightOf="@id/id_friendgroup_item_child_iv_head"/>

    </RelativeLayout>

</LinearLayout>
```
效果图:
![](https://github.com/devallever/DataProject/blob/master/data/expandablelistview/user_item.png?raw=true)

## 封装数据
FriendGroupItem
```
public class FriendGroupItem {
    private String id;
    private String friendgroup_name;
    private List<FriendItem> list_friend;
    public String getId() {
        return id;
    }
    public void setId(String id) {
        this.id = id;
    }
    public String getFriendgroup_name() {
        return friendgroup_name;
    }
    public void setFriendgroup_name(String friendgroup_name) {
        this.friendgroup_name = friendgroup_name;
    }
    public List<FriendItem> getList_friend() {
        return list_friend;
    }
    public void setList_friend(List<FriendItem> list_friend) {
        this.list_friend = list_friend;
    }
    
}
```

FriendGroupItem
```
public class FriendItem  {
    private String sortLetters;  //显示数据拼音的首字母
    private String user_id;
    private String nickname;
    private String username;
    private String user_head_path;
    private String signature;
    public String getUser_id() {
        return user_id;
    }
    public void setUser_id(String user_id) {
        this.user_id = user_id;
    }
    public String getNickname() {
        return nickname;
    }
    public void setNickname(String nickname) {
        this.nickname = nickname;
    }
    public String getUser_head_path() {
        return user_head_path;
    }
    public void setUser_head_path(String user_head_path) {
        this.user_head_path = user_head_path;
    }
    public String getSignature() {
        return signature;
    }
    public void setSignature(String signature) {
        this.signature = signature;
    }
    public void setUsername(String username){
        this.username = username;
    }
    public String getUsername(){
        return this.username;
    }
    public String getSortLetters() {
        return sortLetters;
    }
    public void setSortLetters(String sortLetters) {
        this.sortLetters = sortLetters;
    }
}
```

## Adapter适配器
FriendGroupItemExpandableBaseAdapter
```
/**
 * Created by Allever on 2016/6/14.
 * 好友分组项适配器
 */
public class FriendGroupItemExpandableBaseAdapter extends BaseExpandableListAdapter {

    private Context context;
    private LayoutInflater inflater;
    private List<FriendGroupItem> list_friendgroupItem;

    public FriendGroupItemExpandableBaseAdapter(Context context, List<FriendGroupItem> list_friendgroupItem){
        this.context = context;
        this.list_friendgroupItem = list_friendgroupItem;
        inflater = (LayoutInflater) context.getSystemService(Context.LAYOUT_INFLATER_SERVICE);
    }


    @Override
    public View getGroupView(int parentPosition, boolean isExpanded, View convertView, ViewGroup parent) {
        FriendGroupItem friendGroupItem = list_friendgroupItem.get(parentPosition);
        View view;
        ParentViewHolder parentViewHolder;
        if (convertView == null){
            view = inflater.inflate(R.layout.friendgroup_item_parent,parent,false);
            parentViewHolder = new ParentViewHolder();
            parentViewHolder.tv_friendgroup_name = (TextView)view.findViewById(R.id.id_friendgroup_item_parent_tv_friendgroup_name);
            parentViewHolder.iv_expand = (ImageView)view.findViewById(R.id.id_friendgroup_item_parent_iv_expand);
            parentViewHolder.tv_friend_count = (TextView)view.findViewById(R.id.id_friendgroup_item_parent_tv_friend_count);
            view.setTag(parentViewHolder);
        }else{
            view  = convertView;
            parentViewHolder = (ParentViewHolder)view.getTag();
        }

        //判断isExpanded就可以控制是按下还是关闭，同时更换图片
        if (isExpanded){
            parentViewHolder.iv_expand.setImageDrawable(context.getResources().getDrawable(R.mipmap.ic_expanded));
        }else{
            parentViewHolder.iv_expand.setImageDrawable(context.getResources().getDrawable(R.mipmap.ic_unexpanded));
        }

        parentViewHolder.tv_friend_count.setText(friendGroupItem.getList_friend().size()+"");
        parentViewHolder.tv_friendgroup_name.setText(friendGroupItem.getFriendgroup_name());
        return view;
    }

    @Override
    public View getChildView(int parentPosition, int childPosition, boolean b, View convertView, ViewGroup parent) {
        FriendItem friendItem = list_friendgroupItem.get(parentPosition).getList_friend().get(childPosition);
        View view;
        ChildViewHolder childViewHolder;
        if(convertView == null){
            view = inflater.inflate(R.layout.friendgroup_item_child,parent,false);
            childViewHolder = new ChildViewHolder();
            childViewHolder.iv_head = (CircleImageView)view.findViewById(R.id.id_friendgroup_item_child_iv_head);
            childViewHolder.tv_nickname = (TextView)view.findViewById(R.id.id_friendgroup_item_child_tv_nickname);
            childViewHolder.tv_signature = (TextView)view.findViewById(R.id.id_friendgroup_item_child_tv_signature);
            view.setTag(childViewHolder);
        }else{
            view = convertView;
            childViewHolder = (ChildViewHolder)view.getTag();
        }
        Glide.with(context)
                .load(WebUtil.HTTP_ADDRESS + friendItem.getUser_head_path())
                .into(childViewHolder.iv_head);
        childViewHolder.tv_nickname.setText(friendItem.getNickname());
        childViewHolder.tv_signature.setText(friendItem.getSignature());
        return view;
    }

    private class ParentViewHolder{
        ImageView iv_expand;
        TextView tv_friendgroup_name;
        TextView tv_friend_count;
    }

    private class ChildViewHolder{
        CircleImageView iv_head;
        TextView tv_nickname;
        TextView tv_signature;
    }

    @Override
    public int getGroupCount() {
        return list_friendgroupItem.size();
    }

    @Override
    public int getChildrenCount(int parentPosition) {
        return list_friendgroupItem.get(parentPosition).getList_friend().size();
    }

    @Override
    public Object getGroup(int parengPosition) {
        return list_friendgroupItem.get(parengPosition);
    }

    @Override
    public Object getChild(int parentPosition, int childPosition) {
        return list_friendgroupItem.get(parentPosition).getList_friend().get(childPosition);
    }

    @Override
    public long getGroupId(int parentPosition) {
        return parentPosition;
    }

    @Override
    public long getChildId(int parentPosition, int childPosition) {
        return childPosition;
    }

    @Override
    public boolean hasStableIds() {
        return false;
    }
    
    @Override
    public boolean isChildSelectable(int i, int i1) {
        return true;
    }

}

```
继承自BaseExpandableListAdapter,重写里面几个方法, 看方法名就知道该方法是干什么的了,如果你熟悉ListView的话. 方法中很多都是成对存在的.根据方法名返回相应的值.重点就是getChildView()方法和getGroupView()方法;
```
	@Override
	public View getGroupView(int parentPosition, boolean isExpanded, View convertView, ViewGroup parent) {
	}
	
	@Override
	public View getChildView(int parentPosition, int childPosition, boolean b, View convertView, ViewGroup parent) {
	}
```
参数含义在上面已经标明了.

## 使用ExpandableListView
在布局文件中使用该控件
```
<ExpandableListView
	android:id="@+id/id_contact_fg2_expandable_listview"
	android:layout_width="match_parent"
	android:layout_height="match_parent">
</ExpandableListView>
```

在代码中
```
expandableListView = (ExpandableListView)view.findViewById(R.id.id_contact_fg2_expandable_listview);
expandableListView.setGroupIndicator(null); //设置指示器隐藏

friendGroupItemExpandableBaseAdapter = new FriendGroupItemExpandableBaseAdapter(getActivity(),list_friendgroupItem);
expandableListView.setAdapter(friendGroupItemExpandableBaseAdapter);

//设置监听器
expandableListView.setOnChildClickListener(new ExpandableListView.OnChildClickListener() {
	@Override
	public boolean onChildClick(ExpandableListView expandableListView, View view, int parentPosition, int childPosition, long l) {
		Intent intent = new Intent(getActivity(), UserDataDetailActivity.class);
		intent.putExtra("username", list_friendgroupItem.get(parentPosition).getList_friend().get(childPosition).getUsername());
		startActivity(intent);
		return false;
	}
});

```
效果图:
![](https://github.com/devallever/DataProject/blob/master/data/expandablelistview/review.png?raw=true)
