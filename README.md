# Spring-AOP学习总结
### 什么是AOP
AOP:面向切面编程，通过预编译方式和运行期动态代理实现程序功能的统一维护的一种技术<br>
功能例如：日志记录，性能统计，安全控制，事务处理，异常处理等等
### AOP实现方式
* 预编译<br>
-AspectJ
* 运行期动态代理<br>
-SpringAOP,JbossAOP
### 几个相关的概念
名称 | 说明 |
--------- | --------|
切面（Aspect）  | 一个关注点的模块话，这个关注点可能会横切多个对象 |
连接点（Joinpoint）  | 程序执行过程中某个特定的点 |
通知（Advice）  | 在切面的某个特定的连接点上执行的动作 |
切入点（Pointcut）  | 匹配连接点的断言，在AOP中通知和一个切入点表达式关联 |
引入（Introduction）  | 在不修改类代码的前提下，为类添加新的方法和属性 |
目标对象（Target Object）  | 被一个或者多个切面所通知的对象 |
AOP代理（AOP Proxy）  | Aop框架创建的对象，用来实现切面契约 |
织入（Weaving）  | 把切面连接到其他的应用程序类型或者对象上，并且创建一个被通知的对象，分为<br>编译时织入，类加载时织入，执行时织入 |
### AOP术语通俗解释
* 通知、增强处理（Advice） 就是切面里面所有定义的方法
* 连接点（JoinPoint） 这个就更好解释了，就是spring允许你是通知（Advice）的地方，那可就真多了，基本每个方法的前、后（两者都有也行），或抛出异常是时都可以是连接点，spring只支持方法连接点。
* 切入点（Pointcut） 上面说的连接点的基础上，来定义切入点，你的一个类里，有15个方法，那就有十几个连接点了对吧，但是你并不想在所有方法附件都使用通知（使用叫织入，下面再说），你只是想让其中几个，在调用这几个方法之前、之后或者抛出异常时干点什么，那么就用切入点来定义这几个方法，让切点来筛选连接点，选中那几个你想要的方法。
* 切面（Aspect） 切面是通知和切入点的结合。现在发现了吧，没连接点什么事，链接点就是为了让你好理解切点搞出来的，明白这个概念就行了。通知说明了干什么和什么时候干（什么时候通过方法名中的befor，after，around等就能知道），二切入点说明了在哪干（指定到底是哪个方法），这就是一个完整的切面定义。
* 引入（introduction） 允许我们向现有的类添加新方法属性。这不就是把切面（也就是新方法属性：通知定义的）用到目标类中吗
* 目标（target） 引入中所提到的目标类，也就是要被通知的对象，也就是真正的业务逻辑，他可以在毫不知情的情况下，被咱们织入切面。二自己专注于业务本身的逻辑。
* 代理（proxy） 怎么实现整套AOP机制的，都是通过代理。
* 织入（weaving） 把切面应用到目标对象来创建新的代理对象的过程。有三种方式，spring采用的是运行时。
* 目标对象 – 项目原始的Java组件。
* AOP代理  – 由AOP框架生成java对象。
* AOP代理方法 = advice +　目标对象的方法。
### 代码演示
首先配置beans.xml<br>
```Java
<?xml version="1.0" encoding="UTF-8"?>
<beans
	xmlns="http://www.springframework.org/schema/beans"
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xmlns:p="http://www.springframework.org/schema/p"
	xmlns:context="http://www.springframework.org/schema/context"
	xmlns:aop="http://www.springframework.org/schema/aop"
	xsi:schemaLocation="http://www.springframework.org/schema/beans 
	                              http://www.springframework.org/schema/beans/spring-beans-3.0.xsd
	                              http://www.springframework.org/schema/context
	                              http://www.springframework.org/schema/context/spring-context-3.0.xsd
	                              http://www.springframework.org/schema/aop
	                              http://www.springframework.org/schema/aop/spring-aop-3.0.xsd
	                              "
	default-autowire="no">
	<bean id="BaseDao" class="cn.dao.impl.BaseDaoImpl" scope="prototype">
	</bean>	
	<bean id="ServiceDao" class="cn.service.impl.ServiceDaoImpl">
	</bean> 
	<bean id="aspectDemo" class="cn.aspect.AspectDemo">
	</bean>  
	<aop:config>
		<aop:aspect id="aspectDemoAop" ref="aspectDemo">
			<aop:pointcut expression="execution(* cn.service.impl.ServiceDaoImpl.*(..))" id="aspectPointCut"/>
			<aop:before method="before" pointcut-ref="aspectPointCut"/>
			<aop:after-returning method="afterReturning" pointcut-ref="aspectPointCut"/>
			<aop:after-throwing method="afterThrowing" pointcut-ref="aspectPointCut"/>
			<aop:after method="after" pointcut-ref="aspectPointCut"/>
			<aop:around method="around" pointcut-ref="aspectPointCut"/>
		</aop:aspect>
	</aop:config>
</beans>
```
其中<aop:aspect id="aspectDemoAop" ref="aspectDemo">	</aop:aspect>是对切面的定义，要写在<aop:config>属性中，pointcut是切入点的意思
在expression写触发时会发生的方法事件，* cn.service.impl.ServiceDaoImpl.*(..)这一行代表的意思是在ServiceDaoImpl类下所有的方法执行都会触发。
<br>
<aop:before method="before" pointcut-ref="aspectPointCut"/><br>
<aop:after-returning method="afterReturning" pointcut-ref="aspectPointCut"/><br>
<aop:after-throwing method="afterThrowing" pointcut-ref="aspectPointCut"/><br>
<aop:after method="after" pointcut-ref="aspectPointCut"/><br>
<aop:around method="around" pointcut-ref="aspectPointCut"/><br>
这五行都是通知的不同类型分别是方法事件触发前执行，方法事件出发后返回执行，方法事件抛出异常后执行，方法结束后执行，方法环绕执行<br>
```java
package cn.aspect;

import org.aspectj.lang.ProceedingJoinPoint;

public class AspectDemo {
	public void before(){
		System.out.println("通知前已响应");
	}
	public void afterReturning(){
		System.out.println("返回后响应");
	}
	public void afterThrowing(){
		System.out.println("抛出异常后响应");
	}
	public void after(){
		System.out.println("方法结束后响应");
	}
	public Object around(ProceedingJoinPoint pjp){
		Object obj = null;
		try {
			System.out.println("1");
			obj = pjp.proceed();
			System.out.println("2");
		} catch (Throwable e) {
			e.printStackTrace();
		}
		return obj;
	}
}

```
运行结果：
```java
log4j:WARN No appenders could be found for logger (org.springframework.context.support.ClassPathXmlApplicationContext).
log4j:WARN Please initialize the log4j system properly.
通知前已响应
1
逻辑层判断
返回后响应
方法结束后响应
2
```
### after-returing和after-throwing和after
after-returing和after-throwing两者只能其中一个执行，若抛出异常则after-throwing，否则就after-returing，after则在两者之后执行，且与前两者是否执行无关。<br>
round的话必须在方法里写ProceedingJoinPoint参数才能执行
### execution表达式例子如下：
任意公共方法的执行：<br>
　　　　execution(public * *(..))<br>
　　任何一个以“set”开始的方法的执行：<br>
　　　　execution(* set*(..))<br>
　　AccountService 接口的任意方法的执行：<br>
　　　　execution(* com.xyz.service.AccountService.*(..))<br>
　　定义在service包里的任意方法的执行：<br>
　　　　execution(* com.xyz.service.*.*(..))<br>
　　定义在service包和所有子包里的任意类的任意方法的执行：<br>
　　　　execution(* com.xyz.service..*.*(..))<br>
　　定义在pointcutexp包和所有子包里的JoinPointObjP2类的任意方法的执行：<br>
　　　　execution(* com.test.spring.aop.pointcutexp..JoinPointObjP2.*(..))")<br>
### Spring声明式事务管理
声明式事务管理是建立在AOP的基础之上的，其本质是对方法前后进行拦截，然后在目标方法开始之前创建或者加入一个事务，在执行完目标方法之后根据执行情况提交或者回滚事务。声明式事务最大的优点就是不需要通过编程的方式管理事务，这样就不需要在业务逻辑代码中掺杂事务管理的代码，只需在配置文件中做相关的事务规则声明(或通过基于@Transactional注解的方式)，便可以将事务规则应用到业务逻辑中。<br>
Spring配置文件加入如下配置
```java
<!-- Spring事务管理 -->
	<bean id="txDao" class="cn.spring.dao.impl.TxDaoImpl">
		<property name="jdbcTemplate" ref="jdbcTemplate"></property>
	</bean>
	<bean id="dataSource" class="org.springframework.jdbc.datasource.DriverManagerDataSource">  
        <property name="driverClassName" value="com.mysql.jdbc.Driver" />  
        <property name="url" value="jdbc:mysql://localhost:3306/test?characterEncoding=utf-8" />  
        <property name="username" value="root" />  
        <property name="password" value="123456" />  
        </bean>  
	<bean name="transactionManager" class="org.springframework.jdbc.datasource.DataSourceTransactionManager">  
    	<property name="dataSource" ref="dataSource" />  
	</bean> 
			
	<tx:advice id="advice" transaction-manager="transactionManager">  
	    <tx:attributes>  
	        <tx:method name="*"  rollback-for="Exception"/>   
	    </tx:attributes>  
	</tx:advice>    
	<aop:config>  
	    <aop:pointcut id="testService" expression="execution (* cn.spring.dao.impl.TxDaoImpl.*(..))"/>  
	    <aop:advisor advice-ref="advice" pointcut-ref="testService"/>  
	</aop:config>  
```
TxDaoImpl：
```java
package cn.spring.dao.impl;

import org.springframework.jdbc.core.JdbcTemplate;


public class TxDaoImpl {

	private JdbcTemplate jdbcTemplate;
	public JdbcTemplate getJdbcTemplate() {  
        return jdbcTemplate;  
    }  
  
    public void setJdbcTemplate(JdbcTemplate jdbcTemplate) {  
        this.jdbcTemplate = jdbcTemplate;  
    }  
	
	public void execute() throws Exception{
		
			String sql = "INSERT INTO aaa(number) VALUES(?)"; 
		    for(int i=1;i<=20;i++){  
		    jdbcTemplate.update(sql,i);  
		    if(i==8) throw new Exception();  
		    }  
		 
		
	}
}
```
当执行到第八条插入语句时显然会抛出异常，此时事务发生回滚，数据库中将不会有任何新添加的数据



