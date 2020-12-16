#Spring循环依赖问题

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

# Spring生命周期

https://www.iteye.com/topic/1122859

![](\images\Center)

1. 实例化 Instantiation
2. 属性赋值 Populate
3. 初始化 Initialization
4. 销毁 Destruction

前面三个可以理解为创建bean、第四个是销毁bean

第一，创建

AbstractApplicationContext内部使用DefaultListableBeanFactory

* 实例化Bean对象
* 设置Bean属性
* 如果我们通过各种Aware接口声明了依赖关系，则会出注入Bean对容器基础设施层面的依赖。具体包括BeanNameAware、BeanFactoryAware和ApplicationContextAware,分别会注入Bean ID、Bean Factory或者ApplicationContext
* 调用BeanPostProcessor的前置初始化方法postProcessBeforeInitialization
* 如果实现了InitializingBean接口、则会调用afterPropertiesSet方法
* 调用Bean自身定义的init方法
* 调用BeanPostProcessor的后置初始化方法postProcessAfterInitialization
* 创建过程完毕

第二，Spring Bean 的销毁过程会依次调用 DisposableBean 的 destroy 方法和 Bean 自身定制的 destroy 方法。

**详细**

createBean里面有bean时走resolveBeforeInstantiation

实例化前后的扩展点，即resolveBeforeInstantiation里面

AbstractAutowireCapableBeanFactory的resolveBeforeInstantiation里面有执行InstantiationAwareBeanPostProcessor的postProcessBeforeInstantiation回调方法和执行InstantiationAwareBeanPostProcessor的postProcessAfterInitialization回调方法 。这是实例化的预处理（自定义实例化bean，如创建相应的代理对象）和后处理（如进行自定义实例化的bean的依赖装配）。

当不存在bean的时候，走doCreateBean创建

doCreateBean里面有populateBean、initializeBean

初始化前后的扩展点，即initializeBean里面有

执行BeanPostProcessor扩展点的postProcessBeforeInitialization进行修改实例化Bean

执行BeanPostProcessor扩展点的postProcessAfterInitialization进行修改实例化Bean



