#Spring

**扫盲**

https://javadoop.com/post/spring-ioc

## IOC

由于获取资源（数据库、读取文件、对象），类内部主动创建依赖对象，从而导致类与类之间高耦合,局部变更会影响到全部，ioc相当于在他们耦合关系之间加了一层，现在IOC利用反射机制把创建对象的控制权交给Spring IOC容器来创建（时机）、管理（对象的生命周期）、装配（通过依赖注入，配置对象）。spring通过依赖注入实现ioc。

> IoC 是设计思想，DI 是具体的实现方式；

**当然，IoC 也可以通过其他的方式来实现，而 DI 只是 Spring 的选择。**

### IOC优点

* 降低了耦合
* IOC容器支持加载服务的饿汉式初始化和懒加载
* 不用关注对象的创建具体实现，我们只需要预先配置好，需要的时候使用即可，没有IOC的话我们需要底层查看并按要求用构造参数创建

本质上用的是**工厂模式和反射机制**

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

###依赖注入

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

### Springbean 作用域/对象的生命周期

（1）singleton：默认，每个 IOC 容器创建中只有一个bean的实例，单例的模式由BeanFactory自身来维护。（适用于无状态的bean场景，减少频繁创建）

（2）prototype：为每一个bean请求提供一个实例。（适用于有状态的bean场景）

（3）request：每一次HTTP请求都会产生一个新的bean，该bean仅在当前HTTP request内有效。

（4）session：每一次HTTP请求都会产生一个新的 bean，该bean仅在当前 HTTP session 内有效。

（5）global-session：全局作用域，global-session和Portlet应用相关。表示所有的portlet共用全局的存储变量。

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
> 对于“prototype”作用域Bean，Spring容器无法完成依赖注入，**因为“prototype”作用域的Bean，Spring容器不进行缓存，因此无法提前暴露一个创建中的Bean。**
>
> @Lazy无法解决，当你真正使用到它（初始化）的时候，依旧会报异常UnsatisfiedDependencyException: Error creating bean with name 



###Spring创建Bean的流程

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

![image-20201216155819530](.\images\image-20201216155819530.png)

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



```java
protected Object createBean(String beanName, RootBeanDefinition mbd, @Nullable Object[] args) throws BeanCreationException {
    if (this.logger.isTraceEnabled()) {
        this.logger.trace("Creating instance of bean '" + beanName + "'");
    }

    RootBeanDefinition mbdToUse = mbd;
    Class<?> resolvedClass = this.resolveBeanClass(mbd, beanName, new Class[0]);
    if (resolvedClass != null && !mbd.hasBeanClass() && mbd.getBeanClassName() != null) {
        mbdToUse = new RootBeanDefinition(mbd);
        mbdToUse.setBeanClass(resolvedClass);
    }

    try {
        mbdToUse.prepareMethodOverrides();
    } catch (BeanDefinitionValidationException var9) {
        throw new BeanDefinitionStoreException(mbdToUse.getResourceDescription(), beanName, "Validation of method overrides failed", var9);
    }

    Object beanInstance;
    try {
        beanInstance = this.resolveBeforeInstantiation(beanName, mbdToUse);
        if (beanInstance != null) {
            return beanInstance;
        }
    } catch (Throwable var10) {
        throw new BeanCreationException(mbdToUse.getResourceDescription(), beanName, "BeanPostProcessor before instantiation of bean failed", var10);
    }

    try {
        beanInstance = this.doCreateBean(beanName, mbdToUse, args);
        if (this.logger.isTraceEnabled()) {
            this.logger.trace("Finished creating instance of bean '" + beanName + "'");
        }

        return beanInstance;
    } catch (ImplicitlyAppearedSingletonException | BeanCreationException var7) {
        throw var7;
    } catch (Throwable var8) {
        throw new BeanCreationException(mbdToUse.getResourceDescription(), beanName, "Unexpected exception during bean creation", var8);
    }
}
```

```
@Nullable
protected Object resolveBeforeInstantiation(String beanName, RootBeanDefinition mbd) {
    Object bean = null;
    if (!Boolean.FALSE.equals(mbd.beforeInstantiationResolved)) {
        if (!mbd.isSynthetic() && this.hasInstantiationAwareBeanPostProcessors()) {
            Class<?> targetType = this.determineTargetType(beanName, mbd);
            if (targetType != null) {
                bean = this.applyBeanPostProcessorsBeforeInstantiation(targetType, beanName);
                if (bean != null) {
                    bean = this.applyBeanPostProcessorsAfterInitialization(bean, beanName);
                }
            }
        }

        mbd.beforeInstantiationResolved = bean != null;
    }

    return bean;
}
```