# Spring-IOC小结
### Bean容器初始化的三种方式：
* 本地文件<br>
* ClassPath<br>
* Web应用中依赖servlet或Listener<br>
### Spring注入的两种方式：
* 设值注入：
```java
<bean id="injectionService" class="com.imooc.ioc.injection.service.InjectionServiceImpl"> 
      <property name="injectionDAO" ref="injectionDAO"></property> -->
</bean>
 <bean id="injectionDAO" class="com.imooc.ioc.injection.dao.InjectionDAOImpl"></bean>
</beans>
```
* 构造注入：
```java
<bean id="injectionService" class="com.imooc.ioc.injection.service.InjectionServiceImpl">
        <constructor-arg name="injectionDAO" ref="injectionDAO"></constructor-arg>
 </bean>
  <bean id="injectionDAO" class="com.imooc.ioc.injection.dao.InjectionDAOImpl"></bean>
</beans>

```
### Bean的作用域：
1、singleton:当一个bean的作用域为singleton, 那么Spring IoC容器中只会存在一个共享的bean实例，并且所有对bean的请求，只要id与该bean定义相匹配，则只会返回bean的同一实例。
注意：Singleton作用域是Spring中的缺省作用域。要在XML中将bean定义成singleton，可以这样配置： 
<bean id="empServiceImpl" class="cn.csdn.service.EmpServiceImpl" scope="singleton"><br>
2、prototype：一个bean定义对应多个对象实例。Prototype作用域的bean会导致在每次对该bean请求（将其注入到另一个bean中，或者以程序的方式调用容器的getBean()方法）时都会创建一个新的bean实例。根据经验，对有状态的bean应该使用prototype作用域，而对无状态的bean则应该使用singleton作用域。<br>
3、request：在一次HTTP请求中，一个bean定义对应一个实例；即每次HTTP请求将会有各自的bean实例， 它们依据某个bean定义创建而成。该作用域仅在基于web的Spring ApplicationContext情形下有效。
考虑下面bean定义：
<bean id="loginAction" class=cn.csdn.LoginAction" scope="request"/>
针对每次HTTP请求，Spring容器会根据loginAction bean定义创建一个全新的LoginAction bean实例， 且该loginAction bean实例仅在当前HTTP request内有效，因此可以根据需要放心的更改所建实例的内部状态， 而其他请求中根据loginAction bean定义创建的实例，将不会看到这些特定于某个请求的状态变化。 当处理请求结束，request作用域的bean实例将被销毁。<br>
4、session：在一个HTTP Session中，一个bean定义对应一个实例。该作用域仅在基于web的Spring ApplicationContext情形下有效。
考虑下面bean定义：
<bean id="userPreferences" class="com.foo.UserPreferences" scope="session"/>
针对某个HTTP Session，Spring容器会根据userPreferences bean定义创建一个全新的userPreferences bean实例， 且该userPreferences bean仅在当前HTTP Session内有效。 与request作用域一样，你可以根据需要放心的更改所创建实例的内部状态，而别的HTTP Session中根据userPreferences创建的实例， 将不会看到这些特定于某个HTTP Session的状态变化。 当HTTP Session最终被废弃的时候，在该HTTP Session作用域内的bean也会被废弃掉。<br>
5、global session：在一个全局的HTTP Session中，一个bean定义对应一个实例。典型情况下，仅在使用portlet context的时候有效。该作用域仅在基于web的Spring ApplicationContext情形下有效。<br>
### Bean的生命周期
* 定义
* 初始化
* 使用
* 销毁<br>
Bean初始化/销毁的三种方法：<br>
（1）通过实现 InitializingBean/DisposableBean 接口来定制初始化之后/销毁之前的操作方法；要覆盖相应的方法

