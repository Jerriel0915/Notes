---
tags:
  - 后端
  - 面试
  - Bean
  - Spring
  - DI
  - IoC
---
`@Component` 是 Spring 框架中**最核心的注解之一**，它的作用一句话概括就是：
> **标记一个类为 Spring 的组件，告诉 Spring：“请你帮我创建这个类的实例（bean），并交给容器管理。”**

---
### ✅ 具体作用分解：

| 作用 | 说明 |
|------|------|
| **1. 声明为 Bean** | 被 `@Component` 标注的类会被 Spring 自动扫描并注册为 Bean。 |
| **2. 支持自动装配** | 这个 Bean 可以被 `@Autowired` 注入到其他类中。 |
| **3. 配合组件扫描** | 必须配合 `@ComponentScan` 使用，Spring 才会去扫描并注册它。 |
| **4. 通用型注解** | 是 `@Service`、`@Repository`、`@Controller` 的**父注解**，功能相同，只是语义不同。 |

---
### ✅ 举个例子

```java
@Component
public class EmailService {
    public void sendEmail(String to) {
        System.out.println("Sending email to " + to);
    }
}
```
```java
@Component
public class UserService {
    @Autowired
    private EmailService emailService;

    public void register(String email) {
        emailService.sendEmail(email);
    }
}
```
Spring 会自动：
1. 创建 `EmailService` 和 `UserService` 的实例；
2. 把 `EmailService` 注入到 `UserService` 中。

---
### ✅ 和 `@Bean` 的区别（面试常问）

1. 作用对象不同：`@Component`注解作用于类，而 `@Bean` 注解作用于方法、
2. `@Component`通常是通过路径扫描来自动侦测，以及自动装配到 Spring容器中(我们可以使用`@ComponentScan`注解定义要扫描的路径，从中找出标识了需要装配的类，自动装配到 Spring的bean 容器中)。
3. `@Bean`注解通常是我们在标有该注解的方法中定义产生这个 bean，`@Bean`告诉了 Spring 这是某个类的实例，当我们需要用它的时候还给我。
4. `@Bean`注解比`@Component` 注解的自定义性更强，而且很多地方我们只能通过`@Bean`注解来注册bean。比如当我们引用第三方库中的类需要装配到Spring 容器时，只能通过`@Bean`来实现。

比如下面这个例子就只能通过`@Bean`实现：
```JavaScript
@Bean public OneService getService(status) {
	case (status) {
	when 1: return new serviceImpl1();
	when 2: return new serviceImpl2();
	when 3: return new serviceImpl3();
	}
}
```

| 对比项  | `@Component`   | `@Bean`                    |
| ---- | -------------- | -------------------------- |
| 作用位置 | 类上             | 方法上（在 `@Configuration` 类中） |
| 控制权  | Spring 自动扫描    | 开发者手动控制实例化过程               |
| 适用场景 | 自己写的类          | 第三方类（如 `new Date()`）       |
| 灵活性  | 低（只能默认构造或自动注入） | 高（可自定义构造参数、初始化逻辑）          |

---
### ✅ 总结
> `@Component` 就是告诉 Spring：“这个类你帮我管起来，我要用它的时候你直接给我。”

它是 Spring **自动装配**和**IoC容器管理**的起点。