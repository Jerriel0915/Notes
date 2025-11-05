## 1. 为什么需要事务

项目中往往会存在一些需要原子化操作的业务，一个经典例子就是转账——当转账这个操作发生时，它要么必须完成，要么就失败完全不发生。
比如下面这个例子：
```Java
@Service  
public class UserService {  
    @Autowired  
    UserMapper userMapper;  
   
    public void transfer(String from, String to, BigDecimal amount) {  
        userMapper.subMoney(from, amount);  
        int i = 1/0; // 模拟系统中断  
        userMapper.addMoney(to, amount);  
  
        System.out.println("Transfer from " + from + " to " + to + " amount: " + amount + " finished!");  
    }  
}
```
对应Mapper层方法为：
```Java
@Mapper  
public interface UserMapper {  
  
    @Update("update user set account = account + #{account} where username = #{username}")  
    void addMoney(@Param("username")String username, @Param("account") BigDecimal account);  
  
    @Update("update user set account = account - #{account} where username = #{username}")  
    void subMoney(@Param("username")String username, @Param("account") BigDecimal account);  
}
```
测试类：
```Java
@RunWith(SpringJUnit4ClassRunner.class)  
@Component  
@ContextConfiguration(classes = SpringConfig.class)  
public class UserServiceTest extends TestCase {  
    @Autowired  
    private UserService userService;  
  
    @Test  
    public void testTransfer() {  
        userService.transfer("Tom", "Jerry", BigDecimal.valueOf(100));  
    }  
}
```
配置类可参考：[[Spring jdbc+Mybatis配置类]]
数据库初始状态：
![数据库初始状态](https://pic-1371809842.cos.ap-chengdu.myqcloud.com/PicGo/20251029152829737.webp?imageSlim)

测试运行结果：
![程序出错](https://pic-1371809842.cos.ap-chengdu.myqcloud.com/PicGo/20251029153024472.webp?imageSlim)
数据库状态：
![数据不一致了](https://pic-1371809842.cos.ap-chengdu.myqcloud.com/PicGo/20251029153053219.webp?imageSlim)
可以看到这种后果是灾难性的——A账户的钱转出了，但是B账户却因为系统错误没有收到任何入账！

## 2. 什么是事务

**事务是逻辑上的一组操作，要么都执行，要么都不执行。**

事务是具有 ACID 性质的：
- **原子性**（**Atomicity**）：事务是最小的执行单位，不允许分割。事务的原子性确保动作要么全部完成，要么完全不起作用；
- **一致性**（**Consistency**）：执行事务前后，数据保持一致，例如转账业务中，无论事务是否成功，转账者和收款人的总额应该是不变的；
- **隔离性**（**Isolation**）：并发访问数据库时，一个用户的事务不被其他事务所干扰，各并发事务之间数据库是独立的；
- **持久性**（**Durability**）：一个事务被提交之后。它对数据库中数据的改变是持久的，即使数据库发生故障也不应该对其有任何影响。
只有保证了事务的持久性、原子性、隔离性之后，一致性才能得到保障。

## 3. 在Spring中引入事务

> 程序中的事务能否生效，数据库引擎是否支持事务是关键。比如常用的 MySQL 数据库默认使用支持事务的 **innodb**引擎。但是，如果把数据库引擎变为 **myisam**，那么程序也就不再支持事务了！

推荐使用注解`@Transactional`的方式引入事务，这样做的代码侵入性最小。
比如，我们只需要在上述例子的业务方法中加上该注解：
```Java
@Service  
public class UserService {  
    @Autowired  
    UserMapper userMapper;  
  
    @Transactional  
    public void transfer(String from, String to, BigDecimal amount) {  
        userMapper.subMoney(from, amount);  
        int i = 1/0; // 模拟中断  
        userMapper.addMoney(to, amount);  
  
        System.out.println("Transfer from " + from + " to " + to + " amount: " + amount + " finished!");  
    }  
}
```
然后在数据源配置类中加上事务的接口：
```Java
@Configuration  
public class jdbcConfig {  
    @Value("${jdbc.driver}")  
    private String driver;  
  
    @Value("${jdbc.url}")  
    private String url;  
  
    @Value("${jdbc.username}")  
    private String user;  
  
    @Value("${jdbc.password}")  
    private String password;  
  
    @Bean  
    public DataSource dataSource() {  
        DruidDataSource dataSource = new DruidDataSource();  
        dataSource.setDriverClassName(driver);  
        dataSource.setUrl(url);  
        dataSource.setUsername(user);  
        dataSource.setPassword(password);  
  
        return dataSource;  
    }  
  
  // 加上这个
    @Bean  
    public PlatformTransactionManager transactionManager() {  
        return new DataSourceTransactionManager(dataSource());  
    }  
}
```
最后还需要在项目配置类中加上`@EnableTransactionManagement`，如此最简三步完成后，虽然结果上还是不可避免的会出现程序出错、应用崩溃：
![依旧出错](https://pic-1371809842.cos.ap-chengdu.myqcloud.com/PicGo/20251029152923220.webp?imageSlim)
但是数据库内容却能被回滚至操作发生前的状态：
![数据保持了一致性](https://pic-1371809842.cos.ap-chengdu.myqcloud.com/PicGo/20251029152829737.webp?imageSlim)
事务的加入做到了我们之前提到的**ACID**性质。

## 4. TransactionDefinition：事务属性

Spring中并不会直接管理事务的实现，而是提供多种事务管理器，其中一个便是在之前配置类中写到的`PlatformTransactionManager`。

通过这个接口，Spring为各个平台提供对应的事务管理器，如：JDBC(`DataSourceTransactionManager`)、Hibernate(`HibernateTransactionManager`)、JPA(`JpaTransactionManager`)等，但是具体的实现就是各个平台自己的事情了。

`PlatformTransactionManager`接口定义了三个方法：
```Java
package org.springframework.transaction;  
  
import org.springframework.lang.Nullable;  
  
public interface PlatformTransactionManager extends TransactionManager {  
	// 获得事务
    TransactionStatus getTransaction(@Nullable TransactionDefinition definition) throws TransactionException;  
	// 提交事务
    void commit(TransactionStatus status) throws TransactionException;  
	// 回滚事务
    void rollback(TransactionStatus status) throws TransactionException;  
}
```
事务管理器接口`PlatformTransactionManager` 通过 `getTransaction(TransactionDefinition definition)`方法来得到一个事务，这个方法里面的参数是`TransactionDefinition`类 ，这个类就定义了一些基本的事务属性。

事务属性包含了 5 个方面：

- 隔离级别
- 传播行为
- 回滚规则
- 是否只读
- 事务超时

### 4.1 事务传播

**事务传播行为是为了解决业务层方法之间互相调用的事务问题**。

当事务方法被另一个事务方法调用时，必须指定事务应该如何传播。例如：方法可能继续在现有事务中运行，也可能开启一个新事务，并在自己的事务中运行。

比如之前的代码中，`Mapper`层中的方法`addMoney()`和`subMoney()`对数据库来说是两个独立的事务，而我们通过加上注解`@Transactional`的方式使`Service`层中的方法`transfer()`成为了一个新的事务。似乎在这时这三个事务都是互相独立的，但是结果上看这三个事务被归并为了同一个事务并进行了回滚。这便是Spring的事务传播机制导致的。

在`TransactionDefinition`定义中包括了如下几个表示传播行为的常量：
```java
public interface TransactionDefinition {
    int PROPAGATION_REQUIRED = 0;
    int PROPAGATION_SUPPORTS = 1;
    int PROPAGATION_MANDATORY = 2;
    int PROPAGATION_REQUIRES_NEW = 3;
    int PROPAGATION_NOT_SUPPORTED = 4;
    int PROPAGATION_NEVER = 5;
    int PROPAGATION_NESTED = 6;
    ......
}
```

不过为了方便使用，Spring 相应地定义了一个枚举类：`Propagation`
```java
package org.springframework.transaction.annotation;

import org.springframework.transaction.TransactionDefinition;

public enum Propagation {

    REQUIRED(TransactionDefinition.PROPAGATION_REQUIRED),

    SUPPORTS(TransactionDefinition.PROPAGATION_SUPPORTS),

    MANDATORY(TransactionDefinition.PROPAGATION_MANDATORY),

    REQUIRES_NEW(TransactionDefinition.PROPAGATION_REQUIRES_NEW),

    NOT_SUPPORTED(TransactionDefinition.PROPAGATION_NOT_SUPPORTED),

    NEVER(TransactionDefinition.PROPAGATION_NEVER),

    NESTED(TransactionDefinition.PROPAGATION_NESTED);

    private final int value;

    Propagation(int value) {
        this.value = value;
    }

    public int value() {
        return this.value;
    }

}
```

#### 4.1.1 `PROPAGATION_REQUIRED`
这是最为常用的一个事务传播行为，也是`@Transactional`注解的默认值。
如果当前存在事务，则加入该事务；如果当前没有事务，则创建一个新的事务。也就是说：
- 如果外部方法没有开启事务的话，`Propagation.REQUIRED`修饰的内部方法会新开启自己的事务，且开启的事务相互独立，互不干扰。
- 如果外部方法开启事务并且被`Propagation.REQUIRED`的话，所有`Propagation.REQUIRED`修饰的内部方法和外部方法均属于同一事务 ，只要一个方法回滚，整个事务均回滚。

这样就能解释之前的转账例子了。`addMoney()`和`subMoney()`方法在被外部方法调用时，被归入到了外部方法的事务当中，这样转账过程中无论在哪出错发生了回滚，整个事务都会发生回滚。

#### 4.1.2 `PROPAGATION_REQUIRES_NEW`
创建一个新的事务，如果当前存在事务，则把当前事务挂起。也就是说不管外部方法是否开启事务，`Propagation.REQUIRES_NEW`修饰的内部方法会新开启自己的事务，且开启的事务相互独立，互不干扰。
比如项目中有一个日志模块，我们希望无论操作是否成功，都应该有相关的日志记录进数据库，即写入日志这一操作不会随着外层事务的回滚而回滚：
```Java
@Service  
public class LogService {  
    @Autowired  
    LogMapper logMapper;  
  
    @Transactional(propagation = Propagation.REQUIRES_NEW)  
    public void printLog(String msg) {  
	    // 这个函数会将 msg 写入进对应数据库
        logMapper.log(msg);  
    }  
}
```
测试类：
```Java
@Service  
public class UserService {  
    @Autowired  
    UserMapper userMapper;  
    @Autowired  
    LogService logService;  
  
    @Transactional(propagation = Propagation.REQUIRED)  
    public void transfer(String from, String to, BigDecimal amount) {  
        userMapper.subMoney(from, amount);  
        userMapper.addMoney(to, amount);  
        logService.printLog("Transfer from " + from + " to " + to + " amount: " + amount);  
        int i = 1/0; // 模拟中断  
    }  
}
```
运行结果：
![回滚](https://pic-1371809842.cos.ap-chengdu.myqcloud.com/PicGo/20251029180738193.webp?imageSlim)
转账操作被成功回滚。

![未被回滚](https://pic-1371809842.cos.ap-chengdu.myqcloud.com/PicGo/20251029180748156.webp?imageSlim)
而日志模块的写入不受影响。

> [!TIP] 注意
> **Spring 的事务是通过 AOP 代理实现的。当你在同一个类内部调用一个带有 `@Transactional` 注解的方法时，调用不会经过代理对象，而是直接通过 `this` 调用，导致事务注解失效。**
> 比如像这样：
>```Java
> @Transactional(propagation = Propagation.REQUIRED)
public void transfer(String from, String to, BigDecimal amount) {
   userMapper.subMoney(from, amount);
   userMapper.addMoney(to, amount);
   printLog("Transfer from " + from + " to " + to + " amount: " + amount); // ❗这里
   int i = 1/0; // 模拟中断
}
@Transactional(propagation = Propagation.REQUIRES_NEW)
public void printLog(String msg) {
   userMapper.log(msg);
}
>```
>是会将`printlog()`方法一同回滚的。
>
>这是因为Spring AOP的代理对象的原理导致的，Spring会在运行的时候为带有 `@Transactional` 注解的方法生成代理对象，代理对象能够拦截外部的方法调用并处理事务，但的代理对象内部的方法调用直接使用`this`调用，不会经过拦截，事务也就失效了。

#### 4.1.3 `PROPAGATION_NESTED`
如果当前存在事务，则创建一个事务作为当前事务的嵌套事务执行； 如果当前没有事务，就执行与`PROPAGATION_REQUIRED`类似的操作。也就是说：
- 在外部方法开启事务的情况下，在内部开启一个新的事务，作为嵌套事务存在。
- 如果外部方法无事务，则单独开启一个事务，与 `PROPAGATION_REQUIRED` 类似。

`PROPAGATION_NESTED`代表的嵌套事务以父子关系呈现，其核心理念是子事务不会独立提交，依赖于父事务，在父事务中运行；当父事务提交时，子事务也会随着提交，理所当然的，当父事务回滚时，子事务也会回滚；

> 与`PROPAGATION_REQUIRES_NEW`区别于：`PROPAGATION_REQUIRES_NEW`是独立事务，不依赖于外部事务，以平级关系呈现，执行完就会立即提交，与外部事务无关；

子事务也有自己的特性，可以独立进行回滚，不会引发父事务的回滚，但是前提是需要处理子事务的异常，避免异常被父事务感知导致外部事务回滚；

#### 4.1.4 `PROPAGATION_MANDATORY`
如果当前存在事务，则加入该事务；如果当前没有事务，则抛出异常。（mandatory：强制性）

#### 4.1.5 总结
除了上面提到的4个较常见传播方式，Spring事务的传播方式还有几种，下面直接给出一个汇总：
![事务汇总](https://pic-1371809842.cos.ap-chengdu.myqcloud.com/PicGo/20251029182949825.webp?imageSlim)
具体讲解可以看这个视频：[黑马程序员](https://www.bilibili.com/video/BV1Fi4y1S7ix?p=42&vd_source=367f9dfa83a55e4607b96388b5b70009)
也可以通过这个链接了解更多事务相关内容：[Spring 事务详解 | JavaGuide](https://javaguide.cn/system-design/framework/spring/spring-transaction.html#transactiondefinition-%E4%BA%8B%E5%8A%A1%E5%B1%9E%E6%80%A7)

### 4.2 事务超时属性

所谓事务超时，就是指一个事务所允许执行的最长时间，如果超过该时间限制但事务还没有完成，则自动回滚事务。在 `TransactionDefinition` 中以 int 的值来表示超时时间，其单位是秒，默认值为-1，这表示事务的超时时间取决于底层事务系统或者没有超时时间。

### 4.3 事务只读属性

```java
package org.springframework.transaction;

import org.springframework.lang.Nullable;

public interface TransactionDefinition {
    ......
    // 返回是否为只读事务，默认值为 false
    boolean isReadOnly();

}
```
对于只有读取数据查询的事务，可以指定事务类型为 `readonly`，即只读事务。只读事务不涉及数据的修改，数据库会提供一些优化手段，适合用在有多条数据库查询操作的方法中。

通常应用在需要多次查询的业务中，例如统计调查，报表查询，在这种场景下，多条查询 SQL 必须保证整体的读一致性。否则，在前条 SQL 查询之后，后条 SQL 查询之前，数据被其他用户改变，则该次整体的统计查询将会出现读数据不一致的状态，此时，应该启用事务支持。

### 4.4 事务回滚规则

实物回滚规则定义了哪些异常会导致事务回滚而哪些不会。默认情况下，事务只有遇到运行期异常（`RuntimeException` 的子类）时才会回滚，`Error` 也会导致事务回滚，但是，在遇到检查型（Checked）异常时不会回滚。

如果需要回滚定义的特定的异常类型的话，可以这样：
```java
@Transactional(rollbackFor= MyException.class)
```

## 5. `@Transactional`注解使用

通常将`@Transactional`应用在具体方法上，不推荐在接口上使用。

> [!TIP] 注意
> `@Transactional`注解只能应用到 **public** 方法上，否则不会生效。如果这个注解使用在类上的话，表明该注解对该类中所有的 **public** 方法都生效。

最后的总结：
- `@Transactional` 注解只有作用到 public 方法上事务才生效，不推荐在接口上使用；
- 避免同一个类中调用 `@Transactional` 注解的方法，这样会导致事务失效；
- 正确的设置 `@Transactional` 的 `rollbackFor` 和 `propagation` 属性，否则事务可能会回滚失败;
- 被 `@Transactional` 注解的方法所在的类必须被 Spring 管理，否则不生效；
- 底层使用的数据库必须支持事务机制，否则不生效；