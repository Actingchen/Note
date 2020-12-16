#Spring





## Spring context

类似进程在执行程序（不管是用户程序还是内核中的程序）时，都会依赖一个上下文，这个上下文提供我们运行时需要的一些数据和保存运行时的一些数据。如果执行中遇到了中断，CPU会从用户态切换到内核态，此时进程处于的进程上下文会被切换到中断上下文中，从而可以根据中断号去执行相应的中断程序。

spring的ioc也是程序，它的执行也需要一个上下文。spring context就是spring的上下文。spring context是在core封装包基础上的context封装包，提供了一种框架式的对象访问方法。

主要包括：

- DefaultListableBeanFactory
  这就是大家常说的 ioc 容器，它里面有很多 map、list。spring 帮我们创建的 singleton 类型的 bean 就存放在其中一个 map 中。我们定义的监听器（ApplicationListener）也被放到一个 Set 集合中。
- BeanDefinitionRegistry
  把一个 BeanDefinition 放到 beanDefinitionMap。
- AnnotatedBeanDefinitionReader
  针对 AnnotationConfigApplicationContext 而言。一个 BeanDefinition 读取器。
- 扩展点集合
  存放 spring 扩展点（主要是 BeanFactoryPostProcessor、BeanPostProcessor）接口的 list 集合。





**初始化和启动**

我们平时常说的spring 启动其实就是调用 AbstractApplicationContext#refresh 完成 spring context 的初始化和启动过程。spring context 初始化从开始到最后结束以及启动，这整个过程都在 refresh 这个方法中。refresh 方法刚开始做的是一些 spring context 的准备工作，也就是 spring context 的初始化，比如：创建 BeanFactory、注册 BeanFactoryPostProcessor 等，只有等这些准备工作做好以后才去开始 spring context 的启动。

与现实生活联系一下，你可以把初始化理解为准备原料（对应到编程中就是创建好一些数据结构，并为这些数据结构填充点数据进去），等准备了你才能去真正造玩偶、造东西呀（对应到编程中就是执行算法）。在编程中数据结构与算法是分不开的也是这个道理呀，它们相互依赖并没有严格的界限划分。

 **运行**

spring context 启动后可以提供它的服务的这段时间。

**关闭/销毁**

不需要用 spring context ，关闭它时，其实对应到代码上就是 acaContext.close();

> 用户态和内核态
>
> 内核态（Kernel Mode）：运行操作系统程序，操作硬件
>
> 用户态（User Mode）：运行用户程序
>
> 指令区别：
>
> 特权指令：只能由操作系统使用、用户程序不能使用的指令。 举例：启动I/O 内存清零 修改程序状态字 设置时钟 允许/禁止终端 停机
>
> 非特权指令：用户程序可以使用的指令。 举例：控制转移 算数运算 取数指令 **访管指令**（使用户程序从用户态陷入内核态）
>
> 特权级别：
>
> R0相当于内核态，R3相当于用户态；
>
> 区别：
>
> - 处于用户态执行时，进程所能访问的内存空间和对象受到限制，其所处于占有的处理器是可被抢占的
> - 处于内核态执行时，则能访问所有的内存空间和对象，且所占有的处理器是不允许被抢占的。
>
> 用户态什么时候变成内核态：
>
> 1、系统调用
>
> 2、异常
>
> 3、外围设备的中断

**扫盲**

https://javadoop.com/post/spring-ioc