（2）通过 <bean> 元素的 init-method/destroy-method属性指定初始化之后 /销毁之前调用的操作方法；

（3）还有一种是
```java
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans.xsd" 
      default-init-method="defautInit" default-destroy-method="defaultDestroy">
```
这三种方法的执行顺序是（1）>（2）但（1）或者（2）实现的时候（3）不会实现，而且（3）可以有定义而没方法，但（2）只要定义了方法名就一定要在相应的类中写方法
### Spring容器为什么要设置单例：
理由很简单，一方面spring容器在初始化时就实例化了bean类，提高了效率，另一方面也能大大的减少内存开销，但仍然存在一些问题：<br>
比如在Controller类中如果设置了非final类型修饰全局变量的话，可能会出现线程安全问题，这个时候通过注解@Scope("prototype")来把Controller类设置为原型模式。

### Spring自动装配
No:不做任何操作<br>
ByName：根据属性名自动装配。此选项将检查容器并根据名字查找与属性完全一致的bean，并将其与属性自动装配。<br>
ByType：如果容器中存在一个预指定属性类型相同的bean，那么将于该属性自动装配，并不考虑ID，如果存在多个该类型的bean，抛出异常，找不到相匹配的则什么事都不会发生<br>
constructor：与ByType方式类似，不同之处在于她应用于构造器参数，如果容器中没有找到与构造器参数类型一致的bean，那么抛出异常。<br>
即不必在bean属性下写property属性或者constructor-arg属性，只需在beans xmlns定义default-autowire即可。<br>
注意byName和byType都是通过set方法设值注入的，constructor通过构造器注入，记住要写出原本类的默认构造方法

