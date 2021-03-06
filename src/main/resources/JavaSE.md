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

### this() super()

- this(),super()都只能出现在构造方法的第一行，也就意味着this()和super()只能同时出现一个。

- **当子类构造方法没有调用this()和super()时，编译器会隐式自动在子类构造方法第一行加上super().**

- 静态方法不能使用this.和super.关键字，因为静态成员随着类的加载而加载，此时对象还没有被创建出来。

- 当不写构造方法的时候默认为无参构造，当定义一个有参构造的时候，则认定该类没有无参构造，所以如果一个类具有有参构造的情况下还想要无参构造的话必须显示声明。

    - 当父类只有有参构造没有无参构造的话，子类必须实现父类的有参构造。



### 零星知识点：

- #### switch语句里面如果case命中之后没有break关键字的话会一直执行到default。

- #### 二分查找时间复杂度为logn

- #### 多态存在的三个必要条件：1.继承2.重写3.父类引用指向子类对象

    - ```java
    Animal a = new Dog()     ==    Dog d = new Dog; Animal a = (Animal) d;
    ```

      上面两式就是同一个表达式，都是父类引用指向子类对象，a对象为Animal对象

        1. a对象能访问Animal类里的变量但是不能访问Animal子类的变量。
        2. a对象可以访问父类的所有方法，对于子类方法而言，**a对象只能访问重写了父类方法的子类方法**，对于子类独特的方法（即父类没有的），a对象是无法访问的。对于子类重写了父类的方法，a对象调用这个方法时，**优先调用的是子类的方法**，除非子类将此方法变为独特的才能调用父类方法。
        3. 父类怎样访问子类方法？
           子类重写父类方法之后，通过父类引用指向子类对象，就可以通过父类的引用访问子类方法。

- #### **方法的重写遵循两同两小一大原则：**

    - 两同：方法名相同，参数类型数量相同。
    - 两小: 子类返回类型小于等于父类，子类抛出异常小于等于父类。
    - 一大:  子类访问权限大于等于父类。
    - 方法签名只有方法名和参数类型，即判断方法重载与否只看方法签名，不看返回体。看返回体来区分是否重载太过繁琐。

- #### 加载顺序：

  > 新建子类对象时，会首先加载父类的静态变量--->父类的静态代码块--->子类的静态常量--->子类的静态代码块--->加载父类的成员变量--->调用父类的构造方法--->父类的代码块--->加载子类的成员变量--->子类的构造方法--->子类的代码块，当新建过一次对象之后，再新建该对象时就不会再加载静态代码块和变量了，因为这些都是随着类加载的时候加载进内存的，就不会再此加载了，但是构造方法依然会重新调用。

    1. 构造方法也是静态方法，因此当首次创建Dog的对象或者访问Dog类的静态方法/静态成员变量时，Java解释器必须查找类路径，以定位Dog.class文件。

    2. 然后载入Dog.class文件(这将会创建一个Class对象)，**有关静态初始化的所有动作都会执行，包括静态代码块，静态成员变量，即：哪怕访问某个类的静态方法/静态常量，载入该类Class文件的时候都会加载其他的静态常量/静态方法，和静态代码块**，因此静态初始化只会再Class对象首次加载的时候进行一次。

    3. 当用new Dog()创建对象的时候，首先再堆上为Dog对象分配足够的存储空间。

    4. 这块存储空间会被清零，这就自动地将Dog对象中的所有基本类型数据都设置成了默认值，引用则被设置成了null。

    5. 执行所有出现于字段定义处的初始化动作。初始化程序员给定的成员变量。

    6. 执行构造器。

       注意，是先将变量设置为默认值，然后再将变量设置为程序员给定的初始值。

- ### java值传递问题：

  > ​	JAVA语言方法调用都是值传递的，在类里定义的变量a，b，调用方法swap（a，b），之后打印的a ，b还是之前的值，并没有发生改变。因为在调用swap方法的时候，实际上传递了一个a，b变量的副本，并不是他们本身，在方法里面副本变量发生了变化并不会影响变量本身，所以之后打印的变量还是真正的变量，而不是变化之后的副本变量。
  >
  > ​	而在方法里传递引用对象时，实际上传递的是对该对象地址的引用，所以在方法里对该对象进行改变的话，实际上就是改变了对该对象地址的引用，所以最后对象确实是发生了变化，把数组传到方法里就是一个例子，传到方法里的数组引用发生了变化，最后实际上就发生了变化。
  >
  > ​	原先的在栈里的对象引用 引用着堆里的对象，**当调用方法的时候实际上就是新建一个引用引用着同一个堆里的对象，所以当新的引用对堆里的对象做出改变时，原先的引用 引用的对象自然就发生了变化，因为是同一个对象。** 当方法里该引用新建一个新的对象时，该引用就引用着一个新的堆对象，当该引用对新对象做出任何改变时都与原引用 引用的对象无关了

