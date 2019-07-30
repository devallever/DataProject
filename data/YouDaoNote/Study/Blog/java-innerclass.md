---
title: Java 内部类
date: 2017-05-05 10:26:16
tags:
 - Java
 - 内部类
categories: [Java]
---


内部类是指在一个外部类的内部再定义一个类。内部类作为外部类的一个成员，并且依附于外部类而存在的。内部类可为静态，可用protected和private修饰（而外部类只能使用public和缺省的包访问权限）。内部类主要有以下几类：成员内部类、局部内部类、静态内部类、匿名内部类

为什么需要内部类？
典型的情况是，内部类继承自某个类或实现某个接口，内部类的代码操作创建其的外围类的对象。所以你可以认为内部类提供了某种进入其外围类的窗口。使用内部类最吸引人的原因是：
每 个内部类都能独立地继承自一个（接口的）实现，所以无论外围类是否已经继承了某个（接口的）实现，对于内部类都没有影响。如果没有内部类提供的可以继承多 个具体的或抽象的类的能力，一些设计与编程问题就很难解决。从这个角度看，内部类使得多重继承的解决方案变得完整。接口解决了部分问题，而内部类有效地实 现了“多重继承”。

# 静态内部类型(static nested type)

静态内部类型可以是class,interface,或者enum。而其他类型的内部类型只能是class。

静态内部类其实还是一个顶层类（源代码文件级别的类），他不依赖外部类,只不过它被封装在另一个class或者interface中，而不是直接定义在文件级别中。因此，它和一般的类静态成员很类似:

1、它不包含外部类当前对象引用this,因此不能直接访问外部类的实际成员,但可以使用外部类的static成员。

2、静态内部类作为一个静态成员,因此可以用访问权限修饰符:public . private .......等。用的最多一般是private

    引用静态内部类：  Wapper.Inner

3、不能在非静态内部类中再定义静态内部类。静态内部类可以无限深度的嵌套下去。


提升

内部类最终会被javac编译为独立的类，JVM看见的都是top-level类。

编译后的class文件形如：WrapperClass $ InnerStaticClass.class

 

下面是使用静态内部类简单实现链式栈数据结构的例子。

```
class LinkedSatck<E> {
    private static class Node<T> {

        public Node(Node<T> next, T data) {
            this.next = next;
            this.data = data;
        }

        @SuppressWarnings("unused")
        public Node() {
            this(null, null);
        }

        private Node<T> next;
        private T data;

        public T getData() {
            return data;
        }

        public Node<T> getNext() {
            return next;
        }

        public boolean isEndNode() {
            return (next == null);
        }

    }   



    public LinkedSatck() {
        topNode = new Node<E>(null, null);

        // size =0;
    }

    private int size = 0;
    private Node<E> topNode = null;

    public void push(E e) {

        Node<E> newTopNode = new Node<E>(topNode, e);
        ++size;
        topNode = newTopNode;

    }

    public E pop() {
        if (topNode.isEndNode())
            return null;
        else {
            E re = topNode.getData();

            topNode = topNode.getNext();

            --size;
            return re;

        }
    }

    public int size() {
        return size;
    }

    public boolean isEmpty() {
        return size == 0;
        // return topNode.isEnd();
    }

}
```

# 成员内部类(inner member class)

成员内部类最重要的特点就是它可以直接使用外部类的实例成员和static成员，即便是使用private修饰也是如此。 

因为成员内部类总包含了一个外部类的当前对象引用 ，奇怪的名字 this$0，这个引用在成员内部类实例化时被外部类的当前对象引用this初始化。

大致实现如下：
```
class Outter
{
    private class Inner
    {
        private final Outter this$0;  //javac自动添加
        Inner(Outter o)         //javac自动添加
        {
            this.this$0 = o;
        }
    }
    //使用成员内部类对象时发生：new Inner(Outter.this)
}
```

 这也是为什么在创建一个成员内部类对象时,要先创建一个外部类对象的原因了。

 

和普通实例成员一样，成员内部里是属于外部类的对象的，那么，在成员内部类就理所当然可以直接使用外部类的其他实例成员以及static成员。

因为是实例成员，所以可以使用访问修饰符:public 、protected、private、和默认的包访问权限。

因为是实例成员，因此，在类的外部使用内部类时，必须先创建1个外部类对象，在实际开发中很少使用这个。

