---
name: spring-in-one
title: Spring
date: 2024-08-15
tags: 
- spring
- spring-boot
- aop
- 拦截器
- 事务
- 自动装配
- bean
- 注解开发
categories: spring

---



# Spring IoC

IoC: Inverse of Control 原先调用服务或者DAO的需要自行new出来对象，硬编码，耦合程度高，Spring的Container能够接管对象的创建工作（实际上就是管理Bean） 并且能够根据对象Bean之间的关系进行依赖注入，创建A对象的同时会把B对象创建起来，也就是DI(Dependency Injection)

管理方式：配置文件xml            IoC容器的获取：Spring提供接口

把业务接口的实现类交给Spring管理，遇到接口类，Spring就会自动去找Bean中是否有接口的实现类。

![image-20241020194848409](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20241020194848409.png)

BookDao是接口，实现类为BookDaoImpl，Impl交给Spring管理

DI：依赖注入，依赖用方法传参的方式传入

![image-20241019114405108](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20241019114405108.png)

property name 是成员变量的名字

ref 是要引用的bean id/name

## IoC 配置

### bean 管理

![image-20241019222443537](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20241019222443537.png)

![image-20241019115106810](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20241019115106810.png)

#### name 别名

<u>ATTRIBUTE</u>

bean **name** = "s1 s2 s3"  alias

**ref**可以使用name也可以使用id

getBean 

![image-20241019144037398](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20241019144037398.png)

#### scope 作用范围

<u>ATTRIBUTE</u>

Spring默认创建**单例**bean，scope="singleton" prototype为多例。

- 适合复用的才作为bean交给IoC容器管理
  - 表现层，业务层，DAO层，工具层
- 不适合复用的对象
  - 封装的实体域对象

#### bean 创建方式

##### <mark>使用构造方法<mark>

无参构造器，如果使用构造器进行依赖注入，则走的是有参构造

##### 使用静态工厂实例化Bean

- 工厂的静态方法factoryMethod(return new Bean)，不造工厂，调用工厂的**静态**方法造Bean <u>ATTRIBUTE</u>: factory-method

![image-20241019160109618](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20241019160109618.png)

#####  使用实例工厂实例化Bean

- 先造工厂bean再调用工厂的**实例**方法(return newBean) 造bean

##### <mark>FactoryBean 实例工厂bean<mark>

- 第三方自定义工厂Bean类实现FactoryBean接口，重写方法![image-20241019160946566](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20241019160946566.png)
  - getObject 工厂类的returnNewBean方法
  - getObjectType return Bean.class bean的 字节码
  - isSingleton 单例
- xml 配置![image-20241019161457148](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20241019161457148.png)	

- 主要用于第三方框架和Spring框架对接，他们创建的对象要配置一些参数，这时就需要一个FactoryBean，工厂bean会提供set对象参数的方法，返回的就是配好参的对象，可以省去手动配参的麻烦

#### bean 生命周期

