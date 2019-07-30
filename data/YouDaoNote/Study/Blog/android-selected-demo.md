---
title: Android  选择器
date: 2017-07-30 14:13:16
tags:
 - Android
 - 选择器
categories: [Android]
---

# 功能概述：
选中状态：选中时候变为蓝色，并有个减号，里面也是蓝色，
未选中状态：未选中时候为空白，并有个加号，
点中选中的状态变为未选中状态，点击未选中状态变为选中状态，当仅剩下一个选中时候，减号消失，表示不允许改变当前状态，即最少有一个选中状态

效果图1：
![](https://github.com/devallever/DataProject/blob/master/data/bicycle/selected-item-01.png?raw=true)

效果图2：
![](https://github.com/devallever/DataProject/blob/master/data/bicycle/selected-item-02.png?raw=true)

# 实现思路：
UI：因为背景是透明的，所以使用shape逐个绘图，然后把这些绘图用layer组合起来。
逻辑：可以有两中方式
1.每一项是一个布局，然后在这个布局添加内容，对这个布局使用单击监听。
2.使用列表RecyclerView。





# 详细实现

## 画UI图
### 绘制蓝色背景：
target_item_one_selected.xml
这个很简单，就是画个矩形，使用圆角和填充颜色
![](https://github.com/devallever/DataProject/blob/master/data/bicycle/bg_blue.png?raw=true)
```
<?xml version="1.0" encoding="utf-8"?>
<shape xmlns:android="http://schemas.android.com/apk/res/android"
    android:shape="rectangle">
    <solid
        android:color="@color/deep_blue" />
    <corners
        android:radius="10dp"/>
</shape>
```

### 绘制白色边框：
target_item_none_selected_framework.xml
这个跟上面差不多，不适用填充，设置边框颜色和圆角即可
效果图：
![](https://github.com/devallever/DataProject/blob/master/data/bicycle/bg_framework.png?raw=true)
```
<?xml version="1.0" encoding="utf-8"?>
<shape xmlns:android="http://schemas.android.com/apk/res/android"
    android:shape="rectangle">

    <stroke
        android:width="1dp"
        android:color="@color/white"/>
    <corners
        android:radius="10dp"/>
</shape>
```

### 绘制减号
首先绘制一个圆形：target_item_selected_circle_delete.xml
画一个圆形，填充蓝色，边框白色
效果图:
![](https://github.com/devallever/DataProject/blob/master/data/bicycle/delete_circle.png?raw=true)
```
<?xml version="1.0" encoding="utf-8"?>
<shape xmlns:android="http://schemas.android.com/apk/res/android"
    android:shape="oval">
    <solid
        android:color="@color/deep_blue"/>
    <stroke
        android:width="1dp"
        android:color="@color/white"/>
    <size
        android:width="20dp"
        android:height="20dp"/>
</shape>
```
绘制减号的横线：target_item_selected_line.xml
效果图：
![](https://github.com/devallever/DataProject/blob/master/data/bicycle/h_line.png?raw=true)
```
<?xml version="1.0" encoding="utf-8"?>
<shape xmlns:android="http://schemas.android.com/apk/res/android"
    android:shape="line"
    android:gravity="bottom|right">
        <stroke android:width="1dp"
            android:color="@color/white" />
        <size
            android:width="10dp"
            android:height="20dp" />
        <padding
            android:left="10dp"
            android:right="10dp"/>
</shape>
```
绘制减号：target_item_delete.xml
使用layer把上面两个shape组合起来
效果图：
![](https://github.com/devallever/DataProject/blob/master/data/bicycle/delete.png?raw=true)
```
<?xml version="1.0" encoding="utf-8"?>
<layer-list xmlns:android="http://schemas.android.com/apk/res/android">
    <item
        android:drawable="@drawable/target_item_selected_circle_delete"
        android:gravity="bottom|right">
    </item>
    <item
        android:gravity="bottom|right"
        android:drawable="@drawable/target_item_selected_line"
        android:right="5dp">
    </item>
</layer-list>
```

### 绘制加号
首先绘制一个圆:target_item_none_selected_circle.xml
填充白色，边框白色
效果图：
![](https://github.com/devallever/DataProject/blob/master/data/bicycle/add_circle.png?raw=true)
```
<?xml version="1.0" encoding="utf-8"?>
<shape xmlns:android="http://schemas.android.com/apk/res/android"
    android:shape="oval">
    <solid
        android:color="@color/black"/>
    <stroke
        android:width="1dp"
        android:color="@color/white"/>
    <size
        android:width="20dp"
        android:height="20dp"/>
</shape>
```

绘制加号的横线，跟上面的横线相同的

绘制加号的竖线：target_item_none_selected_vertical_line.xml
就是把横线旋转90度即形成竖直的线
效果图：
![](https://github.com/devallever/DataProject/blob/master/data/bicycle/v_line.png?raw=true)
```
<?xml version="1.0" encoding="utf-8"?>
<rotate xmlns:android="http://schemas.android.com/apk/res/android"
    android:fromDegrees="90"
    android:toDegrees="90">

    <shape
        android:shape="line">
        <stroke
            android:width="1dp"
            android:color="@color/white"/>
        <size
            android:width="10dp"
            android:height="10dp"/>
    </shape>


</rotate>
```

最后把以上三个shape组合起来就成了一个加号：target_item_add.xml
效果图：
![](https://github.com/devallever/DataProject/blob/master/data/bicycle/add.png?raw=true)
```
<?xml version="1.0" encoding="utf-8"?>
<layer-list xmlns:android="http://schemas.android.com/apk/res/android">
    <item
        android:drawable="@drawable/target_item_none_selected_circle"
        android:gravity="bottom|right">
    </item>
    <item
        android:drawable="@drawable/target_item_none_selected_vertical_line"
        android:gravity="bottom|right"
        android:right="5dp"
        android:bottom="5dp">
    </item>
    <item
        android:gravity="bottom|right"
        android:drawable="@drawable/target_item_none_selected_line"
        android:right="5dp">
    </item>
</layer-list>
```
### UI组合
我觉得这里的难点就是右下角的加减好在边框上向右下方移动那么一点点，这里及解决方法就是，用一个总体父布局，里面包含两部分，第一部分是中心框部分，把这部分设置居中，并且在四周留有空间，第二部分是加减号，设置在父布局的右下方，这样看起来就是个效果了。当然还有其他方法，例如找图片，这种图片应该很难找得到，或者是知己制作这种图片，设置给父布局就好了。

# 逻辑实现
这里有两种方式：
第一种就是每一项就是一个布局，总共有八个布局
第二种就是使用列表RecyclerView

## 布局实现方式
### 布局
在布局文件中放这八个布局
```
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:orientation="vertical" android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:background="@mipmap/bg1"
    android:gravity="right">
    <LinearLayout
        android:id="@+id/id_target_ll_first_line"
        android:layout_width="wrap_content"
        android:layout_height="0dp"
        android:layout_weight="1"
        android:orientation="horizontal"
        android:layout_alignParentRight="true"
        android:layout_marginTop="10dp">

        <RelativeLayout
            android:layout_width="110dp"
            android:layout_height="110dp"
            android:padding="10dp">
            <RelativeLayout
                android:id="@+id/id_target_item_container_speed"
                android:layout_width="80dp"
                android:layout_height="80dp"
                android:layout_centerInParent="true"
                android:background="@drawable/target_item_one_selected">

                <TextView
                    android:layout_width="wrap_content"
                    android:layout_height="wrap_content"
                    android:text="速度"
                    android:layout_centerHorizontal="true"
                    android:textColor="@color/white"
                    android:layout_marginTop="10dp"/>
                <TextView
                    android:id="@+id/id_target_item_tv_speed"
                    android:layout_width="wrap_content"
                    android:layout_height="wrap_content"
                    android:text="6"
                    android:textSize="30sp"
                    android:textColor="@color/white"
                    android:layout_centerHorizontal="true"
                    android:layout_alignParentBottom="true"
                    android:layout_marginBottom="10dp"/>

            </RelativeLayout>
            <RelativeLayout
                android:id="@+id/id_target_item_rl_speed_option"
                android:layout_width="wrap_content"
                android:layout_height="wrap_content"
                android:layout_alignParentRight="true"
                android:layout_alignParentBottom="true"
                android:background="@drawable/target_item_one_selected">
            </RelativeLayout>

        </RelativeLayout>

        <RelativeLayout
            android:layout_width="110dp"
            android:layout_height="110dp"
            android:padding="10dp">
            <RelativeLayout
                android:id="@+id/id_target_item_container_distance"
                android:layout_width="80dp"
                android:layout_height="80dp"
                android:layout_centerInParent="true"
                android:background="@drawable/target_item_none_selected_framework">

                <TextView
                    android:layout_width="wrap_content"
                    android:layout_height="wrap_content"
                    android:text="距离"
                    android:layout_marginLeft="10dp"
                    android:textColor="@color/white"
                    android:layout_marginTop="10dp"/>
                <TextView
                    android:id="@+id/id_target_item_tv_distance"
                    android:layout_width="wrap_content"
                    android:layout_height="wrap_content"
                    android:text="0.7"
                    android:textSize="30sp"
                    android:textColor="@color/white"
                    android:layout_marginLeft="10dp"
                    android:layout_alignParentBottom="true"
                    android:layout_marginBottom="10dp"/>

                <TextView
                    android:layout_width="wrap_content"
                    android:layout_height="wrap_content"
                    android:text="km"
                    android:textColor="@color/white"
                    android:layout_alignParentBottom="true"
                    android:layout_marginBottom="15dp"
                    android:layout_marginLeft="2dp"
                    android:layout_toRightOf="@id/id_target_item_tv_distance" />

            </RelativeLayout>
            <RelativeLayout
                android:id="@+id/id_target_item_rl_distance_option"
                android:layout_width="wrap_content"
                android:layout_height="wrap_content"
                android:layout_alignParentRight="true"
                android:layout_alignParentBottom="true"
                android:background="@drawable/target_item_add">
            </RelativeLayout>
        </RelativeLayout>

        <RelativeLayout
            android:layout_width="110dp"
            android:layout_height="110dp"
            android:padding="10dp"
            android:layout_alignParentRight="true">
            <RelativeLayout
                android:id="@+id/id_target_item_container_power"
                android:layout_width="80dp"
                android:layout_height="80dp"
                android:layout_centerInParent="true"
                android:background="@drawable/target_item_none_selected_framework">

                <TextView
                    android:layout_width="wrap_content"
                    android:layout_height="wrap_content"
                    android:text="功率"
                    android:layout_marginLeft="10dp"
                    android:textColor="@color/white"
                    android:layout_marginTop="10dp"/>
                <TextView
                    android:id="@+id/id_target_item_tv_power"
                    android:layout_width="wrap_content"
                    android:layout_height="wrap_content"
                    android:text="74"
                    android:textSize="30sp"
                    android:textColor="@color/white"
                    android:layout_marginLeft="10dp"
                    android:layout_alignParentBottom="true"
                    android:layout_marginBottom="10dp"/>

                <TextView
                    android:layout_width="wrap_content"
                    android:layout_height="wrap_content"
                    android:text="w"
                    android:textColor="@color/white"
                    android:layout_alignParentBottom="true"
                    android:layout_marginBottom="15dp"
                    android:layout_marginLeft="2dp"
                    android:layout_toRightOf="@id/id_target_item_tv_power" />

            </RelativeLayout>
            <RelativeLayout
                android:id="@+id/id_target_item_rl_power_option"
                android:layout_width="wrap_content"
                android:layout_height="wrap_content"
                android:layout_alignParentRight="true"
                android:layout_alignParentBottom="true"
                android:background="@drawable/target_item_add">
            </RelativeLayout>
        </RelativeLayout>

    </LinearLayout>


    <!-- The Second line-->
    <LinearLayout
        android:id="@+id/id_target_ll_second_line"
        android:layout_width="wrap_content"
        android:layout_height="0dp"
        android:layout_weight="1"
        android:orientation="horizontal"
        android:layout_alignParentRight="true"
        android:layout_below="@id/id_target_ll_first_line">

        <RelativeLayout
            android:layout_width="110dp"
            android:layout_height="110dp"
            android:padding="10dp"
            android:layout_alignParentRight="true">
            <RelativeLayout
                android:id="@+id/id_target_item_container_slope_degree"
                android:layout_width="80dp"
                android:layout_height="80dp"
                android:layout_centerInParent="true"
                android:background="@drawable/target_item_none_selected_framework">

                <TextView
                    android:layout_width="wrap_content"
                    android:layout_height="wrap_content"
                    android:text="坡度"
                    android:layout_centerHorizontal="true"
                    android:textColor="@color/white"
                    android:layout_marginTop="10dp"/>
                <!-- slope degree 坡度-->
                <TextView
                    android:id="@+id/id_target_item_tv_slope_degree"
                    android:layout_width="wrap_content"
                    android:layout_height="wrap_content"
                    android:text="5"
                    android:textSize="30sp"
                    android:textColor="@color/white"
                    android:layout_centerHorizontal="true"
                    android:layout_alignParentBottom="true"
                    android:layout_marginBottom="10dp"/>

            </RelativeLayout>
            <RelativeLayout
                android:id="@+id/id_target_item_rl_slope_degree_option"
                android:layout_width="wrap_content"
                android:layout_height="wrap_content"
                android:layout_alignParentRight="true"
                android:layout_alignParentBottom="true"
                android:background="@drawable/target_item_add">
            </RelativeLayout>
        </RelativeLayout>

        <RelativeLayout
            android:layout_width="110dp"
            android:layout_height="110dp"
            android:padding="10dp">
            <RelativeLayout
                android:id="@+id/id_target_item_container_patch_speed"
                android:layout_width="80dp"
                android:layout_height="80dp"
                android:layout_centerInParent="true"
                android:background="@drawable/target_item_none_selected_framework">

                <TextView
                    android:layout_width="wrap_content"
                    android:layout_height="wrap_content"
                    android:text="配速"
                    android:layout_marginLeft="10dp"
                    android:textColor="@color/white"
                    android:layout_marginTop="10dp"/>
                <TextView
                    android:id="@+id/id_target_item_tv_patch_speed"
                    android:layout_width="wrap_content"
                    android:layout_height="wrap_content"
                    android:text="00:30"
                    android:textSize="25sp"
                    android:textColor="@color/white"
                    android:layout_marginLeft="10dp"
                    android:layout_alignParentBottom="true"
                    android:layout_marginBottom="15dp"/>

            </RelativeLayout>
            <RelativeLayout
                android:id="@+id/id_target_item_rl_patch_speed_option"
                android:layout_width="wrap_content"
                android:layout_height="wrap_content"
                android:layout_alignParentRight="true"
                android:layout_alignParentBottom="true"
                android:background="@drawable/target_item_add">
            </RelativeLayout>
        </RelativeLayout>

        <RelativeLayout
            android:layout_width="110dp"
            android:layout_height="110dp"
            android:padding="10dp"
            android:layout_alignParentRight="true">
            <RelativeLayout
                android:id="@+id/id_target_item_container_calorie"
                android:layout_width="80dp"
                android:layout_height="80dp"
                android:layout_centerInParent="true"
                android:background="@drawable/target_item_none_selected_framework">

                <TextView
                    android:layout_width="wrap_content"
                    android:layout_height="wrap_content"
                    android:text="卡路里"
                    android:layout_marginLeft="10dp"
                    android:textColor="@color/white"
                    android:layout_marginTop="10dp"/>
                <TextView
                    android:id="@+id/id_target_item_tv_calorie"
                    android:layout_width="wrap_content"
                    android:layout_height="wrap_content"
                    android:text="119"
                    android:textSize="25sp"
                    android:textColor="@color/white"
                    android:layout_marginLeft="10dp"
                    android:layout_alignParentBottom="true"
                    android:layout_marginBottom="15dp"/>

                <TextView
                    android:layout_width="wrap_content"
                    android:layout_height="wrap_content"
                    android:text="kcal"
                    android:textColor="@color/white"
                    android:layout_alignParentBottom="true"
                    android:layout_marginBottom="20dp"
                    android:layout_marginLeft="2dp"
                    android:layout_toRightOf="@id/id_target_item_tv_calorie" />


            </RelativeLayout>
            <RelativeLayout
                android:id="@+id/id_target_item_rl_calorie_option"
                android:layout_width="wrap_content"
                android:layout_height="wrap_content"
                android:layout_alignParentRight="true"
                android:layout_alignParentBottom="true"
                android:background="@drawable/target_item_add">
            </RelativeLayout>
        </RelativeLayout>

    </LinearLayout>


    <!-- The Third line -->
    <LinearLayout
        android:id="@+id/id_target_ll_third_line"
        android:layout_width="wrap_content"
        android:layout_height="0dp"
        android:layout_weight="1"
        android:orientation="horizontal"
        android:layout_alignParentRight="true"
        android:layout_below="@id/id_target_ll_second_line">

        <RelativeLayout
            android:layout_width="110dp"
            android:layout_height="110dp"
            android:padding="10dp"
            android:layout_alignParentRight="true">
            <RelativeLayout
                android:id="@+id/id_target_item_container_time"
                android:layout_width="80dp"
                android:layout_height="80dp"
                android:layout_centerInParent="true"
                android:background="@drawable/target_item_none_selected_framework">

                <TextView
                    android:layout_width="wrap_content"
                    android:layout_height="wrap_content"
                    android:text="时间"
                    android:layout_marginLeft="10dp"
                    android:textColor="@color/white"
                    android:layout_marginTop="10dp"/>
                <TextView
                    android:id="@+id/id_target_item_tv_time"
                    android:layout_width="wrap_content"
                    android:layout_height="wrap_content"
                    android:text="02:30"
                    android:textSize="25sp"
                    android:textColor="@color/white"
                    android:layout_marginLeft="10dp"
                    android:layout_alignParentBottom="true"
                    android:layout_marginBottom="15dp"/>

            </RelativeLayout>
            <RelativeLayout
                android:id="@+id/id_target_item_rl_time_option"
                android:layout_width="wrap_content"
                android:layout_height="wrap_content"
                android:layout_alignParentRight="true"
                android:layout_alignParentBottom="true"
                android:background="@drawable/target_item_add">
            </RelativeLayout>
        </RelativeLayout>

        <RelativeLayout
            android:layout_width="110dp"
            android:layout_height="110dp"
            android:padding="10dp">
            <RelativeLayout
                android:id="@+id/id_target_item_container_bmp"
                android:layout_width="80dp"
                android:layout_height="80dp"
                android:layout_centerInParent="true"
                android:background="@drawable/target_item_none_selected_framework">

                <TextView
                    android:layout_width="wrap_content"
                    android:layout_height="wrap_content"
                    android:text="心率"
                    android:layout_marginLeft="10dp"
                    android:textColor="@color/white"
                    android:layout_marginTop="10dp"/>
                <TextView
                    android:id="@+id/id_target_item_tv_bmp"
                    android:layout_width="wrap_content"
                    android:layout_height="wrap_content"
                    android:text="68"
                    android:textSize="30sp"
                    android:textColor="@color/white"
                    android:layout_marginLeft="10dp"
                    android:layout_alignParentBottom="true"
                    android:layout_marginBottom="10dp"/>

                <TextView
                    android:layout_width="wrap_content"
                    android:layout_height="wrap_content"
                    android:text="bmp"
                    android:textColor="@color/white"
                    android:layout_alignParentBottom="true"
                    android:layout_marginBottom="15dp"
                    android:layout_marginLeft="2dp"
                    android:layout_toRightOf="@id/id_target_item_tv_bmp" />


            </RelativeLayout>
            <RelativeLayout
                android:id="@+id/id_target_item_rl_bmp_option"
                android:layout_width="wrap_content"
                android:layout_height="wrap_content"
                android:layout_alignParentRight="true"
                android:layout_alignParentBottom="true"
                android:background="@drawable/target_item_add">
            </RelativeLayout>
        </RelativeLayout>

        <RelativeLayout
            android:layout_width="110dp"
            android:layout_height="110dp"
            android:padding="10dp"
            android:layout_alignParentRight="true">
        </RelativeLayout>

    </LinearLayout>

</LinearLayout>
```
文件有一点长，并且很多都是重复的，所以我推荐使用这种方式，但是我第一次想到就是这种方式，所以试试看，因为使用列表的话，其他方面会很复杂，因为每个项其实是有些不同的。

### 本地数据库
TargetItem表
name：每一项的名称，根据这个名称进行更新状态操作
valeu：数值
flag_selected：选中状态 0 和 1 
index：索引，列表方式使用到
```
package com.allever.bicycle.bean;

import org.litepal.crud.DataSupport;

/**
 * Created by allever on 17-7-26.
 */

public class TargetItem extends DataSupport{
    private int id;
    private String name;
    private String value;
    private int flag_selected;
    private int index;

    public int getId() {
        return id;
    }

    public void setId(int id) {
        this.id = id;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public String getValue() {
        return value;
    }

    public void setValue(String value) {
        this.value = value;
    }

    public int getFlag_selected() {
        return flag_selected;
    }

    public void setFlag_selected(int flag_selected) {
        this.flag_selected = flag_selected;
    }

    public int getIndex() {
        return index;
    }

    public void setIndex(int index) {
        this.index = index;
    }
}

```


### 交互逻辑
首次启动创建数据库表和初始化数据
```
final List<TargetItem> targetItems = DataSupport.findAll(TargetItem.class);
if (targetItems.size()==0){
	initData();
	//Toast.makeText(this,"initData",Toast.LENGTH_LONG).show();
}
showData();
```

initData()
```
    /**
     * 初始化数据
     * */
    private void initData(){
        DataSupport.deleteAll(TargetItem.class);

        TargetItem targetSpeed = new TargetItem();
        targetSpeed.setName("speed");
        targetSpeed.setValue("6");
        targetSpeed.setFlag_selected(1);
        targetSpeed.save();

        TargetItem targetDistance = new TargetItem();
        targetDistance.setName("distance");
        targetDistance.setValue("0.7");
        targetDistance.setFlag_selected(0);
        targetDistance.save();

        TargetItem targetPower = new TargetItem();
        targetPower.setName("power");
        targetPower.setValue("74");
        targetPower.setFlag_selected(0);
        targetPower.save();


        TargetItem targetSlopeDegree = new TargetItem();
        targetSlopeDegree.setName("slope_degree");
        targetSlopeDegree.setValue("5");
        targetSlopeDegree.setFlag_selected(0);
        targetSlopeDegree.save();


        TargetItem targetPatchSpeed = new TargetItem();
        targetPatchSpeed.setName("patch_speed");
        targetPatchSpeed.setValue("00:29");
        targetPatchSpeed.setFlag_selected(0);
        targetPatchSpeed.save();


        TargetItem targetCalorie = new TargetItem();
        targetCalorie.setName("calorie");
        targetCalorie.setValue("119");
        targetCalorie.setFlag_selected(0);
        targetCalorie.save();

        TargetItem targetTime = new TargetItem();
        targetTime.setName("time");
        targetTime.setValue("02:30");
        targetTime.setFlag_selected(0);
        targetTime.save();

        TargetItem targetBmp = new TargetItem();
        targetBmp.setName("bmp");
        targetBmp.setValue("74");
        targetBmp.setFlag_selected(0);
        targetBmp.save();

    }
```

showData()
```
    private void showData(){
        TargetItem targetSpeed = DataSupport.where("name=?", "speed").find(TargetItem.class).get(0);
        tv_speed.setText(targetSpeed.getValue());
        refreshTargetItem("speed",false);

        TargetItem targetDistance = DataSupport.where("name=?", "distance").find(TargetItem.class).get(0);
        tv_distance.setText(targetDistance.getValue());
        refreshTargetItem("distance",false);

        TargetItem targetPower = DataSupport.where("name=?", "power").find(TargetItem.class).get(0);
        tv_power.setText(targetPower.getValue());
        refreshTargetItem("power",false);

        TargetItem targetSlopeDegree = DataSupport.where("name=?", "slope_degree").find(TargetItem.class).get(0);
        tv_slope_degree.setText(targetSlopeDegree.getValue());
        refreshTargetItem("slope_degree",false);

        TargetItem targetPatchSpeed = DataSupport.where("name=?", "patch_speed").find(TargetItem.class).get(0);
        tv_patch_speed.setText(targetPatchSpeed.getValue());
        refreshTargetItem("patch_speed",false);

        TargetItem targetCalorie = DataSupport.where("name=?", "calorie").find(TargetItem.class).get(0);
        tv_calorie.setText(targetCalorie.getValue());
        refreshTargetItem("calorie",false);


        TargetItem targetTime = DataSupport.where("name=?", "time").find(TargetItem.class).get(0);
        tv_time.setText(targetTime.getValue());
        refreshTargetItem("time",false);

        TargetItem targetBmp = DataSupport.where("name=?", "bmp").find(TargetItem.class).get(0);
        tv_bmp.setText(targetBmp.getValue());
        refreshTargetItem("bmp",false);

        List<TargetItem> targetItems = DataSupport.where("flag_selected=?", "1").find(TargetItem.class);
        if (targetItems.size()==1){
            refreshTargetItem(targetItems.get(0).getName(),true);
        }

    }
```
默认状态选中是，蓝色背景+减号组成，
未选中是白色边框背景+加号组成，考虑到只用一种选中状态时，把减号去掉
因此，要判断选中状态的个数，如果是一个，则刷新这个显示方式。即
```
List<TargetItem> targetItems = DataSupport.where("flag_selected=?", "1").find(TargetItem.class);
if (targetItems.size()==1){
	refreshTargetItem(targetItems.get(0).getName(),true);
}
```
刷新界面：refreshTargetItem(String name, boolean isOneSelected)
isOneSelected：只有一种选中状态标记

```
    private void refreshTargetItem(String name, boolean isOneSelected){
        TargetItem target = DataSupport.where("name=?", name).find(TargetItem.class).get(0);
        switch (name){
            case "speed":
                if (target.getFlag_selected()==1){
                    rl_container_speed.setBackgroundResource(R.drawable.target_item_one_selected);
                    rl_speed_option.setBackgroundResource(R.drawable.target_item_delete);
                    if (isOneSelected) rl_speed_option.setVisibility(View.INVISIBLE);
                    else rl_speed_option.setVisibility(View.VISIBLE);
                }else {
                    rl_container_speed.setBackgroundResource(R.drawable.target_item_none_selected_framework);
                    rl_speed_option.setBackgroundResource(R.drawable.target_item_add);
                }
                break;

            case "distance":
                if (target.getFlag_selected()==1){
                    rl_container_distance.setBackgroundResource(R.drawable.target_item_one_selected);
                    rl_distance_option.setBackgroundResource(R.drawable.target_item_delete);
                    if (isOneSelected) rl_distance_option.setVisibility(View.INVISIBLE);
                    else rl_distance_option.setVisibility(View.VISIBLE);
                }else {
                    rl_container_distance.setBackgroundResource(R.drawable.target_item_none_selected_framework);
                    rl_distance_option.setBackgroundResource(R.drawable.target_item_add);
                }
                break;
            case "power":
                if (target.getFlag_selected()==1){
                    rl_container_power.setBackgroundResource(R.drawable.target_item_one_selected);
                    rl_power_option.setBackgroundResource(R.drawable.target_item_delete);
                    if (isOneSelected) rl_power_option.setVisibility(View.INVISIBLE);
                    else rl_power_option.setVisibility(View.VISIBLE);
                }else {
                    rl_container_power.setBackgroundResource(R.drawable.target_item_none_selected_framework);
                    rl_power_option.setBackgroundResource(R.drawable.target_item_add);
                }
                break;

            case "slope_degree":
                if (target.getFlag_selected()==1){
                    rl_container_slope_degree.setBackgroundResource(R.drawable.target_item_one_selected);
                    rl_slope_degree_option.setBackgroundResource(R.drawable.target_item_delete);
                    if (isOneSelected) rl_slope_degree_option.setVisibility(View.INVISIBLE);
                    else rl_slope_degree_option.setVisibility(View.VISIBLE);
                }else {
                    rl_container_slope_degree.setBackgroundResource(R.drawable.target_item_none_selected_framework);
                    rl_slope_degree_option.setBackgroundResource(R.drawable.target_item_add);
                }
                break;

            case "patch_speed":
                if (target.getFlag_selected()==1){
                    rl_container_patch_speed.setBackgroundResource(R.drawable.target_item_one_selected);
                    rl_patch_speed_option.setBackgroundResource(R.drawable.target_item_delete);
                    if (isOneSelected) rl_patch_speed_option.setVisibility(View.INVISIBLE);
                    else rl_patch_speed_option.setVisibility(View.VISIBLE);
                }else {
                    rl_container_patch_speed.setBackgroundResource(R.drawable.target_item_none_selected_framework);
                    rl_patch_speed_option.setBackgroundResource(R.drawable.target_item_add);
                }
                break;

            case "calorie":
                if (target.getFlag_selected()==1){
                    rl_container_calorie.setBackgroundResource(R.drawable.target_item_one_selected);
                    rl_calorie_option.setBackgroundResource(R.drawable.target_item_delete);
                    if (isOneSelected) rl_calorie_option.setVisibility(View.INVISIBLE);
                    else rl_calorie_option.setVisibility(View.VISIBLE);
                }else {
                    rl_container_calorie.setBackgroundResource(R.drawable.target_item_none_selected_framework);
                    rl_calorie_option.setBackgroundResource(R.drawable.target_item_add);
                }
                break;
            case "time":
                if (target.getFlag_selected()==1){
                    rl_container_time.setBackgroundResource(R.drawable.target_item_one_selected);
                    rl_time_option.setBackgroundResource(R.drawable.target_item_delete);
                    if (isOneSelected) rl_time_option.setVisibility(View.INVISIBLE);
                    else rl_time_option.setVisibility(View.VISIBLE);
                }else {
                    rl_container_time.setBackgroundResource(R.drawable.target_item_none_selected_framework);
                    rl_time_option.setBackgroundResource(R.drawable.target_item_add);
                }
                break;
            case "bmp":
                if (target.getFlag_selected()==1){
                    rl_container_bmp.setBackgroundResource(R.drawable.target_item_one_selected);
                    rl_bmp_option.setBackgroundResource(R.drawable.target_item_delete);
                    if (isOneSelected) rl_bmp_option.setVisibility(View.INVISIBLE);
                    else rl_bmp_option.setVisibility(View.VISIBLE);
                }else {
                    rl_container_bmp.setBackgroundResource(R.drawable.target_item_none_selected_framework);
                    rl_bmp_option.setBackgroundResource(R.drawable.target_item_add);
                }
                break;
        }
    }
```

监听各个布局
```
    @Override
    public void onClick(View v) {
        int id  = v.getId();
        switch (id){
            case R.id.id_target_item_container_speed:
                updateFlag("speed");
                break;
            case R.id.id_target_item_container_distance:
                updateFlag("distance");
                break;
            case R.id.id_target_item_container_power:
                updateFlag("power");
                break;
            case R.id.id_target_item_container_slope_degree:
                updateFlag("slope_degree");
                break;
            case R.id.id_target_item_container_patch_speed:
                updateFlag("patch_speed");
                break;
            case R.id.id_target_item_container_calorie:
                updateFlag("calorie");
                break;
            case R.id.id_target_item_container_time:
                updateFlag("time");
                break;
            case R.id.id_target_item_container_bmp:
                updateFlag("bmp");
                break;
        }
    }
```

更新标记
首先根据name找到数据库中记录的状态值，如果是1，进行删除操作，如果是0进行添加操作，简单地说就是把标记改为相反。
删除操作：
首先，查找数据库中选中状态的数量，如果查找到的记录数小于2(即只有一个选中)则不允许删除操作，否则进行删除操作
```
target.setFlag_selected(0);
target.save();
```
删除之后还要判断，当前选中状态为1的记录数，如果记录数为1，则把刷新该记录的界面
```
targetItems = DataSupport.where("flag_selected=?","1").find(TargetItem.class);
if (targetItems.size() == 1){
	refreshTargetItem(targetItems.get(0).getName(),true);
}
```
添加操作：
```
target.setFlag_selected(1);
target.save();
```
添加之后，还是判断当前选中状态为1的记录数，如果记录数为2，就刷新该记录的界面，因为原来只有一个选中时，减号是不显示的，要把原来的减号显示出来。
```
targetItems = DataSupport.where("flag_selected=?","1").find(TargetItem.class);
if (targetItems.size() == 2){
	refreshTargetItem(targetItems.get(0).getName(),false);
	refreshTargetItem(targetItems.get(1).getName(),false);
}else {
	refreshTargetItem(name,false);
}
```
updateFlag()
```
    private void updateFlag(String name){
        TargetItem target = DataSupport.where("name=?", name).find(TargetItem.class).get(0);
        List<TargetItem> targetItems;
        if (target.getFlag_selected()==1){
            //删除操作
            targetItems = DataSupport.where("flag_selected=?","1").find(TargetItem.class);
            Log.d(TAG, "selected count = " + targetItems.size());
            if (targetItems.size() < 2){
                //即只有一个选中,不允许操作
                //Toast.makeText(this,"不允许删除",Toast.LENGTH_LONG).show();
            }else {
                //进行删除操作
                //Toast.makeText(this,"许删除",Toast.LENGTH_LONG).show();
                target.setFlag_selected(0);
                target.save();
                //刷新界面
                refreshTargetItem(name,false);
                //
                targetItems = DataSupport.where("flag_selected=?","1").find(TargetItem.class);
                if (targetItems.size() == 1){
                    refreshTargetItem(targetItems.get(0).getName(),true);
                }
            }
        }else {
            //添加操作
            //Toast.makeText(this,"添加",Toast.LENGTH_LONG).show();
            target.setFlag_selected(1);
            target.save();
            //刷新界面
            targetItems = DataSupport.where("flag_selected=?","1").find(TargetItem.class);
            if (targetItems.size() == 2){
                refreshTargetItem(targetItems.get(0).getName(),false);
                refreshTargetItem(targetItems.get(1).getName(),false);
            }else {
                refreshTargetItem(name,false);
            }

        }
    }
```
## RecyclerView实现方式
### 列表项布局
跟上面布局中差不多，这里就只重复的那部分
```
<?xml version="1.0" encoding="utf-8"?>
<RelativeLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="110dp"
    android:layout_height="110dp"
    android:padding="10dp">

    <RelativeLayout
        android:id="@+id/id_target_recycler_item_container"
        android:layout_width="80dp"
        android:layout_height="80dp"
        android:layout_centerInParent="true"
        android:background="@drawable/target_item_none_selected_framework">

        <TextView
            android:id="@+id/id_target_recycler_item_tv_name"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:text="距离"
            android:layout_marginLeft="10dp"
            android:textColor="@color/white"
            android:layout_marginTop="10dp"/>
        <TextView
            android:id="@+id/id_target_recycler_item_tv_value"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:text="0.7"
            android:textSize="25sp"
            android:textColor="@color/white"
            android:layout_marginLeft="10dp"
            android:layout_alignParentBottom="true"
            android:layout_marginBottom="10dp"/>

        <TextView
            android:id="@+id/id_target_recycler_item_tv_unit"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:text="km"
            android:textColor="@color/white"
            android:layout_alignParentBottom="true"
            android:layout_marginBottom="15dp"
            android:layout_marginLeft="2dp"
            android:layout_toRightOf="@id/id_target_recycler_item_tv_value" />

    </RelativeLayout>
    <RelativeLayout
        android:id="@+id/id_target_recycler_item_rl_option"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_alignParentRight="true"
        android:layout_alignParentBottom="true"
        android:background="@drawable/target_item_add">
    </RelativeLayout>

</RelativeLayout>
```

### Adapter
重点就是Adapter实现啦，其实也很简单，根据选中标记选择显示方式
初始显示之后判断选中状态为1的数量，如果只有一个选中，就把减号隐藏，这里设置了蓝色背景，可能是没设置大小的原因，看不出而已，正常因该是隐藏布局。
```
    @Override
    public void onBindViewHolder(TargetItemViewHolder holder, int position) {
        TargetItem targetItem = targetItems.get(position);
        holder.tv_value.setText(targetItem.getValue());
        if (targetItem.getFlag_selected()==1){
            holder.rl_container.setBackgroundResource(R.drawable.target_item_one_selected);
            holder.rl_option.setBackgroundResource(R.drawable.target_item_delete);
        }else {
            holder.rl_container.setBackgroundResource(R.drawable.target_item_none_selected_framework);
            holder.rl_option.setBackgroundResource(R.drawable.target_item_add);
        }

        List<TargetItem> list = DataSupport.where("flag_selected=?","1").find(TargetItem.class);

        if (list.size() == 1){
            if (position == list.get(0).getIndex()){
                holder.rl_option.setBackgroundResource(R.drawable.target_item_one_selected);
            }
        }


        switch (position){
            case 0:
                holder.tv_name.setText("速度");
                holder.tv_unit.setText("");
                break;

            case 1:
                holder.tv_name.setText("距离");
                holder.tv_unit.setText("km");
                break;

            case 2:
                holder.tv_name.setText("功率");
                holder.tv_unit.setText("w");
                break;

            case 3:
                holder.tv_name.setText("坡度");
                holder.tv_unit.setText("");
                break;

            case 4:
                holder.tv_name.setText("配速");
                holder.tv_unit.setText("");
                break;

            case 5:
                holder.tv_name.setText("卡路里");
                holder.tv_unit.setText("kcal");
                break;

            case 6:
                holder.tv_name.setText("时间");
                holder.tv_unit.setText("");
                break;

            case 7:
                holder.tv_name.setText("心率");
                holder.tv_unit.setText("bmp");
                break;
        }
    }
```

### 事件监听
RecyclerItemClickListener
```
package com.allever.bicycle.listener;

import android.content.Context;
import android.support.v7.widget.RecyclerView;
import android.view.GestureDetector;
import android.view.MotionEvent;
import android.view.View;


/**
 * Created by Allever on 2016/11/5.
 */



public class RecyclerItemClickListener implements RecyclerView.OnItemTouchListener {


    private OnItemClickListener mListener;

    private GestureDetector mGestureDetector;

    public RecyclerItemClickListener(Context context, final RecyclerView recyclerView, OnItemClickListener listener) {
        mListener = listener;

        mGestureDetector = new GestureDetector(context, new GestureDetector.SimpleOnGestureListener() {
            @Override
            public boolean onSingleTapUp(MotionEvent e) {
                return true;
            }

            @Override
            public void onLongPress(MotionEvent e) {
                View childView = recyclerView.findChildViewUnder(e.getX(), e.getY());

                if (childView != null && mListener != null) {
                    mListener.onItemLongClick(childView, recyclerView.getChildAdapterPosition(childView));
                }
            }
        });
    }

    @Override
    public boolean onInterceptTouchEvent(RecyclerView view, MotionEvent e) {
        View childView = view.findChildViewUnder(e.getX(), e.getY());
        if (childView != null && mListener != null && mGestureDetector.onTouchEvent(e)) {
            mListener.onItemClick(childView, view.getChildAdapterPosition(childView));
        }
        return false;
    }
    @Override
    public void onTouchEvent(RecyclerView view, MotionEvent motionEvent) {
    }
    @Override
    public void onRequestDisallowInterceptTouchEvent(boolean disallowIntercept) {
    }

    public interface OnItemClickListener {
        void onItemClick(View view, int position);
        void onItemLongClick(View view, int position);
    }
}

```
RecyclerView设置监听
具体逻辑跟上面大同小异，注意这里是更新了数据库数据已更新了列表数据
```
        recyclerView.addOnItemTouchListener(new RecyclerItemClickListener(this, recyclerView, new RecyclerItemClickListener.OnItemClickListener() {
            @Override
            public void onItemClick(View view, int position) {
                //根据flag判断进行移除还是添加操作
                TargetItem target = targetItems.get(position);
                if (target.getFlag_selected()==1){
                    //移除操作
                    List<TargetItem> list = DataSupport.where("flag_selected=?","1").find(TargetItem.class);
                    if (list.size() < 2){
                        //即只有一个选中,不允许操作
                        Toast.makeText(TargetItemListActivity.this,"不允许删除",Toast.LENGTH_LONG).show();
                    }else {
                        //进行移除操作
                        //Toast.makeText(this,"许删除",Toast.LENGTH_LONG).show();
                        target.setFlag_selected(0);
                        targetItems.set(position,target);
                        target.save();
                        targetItemAdapter.notifyDataSetChanged();
                    }
                }else {
                    //添加操作
                    target.setFlag_selected(1);
                    targetItems.set(position,target);
                    target.save();
                    //刷新界面
                    targetItemAdapter.notifyDataSetChanged();
                }
            }

            @Override
            public void onItemLongClick(View view, int position) {

            }
        }));
```