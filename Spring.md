> 参考来源：专注写bug、狂神、尚硅谷。

#Spring



## IOC：控制反转

由于获取资源（数据库、读取文件、对象），类内部主动创建依赖对象，从而导致类与类之间高耦合,局部变更可能影响全部，直到IOC/DI出现。

* **IOC**：以前创建对象是由程序主动创建对象、主动去控制获取依赖对象，类之间耦合度高，难于测试，现在把创建对象的控制权交给Spring IOC容器来创建（时机）、管理（对象的生命周期）、装配（通过依赖注入，配置对象）。
* **DI Dependency Injection** 依赖注入：容器能知道哪个组件（类）运行的时候，需要或者说是依赖另一个类（组件）；容器通过反射的形式，将准备好的BookService对象注入（利用反射给属性赋值）到BookServlet中。

简单过程描述：

1、创建Service类
2、看Service类是否有依赖对象需要注入
3、有Service信息类需要注入，首先创建Service信息类，然后注入到Service类
4、由Spring-IOC`容器`，对服务类进行统一的管理，比如对象的生命周期。

### IOC优点

* 对程序最小的代价和最小的入侵性实现松散耦合
* 更容易测试，单元测试不再需要单例和JNDI查找机制
* IOC容器支持加载服务的饿汉式初始化和懒加载

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

### IOC支持功能

* 依赖注入
* 依赖检查
* 自动装配
* 指定初始化方法和销毁方法
* 支持回调某些方法，但是需要实现spring接口

其中，最重要的就是依赖注入，从 XML 的配置上说，即 ref 标签。对应 Spring RuntimeBeanReference 对 象。 对于 IoC 来说，最重要的就是容器。容器管理着 Bean 的生命周期，控制着 Bean 的依赖注入

### DI的优势

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

两种依赖方式都可以使用，构造器注入和setter方法注入。最好的解决方法使用构造器参数实现强制依赖，setter实现可选依赖



## AOP面向切面

AOP 本质上是==通过**预编译方式**和**运行期动态代理**实现在**不修改源代码**的情况下给程序**动态统一添加功能**的一种技术。==

> 横向关注点：跨越应用程序多个模块的方法或功能。与我们业务代码无关，但是我们需要关注的部分，就是横切关注点。比如，日志，安全、缓存、事务等等
>
> 切面（Aspect）：横向关注点 被模块化的特殊对象，本质是一个类
>
> 通知（Advice）：切面必须要完成的工作。本质上是类的一个方法。
>
> 目标（Target）：被通知的对象
>
> 代理（Proxy）：向目标对象应用通知之后的对象
>
> 切入点（PointCut）：切面通知 执行的地点的定义
>
> 连接点：与切入点匹配的执行点

本质上用的是代理模式

静态代理

好处：

* 可以使真实角色的操作更加纯粹！不用去关注一些公共的业务
* 公共也就就交给代理角色！实现业务的分工！
* 公共业务发生扩展的时候，方便集中管理

缺点

* 一个真实角色就会产生一个代理角色；代码量会翻倍~开发效率会变低

**动态代理**

好处：

* 可以使真实角色的操作更加纯粹！不用去关注一些公共的业务
* 公共也就就交给代理角色！实现业务的分工！
* 公共业务发生扩展的时候，方便集中管理
* 一个动态代理类 代理的是一个接口，一般就是对应的一类业务
* 一个动态代理类 可以代理多个类 只要实现了同一个接口即可













##什么是Spring beans？



##Bean的作用域





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













## 静态代理



优点：

缺点：每一个真实对象都要一个代理对象

## 动态代理

动态代理和静态代理角色一样

动态代理的代理类是动态生成的，不是我们直接写好的

动态代理类分为两大类：基于接口的动态的代理，基于类的动态代理

* 基于接口——jdk动态代理
* 基于类——cglib
* java字节码实现：javasist



需要了解两个类：Proxy：代理，InvocationHandler：调用处理程序



