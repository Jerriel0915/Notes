## 1. 没有`Optional`时

开发中，我们往往会遇到一个情景：从数据层的某个接口处获取数据，然后做一些处理。通常我们不可保证数据层的绝对可靠性，因此获取到的数据有可能为空。

比如这个例子：
```Java
// 可能返回 null
public User findUserFromDB(long id) {
    return repo.selectById(id);   // 查不到就返回 null
}

// 调用方
User u = findUserFromDB(1L);
if (u != null) {                // 必须手动判断
    System.out.println(u.getName());
}
```

在 Java 8 之前，当方法可能“没有返回值”时，我们通常有两种做法：
1. 返回 `null`
2. 抛异常（如 `IllegalArgumentException`、`NoSuchElementException` 等）交给处理器

这两种方式都有明显缺点：
1. 返回 `null`：调用方必须记得手动做空指针检查，但凡忘记了`u != null`检查，那分分钟报错NPE：
```Console
Exception in thread "main" java.lang.NullPointerException
```
2. 抛异常：异常只应该在“异常”情况下使用，频繁抛出和捕获性能差，而且语义上也不清晰。

## 2. 引入`Optional`

针对上面这种场景，Java 8 之后引入了`Optional<T>`，就是为了把“可能没有值”这一事实**显式地**体现在类型系统中，从而：
- 让调用方一眼就能看出“这个方法可能不会返回结果”，强制思考无值时该怎么处理；
- 避免到处写 `if (obj != null)`；
- 减少 `NullPointerException` 出现的概率；
- 提供函数式风格的链式操作（`map`、`flatMap`、`filter` 等），让代码更流畅、可读性更高。

上面的代码可以改为：
```Java
public Optional<User> findUserFromDB(long id) {
    return Optional.ofNullable(repo.selectById(id));
}

// 调用方
findUserFromDB(1L)
        .ifPresent(u -> System.out.println(u.getName()));
```

通过阅读代码我们就可以得知`findUserFromDB()`方法返回一个`Optional<User>`，这就表明`User`可能存在，也可能不存在，如果存在的话，使用Optional提供的`ifPresent()`使用Lambda 表达式来直接打印结果。

可能这个简短的例子还不足以体现Optional的意义，我们可以看一个复杂点的例子：

假设我们要完成“根据订单号查订单 → 查客户 → 查客户邮箱 → 把邮箱转小写”，传统写法是：
```java
public String lowerEmail {
	Order order = orderService.findOrder(orderId);
	if (order != null) {
	    Customer c = order.getCustomer();
	    if (c != null) {
	        String email = c.getEmail();
	        if (email != null) {
	            return email.toLowerCase();
	        }
	    }
	}
	return "unknown";
}
```
即使用短路改写，这个函数依旧充斥着大量if语句：
```java
public String lowerEmail {
	Order order = orderService.findOrder(orderId);
	String defaultResult = "unknown";
	if (order == null) {
		return defaultResult;
	}
	
	Customer c = order.getCustomer();
	if (c == null) {
		return defaultResult;
	}
	
	String email = c.getEmail();
	if (email == null) {
		return defaultResult;
	}
	
	return email.toLowerCase();
}
```

但是使用`Optional`就可以很简洁了：
```java
public String lowerEmail {
	return orderService.findOrder(orderId)
	        .map(Order::getCustomer)
	        .map(Customer::getEmail)
	        .map(String::toLowerCase)
	        .orElse("unknown");
}
```

当链中的任何一个判断为空值时，都将会传递至链尾的`orElse()`进行处理，返回默认值。

## 3. 创建`Optional`对象

1. 使用静态方法 `empty()` 创建一个**空的** Optional 对象
```
Optional<String> empty = Optional.empty();
System.out.println(empty); // 输出：Optional.empty
```

2. 使用静态方法 `of()` 创建一个**非空的** Optional 对象
```
Optional<String> opt = Optional.of("王二");
System.out.println(opt); // 输出：Optional[王二]
```

当然了，传递给 `of()` 方法的参数必须是非空的，也就是说不能为 null，否则仍然会抛出 `NullPointerException`。
```
String name = null;
Optional<String> optnull = Optional.of(name);
```

3. 可以使用静态方法 `ofNullable()` 创建一个**既可空又可非空的** Optional 对象
```
String name = null;
Optional<String> optOrNull = Optional.ofNullable(name);
System.out.println(optOrNull); // 输出：Optional.empty
```

`ofNullable()` 方法内部有一个三元表达式，如果为参数为 null，则返回私有常量 EMPTY；否则使用 new 关键字创建了一个新的 Optional 对象——不会再抛出 NPE 异常了。

## 4. 常用方法
### `isPresent()`
判断 optional 对象是否为存在，如果存在则返回 true——取代 `obj != null`
```Java
Optional<String> optional = Optional.of("Hello");  
System.out.println(optional.isPresent()); // 输出 True
```

### `isEmpty()`
 Java 11 后还可以通过方法 `isEmpty()` 判断与 `isPresent()` 相反的结果。
```Java
Optional<String> optional = Optional.of("Hello");  
System.out.println(optional.isEmpty()); // 输出 False
```

