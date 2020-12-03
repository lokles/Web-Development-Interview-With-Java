# Spring问题

## 1、http请求过来springMVC是怎么处理的？

1. 请求被Spring 前端控制器 `DispatcherServlet` 捕获
2. `DispatcherServlet`对请求URL进行解析（核心方法`doDispatch()`），得到请求资源标识符（URI）然后根据该URI，调用`HandlerMapping`获得该`Handler（Controller)`配置的所有相关的对象
3. `DispatcherServlet` 根据获得的Handler，选择一个合适的`HandlerAdapter（执行目标方法的反射工具）`
4. 提取Request中的模型数据，填充Handler入参，开始执行Handler
5. Controller -> Service -> Dao 查询到数据
6. Controller执行完成后，向`DispatcherServlet` 返回一个`ModelAndView`对象
7. 根据返回的`ModelAndView`，选择一个适合的`ViewResolver`(视图解析器)
8. `ViewResolver` 结合Model和View，来渲染视图
9. 由`DispatcherServlet` 响应给客户端

## 2、视图解析器怎么解析的？

1. 任何方法的返回值最终都会封装为`ModelAndView`对象。
2. `viewResolver`的**唯一作用**是根据`ModelAndView`得到`view`对象，视图对象才能真正的转发或者重定向到页面（并将模型中的数据暴露到请求域中）。
3. 视图对象是真正进行视图渲染的。调用view的方法：render(`ModelAndView`, request, response) 进行页面渲染。
4. 根据view的种类不同它渲染出来的视图也不相同。一般我们使用的都是`InternalResourceView`视图。

## 3、注解实现的原理？如果让你实现一个注解你会怎么做？

Annotation其实是一种接口。通过java的反射机制相关的API来访问Annotation信息。相关类（框架或工具中的类）根据这些信息来决定如何使用该程序元素或改变它们的行为。



## 4、spring循环依赖？

多个bean之间的互相引用，导致一个闭环的出现。

采用**三级缓存模式**来解决循环依赖问题。

```Java
singletonFactories ： //单例对象工厂的cache
earlySingletonObjects ：//提前暴光的单例对象的Cache
singletonObjects：//单例对象的cache
```

注意：构造器注入导致的循环依赖无法解决。

假设现在有两个bean X Y互相依赖，且都是单例的，X开始生命周期后直到X通过构造器以及创建对象后，会有一个暴露阶段，此时会将X的一个`ObiectFcatory`对象暴露出去并存入二级缓存中。然后会进行X的属性注入，这是会将Y注入，但是还没有Y，然后进入到Y bean的生命周期。一直到Y暴露出自己的`ObjectFcatory`对象暴露出去并存入二级缓存中后，Y进行依赖注入，需要注入X，然后二级缓存中有X的一个对应的工厂对象。至此完成了循环依赖。需要注意的是此过程仅适用于由于属性注入引起的循环依赖，对于由于构造器注入引起的循环依赖不能解决，原因是`ObiectFcatory`对象是在根据构造器通过反射创建对象后才产生的。对于构造器注入引起的循环依赖无法起作用。

## 5、spring bean 生命周期？

#### 1、实例化过程

**1.1** 首先spring通过`BeanDefinitionReader`会将xml、Java类型的配置文件解析为`BeanDefinition`类型注册到容器中。`BeanDefinition`实际上是一个用来存储class信息的对象。它里面包含了一个类的基本信息、类的父类的信息、是否懒加载、是否为单例等等。`beandifinition`定义了bean的基本信息，根据它来创造bean然后<`BeanDefinition`，`beanName`>分别作为<value,key>存入一个map中。这个map是存在于`BeanFactory`中的。

**1.2** `BeanDefinition`会转化为`mergebeandefinition`，其中包括了`BeanDefinition`以及 parent `BeanDefinition`的信息。

**1.3** 配置 `BeanDefinition` 的depends-on `BeanDefinition`

**1.4** 根据`BeanDefinition`中指定的class信息，以及构造器信息最终**通过反射获取到`BeanDefinition`的实列对象**（注意此时还不是一个bean，经过后续的一些操作才会变成一个完整的bean）。

**1.5** 判断对象是否允许循环依赖？是否需要AOP，**属性注入。**

1.6 判断是否需要暴露。需要的话会将一个objectFactory对象存入一个二级缓存中。

**1.7 spring bean 的实例化完成，加入到spring 的单例缓冲池中（一个map）**。

#### 2、初始化过程

调用`init-method`进行bean的初始化，主要用于项目的一些依赖（配置文件或者数据库连接等等）

#### 3、 销毁bean

`destory-metnod`方法进行bean的销毁。

## 6、spring容器启动流程？

1. 将配置文件加载为`BeanDefinition`并注册到容器中；这一步需要`XmlBeanDefinitionReader`的配合。
2. 注册`BeanFactoryPostProcesser`，包含一个可以传入`beanFactory`引用的方法，获取到容器之后可以做很多事情。由于`BeanFactoryPostProcesser`工作在bean实例化之前，所以可以通过`beanFactory`获取到map从而手动修改或者移除`beanDefinition`
3. 注册`BeanPostProcesser`，工作于bean实例化或者初始化前后。其包含两个方法。是spring作为扩展接口留给开发人员使用的。

