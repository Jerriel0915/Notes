---
tags:
  - 后端
  - Java
  - SpringBoot
---
## jdbc
Maven pom.xml文件：
```Java
<dependency>  
    <groupId>org.springframework</groupId>  
    <artifactId>spring-jdbc</artifactId>  
    <version>6.2.10</version>  
</dependency>
<!-- 以 druid 作为daatsource -->
<dependency>  
    <groupId>com.alibaba</groupId>  
    <artifactId>druid</artifactId>  
    <version>1.2.26</version>  
</dependency>
```

jdbcConfig：
```Java
import com.alibaba.druid.pool.DruidDataSource;  
import org.springframework.beans.factory.annotation.Value;  
import org.springframework.context.annotation.Bean;  
import org.springframework.context.annotation.Configuration;  
import org.springframework.jdbc.datasource.DataSourceTransactionManager;  
import org.springframework.transaction.PlatformTransactionManager;  
  
import javax.sql.DataSource;  
  
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
  
  // 开启事务
    @Bean  
    public PlatformTransactionManager transactionManager() {  
        return new DataSourceTransactionManager(dataSource());  
    }  
}
```

## Mybatis
Maven pom.xml文件：
```xml
<dependency>  
    <groupId>com.baomidou</groupId>  
    <artifactId>mybatis-plus</artifactId>  
    <version>3.5.11</version>  
</dependency>  
<dependency>  
    <groupId>org.mybatis</groupId>  
    <artifactId>mybatis-spring</artifactId>  
    <version>3.0.4</version>  
</dependency>
```

MybatisConfig：
```Java
import org.mybatis.spring.SqlSessionFactoryBean;  
import org.mybatis.spring.mapper.MapperScannerConfigurer;  
import org.springframework.context.annotation.Bean;  
import org.springframework.context.annotation.Configuration;  
  
import javax.sql.DataSource;  
  
@Configuration  
public class MybatisConfig {  
  
    @Bean  
    public static SqlSessionFactoryBean getSqlSessionFactoryBean(DataSource dataSource) {  
        SqlSessionFactoryBean factoryBean = new SqlSessionFactoryBean();  
        //注入数据源  
        factoryBean.setDataSource(dataSource);  
        //注入mybatis核心配置文件路径  
  
        //创建mybatis配置项类  可以配置驼峰命名、日志实现、、、、  
        org.apache.ibatis.session.Configuration config = new org.apache.ibatis.session.Configuration();  
        return factoryBean;  
    }  
  
    /**  
     * Mapper扫描配置器  
     * 指定扫描的包  
     */  
    @Bean  
    public static MapperScannerConfigurer mapperScanner() {  
        //创建mapper扫描配置器  
        MapperScannerConfigurer configurer = new MapperScannerConfigurer();  
        configurer.setBasePackage("Transaction.Mapper");  
        return configurer;  
    }  
  
}
```

## SpringConfig
```Java
import org.springframework.context.annotation.*;  
import org.springframework.transaction.annotation.EnableTransactionManagement;  
  
@Configuration  
@ComponentScan("{packagename}")  
@PropertySource("classpath:jdbc.properties")  
@Import({jdbcConfig.class, MybatisConfig.class})  
@EnableTransactionManagement  
public class SpringConfig {  
}
```