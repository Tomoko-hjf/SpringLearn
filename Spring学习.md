#  IOC容器

## bean装配方法

**方法一：**通过bean工厂调用类属性的set方法进行注入，所以要注入的bean必须要实现属性的set方法；

在配置文件中指定value和ref就是调用set方法注入的；

**方式二：**通过构造器方式注入

**方式三：**使用工厂bean实例化对象

```xml
<bean id="serviceLocator" class="examples.DefaultServiceLocator">
    <!-- inject any dependencies required by this locator bean -->
</bean>

<bean id="clientService"
    factory-bean="serviceLocator"
    factory-method="createClientServiceInstance"/>

<bean id="accountService"
    factory-bean="serviceLocator"
    factory-method="createAccountServiceInstance"/>
```

```java
public class DefaultServiceLocator {

    private static ClientService clientService = new ClientServiceImpl();

    private static AccountService accountService = new AccountServiceImpl();

    public ClientService createClientServiceInstance() {
        return clientService;
    }

    public AccountService createAccountServiceInstance() {
        return accountService;
    }
}
```







## bean注入时配置文件

```xml
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
        https://www.springframework.org/schema/beans/spring-beans.xsd">

    <bean id="..." class="...">  
        <!-- collaborators and configuration for this bean go here -->
    </bean>


</beans>
```

## bean的属性

id 指示类的别名；

class 指示类；

如果一个类是内部类，配置时可以这样写： `com.example.SomeThing$OtherThing`. 其中Something是外部类，otherThing是内部类；

value 注入属性值，如基本类型和String；

ref 指示所注入的bean；

## 串联多个配置文件

当bean对象比较多时，可以为每一层设置一个配置文件，然后通过resource标签将多个配置文件串联起来；

```xml
<!--引入别的配置文件-->
<import resource="dao.xml"/>
```



## 容器和bean对象的获取

在spring中bean主要是由BeanFactory接口和ApplicationContext接口下的类完成注入的，如果要获取已注入的对象，首先要实例化一个ApplicationContext对象，即容器，不同的注入方法对获取对象容器有不同的方式：

### xml配置文件方式注入：

```java
ApplicationContext context = new ClassPathXmlApplicationContext("service.xml");
UserService userService = (UserService) context.getBean("userService");
```

## bean生命周期

![1613221722427](C:\Users\DELL\AppData\Roaming\Typora\typora-user-images\1613221722427.png)

## 2.1 Spring配置的可选方案

+ XML显示配置
+ Java中显式配置
+ 利用注解自动装配

## 2.2 自动化装配bean

### 创建bean

bean上标记@Component注解

### 为组件扫描的bean命名

```
@Component(" ") ：相当于XML中的id

package com.hejianfei;

import org.springframework.stereotype.Component;

@Component("sgtPeppers")
public class SgtPeppers implements CompactDisc{
    private String title = "我爱你中国";
    private String artist = "汪峰";

    @Override
    public String toString() {
        return "SgtPeppers{" +
                "title='" + title + '\'' +
                ", artist='" + artist + '\'' +
                '}';
    }

    @Override
    public void play() {
        System.out.println("歌曲" + title + "歌手" + artist);
    }
}
```

```
@Named(" ") :另一种方式
```



### 开启注解扫描：

**方式一**：通过XMl启用组件扫描

```
<context:component-scan base-package=" "
```

**方式二**：利用Java配置，主要是为了扫描所有的bean

```java
package com.hejianfei;

import org.springframework.context.annotation.ComponentScan;
import org.springframework.context.annotation.Configuration;

@Configuration
@ComponentScan(basePackages = {"com.hejianfei"})
public class CDPlayerConfig {

}
```

### 设置组件扫描的基础包

```java
@Configuration
@ComponentScan(basePackage = {"com.hejianfei", " "})//通过设置包
@ComponentScan(basePackageClasses = {CD.class, DVD.class})
```

### 通过为bean添加注解实现自动装配

@AutoWired注解不仅能用在构造器上，还能用在属性的Setter方法上；



实际上@AutoWired注解可以用在类的任何方法上，不管是构造器，Setter方法还是其他方法，Spring都会尝试满足方法参数上所声明的依赖；

```java
package com.hejianfei;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Component;

@Component
public class CDPlayer {

    private CompactDisc cd;

    @Autowired(required = false)
    public CDPlayer(CompactDisc cd) {
        this.cd = cd;
    }

    @Autowired
    public void setCd(CompactDisc cd) {
        this.cd = cd;
    }

    @Override
    public String toString() {
        return "CDPlayer{" +
                "cd=" + cd +
                '}';
    }
}

```

## 2.3 通过Java代码装配bean

通常情况下，使用注解@Component和@AutoWired是比较好的选择，但是有的情况下我们必须进行显式扫描（如当类是插件中的时）；

### 创建配置类

配置类一般要放在一个单独的包下，用来说明他和其他的bean没有什么关系