# Spring事务
## 什么是事务？
首先我们要了解一下并发控制，并发控制是指多个用户同时更新运行时，用于保护数据库完整性的各种技术。简单的讲是为了保证一个用户的操作不会对另外一个用户的操作造成不利的影响。而事务，就是并发控制的基本单位。是用户定义的一个操作序列。这些操作要么全做，要么全不做，是一个不可分割的工作单位。
事务通常以begin transaction开始，以commit和rollback结束。
事务的特性（ACID）
A.	原子性（Atonicity）
要么全做，要么不做
B.	一致性（Consistency）
事务的执行结果要使数据库从一个一致性状态变成另一个一致性状态
C.	隔离性（Isolation）
一个事务的执行不能被其他事务干扰
D.	持续性/永久性（Durability）
一个事务一旦提交，他对数据库中数据的改变应该是永久性的
事务并发引起的问题
1.	脏读
在事务未commit前进行查询
2.	不可重复读
有两次查询，事务B有一次修改操作，顺序是A第一次查询-->B修改，comiitB第二次查询发现数据不一样
3.	幻读
事务A有两次查询数据量（聚合函数），事务B有一次添加或删除操作，顺序是A第一次查询-->B修改，comiitB第二次查询发现数据不一样
4.	第一类丢失更新(lost update)： 在完全未隔离事务的情况下，两个事务更新同一条数据资源，某一事物异常终止，回滚造成第一个完成的更新也同时丢失。
5.	第二类丢失更新(second lost updates)：是不可重复读的特殊情况，如果两个事务都读取同一行，然后两个都进行写操作，并提交，第一个事务所做的改变就会丢失。

