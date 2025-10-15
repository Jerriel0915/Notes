## 1. IoC思想
### 1.1 什么是 IoC?
IoC，英文全称**Inversion of Control**，中文翻译为**控制反转**。

### 1.2 为什么需要控制反转呢？
直接看下面这个例子吧：
```Java
// 业务层代码
public class BookServiceImpl implements BookService {
	private BookDao bookDao = new BookDaoImpl();
	
	public void save() {
		bookDao.save();
	}
}

// 数据层
public class BookDaoImpl implements BookDao {
	public void save() {
		// do something
	}
}
```
这份代码确实能跑起来，也能完美实现你的**现有**需求。

可是过了不久，你需要对数据层的逻辑进行修改，于是你有了：
```Java
public class BookDaoImpl2 implements BookDao {
	public void save2() {
		// do something else
	}
}
```
光是更改数据层肯定不够，因为你还在业务层调用了这个接口，于是你跟着把业务层的代码也改了：
```Java
public class BookServiceImpl implements BookService {
	private BookDao bookDao = new BookDaoImpl2();
	
	public void save() {
		bookDao.save2();
	}
}
```
诶，又过了几天，你发现你之前更改的数据层有BUG，于是又在各个地方进行更改。到这时，有些东西就开始初现端倪了——你每做出一次更改，都要在各种调用处修改代码。

这就是代码的**高耦合度**，这对于一个项目的后期维护是灾难性的。
需要补充的是，耦合具有两面性。一方面，紧密耦合的代码几乎难以复用、难以测试、难以理解。另一方面，一定程度的耦合是必须的——完全没有耦合的代码等于什么也做不了。因此对于每个项目中的代码，我们应该做的是尽量降低耦合度。

上面这个问题的解决方案也简单，那就是每次使用对象的时候，在程序中不主动使用`new`产生对象实例，转换为**外部**提供对象。也就是说，将对象的创建控制权由程序本身转移到外部。
这就是控制反转思想的体现。

### 1.3 Spring 中的IoC思想
Spring给我们提供了一个容器，称为IoC容器，由这个容器来充当我们之前提到的IoC思想中的**外部**。
IoC容器负责对象的创建、初始化等等关于生命周期的一系列工作，而这些被创建或者被管理的对象在IoC容器中有一个统一的名字——**Bean**。

## 2. DI 依赖注入
回到刚才的例子，业务层的逻辑依靠数据层的Dao对象才能运行，也就是说业务层依赖于数据层对象的实现。那么光有一个外部的对象是不够的，还需要将业务层的依赖对象和外部的对象相绑定，相关联，将对象实例“注入”进代码，才能运行起整个项目。

所以这就是**DI(Dependency Injection) 依赖注入**

通过DI，对象的依赖关系由系统中负责协调各对象的第三方组件在创建对象时设定，对象无需自行创建或管理它们的依赖关系。

### 2.1 依赖注入的方式
#### 2.1.1 构造器
首先举个例子（来源——《Spring实战第四版P30》）：
```Java
public interface Knight {  
    void embarkOnQuest();  
}

public class Quest {  
    public void embark() {  
        System.out.println("embark");  
    }  
}

// 这是一位勇敢的骑士
public class BraveKnight implements Knight{  
    private final Quest quest;  
  
    public BraveKnight(Quest quest) {  
        this.quest = quest;  
    }  
  
    @Override  
    public void embarkOnQuest() {  
        if (quest != null) {quest.embark();} 
    }  
}
```
可以看到，我们勇敢的骑士足够灵活可以接受任何指派给他的任务，这是因为`BraveKnight`没有在类内部自行创建任务，而是构造时将探险任务作为构造器参数传入。
这就是**构造器注入（constructor injection）**

更进一步，可以将`Quest`抽象化：
```Java
public abstract class Quest {  
    public void embark() {
    System.out.println("embark on base quest");
    }
}
```
而后续传入的所有任务只要继承并实现了这个类，我们的`BraveKnight`就能够响应。那么我们就可以说做到了骑士与任务的解耦。这就是**DI**带来的最大收益。
```Java
// 新的委托任务
public class SlayDragonQuest extends Quest{  
    @Override  
    public void embark() {  
        System.out.println("Embarking on SlayDragonQuest!");  
    }  
}
```
解耦的另一个好处就是便于编写测试类（使用mock）：
```Java
class BraveKnightTest {  
  
    @Test  
    void embarkOnQuest() {  
        Quest mockQuest = mock(Quest.class);  
        BraveKnight braveKnight = new BraveKnight(mockQuest);  
        braveKnight.embarkOnQuest();  
        verify(mockQuest, times(1)).embark();  
    }  
}
```

#### 2.1.2 使用 Spring 自动装配
> 创建应用组件之间协作的行为通常称为**装配（wiring）**。Spring有多种装配bean的方式，采用XML是很常见的一种装配方式。

在项目中创建 resources 包并与 java 包置于同级中，然后创建项目配置beans.xml文件：
```xml
<?xml version="1.0" encoding="UTF-8"?>  
<beans xmlns="http://www.springframework.org/schema/beans"  
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"  
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">  
  
    <bean id="knight" class="jerriel.springlearning1.BraveKnight">  
        <constructor-arg ref="quest"/>  
    </bean> 
     
    <bean id="quest" class="jerriel.springlearning1.SlayDragonQuest">  
    </bean>
</beans>
```

> [!NOTE] Tips：
> 类中含有构造方法且未在配置中指明时，Spring使用的是类的无参构造
#### 2.1.3 FactoryBean
还可以使用FactoryBean来实例化对象：
```Java
// 骑士训练营
public class KnightCamp implements FactoryBean {
	private final Quest quest;  
  
	public KnightCamp(Quest quest) {  
	    this.quest = quest;  
	} 
	
	// 获取实例对象 
    @Override  
    public Object getObject() throws Exception {  
        return new BraveKnight(null);  
    }  
  
	// 获取对象类型
    @Override  
    public Class<?> getObjectType() {  
        return Knight.class;  
    }  
  
	// 是否为单例
    @Override  
    public boolean isSingleton() {  
        return FactoryBean.super.isSingleton();  
    }  
}
```
使用这种方式必须要实现FactoryBean接口中的方法，其中`getObject()`用于获取实例对象，`getObjectType()`用于获取对象类型，而`isSingleton()`则控制对象是否为单例（通常情况下使用的都是单例Bean，因此这个方法可以不写）。

使用这种方法，配置文件中应指派对应的类：
```xml
<bean id="knight" class="jerriel.springlearning1.KnightCamp">  
    <constructor-arg ref="quest"/>  
</bean>
```
运行看看：
```Java
public class SpringLearning1Application {  
    public static void main(String[] args) {
	    // 由于我们选择XML文件配置Bean，故采用ClassPathXmlApplicationContext作为应用上下文
        ClassPathXmlApplicationContext context = new ClassPathXmlApplicationContext("beans.xml");  
        BraveKnight knight = (BraveKnight) context.getBean("knight");  
  
        knight.embarkOnQuest();  
    }  
}
```
![运行结果](https://pic-1371809842.cos.ap-chengdu.myqcloud.com/PicGo/20251015183056813.webp?imageSlim)