```java
protected void populateBean(String beanName, RootBeanDefinition mbd, @Nullable BeanWrapper bw) {
    if (bw == null) {
        if (mbd.hasPropertyValues()) {
            throw new BeanCreationException(mbd.getResourceDescription(), beanName, "Cannot apply property values to null instance");
        }
    } else {
        if (!mbd.isSynthetic() && this.hasInstantiationAwareBeanPostProcessors()) {
            Iterator var4 = this.getBeanPostProcessors().iterator();

            while(var4.hasNext()) {
                BeanPostProcessor bp = (BeanPostProcessor)var4.next();
                if (bp instanceof InstantiationAwareBeanPostProcessor) {
                    InstantiationAwareBeanPostProcessor ibp = (InstantiationAwareBeanPostProcessor)bp;
                    if (!ibp.postProcessAfterInstantiation(bw.getWrappedInstance(), beanName)) {
                        return;
                    }
                }
            }
        }

        PropertyValues pvs = mbd.hasPropertyValues() ? mbd.getPropertyValues() : null;
        int resolvedAutowireMode = mbd.getResolvedAutowireMode();
        if (resolvedAutowireMode == 1 || resolvedAutowireMode == 2) {
            MutablePropertyValues newPvs = new MutablePropertyValues((PropertyValues)pvs);
            if (resolvedAutowireMode == 1) {
                this.autowireByName(beanName, mbd, bw, newPvs);
            }

            if (resolvedAutowireMode == 2) {
                this.autowireByType(beanName, mbd, bw, newPvs);
            }

            pvs = newPvs;
        }

        boolean hasInstAwareBpps = this.hasInstantiationAwareBeanPostProcessors();
        boolean needsDepCheck = mbd.getDependencyCheck() != 0;
        PropertyDescriptor[] filteredPds = null;
        if (hasInstAwareBpps) {
            if (pvs == null) {
                pvs = mbd.getPropertyValues();
            }

            Iterator var9 = this.getBeanPostProcessors().iterator();

            while(var9.hasNext()) {
                BeanPostProcessor bp = (BeanPostProcessor)var9.next();
                if (bp instanceof InstantiationAwareBeanPostProcessor) {
                    InstantiationAwareBeanPostProcessor ibp = (InstantiationAwareBeanPostProcessor)bp;
                    PropertyValues pvsToUse = ibp.postProcessProperties((PropertyValues)pvs, bw.getWrappedInstance(), beanName);
                    if (pvsToUse == null) {
                        if (filteredPds == null) {
                            filteredPds = this.filterPropertyDescriptorsForDependencyCheck(bw, mbd.allowCaching);
                        }

                        pvsToUse = ibp.postProcessPropertyValues((PropertyValues)pvs, filteredPds, bw.getWrappedInstance(), beanName);
                        if (pvsToUse == null) {
                            return;
                        }
                    }

                    pvs = pvsToUse;
                }
            }
        }

        if (needsDepCheck) {
            if (filteredPds == null) {
                filteredPds = this.filterPropertyDescriptorsForDependencyCheck(bw, mbd.allowCaching);
            }

            this.checkDependencies(beanName, mbd, filteredPds, (PropertyValues)pvs);
        }

        if (pvs != null) {
            this.applyPropertyValues(beanName, mbd, bw, (PropertyValues)pvs);
        }

    }
}
```



###ApplicationContext和BeanFactory的区别？

BeanFactory和AppliationContext是Spring的两大核心接口，都可以当作Spring的容器。其中ApplicationContext是BeanFactory的子接口

（1）依赖关系

BeanFactory：是Spring里面最底层的接口，包含了各种bean的定义，读取bean配置文档，管理bean的加载、实例化、控制bean的生命周期，维护bean之间的依赖关系。

ApplicationContext接口作为BeanFactory的派生，除了提供beanFactory所具有的功能外，还提供了更完整的框架功能：