## 事务的隔离级别
Mysql数据库给我们提供的四种隔离级别：
1.	Serializable（串行化）：可避免脏读，不可重复读，幻读的发生。但是性能极差，一般不会这么用。
2.	Repeatable read（可重复读）：可以避免脏读，不可重复读的发生。（mysql默认）
3.	Read commited（读已经提交）：可避免脏读的发生
4.	Read uncommitted（读未提交）：最低级别，任何情况都无法保证。


## 事务的传播行为
我的感觉是：一个被定义为事务的方法A在执行的过程中，去调用了另一个被定义为事务的方法B，此时方法B的事务是该新建，还是嵌入方法A事务，这就是事务传播行为去定义的。
	Propagation_required: 如果当前存在事务，则加入该事务；如果当前没有事务，则创建一个新的事务。这是默认值。<br>
	Propagation_requires_new:创建一个新的事务，如果当前存在事务，则将其高高挂起<br>
	Propagation_supports:如果当前存在事务，则加入该事务，如果当前没有事务，则以非事务的方式继续运行<br>
	Propagation_not_supported:以非事务方式运行，如果当前存在事务，则把事务高高挂起。<br>
	Propagation_never:以非事务方式运行，如果当前存在事务，抛异常<br>
	Propagation_mandatory:如果当前存在事务，则加入该事务，如果当前没有事务，抛异常<br>
	Propagation_nested:如果当前存在事务，则创建一个事务作为当前事务的嵌套事务来运行；如果当前没有事务，则该取值等价于TransactionDefinition.PROPAGATION_REQUIRED。<br>

疑难杂点：
	Required事务A和事务B只要有一处发生异常都会回滚<br>
	Requires_new：假设事务A调用事务B，事务B抛出异常，那么事务B肯定回滚，但是如果此时事务A去捕获这个异常，那么事务A还是可以正常commit的<br>
	Nested：Nested的事务和他的父事务是相依的，他的提交是要等和他的父事务一块提交的。也就是说，如果父事务最后回滚，他也要回滚的。
## Spring事务
Spring事务解决了全局事务和本地事务的缺陷，支持编程式事务管理和声明式事务管理。
声明式事务管理又分为两大部分：
1.	基于xml配置的事务管理
2.	注解

第一点昨天已经做了一个demo，今天做第二点有关的demo
先讲一些有关注解的基本配置：
@Component，@Service，@Controller功能相同，字面含义不同，方便开发者了解这是哪一个类，都是在类名上面添加，表明这个类的实例化交给Spring容器去托管

@Autowired和@Resource
这两个意思都是在某一个类中注入其他类的实例，一般都写在类属性的前面，其中@Autowired是用byType自动注入，@Resource默认是用byName，也可以用byType


@PostConstruct和@PreDestroy	，这两个只能应用于方法上，标注了 @PostConstruct 注释的方法将在类实例化后调用，而标注了 @PreDestroy 的方法将在类销毁之前调用。

@Scope指定bean作用域范围
@Transactional()事务注解
一些比较常用的属性：
1.	rollbackFor：设置哪些异常会执行回滚

2.	noRollbackFor:设置抛出哪些异常不会执行回滚

3.	timeout：事务超时间设置

4.	isolation：可选的事务隔离级别设置：默认为数据源的默认隔离级别

5.	propagation：可选的事务传播行为设置：默认为required

6.	readOnly：读写或者只读事务，默认读写

如果全部都是读取操作的话，可以设置为只读事务，这是一个优化的提示，在一些情况下，一些事务策略能够起到显著的最优化效果，例如在使用Object/Relational映射工具（如：Hibernate或TopLink）时避免dirty checking（试图“刷新”）。





