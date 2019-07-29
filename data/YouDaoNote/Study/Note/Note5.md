**Note5**

##### 1.（）文件、SharePreference和SQLite使用场景

 - 文件：适用于存储一些简单的文本数据类型或二进制数据
 - SharePreference：适用于存储一些少量纪录的简单数据，如单个用户信息、配置信息。
 - SQLite：适用于大量记录的复杂关系型数据，如所有用户的信息
 
##### 2.（）Okhttp、Volley和Retrofit比较

 - 

##### 1.（）字符和字符串所占字节数

 - char：2字节
 - String：
     - 英文字符串的字节数与英文字母相同，如“hello”占5个字节
     - 中文字符串根据编码而定，如GBK下，一个汉字占2个字节；UTF-8下，一个汉字占3个字节
     
##### 2.（）String、StringBuffered和StringBuilder的区别。（）

##### 3.（）equals与“==”的区别

 - equals：比较内容是否相等
 - ==：比较地址是否相等
	JVM有个区域叫字符串缓冲池，使用String a = "abc"时，会检查缓冲池中是否有相同内容的对象，有就直接引用，没有才新建一个值为“abc”的对象，并放入缓冲池，再把引用返回给a。而new String("abc")则创建一个新的String对象，在堆内存开辟新的空间来存放这个对象。