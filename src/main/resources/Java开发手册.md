### 开发规范：

###### 防止NPE是程序员的基本修养，对于可能产生NPE的场景，下面的说法符合规范：

- 数据库的查询结果可能为空
- **远程调用返回对象，一律要求进行NPE判断**
- 对于Session中获取的数据，建议NPE检查，避免空指针。

###### 关于数据库查询的规范：

- **绝对禁止左模糊和全模糊查询，限制使用右模糊查询。全模糊或左模糊搜索的需求走搜索引擎实现**
- 禁止使用存储过程来查询返回数据。

###### 集合处理：

- 关于hashcode和equals的处理，遵顼如下规则：

    1. 只要覆写equals，就必须覆写hashcode。

    2. 因为set存储的是不重复的对象，依据hashcode和equals进行判断，所以hashset存储的对象必须覆写这两个方法。

    3. 如果自定义对象作为map的键，那么必须覆写hashcode和equals。

       说明：String已经重写了hashcode和equals，所以可以愉快地在map中用String做key。

- 使用map的方法keyset()/values()/entrySrt()返回集合对象时，不可以对其进行添加元素操作，否则会抛出UnsupportedOperationException异常。

- 使用集合转数组的方法，必须使用集合的toArray(T[] array), **传入的是类型完全一致，长度为0的空数组。**

    - 反例：直接使用toArray无参方法的话返回值是Object[]类，若强转其他类型数组将出现ClassCastException错误。
    - 正例：String[] array = list.toArray(new String[0]);

- 使用Arrays.asList把数组转换成集合时对其中的元素只能读，不能用修改集合相关的方法对其写:add/remove/update，因为转换之后的数据结构不是List，而是Arrays的内部类。

- 不要在foreach里面进行元素的remove/add操作。remove元素请使用Iterator方式，如果并发操作，需要对Iterator对象加锁。

- 合理利用好集合的有序性(sort)和稳定性(order),避免集合的无序性(unsort)和不稳定性(unorder)带来的负面影响

    - 有序性是指遍历的结果是按某种比较规则依次排列的。稳定性是指集合每次遍历的元素次序是一定的。如：ArrayList是order/unsort;HashMap是unorder/unsort;TreeSet是order/sort。

- 如果是JDK8的应用，可以使用Instant代替Date，LocalDateTime代Calender,DateTimeFormatter代替SimpleDateFormat。

###### 并发编程：

- 在使用阻塞等待获取锁的方式中， 必须在try代码块之外，并且在加锁方法与try代码块之间没有任何可能抛出异常的方法调用，避免加锁成功后，在finally中无法解锁。（**直接加锁后进入try代码块**）
    - 说明一：如果在lock方法与try代码块之间的方法调用抛出异常，那么无法进入finally解锁，造成其他线程无法成功获取锁。
    - 如果lock方法在try代码块内，可能由于其他方法抛出异常，导致在finally代码块中，unlock对未加锁的对象解锁。
    - ![](images\加锁规范1.png)
    - ![](images\加锁规范反例.png)

###### 异常处理：

- 不要在finally块中使用return：try块中的return语句执行成功后，并不马上返回，而是继续执行finally块中的语句，如果此处存在return语句，则在此直接返回，无情丢掉try块中的返回点。
- 捕获异常与抛异常，必须是完全匹配，或者捕获异常是抛的异常的父类
    - 说明：如果预期对方抛的是绣球，实际接到的是铅球，就会发生意外情况。
- 在调用RPC、二方包、或动态生成类的相关方法（反射）时，捕捉异常必须使用throwable类来进行拦截。
    - ![](images\throwable.png)
- 定义时区分unchecked/checked异常，避免直接抛出new RuntimeException(),更不允许抛出Exception或者Throwable,应使用有业务含义的自定义异常。推荐业界已定义过的自定义异常，如：DAOException/ServiceException等。

###### 日志规约：

- 应用中不可直接使用日志系统(Log4j、Logback)中的API，而应依赖使用日志框架SLF4J中的API，使用门面模式的日志框架，有利于维护和各个类的日志处理方式统一。

    - ```java
    private static final Logger logger = LoggerFactory.getLogger(Test.class);
    ```

