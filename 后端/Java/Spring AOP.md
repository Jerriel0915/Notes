---
tags:
  - 后端
  - Java
  - 面试
  - SpringBoot
  - Spring
---
## 1. 为什么需要AOP
在我们的项目系统中必然会存在一些组件功能，比如日志功能，权限验证等，这些功能的特点是广泛的分布于各个系统功能内，比如关键的数据库操作需要日志记录，而处理客户端的很多业务则需要验证用户的操作权限...这种分布式的范围会使得大量的重复编码和增加维护成本，而且很多时候一个系统功能就应该干好一件事，而不是额外考虑是否需要亲自记录日志、验证用户权限。就算将这些组件抽象为组件，大量重复的调用也会减少代码的美观度与整洁度。

[[Bean、IoC 与 DI]]
如果说DI能够让各组件间保持松散耦合，那么**面向切片（aspect-oriented programming）** 的编程方法则允许我们将各种组件的功能单独剥离出来，甚至在不改动源代码的基础上增加新的功能组件。

## 2. AOP术语解释
为了通俗地解释一些术语，可以使用一个贴切的例子来说明：

炎炎夏日，家家户户都开启了空调。为了记录每家的用电量，电力公司为一个小镇上的所有门户都安装了一个电表，每个月都会有人来查看电表，上报这个月的电费。
![AOP示意图](https://pic-1371809842.cos.ap-chengdu.myqcloud.com/PicGo/20251021170602954.webp?imageSlim)

### 2.1 通知（Advice）
当抄表员出现在我们家门口时，他们要登记用电量并回去向电力公司报告。显然，他们必须有一份需要抄表的住户清单，他们所汇报的信息也很重要，但记录用电量才是抄表员的主要工作。

类似地，切面也有目标——它必须要完成的工作。在AOP术语中，切面的工作被称为通知。

Spring切面可以应用5种类型的通知：
- 前置通知（`@Before`）：在目标方法被调用之前调用通知功能；
- 后置通知（`@After`）：在目标方法完成之后调用通知，此时不会关心方法的输出是什么；
- 返回通知（`@AfterReturning`）：在目标方法成功执行之后调用通知；
- 异常通知（`@AfterThrowing`）：在目标方法抛出异常后调用通知；
- 环绕通知（`@Around`）：通知包裹了被通知的方法，在被通知的方法调用之前和调用之后执行自定义的行为。

### 2.2 连接点（Join Point）
电力公司为多个住户提供服务，甚至可能是整个城市。每家都有一个电表，这些电表上的数字都需要读取，因此每家都是抄表员的潜在目标。

同样，我们的应用可能也有数以千计的时机应用通知。这些时机被称为连接点。连接点是在应用执行过程中能够插入切面的一个点。这个点可以是调用方法时、抛出异常时、甚至修改一个字段时。切面代码可以利用这些点插入到应用的正常流程之中，并添加新的行为。

### 2.3 切点（Pointcut）
如果让一位抄表员访问电力公司所服务的所有住户，那肯定是不现实的。实际上，电力公司为每一个抄表员都分别指定某一块区域的住户。类似地，一个切面并不需要通知应用的所有连接点。切点有助于缩小切面所通知的连接点的范围。

如果说通知定义了切面的“什么”和“何时”的话，那么切点就定义了“何处”。

### 2.4 切面（Aspect）
切面是通知和切点的结合。通知和切点共同定义了切面的全部内容 ——它是什么，在何时和何处完成其功能。

### 2.5 织入（Weaving）
织入是把切面应用到目标对象并创建新的代理对象的过程。切面在指定的连接点被织入到目标对象中。在目标对象的生命周期里有多个点可以进行织入： 
- 编译期：切面在目标类编译时被织入。这种方式需要特殊的编译器。AspectJ的织入编译器就是以这种方式织入切面的。
- 类加载期：切面在目标类加载到JVM时被织入。这种方式需要特殊的类加载器（ClassLoader），它可以在目标类被引入应用之前增强该目标类的字节码。AspectJ 5的加载时织入（load-time weaving，LTW）就支持以这种方式织入切面。
- 运行期：切面在应用运行的某个时刻被织入。一般情况下，在织入切面时，AOP容器会为目标对象动态地创建一个**代理对象**。Spring AOP就是以这种方式织入切面的。

 Spring AOP 基于代理，仅支持方法级的连接点，无法像 AspectJ 那样拦截字段、构造器等。

## 3. Spring AOP工作流程
1. Spring容器启动；
2. 读取所有**切面**配置中的切入点。如果只定义了切点，但没有对应的通知与之绑定成切面，那么这个切点就不会被Spring读取；
3. 初始化bean，判定bean对应的类中的方法是否匹配到切入点。如果匹配成功，则会创建原始对象的**代理**对象。匹配失败则创建普通bean；
4. 当切面对应的方法被调用时，运行的是代理对象的增强方法。

## 4. 通过切点选择连接点
### 4.1 切点指示器
在Spring AOP中，使用AspectJ的切点表达式语言来定义切点。下面给出具体的指示器用法：
1. args() —— 按**方法参数类型**匹配
```java
// 匹配任何只有一个、且参数类型为 Serializable 的方法
@Pointcut("args(java.io.Serializable)")
public void serializableArgMethod(){}
```
2. @args() —— 按**参数上的注解**匹配
```java
// 匹配第一个参数上带有 @Sensitive 注解的方法
@Pointcut("@args(com.x.y.aop.Sensitive,..)")
public void sensitiveFirstArg(){}

// 自定义参数注解
@Retention(RUNTIME)
public @interface Sensitive {}
```

3. execution() —— 最细粒度，按**方法签名**匹配
```java
// 匹配 com.x.y.service 包及其子包下所有 public 方法，返回类型任意
@Pointcut("execution(public * com.x.y.service..*.*(..))")
public void servicePublicMethod(){}
```

4. this() —— 按**代理对象类型**匹配（实现接口也可）
```java
// 匹配代理对象实现了 UserService 接口的所有方法
@Pointcut("this(com.x.y.service.UserService)")
public void proxyImplementsUserService(){}
```

5. target() —— 按**目标对象类型**匹配（实际类）
```java
// 匹配目标对象就是 UserServiceImpl 类本身的方法
@Pointcut("target(com.x.y.service.UserServiceImpl)")
public void targetIsUserServiceImpl(){}
```

6. @target() —— 按**目标对象类上的注解**匹配
```java
// 匹配目标对象所属类上带 @Repository 注解的所有方法
@Pointcut("@target(org.springframework.stereotype.Repository)")
public void repositoryBean(){}
```

7. within() —— 按 **包/类名**匹配
```java
// 匹配 dao 包下所有类中的所有方法
@Pointcut("within(com.x.y.dao.*)")
public void inDaoPackage(){}
```

8. @within() —— 按**类上的注解**匹配（Spring AOP 只到方法级，但类必须带注解）
```java
// 匹配所有类上带 @Loggable 注解的方法
@Pointcut("@within(com.x.y.aop.Loggable)")
public void loggableType(){}

// 自定义类级注解
@Retention(RUNTIME)
public @interface Loggable {}
```

9. @annotation() —— 按**方法上的注解**匹配（最常用）
```java
// 匹配所有带 @Metric 的方法
@Pointcut("@annotation(com.x.y.aop.Metric)")
public void metricMethod(){}

// 自定义方法注解
@Retention(RUNTIME)
public @interface Metric {}
```

组合示例（把多个指示器串起来）  
```java
// 仅拦截 service 包下、public 方法、且参数第一个是 @Sensitive 注解、方法本身带 @Metric
@Pointcut("within(com.x.y.service..*) && execution(public * *(..)) && @args(com.x.y.aop.Sensitive,..) && @annotation(com.x.y.aop.Metric)")
public void combo(){}
```
虽然有很多指示器，但只有`execution` 是最常用、最细粒度的匹配指示器，其他指示器如`@annotation`、`within`、`args`等通常与 `execution` 组合使用，以缩小匹配范围。

### 4.2 编写切点
现在有一个接口：
```Java
package Concert;  
  
public interface Performance {  
    void perform();  
}
```
那么对应的切点表达式可以这样写：
```Java
@Pointcut("execution(* Concert.Performance.perform())")
```
这表示当`perform()`方法触发时的通知。

![切点表达式](https://pic-1371809842.cos.ap-chengdu.myqcloud.com/PicGo/20251021232825612.webp?imageSlim)

## 5. 使用注解创建切面
### 5.1 切面定义
一个基本的切面大致就像这样：
```Java
package Concert.Aspect;  
  
import org.aspectj.lang.annotation.*;  
import org.springframework.stereotype.Component;  
  
@Aspect  
@Component  
public class Audience {  
    @Pointcut("execution(* Concert.Performance.perform())")  
    private void performPt() {};  
  
    @Before("performPt()")  
    public void silence() {  
        System.out.println("Audience start to silence");  
    }  
  
    @Before("performPt()")  
    public void takeSeats() {  
        System.out.println("Audience start to take seats");  
    }  
  
    @AfterReturning("performPt()")  
    public void applause() {  
        System.out.println("Audience start to applause");  
    }  
  
    @AfterThrowing("performPt()")  
    public void beSad() {  
        System.out.println("Audience start to be sad");  
    }  
}
```
但即使使用了AspectJ注解，Spring并不会将这个bean视为切面，不会有任何切点被解析。
如果使用JavaConfig作为配置，可以在配置类的类级别上通过使用`@EnableAspectJAutoProxy`注解启用自动代理功能。
```Java
@Configuration  
@ComponentScan("Concert")  
@EnableAspectJAutoProxy(proxyTargetClass = true)  
public class SpringConfig {  
}
```
假如在Spring中使用XML来装配bean的话，那么需要使用**Spring aop**命名空间中的<aop:aspectj-autoproxy>元素来启用。

经过这个步骤，就可以在`perform()`方法被调用时自动执行这些逻辑了：
```Console
Audience start to silence
Audience start to take seats
Performer start to perform
Audience start to applause
```

### 5.2 原方法的拦截
之前提到过，Spring切面中总有5种不同的通知类型，其中`@Around`环绕通知的功能最为强大，也最常在实际中使用。带有`@Around`的通知执行时可以简单分为三个阶段：原方法执行前，原方法执行，原方法执行后。因此使用`@Around`可以很轻松地做到对原方法的拦截，从而实现诸如功能扩展，参数校验与修改等功能。
比如上述`Audience`类中的前置通知和后置通知可以整合为环绕通知：
```Java
@Around("performPt()")  
public Object watch(ProceedingJoinPoint pjp) throws Throwable {  
    System.out.println("Audience start to silence");  
    System.out.println("Audience start to take seats");  
    try {  
        pjp.proceed();  
    } catch (Throwable throwable) {  
        System.out.println("Audience start to be sad");  
        throw   throwable;  
    }  
    System.out.println("Audience start to applause");  
    return null;  
}
```
环绕通知的书写还是应该遵循一些规范的。首先能注意到，相比于其他的通知，环绕通知多了一个类型为`ProceedingJoinPoint`的参数，它包含了aop代理链执行所需要的必要信息如切入点的对象，方法，属性等。
方法应该向上抛出一个异常，这是因为程序编译时无法确定被代理bean的方法是否会抛出异常；同样的，方法还需要返回值，因为也同样无法确定原方法是否包含返回值。并且环绕通知必须返回 `pjp.proceed()` 的结果，否则原方法的返回值将被丢失。
`pjp.proceed()`的作用是调用原方法，不要忘记了，通知的核心职责是增强原方法行为，而非替代它。

### 5.3 结合反射
Spring AOP结合反射的使用可以变得更加强大。
```Java
@Component  
public class Performer implements Performance{  
    private Integer score;  
  
    @Override  
    public void perform() {  
        System.out.println("Performer start to perform");  
    }  
  
    public void setScore(Integer score) {  
        this.score = score;  
    }  
  
    public int getScore() {  
        return score;  
    }  
}

// Audience类中
@Around("performPt()")  
public Object appraise(ProceedingJoinPoint joinPoint) throws Throwable {  
    joinPoint.proceed();  
  
    Object object = joinPoint.getTarget();  
    Method method = object.getClass().getMethod("setScore", Integer.class);  
    Integer i = new Random().nextInt(100) + 1;  
  
    method.invoke(object, i);  
  
    return null;  
}
```
Main类：
```Java
public static void main(String[] args) {  
        ApplicationContext context = new AnnotationConfigApplicationContext(SpringConfig.class);  
        Performer performer = context.getBean(Performer.class);  
        performer.perform();  

        System.out.println(performer.getScore());  
	}
```
结果：
```Console
Audience start to silence
Audience start to take seats
Performer start to perform
Audience start to applause
58
```
上述例子展示了使用反射，实现观众对演出者的随机评分。

但需要强调的是，Spring AOP的目的就是减少代码中反射的使用，此示例仅用于演示反射能力，实际开发中应避免直接操作目标对象内部状态，以保持封装性和可维护性。

### 5.4 处理通知中的参数
到目前为止，我们的切面都很简单，没有任何参数。但实际项目中，往往还需要对原方法的参数做一些处理，比如空指针校验、登录合法性校验，去除字符串中的空格：
```Java
public class DataAdvice{
	@Pointcut("execution(boolean *.*.service.*Service.*(*,*))")
	private void servicePt(){}
	
	@Around("DataAdvice.servicePt()")
	public Object trimStr(ProceedingJoinPoint pjp) throws Throwable {
			Object[] args = pjp.getArgs();
			for (int i = 0 ; i < args.length; i++) {
			//判断参数是不是字符串
			if (args[i].getclass().equals(String.class))
				// trim() 处理
				args[i] = args[i].toString().trim();
		}
		// 回调原方法，此时参数已被修改
		Object ret = pjp.proceed(args);
		return ret;
	}
}
```
`proceed()`方法可以接受多个参数：
```Java
Object proceed(Object[] var1) throws Throwable;
```
还注意到`ProceedingJoinPoint`还有方法`getArgs()`可以获取到原方法被调用时的入参，因此二者的结合便可以实现将原方法的参数“抽出来”，做一些修改后再“放回去”。