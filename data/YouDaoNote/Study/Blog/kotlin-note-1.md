---
title: Kotlin学习笔记 1
date: 2017-05-22 17:30:55
tags:
 - Kotlin
 - Android
categories: [Kotlin]
---

> 这里并没有系统的讲解Kotlin语法, 只是在做Demo时候刻意使用Kotlin语言,并记录一下使用方法
如果要学习可以看看官网的文档,以及它推荐的一些书籍

# Activity
例如创建一个BaseActivity,则这样写
```
 open class BaseActivity : AppCompatActivity() {
 }
```
 - open: 关键字, 如果该类用来继承的要加上open, 或者abstract
 - 继承的方式是 : 
 
既然是Activity的子类,那么在重写onCreate方法时候会自动补全代码
```
open class BaseActivity : AppCompatActivity() {
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
    }
}
```
 - override: 不用看就知道是方法覆盖啦
 - fun 是定义方法
 - 注意方法参数的表达方式跟Java有点区别,先是变量名,再指明类型.
 - 注意问号?, Kotlin是空安全的, 这里表示Bundle可以为空, 第一次自学Kotlin,理解的不是很到位,望指正哈.
 
# Button
创建了Activity自然就是添加几个按钮,监听,
```
val btn_scale = findViewById(R.id.btn) as TextView
btn_scale.setOnClickListener {
	//do something
}
```
 - val 定义变量, 相当与final, 之后不能修改引用, 
 var 则是初始化以后也可以指向其他引用
 - findViewById(R.id.btn) as TextView
 找到控件之后通过 as 关键字转化TextView类型
 - 设置监听,方法跟java一样,但是不再是传递一个OnClickListener,直接大括号{}
 当然也可以看看标准使用方式是这样的
```
btn.setOnClickListener(View.OnClickListener { 
	//do something
})
```

如果不想每个Button设置监听都串OnClickListener,则要Activity实现OnClickListener只要在继承的类后面加入 , OnClickListener, 并从写onClick方法
```
class AnimationActivity : BaseActivity(), View.OnClickListener {

	...
	override fun onClick(v: View?) {
        val intent: Intent
        when(v?.id){
            R.id.id_animation_activity_view_animation ->{
                intent = Intent(this, ViewAnimationActivity::class.java)
                startActivity(intent)
            }
            R.id.id_animation_activity_property_animation ->{
                intent = Intent(this, PropertyAnimationActivity::class.java)
                startActivity(intent)
            }
        }

    }
}
```

细心的你已经发现,这里有很多新奇好玩的东西,下面逐一讲解

# 使用分支
在java使用条件的分之是通过switch(){}来使用的,而在Kotlin就是用 when 关键字
```
when(v?.id){
	R.id.id_animation_activity_view_animation ->{
		// do something
	}
	R.id.id_animation_activity_property_animation ->{
		//do something
	}
}
```
很明显when()括号内就是判断的类型,这里是view的id
然后每个分之就是这么判断
```
R.id.id_animation_activity_view_animation ->{
	//do something
}
```

# 启动另一个Acivity
```
val intent: Intent
intent = Intent(this, XXXActivity::class.java)
startActivity(intent)
```
至于为什么这样用,我现在还不是很清楚哎
另外,如果在某某控件的监听方法中启动另一个Activity,应该是这样,这里使用的是recyclerView的监听
```
intent = Intent(this@MainActivity, XXXActivity::class.java)
startActivity(intent)
```

# 声明变量
在声明局部变量时候, 通常使用val,但是当声明一个全局变量时候,通常使用var,并在声明的时候初始化为空, 例如
```
private var tv_target: TextView? = null
```
表示声明一个TextView对象, 并初始化为空,不初始化就报错的
然后在Activity中正常使用
```
tv_target = findViewById(R.id.id_view_animation_activity_tv_target) as TextView
```
当要修改TextView的显示的值时候, 不再是setText(),
```
tv_target.setText("");
```
而是通过设置属性值方式,它会调用setText()方法
```
tv_target.text = "hello"
```
以后setXXX方法都是按这种方式使用 ,但上面的还是报错,因为tv_target定义为 TevtView?
报错信息:
```
Smart cast to 'TextView' is impossible, because 'tv_target' is a mutable property that could have been changed by this time
```
so 正确的姿势是
```
tv_target?.text = "hello"
```

# 定义实体类
在Java中, 需要定义属性,并且创建该属性的gettet和setter方法  
在Kotlin中只需要这样
```
class ListItem(
        var title: String,
        var description: String
)
```
这样它会自动生成getter和setter方法,并不需要要我们实现
通常我们这样使用,
```
var listItemA = ListItem("titleText", "descriptionText")
listItemA.title = "HI Title"
listItemA.description = "Hi Description"
```

# 定义方法并设置参数的默认值
例如在Activity中定义Toast的方法
```
private fun toast(message: String, length: Int=Toast.LENGTH_LONG):Unit{
	Toast.makeText(this@MainActivity, message,length).show()
}
```
Unit为返回类型,Unit表相当于示返回值为void,并且可以省略Unit,如
```
private fun toast(message: String, length: Int=Toast.LENGTH_LONG){
        Toast.makeText(this@MainActivity, message,length).show()
}
```
这里length参数指定了一个默认值就是Toast.LENGTH_LONG
调用是应该这样
```
toast("HI Android")
//或
toast("Hi Android",1000)
```
# 使用RecyclerView
在Activity通常定义一个ArrayList类型的全局变量,作为Adapter的参数
因此这样声明一个全局变量
```
private var listItems: ArrayList<ListItem>? = null
```
> 注意这里是ArrayList? 类型

然后创建Adapter,他的构造方法如下:
```
class MyRecyclerViewAdapter(private val context: Context,
                            private val listItems: ArrayList<ListItem>?) 
    : RecyclerView.Adapter<MyRecyclerViewAdapter.MyViewHolder>() {
}
```
构造方法中listItems的类型也是ArrayList?, 因为在Activity中, 定义一个可变的全局变量时候初始化为空,因此这里与之对应  
初始化listItems对象
```
listItems =  ArrayList<ListItem>()
listItems?.add(ListItem("chapter1", "Android Framework"))
listItems!!.add(ListItem("chapter2", "Android Developer tools"))
listItems!!.add(ListItem("chapter3", "Custom View"))
listItems!!.add(ListItem("chapter4", "listView Demo"))
listItems!!.add(ListItem("chapter5", "Scroll Analysis"))
listItems!!.add(ListItem("chapter6", "Draw"))
listItems!!.add(ListItem("chapter7", "Animation Demo"))
```
在这里不知道?. 和 !!. 到底有什么区别.有待深入研究,这时候要知道该这么用就够了  
最后就是这样使用
```
val recyclerView = findViewById(R.id.id_main_recycler_view) as RecyclerView
val recyclerViewAdapter = MyRecyclerViewAdapter(this,listItems)
recyclerView.layoutManager = LinearLayoutManager(this)
recyclerView.adapter = recyclerViewAdapter
```

