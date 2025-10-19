### 1. 创建容器
- 方法一：类路径创建
```Java
ApplicationContext context = new ClassPathXmlApplicationContext("Beans.xml");
```
- 方法二：文件路径创建
```Java
ApplicationContext context = new FileSystemXmlApplicationContext("D:\\Beans.xml");
```
函数接收多个参数值，因此可以同时加载多个参数。

`ApplicationContext`接口是Spring容器的核心接口，初始化时bean立即加载。
`BeanFactory`是IoC容器的顶层接口，初始化BeanFactory对象时，加载的bean延迟加载。


### 2. 获取Bean的方式
- 方式一：使用bean名称获取
```Java
BookDao bookDao=(BookDao) ctx.getBean("bookDao");
```
- 方式二：使用bean名称获取并指定类型
```Java
BookDao bookDao= ctx.getBean("bookDao",BookDao.class);
```
- 方式三：使用bean类型获取
```Java
BookDao bookDao= ctx.getBean(BookDao.class);
```