```java
package com.hejianfei;

import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.ComponentScan;
import org.springframework.context.annotation.Configuration;

//一定要记得声明配置类注解
@Configuration
//@ComponentScan(basePackages = {"com.hejianfei"})
public class CDPlayerConfig {
    @Bean(name = "sgtPeppers")
    public CompactDisc sgtPeppers() {
        return new SgtPeppers();
    }
    //使用构造器注入
    @Bean
    public CDPlayer cdPlayer() {
        return new CDPlayer(sgtPeppers());
    }
    //使用自动装配，只要上面被装配过，下面就能一直使用，且对象是同一个
    public CDPlayer cdPlayer2(CompactDisc cd) {
        return  new CDPlayer(cd);
    }
}

```

**注意：**默认情况下，bean的ID与带有@Bean注解的方法名是一样的，当然也可以通过设置name改变id名

## 2.4 通过XML装配Bean

### 创建XML配置规范

```xml
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
        https://www.springframework.org/schema/beans/spring-beans.xsd">

    <bean id="..." class="...">  
        <!-- collaborators and configuration for this bean go here -->
    </bean>


</beans>
```

### 装配Bean

通过property标签设置的属性值，都是通过调用该类的set方法设置的，所以类必须有相应的set方法

```xml
<bean id="sgtPeppers" class="com.hejianfei.SgtPeppers">    <property name="title" value="我爱你中国"/>
</bean>
```

### 构造器注入bean引用

使用构造器注入时，该类必须有相应的构造方法

```xml
<bean id="sgtPeppers" class="com.hejianfei.SgtPeppers">    </bean>
<bean id="cdPlayer" class="com.hejianfei.CDPlayer">    <constructor-arg ref="sgtPeppers"/>//注入引用
 <constructor-arg value="sgtPeppers"/>//注入值
</bean>
```



## 2.5 总结

### bean的声明，创建，注入

![1613307012836](C:\Users\DELL\AppData\Roaming\Typora\typora-user-images\1613307012836.png)

### 同时使用XML和Java配置类实现bean的扫描创建

使用@Import(classes)和@ImportResource("classPath:*.xml")

### 好的习惯

不管使用 JavaConfig 还是使用 XML 进行装配，通常都会创建一个根配置（ root configuration ），这个配置会将两个或更多的装配类和 / 或 XML 文件组合起来。同时也会在根配置中启用组件扫描（通过 <context:component-scan> 或 @ComponentScan ）。

# 高级装配

## 3.1 环境与Profile

简而言之：同一个bean在不同的环境下需要创建不同版本的bean

### 配置 profile bean

* 在Java配置类或方法上使用注解@Profile("dev")来指定相应的Profile；

* 在xml中配置profile：

  ```xml
  <beans profile = "dev">
  	<bean id="" class="">
      </bean>
  </beans>
  ```

  

* 只有在相应Profile激活的情况下，相应的bean才会被实例化；

### 激活profile

* 作为DispatcherServlet的初始化参数

  ```xml
  <context-param>
  	<param-name>spring.profile.default</param-name>
      <param-value>dev</param-value>
  </context-param>
  ```

* 在集成测试上类上，使用@ActiveProfiles注解

  ```xml
  @RunWith(SpringJUnit4ClassRunner.class)
  @ContextConfiguration(classes = SystemConfig.class)
  @ActiveProfiles("dev")
  ```

## 3.2 条件化的bean

### 实现方法

通过@Conditional(class)注解实现，设置给 @Conditional 的类可以是任意实现了 Condition 接口的类型。可以看出来，这个接口实现起来很简单直接，只需提
供 matches() 方法的实现即可。如果 matches() 方法返回 true ，那么就会创建带有 @Conditional 注解的 bean 。如果 matches() 方法返
回 false ，将不会创建这些 bean 。

## 3.3 处理自动装配的歧义性

当不同的类实现了同一个接口就可能会在自动注入时产生歧义性，如下面这个例子：

```java
package com.hejianfei.qiyixing;

public interface Dessert {

}

package com.hejianfei.qiyixing;

public class Cake implements Dessert{

}

package com.hejianfei.qiyixing;
public class Cookies implements Dessert {
    
}
```

### 标示首选的bean

* 可以在配置类@Bean注解下使用

  ```java
  @Configuration
  @ComponentScan(basePackages = "com.hejianfei.qiyixing")
  public class qiyixingConfig {
      @Bean
      @Primary
      public Cake cake() {
          return new Cake();
      }
  
      @Bean
      public Cookies cookies() {
          return new Cookies();
      }
  
  }
  ```

* 可以在@Component注解下使用

  ```java
  @Component
  @Primary
  public class Cake implements Dessert{
      @Override
      public void sayHello() {
          System.out.println("我是蛋糕");
      }
  }
  ```

* 在xml中使用primary属性

  ```xml
  <bean id = "iceCream" class = "" primary = "true"/>
  ```

**注意**

> 如果多个bean都使用primary，那么将失去效果

### 限定自动装配的bean

@Qualifier 注解是使用限定符的主要方式。它可以与 @Autowired 和 @Inject 协同使用，在注入的时候指定想要注入进去的是哪个 bean 