- 在日志输出时，字符串变量之间的拼接使用占位符的方式

    - 说明：因为String字符串的拼接会使用StringBuilder的append()方式，有一定的性能损耗。使用占位符仅是替代工作，可以有效提升性能。

    - ```java
    logger.debug("Processing trade with id: {} and symbol: {}",id,symbol);
    ```

*总结起来，响应式编程（reactive programming）是一种基于数据流（data stream）和变化传递（propagation of change）的声明式（declarative）的编程范式。*

> 假设Java引入了一种新的赋值方式`:=`，表示一种对a的绑定关系，如
>
> ```
> a = 1;
> b := a + 1;
> a = 2;
> ```

由于b保存的不是某次计算的值，而是针对a的一种绑定关系，所以b能够随时根据a的值的变化而变化，这时候`b==3`，我们就可以说`:=`是一种声明式赋值方式。

响应式编程的“变化传递”就相当于果汁流水线的管道；在入口放进橙子，出来的就是橙汁；放西瓜，出来的就是西瓜汁，橙子和西瓜、以及机器中的果肉果汁以及残渣等，都是流动的“数据流”；管道的图纸是用“声明式”的语言表示的。

### 函数式编程：

- Consumer:根据入参进行操作，无出参。无返回值：foreach collect
- Function：根据入参进行操作，返回另一个结果。返回另一个结果：map(直接返回)
- Predicate：对入参进行判断，返回判断后的结果。返回筛除后的list：filter

###### Stream中间操作函数：

![](images\stream_mid.png)

###### Stream终端操作函数：

![](images\stream_end.png)

###### flatMap：

元素一对多转换：对原Stream中的所有元素使用传入的Function进行处理，**每个元素经过处理后生成多个元素**，然后将所有元素组合成一个新的Stream返回

> 假设要对一个String类型的Stream进行处理，将每一个元素都拆分成单个字母，并打印：
>
> ```java
> Stream<String> s = Stream.of("test", "t1", "t2", "teeeee", "aaaa");
> s.flatMap(n -> Stream.of(n.split(""))).forEach(System.out::println);
> ```

### Optional：

> Optional用于简化Java中对空值的判断处理，以防止出现各种空指针异常。  Optional实际上是对一个变量进行封装，它包含有一个属性value，实际上就是这个变量的值。

Optional的构造函数都是private类型的，因此要初始化一个Optional的对象需要通过他一系列的静态方法。

![](images\Optional.png)

- Optional.of(obj)：它要求传入的 obj 不能是 null 值的, 否则直接报NullPointerException 异常。
- Optional.ofNullable(obj)：它以一种智能的，宽容的方式来构造一个 Optional 实例。来者不拒，传 null 进到就得到 Optional.empty()，非 null 就调用 Optional.of(obj).
- Optional.empty()：返回一个空的 Optional 对象

```java
String optional = Optional.nullable(user)
    					  .map()
    					  .flter()
    					  .orElse("error");
```

**重要的一点是 Optional 不是 Serializable。因此，它不应该用作类的字段。Optional 主要用作返回类型，在获取到这个类型的实例后，如果它有值，你可以取得这个值，否则可以进行一些替代行为。**

*基本套路用法：先拿到需要判空对象的nullable，然后进行一系列判断之后再orElse。按照这种思路，我们可以安心的进行链式调用，而不是一层层判断了。*

- 避免使用Optional作为类或者实例的属性，而应该在返回值中用来包装返回实例对象。
- 只有每当结果不确定时，使用Optional作为返回类型，从某种意义上讲，这是使用Optional的唯一好地方，用java官方的话讲就是：**我们的目的是为库方法的返回类型提供一种有限的机制，其中需要一种明确的方式来表示“无结果”，并且对于这样的方法使用null 绝对可能导致错误。**