* 继承MessageSource，因此支持国际化。
* 统一的资源文件访问方式
* 提供在监听器中注册bean的事件
* 同时加载多个配置文件
* 载入多个（有继承关系的）上下文，使得每一个上下文都专注于一个特定的层次，比如应用的Web层。

（2）加载方式

BeanFactory采用的是延迟加载形式来注入Bean的，即只有在使用到某个Bean时（调用getBean（））,才对该bean进行加载实例化。所以，我们就不能发现一些只有加载后才会出现的异常。

ApplicationContext，它是容器启动时一次性创建了所有Bean。这样，在容器启动时，我们可以及时发现配置错误问题。ApplicationContext启动后预载入所有的单实例Bean，通过预载入所有的单实例Bean，这样需要用的时候，不用等待，因为已经创建好了，缺点是启动慢。

（3）创建方式

BeanFactory通常以编程的方式被创建，ApplicationContext还能以声明的方式创建，如使用ContextLoader。



（4）注册方式

BeanFactory和ApplicationContext都支持BeanPostProcess、BeanFactoryPostProcess的使用，但两者的区别时：前者需要手动注册bean，比如registerSingleton（）方法，而ApplicationContext则是自动注册bean。

```java
@Test
public void test(){
    ClassPathXmlApplicationContext appContext =
            new ClassPathXmlApplicationContext("classpath:app-context.xml");
    Object object = new Object();
    appContext.getBeanFactory().registerSingleton("object",object);
    System.out.println(object == appContext.getBean("object"));
}
```



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

###静态代理

好处：

* 可以使真实角色的操作更加纯粹！不用去关注一些公共的业务
* 公共也就就交给代理角色！实现业务的分工！
* 公共业务发生扩展的时候，方便集中管理

缺点

* 一个真实角色就会产生一个代理角色；代码量会翻倍~开发效率会变低

> AspectJ是静态代理的增强，所谓静态代理，就是AOP框架会在编译阶段生成AOP代理类，因此也称为编译时增强，他会在编译阶段将AspectJ(切面)织入到Java字节码中，运行的时候就是增强之后的AOP对象。

###动态代理

好处：

* 一个动态代理类 代理的是一个接口
* 一个动态代理类 可以代理多个类 只要实现了同一个接口即可

####Spring的jdk动态代理和cglib动态代理

>①JDK动态代理只提供接口的代理，不支持类的代理。核心InvocationHandler接口和Proxy类，InvocationHandler 通过invoke方法反射来调用目标类中的代码，动态地将横切逻辑和业务编织在一起；接着，Proxy利用 InvocationHandler动态创建一个符合某一接口的的实例, 生成目标类的代理对象。

> ②如果代理类没有实现 InvocationHandler 接口，那么Spring AOP会选择使用CGLIB来动态代理目标类。CGLIB（Code Generation Library），是一个代码生成的类库，可以在运行时动态的生成指定类的一个子类对象，并覆盖其中特定方法并添加增强代码，从而实现AOP。CGLIB是通过继承的方式做的动态代理，因此如果某个类被标记为final，那么它是无法使用CGLIB做动态代理的。

# 常用注解

https://segmentfault.com/a/1190000021434929

## 1）用于注册bean对象注解

### 1.1 `@Component`

**作用：**

```
调用无参构造创建一个bean对象，并把对象存入spring的IOC容器，交由spring容器进行管理。相当于在xml中配置一个bean。
```

**属性：**

```
value：指定bean的id。如果不指定value属性，默认bean的id是当前类的类名。首字母小写。
```

### 1.2 `@Controller`

**作用：**

```
作用上与@Component。一般用于表现层的注解。
```

**属性：**

```
value：指定bean的id。如果不指定value属性，默认bean的id是当前类的类名。首字母小写。
```

### 1.3 `@Service`

**作用：**

```
作用上与@Component。一般用于业务层的注解。
```

**属性：**

```
value：指定bean的id。如果不指定value属性，默认bean的id是当前类的类名。首字母小写。
```

### 1.4 `@Repository`

**作用：**

```
作用上与@Component。一般用于持久层的注解。
```

**属性：**

```
value：指定bean的id。如果不指定value属性，默认bean的id是当前类的类名。首字母小写。
```

### 1.5 `@Bean`

**作用：**

```
用于把当前方法的返回值作为bean对象存入spring的ioc容器中
```

**属性：**