```
class Outter
{
    public class Inner
    {
        
    }
}

public class Test 
{
public static void main(String[] args) {

        Outter out = new  Outter();
        Outter.Inner in = out.new Inner();
    }

}
```

一个例子，自定义一个Str类，来支持迭代。
```
class Str implements Iterable<Character>
{
    
    private String innerStr;

    public Str(String s)
    {
        this.innerStr = (s==null?"":s);
    }
    
    
    private class StrIterator implements Iterator<Character>    //迭代器类 作为成员内部类
    {
        
        private int curIndex = 0;

        @Override
        public boolean hasNext() {
            
            return curIndex < innerStr.length();    //直接访问外部类的成员 innerStr
        }

        @Override
        public Character next() {
            
            return innerStr.charAt(curIndex++);   //直接访问外部类的成员 innerStr
        }

        @Override
        public void remove() {
            throw new UnsupportedOperationException();
            
        }
        
    }
    
    @Override
    public Iterator<Character> iterator() {
        
        return new StrIterator();
    }
    
}
```

成员内部类中不能有static成员，即不能有static方法 和 static字段（除非static字段修饰为static final，那样的话它只不过是一个符号常量罢了）。因为java中类的静态成员必须定义在一个top-level顶层类中，而成员内部类（包括后面的方法内部类，匿名内部类）不是top-level顶层类。

static成员需要定义在top-level类中，而成员内部类不是top-level类。

 
提升

JVM是不理解nested类型的，也就是在它看来，所有的类型都是top-level的， 

在每一个成员内部类中，javac都会自动添加一个字段：this$0，用来引用外部类当前对象。同时， 内部类的构造函数会自动为这个字段添加一个参数，当构造内部类对象时，

外部类当前对象就会传递给this$0，让这个字段引用外部类当前实例对象。

从这点我们也会发现，为什么要实例化一个成员内部类前，需要先实例化一个外部类对象。因为成员内部包含了一个外部类对象。

 

编译后的class文件形如：WrapperClass $ InnerClass.class

# 局部内部类(local inner class)

定义在一个方法（包括了类的构造块和static构造块）内部的类，叫局部内部类。它不能有任何访问权限修饰符，因为它只能被包装它的方法使用，离开方法后就不可用了。

局部内部类可以和成员内部类一样，访问外部类的实例成员。同时，它还能直接使用包含它的方法的局部final常量,final参数。javac会复制使用了的外部方法的局部final量保存在局部内部类中作为私有的备份。

因此,当这个外部方法执行完毕后，虽然方法中的局部变量的 lifetime结束了，但是如果局部类的实例作为返回值,它会带着外部方法的局部final量离开这个局部作用域,也就是说,局部变量的生命延长到了和局部内部类的对象的生命周期一致。并不会随着方法执行完立刻被清理掉。我们可以以此来形成闭包。

同样,局部内部类不是top-level类，不能有static成员，除非是static final 字段。

```
public class Main
{
    public static void main(String[] args) 
    {
        
        MsgGenerator g5 = fac(5);
        System.out.println(g5.generatorMsg());
        
        MsgGenerator g2 = fac(2);
        System.out.println(g2.generatorMsg());
        
    }
   
    public static MsgGenerator fac(final int times)
    {
        class Generator implements MsgGenerator
        {
            @Override
            public StringBuffer generatorMsg()
            {
                
                StringBuffer s=  new StringBuffer() ;
                
                for(int i=0;i<times;++i)
                {
                    s.append("hello  ");
                }
                
                return s;
            }
            
            
        }  //end of class
        
        
        return new Generator();    //向外发出闭包
    }
   
}


interface MsgGenerator
{
    StringBuffer generatorMsg();
}
```

提升：

局部内部类之所以能访问外部类的实例成员，其原因和成员内部类是一样的：内部类中有保存了外部类对象的引用。除此之外，局部内部类还能访问包装方法的final字段，javac会将内部类使用了的final 局部常量拷贝到局部内部类中保存，并在局部内部类对象实例化时，初始化这些final常量。因此，局部内部类使用的final常量是自己的拷贝分。

 

局部内部类的实现原理（模拟）