```java
Optional
 .ofNullable(someVariable)
 .map(variable -> {
   try{
      return someREpozitory.findById(variable.getIdOfOtherObject());
   } catch (IOException e){
     LOGGER.error(e); 
     throw new RuntimeException(e); 
   }})
 .filter(variable -> { 
   if(variable.getSomeField1() != null){
     return true;
   } else if(variable.getSomeField2() != null){
     return false;   
   } else { 
     return true;
   }
  })
 .map((variable -> {
   try{
      return jsonMapper.toJson(variable);
   } catch (IOException e){
     LOGGER.error(e); 
     throw new RuntimeException(e); 
   }}))
 .map(String::trim)
 .orElseThrow(() -> new RuntimeException("something went horribly wrong."))

```



**better:**

```java
Optional
 .ofNullable(someVariable)
 .map(this::findOtherObject)
 .filter(this::isThisOtherObjectStale)
 .map(this::convertToJson)
 .map(String::trim)
 .orElseThrow(() -> new RuntimeException("something went horribly wrong."));

```





### 链式编程：

实体bean中类加入@Accessors(chain = true)即可使成员变量setter方法实现链式编程。

##### 构建者模式：

```java
public class StudentBean {
	private String name;
	
	private int age;
 
	public String getName() {
		return name;
	}
 
	public void setName(String name) {
		this.name = name;
	}
 
	public int getAge() {
		return age;
	}
 
	public void setAge(int age) {
		this.age = age;
	}
	
	public static Builder builder() {
		return new Builder();
	}
	
	public static class Builder{
		private String name;
		
		private int age;
 
		public Builder setName(String name) {
			this.name = name;
			return this;
		}
 
		public Builder setAge(int age) {
			this.age = age;
			return this;
		}
		
		public StudentBean build() {
			StudentBean studentBean = new StudentBean();
			studentBean.setName(name);
			studentBean.setAge(age);
			return studentBean;
		}
	}
}
```

### Mybatis

##### foreach标签：

```java
<foreach collection="list" item="item" separator=",">
		（#{item}）
</foreach>
```

这种形式一般用于数据库的批量增加

```sql
insert into xxx values (),(),()
```



```java
<foreach collection="list" item="item" open="(" separator="," close=")">
		#{item}
</foreach>
```

这种形式被用来构建in条件语句。

```sql
SELECT * FROM user WHERE user_name in ('zhangsan','lisi','wangwu')
```



| 标签      | 描述                                                         |
| :-------- | ------------------------------------------------------------ |
| open      | 表示该语句以什么开始，最常用的是左括弧’(’，注意:mybatis会将该字符拼接到整体的sql语句之前，并且只拼接一次，该参数为可选项 |
| close     | 表示该语句以什么结束，最常用的是右括弧’)’，注意:mybatis会将该字符拼接到整体的sql语句之后，该参数为可选项 |
| item      | 表示本次迭代获取的元素，若collection为List、Set或者数组，则表示其中的元素；若collection为map，则代表key-value的value，该参数为必选 |
| separator | mybatis会在每次迭代后给sql语句append上separator属性指定的字符，该参数为可选项 |
| index     | 在list、Set和数组中,index表示当前迭代的位置，在map中，index代指是元素的key，该参数是可选项。 |

### DTO

> model实体类是与数据库表字段一一对应的，所以要实现序列化接口，为了接收数据库表转化为Java对象。而DTO是数据传输对象，是根据前端的需求来对model进行CRUD字段的对象。

​		表现层与应用层之间是通过数据传输对象（DTO）进行交互的，数据传输对象是没有行为的POJO对象，它的目的只是为了对实体对象进行**数据封装**，实现层与层之间的数据传递。

###### 为何不能直接将实体对象用于 数据传递？

​		因为实体对象更注重实体本身，而DTO更注重数据。不仅如此，由于“富领域模型”的特点，**这样做会直接将领域对象的行为暴露给表现层**。

​		对于绝大部分的应用场景来说，DTO和VO的属性值基本是一致的，而且他们通常都是POJO，因此没必要多此一举，但不要忘记这是实现层面的思维，对于设计层面来说，概念上还是应该存在VO和DTO，因为两者有着本质的区别，DTO代表服务层需要接收的数据和返回的数据，而VO代表展示层需要显示的数据。
前端如果能将DTO