## 1. Spring配置的可选方案
在之前[[Bean、IoC 与 DI#^48f220]]提到过，Spring容器负责创建应用程序中的bean并通过DI来协调这些对象之间的关系。但是，作为开发人员，我们需要告诉Spring要创建哪些bean并且如何将其装配在一起。当描述bean如何进行装配时，Spring具有非常大的灵活性，它提供了三种主要的装配机制：
- 在XML中进行显式配置。 
- 在Java中进行显式配置。 
- 隐式的bean发现机制和自动装配。

选择很多，但这些选择并没有唯一的正确答案，而是应该根据项目实际来抉择。但一般来说，应该尽可能地使用自动装配，减少显式配置的占比；而当需要为第三方代码装配bean的时候，就必须需要显式配置bean，并且建议使用类型安全的JavaConfig；最后，只有当想要使用XML的命名空间，并且在JavaConfig中没有同样的实现时，才应该 使用XML。

## 2. 自动化装配bean
Spring从两个角度来实现自动化装配：
- 组件扫描（component scanning）：Spring会自动发现应用上下文 中所创建的bean。
- 自动装配（autowiring）：Spring自动满足bean之间的依赖。

### 2.1 创建可被发现的bean
首先定义一个接口：
```Java
public interface BookDao {  
    void save();  
}
```
然后是带有`@Component`实现类：[[@Component]]
```Java
@Component
public class BookDaoImpl implements BookDao {  
    @Override  
    public void save() {  
        System.out.println("BookDaoImpl save");  
    }  
}
```
具体的类或者方法不重要，需要注意的是实现类上带有了`@Component`注解。这个简单的注解表明该类会作为组件类，并告知Spring要为这个类创建 bean。没有必要显式配置BookDaoImpl，Spring会把事情处理妥当。

但是你就这样去运行肯定是不行的，因为组件扫描默认是不启用的。我们还需要显式配置一下Spring， 从而命令它去寻找带有`@Component`注解的类，并为其创建bean。
```Java
@Configuration  
@ComponentScan  
public class SpringConfig {  
}
```
`SpringConfig`类用了`@ComponentScan`注解，这个注解能够在Spring中启用组件扫描。如果没有其他配置的话，`@ComponentScan`默认会扫描与配置类相同的包。

### 2.2 为自动扫描的bean命名
Spring应用上下文中所有的bean都会给定一个ID。在前面的例子中，尽管我们没有明确地为`BookDaoImpl`设置ID，但Spring还是会根据类名为其指定一个ID。具体来讲，这个bean所给定的ID为`bookDaoImpl`，也就是将类名的第一个字母变为小写，但如果类名前两个字母都是大写（如 `URLService`），则保持原样。

如果想为这个bean设置不同的ID，可以将设定的ID值传递给`@Component`。

## 3. 为第三方代码手动装配bean
通常我们的项目中还会包含许多的第三方库，使用这些库时，我们会希望也给某些组件装配bean，从而进行应用。修改第三方的源代码是不现实的，但我们可以通过JavaConfig来显式装配。

### 3.1 声明简单的bean
要在JavaConfig中声明bean，我们需要编写一个方法，这个方法会创建所需类型的实例，然后给这个方法添加`@Bean`注解。
```Java
@Configuration  
public class SpringConfig {  
    @Bean  
    public BookDaoImpl bookDao() {  
        return new BookDaoImpl();  
    }  
}
```
`@Bean`注解会告诉Spring这个方法将会返回一个对象，该对象要注册为Spring应用上下文中的bean。

### 3.2 使用JavaConfig实现较复杂的注入
现在的代码变成：
```Java
public class Book {  
    private String bookName;  
  
    public void setBookName(String bookName) {  
        this.bookName = bookName;  
    }  
  
    public String getBookName() {  
        return bookName;  
    }  
}

public interface BookDao {  
    void save();  
  
    void read();  
}
 
public class BookDaoImpl implements BookDao {  
    private Book book;  
  
    public BookDaoImpl(Book book) {  
        this.book = book;  
    }  

    @Override  
    public void save() {  
        System.out.println("BookDaoImpl save");  
    }  
  
    @Override  
	public void read() {  
    	System.out.println("read" +  this.book.getBookName());  
	}
}
```
我们需要在JavaConfig中将`Book`和`BookDaoImp`l装配在一起，下面给出一种方案：
```Java
@Configuration  
public class SpringConfig {  
    @Bean  
    public Book book() {  
        Book book = new Book();  
        book.setBookName("storybook");  
        return book;  
    }  
  
    @Bean  
    public BookDaoImpl bookDao(Book book) {  
        return new BookDaoImpl(book);  
    }  
}
```
在这个例子中，`bookDao()`方法请求一个`Book`作为参数。在 这个方法中，我们通过调用 `book()` 方法手动注入了 `Book` 实例。由于 `book()` 也标注了 `@Bean`，Spring 会确保返回的是同一个 Bean 实例（默认单例）。