![](https://www.javadoop.com/blogimages/spring-context/1.png)

```java
ApplicationContext context = new ClassPathXmlApplicationContext("classpath:applicationfile.xml");

ApplicationContext ctx=new FileSystemXmlApplicationContext( "G:/Test/applicationcontext.xml ");   

```

>获取ApplicationContext：
>
>**ClassPathXmlApplicationContext**从classpath下加载配置文件(适合于相对路径方式加载)
>
>**FileSystemXmlApplicationContext** 的构造函数需要一个 xml 配置文件在系统中的路径，即从文件绝对路径加载配置文件，其他和 ClassPathXmlApplicationContext 基本上一样。
>
>WebXmlApplicationContext：此容器加载一个XML文件，此文件定义了一个WEB应用的所有 bean。
>
>**AnnotationConfigApplicationContext** 是基于注解来使用的，它不需要配置文件，采用 java 配置类和各种注解来配置，简洁简单

> BeanFactory，从名字上也很好理解，生产 bean 的工厂，它负责生产和管理各个 bean 实例。

> BeanDefinition 中保存了我们的 Bean 信息，比如这个 Bean 指向的是哪个类、是否是单例的、是否懒加载、这个 Bean 依赖了哪些 Bean 等等。

> Spring IoC容器允许BeanFactoryPostProcessor在容器实例化任何bean之前读取bean的定义(配置元数据)，并可以修改它。同时可以定义BeanFactoryPostProcessor，通过设置’order’属性来确定各个BeanFactoryPostProcessor执行顺序。    注册一个BeanFactoryPostProcessor实例需要定义一个Java类来实现BeanFactoryPostProcessor接口，并重写该接口的postProcessorBeanFactory方法。通过beanFactory可以获取bean的定义信息，并可以修改bean的定义信息。这点是和BeanPostProcessor最大区别,

![](https://www.javadoop.com/blogimages/spring-context/2.png)

1. ApplicationContext 继承了 ListableBeanFactory，这个 Listable 的意思就是，通过这个接口，我们可以获取多个 Bean，大家看源码会发现，最顶层 BeanFactory 接口的方法都是获取单个 Bean 的。
2. ApplicationContext 继承了 HierarchicalBeanFactory，Hierarchical 单词本身已经能说明问题了，也就是说我们可以在应用中起多个 BeanFactory，然后可以将各个 BeanFactory 设置为父子关系。
3. AutowireCapableBeanFactory 这个名字中的 Autowire 大家都非常熟悉，它就是用来自动装配 Bean 用的，但是仔细看上图，ApplicationContext 并没有继承它，不过不用担心，不使用继承，不代表不可以使用组合，如果你看到 ApplicationContext 接口定义中的最后一个方法 getAutowireCapableBeanFactory() 就知道了。
4. ConfigurableListableBeanFactory 也是一个特殊的接口，看图，特殊之处在于它继承了第二层所有的三个接口，而 ApplicationContext 没有。这点之后会用到。

## IOC

由于获取资源（数据库、读取文件、对象），类内部主动创建依赖对象，从而导致类与类之间高耦合,局部变更可能影响全部，直到IOC/DI出现。

* **IOC**：以前创建对象是由程序主动创建对象、主动去控制获取依赖对象，类之间耦合度高，难于测试，现在IOC利用反射机制把创建对象的控制权交给Spring IOC容器来创建（时机）、管理（对象的生命周期）、装配（通过依赖注入，配置对象）。
* **DI Dependency Injection** 依赖注入：容器能知道哪个组件（类）运行的时候，需要或者说是依赖另一个类（组件）；容器通过反射的形式，将准备好的BookService对象注入（利用反射给属性赋值）到BookServlet中。

### IOC优点

* 对程序最小的代价和最小的入侵性实现松散耦合
* 由于降低了耦合，测试更容易了
* IOC容器支持加载服务的饿汉式初始化和懒加载
* 不用关注对象的创建具体实现，我们只需要预先配置好，需要的时候使用即可，没有IOC的话我们需要底层查看并按要求用构造参数创建

本质上用的是工厂模式和反射机制

举个例子

```java
interface Fruit {
	public abstract void eat();
}
class Apple implements Fruit {
	public void eat(){
		System.out.println("Apple");
	}
}
class Orange implements Fruit {
	public void eat(){
		System.out.println("Orange");
	}
}
class Factory {
	public static Fruit getInstance(String ClassName) {
		Fruit f=null;
		try {
		f=(Fruit)Class.forName(ClassName).newInstance();
	} catch (Exception e) {
		e.printStackTrace();
		}
	return f;
	}
}
class Client {
	public static void main(String[] a) {
		Fruit f=Factory.getInstance("io.github.dunwu.spring.Apple");
		if(f!=null){
			f.eat();
		}
	}
}
```

### IOC功能

* 依赖注入
* 依赖检查
* 自动装配
* 指定初始化方法和销毁方法
* 支持回调某些方法，但是需要实现spring接口

其中，最重要的就是依赖注入，从 XML 的配置上说，即 ref 标签。对应 Spring RuntimeBeanReference 对 象。 对于 IoC 来说，最重要的就是容器。容器管理着 Bean 的生命周期，控制着 Bean 的依赖注入

### DI的优点

* 查找定位操作与应用代码完全无关。 

* 不依赖于容器的API，可以很容易地在任何容器以外使用应用对象。 

* 不需要特殊的接口，绝大多数对象可以做到完全不必依赖容器。

### DI的注入实现方式

依赖注入是时下最流行的IoC实现方式，依赖注入分为接口注入（Interface Injection），Setter方法注入 （Setter Injection）和构造器注入（Constructor Injection）三种方式。

其中接口注入由于在灵活性和易用 性比较差，现在从Spring4开始已被废弃。

构造器依赖注入：构造器依赖注入通过容器触发一个类的构造器来实现的，该类有一系列参数，每个参 数代表一个对其他类的依赖。

Setter方法注入：Setter方法注入是容器通过调用无参构造器或无参static工厂 方法实例化bean之后，调 用该bean的setter方法，即实现了基于setter的依赖注入。

| 构造函数注入               | Setter注入                 |
| -------------------------- | -------------------------- |
| 没有部分注入               | 有部分注入                 |
| 不会覆盖setter属性         | 会覆盖setter属性           |
| 任意修改都会创建一个新实例 | 任意修改不会创建一个新实例 |
| 适用于设置很多属性         | 适用于设置少量属性         |

两种依赖方式都可以使用，构造器注入和setter方法注入。==最好的解决方法使用构造器参数实现强制依赖，setter实现可选依赖==

### Springbean 作用域

- Singleton，这是 Spring 的默认作用域，也就是为每个 IOC 容器创建唯一的一个 Bean 实例。
- Prototype，针对每个 getBean 请求，容器都会单独创建一个 Bean 实例。

从 Bean 的特点来看，Prototype 适合有状态的 Bean，而 Singleton 则更适合无状态的情况。另外，使用 Prototype 作用域需要经过仔细思考，毕竟频繁创建和销毁 Bean 是有明显开销的。

如果是 Web 容器，则支持另外三种作用域：

- Request，为每个 HTTP 请求创建单独的 Bean 实例。
- Session，很显然 Bean 实例的作用域是 Session 范围。
- GlobalSession，用于 Portlet 容器，因为每个 Portlet 有单独的 Session，GlobalSession 提供一个全局性的 HTTP Session。

###SpringIOC Bean加载 存储 获取

>1.进入`ClassPathXmlApplicationContext`的构造方法，最终会走他的父类`AbstractApplicationContext`的refresh方法。2.refresh方法会先new一个`DefaultListableBeanFactory`，之后读取和解析我们路径中的xml文件，然后将我们xml文件中配置的类信息解析成`BeanDefinition`，再放到`DefaultListableBeanFactory`的beanDefinitionMap中，也就是将我们配置的Bean进行一个注册。
>
>3.回到refresh方法中，接下来会执行postProcessBeanFactory方法注册`BeanFactoryPostProcessor`，postProcessBeanFactory方法是Spring提供的第一个扩展点。
>
>4.调用上一步注册的`BeanFactoryPostProcessor`的postProcessBeanFactory方法，一般可以在这个方法里去修改已经注册的`BeanDefinition`类信息。
>
>5.注册`BeanPostProcessor`，这是Spring提供的第二个扩展点。
>
>6.注册一些监听，事件啥的。
>
>7.会去实例化我们配置的非懒加载的单例Bean。这一步能再拆开来说。
>
>7.1会走`DefaultListableBeanFactory`的getBean方法，之后会走到父类`AbstractAutowireCapableBeanFactory`的doCreateBean方法里面，然后会先实例化Bean，实例化Bean的过程中会去判断Bean的创建方式，是通过Supplier创建，还是工厂方法创建，还是通过构造器创建。还会使用一个`BeanPostProcessor`去推断构造器。然后去实例化Bean。
>
>7.2之后会走populateBean去进行Bean属性的填充。
>
>7.3走initializeBean方法去调用注册的BeanProcessor，还有完成一些Bean生命周期的一些工作。7.4最后将Bean放进单例池里缓存起来。

###三级缓存如何解决循环依赖问题

https://www.cnblogs.com/zzq6032010/p/11406405.html

引入 了中间态的概念，即已经实例化但没有初始化，通过提前暴露来解决循环依赖问题

三大循环依赖

* 构造器注入循环依赖

```java
@Service
public class A {
    public A(B b) {
    }
}
@Service
public class B {
    public B(A a) {
    }
}
```



>构造器注入构成的循环依赖，此种循环依赖方式**是无法解决的**，只能抛出`BeanCurrentlyInCreationException`异常表示循环依赖。这也是构造器注入的最大劣势（它有很多独特的优势，请小伙伴自行发掘）

> `根本原因`：Spring解决循环依赖依靠的是Bean的“中间态”这个概念，而这个中间态指的是`已经实例化`，但还没初始化的状态。而构造器是完成实例化的东东，所以构造器的循环依赖无法解决~~~

* ##### field属性注入（setter方法注入）循环依赖

```java
@Service
public class A {
    @Autowired
    private B b;
}

@Service
public class B {
    @Autowired
    private A a;
}
```



> 成功

* ##### `prototype` field属性注入循环依赖

```java
@Scope(ConfigurableBeanFactory.SCOPE_PROTOTYPE)
@Service
public class A {
    @Autowired
    private B b;
}

@Scope(ConfigurableBeanFactory.SCOPE_PROTOTYPE)
@Service
public class B {
    @Autowired
    private A a;
}
```



> scope="prototype" 意思是 每次请求都会创建一个实例对象。
>
> 两者的区别是：有状态的bean都使用Prototype作用域，无状态的一般都使用singleton单例作用域。
>
> 对于“prototype”作用域Bean，Spring容器无法完成依赖注入，因为“prototype”作用域的Bean，Spring容器不进行缓存，因此无法提前暴露一个创建中的Bean。
>
> @Lazy无法解决，当你真正使用到它（初始化）的时候，依旧会报异常UnsatisfiedDependencyException: Error creating bean with name 



Spring创建Bean的流程

**AbstractBeanFactory的getBean跟doGetBean方法**

**AbstractAutowireCapableBeanFactory#createBean**

![](\images\Center)

![](https://img-blog.csdnimg.cn/20190619142513115.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2Y2NDEzODU3MTI=,size_16,color_FFFFFF,t_70)

- `createBeanInstance`：例化，其实也就是调用对象的**构造方法**实例化对象
- `populateBean`：填充属性，这一步主要是对bean的依赖属性进行注入(`@Autowired`)
- `initializeBean`：回到一些形如`initMethod`、`InitializingBean`等方法

```java
public class DefaultSingletonBeanRegistry extends SimpleAliasRegistry implements SingletonBeanRegistry {
	...
	// 从上至下 分表代表这“三级缓存”
	private final Map<String, Object> singletonObjects = new ConcurrentHashMap<>(256); //一级缓存
	private final Map<String, Object> earlySingletonObjects = new HashMap<>(16); // 二级缓存
	private final Map<String, ObjectFactory<?>> singletonFactories = new HashMap<>(16); // 三级缓存
	...
	
	/** Names of beans that are currently in creation. */
	// 这个缓存也十分重要：它表示bean创建过程中都会在里面呆着~
	// 它在Bean开始创建时放值，创建完成时会将其移出~
	private final Set<String> singletonsCurrentlyInCreation = Collections.newSetFromMap(new ConcurrentHashMap<>(16));

	/** Names of beans that have already been created at least once. */
	// 当这个Bean被创建完成后，会标记为这个 注意：这里是set集合 不会重复
	// 至少被创建了一次的  都会放进这里~~~~
	private final Set<String> alreadyCreated = Collections.newSetFromMap(new ConcurrentHashMap<>(256));
}
```

==解决循环依赖==

>1. 使用`context.getBean(A.class)`，旨在获取容器内的单例A(若A不存在，就会走A这个Bean的创建流程)，显然初次获取A是不存在的，因此走**A的创建之路~**
>
>2. `实例化`A（注意此处仅仅是实例化），并将它放进`缓存`（此时A已经实例化完成，已经可以被引用了）先通过createBeanInstance实例化A对象，又将该实例化的对象通过addSingletonFactory方法放入singletonFactories中，完成A对象早期的暴露
>
>3. `初始化`A：`@Autowired`依赖注入B（此时需要去容器内获取B）
>
>4. 为了完成依赖注入B，会通过`getBean(B)`去容器内找B。但此时B在容器内不存在，就走向**B的创建之路~**
>
>5. `实例化`B，并将其放入缓存。（此时B也能够被引用了）
>
>6. `初始化`B，`@Autowired`依赖注入A（此时需要去容器内获取A）
>
>7. `此处重要`：初始化B时会调用`getBean(A)`去容器内找到A，上面我们已经说过了此时候因为A已经实例化完成了并且放进了缓存singletonFactories中里，所以这个时候去看缓存里是已经存在A的引用了的，所以`getBean(A)`能够正常返回
>
>8. **B初始化成功**（此时已经注入A成功了，已成功持有A的引用了），return（注意此处return相当于是返回最上面的`getBean(B)`这句代码，回到了初始化A的流程中~）。
>
>9. 因为B实例已经成功返回了，因此最终**A也初始化成功**
>
>10. **到此，B持有的已经是初始化完成的A，A持有的也是初始化完成的B，完美~**
>
>    总结：Spring通过将实例化后的对象提前暴露给Spring容器中的singletonFactories，解决了循环依赖的问题

去除第三级是否可以？

**可以的**

但是二级无法实现代理对象的循环引用部分

### Spring Bean的生命周期

细分的话

1. 实例化 Instantiation
2. 属性赋值 Populate
3. 初始化 Initialization
4. 销毁 Destruction

另外、还有扩展点

![](https://upload-images.jianshu.io/upload_images/4558491-dc3eebbd1d6c65f4.png?imageMogr2/auto-orient/strip|imageView2/2/w/823/format/webp)

![image-20201216155819530](\images\image-20201216155819530.png)

影响多个Bean

- BeanPostProcessor
- InstantiationAwareBeanPostProcessor

影响单个Bean

- Aware
  - Aware Group1
    - BeanNameAware
    - BeanClassLoaderAware
    - BeanFactoryAware
  
  比如：
  
  ```java
  //已知
  public interface BeanNameAware extends Aware {
        void setBeanName(String name);
  }
  
  //定义一个用户类，实现BeanNameAware接口，重写setBeanName方法，此时bean会感知到自身的BeanName（对应Spring容器的BeanId属性）属性
  ```
  
  - Aware Group2
    - EnvironmentAware
    - EmbeddedValueResolverAware
    - ApplicationContextAware(ResourceLoaderAware\ApplicationEventPublisherAware\MessageSourceAware)
  
- 生命周期
  - InitializingBean
  - DisposableBean

Spring Bean 生命周期比较复杂，可以分为创建和销毁两个过程。首先，创建 Bean 会经过一系列的步骤，主要包括：

第一，创建

* 实例化Bean对象
* 设置Bean属性
* 如果我们通过各种Aware接口声明了依赖关系，则会出注入Bean对容器基础设施层面的依赖。具体包括BeanNameAware、BeanFactoryAware和ApplicationContextAware,分别会注入Bean ID、Bean Factory或者ApplicationContext
* 调用BeanPostProcessor的前置初始化方法postProcessBeforeInitialization
* 如果实现了InitializingBean接口、则会调用afterPropertiesSet方法
* 调用Bean自身定义的init方法
* 调用BeanPostProcessor的后置初始化方法postProcessAfterInitialization
* 创建过程完毕

第二，Spring Bean 的销毁过程会依次调用 DisposableBean 的 destroy 方法和 Bean 自身定制的 destroy 方法。

https://www.jianshu.com/p/1dec08d290c1

- postProcessBeforeInstantiation调用点，忽略无关代码：

```java
@Override
    protected Object createBean(String beanName, RootBeanDefinition mbd, @Nullable Object[] args)
            throws BeanCreationException {

        try {
            // Give BeanPostProcessors a chance to return a proxy instead of the target bean instance.
            // postProcessBeforeInstantiation方法调用点，这里就不跟进了，
            // 有兴趣的同学可以自己看下，就是for循环调用所有的InstantiationAwareBeanPostProcessor
            Object bean = resolveBeforeInstantiation(beanName, mbdToUse);//《《《《重点》》》》
            if (bean != null) {
                return bean;
            }
        }
        
        try {   
            // 上文提到的doCreateBean方法，可以看到
            // postProcessBeforeInstantiation方法在创建Bean之前调用
            Object beanInstance = doCreateBean(beanName, mbdToUse, args);
            if (logger.isTraceEnabled()) {
                logger.trace("Finished creating instance of bean '" + beanName + "'");
            }
            return beanInstance;
        }
        
    }

```

###ApplicationContext和BeanFactory的区别？





## AOP

AOP 本质上是==通过**预编译方式**和**运行期动态代理**实现在**不修改源代码**的情况下给程序**动态统一添加功能**的一种技术。==

> 横向关注点：跨越应用程序多个模块的方法或功能。与我们业务代码无关，但是我们需要关注的部分，就是横切关注点。比如，日志，安全、缓存、事务等等

- **切面：**（Aspect) 一个关注点的模块化，就比较笼统的一个概念，关注点可能横切多个对象。若不理解请往后看图片理解，对应的注解有@Aspect。
- **连接点：**（Joinpoint) 在程序执行过程中某个特定的点，一个连接点总是代表一个方法的执行。
- **通知：**（Advice) 通知表示在一个连接点执行的具体的动作，比如After Before 表明通知的具体动作
- **切入点：**（Pointcut）通过一个表达式去表明我所定义的通知在那个地点具体执行。
- **前置通知：**（Before advice）表明在连接点执行之前执行的动作。
- **后置通知：**（After returning advice）在某个连接点完成后的通知，比如一个方法没有抛出任何异常，正常返回。
- **环绕通知：**（Around Advice) 环绕可以看作是包含前置通知和后置通知的一个通知，先了解，后面具体理解。
- **异常通知：**（After throwing advice) 在方法异常推出时候执行的通知。
- **最终通知：**（After advice) 在连接点退出时候执行的通知。不论是正常退出还是异常退出。