```
name：用于指定bean的id。当不写时，默认值是当前方法的名称。注意：当我们使用注解配置方法时，如果方法有参数，spring框架会去容器中查找有没有可用的bean对象，查找的方式和Autowired注解的作用是一样的。
```

#### **Bean的作用范围的注解:**

@Scope:

*** singleton:单例**

*** prototype:多例**

#### Bean的生命周期的配置:

@PostConstruct :相当于init-method

@PreDestroy :相当于destroy-method

  @PostConstruct说明

​     被@PostConstruct修饰的方法会在服务器加载Servlet的时候运行，并且只会被服务器调用一次，类似于Serclet的inti()方法。被@PostConstruct修饰的方法会在构造函数之后，init()方法之前运行。

  @PreConstruct说明

​     被@PreConstruct修饰的方法会在服务器卸载Servlet的时候运行，并且只会被服务器调用一次，类似于Servlet的destroy()方法。被@PreConstruct修饰的方法会在destroy()方法之后运行，在Servlet被彻底卸载之前。

**案例：**

```
/**
 * 获取DataSource对象
 * @return
 */
@Bean(value = "dataSource")
public DataSource getDataSource() {
    try {
        ComboPooledDataSource dataSource = new ComboPooledDataSource();
        dataSource.setDriverClass(this.driver);
        dataSource.setJdbcUrl(this.url);
        dataSource.setUser(this.username);
        dataSource.setPassword(this.password);
        return dataSource;
    }catch (Exception exception) {
        throw new RuntimeException(exception);
    }
}
```

## 2）用于依赖注入的注解

###  `@Autowired`

**作用：**

```java
@Autowire和@Resource都是Spring支持的注解形式动态装配bean的方式。Autowire默认[按照类型装配](byType)，如果想要[按照名称](byName)装配，需结合@Qualifier注解使用。
```

**属性：**

```
required：@Autowire注解默认情况下要求依赖对象必须存在。如果不存在，则在注入的时候会抛出异常。如果允许依赖对象为null，需设置required属性为false。
```

**案例：**

```
@Autowire 
@Qualifier("userService") 
private UserService userService;
```

###  `@Qualifier`

**作用：**

```
在自动按照类型注入的基础之上，再按照 Bean 的 id 注入。它在给字段注入时不能独立使用，必须和 @Autowire一起使用；但是给方法参数注入时，可以独立使用。
```

**属性：**

```
value：用于指定要注入的bean的id，其中，该属性可以省略不写。
```

**案例：**

```
@Autowire
@Qualifier(value="userService") 
//@Qualifier("userService")     //value属性可以省略不写
private UserService userService;

//可能存在多个UserDao实例
@Autowired 
@Qualifier("userServiceImpl") 
public IUserService userService; 
或者

@Autowired 
public void setUserDao(@Qualifier("userDao") UserDao userDao) { 
	this.userDao = userDao; 
//可能不存在UserDao实例
@Autowired(required = false) 
public IUserService userService

```

###  `@Resource`

**作用：**

```
@Autowire和@Resource都是Spring支持的注解形式动态装配bean的方式。@Resource默认按照名称(byName)装配，名称可以通过name属性指定。如果没有指定name，则注解在字段上时，默认取（name=字段名称）装配。如果注解在setter方法上时，默认取（name=属性名称）装配。
```

**属性：**

```
name：用于指定要注入的bean的id
type：用于指定要注入的bean的type
```

**装配顺序**

```
1.如果同时指定name和type属性，则找到唯一匹配的bean装配，未找到则抛异常；
2.如果指定name属性，则按照名称(byName)装配，未找到则抛异常；
3.如果指定type属性，则按照类型(byType)装配，未找到或者找到多个则抛异常；
4.既未指定name属性，又未指定type属性，则按照名称(byName)装配；如果未找到，则按照类型(byType)装配。
```

**案例：**

```
@Resource(name="userService")
//@Resource(type="userService")
//@Resource(name="userService", type="UserService")
private UserService userService;
```

###  `@Value`

**作用：**

```
通过@Value可以将外部的值动态注入到Bean中，可以为基本类型数据和String类型数据的变量注入数据
```

**案例：**