![](images\浅引用1.png)
![](images\浅引用2.png)

​		总结：**把对象句柄传进方法只是在方法里复制了一个句柄引用着堆中同一个对象**，**方法中的句柄改变引用的对象不对原先的句柄造成任何影响，原先的句柄依然引用着原先的对象**（所以如果打印原先的句柄的话还是会得到原先的引用对象，不管方法里的句柄引用了谁）；**当方法中的句柄对引用的堆对象进行操作从而使得堆对象发生变化时，原先的句柄才会发现他引用的对象发生了变化**。

- **java类的成员变量会给一个初始默认值，但是方法内的局部变量不会给初始默认值，而是和CPP一样的随机值，Java会在编译的时候就不通过。**
- 在花括号作用域结束之后，在该作用域里创建的位于栈中的引用句柄将会被回收。

- **当需要装载某个类的时候（通常是在为该类创建第一个对象的时候或访问该类静态成员的时候），编译器会先找到其.class文件，然后将该类的字节码装入内存**。
- 如果构造器里有对成员变量进行赋值，那么不管成员变量的值设置为多少最终都会是构造器里设置的值。
- **可变参数列表 Object... obs,获取的仍然是一个数组，编译器会把传来的对象列表转换成数组，也可以直接传一个数组，也可以什么都不传**。String... strs,表示可变参数只能是String类型，**应该只在重载方法的一个版本上使用可变参数列表，或者干脆不用，因为当不传参数的时候编译器不知道是哪个方法。**
- 作为一名类库设计员，会尽一切可能将一切方法都定为private，而仅向客户端程序员公开你愿意让他们使用的方法。

- static强调唯一性。final修饰基本类型时为常量，但修饰引用类型时只是让引用不可变，但是引用的对象内容是可变的。
- 在参数列表中，将参数声明为final意味着调用该方法的用户不能改变参数指向的对象或基本变量，只能用给定的对象。**只有在为了明确禁止覆写方法时才使用final修饰方法**。给类加上final时意为不可以有子类继承来改变它的设计。
- 准确地说，一个类当它任意一个static成员被访问时，就会被加载，所有的static对象和static代码块在加载时按照文本的顺序（在类中定义的顺序）依次初始化。
- 单例模式双重校验锁为什么要给instance加上volatile修饰：因为编译器为了优化代码可能会重排序代码导致虚拟机已经在线程A中为对象分配好了内存空间但是还没有初始化该对象，这时B线程进入方法发现对象已经不为null了，就会返回一个只分配好了内存空间但是没有初始化的instance。
- 销毁的顺序应该与初始化的顺序相反，以防一个对象依赖另一个对象。对于属性来说，就意味着声明的顺序相反（因为属性是按照声明顺序初始化的）。因为派生类可能调用了基类的一些方法，所以基类组件这时得存活，不能过早地被销毁。



###### 接口

- 接口的成员默认都是public修饰。

- **jdk8允许接口有默认和静态方法。关键字default、static允许在接口中提供方法实现。**default实现了多继承的某些特性。static把工具功能置于接口中。
- 接口的典型使用是代表一个类的类型或形容词，如Runnable或Serializable，而抽象类通常是类层次结构的一部分或一件事物的类型，如String或ActionHero。
- **接口同样可以包含属性，这些属性被隐式指明为static和final。接口的方法隐式为public abstract**

###### 函数式编程：

- **所有lambda表达式方法体都是单行，该表达式的结果自动成为lambda表达式的返回值，在此处使用return关键字是非法的。**
- 如果在lambda表达式中确实需要多行，则必须将这些行放到花括号中，且在这种情况下必须要有return关键字。



#### 文件：

相对路径：getPath()

绝对路径：getAbsolutePath()

规范路径:getCanonicalPath()：**拿到的是当前文件的工作目录，即根目录!!!**

规范路径是绝对路径，但绝对路径不一定是规范路径

> 规范路径（绝对路径1）  :  d:\\temp\\file
>
> 绝对路径2：d:\\temp\\.\\file
>
> 绝对路径3:   d:\\temp\\..\\temp\\file

##### 值得注意的是，可能有大量指向同一文件的绝对路径，但只有一个规范路径。

