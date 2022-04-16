**什么时候重写hashCode()和equals()?**

hashMap或者hashSet中key的类型为业务对象时需要重写业务对象实体的hashCode()和equals()方法，不重写的话hashMap会加入相同值的对象key。

重写hashCode()不重写equals()?

相同值的两个对象的hashcode经过重写之后是相同的，经过散列算法之后会落到同一个桶中，如果不重写equals()，默认是比较两个对象的地址，地址肯定是不同的，就会导致桶中会出现两个相同值的value。

重写equals()不重写hashCode()?

相同值的两个对象的hashcode如果不重写是不同的，这样经过散列算法后会落到不同的桶中，就会导致数组中出现两个相同值的value。



#### 深拷贝、浅拷贝、引用拷贝

浅拷贝：A对象中含有X对象，如果对A对象进行拷贝一个B对象，则B对象中的X对象依然和A对象中X对象是同一个。

深拷贝：A对象中含有X对象，如果对A对象进行拷贝一个B对象，则B对象中的X对象和A对象中X对象是两个对象。

引用拷贝：A对象中含有X对象，ａ指针引用A对象，拷贝ａ指针为ｂ指针，也引用A对象。

![](D:\images\拷贝.png)

#### String、StringBuilder、StringBuffer

**String为什么无法修改？**

String底层是一个private　final修饰的字符数组，因为是private的，且又是final的，尽管数组的引用不能修改，数组的内容可以修改，但是String类没有暴露修改字符数组的方法，所以String是无法修改的。

StringBuffer　append方法加了synchronized关键字，保证了线程安全。

StringBuilder性能比StringBuffer好１５％－２０％，但是线程不安全。

String　ａ　＝　ｘ　＋　ｙ　＋　ｚ；

对象引用和“+”的字符串拼接方式，实际上是通过 `StringBuilder` 调用 `append()` 方法实现的，拼接完成之后调用 `toString()` 得到一个 `String` 对象 。不过，在循环内使用“+”进行字符串的拼接的话，存在比较明显的缺陷：**编译器不会创建单个 `StringBuilder` 以复用，会导致创建过多的 `StringBuilder` 对象**。