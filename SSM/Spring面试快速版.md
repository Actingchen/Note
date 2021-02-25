#IOC

**--工厂模式和反射机制--**

类主动创建依赖对象，从而导致类与类之间高耦合,局部变更会影响到全部，ioc相当于在他们耦合关系之间加了一层，现在IOC利用反射机制把创建对象的控制权交给Spring IOC容器来创建（时机）、管理（对象的生命周期）、装配（通过依赖注入，配置对象）。IOC让对象的创建不用去new了，可以由spring自动生产，在运行时动态的去创建对象以及管理对象，并调用对象的方法。

> IoC 是设计思想，DI 是具体的实现方式；

# AOP

**--代理模式：Spring的AOP功能用到了JDK的动态代理和CGLIB字节码生成技术--**

AOP，称为面向切面，是面向对象的一种补充，用于将那些与业务无关，但却对多个对象产生影响的公共行为和逻辑的代码，抽取并封装为一个可重用的模块，这个模块被命名为“切面”（Aspect），减少系统中的重复代码，降低了模块间的耦合度，同时提高了系统的可维护性。可用于权限认证、日志、事务处理。

AOP实现的关键在于 代理模式，AOP代理主要分为静态代理和动态代理。静态代理的代表为AspectJ；动态代理则以Spring AOP为代表。

（1）AspectJ是静态代理的增强，所谓静态代理，就是AOP框架会在编译阶段生成AOP代理类，因此也称为编译时增强，他会在编译阶段将AspectJ(切面)织入到Java字节码中，运行的时候就是增强之后的AOP对象。

（2）Spring AOP使用的动态代理，所谓的动态代理就是说AOP框架不会去修改字节码，而是每次运行时在内存中临时为方法生成一个AOP对象，这个AOP对象包含了目标对象的全部方法，并且在特定的切点做了增强处理，并回调原对象的方法。

##jdk动态代理和cglib动态代理

>①JDK动态代理只提供接口的代理，不支持类的代理。核心InvocationHandler接口和Proxy类，InvocationHandler 通过invoke方法反射来调用目标类中的代码，动态地将横切逻辑和业务编织在一起；接着，Proxy利用 InvocationHandler动态创建一个符合某一接口的的实例, 生成目标类的代理对象。

> ②如果代理类没有实现 InvocationHandler 接口，那么Spring AOP会选择使用CGLIB来动态代理目标类。CGLIB（Code Generation Library），是一个代码生成的类库，可以在运行时动态的生成指定类的一个子类对象，并覆盖其中特定方法并添加增强代码，从而实现AOP。CGLIB是通过继承的方式做的动态代理，因此如果某个类被标记为final，那么它是无法使用CGLIB做动态代理的。

#Spring Bean的生命周期

首先说一下Servlet的生命周期：实例化，初始init，接收请求service，销毁destroy；

Spring上下文中的Bean生命周期也类似，

1. 实例化 Instantiation
2. 属性赋值 Populate
3. 初始化 Initialization
4. 销毁 Destruction

具体过程：

（1）实例化Bean：

对于BeanFactory容器，当客户向容器请求一个尚未初始化的bean时，或初始化bean的时候需要注入另一个尚未初始化的依赖时，容器就会调用createBean进行实例化。对于ApplicationContext容器，当容器启动结束后，通过获取BeanDefinition对象中的信息，实例化所有的bean。

（2）设置Bean属性 Spring根据BeanDefinition中的信息 以及 通过BeanWrapper提供的设置属性的接口完成依赖注入。

（3）处理Aware接口：

如果我们通过各种Aware接口声明了依赖关系，则会出注入Bean对容器基础设施层面的依赖。具体包括BeanNameAware、BeanFactoryAware和ApplicationContextAware,分别会注入Bean ID、Bean Factory或者ApplicationContext

（4）调用BeanPostProcessor的前置初始化方法postProcessBeforeInitialization

（5）如果实现了InitializingBean接口、则会调用afterPropertiesSet方法

（6）调用Bean自身定义的init方法

（7）调用BeanPostProcessor的后置初始化方法postProcessAfterInitialization

创建过程完毕

