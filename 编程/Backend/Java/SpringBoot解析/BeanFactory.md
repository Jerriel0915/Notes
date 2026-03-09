---
tags:
  - SpringBoot
  - 后端
---
## 1. 引入
当我们使用IDEA新建并初始化一个SpringBoot项目时，会自动创建好一个启动类`项目名+Application`——其实你也可以自己定义一个类实现启动功能，只要你打上了`@SpringBootApplication`注解，这为框架标识着应用的主入口。

通常会使用方法`SpringApplication.run()`启动Spring应用的上下文，进行自动配置和组件扫描。
如下是一个完整的项目启动类：
```Java
@SpringBootApplication  
public class ProjectApplication {  
    public static void main(String[] args){
    SpringApplication.run(ProjectApplication.class, args);
}
```
而只要我们稍微细心一些，就能发现这个`SpringApplication.run()`方法是有返回值的，不过通常会忽略它。

```Java
public static ConfigurableApplicationContext run(Class<?> primarySource, String... args);
```
方法返回`ConfigurableApplicationContext`类，通过IDEA自带的工具（通过快捷键Ctrl + Alt + U调用）可以查看这个类的类图。

![ConfigurableApplicationContext 的类图](https://pic-1371809842.cos.ap-chengdu.myqcloud.com/PicGo/202510140006338.png) ^ConfigurableApplicationContext

可以看到`ConfigurableApplicationContext`接口继承了`ApplicationContext`接口，而`ApplicationContext`接口又间接地继承了`BeanFactory`接口。

## 2. BeanFactory
### 2.1 什么是 BeanFactory
BeanFactory 是 Spring **IoC（控制反转） 容器最顶层的根接口**，用于管理和维护Bean的生命周期，负责从配置文件或者注解中读取Bean的定义信息，并根据需要创建相应的Bean实例。

### 2.2 BeanFactory 能做什么

![BeanFactory 中的方法](https://pic-1371809842.cos.ap-chengdu.myqcloud.com/PicGo/20251014172209590.webp?imageSlim)

表面上看，可能只会注意到`getBean()` 或者 `containsBean()`这些方法，错误地以为就只有这些功能。实际上，控制反转，基本的依赖注入直至 Bean 的生命周期管理，都由它的实现类完成。

### 2.3 BeanFactory 的实现类 DefaultListableBeanFactory
`BeanFactory` 的实现类是`DefaultListableBeanFactory`。
可以看到它的类关系就很复杂了，不仅仅是实现了 `BeanFactory` 接口，还有其他的很多接口。

![DefaultListableBeanFactory 的类图](https://pic-1371809842.cos.ap-chengdu.myqcloud.com/PicGo/20251014174915646.webp?imageSlim)

其中有一个很重要的父类，叫 `DefaultSingletonBeanRegistry` ，通过内部的 Map 负责管理单例对象：
```Java
private final Map<String, Object> singletonObjects = new ConcurrentHashMap(256);
```
实际开发中可以通过反射拿到这个 Map：
```Java
ConfigurableApplicationContext context = SpringApplication.run(ProjectApplication.class, args);  
  
Field singletonObjects = DefaultSingletonBeanRegistry.class.getDeclaredField("singletonObjects");  
singletonObjects.setAccessible(true);  
ConfigurableListableBeanFactory beanFactory = context.getBeanFactory();  
Map<String, Object> mp = (Map<String, Object>) singletonObjects.get(beanFactory);
```
如果我们有两个 Bean：
```Java
@Component
public class Component1 {
}

@Component
public class Component2 {
}
```
那就可以找到这两个 Bean 并获取到它们的信息：
```Java
 map.entrySet().stream().filter(e -> e.getKey().startsWith("component"))
            .forEach(e -> System.out.println(e.getKey() + "=" + e.getValue()));
```
输出内容：
```Console
component1=indi.mofan.bean.a01.Component1@25a5c7db
component2=indi.mofan.bean.a01.Component2@4d27d9d
```

## 3. ApplicationContext
从上文[[#^ConfigurableApplicationContext]]提到过了`ApplicationContext`父类，它的功能也很多：
- MessageSource：处理国际化资源
- ResourcePatternResolver：使用通配符进行资源匹配
- EnvironmentCapable：读取 Spring 环境信息、配置文件信息
- ApplicationEventPublisher：发布事件

### 3.1 MessageSource
作为一个国际化的框架，自然少不了国际化处理。`MessageSource`就是Spring 中对国际化文件支持的基础接口。

在 SpringBoot 项目的 resources 目录下创建 messages.properties、messages_en.properties，在 messages_en.properties 中写入：
```properties
hi=hello
```
之后运行：
```Java
String message = context.getMessage("hi", null, Locale.ENGLISH);  
System.out.println(message);
```
便能得到对应的“翻译”结果
```Console
hello
```

### 3.2 ResourcePatternResolver
```Java
// 资源获取
Resource[] resources = context.getResources("classpath*:META-INF/spring.factories");  
for (Resource resource : resources) {  
    System.out.println(resource);  
}
```

### 3.3 EnvironmentCapable
使用`EnvironmentCapable`可以很方便地获取一些环境变量：
```Java
// 获取环境变量  
        System.out.println(context.getEnvironment().getProperty("java_home"));  
        System.out.println(context.getEnvironment().getProperty("java.version"));  
        System.out.println(context.getEnvironment().getProperty("os.name"));  
        System.out.println(context.getEnvironment().getProperty("os.arch"));  
  
        if (context.getEnvironment().getProperty("os.name").startsWith("Windows")) {  
            System.out.println("安卓电脑！你不配运行这个项目！");  
            System.exit(0);  
        }
```
其中`getProperty`可以从配置文件或系统环境变量中获取相关值。
### 3.4 ApplicationEventPublisher
`ApplicationEventPublisher`是底层的消息推送类，使用它可以实现功能解耦——对于我这个类来说，完成了某项功能之后，只需要发送相应的消息，之于谁处理这个消息不管，留给别人。比如用户完成注册后，需要通过邮箱或手机短信发送欢迎消息，那么只需要在Controller层处理完之后发送相关消息，具体是发邮件还是短信不用考虑，相关功能的实现类就能监听到这个消息后进行处理。

消息的发送默认为同步，若需异步处理需加上@Async，然后在启动类上加@EnableAsync。
```Java
context.publishEvent(new ApplicationReadyEvent(context));
```
建议使用自定义事件类：
```Java
// 自定义的Event  
public class ApplicationReadyEvent extends ApplicationEvent {  
  
    public ApplicationReadyEvent(Object source) {  
        super(source);  
    }  
}
```
若想监听相关事件，只要方法参数能对应上，并加上@EventListener注解即可：
```Java
@Component  
public class Component1 {  
    private static final Logger logger= LoggerFactory.getLogger(Component1.class);  
  
    // 消息监听  
    @EventListener(ApplicationReadyEvent.class)  
    public void listenEvent(ApplicationEvent event){  
        logger.info("ApplicationReadyEvent");  
    }  
}
```
运行结果：
```Console
2025-10-14 22:45:19.287  INFO 25276 --- [           main] com.musicplayer.Component.Component1     : ApplicationReadyEvent
```
 