[Review: Bean 生命周期](https://scatteredream.github.io/2025/02/03/rpc-interpretation/#Review-Spring-Bean-%E7%94%9F%E5%91%BD%E5%91%A8%E6%9C%9F%E4%B8%80%E8%A7%88)

[Customizing the Nature of a Bean :: Spring Framework](https://docs.spring.io/spring-framework/reference/core/beans/factory-nature.html#beans-factory-lifecycle-initializingbean) 

##### init-method 初始化

<u>ATTRIBUTE</u> 方法名

##### destroy-method 销毁

<u>ATTRIBUTE</u> 方法名

销毁方式1: 容器关闭 `ctx.close()` appctx这个类没有关闭功能，换一个annotationConfigAppctx才有

销毁方式2: 注册关闭钩子`ctx.registerShutdownHook()` 

##### 自定义实体类实现接口

`DisposableBean` `InitializingBean` 

分别重写destory() afterPropertiesSet()

属性设置就是在属性注入(调用setter)之后调用的方法

##### 生命周期示意图

![image-20241019163321664](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20241019163321664.png)

### DI 依赖注入

**注入** ⇔ **给bean的属性赋值**

![image-20241019222530413](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20241019222530413.png)

![image-20241019163720566](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20241019163720566.png)

注入多个bean，填写多个property

#### 方式

##### <mark>setter注入<mark>

1. setter 引用其他的bean property ref = 其他bean的名称 <u>ATTRIBUTE</u>
2. setter 注入基本数据类型和简单值 property value = 值  <u>ATTRIBUTE</u>
3. property name实际上是根据setter方法 setUserDao 去掉set首字母小写 userDao得到的
4. 先无参构造创建bean，再用setter注入依赖

##### 构造器注入

针对有参构造器，必须显式声明有参构造器

 <u>ATTRIBUTE</u> `<constructor-arg name>`

1.  引用其他bean name是构造器形参名，**耦合度高**，参数先后顺序固定不能变![image-20241019173838816](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20241019173838816.png)
2.  基本数据类型和简单值![image-20241019174107702](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20241019174107702.png)

3.  耦合度高解决方案：**参数适配**

    - `<constructor-arg name>`改成type，解决参数名的高耦合，但是type相同的参数会混淆

    - 改成index，index表示参数的位置
4.  直接有参构造创建bean，可以没有无参构造

#### 方式选择

- **强制依赖**使用构造器进行，使用setter注入有概率不进行注入导致NullPointerException
- **可选依赖**使用setter注入进行，灵活性强
- Spring框架倡导使用构造器,第三方框架内部大多数采用构造器注入的形式进行数据初始化，相对严谨
- 如果有必要可以两者同时使用，使用构造器注入完成强制依赖的注入，使用setter注入完成可选依赖的注入
- 实际开发过程中还要根据实际情况分析，如果受控对象没有提供setter方法就必须使用构造器注入
- <u>自己开发的模块推荐使用setter注入</u>

#### 依赖自动装配 <span id="autowire">autowire</span>

只适用于引用类型

<u>ATTRIBUTE</u> 

不去手动指定，在容器的bean中自动匹配适合的bean。依赖于有参构造或者setter

##### <mark>byType<mark> (依赖setter)

`bean属性的type` 要去匹配 `容器中bean的class`

保证相同class的bean唯一 推荐

##### byName (依赖setter)

`bean属性的name` 要去匹配 `容器中bean的id`

保证必须要有指定名称的bean  耦合度高，不推荐

##### constructor(依赖有参构造器)

##### default

如果<beans\>指定了autowire 此bean跟随beans

##### no

##### 注意事项

- 只能自动装配引用类型（IoC容器不会去管理简单类型）包装类bean根本没法写

- 优先级 < 手动装配

#### 集合注入

集合要注入内容，而不是注一个空壳

```xml
<property name="myArray">
    <array>
    	<value>1</value>
        <value>3</value>
        <value>5</value>
    </array>
</property>

<property name="myList">
    <list>
    	<value>this</value>
        <value>that</value>
        <value>where</value>
    </list>
</property>

<property name="mySet">
    <set>
    	<value>this</value>
        <value>that</value>
        <value>where</value>
    </set>
</property>

<property name="myMap">
    <map>
    	<entry key="A" value="1"/>
        <entry key="B" value="2"/>
        <entry key="C" value="3"/>
    <map>
</property>
        
<property name="myProperties">
    <props>
    	<prop key="A">1</prop>
        <prop key="B">2</prop>
        <prop key="C">3</prop>
    </props>
</property>
```





### 管理第三方Bean

别人写的对象，创建bean，类型是什么？你要配哪些参数？

![image-20241019191451764](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20241019191451764.png)

#### 加载.properties XML Namespace

创建context命名空间

![image-20241019192317628](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20241019192317628.png)

![image-20241019220634669](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20241019220634669.png)

classpath:*.properties 当前模块下所有的配置文件

### 容器 ctx

#### 创建容器方式

![image-20241019221415451](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20241019221415451.png)

#### getBean()

![image-20241019221407031](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20241019221407031.png)

![image-20241019221914707](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20241019221914707.png)

![image-20241019222158026](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20241019222158026.png)

![image-20241019221938964](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20241019221938964.png)

立即加载（饿汉），lazy-init 延迟加载 (懒汉)

## 注解开发

### Quick Start

#### <mark>定义bean@Component</mark>

![image-20241019223116345](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20241019223116345.png)

![image-20241019223156078](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20241019223156078.png)

加上对应的bean的id ，不加就要加载字节码class

#### <mark>纯注解开发@Configuration  @ComponetScan</mark>

![image-20241019224153002](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20241019224153002.png)

获取ctx: `ApplicationContext ctx = new AnnotationConfigApplicationContext(SpringConfig.class)` 

默认xml配置文件只给了beans的命名空间，context还得另外自己加，纯注解开发需要定义一个SpringConfig类，常用的配置都有，不用手动去加命名空间

XML out!

### 2. bean管理

#### 作用范围 @Scope

![image-20241019225307356](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20241019225307356.png)

#### 生命周期 @PostConstruct @PreDestroy

[Review-Spring-Bean-生命周期一览](https://scatteredream.github.io/2025/02/03/rpc-interpretation/#Review-Spring-Bean-生命周期一览) 

[探究Spring Boot中@PostConstruct注解的使用场景-腾讯云开发者社区-腾讯云 (tencent.com)](https://cloud.tencent.com/developer/article/2426419)

[Spring 框架中 @PostConstruct 注解详解-腾讯云开发者社区-腾讯云 (tencent.com)](https://cloud.tencent.com/developer/article/1588212)

![image-20241019225254913](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20241019225254913.png)

Instantiate(Constructor)> @Autowired > @PostConstruct

依赖注入完成，被显式调用之前

[Using @PostConstruct and @PreDestroy :: Spring Framework](https://docs.spring.io/spring-framework/reference/core/beans/annotation-config/postconstruct-and-predestroy-annotations.html)

### DI 自动装配

#### <mark>自动装配@Autowired（引用类型）<mark>

在需要注入依赖的**一个**属性

与配置文件[autowire Attribute of Bean](#autowire)不同，注解Autowired不依赖于setter和有参构造器，直接暴力反射访问private属性，创建对象并注入依赖。 

![image-20241020011345128](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20241020011345128.png)

##### 按名称匹配@Qualifier

autowired默认按类型装 配，同一类型多个实现，用Qualifier指定具体bean名称，*不加Qualifier就按一定规则选择* 

![image-20241020011356460](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20241020011356460.png)

##### 先名称匹配，再按照类型匹配@Resource

[面试突击78：@Autowired 和 @Resource 有什么区别？-阿里云开发者社区 (aliyun.com)](https://developer.aliyun.com/article/1003903) 

- **@Autowired 先根据类型（byType）查找，如果存在多个（Bean）再根据名称（byName）进行查找；**
- **@Resource 先根据名称（byName）查找，如果（根据名称）查找不到，再根据类型（byType）进行查找。** 

注意下方的[Bean注解](#bean)

**@Autowired 支持属性注入、构造方法注入和 Setter 注入，而 @Resource 只支持属性注入和 Setter 注入** 

@Autowired 来自 Spring 框架，而 @Resource 来自于（Java）JSR-250；

@Autowired 只支持设置 1 个参数，而 @Resource 支持设置 7 个参数；

@Autowired 既支持构造方法注入，又支持属性注入和 Setter 注入，而 @Resource 只支持属性注入和 Setter 注入；

##### 为什么属性注入不推荐使用 @Autowired

1. @Autowired 注解注入依赖容器，单元测试不如setter和构造器方便。
2. 无法实现不可变性：final 属性不支持 @Autowired 注解。
3. Autowired默认按照type，如果有两个相同type的bean，就会报错，不如 @Resource

```java
// setter........
@Autowired(required = false)
public void setUserService(UserService userService) {
    this.userService = userService;
}

// constructor..........
@Component
public class MyController {

    private final UserService userService;

    @Autowired
    public MyController(UserService userService) {
        this.userService = userService;
    }
}

```



#### 简单类型注入依赖@Value

需要注入的属性上 写value

![image-20241020011954765](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20241020011954765.png)

#### 导入配置文件@PropertySource

![image-20241020011844984](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20241020011844984.png)

#### <mark>循环依赖</mark>

Spring循环依赖指的是两个或多个Bean之间相互依赖，形成一个环状依赖的情况。简单来说，就是A依赖B，B依赖C，C依赖A，这样就形成了一个循环依赖的环。

Spring循环依赖通常会导致Bean无法正确地被实例化，从而导致应用程序无法正常启动或者出现异常。因此，Spring循环依赖是一种需要尽量避免的情况。

Spring 使用[三级缓存机制](https://scatteredream.github.io/2025/05/25/spring-circle-ref/)部分解决循环依赖问题，但是从 SpringBoot 2.6 开始默认禁止循环依赖，因为这是顶层设计出现问题的表现。 

![](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/633066ae3fcb2fcc117ab142dd90d3da-1730639875106-2.png)

##### 使用构造函数注入

构造函数注入是一种相对保险的方式，因为在实例化Bean时，Spring会检查是否存在循环依赖，并在发现循环依赖时抛出异常，避免死循环。示例代码如下：

```java
@Component
public class A {
    private B b;
    public A(B b) {
        this.b = b;
    }
}
@Component
public class B {
    private A a;
    public B(A a) {
        this.a = a;
    }
}
```

##### 使用@Lazy注解

@Lazy注解可以延迟Bean的实例化，从而避免循环依赖的问题。示例代码如下：

```java
@Component
@Lazy
public class A {
    @Autowired
    private B b;
}
@Component
@Lazy
public class B {
    @Autowired
    private A a;
}
```

##### 使用 setter 注入

使用setter方法注入也可以解决循环依赖的问题，但要注意可能出现的空指针异常。示例代码如下：

```java
@Component
public class A {
    private B b;
	@Autowired
	public void setB(B b) {
        this.b = b;
    }
}
@Component
public class B {
    private A a;
    @Autowired
    public void setA(A a) {
        this.a = a;
    }
}
```

### 管理第三方Bean

#### <span id="bean"><mark>Config配置类中bean的创建@Bean</mark></span>

与@Component不同 这个是方法级别的注解，方法返回的对象将由Spring容器管理

![image-20241020012511399](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20241020012511399.png)

##### @Bean声明的Bean名称？

[spring boot中通过注解@Bean声明的bean的名称是什么？_springboot 声明bean的名称-CSDN博客](https://blog.csdn.net/w1014074794/article/details/106768607)

###### 不指定name属性，bean名称为方法名

###### 指定name属性，bean名称为name



##### 导入其他Config类到核心配置 @Import

不建议直接把其他的配置写到SpringConfig里面，分出去然后import

![image-20241020012755726](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20241020012755726.png)

#### DI 依赖注入

<span id="thirdpartydi">最简单的方法：自己new个对象出来，手动配参数，丢给spring</span> (其实xml就是把手动配参的过程从业务代码中解耦出来)

##### 简单类型依赖注入：成员变量@Value

![image-20241020013246624](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20241020013246624.png)

##### <mark>引用类型依赖注入：方法形参<mark>

![image-20241020013406409](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20241020013406409.png)

### XML vs. Annotation   

![image-20241020013703651](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20241020013703651.png)

## 整合第三方框架

### Spring & MyBatis

[使用纯注解方式Spring整合MyBatis_spring整合mybatis基于注解-CSDN博客](https://blog.csdn.net/m0_64737877/article/details/122608987)

[Spring整合Mybatis(注解方式完整过程，摒弃MyBatis配置文件)_springboot启动去除mybatis-CSDN博客](https://blog.csdn.net/SerikaOnoe/article/details/90639135) 

![image-20241020123544343](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20241020123544343.png)

![image-20241020123756444](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20241020123756444.png)

dao是session用动态代理造出来的，不同业务的内部实现有区别。session也不会一直复用，根源在sqlSessionFactory。

还有一个就是mapper映射，这个跟ssf没什么关系。

> MyBatisConfig - SqlSessionFactoryBean

导入mybatis-spring spring-jdbc，mybatis实现了Spring规定的FactoryBean接口，专门用来造sqlSessionFactory对象。

回顾spring创建对象的方法，一种是[使用构造](#thirdpartydi)器直接得到对象，另一种就是使用factoryBean<E\>得到对象E，定义一个造E的工厂Bean，这样spring就知道类型E创对象需要用工厂Bean的方式。

![image-20241020130218463](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20241020130218463.png)

factorybean中提供了很多设置E参数的方法，最终返回的是一个设置好参数的E，思想还是一样的，只不过套了一层工厂的皮，封装进去很多固定的参数set方法，一般这个E需要xml进行配置(跟真正的业务代码解耦)，工厂Bean就取代了xml，直接给你返回一个配置好的对象。

需要传参就直接在写上方法参数即可，spring自动匹配

![image-20241020145504890](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20241020145504890.png)

> MyBatisConfig - MapperScannerConfigurer

![image-20241020145526730](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20241020145526730.png)

DAO没有实现类了，在原始接口上加Component、Repository给ioc容器标识一下，不标也行。

这个mapperScannerConfigurer是mybatis和spring集成的部分，扫描指定mapper所在的包，mapper生成代理对象，通过factoryBean方式交给Spring容器，所以重点不是让spring知道dao的实现类在哪，重点是要让mybatis知道mapper位置

> JdbcConfig - 创建DataSource的Bean交给Spring管理



### Spring & JUnit

导入spring-test 在Test![image-20241020191443245](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20241020191443245.png)

在test.java中测试。

> @RunWith @ContextConfiguration

> @Autowired @Test

![image-20241020145826024](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20241020145826024.png)

需要引用类型参数直接autowired注入即可，一般是业务类做测试

## BeanDefinition BeanPostProcessor

[Spring-扫描自定义注解](https://scatteredream.github.io/2025/02/03/rpc-interpretation/#Spring-扫描自定义注解) 

# Spring AOP 

## AOP的含义

AOP 面向切面编程 **不惊动原始设计**的情况下增强功能

Spring理念：无侵入式增强功能

- 所有原始方法->连接点(joint point) 在SpringAOP中如此
  - save update delete select
- 需要追加功能的方法->切入点(pointcut)
  - save update delete 
- 具备的共性功能->通知 (advice)
  - method1   method2
- 通知和切入点产生关系->切面 (aspect) 
  - save update delete追加method
- 功能的集合->通知类

![image-20241020152910091](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20241020152910091.png)

连接点包含切入点

## Spring中进行AOP编程

> 导入坐标

![image-20241020154236445](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20241020154236445.png)

### <mark>MyAdvice<mark>

#### 定义通知（功能增强点）

#### 定义切入点 @Pointcut

![image-20241020154309848](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20241020154309848.png)

`private` `void` `空参` 

#### 绑定通知与切入点

![image-20241020154407660](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20241020154407660.png)

#### Spring接管此类 @Component

#### 定义AOP @Aspect

![image-20241020154529656](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20241020154529656.png)

### <mark>SpringConfig @EnableAspectAutoProxy</mark> 

![image-20241020154505110](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20241020154505110-1729410361899-4.png)

## AOP 

### 工作流程

![image-20241020155857766](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20241020155857766.png)

**基于<mark>动态代理<mark>**：

- 匹配失败，new原始对象；

  ![image-20241020160228368](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20241020160228368.png)

  ![image-20241020160243146](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20241020160243146.png)

- 匹配成功，new出来的是原始对象的代理对象；

  ![image-20241020160302140](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20241020160302140.png)

   ![image-20241020160341452](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20241020160341452.png)

**用获取到的bean执行方法**：如果是代理的bean，根据通知和切入点进行方法执行。

### 切入点表达式

切入点：要对其进行增强的方法

切入点表达式：对切入点的描述方式

![image-20241020161523488](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20241020161523488.png)

execution(`public` `User` `com.itheima.service.UserService.findById`(`int`))

public exception 可省略

参数必须有

#### 通配符使用

![image-20241020161833474](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20241020161833474.png)

* ..和*的区别 *用于精准匹配到某个位置

![image-20241020163432072](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20241020163432072.png)

```java
@Pointcut("execution(void com.itheima.dao.BookDao.update())")
@Pointcut("execution(void com.itheima.dao.impl.BookDaoImpl.update())")
@Pointcut("execution(* com.itheima.dao.impl.BookDaoImpl.update(*))")
@Pointcut("execution(void com.*.*.*.update())")
@Pointcut("execution(* *..*(..))")
@Pointcut("execution(* *..*e(..))")
@Pointcut("execution(void com..*())")
@Pointcut("execution(* com.itheima.*.*Service.find*(..))")
//执行com.itheima包下的任意包下的名称以Service结尾的类或接口中的save方法，参数任意，返回值任意

```

### 通知类型

#### 前置@Before

#### 后置@After

#### <u><mark>环绕@Around</mark></u> 

![image-20241020170038080](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20241020170038080.png)

- ProceedingJoinPoint

- 原始方法在环绕方法中执行，用`pjp.proceed()`执行原始方法，<mark>不出现就能隔离原始方法，（权限校验）<mark>

- pjp能接原始方法的返回值，类型为Object，强转后可以在给他返回去，思想和动态代理里的案例比较像：利用反射invoke调用可以拿到返回值，**注意修改通知方法的返回值为Object。**没返回值也可以
- 强制抛Throwable 

#### 得到返回值之后@AfterReturning

和after区别：after只要方法结束即可，不管是得到返回值正常结束还是抛异常。AfterReturning需要得到返回值正常结束才能

#### 抛出异常之后@AfterThrowing

### 案例：JUnit 测量业务层接口执行效率

JUnit 测试服务类就private服务出来，`@Autowired` 

下面写test具体方法。详见JUnit单元测试篇(Java SE)

![image-20241020171154617](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20241020171154617.png)

获取方法签名 (执行信息) `getSignature()`

![image-20241020171252962](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20241020171252962.png)

### 通知获取数据

#### JoinPoint & ProceedingJoinPoint

**作为通知的参数**，如果出现，必须在第一个参数的位置上,PJP是JP的子类

`Object[] getArgs()`: 获取原始方法的参数

` Object proceed()` :环绕 PJP专用，调用原始方法同时返回这个方法的返回值

#### <mark>AOP获取原始方法调用参数<mark>

![image-20241020173959692](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20241020173959692.png)

这样可以对原始参数进行处理，可以增加程序健壮性。

`Object proceed(Object[] args)` 可以把处理以后的参数传给原始方法。

#### 案例：网盘提取码去空格

![image-20241020180157823](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20241020180157823.png)

args本身是Object数组，拿进来需要转成字符串toString getArgs 然后遍历参数数组，对每个字符串参数trim，再把处理以后的传给proceed

#### AOP获取返回值

1. 环绕 pjp proceed 
2. AfterReturning 注解的returning要和形参名字相同

![image-20241020175037570](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20241020175037570.png)

#### AOP接收异常

1. 环绕 不要往出抛Throwable 内部try-catch

2. AfterThrowing 注解的throwing和形参名字相同

![image-20241020174307577](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/屏幕截图 2024-10-20 174257.jpg)

### 环绕通知模拟其他四种通知

| 前置           | 最后调用proceed                          |
| -------------- | ---------------------------------------- |
| 后置           | try catch finally 在finally里写          |
| AfterReturning | Object接住proceed的返回值                |
| AfterThrowing  | 不要往出抛Throwable，try catch Throwable |



## 代理对象的实质—this调用实例方法失效

AOP的核心，是从调用对象的方法时生成代理对象—PROXY，this指向真正的目标对象—TARGET

```java
interface IService{
	void foo();
	void zoo();
}
@Service
class ServiceImpl implements IService{
    
    
	@Override
	void foo(){
		System.println("fooStart...");
		zoo();//this指针调用实例方法
		System.println("fooFinish...");
	}
	@Transactional
	void zoo(){
		save(a);//假设是将a保存到数据库
        int i = 1/0;
		save(b);//假设是将b保存到数据库
	}
}
//
class Test{
    @Autowired
    private IService service;
    
    public static void main(String[] args){
        service.foo();//从外部调用内部的事务方法
    }
}
```

事务的实现基于spring aop，如果直接用this，则会使用target对象进行方法调用，

[Proxying Mechanisms :: Spring Framework](https://docs.spring.io/spring-framework/reference/core/aop/proxying.html#aop-understanding-aop-proxies) 

[Understanding the Spring Framework’s Declarative Transaction Implementation :: Spring Framework](https://docs.spring.io/spring-framework/reference/data-access/transaction/declarative/tx-decl-explained.html#page-title) 

如果是在类内部开启的事务，就需要CGLIB动态代理，实现的基础是方法拦截器，环绕通知，





### 直接调用方法

![image-20241104161707487](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20241104161707487.png)

### 通过代理调用方法

![image-20241104161648309](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20241104161648309.png)

也就是说，如果要让代理生效，首先就是要获取代理对象，而通过@Transactional注解的方法`zoo()`，如果从外部调用，只要获得了代理对象的引用，事务功能就是生效的，因此直接在main中只要获取了代理对象的引用调用service.zoo()方法是没有问题的。

而在目标对象内部，this就是目标对象本身，肯定不会走代理，因此如果实在

### 解决方案—在类的内部获取代理对象

#### AopContext.currentProxy()

```java
synchronized (UserHolder.getUser()) {
    //开启事务需要获取当前的代理对象
    IVoucherOrderService proxy = (IVoucherOrderService)AopContext.currentProxy();
    return proxy.createVoucherOrder(voucherId);
}
```

#### @Autowired 注入服务对象本身

获取service对象即可，这样就能在类内部的上下文中获取proxy代理都象的引用

```java
@Autowired
private ServiceImpl service;

synchronized (UserHolder.getUser()) {
    //开启事务需要获取当前的代理对象
    return service.createVoucherOrder(voucherId);
}
//也可以新开一个Impl2类，把方法移植进去，注入Impl2对象，思路一样
```



## AOP：动态代理对象的生成时机

[【spring系列】spring的AOP是在哪个阶段创建的动态代理对象，spring bean的生命周期中在什么阶段创建的aop动态代理对象，很多人会说第一种，其实还有一种情况也会进行aop_spring的aop代理对象什么时候创建-CSDN博客](https://blog.csdn.net/xzb5566/article/details/141639614)

[关于Spring的两三事：代理对象的生成时机-腾讯云开发者社区-腾讯云 (tencent.com)](https://cloud.tencent.com/developer/article/2154184)

**1） Bean的实例化**：首先，Spring容器会实例化Bean。

**2）Bean的属性填充**：然后，为Bean填充依赖注入的属性。

**3）Bean的初始化：**

- **初始化前（postProcessBeforeInitialization）**：在这一阶段，Spring会调用所有BeanPostProcessor的postProcessBeforeInitialization方法。但此时，代理对象可能还未被创建，因为还需要进一步判断该Bean是否需要被代理。
- **初始化**：接着，执行Bean的初始化方法（如@PostConstruct注解的方法或实现了InitializingBean接口的afterPropertiesSet方法）。
- **初始化后（postProcessAfterInitialization）**：在Bean初始化完成后，Spring会调用所有BeanPostProcessor的postProcessAfterInitialization方法。这通常是**创建动态代理对象的时机(AOP)**，因为此时Bean已经完全初始化并准备使用，而且代理对象可以在这一阶段被创建并替换掉原始的Bean实例。

**4）代理对象的创建：** 

- 在postProcessAfterInitialization方法中，**Spring会检查该Bean是否需要被代理**（通常基于是否存在对应的Advisor或Aspect、注解）。
- 如果需要，Spring会根据Bean的类型（是否实现了接口）选择合适的代理方式（JDK动态代理或CGLIB代理）来创建代理对象。
- 代理对象会封装原始Bean，并在方法调用时插入增强的逻辑（如前置通知、后置通知等）。

**5）Bean的交付**：最后，将创建好的代理对象（如果需要的话）或原始Bean实例交付给客户端使用。

因此，**Spring AOP的动态代理对象主要是在Bean的初始化后的postProcessAfterInitialization阶段创建的**。这一过程确保了代理对象能够封装并增强原始Bean的方法调用，同时保持了Bean的生命周期和依赖注入的完整性。

Spring在完成对象的实例化之前，都会将对象代表的函数式接口放入自身的三级缓存，三级缓存本质就是一个map结构。

[spring中autowired注入自己的代理后，最后容器中的对象是原对象还是代理对象_autowired 注入自己-CSDN博客](https://blog.csdn.net/qq_39002724/article/details/113609903) 

autowired注入自己的代理后，最后容器中的对象只有一个，而且是代理对象。

# Spring 事务

事务用于业务层或者Dao层

[【Spring事务三千问】Spring的事务管理与MyBatis事务管理结合的原理_spring transaction和mybatis的整合 原理-CSDN博客](https://blog.csdn.net/wang489687009/article/details/129259394)

[【spring源码深度解析】：spring是如何利用@Transactional注解实现数据库事务的？把握住事务的基本用法你就懂了 - 知乎 (zhihu.com)](https://zhuanlan.zhihu.com/p/358657396)

## 事务管理整合

[图解Java JDBC和JPA的区别 - 快乐随行 - 博客园 (cnblogs.com)](https://www.cnblogs.com/jddreams/p/14024754.html)

### MySQL

- InnoDB存储引擎支持事务（SQL语句）

### 原生 JDBC

-  注册驱动，
-  获取Connection，
-  建立Statement执行SQL语句，Connection可以管理事务（本质是执行SQL语句）

### DataSource数据源

- 主要用来获取并管理，调度Connection

### 原生 MyBatis (ORM)

- 可以调用外部数据源获取Connection，也可以使用原生JDBC来获取，最终这些Connection可以呗SqlSession获取到。

- SqlSession同样能管理事务，底层是基于Transaction(也是mybatis的一个类)

```java
  SqlSession session = sqlSessionFactory.openSession();
  try {
      int affected_rows = session.insert("com.kvn.mapper.UserMapper.insert", user);
  } catch (Exception e) {
      // 捕获到异常，将操作回滚
      session.rollback();
  }
  // 正常执行，提交事务
  session.commit();
  session.close();
  
```

### Spring联合MyBatis事务管理

`SpringManagedTransaction` 打通了 MyBatis 的事务管理、连接管理 和 spring-tx 的 事务管理、连接管理，使得 MyBatis 与 Spring 可以使用统一的方式来管理连接的生命周期 和 事务处理。

0. 原生的MyBatis 使用的是JdbcTransaction实现类

1. 在一个非 `@Transactional` 标记的方法中执行 sql 命令，则事务的管理会通过 `SpringManagedTransaction` 来执行。
2. 在一个 `@Transactional` 标记的事务方法中执行 sql 命令，则 `SpringManagedTransaction` 的 `commit()/rollback()` 方法不会执行任何动作，而事务的管理会走 Spring AOP 事务管理，即通过 `org.springframework.transaction.interceptor.TransactionInterceptor` 来进行拦截处理。

![image-20241020215834772](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20241020215834772.png)

3. SqlSessionInterceptor 保证了 MyBatis 的 SqlSession 在执行 sql 时使用的连接与 Spring 事务管理操作使用的连接是<u>同一个</u>连接。具体就是通过 Spring 的事务同步器 `TransactionSynchronizationManager` 来保证的。
4. SpringManagedTransaction 中连接的获取是从 Spring 管理的 DataSource 中获取的，这样，数据库连接池也就和 spring 整合在一起了。

## 多线程事务

### javax.sql.Connection

简单地来说，建立`Connection`连接，会消耗数据库系统的如下资源：

|         资源         |                             说明                             |
| :------------------: | :----------------------------------------------------------: |
|        线程数        |     线程越多，线程的上下文切换会越频繁，会影响其处理能力     |
| 创建Connection的开销 | 由于Connection负责和数据库之间的通信，在创建环节会做大量的初始化 ，创建过程所需时间和内存资源上都有一定的开销 |
|       内存资源       |            为了维护Connection对象会消耗一定的内存            |
|        锁占用        | 在高并发模式下，不同的Connection可能会操作相同的表数据，就会存在锁的情况，数据库为了维护这种锁会有不少的内存开销 |

事务的执行依赖于JDBC-connection，connection的建立基于tcp连接，需要耗费很多资源，所以在多线程并发的情况下，connection数目远少于thread数，需要尽可能考虑connection的共用和复用。

connection可以显式开启和关闭事务，遵循事务的ACID原则，因此虽然共用connection，但是同一时间同一connection只能有同一个事务正在执行，也就是串行执行，否则会造成事务紊乱。

一个最佳实践：**当线程需要做数据库操作时，才会真正请求获取JDBC数据库连接,线程使用完了之后，立即释放，被释放的JDBC数据库连接等待下次分配使用**

最简单的方式就是把事务执行的代码块用connection锁对象锁住：事务执行完以后释放（但不销毁）

```java
synchronized(connection){
    //tx......
}
```

线程如何获取锁对象？为了保证一个线程所有dao操作都是用的同一个connection，使用threadLocal存放属于线程自己的connection，如果是直接从连接池获得的话，多个 DAO 就用到了多个Connection，不能完成一个事务。而连接池负责提供缓存和提供connection

### 连接池

[《深入理解mybatis原理》 Mybatis数据源与连接池_mybatis 连接池-CSDN博客](https://blog.csdn.net/luanlouis/article/details/37671851)

原生的JDBC会让connection的close()方法执行数据库连接的释放与销毁，为了保证不更改原生的功能，我们可以使用代理对象，让其close方法不会真正执行，而是回收到数据库连接池中。

使用数据库连接池，通常都是得到一个javax.sql.DataSource[接口]的实例对象，它里面包含了Connection，并且数据库连接池工具类（比如C3P0、JNDI、DBCP等）重新定义了getConnection、closeConnection等方法，所以每次得到的Connection，几乎都不是新建立的连接（而是已经建立好并放到缓存里面的连接），调用closeConnection方法，也不是真正的关闭连接（一般都是起到一个标识作用，标识当前连接已经使用完毕，归还给连接池，让这个连接处于待分配状态）【PS：所以说：使用数据库连接池时，还是要显式的调用数据库连接池API提供的关闭连接的方法】。



至于为什么要用ThreadLocal呢?这个和连接池无关,我认为更多的是和程序本身相关,为了更清楚的说明,我举个例子

```
servlet中获取一个连接.首先,servlet是线程安全的吗?

     class MyServlet extends HttpServlet{
         private Connection conn;
     }
     ok,遗憾的告诉你,这个conn并不是安全的,所有请求这个servlet的连接,使用的都是一个Connection,这个就是致命的了.多个人使用同一个连接,算上延迟啥的,天知道数据会成什么样.
     因此我们要保证Connection对每个请求都是唯一的.这个时候就可以用到ThreadLocal了,保证每个线程都有自己的连接.
     改为 private ThreadLocal<Connection> ct = new ThreadLocal<Connnection>();
     然后从连接池获取Connection,set到ct中,再get就行了,至于得到的是哪个Connection就是连接池的问题了,你也管不到.
```

### ThreadLocal?

就是为每一个使用该变量的线程都提供一个变量值的副本，是每一个线程都可以独立地改变自己的副本，而不会和其它线程的副本冲



## 开启步骤

### 业务层**<mark>接口<mark>**为业务方法打开事务@Transactional

![image-20241020204651727](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20241020204651727.png)

- @Transactional是方法级别或者类级别的注解，可以**开在整个接口上**，也可以开在单个方法上

- 接口能够提高复用性，降低耦合

### JdbcConfig创建事务管理器Bean@Bean

![image-20241020204933459](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20241020204933459.png)

PlatformTransactionManager是Spring规定的，DataSourceTransactionManager可以动，根据具体的技术选择

要注意，事务管理器的datasource和mybatis用的datasource必须是同一个，不然

### SpringConfig打开事务@EnableTransactionManagement

![image-20241020205233232](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20241020205233232.png)

## 事务角色

![image-20241020205625500](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20241020205625500.png)

![image-20241020205648701](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20241020205648701.png)



## 事务配置

![image-20241020222103484](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20241020222103484.png)

有些异常不会触发回滚，需要手动设置一下rollbackFor

### 追加日志

try finally结构，finally 记日志功能必定触发

![image-20241020222628555](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20241020222628555.png)

### 事务传播行为控制

[(1) java - Spring事务传播行为详解 - 个人文章 - SegmentFault 思否](https://segmentfault.com/a/1190000013341344#item-2-2)

![image-20241020222917510](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20241020222917510.png)

transfer、AccountDao中所有的数据层方法、日志记录的业务方法都加了Transacitonal注解。

- transfer作为方法的调用者，是事务的管理员。
- 其他作为被调用者，是事务的协调员。
- 如果默认设置Required，管理员开事务，协调员都会加入

`@Transactional(propagation = Propagation.REQUIRES_NEW)`

开启事务：`@Transactional`

管理员肯定要开启事务，管理员默认是`REQUIRED`一般不用改，某一个协调员要单开另外一个事务，那么就可以把这个协调员的事务传播机制改成`REQUIRES_NEW` 

#### Propagation.REQUIRED

**外围方法未开启事务的情况下`Propagation.REQUIRED`修饰的内部方法会新开启自己的事务，且开启的事务相互独立，互不干扰。** 

**在外围方法开启事务的情况下`Propagation.REQUIRED`修饰的内部方法会加入到外围方法的事务中，所有`Propagation.REQUIRED`修饰的内部方法和外围方法均属于同一事务，只要一个方法回滚，整个事务均回滚。** 

#### Propagation.REQUIRES_NEW

用途：某段逻辑必须**独立提交或回滚**（如记录日志、发操作记录），避免日志失败导致主逻辑失败）

**外围方法未开启事务的情况下`Propagation.REQUIRES_NEW`修饰的内部方法会新开启自己的事务，且开启的事务相互独立，互不干扰。** 

**在外围方法开启事务的情况下`Propagation.REQUIRES_NEW`修饰的内部方法依然会单独开启独立事务，且与外部方法事务也独立，内部方法之间、内部方法和外部方法事务均相互独立，互不干扰。** *并且使用不同的 Connection 连接。*

## 事务失效

[spring 事务失效的 12 种场景_spring 截获duplicatekeyexception 不抛异常-CSDN博客](https://blog.csdn.net/hanjiaqian/article/details/120501741)

- 访问权限，private,default无法生效
- 方法用final或static修饰，代理对象无法重写
- 多线程调用事务方法，两个线程获取的不是同一个连接
- 数据库或表不支持事务（MySQL的MyISAM不支持事务）
- 未开启事务或未将类纳入Spring管理`@Transactional` `@Service` 

### <span id="selfinvoke">方法自调用</span>

```java
@Service
public class UserService {
 
    @Autowired
    private UserMapper userMapper;
 
  
    public void add(UserModel userModel) {
        userMapper.insertUser(userModel);
        updateStatus(userModel);
    }
 
    @Transactional
    public void updateStatus(UserModel userModel) {
        doSameThing();
    }
}
```

我们看到在事务方法 add 中，直接调用事务方法 updateStatus。从前面介绍的内容可以知道，updateStatus 方法拥有事务的能力是因为 spring aop 生成代理对象proxy，但是这种方法直接调用了 this 对象的方法，所以 updateStatus 方法不会生成事务。

由此可见，在同一个类中的方法直接内部调用，会导致事务失效。



## 子线程为什么不能共享父线程的事务

![img](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/1911189-20200407004610684-971276319.jpg)

事务：JDBC connection（连接）  MyBatis sqlSessionFactory（会话）这些都是一个意思。Spring 在将其整合后，会绑定到每个线程私有的 ThreadLocal 中。

- 连接池中的每个 `Connection` 对应不同的 TCP 四元组，都是一个独立的物理连接，物理上不同（不同的socket、不同的会话）。
- 数据库事务是“会话级别”的，绑定在**某一条连接**上，每条连接有**独立的事务上下文、会话状态、隔离级别**，不同连接之间的事务、锁、变量 **互不干扰**。
- 连接池除了管理 Connection 对象本身，还会：记录该连接是否存活、检查该连接是否超时、检查连接是否被污染（如事务未提交、连接状态被改）、所以每条 Connection 在连接池中不仅仅是个“Connection对象”，还包含一些“元数据”状态。
- 为了支持并发访问，一个线程用一条连接，多个线程可以并发请求数据库（否则一条连接只能排队等待）。

- 在Spring框架中，`@Transactional`的事务管理默认是基于线程绑定的（使用`ThreadLocal`存储事务上下文）。

```yaml
spring:
  datasource:
    url: jdbc:mysql://localhost:3306/test
    username: root
    password: root
```

**子线程访问自己的ThreadLocal，发现没有存父线程的连接 → 所以子线程会重新去连接池获取一个新的连接**：

- 保证多线程访问数据库时线程间互不干扰
- 保证每个线程内的事务隔离
- 防止一个线程操作数据库时，另一个线程“偷用”这个连接引发混乱（如事务状态、关闭连接）

```java
@Transactional
public void parentMethod() {
    saveData1(); 
    new Thread(() -> {
        saveData2(); 
    }).start();
}
```

- 如果像这样，新的线程有自己的 ThreadLocal，必须自己再新拿一条连接来操作数据。

<mark>总结：不要在事务里面开启子线程，应该是在子线程里开启事务。</mark>

# Spring MVC

## 入门案例

Java实现MVC模型的web框架，灵活性强，主要进行**表现层开发** Controller。

### bean创建@Controller

#### 方法级别注解 请求映射 @RequestMapping

#### 方法级别注解 设置响应 @ResponseBody

![image-20241021161644841](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20241021161644841.png)

都是方法级别注解	

### 创建SpringMvcConfig@Configuration@ComponentScan

扫描到controller

### 创建servlet容器Config

ServletContainerInitConfig继承AbstractDispatcherServeltInitializer类重写对应方法。都是一次性

![image-20241021161341011](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20241021161341011.png)

- `getServletMappings` 表示接管URL的那个部分的映射
- `createRootApplicationContext` 创建Spring Framework容器并指定配置	
- `createServletApplicationContext` 创建web容器并指定配置
- web容器 **servlet容器**

### 工作流程

![image-20241021162133487](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20241021162133487.png)



### bean加载@ComponentScan.Filter

<u>Spring避免加载springmvc的controller</u>

![image-20241021162700082](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20241021162700082.png)

导包：mybatis自动代理会返回dao接口的实现对象，可以不写，但是其他技术不一定是这样，所以为了通用性还是应该导dao

![image-20241021164147565](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20241021164147565.png)

SpringConfig扫描排除含有Controller注解的类：**Filter** 可以更细粒度地加载bean

exclude排除

![image-20241021164311077](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20241021164311077.png)

加了configuration的类，spring都会将其作为配置类，里面如果有componentScan，就会扫上。

![image-20241021164930504](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20241021164930504.png)

创建容器设定配置再去返回容器，简化过程只需要指定类的字节码即可

![image-20241021165128160](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20241021165128160.png)

## 配置Controller

### 请求Request相关

#### 请求映射路径@RequestMapping

对于不同的controller可能会优相同方法，这时会有冲突问题，解决办法是在controller**<mark>类<mark>**上加一个@RequestMapping，要和**<mark>方法<mark>**的@RequestMapping注解结合一下。

![image-20241021172423291](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20241021172423291.png)

#### 名称匹配 指定<mark>请求参数<mark>名@RequestParam

用于接收GET请求中URL的查询参数，也可以接收POST请求的参数（表单）

> You can use the `@RequestParam` annotation to bind Servlet request parameters (that is, query parameters or form data) to a method argument in a controller.

与mybatis类似，对于外部传进来的请求参数用map封装，因此**@RequestParam接收的是key-value形式的参数**，**发送get请求**只会处理URL中的参数，忽略请求体中的数据

**发送post请求**时，<mark>表单数据在请求体中<mark>，不过仍然是username=root这样键值对的形式存在，如果URL里有请求参数，服务端收到以后会一并加到map中，打印出来，即使方法里的参数只是一个String也能打印出来

![image-20241021220731172](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20241021220731172.png)<img src="https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20241021220755961.png" alt="image-20241021220755961" style="zoom:150%;" />

@RequestParam XXX xxx 表示查询参数用XXX类型接

[SpringMvc--@RequestBody和@RequestParam注解以及不加注解接收参数的区别_不写接收参数的注解,默认使用什么的-CSDN博客](https://blog.csdn.net/weixin_43606226/article/details/106545024)

[解决mybatis不加@Param报错 org.apache.ibatis.binding.BindingException_51CTO博客_mybatis @Param](https://blog.51cto.com/u_12393361/5021343)

[记一次SpringMVC碰到的坑 - zeng1994 - 博客园 (cnblogs.com)](https://www.cnblogs.com/zeng1994/p/9110632.html)

[关于SpringMvc使用时,不加@RequestParam注解,根据方法形参名也可以获取请求值的分析_spring 请求体不写注解-CSDN博客](https://blog.csdn.net/as513385/article/details/93512699)

和 **MyBatis** 一样，使用反射机制获取参数名称，JDK8以后 java.lang.reflect.Parameter 中能够获取参数相关信息，框架就是利用这个机制，不加RequestParam获取参数信息。不然就只有arg0 arg1这种形式。

- required参数 是否为必传参数，默认必传
- defaultValue 参数默认值

#### 传递多种类型的<mark>请求参数<mark>



与mybatis类似

- pojo：直接使用pojo内部的字段名称

- 嵌套pojo：address.字段名称

- 数组：直接接收字符串数组即可，请求的参数名就是数组的形参名，会把数组当成一个独立的参数

  ![image-20241021192646729](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20241021192646729.png)

- 集合类型：加RequestParam注解。 因为集合属于**引用类型**，spring会把它当成pojo处理（造pojo然后根据字段名注入依赖），不加param注解，spring就不会像数组一样把他当成一个独立的参数。

##### 传JSON @<mark>EnableWebMvc<mark>

导坐标 webMvc

![image-20241021193842326](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20241021193842326.png)

###### 参数在请求体里@RequestBody

@RequestBody XXX xxx 表示请求体中的数据用XXX类型接

RequestBody请求体中的数据通常是以JSON、XML等格式发送的，可以将请求体中的数据自动绑定到指定的Java对象上。

1. 参数写在请求体里，用List接收json数组

![image-20241021194047781](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20241021194047781.png)

2. 用Pojo类接单个pojo对象json

![image-20241021194303198](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20241021194303198.png)

![image-20241021194445000](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20241021194445000.png)

3. 用List<pojo>接收多个pojo对象的json，

![image-20241021194340592](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20241021194340592.png)

一个请求，只有一个RequestBody；一个请求，可以有多个RequestParam。



**Body vs Param:**

![image-20241021222507281](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20241021222507281.png)

##### 日期参数格式化@DateTimeFormat(pattern="yyyy-MM-dd")

![image-20241021223057976](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20241021223057976.png)

默认yyyy/MM/dd 其他形式不认识，需要自己手动指明formatPattern

##### Converter接口-将字符串参数转换成Java类型

请求里面的参数都是以字符串形式发来的，converter要根据形参类型，把字符串转成对应类型提供给方法，这就是为什么前面能把字符串12按照需求转换成int 12

![image-20241021223621200](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20241021223621200.png)

@EnableWebMvc

### 响应Response相关

#### 响应页面（跳转页面）

```java
@RequestMapping("/toPage")
public String toJumpPage(){
    return "page.jsp";
}
```

直接在方法里 return "page.jsp" 

Spring默认认为Controller的方法返回的就是一个页面

#### 响应文本数据@ResponseBody

![image-20241021225227708](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20241021225227708.png)

如果直接return一个字符串Spring会认为这个字符串是一个网页，响应分为响应行响应头和响应体，响应头里是放状态码之类的，只能在响应体里返回数据，加@ResponseBody注解表示返回值就是响应体。

```java
@RequestMapping("/toText")
@ResponseBody
public String toText(){
    return "page.jsp";
}
//表示返回 page.jsp 这个字符串
```

#### 响应POJO对象(JSON形式)

```java
@RequestMapping("/toJsonPOJO")
@ResponseBody
public User toJsonPOJO(){
    System.out.println("返回json对象数据");
    User user = new User();
    user.setName("itcast");
    user.setAge(15);
    return user;
}
```

直接返回user即可，jackson帮我们做的

##### 响应POJO对象集合（JSON数组）

```java
@RequestMapping("/toJsonList")
@ResponseBody
public List<User> toJsonList(){
    System.out.println("返回json集合数据");
    User user1 = new User();
    user1.setName("传智播客");
    user1.setAge(15);

    User user2 = new User();
    user2.setName("黑马程序员");
    user2.setAge(12);

    List<User> userList = new ArrayList<User>();
    userList.add(user1);
    userList.add(user2);

    return userList;
}
```

##### HttpMessageConverter接口

POJO转JSON字符串

![image-20241021225843796](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20241021225843796.png)

![image-20241021225823024](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20241021225823024.png)

### REST 风格 

**Re**presentational **S**tate **T**ransfer

![image-20241021230252993](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20241021230252993.png)

根据请求的方式区分 GET POST PUT DELETE

同一URL，请求方式不同，调用的方法也不同

![image-20241021230150675](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20241021230150675.png)

RESTful：用REST风格访问资源

#### @RequestMapping 加 method 参数

`@RequestMapping(value = "/users/{id}" ,method = RequestMethod.DELETE)` 

![image-20241021231918974](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20241021231918974.png)

![image-20241021231836548](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20241021231836548.png)

##### URL占位符传参 <mark>@PathVariable<mark>

value="/users/{id}"  URL中的{id}和用@PathVariable修饰的方法参数id是一致的

![image-20241021231846312](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20241021231846312.png)

##### @RequestBody@RequestParam@PathVariable

![image-20241021232010983](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20241021232010983.png)

#### <mark>RESTful 快速开发<mark>

##### 类级别注解 @RequestMapping

省去所有user前缀，写一次就好。

##### 类级别注解 @RestController

类级别的@RequestBody，表示所有类的返回值都是请求体的数据，既然@Controller和RequestBody都要写，合而为一即可

##### 方法级别注解 @PostMapping

```java
@RequestMapping(value = "/{id}" ,method = RequestMethod.DELETE)
@GetMapping
@PostMapping("/{id}")
```

![image-20241021232955549](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20241021232955549.png)

#### RESTful 页面交互案例

RestController PostMapping GetMapping DeleteMapping PutMapping 

方法参数RequestBody

![image-20241021235556399](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20241021235556399.png)

##### Config 类 SpringMvcSupport 放行静态页面访问

@Configuration

默认SpringMvcConfig接管/后所有东西，通过ResourceHandlerRegistry 设置SpringMVC如何处理对静态资源的访问 

![image-20241021235616215](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20241021235616215.png)

##### 前端ajax

![image-20241021235623670](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20241021235623670.png)

## SSM整合

### 创建工程

### 整合Config包

#### SpringConfig

`Configuration` `ComponentScan` `Import`

##### MyBatisConfig & JdbcConfig

JdbcConfig：数据源 DataSource Bean

MyBatisConfig：sqlSessionFactoryBean

`Bean` `jdbc.properties` 

#### SpringMvcConfig

`Configuration` `ComponentScan` `EnableWebMvc`

##### ServletConfig

rootApplicationContext和webApplicationContext

### 编写后端模块

#### Domain

实体类，User

#### Dao

MyBatis Mapper自动代理，写接口，写方法结合注解

![image-20241022135643205](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20241022135643205.png)

占位符对应参数的名称，这里映射实体类中的字段信息	 

![image-20241022135845990](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20241022135845990.png)

#### Service

BookDao@Autowired

dao接口加repository注解（可加可不加）

#### RestController

参数在url中：PathVariable

参数在请求体：RequestBody

#### JUnit 测试 Service

#### Postman 测试 Controller

#### Spring 事务激活

`JdbcConfig` 里 加PlatformTransactionManager Bean, 接dataSource参数

`Service`接口添加@Transactional

### 前后端联调

#### 表现层数据封装模型 - 设置统一的返回结果集Result

实际开发过程中前后端<mark>约定<mark>

```java
public class Result{
    private Object data;
    private Integer code;
    private String message;
    //下方提供若干构造方法,有参无参
    public Result() {
    }

    public Result(Integer code,Object data) {
        this.data = data;
        this.code = code;
    }

    public Result(Integer code, Object data, String msg) {
        this.data = data;
        this.code = code;
        this.msg = msg;
    }
    //.....getter setter
}
```

![image-20241022152253512](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20241022152253512-1729581786093-1.png)

##### Result.data

业务方法不同返回数据格式也不同，可能是true false这样的text，也可能是json数据，还可能是json数组，约定将数据封装到<mark>data<mark>字段中

![image-20241022144637267](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20241022144637267.png)

##### Result.code

不同业务方法可能会返回相同的内容，返回一个true可能对应新增，修改，删除的业务方法，加一个识别码<mark>code<mark>字段区分 ，可以约定尾数是0表示失败，尾数是1表示成功：

![image-20241022145108783](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20241022145108783.png)

```java
public class Code {
    public static final Integer SAVE_OK = 20011;
    public static final Integer DELETE_OK = 20021;
    public static final Integer UPDATE_OK = 20031;
    public static final Integer GET_OK = 20041;

    public static final Integer SAVE_ERR = 20010;
    public static final Integer DELETE_ERR = 20020;
    public static final Integer UPDATE_ERR = 20030;
    public static final Integer GET_ERR = 20040;
}
//这里Result的构造器识别的是Integer code
/*
Enum枚举：CodeEnum是一个类，类内部有一字段code(Integer)
*/
```

![image-20241022152238169](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20241022152238169.png)

##### Result.message

一些业务方法，本来应该返回json，没查到只能返回null，不能直接把null展示给用户看，展示的是message信息

##### Controller 返回值统一设定为 Result

将返回值封装到Result中，data

```java
@GetMapping
    public Result getAll() {
        List<Book> bookList = bookService.getAll();
        Integer code = bookList != null ? Code.GET_OK : Code.GET_ERR;
        String msg = bookList != null ? "" : "数据查询失败，请重试！";
        return new Result(code,bookList,msg);
    }
//data字段是否为null？
```

#### 返回数据格式统一 - 异常处理器<mark>@RestControllerAdvice<mark>

- <mark>类级别注解<mark>

- 后端抛出的异常如果不处理，就会抛到前端页面，不美观，并且不会返回任何数据，导致<mark>数据不统一<mark> 
- 要让WebMvcConfig扫到这个Advice类

##### 常见异常诱因

- **框架内部抛出的异常**:因使用不合规导致
- **数据层抛出的异常**:因外部服务器故障导致(例如:服务器访问超时)
- **业务层抛出的异常**:因业务逻辑书写错误导致(例如:遍历业务书写操作，导致索引异常等)
- **表现层抛出的异常**:因数据收集、校验等规则导致(例如:不匹配的数据类型间导致异常)
- **工具类抛出的异常**:因工具类书写不严谨不够健壮导致(例如:必要释放的连接长期未释放等)

<u>处理方法</u>：全部抛到表现层Controller —— AOP 编程，**用最少量的代码实现最强大的功能**，快速统一地处理异常 

##### 方法级别注解 处理具体类别的异常<mark>@ExceptionHandler<mark>

![image-20241022153703868](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20241022153703868.png)

处理异常返回的结果也要封装成Result

### 项目异常处理方案 (捕获异常并返回Result)

#### 异常分类

- **业务异常 BusinessException** 可预期
  - 发送对应消息，提醒用户规范操作
- **系统异常 SystemException** 
  - 发送固定消息，安抚用户
  - 发送特定消息给运维，提醒维护
  - 记录日志
- **其他异常 Exception**
  - 发送固定消息，安抚用户
  - 发送特定消息给开发，提醒维护（纳入预期范围）
  - 记录日志

#### 自定义异常

![image-20241022154704744](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20241022154704744.png)

##### 继承RuntimeException

![image-20241022155606802](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20241022155606802.png)

加一个code属性（getter setter），重写RuntimeException的方法。异常构造的时候需要用到这些构造器，包装返回数据要用到code和message

建议放在源根的exception包下面

##### 异常代码扩充

![image-20241022160311854](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20241022160311854.png)

##### 触发异常

![image-20241022160332391](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20241022160332391.png)

##### 拦截并处理异常 返回Result

在RestControllerAdvice下方的类中处理对应类型的异常，将异常继续封装成Result返回

![image-20241022160546843](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20241022160546843.png)

### 放行静态资源配置

#### SpringMvcSupport（Config类）

加Configuration注解，继承WebMvcConfigurationSupport类，重写resourceHandler方法

#### <mark>Config包详解<mark> 

##### ServletContainersInitializerConfig (Servlet容器配置类)

用来构建ServletContext，继承自 AbstractDispatcherServletInitializer，Spring MVC是建立在 DispatcherServlet 基础之上的，每一个请求最先访问的都是它，负责转发每一个Request请求，所以是必不可少的。

先以 <mark>Abstract<u>DispatcherServlet</u>Initializer<mark> 为例介绍这个加载Config类的职责 具体介绍在[下方](#webappinit)

###### a. createRootApplicationContext

需要加载Spring IoC容器的配置类(SpringConfig)，返回配置好的 [Root WAC](#wac) 

###### b.createServletApplicationContext

需要加载WebMvc容器的配置类(SpringMvcConfig) 返回配置好的 [Servlet WAC](#swac) 

![image-20241022205635235](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20241022205635235.png)

###### c.getServletMappings

配置由此DispatcherServlet接管的URL映射路径

##### SpringConfig(@Configuration)

<u>对应applicationContext.xml</u>，配置 [Root WAC](#wac)  

[applicationContext.xml及spring-servlet.xml详解 - 长木木弓 - 博客园 (cnblogs.com)](https://www.cnblogs.com/zzjlxy-225223/p/12611093.html)  

注解开发用来替代传统的XML配置文件，因此可以透过xml与注解的映射关系来了解，`@Configuration`用来替代`<beans>` `</beans>` 在应用启动时，Spring 会自动扫描并加载所有带有 `@Configuration` 注解的类，根据`@ComponentScan`扫描要加入的`@Component`(代替`<bean>` `</bean>`)，最终创建出对应的容器(Context) 

###### @Import({MyBatisConfig.class,JdbcConfig.class})

`MyBatisConfig` 和 `JdbcConfig`中方法级别注解`@Bean`用来替代 `<bean>` `</bean>` 标签（表示方法返回的对象当成Bean/Component交给Spring容器管理）虽然没有加`Configuration`注解，但是会由`SpringConfig` `@Import`，导入的还是SpringConfig配置的容器，属于 [Root WAC](#wac) 

##### SpringMvcConfig(@Configuration @EnableWebMvc)

<u>对应spring-servlet.xml</u>，配置 [Servlet WAC](#swac) 

[@EnableWebMvc (Spring Framework 6.1.14 API)](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/web/servlet/config/annotation/EnableWebMvc.html) 

[WebMvcConfigurationSupport (Spring Framework 6.1.14 API)](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/web/servlet/config/annotation/WebMvcConfigurationSupport.html)  (WMCS)

- `@EnableWebMvc` 会通过导入 `WMCS` 完成 Spring MVC 默认配置的添加，**只有一个类能拥有此注解** 
- 不写 `@EnableWebMvc` 直接继承 WMCS 也可以实现相同效果

- 自定义具体某项WebMvc配置：`extends WebMvcConfigurer` 允许多个WebMvcConfigurer存在，但是具有一定侵入性


![image-20241023210351697](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20241023210351697.png)

- 主要Config：`@EnableWebMvc` + 继承 `WebMvcConfigurer` 重写方法 + `@ComponentScan` 其他Config类
- 次要Config：继承 `WebMvcConfigurer`重写方法+@Configuration确保被主要Config扫描到
- 没有暴露高级设置，如果需要高级设置 需要第二种方式直接继承 WMCS 来做更高级别的配置，此时要移除@Enable注解

###### SpringMvcSupport(@Configuration) 

<u>对应springMvcContext.xml</u>，配置 [Servlet WAC](#swac) 

- 继承WMCS，完成对SpringMVC的默认配置，重写resourceHandler方法，实现在默认配置基础上的自定义。
- 案例中的SMS是配置类，用于配置容器，被SMC扫config包扫到了，此时SMS这里的自定义配置会覆盖SMC的Enable注解

### 前端逻辑

查询： get请求

保存/添加：post请求

新增要弹出表单，添加成功要关闭表单并**清空表单数据**，不论成功与否finallyGetAll回显 ，按照识别码判别成功与否

![image-20241023190220059](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20241023190220059.png)

修改：put请求

![image-20241023191646261](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20241023191646261.png)

![image-20241023191751771](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20241023191751771.png)

删除：delete请求

![image-20241023191526268](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20241023191526268.png)

![image-20241023191952545](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20241023191952545.png)



### 容器之间的嵌套关系 + 概念解释 (源码解析)

![image-20241022175254007](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20241022175254007-1729595662475-3.png)

#### <span id="wac">WebApplicationContext(WAC)</span> 

- ApplicationContext(AC) 表示整个 Spring 应用的上下文。WAC是普通AC的扩展，它具有Web应用程序所需的一些额外功能，比如可以<u>get</u>ServletContext或者<u>set</u>ServletContext
- `Root WAC`在应用启动时首先被加载，并且作为父上下文，供表示层使用，主要负责管理服务层（Service）、数据访问层（DAO）、中间件配置等非 Web 层（表示层）的 Bean

#### <span id="swac">Servlet WebApplicationContext(Servlet WAC)</span> 

- `Servlet WAC` 是 `Root WAC` 的**子上下文**，专门用于处理表示层的 Bean 和配置。比如控制器（`Controller`）、视图解析器、拦截器(`Interceptor`)等
- 每个 `DispatcherServlet` 实例会有一个独立的 `Servlet WAC` 

##### Parent & Child ApplicatitonContext 

Root WAC 作为 所有 Servlet WAC 的 Parent，DispatherServlet在创建属于自己的ServletContext的getAttribute方法来判断是否存在Root WebApplicationContext。如果存在，则将其设置为自己的parent。这就是父子上下文(父子容器)的概念，getParentBeanFactory。

对于作用范围而言，在DispatcherServlet中可以引用由ContextLoaderListener所创建的RootWAC中的内容，而反过来不行。当Spring在执行ApplicationContext的getBean时，如果在自己context中找不到对应的bean，则会在父ApplicationContext中去找。这也解释了为什么我们可以在DispatcherServlet中获取到RootWAC中的bean。

![image-20241023154504980](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20241023154504980.png)

#### ServletContext

- Servlet容器（Tomcat）在启动一个web应用时，根据web.xml 会为整个应用创建一个<mark>唯一<mark>的ServletContext(SC)对象，应用内部所有的Servlet共享同一个SC。
- ServletContext是Servlet与Servlet容器（Tomcat）之间直接通信的接口。
- 容器中的Servlet可以通过它来访问容器中的各种资源
- ServletContext跟XML一样，由Attributes组成，要访问资源就要通过字符串name访问，可以通过`void setAttribute(name, object) `来将ServletContext与你的object绑定，`Object getAttribute(name)`可以得到object
- `Enumeration<String> getInitParameterNames()` 获取所有 `<context-param/>` 参数的名称 字符串枚举
- ` String getInitParameter(name)` 根据name获取指定的 `<context-param/>` 参数值

![image-20241023013135254](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20241023013135254.png)

##### Root WAC, Servlet WAC, ServletContext之间的关系

![image-20241022230909096](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20241022230909096.png)

- Root WebApplicationContext存储key为`WebApplicationContext.ROOT_WEB_APPLICATION_CONTEXT_ATTRIBUTE`，可以通过此Key访问Root WAC。
- WebApplicationContextUtis工具类提供了从ServletContext获取RootWAC的方法：
  - `WebApplicationContextUtils.getWebApplicationContext(ServletContext sc)`

- WAC提供了获取ServletContext的抽象方法 `getServletContext()` 

![context](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/无标题-1729654361571-5.png)

##### web.xml 配置 ServletContext

Tomcat创建web应用时，会构建ServletContext对象，根据web.xml中的配置把如下参数都存到ServletContext对象中，注册Listener，Servlet等

```xml
<?xml version="1.0" encoding="UTF-8"?>  
  
<web-app version="3.0" xmlns="http://java.sun.com/xml/ns/javaee"  
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"  
    xsi:schemaLocation="http://java.sun.com/xml/ns/javaee http://java.sun.com/xml/ns/javaee/web-app_3_0.xsd">  
    
    <!—ServletContext自有的init 参数-->
    <context-param>  
        <!—创建Root WAC所需要的配置文件路径-->
        <param-name>contextConfigLocation</param-name>
        <param-value>/WEB-INF/spring/applicationContext.xml</param-value>  
    </context-param>  
  	
    <!—注册ContextLoaderListener-->
    <listener>  
        <listener-class>org.springframework.web.context.ContextLoaderListener</listener-class>  
    </listener>  
    
    <!—注册DispatcherServlet ServletConfig-->
    <servlet>
        <servlet-name>dispatcher</servlet-name>  
        <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class> 
        <!—init Servlet所需参数-->
        <init-param>  
            <!—创建Servlet WAC所需参数-->
            <param-name>contextConfigLocation</param-name>  
            <param-value>/WEB-INF/spring/spring-servlet.xml</param-value>  
        </init-param>  
        <load-on-startup>1</load-on-startup>  
    </servlet>  
    <servlet-mapping>  
        <!—指定某个servlet的URL映射路径-->
        <servlet-name>dispatcher</servlet-name>  
        <url-pattern>/*</url-pattern>  
    </servlet-mapping>  
  
</web-app>
```



#### ContextLoader<mark>Listener<mark> - 创建 Root WAC

- 本质就是一个Listener，因此需要在web.xml中注册

- 实现了ServletContextListener接口，EventListener->ServletContextListener

- 继承了ContextLoader类，见名知意，是用来加载WAC的，有一个WAC参数context，所有方法都是围绕加工这个context字段进行的

##### WebApplicationContext initWAC(ServletContext sc) 

ContextLoaer 接收一个ServletContext参数sc，调用initWAC方法返回加载好的WAC对象this.context

打印在服务器日志上 `servletContext.log("Initializing Spring root WebApplicationContext");`

中间调用`createWAC(ServletContext sc)`返回一个ConfigurableWAC对象，将其**parentContext设置为sc**，

再把这个CWAC和sc传给`configureAndRefreshWAC(CWAC cwac,ServletContext sc)`方法进行配置（ServletContext是根据web.xml构建的，根据key: contextConfigLocation找到RootWAC的配置文件`applicationContext.xml`）

之后在sc中创建一个Attribute，使得能通过ServletContext对这个Root WAC进行访问（键值对形式）

`setAttribute(` `WebApplicationContext.ROOT_WEB_APPLICATION_CONTEXT_ATTRIBUTE`, `this.context)`

最终返回 `this.context` 作为 Root WAC

##### WebApplicationContext contextInitialized(ServletContextEvent sce)

ContextLoaderListener 能监听Web应用启动或关闭的事件（会修改ServletContext中的参数），触发contextInitializaed/contextDestroyed，创建或销毁Root WAC。

#### Dispatcher<mark>Servlet<mark> - 创建 Servlet WAC

- 本质就是一个Servlet，所以需要在web.xml中注册，继承自HttpServlet->HttpServletBean->FrameworkServlet

- Spring MVC 的核心前端控制器，用于处理所有进入的 HTTP 请求。将请求分发给适当的处理器（控制器 Controller），并在处理后将响应返回给客户端。

- 每一个 `DispatcherServlet` 都拥有自己的 [Servlet WebApplicationContext](#swac)，管理与 Web 层(表现层)相关的 Bean，如控制器、视图解析器、拦截器等。

- HttpServletBean有一个final的init()**[Servlet的入口方法]**  其中会调用抽象方法initServletBean()

- FrameServlet实现了initServletBean(): **[生成Servlet WAC，设置parent和ServletContext]** 最后会调用initStrategies

  ```java
  ServletContext var10000 = this.getServletContext();
  String var10001 = this.getClass().getSimpleName();
  var10000.log("Initializing Spring " + var10001 + " '" + this.getServletName() + "'");
  //记录日志
  ```

  - 同样的，途中也会调用自己的initWAC方法：
    - 调用WACUtils工具类，获得自己所在的ServletContext的**Root WAC** 
    - 将自己的WAC转换成CWAC，如果存在RootWAC，则将其设置为自己的parent
    - 然后configureAndRefreshWAC(cwac)：设置ServletContext为sc，从其中ServletConfig中获取 `<init-param>` 参数的值

  ![image-20241023013013442](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20241023013013442.png)

  - 最后根据自己的ServletConfig获取到ServletContext，根据自己的名称设置自己的Servlet WAC在ServletContext中的Key

- DispatcherServlet实现了initStrategies [生成各个功能组件，异常处理器，视图处理，请求映射]

![mvc-context-hierarchy](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/mvc-context-hierarchy.png)

这两个context都是在ServletContext中，属于dispatcherServlet的上下文是servletWAC，找不到的话就去rootWAC中找

#### <span id="webappinit">替代web.xml，以Java形式配置ServletContext——WebApplicationInitializer</span>

![屏幕截图 2024-10-23 133904](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/屏幕截图 2024-10-23 133904.png)

##### ServletContainerInitializer

之前，web容器（Tomcat）会根据WEB-INF下的web.xml初始化ServletContext

Java EE Servlet 规范定义了这个接口，web容器（Tomcat）启动时根据这个初始化器做一些组件内的初始化工作。 

**SpringServletContainerInitializer** 是Spring 对其的实现，其onStartup方法会调用 **[WebApplicationInitializer](#webappinit)** 的onStartup(**ServletContext sc**)初始化Web应用

#### SpringMVC Web应用启动流程

- Tomcat 读取web.xml中 `<context-param>` `<listener>`  然后创建一个全局共享的ServletContext
- Tomcat 将`<context-param>` `<listener>`转化为键值对，存到ServletContext 
- Tomcat 加载Listener实例，实施监听，Listener必须实现<u>ServletContext</u>Listener接口（比如ContextLoaderListener）
- **Web项目继续启动中**，触发Listener中的contextInitialized(ServletContexEvent event)，根据ServletContext中 `<context-param>` 部分创建父容器，configClass是类的形式，configLocation是xml配置文件的形式
- 创建完父容器，如果有`<filter>`会创建filter，然后读取 `<servlet>` 用于注册DispatcherServlet（这块流程建议从init方法一步步往下看，流程还是很清晰的），因为DispatcherServlet实质是一个Servlet，所以会先执行它的init方法。这个init()方法在**HttpServletBean**这个类中实现，其主要工作是做一些初始化工作，将我们在web.xml中配置的参数设置到ServletContext的ServletConfig中，然后再触发**FrameworkServlet**的initServletBean()方法；
  - **FrameworkServlet**主要作用是初始化Spring子容器，设置其父容器，并将其放入ServletContext中；
  - **FrameworkServlet**在调用initServletBean()的过程中同时会触发**DispatcherServlet**的onRefresh()方法，这个方法会初始化Spring MVC的各个功能组件。比如异常处理器、视图处理器、请求映射处理等

[Spring MVC启动流程](https://www.cnblogs.com/54chensongxia/p/12522804.html) 

##### 100% code-based 

用Java类的形式配置ServletContext，有一些细微差异，Spring这边实现了ServletContainerInitializer接口，注册组件的工作就交给了WebApplicationInitializer：

先根据指定的rootWacConfig配置类（SpringConfig）创建出父容器，父容器作为参数进行Listener的有参构造，最后以<mark>add<mark>Listener的方式注册到ServletContext中。

## 拦截器

### Interceptor

![image-20241023193138396](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20241023193138396.png)

#### In Filter

![image-20241023193414214](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20241023193414214.png)

filter在一定是在访问servlet之前，interceptor只能在servlet中， <mark>before Controller<mark>

### 功能类

控制表现层：controller下新建interceptor包，新建一个Interceptor类 **extends HandlerInterceptor** 

注意preHandle返回值和@Component

![image-20241023210048371](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20241023210048371.png)

#### SpringMvcSupport

![image-20241023205518460](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20241023205518460.png)

addInterceptors 自动注入自定义拦截器

addPathPatterns 加的不是前缀，<mark>是严格的URL匹配<mark>，配/books就拦截对/books发的请求，/books/100就拦截不了

![image-20241023205715345](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20241023205715345.png)

<mark>preHandle<mark>，yourService，postHandle，afterCompletion 顺序

#### 示意图

![image-20241023210636045](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20241023210636045.png)

#### 简化开发

![image-20241023210528299](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20241023210528299.png)

已经和Spring接口绑定，侵入性强。

### 拦截方法参数配置

#### boolean preHandle(req,resp,<mark>handler<mark>)

req和resp是servlet的响应和请求，handler实际上是HandlerMethod，通过getMethod能拿到执行的业务方法的对象（反射）

#### void postHandle(req,resp,<mark>handler<mark>,modelAndView)

页面跳转相关。

#### void afterCompletion(req,resp,<mark>handler<mark>,exception)

能拿到原始业务方法执行过程中的异常

### 拦截链顺序

![image-20241023211911820](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20241023211911820.png)

拦截顺序，和注册顺序有关系

![image-20241023212439810](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20241023212439810.png)

如果某个pre返回false，post全部跳过，倒序执行，从最近一个pre返回true的拦截器开始执行afterCompletion

![image-20241023212527269](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20241023212527269.png)



[万字详解 GoF 23 种设计模式（多图、思维导图、模式对比），让你一文全面理解-CSDN博客](https://blog.csdn.net/penriver/article/details/118571991)

# Spring Boot

## 入门案例——Web项目

### 创建boot模块

[idea创建不了spring2.X版本，无法使用JDK8，最低支持JDK17 ， 如何用idea创建spring2.X版本，使用JDK8解决方案_spring3不支持jdk8-CSDN博客](https://blog.csdn.net/dream_ready/article/details/134639886)

SpringBoot2停止维护，SpringBoot3最低Java17

![image-20241025190302057](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20241025190302057.png)

### 写控制器类

把controller类写好

![image-20241025194037167](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20241025194037167.png)

### 启动app

![image-20241025194052846](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20241025194052846.png)

### 快速启动

#### 打包—jar

`package` 之前 `clean` 全部设置为UTF-8参数

#### 命令行启动 java -jar

![image-20241025194844475](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20241025194844475.png)

![image-20241025194925821](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20241025194925821.png)

jar执行要有入口类，boot打包需要插件才能生成可执行的入口类

## 简述boot

![image-20241025195226537](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20241025195226537.png)

### starter 起步依赖

### starter-parent 依赖管理

starter-parent：定义了无数jar包的版本管理和依赖管理，减少依赖冲突。只写GA 不写V

### dependencies-辅助功能

每一个dependency（以web包为例）把真正需要用到的jar包声明，去找parent要即可

spring-boot-starter-web: 

![image-20241025201404198](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20241025201404198.png)

spring-boot-dependencies: 

![image-20241025201443556](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20241025201443556.png)

![image-20241025201805886](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20241025201805886.png)

### 替换starter的某个依赖

依赖排除exclusion，换技术

![image-20241025202142653](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20241025202142653.png)

## 配置文件

### resources目录下配置文件加载优先级

![image-20241025204356892](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20241025204356892.png)

properties><mark>yml<mark>>yaml

### 自动提示功能消失—引入配置文件到boot模块中

![image-20241025204437530](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20241025204437530.png)

![image-20241025204503008](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20241025204503008.png)

debug>info>warn

### YAML—(YAML Ain't Markup Language)

#### YAML 简介

Jamel  Camel

![image-20241025204800590](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20241025204800590.png)

#### 语法规则

![image-20241025204945594](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20241025204945594.png)

```yaml
# 空格数量不限，只要前面格数一样就是同一级，不允许使用tab缩进
enterprise:
 name: John
 likes:
  - Java
  - Python
  - C++
```

![image-20241025205033642](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20241025205033642.png)

### 以Java方式读取yaml配置文件

#### 读取单个数据—定义成员变量@Value(${enterprise.subject)

自动赋值

![image-20241025205805383](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20241025205805383.png)

#### 读取全部数据—Environment对象

定义一个Environment，自动装配，将配置中的属性全部遍历:

`environment.getProperty("name")`

![image-20241025210024893](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20241025210024893.png)

#### **自定义对象封装指定数据**@Component@ConfigurationProperties(prefix = enterprise)

可以拿到需要的某个属性的信息(prefix)

在控制器中定义成员变量自动装配，常用

![image-20241025210710013](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20241025210946272.png)

##### 自定义对象封装数据警告

![image-20241025211036999](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20241025211036999.png)

### 多环境开发

#### 生产环境设定

**独立生产环境设定**：

`spring.profiles` **boot2**

`spring.config.activate.on-profile` **boot3**

`---` 三条横线分割配置



`spring.profiles.active` 设置激活的环境

![image-20241025214421983](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20241025214421983.png)

#### 带参数启动boot

##### 命令行参数临时修改配置内容

```sh
# 修改启动环境为test
java -jar boot_1.0_SNAPSHOT.jar --spring.profiles.active=test
# 修改服务器端口号为88
java -jar boot_1.0_SNAPSHOT.jar --server.port=88

```

##### 参数加载优先级

![image-20241025215406246](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20241025215406246.png)

#### maven&boot开发环境兼容—加载配置文件

maven和boot都设置了多环境，但是打包工作是maven负责，所以maven应该占主导

手动配置resources插件，覆盖parent设定

[maven-resources-plugin详解 - 红尘过客2022 - 博客园 (cnblogs.com)](https://www.cnblogs.com/hcgk/p/17600908.html)

##### `<resources>` (`<filtering>`)

![image-20241026013908079](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20241026013908079.png)

`<resources>`标签其实就是`maven-resources-plugin`的`<resources>`配置，主要用来配置资源目录的。普通项目没有`parent`，默认继承`父pom.xml`: 

![image-20241026024547372](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20241026024547372.png)

可以看到`resources`默认就是项目下的`src/main/resources`，但是没开过滤`filtering`，所以之前maven课程中，<u>在配置文件中引入占位符还得重写一遍resources标签。</u>



而boot模块默认继承的starter-parent默认的`resources`标签是开了过滤的，而且资源明确`包括application.yml`这种配置文件，所以即使子项目里不用手动复写`resources`也能匹配到占位符：

![image-20241026025346547](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20241026025346547.png)

##### `maven-resources-plugin` (`<useDefaultDelimeters>` )

`<useDefaultDelimeters>` 支持使用`${}或@`过滤资源 

![image-20241026021624316](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20241026021624316.png)

boot的starter-parent默认对`resources-plugin`的配置也做了自定义更改（主要就是`<useDefaultDelimeters>` = false）而上文提到的父pom没有，`<useDefaultDelimeters>`默认就是true

![image-20241026025510839](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20241026025510839.png)

#### 定义多环境 `<profiles>`

![image-20241026005328800](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20241026005328800.png)



#### 配置属性替换占位符

##### `@`

boot项目的parent为了防止spring占位符被扩展，所以只允许`@`为占位符，不解析`${}`。如果已经继承starter-parent，直接在配置文件中@xxx@即可。 [parent不是starter-parent的解决办法](https://docs.springjava.cn/spring-boot/how-to/properties-and-configuration.html#howto.properties-and-configuration.expand-properties.maven)

![image-20241026021230152](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20241026021230152.png)

##### `${}`

非要用${}，可以使用如下方法覆盖配置：

![image-20241026081744618](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20241026081744618.png)

或者启用插件的`<useDefaultDelimeters>` = true

![image-20241026005308461](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20241026005308461-1729903011039-5.png)

![image-20241026005348094](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20241026005348094-1729902750823-1-1729902803907-3.png)

#### 多配置文件加载优先级



##### 包外配置

假设JAR包位于file目录下，`file/config/application.yml` > `file/application.yml`

##### 包内配置

`resources/config/application.yml` > `resources/application.yml`

#### Java项目目录结构

<img src="https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20241026140333629.png" alt="image-20241026140333629" style="zoom: 67%;" />

- src/main 就是编译以后的classpath(classes)，java是java源代码，resources资源文件，编译完打包都在同一个classes目录下。
- src/test 是test-classes，属于测试文件，默认不会参与打包。
- 依赖放在**包内**和classes并列的lib目录

- 对于maven webapp骨架，main还有webapp目录，其中WEB-INF文件夹存放web.xml，打包之后web.xml classes lib并列放在WEB-INF中
  - webapp也可存放静态资源，打包之后在JAR包中的第一级



#### JAR包内部结构 

[JAR (文件格式) - 维基百科，自由的百科全书 (wikipedia.org)](https://zh.wikipedia.org/wiki/JAR_(文件格式))

META-INF 整个项目的元数据，MANIFEST.MF 包含执行时的入口类等信息

BOOT-INF boot项目的jar包中 classes+lib

WEB-INF web项目war包中 classes+lib+web.xml

![image-20241026143753352](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20241026143753352.png)

![image-20241026143816570](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20241026143816570.png)

##### APK内部结构

APK作为JAR包的变种，也具有相似的结构：

![image-20241026144246642](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20241026144246642.png)

## 与其他框架整合

### Spring Boot X JUnit 

![image-20241026151902777](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20241026151902777.png)

![image-20241026152538674](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20241026152538674.png)

#### 测试类注解@SpringBootTest

![image-20241026152650043](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20241026152650043.png)

SpringBoot启动类：@SpringBootApplication有加载bean的功能，会扫描当前包同层以及子包中所有的bean，加载bean（包含配置类）

SpringBootTest会自动扫描SpringBootApplication，测试类不在启动类所在包/子包中，需要指定启动类的class文件

### 基于SpringBoot实现SSM整合

#### 整合MyBatis案例

##### 启动依赖-MyBatis,MySQL

##### pojo dao @Mapper @MapperScan

mybatis自动代理注解开发返回的对象就是实体类，所以实体类不用配置，

mybatis注解开发中@Mapper注解取代了bookMapper.xml，对mybatis声明这是一个mapper。

spring-mybatis整合中，mybatis生成mapper的代理对象会以FactoryBean的形式交给Spring容器管理，要让mybatis知道mapper在哪里，就要加@Mapper注解

spring-mybatis整合中，不加@Mapper注解，要么配置mapperScannerConfigurer，要么加@MapperScan扫mapper包。

##### application.yml 配置数据源

![image-20241026163010546](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20241026163010546.png)



#### SSM项目迁移到Spring Boot

TODO 注释前面加TODO 可以有事项清单

##### 配置类全部删除

##### Dao加@Mapper

##### Controller Service不变

##### application.yml 配置端口和数据源

##### 静态资源放到resources/static

###### 静态资源的重定向(JS脚本)

访问一个web资源，如果直接访问 `localhost:port` 一般会请求一个主页index.html，为了能直接从地址访问资源，创建一个index.html，添加一个跳转的js脚本

```html
<script>
	document.location.href="pages/books.html";
</script>
```

## Spring Boot 自动装配

[SpringBoot-自动装配：自动装配框架内部的-Bean](https://scatteredream.github.io/2025/02/03/rpc-interpretation/#SpringBoot-自动装配：自动装配框架内部的-Bean) 

[自定义 starter | scatteredream's blog](https://scatteredream.github.io/2024/10/01/spring-boot-starter/) 

## @SpringBootApplication

`@SpringBootApplication` 是 Spring Boot 的核心注解，它是一个**组合注解**，整合了以下三个关键注解的功能：

### 核心作用
- **`@Configuration`**  
  标记当前类为**配置类**，允许通过 `@Bean` 定义 Spring 容器中的组件。
- **`@EnableAutoConfiguration`**  
  启用 Spring Boot 的**自动配置机制**，根据项目依赖（如 JDBC、Web、Redis 等）自动配置 Spring 应用。
- **`@ComponentScan`**  
  自动扫描**当前包及其子包**下的组件（如 `@Controller`、`@Service`、`@Repository` 等），无需手动注册。

### 典型用法
```java
@SpringBootApplication
public class MyApp {
    public static void main(String[] args) {
        SpringApplication.run(MyApp.class, args); // 启动 Spring Boot 应用
    }
}
```
- 通常放在项目的**主类**（含 `main` 方法）上。
- 启动后会自动初始化 Spring 容器、加载配置、启动内嵌服务器（如 Tomcat）。

### 配置
#### 排除特定自动配置
```java
@SpringBootApplication(exclude = {DataSourceAutoConfiguration.class})
```
- 例如：项目未使用数据库时，可排除数据源自动配置。

#### 自定义扫描路径
```java
@SpringBootApplication(scanBasePackages = "com.example")
```
- 默认扫描主类所在包，如需扫描其他包，可通过 `scanBasePackages` 指定
- 因此，默认情况下，SpringBootApplication修饰的类应该在根目录，确保所有类都能被扫到

### 常见问题
- **Q：为什么我的 `@Component` 组件没被扫描到？**  
  A：确保组件位于主类的**同级或子包**下，或通过 `scanBasePackages` 显式指定路径。

- **Q：如何查看生效的自动配置？**  
  A：启动时添加 `--debug` 参数，日志会输出所有自动配置的评估结果。

### 对比传统 Spring
| 特性     | Spring Boot (`@SpringBootApplication`) | 传统 Spring                 |
| -------- | -------------------------------------- | --------------------------- |
| 配置方式 | 自动配置 + 默认约定                    | 手动 XML 或 Java Config     |
| 组件扫描 | 自动（默认包扫描）                     | 需显式配置 `@ComponentScan` |
| 依赖管理 | 通过 Starter 简化                      | 手动管理依赖版本            |
