---
layout: default
title: SpringBean
---

容器根据组件之间的依赖关系主动将依赖注入到组件中称为装配。

# 创建Bean
容器依据在Java配置、XML配置或注解等声明的Bean来创建Bean。
1. 在Java配置中声明Bean

```
@Bean
public ClassName beanId() {
	return new ClassName();
}
```

2. 在XML中声明Bean

```
<bean id="beanId" class="类路径"></bean>
```

3. 注解标注

```
@Component // 声明Bean
public class ClassName { ... }
```
开启自动扫描机制后，被@Compont或@Named注解标注的类将会被创建为Bean，Bean Id默认为首字母小写的类名。

## 根据条件创建Bean
在Spring4之后，Spring加入了根据条件创建Bean的功能；利用@Conditional注解即可指定Bean是否被创建。

```
@Bean
@Conditional(MyCondition.class)
public Employee employee() {
    return new Employee();
}
```
MyCondition是一个实现了org.springframework.context.annotation.Condition接口的类，Condition接口中只有一个matches方法，MyCondition实现这个方法后根据方法返回的值为true或false来决定是否创建Bean。

# 装配Bean
利用Java配置、XML配置或注解等标识Bean的依赖关系，容器在Bean创建时自动根据标识的依赖关系自动将依赖注入到Bean中。

1. 在Java配置中标识依赖关系

```
@Bean
public Account account() {
    return new Account();
}

// 通过创建Bean方法标识
@Bean
public Employee employee() {
    return new Employee(account());
}

// 通过Bean Id标识
@Bean
public Employee employee(Account account) {
    return new Employee(account);
}

or

@Bean
public Employee employee(Account account) {
    Employee employee = new Employee();
    employee.setAccount(account);
    return employee;
}

```
2. 在XML配置中标识依赖关系

```
<bean id="account" class="com.demo.Account" />

<!-- 通过构造函数引入 -->
<bean id="employee" class="com.demo.Employee">
    <constructor-arg ref="account"></constructor-arg>
</bean>

or

<!-- 通过属性值引入 -->
<bean id="employee" class="com.demo.Employee">
    <property name="account" ref="account"></property>
</bean>
```

3. 注解标识

```
@Component
public class Employee {

    @Autowired
    private Account account;
}

or

@Component
public class Employee {

    private Account account;
    
    @Autowired
    public void setAccount(Account account) {
        this.account = account;
    }
}
```
通过@Autowired注解或@Inject注解标识类的属性或Set方法即可将依赖注入到Bean中。

4. 注入固定值
在Java配置中给返回对象属性赋值即可将固定值注入到Bean中。

```
<bean id="account" class="com.demo.Account">
    <property name="属性名" value="属性值"></property>
</bean>

<!-- 通过构造函数注入 -->
<bean id="account" class="com.demo.Account">
    <constructor-arg value="属性值"></constructor-arg>
    ......
</bean>
```

## 自动装配的歧义性
当存在两个或多个Bean满足依赖需求时就产生了歧义；自动装配在遇到有多个满足的Bean时会抛出NoUniqueBeanDefinitionException异常；解决自动装配的歧义性有两种方法，指定首选Bean和限定自动装配的Bean。
1. 指定首选Bean
>Java Config

```
@Bean
@Primary
public Account account() {
    return new Account();
}
```
通过@Primary注解标识Bean方法即可将Bean标识为首选Bean。

>XML

```
<bean id="account" class="com.demo.Account" primary="true" />
```
在bean元素上指明属性primary的值为true即可指定Bean为首选Bean。

2. 限定自动装配的Bean
Spring可以利用@Qualifier注解限定Bean的选择范围；@Qualifier和@Autowired或@Inject注解协同使用，在注入的时候指定bean。

```
@Autowired
@Qualifier("限定符")
private Order order;
```
@Qualifier注解的值是指定bean的限定符；每个Bean都有一个默认的限定符，与BeanId一致；@Qualifier注解和@Component一起使用可以修改Bean的限定符。

```
@Component
@Qualifier("acc")
public class Account { ... }
```

# Bean的作用域
Spring应用上下文中默认所有的Bean都是以单例的形式创建的，不管一个Bean注入到其它Bean多少次，每次注入的都是同一个实例。
>Spring作用域
* 单例(Singleton)：在整个应用中，只创建bean的一个实例。
* 原型(Prototype)：每次注入或者通过Spring应用上下文获取的时候，都会创建一个新的bean实例。
* 会话(Session)：在Web应用中，为每个会话创建一个bean实例。
* 请求(Rquest)：在Web应用中，为每个请求创建一个bean实例。

1. 将Bean的作用域改为原型
> Java Config

```
@Bean
@Scope(ConfigurableBeanFactory.SCOPE_PROTOTYPE)
public Account account() {
	return new account();
}
```

> XML

```
<bean id="account" class="com.demo.Account" scope="prototype"></bean>
```

> 自动扫描

```
@Component
@Scope(ConfigurableBeanFactory.SCOPE_PROTOTYPE)
public class Account { ... }
```