```
// 1.基本类型数据和String类型数据的变量注入数据
@Value("tom") 
private String name;
@Value("18") 
private Integer age;


// 2.从properties配置文件中获取数据并设置到成员变量中
// 2.1jdbcConfig.properties配置文件定义如下
jdbc.driver \= com.mysql.jdbc.Driver  
jdbc.url \= jdbc:mysql://localhost:3306/eesy  
jdbc.username \= root  
jdbc.password \= root

// 2.2获取数据如下
@Value("${jdbc.driver}")  
private String driver;

@Value("${jdbc.url}")  
private String url;  
  
@Value("${jdbc.username}")  
private String username;  
  
@Value("${jdbc.password}")  
private String password;
```

## 3）用于改变bean作用范围的注解

###  `@Scope`

**作用：**

```
指定bean的作用范围。
```

**属性：**

```
value：
    1）singleton：单例
    2）prototype：多例
    3）request： 
    4）session： 
    5）globalsession：
```

**案例：**

```
@Autowire
@Scope(value="prototype")
private UserService userService;
```

## 4）生命周期相关的注解

### `@PostConstruct`

**作用：**

```
指定初始化方法
```

**案例：**

```
@PostConstruct  
public void init() {  
    System.out.println("初始化方法执行");  
}
```

### `@PreDestroy`

**作用：**

```
指定销毁方法
```

**案例：**

```
@PreDestroy  
public void destroy() {  
    System.out.println("销毁方法执行");  
}
```

##@Resource和@Autowired的区别

* 都是用来自动装配的，都可以放在属性字段上
* @Autowired通过byType的方式实现，而且必须要求这个对象存在！
* @Resource 默认通过byname的方式实现，如果找不到名字，则通过byType实现！如果两个都找不到的情况下就报错。
* 执行顺序不同

##@Component相当于bean；有几个衍生注解，

* dao【@Repository】
* service【@Service】
* controller【@Controller】

四个注解都是把类注册到spring容器中

##@transactional

spring支持编程式事务管理和声明式事务管理两种方式。

* 编程式事务管理使用TransactionTemplate或者直接使用底层的PlatformTransactionManager。对于编程式事务管理，spring推荐使用TransactionTemplate。

* 声明式事务管理建立在AOP之上的。其本质是对方法前后进行拦截，然后在目标方法开始之前创建或者加入一个事务，在执行完目标方法之后根据执行情况提交或者回滚事务。声明式事务最大的优点就是不需要通过编程的方式管理事务，这样就不需要在业务逻辑代码中掺杂事务管理的代码，只需在配置文件中做相关的事务规则声明(或通过基于@Transactional注解的方式)，便可以将事务规则应用到业务逻辑中。【一种是基于tx和aop名字空间的xml配置文件，另一种就是基于@Transactional注解。显然基于注解的方式更简单易用，更清爽。】

和编程式事务相比，声明式事务唯一不足地方是，它的最细粒度只能作用到方法级别，无法做到像编程式事务那样可以作用到代码块级别。但是即便有这样的需求，也存在很多变通的方法，比如，可以将需要进行事务管理的代码块独立为方法等等。

##@RequestMapping

- *@GetMapping*
- *@PostMapping*
- *@PutMapping*
- *@DeleteMapping*
- *@PatchMapping*

从命名约定我们可以看到每个注释都是为了处理各自的传入请求方法类型，即*@GetMapping*用于处理请求方法的*GET*类型，*@ PostMapping*用于处理请求方法的*POST*类型等。

如果我们想使用传统的*@RequestMapping*注释实现URL处理程序，那么它应该是这样的：

```java
@RequestMapping(value = "/get/{id}", method = RequestMethod.GET)
```

新方法可以简化为：

```java
@GetMapping("/get/{id}")
```

##@responseBody

@responseBody注解的作用是将controller的方法返回的对象通过适当的转换器转换为指定的格式之后，写入到response对象的body区，通常用来返回JSON数据或者是XML

　　数据，需要注意的呢，在使用此注解之后不会再走试图处理器，而是直接将数据写入到输入流中，他的效果等同于通过response对象输出指定格式的数据。

##@RestController

1. Controller, RestController的共同点

都是用来表示Spring某个类的是否可以接收baiHTTP请求

2.  Controller, RestController的不同点

@Controller标识一个Spring类是Spring MVC controller处理器

@RestController：  a convenience annotation that does nothing more than adding the@Controller and@ResponseBody annotations。  @RestController是@Controller和@ResponseBody的结合体，两个标注合并起来的作用。