## AOP:

切面 = 切点 + 通知

切面首先是一个被spring容器管理的bean，所以要加@Component注解。

如果一个bean的某个方法被aop所编织修饰，那么该bean在创建的时候在初始化后的过程中会有检查aop的一步操作过程，**最后生成的bean其实是这个普通bean的代理对象**。

检查aop：遍历所有的切面对象--->遍历所有切面对象的编织方法--->如果编织方法中excution表达式中有方法和当前生成bean的方法匹配（说明有一个切点要对当前bean的方法进行增强），则记录这个编织方法，将其放到缓存中（map）。在随后对该bean的方法进行aop增强中用到编织方法时只需从map拿即可。



#### 事务：

事务就是使用了aop，一个bean中如果有方法被事务修饰，则该bean最后得到的依然是一个代理对象，在对代理对象执行被事务修饰的方法时，就用到了aop。before：1.创建一个数据库连接（事务管理器创建）2.conn.autocommit=false

###### 更多的在@Configuration修饰的类中使用@Bean，而不是@Component

>  Full @Configuration vs 'lite' @Beans mode?
>
>  当@Bean方法在没有使用@Configuration注解的类中声明时，它们被称为以“lite”进行处理。例如，用@Component修饰的类或者简单的类中都被认为是“lite”模式。
>
>  不同于full @Configuration，lite @Bean 方法不能简单的在类内部定义依赖关系。通常，在“lite”模式下一个@Bean方法不应该调用其他的@Bean方法。
>
>  只有在@Configuration注解的类中使用@Bean方法是确保使用“full”模式的推荐方法。这也可以防止同样的@Bean方法被意外的调用很多次，并有助于减少在'lite'模式下难以被追踪的细小bug。

#### Spring生命周期

Spring注入依赖的时候首先根据类型去容器里面找，如果该类型的bean有且仅有一个就直接注入这个bean，如果有多个相同类型的bean再根据beanName确定唯一的一个bean。

@Resource是直接去容器里面寻找beanType和beanName都匹配的bean。



### 为什么用单例或者多例？何时用？

之所以用单例，是因为没必要每个请求都新建一个对象，这样子既浪费CPU又浪费内存；

之所以用多例，是为了防止并发问题；即一个请求改变了对象的状态，此时对象又处理另一个请求，而之前请求对对象状态的改变导致了对象对另一个请求做了错误的处理；

当对象含有可改变的状态时（更精确的说就是在实际应用中该状态会改变），则多例，否则单例；

##### 如何在单例bean中注入一个多例bean

如果我们需要给一个单例beanA注入一个多例beanB时（多例注入单例是没有问题的），仅仅通过配置Spring是无法帮我们的单例beanA注入一个多例beanB的，即无法让我们每次使用beanB时都使用的是一个全新的beanB。因为beanA只初始化一次，相对应的Spring只会给beanA注入一个beanB。**解决办法是给beanA注入一个BeanFactory，这样我们就可以在每次需要使用beanB时都从BeanFactory中获取一个新的beanB。**
用applicationContext或者BeanFactory get到的bean是多例bean；

### @Resource、@Autowired

@Autowired支持@primary：一个类型有多个实例时，默认注入配置了@primary的实例。

@Autowired默认按照类型注入，有多个类型的时候需要加上@Qualifier("name")注入指定实例。

@Resource装配顺序

1. 如果同时指定了name和type，则从Spring上下文中找到唯一匹配的bean进行装配，找不到则抛出异常.
2. 如果指定了name，则从上下文中查找名称（id）匹配的bean进行装配，找不到则抛出异常.
3. 如果指定了type，则从上下文中找到类型匹配的唯一bean进行装配，找不到或者找到多个，都会抛出异常.
4. **如果既没有指定name，又没有指定type，则自动按照byName方式进行装配；如果没有匹配，则回退为按照类型进行匹配；**