```
class Wapper
{
    
    public void wapperFunction()       //在方法中定义一个局部类
    {
        final int x = 10;
        
        class Local       //局部类
        {
            
            private final int local_copy_x;   //假如在局部类中使用了局部常量x，则javac自动生成
            
            private final Wapper this$0;            //javac自动生成的字段，用于保存外部类当前对象引用
            
            
            //首先，局部内部类必定会包含外部类对象，着就是javac插入的第一个构造参数，这是必定的。
            //其次，如果我们在局部内部类中使用了包装方法foo中的局部final常量，如x，则会在局部类中
            //自动添加隐藏字段local_copy_x，并在构造器中初始化它。
            Local(Wapper w,final int x)
            {
                this$0 = w;
                local_copy_x = x;
            }
            

        }//end of Local
        
       
　　　   
　　　　 new Local();     //当实例化局部内部类对象时，等价于 new Local(Wapper.this,x)
    
    }
    
}
```

所以，之所以能在局部内部类中访问外部类的实例，是因为javac自动添加并用外部类当前对象this初始化了局部内部类的字段this$0，这样this$0就引用了外部类当前对象。

局部内部类能使用包装 方法的final字段，也是因为javac自动在局部内部类中添加并初始化的结果。

# 匿名内部类 (inner anonymous class)

匿名内部类是特殊的局部内部类,它没有类名。它的访问特性和局部内类一样。如果只会使用类的一个对象,则可以使用匿名内部类,没有名称避免了再引入一个类名称， 匿名内部类是没有名称的局部内部类，访问特性与局部内部类一样。

 

因为没有类名,因此只能使用父类名或者接口名来创建对象。

new + superClass  或者 new+interface  。创建对象 是 使用new表达式。

new 表达式:匿名类对象的创建方式是使用new表达式,创建对象的同时也是类结构的编写。表达式的值是一个匿名类对象的引用。

 ```
 new  SuperClass(param1，param2){   类体    }
 ```
 
 一般来说,匿名类没有构造参数,如果有,则传递给他的父类的构造函数。

 

匿名内部类由于没有类名，所以你不能定义新的构造函数，只能有默认的构造函数（javac添加的）。补救的做法是使用构造块。

 

下面是使用swing 中的Timer定时触发回调函数的例子，使用匿名类创建 ActionListener对象。程序每经过1000ms，就会调用 ActionListener对象的actionPerformed方法。

```
public static void main(String[] args) {


        //javax.swing.Timer;
        Timer t = new Timer(1000, 
                            new ActionListener() {
                        @Override
                        public void actionPerformed(ActionEvent e) 
                        {
                            Toolkit.getDefaultToolkit().beep();  //系统响铃声    
                        }
                    }
                    
            );
        
        t.start();
        
        while(true)
        {
            
        }
        

    }
```

# 内部类的工作原理

内部类只是Java的语法糖，jvm是不理解内部类的，它所看见的都是top-level顶层类。将内部类分离为单独的顶层类，是javac 的任务。内部类被javac合成为单独的类，并形成独立的class文件，这个文件有独特的名称，形式如下：

static内部类:  OutterClass$InnerClass.class

成员内部类：OutterClass$InnerClass.class

局部内部类:  OutterClass$XInnerClass.class           # X为一个正整数

局部内部类:  OutterClass$X.class                          # X为一个正整数

 

对于static内部类，无需多解释，因为 static内部类和外部类是无依赖关系的，static内部类不包含外部类引用，javac只是将他们简单的分离。

 

成员内部类为什么能访问外部类的成员？因为内部类会被javac自动插入一个字段this&0去保存外部类当前对象this的引用。

```
public class OutterClass
{
    
    private int outFiled = 100;
    
    public class InnerClass
    {
        
        public void f()
        {
            int i = outFiled;
        }
    }
}
```

javap反编译后的结果
```
Compiled from "OutterClass.java"
public class OutterClass$InnerClass {
  final OutterClass this$0;      //由javac自动合成：内部类包含1个外部类的当前对象的引用this&0，使用this&0避免和this冲突    
    
  //javac自动合成：合成的构造函数，有1个外部类参数，用于初始化 this&0，this&0被赋值为OutterClass.this
  public OutterClass$InnerClass(OutterClass);
    Code:
       0: aload_0       
       1: aload_1       
       2: putfield      #1                  // Field this$0:LOutterClass;
       5: aload_0       
       6: invokespecial #2                  // Method java/lang/Object."<init>":()V
       9: return        




  public void f();
    Code:
       0: aload_0       
       1: getfield      #1                  // Field this$0:LOutterClass;
       4: invokestatic  #3                  // Method OutterClass.access$000:(LOutterClass;)I
       7: istore_1      
       8: return        
}
```