### `ifPresent(Consumer)`
当对象非空时，允许使用函数式编程的方式执行一些代码。
```Java
Optional<String> optional = Optional.of("Hello");  
optional.ifPresent(System.out::println); // 输出 Hello
```

### `ifPresentOrElse(action, emptyAction)`
Java 9 后还可以通过该方法执行两种结果，非空时执行 action，空时执行 emptyAction。
```Java
Optional<String> opt = Optional.of("Hello");
opt.ifPresentOrElse(str -> System.out.println(str.length()), () -> System.out.println("为空"));
// 输出 5
```

### `orElse(T)`
有时候创建或获取 Optional 对象的时候，需要一个默认值。`orElse(T)` 方法用于返回包裹在 Optional 对象中的值，如果该值不为 null，则返回；否则返回默认值。该方法的参数类型和值的类型一致。
```Java
String nullName = null;  
String name = Optional.ofNullable(nullName).orElse("Hello");  
System.out.println(name); // 输出：Hello
```

### `orElseGet(Supplier)`
与 `orElse(T)` 方法类似，但参数类型不同。如果 Optional 对象中的值为 null，则执行参数中的函数。
```Java
String nullName = null;  
String name = Optional.ofNullable(nullName).orElseGet(() -> "Hello");  
System.out.println(name); // 输出：Hello
```

> [!TIP] 注意
> `orElse()`方法无论对象是否为空，都会初始化参数，也就是说如果使用函数方法传参，方法总是会被执行的；而`orElseGet()`方法在对象不为空时不会初始化参数。

比如这个例子：
```Java
public static void main(String[] args) {  
    String msg = "Message";  
    System.out.println("orElse");  
    String msg2 = Optional.ofNullable(msg).orElse(getDefaultValue());  
    System.out.println(msg2);  
  
    System.out.println("\norElseGet");  
    String msg3 = Optional.ofNullable(msg).orElseGet(Main::getDefaultValue);  
    System.out.println(msg3);  
}  
  
public static String getDefaultValue() {  
    System.out.println("getDefaultValue");  
    return "DefaultMessage";  
}
```
输出结果：
```Console
orElse
getDefaultValue
Message

orElseGet
Message
```
所以建议使用 `orElseGet()`方法以获得更高性能。

### `get()`
直观上来看，`get()` 方法才是最正宗的获取 Optional 对象值的方法，但很遗憾，这个方法是有缺陷的，因为假如 Optional 对象的值为 null，该方法会抛出` NoSuchElementException` 异常。那还不如不用 Optional。
```Java
public static void main(String[] args) { 
	String name = null; 
	Optional<String> optOrNull = Optional.ofNullable(name); 
	System.out.println(optOrNull.get()); 
}
```
结果：
```Console
Exception in thread "main" java.util.NoSuchElementException: No value present
```

### `filter()`
`filter()` 方法的参数类型为 `Predicate`，也就是说可以将一个 Lambda 表达式传递给该方法作为条件，如果表达式的结果为 false，则返回一个 EMPTY 的 Optional 对象，否则返回过滤后的 Optional 对象。
```Java
public static void main(String[] args) {  
    String password = "123456";  
    Optional<String> optional = Optional.of(password);  
    optional.filter(pwd -> pwd.length() > 5)  
            .ifPresent(System.out::println);  
}
```

### `map()`
`map()`允许对 Optional 对象执行计算操作。如果有值，它就对该值应用一个函数，并将结果包装在一个**新的** Optional 对象中；如果原始 Optional 为空，map操作不会执行任何操作。
```Java
public static void main(String[] args) {  
    String password = "123456";  
    Optional<String> pwdOptional = Optional.of(password);  
    Optional<Integer> intOptional = pwdOptional  
            .map(String::length);  
    System.out.println(intOptional.orElse(0));  // 输出 6
}
```

### `flatMap()`
与`map()`作用相似，但它们处理返回值的方式不同。`map()` 方法会将函数的返回值包装在一个新的Optional 中，即使这个值已经是一个Optional；`flatMap()` 方法期望函数返回一个 Optional 对象，并直接返回这个 Optional ，而不会进行额外的包装。
如果函数已经返回了一个 Optional，那么应该使用 `flatMap()` 而不是 `map()`，以避免得到嵌套的Optional 对象。

### `Stream`
Optional 还可以直接和 stream 流结合使用。
```java
List<Order> list = ...
Optional<Order> first = list.stream()
                            .filter(Order::isToday)
                            .findFirst();   // 可能一个都没有
first.ifPresent(this::notifyUser);
```

## 5. 注意事项

-  **不要把 Optional 当字段使用**
```java
public class User {
		private Optional<String> email; // 反模式！序列化、性能都差
}
```
Optional **只用于返回值或局部变量**，不要作为类字段或方法参数。

-  **不要直接 `get()` 而不判断**
```java
Optional<User> op = findUser(1L);
User u = op.get(); // 可能 NoSuchElementException，违背初衷
    ```
用 `orElse / orElseGet / orElseThrow(Supplier)` 显式给出无值策略。

 -  **别用来包装集合**
集合本身就能表达“空集合”，`List<User>` 为空即可，不必 `Optional<List<User>>`。