然后，Spring Bean 的销毁过程会依次调用（如果Bean实现了DisposableBean这个接口 ）DisposableBean 的 destroy 方法和 Bean 自身定制的 destroy 方法。

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

声明式事务管理要优于编程式事务管理，这正是spring倡导的**非侵入式**的开发方式，使业务代码不受污染，只要加上注解就可以获得完全的事务支持。**唯一不足地方是，最细粒度只能作用到方法级别，无法做到像编程式事务那样可以作用到代码块级别。**

##（2）spring的事务传播行为：

spring事务的传播行为说的是，当多个事务同时存在的时候，spring如何处理这些事务的行为。

① **PROPAGATION_REQUIRED**：如果当前存在事务，就加入该事务，如果当前没有事务，就创建一个新事务。该设置是最常用的设置。	

② PROPAGATION_SUPPORTS：支持当前事务，如果当前存在事务，就加入该事务，如果当前不存在事务，就以非事务执行。‘

③ PROPAGATION_MANDATORY：支持当前事务，如果当前存在事务，就加入该事务，如果当前不存在事务，就抛出异常。

④ **PROPAGATION_REQUIRES_NEW**：创建新事务，无论当前存不存在事务，都创建新事务。

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

1. 使用context.getBean(A.class)，旨在获取容器内的单例A(若A不存在，就会走A这个Bean的创建流程)，显然初次获取A是不存在的，因此走**A的创建之路~**
2. 实例化A（注意此处仅仅是实例化），并将它放进缓存（此时A已经实例化完成，已经可以被引用了）先通过createBeanInstance实例化A对象，又将该实例化的对象通过addSingletonFactory方法放入singletonFactories中，完成A对象早期的暴露
3. 初始化A：@Autowired依赖注入B（此时需要去容器内获取B）
4. 为了完成依赖注入B，会通过getBean(B)去容器内找B。但此时B在容器内不存在，就走向**B的创建之路~**
5. 实例化B，并将其放入缓存。（此时B也能够被引用了）
6. 初始化B，@Autowired依赖注入A（此时需要去容器内获取A）
7. 此处重要：初始化B时会调用getBean(A)去容器内找到A，上面我们已经说过了此时候因为A已经实例化完成了并且放进了缓存singletonFactories中里：

所以这个时候从一级缓存

singletonObjects和二级缓存

earlySingletonObjects中未发现A，但是在三级缓存

singletonFactories中发现A，将A移动到二级缓存

earlySingletonObjects，同时从三级缓存删除；所以此时缓存里是已经存在A的引用了的，所以getBean(A)能够正常返回

1. **B初始化成功**（此时已经注入A成功了，已成功持有A的引用了），return（注意此处return相当于是返回最上面的getBean(B)这句代码，回到了初始化A的流程中~）。
2. 因为B实例已经成功返回了，因此最终**A也初始化成功**
3. **到此，B持有的已经是初始化完成的A，A持有的也是初始化完成的B，完美~**
4. 总结：Spring通过将实例化后的对象提前暴露给Spring容器中的singletonFactories，解决了循环依赖的问题

**三级缓存为什么是三级，去除第三级缓存行不行？**

第三级缓存的目的是为了延迟代理对象的创建，因为如果没有依赖循环的话，那么就不需要为其提前创建代理，可以将它延迟到初始化完成之后再创建。但是因为循环依赖的出现，导致了 Spring 不得不提前去创建代理，因为如果不提前创建代理对象，那么注入的就是原始对象，这样就会产生错误。

既然目的只是延迟的话，那么我们是不是可以不延迟创建，而是在实例化完成之后，就为其创建代理对象，这样我们就不需要第三级缓存了。

如果 Spring 选择二级缓存来解决循环依赖的话，那么就意味着所有 Bean 都需要在实例化完成之后就立马为其创建代理，而 Spring 的设计原则是在 Bean 初始化完成之后才为其创建代理（

Spring对于aop的设计，是使用AnnotationAwareAspectJAutoProxyCreator后置处理器让Bean在Bean的生命周期的最后一步再完成aop代理，而不是在Bean实例化之后就立马进行aop代理。

）。所以，Spring 选择了三级缓存。