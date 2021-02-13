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