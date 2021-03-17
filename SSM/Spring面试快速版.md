#IOC

**--工厂模式和反射机制--**

传统上，类主动创建依赖对象，从而导致类与类之间高耦合,局部变更会影响到全部，ioc相当于在他们耦合关系之间加了一层，现在IOC利用反射机制把创建对象的控制权交给Spring IOC容器来创建（时机）、管理（对象的生命周期）、装配（通过依赖注入，配置对象）。IOC让对象的创建不用去new了，可以由spring自动生产，在运行时动态的去创建对象以及管理对象，并调用对象的方法。

> IoC 是设计思想，DI 是具体的实现方式；

Spring的IoC容器在实现控制反转和依赖注入的过程中,可以划分为两个阶段:

容器启动阶段    Bean实例化阶段

这两个阶段中,IoC容器分别作了以下这些事情:

![img](https://t10.baidu.com/it/u=217002030,1143584708&fm=173&app=25&f=JPEG?w=503&h=289&s=5E283463318E49494E5DE0DE000080B1)



##传统：new 一个对象的过程？

> 1. 当虚拟机遇到一条new指令时候，首先去检查这个指令的参数是否能 **在常量池中能否定位到一个类的符号引用** （即类的带路径全名），并且检查这个符号引用代表的类是否已被加载、解析和初始化过，即**验证是否是第一次使用该类**。如果是第一次使用，那必须先执行相应的类加载过程（class.forname()）。
> 2. 在类加载检查通过后，接下来虚拟机将 **为新生的对象分配内存** 。对象所需的内存的大小在类加载完成后便可以确定，为对象分配空间的任务等同于把一块确定大小的内存从Java堆中划分出来，目前常用的有两种方式，根据使用的垃圾收集器的不同使用不同的分配机制：2.1. **指针碰撞**（Bump the Pointer）：假设Java堆的内存是绝对规整的，所有用过的内存都放一边，空闲的内存放在另一边，中间放着一个指针作为分界点的指示器，那所分配内存就仅仅把那个指针向空闲空间那边挪动一段与对象大小相等的距离。2.2. **空闲列表**（Free List）：如果Java堆中的内存并不是规整的，已使用的内存和空闲的内存是相互交错的，虚拟机必须维护一个空闲列表，记录上哪些内存块是可用的，在分配时候从列表中找到一块足够大的空间划分给对象使用。
> 3. 内存分配完后，虚拟机需要将分配到的内存空间中的数据类型都 **初始化为零值（不包括对象头）**；
> 4. 虚拟机要 **对对象头进行必要的设置** ，例如这个对象是哪个类的实例（即所属类）、如何才能找到类的元数据信息、对象的哈希码、对象的GC分代年龄等信息，这些信息都存放在对象的对象头中。至此，从虚拟机视角来看，一个新的对象已经产生了。但是在Java程序视角来看，执行new操作后会接着执行如下步骤：
> 5. **调用对象的init()方法** ,根据传入的属性值给对象属性赋值。
> 6. 在线程 **栈中新建对象引用** ，并指向堆中刚刚新建的对象实例。

##依赖注入的三种方式

接口注入。不提倡。因为它强制被注入对象实现不必要的接口，带有侵入性。

构造方法注入。这种注入方式的优点就是，**对象在构造完成之后，即已进入就绪状态**，可以马上使用。缺点就是，当依赖对象比较多的时候，构造方法的参数列表会比较长。而通过反射构造对象的时候，对相同0类型的参数的处理会比较困难，维护和使用上也比较麻烦。**而且在Java中，构造方法无法被继承**。对于非必须的依赖处理，可能需要引入多个构造方法，而参数数量的变动可能造成维护上的不便。

setter方法注入。setter方法可以被继承，允许设置默认值。缺点当然就是对象无法在构造完成后马上进入就绪状态。如果通过set方法注入属性，那么spring会通过默认的空参构造方法来实例化对象，所以如果在类中写了一个带有参数的构造方法，一定要把空参数的构造方法写上，否则spring没有办法实例化对象，导致报错。

##如何将一个对象交给Spring管理

1.实现FactoryBean接口，重写getObject方法

2.使用@Bean注解，在@Bean注解标注的方法直接返回对象

3.使用xml配置的factory-method，原理和@Bean差不多

* FactoryBean

  是通过调用getObject方法获得对象，然后将这个对象放进factoryBeanInstanceCache中保存。

  具体流程：

  1.FactoryBean如果需要一开始就加载的话我们就需要去实现``SmartFactoryBean``接口然后重写它的isEagerInit方法，表示说我的对象一开始就交给Spring管理。

  2.然后会调用getBean方法，先加个&获取FatoryBean本身然后放进单例池中保存。

  3.然后会去做一些判断，如果符合条件的话就进行Bean初始化，调用getBean方法，这次就不加&了。

  （之后会走doGetBean，然后获取单例池里面的FactoryBean作为instance参数走到getObjectForBeanInstance方法里面。在这个方法里面会去通过name有没有加&去判断要获取的Bean到底是哪个，如果是要获取FactoryBean的话就直接返回instance，如果要获取Bean的话就去看看cache里有没有，没有的话就调用getObject方法获取，然后返回）

  4.这次就会调用getObject方法获取Bean，然后把这个Bean放在cache里面保存。

* factory-method 和 @Bean 是工厂方法将对象交给Spring管理的两种不同的方式（原理相同）

  都会将需要交给Spring管理的Bean信息注册，factory-method在读取xml阶段就能直接注册进beandefinitionMap里面，而@Bean是通过一个BeanFactoryPostProcessor，beanfactory后置处理器去解析@Bean标注的方法然后注册进Map里面，然后在实例化对象的时候使用指定的factory-method实例化Bean，然后将这个Bean放入单例池中保存。

##Spring Bean的生命周期

Spring上下文中的Bean生命周期，

1. 实例化 Instantiation
2. 属性赋值 Populate
3. 初始化 Initialization
4. 销毁 Destruction

具体过程：

首先是在对象实例化之前调用``InstantiationAwareBeanPostProcessor``接口的postProcessBeforeInstantiation方法，如果在这个方法里返回不为空的话能够直接return，会断掉接下来的Bean实例的默认创建。也就是在这里直接返回一个代理对象。

（1）实例化Bean：

对于BeanFactory容器，当客户向容器请求一个尚未初始化的bean时，或初始化bean的时候需要注入另一个尚未初始化的依赖时，容器就会调用createBean进行实例化。对于ApplicationContext容器，当容器启动结束后，通过获取BeanDefinition对象中的信息，实例化所有的bean。

（2）设置Bean属性 Spring根据BeanDefinition中的信息 以及 通过BeanWrapper提供的设置属性的接口完成依赖注入。

（3）处理Aware接口：**为了能够感知到自身的一些属性**

如果我们通过各种Aware接口声明了依赖关系，则会出注入Bean对容器基础设施层面的依赖。比如BeanNameAware、BeanFactoryAware和ApplicationContextAware,分别会注入Bean ID、Bean Factory或者ApplicationContext。

（4）调用BeanPostProcessor的前置初始化方法postProcessBeforeInitialization （BeanPostProcessor针对Spring上下文中所有的bean）

（5）如果实现了InitializingBean接口、则会调用afterPropertiesSet方法（初始化bean的时候执行，可以针对某个具体的bean进行配置）

（6）调用完Bean自身定义的init方法后

（7）接下来，调用BeanPostProcessor的后置初始化方法postProcessAfterInitialization

创建过程完毕

然后，销毁Bean，调用destroy方法，如果有实现``DiposiableBean``接口的话，调用里面的destroy方法。

# AOP

**--代理模式：Spring的AOP功能用到了JDK的动态代理和CGLIB字节码生成技术--**

AOP，面向切面思想，是面向对象的一种补充，用于将那些与业务无关，但却对多个对象产生影响的公共行为和逻辑的代码，抽取并封装为一个可重用的模块，这个模块被命名为“切面”（Aspect），减少系统中的重复代码，降低了模块间的耦合度，同时提高了系统的可维护性或者做一些增强处理。可用于权限认证、日志、事务处理。

AOP实现的关键在于 代理模式，AOP代理主要分为静态代理和动态代理。静态代理的代表为AspectJ；动态代理则以Spring AOP为代表。

（1）AspectJ是静态代理的增强，所谓静态代理，就是AOP框架会在编译阶段生成AOP代理类，因此也称为编译时增强，他会在编译阶段将AspectJ(切面)织入到Java字节码中，运行的时候就是增强之后的AOP对象。

（2）Spring AOP使用的动态代理，所谓的动态代理就是说AOP框架不会去修改字节码，而是每次运行时在内存中临时为方法生成一个AOP对象，这个AOP对象包含了目标对象的全部方法，并且在特定的切点做了增强处理，并回调原对象的方法。

##jdk动态代理和cglib动态代理

>①JDK动态代理只提供接口的代理，不支持类的代理。核心InvocationHandler接口和Proxy类，需要先实现一个``InvocationHandler``去定义我们方法的增强策略，然后拿这个handler去Proxy newProxyInstance去动态生成代理类，最后创建一个代理对象出来，这个代理对象会对方法进行增强之后回调invoke方法反射调用真实对象的方法。动态生成的代理类。                                          如果没有提供接口的话就无法使用jdk动态代理。
>
>* 注意：生成的代理类的成员变量是Method，调用方法的时候会拿着这个Method去到``InvocationHandler``调用invoke方法，这个handler就是我们定义的handler。

> ②如果代理类没有实现 InvocationHandler 接口，那么Spring AOP会选择使用CGLIB来动态代理目标类。CGLIB（Code Generation Library），是一个代码生成的类库，可以在运行时动态的生成指定类的一个子类对象，并覆盖其中特定方法并添加增强代码，从而实现AOP。CGLIB是通过继承的方式做的动态代理，因此如果某个类被标记为final，那么它是无法使用CGLIB做动态代理的。

#BeanFactory 和 FactoryBean的区别

BeanFactory是一个容器是一个顶层接口，为Spring管理Bean提供了一套通用规范，BeanFactory用来存储Bean的注册信息，用来实例化Bean和缓存Bean。

FactoryBean是一个Bean，归BeanFactory管理，是Spring提供的一个扩展点，适用于复杂的Bean的创建，FactoryBean能通过实现getObject方法来生产其他Bean实例。

FactoryBean接口，实现该接口可以通过getObject方法创建自己想要的bean对象。BeanFactory调用getBean获取FactoryBean时，有beanName处理：name加了&为获取FactoryBean本身，如果没加&就是获取getObject方法返回的对象。（？）



#bean的作用域。

Spring容器中的bean可以分为5个范围：

（1）singleton：默认，每个 IOC 容器创建中只有一个bean的实例，单例的模式由BeanFactory自身来维护。（适用于无状态的bean场景，减少频繁创建）

（2）prototype：为每一个bean请求提供一个实例。（适用于有状态的bean场景）

（3）request：每一次HTTP请求都会产生一个新的bean，该bean仅在当前HTTP request内有效。

（4）session：每一次HTTP请求都会产生一个新的 bean，该bean仅在当前 HTTP session 内有效。

（5）global-session：全局作用域，global-session和Portlet应用相关。表示所有的portlet共用全局的存储变量。

#Spring事务的实现方式和实现原理

Spring事务的本质其实就是数据库对事务的支持，没有数据库的事务支持，spring是无法提供事务功能的。真正的数据库层的事务提交和回滚是通过binlog或者redo log实现的。

##（1）Spring编程式和声明式事务：

spring支持编程式事务管理和声明式事务管理两种方式：

①编程式事务管理使用**TransactionTemplate。**

②声明式事务管理建立在AOP之上的。其本质是通过AOP功能，对方法前后进行拦截，将事务处理的功能编织到拦截的方法中，也就是在目标方法开始之前加入一个事务，在执行完目标方法之后根据执行情况提交或者回滚事务。

声明式事务最大的优点就是不需要在业务逻辑代码中掺杂事务管理的代码，只需在**配置文件**中做相关的事务规则声明或通过**@Transactional注解**的方式，便可以将事务规则应用到业务逻辑中。

声明式事务管理要优于编程式事务管理，这正是spring倡导的**非侵入式**的开发方式，使业务代码不受污染，只要加上注解就可以获得完全的事务支持。**缺点是，最细粒度只能作用到方法级别，无法做到像编程式事务那样可以作用到代码块级别。**

##### 事务回滚

默认情况下，事务只有在遇到RuntimeException和Error时才会回滚。但遇到Checked异常不会回滚。

##### @Transaction失效场景

1.数据库不支持事务

2.方法不是public

3.自身调用

4.异常被catch住

5.回滚的异常类型设置错误

##（2）spring的事务传播行为：

spring事务的传播行为说的是，当多个事务同时存在的时候，spring如何处理这些事务的行为。

① **PROPAGATION_REQUIRED**：如果当前存在事务，就加入该事务，如果当前没有事务，就创建一个新事务。该设置是最常用的设置。	

② PROPAGATION_SUPPORTS：支持当前事务(外部事务），如果当前存在事务，就加入该事务，如果当前不存在事务，就以非事务执行。‘

③ PROPAGATION_MANDATORY：支持当前事务，如果当前存在事务，就加入该事务，如果当前不存在事务，就抛出异常。

④ **PROPAGATION_REQUIRES_NEW**：无论当前存不存在事务，都创建新事务。

⑤ PROPAGATION_NOT_SUPPORTED：以非事务方式执行操作，如果当前存在事务，就把当前事务挂起。

⑥ PROPAGATION_NEVER：以非事务方式执行，如果当前存在事务，则抛出异常。

⑦ **PROPAGATION_NESTED**：如果当前存在事务，则在嵌套事务内执行。如果当前没有事务，则按REQUIRED属性执行。

![image-20210206211415043](C:\Users\11468\AppData\Roaming\Typora\typora-user-images\image-20210206211415043.png)

##（3）Spring中的事务隔离级别：

- 脏读：表示一个事务能够读到另一个事务中还未提交的数据。比如，某个事务尝试插入记录A，此时该事务还未提交，然后另一个事务尝试读取到了记录A
- 不可重复读：是指一个事务内，多次读同一个数据，但结果值不一样。
- 幻读：指同一个事务内多次查询返回的结果集不一样。比如同一个事务A第一次查询发现有n条记录，但是第二次同等条件下查询却有n+1条记录，这就是发生了幻读，这是因为被另一个事务增加删除修改了，所以查询的结果变了。

① ISOLATION_DEFAULT：这是个 PlatfromTransactionManager 默认的隔离级别，使用数据库默认的事务隔离级别。

② ISOLATION_READ_UNCOMMITTED：读未提交，允许事务读取未被其他事务提交的变更数据。会出现脏读、不可重复读和幻读问题。

③ ISOLATION_READ_COMMITTED：读已提交，只允许事务读取已经被其他事务提交的变更数据。可避免脏读，仍会出现不可重复读和幻读问题。

④ ISOLATION_REPEATABLE_READ：可重复读，一个事务执行过程中看到的数据，总是跟这个事务在启动时看到的数据是一致的。也是不能看到该事务对已有记录的更新。可以避免脏读和不可重复读，仍会出现幻读问题。

⑤ ISOLATION_SERIALIZABLE：一个事务在执行的过程中完全看不到其他事务对数据库所做的更新。

#Spring的循环依赖

三级缓存解决循环依赖

```java
private final Map<String, Object> singletonObjects = new ConcurrentHashMap<>(256); //一级缓存 单例对象 
private final Map<String, Object> earlySingletonObjects = new HashMap<>(16); // 二级缓存 早期单例对象 
private final Map<String, ObjectFactory<?>> singletonFactories = new HashMap<>(16); // 三级缓存 单例工厂
//在创建过程中，都是从三级缓存(对象工程创建不完整对象)，将提前暴露的对象放入到二级缓存，从二级缓存拿到后，完成初始化，放入一级缓存
```

1. 使用context.getBean(A.class)，旨在获取容器内的单例A(若A不存在，就会走A这个Bean的创建流程)
2. 实例化A（注意此处仅仅是实例化），并将它放进缓存（此时A已经实例化完成，已经可以被引用了）先通过createBeanInstance实例化A对象，又将该实例化的对象通过addSingletonFactory方法放入**singletonFactories**中，完成A对象早期的暴露
3. 初始化A：@Autowired依赖注入B（此时需要去容器内获取B）
4. 为了完成依赖注入B，会通过getBean(B)去容器内找B。但此时B在容器内不存在，就走向**B的创建之路~**
5. 实例化B，并将其放入缓存。（此时B也能够被引用了）
6. 初始化B，@Autowired依赖注入A（此时需要去容器内获取A）
7. 此处重要：初始化B时会调用getBean(A)去容器内找到A，上面我们已经说过了此时候因为A已经实例化完成了并且放进了缓存singletonFactories中里：

所以这个时候从一级缓存

singletonObjects和二级缓存

earlySingletonObjects中未发现A，但是在三级缓存

singletonFactories中**发现A，将A移动到二级缓存**

earlySingletonObjects，同时从三级缓存删除；所以此时缓存里是已经存在A的引用了的，所以getBean(A)能够正常返回

1. **B初始化成功**（此时已经注入A成功了，已成功持有A的引用了），return（注意此处return相当于是返回最上面的getBean(B)这句代码，回到了初始化A的流程中~）。
2. 因为B实例已经成功返回了，因此最终**A也初始化成功**
3. **到此，B持有的已经是初始化完成的A，A持有的也是初始化完成的B，完美~**，最后把初始化好的bean放在 单例池。
4. 总结：Spring通过将实例化后的对象提前暴露给Spring容器中的singletonFactories，解决了循环依赖的问题

**三级缓存为什么是三级，去除第三级缓存行不行？**

第三级缓存的目的是为了延迟代理对象的创建，因为如果没有依赖循环的话，那么就不需要为其提前创建代理，可以将它延迟到初始化完成之后再创建。但是因为循环依赖的出现，导致了 Spring 不得不提前去创建代理，因为如果不提前创建代理对象，那么注入的就是原始对象，这样就会产生错误。

既然目的只是延迟的话，那么我们是不是可以不延迟创建，而是在实例化完成之后，就为其创建代理对象，这样我们就不需要第三级缓存了。

如果 Spring 选择二级缓存来解决循环依赖的话，那么就意味着所有 Bean 都需要在实例化完成之后就立马为其创建代理，而 Spring 的设计原则是在 Bean 初始化完成之后才为其创建代理（

Spring对于aop的设计，是使用AnnotationAwareAspectJAutoProxyCreator后置处理器让Bean在Bean的生命周期的最后一步再完成aop代理，而不是在Bean实例化之后就立马进行aop代理。

）。所以，Spring 选择了三级缓存。

# 注解

##@Autowried注解注入

1.在doCreateBean方法中，当bean实例化完成之后会去调用``MergedBeanDefinitionPostProcessor``接口的postProcessMergedBeanDefinition方法。``AutowAutoiredAnnotationBeanPostProcessor``后置处理类实现了这个接口。

2.在postProcessMergedBeanDefinition方法中，会去找出所有的注入点，也就是被@Autowried注解修饰的方法和字段，之后再排除不符合的注入点，在这一步完成了注解的解析。

3.注入步骤是在populateBean方法进行注入，使用的是``InstantiationAwareBeanPostProcessor``接口的postProcessProperties方法。

##Autowried 与 Resource

Autowried和Resource注解实现注入的时候，本质上都是依靠``MergedBeanDefinitionPostProcessor``接口的postProcessMergedBeanDefinition方法去找切入点，通过``InstantiationAwareBeanPostProcessor``接口的postProcessProperties方法去将属性注入进去。基本用法差不多。

区别：1.后置处理器不一样

2.属性：

@Autowired按类型装配byType依赖对象，默认情况下它要求依赖对象必须存在，如果允许null值，可以设置它required属性为false。如果我们想使用按名称装配，可以结合@Qualifier注解一起使用。

@Resource优先按ByName然后再按类型，有两个中重要的属性：name和type。name属性指定byName，如果没有指定name属性，当注解标注在字段上，即默认取字段的名称作为bean名称寻找依赖对象，当注解标注在属性的setter方法上，即默认取属性名作为bean名称寻找依赖对象。需要注意的是，@Resource如果没有指定name属性，并且按照默认的名称仍然找不到依赖对象时， @Resource注解会回退到按类型装配。但一旦指定了name属性，就只能按名称装配了。 

3.@Autowired spring的 @Resource jdk的 