本质上用的是代理模式

####静态代理

好处：

* 可以使真实角色的操作更加纯粹！不用去关注一些公共的业务
* 公共也就就交给代理角色！实现业务的分工！
* 公共业务发生扩展的时候，方便集中管理

缺点

* 一个真实角色就会产生一个代理角色；代码量会翻倍~开发效率会变低

####动态代理

好处：

* 可以使真实角色的操作更加纯粹！不用去关注一些公共的业务
* 公共也就就交给代理角色！实现业务的分工！
* 公共业务发生扩展的时候，方便集中管理
* 一个动态代理类 代理的是一个接口，一般就是对应的一类业务
* 一个动态代理类 可以代理多个类 只要实现了同一个接口即可

## 事务隔离级别



## 事务传播行为



##依赖注入装配Bean 基于注解

- 注解：就是一个类，使用@注解名称
- 开发中：使用注解 取代 xml配置文件。
  1.`@Component取代<bean class="">`
  `@Component("id") 取代 <bean id="" class="">`
  2.web开发，提供3个@Component注解衍生注解（功能一样）取代
  `@Repository ：dao层`
  `@Service：service层`
  `@Controller：web层`
  3.依赖注入，给私有字段设值，也可以给setter方法设值
  - 普通值：`@Value(" ")`
  - 引用值：
    `方式1：按照【类型】注入@Autowired方式2：按照【名称】注入1@Autowired@Qualifier("名称")方式3：按照【名称】注入2@Resource("名称")`
    4.生命周期
    `初始化：@PostConstruct销毁：@PreDestroy`
    5.作用域
    `@Scope("prototype") 多例`

###@Resource和@Autowired的区别

* 都是用来自动装配的，都可以放在属性字段上
* @Autowired通过byType的方式实现，而且必须要求这个对象存在！
* @Resource 默认通过byname的方式实现，如果找不到名字，则通过byType实现！如果两个都找不到的情况下就报错。
* 执行顺序不同

###@Component相当于bean；有几个衍生注解，

* dao【@Repository】
* service【@Service】
* controller【@Controller】

四个注解都是把类注册到spring容器中