```Java 
public interface BeanPostProcessor {
	//在初始化之前调用
	Object postProcessBeforeInitialization(Object bean, String beanName) throws BeansException;
	//在初始化之后调用
	Object postProcessAfterInitialization(Object bean, String beanName) throws BeansException;
}
```

4. 创建事件传播器对象

5. `beanDefinition`实例化

## 7、BeanFactory和ApplicationContext有什么区别？

`BeanFactory：`是Spring里面最底层的接口，提供了最简单的容器的功能，包含了各种Bean的定义，读取bean配置文件，管理bean的加载、实例化、控制bean的生命周期、维护bean之间的依赖关系等等。

`**ApplicationContext：**`继承了`BeanFactory`接口。是spring中更高一级的容器。提供了比`BeanFactory`更多的功能。

**区别：**

`BeanFactory`采用懒加载形式注入bean，`ApplicationContext`在容器启动时一次性创建所有bean，这样在容器启动时就可以发现Spring中存在的配置错误。`ApplicationContext`启动后预载入所有的单实例Bean，通过预载入单实例bean ,确保当你需要的时候，你就不用等待，因为它们已经创建好了。

## 8、bean的作用域？

默认为单例的，可以通过xml文件中的scope标签来做更改。如原型模式、request、session。

## 9、spring基于XML文件注入bean的方式？

构造器注入：无参构造器注入，有参构造器注入。

set方法注入：要求被注入的属性必须有set方法。

## 10、spring的自动装配？

**Spring自动将某个bean的引用装配给了指定属性，这一过程叫做自动装配。**Spring提供了三种自动装配的策略。

```Java
    //无需自动装配
    int AUTOWIRE_NO = 0;
    //按名称自动装配bean属性
    int AUTOWIRE_BY_NAME = 1;
    //按类型自动装配bean属性
    int AUTOWIRE_BY_TYPE = 2;
    //按构造器自动装配
    int AUTOWIRE_CONSTRUCTOR = 3;
    //过时方法，Spring3.0之后不再支持
```

上面介绍的是基于xml配置文件的自动装配过程。下面介绍基于注解的自动装配过程。

**基于注解的自动装配：**

`@Autowired`注解可以实现bean的自动装配。默认是**按照类型**进行装配的。但是如果匹配到同一类型的多个实例，再通过`byName`来确定要装配的bean

## 11、SpringBoot的关键注解？

`@SpringBootApplication`启动类注解，等同于`@SpringBootConfiguration、 @EnableAutoConfiguration、 @ComponentScan` 这三个注解

## 12、Spring AOP是什么？

AOP意为面向切面编程，与OOP一样，是一种编程理念，如果把OOP看作是自上而下的层层抽象，那么AOP就是从左至右的相同功能模块的抽取和封装。开发中使用AOP可以大大减少冗余代码，降低模块之间的耦合度，并且有利于未来的扩展性。比如商城业务中好多的微服务模块都要先进行用户验证。我们就可以把验证用户这一功能抽取出来作为一个切面。

#### AOP 当中的概念：

- 切入点（Pointcut）
  在哪些类，哪些方法上切入（**where**）
- 通知（Advice）
  在方法执行的什么实际（**when:**方法前/方法后/方法前后）做什么（**what:**增强的功能）
- 切面（Aspect）
  切面 = 切入点 + 通知，通俗点就是：**在什么时机，什么地方，做什么增强！** 
- 织入（Weaving）
  把切面加入到对象，并创建出代理对象的过程。（由 Spring 来完成）

举一个例子：

![](C:\Users\lok666\Desktop\AOP举例.png)

```Java
@Component("landlord")
public class Landlord {
    // 下面方法是连接点
    public void service() {
        // 仅仅只是实现了核心的业务功能
        System.out.println("签合同");
        System.out.println("收房租");
    }
}
```

```Java
@Component // 标识为一个Bean
@Aspect // 标识为一个切面
class Broker {
    // 前置通知，表示在连接点方法之前执行
    // 定义了 execution 的正则表达式，Spring 通过这个正则表达式判断具体要拦截的是哪一个类的哪一个方法
    @Before("execution(* pojo.Landlord.service())")
    public void before(){
        System.out.println("带租客看房");
        System.out.println("谈价格");
    }
    // 后置通知，表示在切入点方法之后执行
    @After("execution(* pojo.Landlord.service())")
    public void after(){
        System.out.println("交钥匙");
    }
}
```



## 13、AspectJ是什么？与Spring AOP的区别？

AspectJ是AOP的一种实现，是目前Java开发社区中最流行的AOP框架，拥有更好的性能。

## 14、SpringMVC如何将纯文本的Http协议的请求转化为Java对象的？

利用`HttpMessageConverter`的实现类将http的请求转化为Java对象的。同时，响应的时候还可以利用`HttpMessageConverter`的实现类将Java对象转化为http响应的格式。

## 15、讲下SpringMVC的核心入口类是什么？

`DispatcherServlet`是`SpringMVC`的核心入口。

## 16、Spring 中的单例 Beans 是线程安全的么？

不是线程安全的，对于单例Bean，所有线程都共享一个单例实例Bean，因此是存在资源的竞争。

但如果单例Bean，是一个无状态Bean，也就是线程中的操作不会对Bean的成员执行**查询**以外的操作，那么这个单例Bean是线程安全的。比如`SpringMVC` 的 Controller、Service、Dao等，这些Bean大多是无状态的，只关注于方法本身。

## 17、spring定时任务？