局部内部类和匿名内部类除了可以访问外部类的成员（原因和成员内部类是相同的），还可以访问外部方法的局部final量和final参数。原因如下：
> A local class can use local variables because javac automatically gives the class a
private instance field to hold a copy of each local variable the class uses.
The compiler also adds hidden parameters to each local class constructor to initial‐
ize these automatically created private fields. A local class does not actually access
local variables but merely its own private copies of them. This could cause inconsis‐
tencies if the local variables could alter outside of the local class.

                                                                         -- 《Java int a Nutshell》

因为javac会自动在（局部和匿名）内部类中插入私有 的实例字段来保存 使用了的外部方法的final量的拷贝。javac还会在（局部和匿名）内部类中的构造函数的添加参数来初始化插入的私有 的字段。所以，（局部和匿名）内部类使用的实质是自己获得的拷贝量，而不是直接使用外部方法的final量。如果外部方法的量不修饰为final的话，那么意味着它的值可以改变，这就可能会导致（局部和匿名）内部类中获得的拷贝和外部方法不一致。所以java强制要求只能使用final量。


# 静态内部类和非静态内部类的区别

如果你不需要内部类对象与其外围类对象之间有联系，那你可以将内部类声明为static。这通常称为嵌套类（nested class）。Static Nested Class是被声明为静态（static）的内部类，它可以不依赖于外部类实例被实例化。而通常的内部类需要在外部类实例化后才能实例化。想要理解static应用于内部类时的含义，你就必须记住，普通的内部类对象隐含地保存了一个引用，指向创建它的外围类对象。然而，当内部类是static的时，就不是这样了。嵌套类意味着： 

1. 嵌套类的对象，并不需要其外围类的对象。 

2. 不能从嵌套类的对象中访问非静态的外围类对象。

```
public class StaticTest{
   private static String name = "woobo";
   private String num = "X001";
   static class Person{ // 静态内部类可以用public,protected,private修饰 

       // 静态内部类中可以定义静态或者非静态的成员  
     private String address = "China";

Private Static String x=“as”;
     public String mail = "kongbowoo@yahoo.com.cn";//内部类公有成员
     public void display(){
       //System.out.println(num);//不能直接访问外部类的非静态成员

// 静态内部类不能访问外部类的非静态成员(包括非静态变量和非静态方法)
       System.out.println(name);//只能直接访问外部类的静态成员

//静态内部类只能访问外部类的静态成员(包括静态变量和静态方法)
       System.out.println("Inner " + address);//访问本内部类成员。
     }
   }
   public void printInfo(){
     Person person = new Person();

// 外部类访问内部类的非静态成员:实例化内部类即可 

person.display();

     //System.out.println(mail);//不可访问
     //System.out.println(address);//不可访问
     System.out.println(person.address);//可以访问内部类的私有成员

System.out.println(Person.x);// 外部类访问内部类的静态成员：内部类.静态成员
     System.out.println(person.mail);//可以访问内部类的公有成员
   }
   public static void main(String[] args){
     StaticTest staticTest = new StaticTest();
     staticTest.printInfo();
   }
}
```



在静态嵌套类内部, 不能访问外部类的非静态成员, 这是由Java语法中"静态方法不能直接访问非静态成员"所限定.注意, 外部类访问内部类的的成员有些特别, 不能直接访问, 但可以通过内部类实例来访问, 这是因为静态嵌套内的所有成员和方法默认为静态的了.同时注意, 内部静态类Person只在类StaticTest 范围内可见, 若在其它类中引用或初始化, 均是错误的.
一 . 静态内部类可以有静态成员，而非静态内部类则不能有静态成员。 
二 . 静态内部类的非静态成员可以访问外部类的静态变量，而不可访问外部类的非静态变量；

三 . 非静态内部类的非静态成员可以访问外部类的非静态变量。

生成一个静态内部类不需要外部类成员：这是静态内部类和成员内部类的区别。静态内部类的对象可以直接生成：Outer.Inner in = new Outer.Inner();而不需要通过生成外部类对象来生成。这样实际上使静态内部类成为了一个顶级类(正常情况下，你不能在接口内部放置任何代码，但嵌套类可以作为接口的一部分，因为它是static 的。只是将嵌套类置于接口的命名空间内，这并不违反接口的规则）

> 原文地址：[http://www.cnblogs.com/WuXuanKun/p/6220964.html](http://www.cnblogs.com/WuXuanKun/p/6220964.html)   
