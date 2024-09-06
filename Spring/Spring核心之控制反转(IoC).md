

进一步解读Ioc的含义以及IoC的使用方式

# 大纲
* 引入
* 如何理解IoC
    * Spring Bean是什么
    * Ioc是什么
    * IoC和DI（依赖注入）是什么关系
* IoC三种配置方式
    * xml配置
    * java配置
    * 注解配置
* 依赖注入的三种方式
    * setter方式
    * 构造函数注入
    * 注解注入
* Ioc使用小结
    * 为什么推荐构造器注入
    * Bas Smell 怎么办
    * @Autowired @Resource @Inject注解注入的区别


# 引入

由用户管理Bean变为框架管理Bean，称为**IoC**

Bean的存放，**IoC Container**

Bean的配置（定义信息），xml，java，注解 三种方式

Bean生命周期管理

程序从容器中获取Bean并使用，称为依赖注入（DI）

依赖注入的方式：三种注解、Bean之间循环依赖


# 如何理解IOC

把创建和查找依赖对象的控制权交给了容器，由容器进行注入组合对象，对象之间是松耦合，便于测试，利于功能复用，使得程序体系结构更加灵活。

# IoC的三种配置方式


## xml 方式

将bean配置信息放在xml文件中，Spring加载文件来创建bean，主要用于早期SSM项目中。

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
 http://www.springframework.org/schema/beans/spring-beans.xsd">
    <!-- services -->
    <bean id="userService" class="tech.pdai.springframework.service.UserServiceImpl">
        <property name="userDao" ref="userDao"/>
        <!-- additional collaborators and configuration for this bean go here -->
    </bean>
    <!-- more bean definitions for services go here -->
</beans>
```

## java配置

将Bean的创建交给配置的JavaConfig类来完成，Spring负责维护和管理。

1. 创建要给Java类，写上`@Configuration`注解
2. 创建方法，写上`@Bean`注解，用于创建实例并返回，交给Spring容器管理，实例类不需要任何注解。

## 注解配置

在实例类上加注解，声明一个类交给Spring管理，常用的有`@Component`,`@Controller`,`@Service`,`@Repository`，需要提前配置Spring的注解扫描器。

缺点是：无法用于第三方资源，因为第三方代码我们无法加注解。



# 依赖注入的三种方式

## setter方式

* xml配置中，setter方式体现为 `property`

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
 http://www.springframework.org/schema/beans/spring-beans.xsd">
    <!-- services -->
    <bean id="userService" class="tech.pdai.springframework.service.UserServiceImpl">
        <property name="userDao" ref="userDao"/>
        <!-- additional collaborators and configuration for this bean go here -->
    </bean>
    <!-- more bean definitions for services go here -->
</beans>
```
1. 对象类需要写默认构造方法
2. 需要在对象类中写setXX方法

* 注解和java配置方法类似，但是对象类中都要写setXX方法

对象类的结构：

```java
public class UserServiceImpl {

    /**
     * user dao impl.
     */
    private UserDaoImpl userDao;

    /**
     * find user list.
     *
     * @return user list
     */
    public List<User> findUserList() {
        return this.userDao.findUserList();
    }

    /**
     * set dao.
     *
     * @param userDao user dao
     */
    @Autowired
    public void setUserDao(UserDaoImpl userDao) {
        this.userDao = userDao;
    }
}
```

## 构造函数注入

* xml配置中体现为 `<constructor-arg>`

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
 http://www.springframework.org/schema/beans/spring-beans.xsd">
    <!-- services -->
    <bean id="userService" class="tech.pdai.springframework.service.UserServiceImpl">
        <constructor-arg name="userDao" ref="userDao"/>
        <!-- additional collaborators and configuration for this bean go here -->
    </bean>
    <!-- more bean definitions for services go here -->
</beans>
```
本质上是创建对象时，传入参数，因此对象类需要实现 `new UserServiceImpl(UserDao)`这样的构造函数

为了防止被更改，这里对象类中的userDao注入对象设置为`final`。

* java和注解方式中，需要在构造函数上加 `@Autowired`

## 注解注入

在对象类的属性上加入`@Autowired`注解即可,(默认按照byType注入)

* constructor：通过构造方法进行自动注入，spring会匹配与构造方法参数类型一致的bean进行注入，如果有一个多参数的构造方法，一个只有一个参数的构造方法，在容器中查找到多个匹配多参数构造方法的bean，那么spring会优先将bean注入到多参数的构造方法中。

* byName：被注入bean的id名必须与set方法后半截匹配，并且id名称的第一个单词首字母必须小写，这一点与手动set注入有点不同。

* byType：查找所有的set方法，将符合符合参数类型的bean注入

也就是说这些注入方式好像会默认为对象类加入set方法。

```java
@Service
public class UserServiceImpl {

    /**
     * user dao impl.
     */
    @Autowired
    private UserDaoImpl userDao;

    /**
     * find user list.
     *
     * @return user list
     */
    public List<User> findUserList() {
        return userDao.findUserList();
    }

}
```

# 使用小结

* 推荐使用构造器注入，这个构造器注入的方式能够保证注入的组件不可变，并且确保需要的依赖不为空。

所以通常是这样的：
```java
 @Service
public class UserServiceImpl {

    /**
     * user dao impl.
     */
    private final UserDaoImpl userDao;

    /**
     * init.
     * @param userDaoImpl user dao impl
     */
    public UserServiceImpl(final UserDaoImpl userDaoImpl) {
        this.userDao = userDaoImpl;
    }

}
```

(但是感觉这样会需要手动创建很多构造函数，而且springboot中是使用注解在类的属性上注入的)

## 三种注解注入有何区别？

@Autowired和@Resource以及@Inject等注解注入有何区别？

默认是按类型注入的，可以配合`@Qualifier`实现按名字注入

可以用在字段、setXX方法、构造函数


**@Resource**

1. @Resource是JSR250规范的实现，在javax.annotation包下
2. @Resource可以作用TYPE、FIELD、METHOD上
3. @Resource是默认根据属性名称进行自动装配的，如果有多个类型一样的Bean候选者，则可以通过name进行指定进行注入


**@Inject**

与`@Autowired`类似

1、@Inject是JSR330 (Dependency Injection for Java)中的规范，需要导入javax.inject.Inject jar包 ，才能实现注入
2、@Inject可以作用CONSTRUCTOR、METHOD、FIELD上
3、@Inject是根据类型进行自动装配的，如果需要按名称进行装配，则需要配合@Named；
