---
tags:
  - SpringBoot
  - 后端
---
## 引入
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

![`ConfigurableApplicationContext`的类图](https://pic-1371809842.cos.ap-chengdu.myqcloud.com/PicGo/202510140006338.png)

可以看到`ConfigurableApplicationContext`接口继承了`ApplicationContext`接口，而`ApplicationContext`接口又间接地继承了`BeanFactory`接口。



BeanFactory 是 Spring **IoC 容器最顶层的根接口**