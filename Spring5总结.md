# IOC操作管理（基于xml方式）

#### 1. 基于xml方式创建对象

   ```html
   <bean id="user" class="com.spring.User"/>
   ```

   (1)在 spring配置文件中，使用bean标签，标签里面添加对应属性，就可以实现对象创建

   (2)在bean标签有很多属性，介绍常用的属性

   * id属性：指的是获取对象的唯一标识（相当于起个获取这个对象的别名），获得配置创建的对象时getBean("id", 类.class); id应一致
   * class属性：类全路径（包类路径）
   * name属性：与id效果一样，但是可以用特殊符号，基本不用这个属性

   (3) 创建对象时默认执行无参构造方法完成对象创建，如果声明了有参构造器，则要声明无参构造器

#### 2.基于xml方式注入属性

 ##### (1)DI：依赖注入，就是注入属性

   * 第一种注入方式：使用set方法进行注入

        1.  创建类，定义属性及对应的set方法

         ```java
         public class Book {
             private String bname;
             private String bauthor;
         
             public void setBname(String bname) {
                 this.bname = bname;
             }
         
             public void setBauthor(String bauthor) {
                 this.bauthor = bauthor;
             }
         
             public static void main(String[] args) {
                 Book book = new Book();
             }
         }
         ```

        2. 在 spring配置文件配置对象创建，配置属性注入

         ```html
         <bean id="book" class="com.spring.Book">
             <property name="bname" value="易筋经"/>
             <!-- name：类里面属性名称
                  value：向属性注入的值
             -->
             <property name="bauthor" value="达摩老祖"/>
         </bean>
         ```

        3. 完成测试

        ```java
        @Test
        public void testBook1() {
        /*1.加载spring配置文件，ApplicationContext在加载的时候就会创建对象。BeamFactory在加载配置文件时候不会创建对象，在获取对象（使用）才去创建对象
        **/
            ApplicationContext context = new ClassPathXmlApplicationContext("beans1.xml");
            //2.获取配置创建的对象
            Book book = context.getBean("book", Book.class);   //"book"与xml配置文件中的id属性值一致
            System.out.println(book);
        }
        ```

   * 第二种注入方式：有参构造器注入

     1. 创建类，定义属性，创建属性对应有参构造方法

        ```java
        public class Orders {
            private String oname;
            private String address;
        
            public Orders(String oname, String address) {
                this.oname = oname;
                this.address = address;
            }
        }
        ```

     2. 在 spring配置文件配置对象创建，配置属性注入

        ```html
        <bean id="orders" class="com.spring.Orders">
            <constructor-arg name="oname" value="电脑"/>
            <constructor-arg name="address" value="China"/>
            <!--
                0，1分别代表有参构造器的第一个参数和第二个参数
                <constructor-arg index="0" value="电脑"/>
                <constructor-arg index="1" value="China"/>
        -->
        </bean>
        ```

     3. 测试
     
        

##### (2)xml注入其他类型属性

   1.字面量（属性的固定值）

   * null值(xml配置文件如下)

     ```html
     <property name="address">
         <null/>
     </property>
     ```

   * 属性值包含特殊符号 (用<<>>举例)

     ```html
     <!--
         把<>进行转义，使用&lt;&gt;
         或者把带特殊符号内容写道CDATA
     -->
     <property name="address" >
         <value><![CDATA[<<南京>>]]></value>
     </property>
     ```

##### (3) 注入属性-外部bean

1. 创建两个类service类和dao类，并在Service调用dao里面方法

   ```java
   package com.spring.dao;
   
   public interface UserDao {
       public void update();
   }
   ```

   ```java
   package com.spring.dao;
   
   public class UserDaoImpl implements UserDao{
       public void update(){
           System.out.println("dao update........");
       }
   }
   ```

   ```java
   package com.spring.service;
   
   import com.spring.dao.UserDao;
   
   public class UserService {
       private UserDao userDao;
   
       public void setUserDao(UserDao userDao) {
           this.userDao = userDao;
       }
   
       public void add() {
           System.out.println("service add ......");
           userDao.update();
       }
   }
   ```

2. 在xml配置文件中进行配置

   ```html
   <bean id="UserService" class="com.spring.service.UserService">
       <property name="userDao" ref="userDaoImpl"/> <!--ref引用外部beam -->
   </bean>
   <bean id="userDaoImpl" class="com.spring.dao.UserDaoImpl"/>
   ```

3. 测试

   ```java
   @Test
   public void testUserService() {
       ApplicationContext context = new ClassPathXmlApplicationContext("bean2.xml");
       UserService userService = context.getBean("UserService", UserService.class);
       userService.add();
   }
   ```

##### (4) 注入属性-内部bean和级联赋值

   1. 一对多关系：部门和员工

      ```java
      package com.spring.bean;
      
      public class Dept {
          private String name;
      
          public void setName(String name) {
              this.name = name;
          }
      
          public String getName() {
              return name;
          }
      }
      ```

      ```java
      package com.spring.bean;
      
      public class Emp {
          private String name;
          private String gender;
          private Dept dept;
      
          public void setName(String name) {
              this.name = name;
          }
      
          public void setGender(String gender) {
              this.gender = gender;
          }
      
          public void setDept(Dept dept) {
              this.dept = dept;
          }
      
          @Override
          public String toString() {
              return "Emp{" +
                      "name='" + name + '\'' +
                      ", gender='" + gender + '\'' +
                      ", dept_name=" + dept.getName()+
                      '}';
          }
      }
      ```

2. xml配置

   方式一

   ```html
   <bean id="emp" class="com.spring.bean.Emp">
       <property name="gender" value="女"/>
       <property name="name" value="Janine"/>
       <property name="dept">
           <bean id="dept" class="com.spring.bean.Dept">
               <property name="name" value="总裁"/>
           </bean>
       </property>
   </bean>
   ```

   方式二

   ```html
   <bean id="Emp" class="com.spring.bean.Emp">
       <property name="name" value="Janine"/>
       <property name="gender" value="女"/>
       <property name="dept" ref="dept"/>
   </bean>
   
   <bean id="dept" class="com.spring.bean.Dept">
       <property name="name" value="总裁"/>
   </bean>
   ```

   方式三（注：Dept类的name属性是private修饰，因此使用方式三时要添加name的Getter，才可以完成。）

   ```html
   <bean id="Emp" class="com.spring.bean.Emp">
       <property name="name" value="Janine"/>
       <property name="gender" value="女"/>
       <property name="dept.name" value="总裁"/>
   </bean>
   ```

3. 测试

   ```java
   @Test
   public void testEmp() {
       ApplicationContext context = new ClassPathXmlApplicationContext("bean3.xml");
       Emp emp = context.getBean("emp", Emp.class);
       System.out.println(emp);
   }
   ```


##### (5)xml注入集合属性

1. 注入数组类型属性

2. 注入List集合类型属性

3. 注入Map集合类型属性

   (1)创建类及相关属性

   ```java
   package com.spring5.collectiontype;
   
   import java.util.List;
   import java.util.Map;
   import java.util.Set;
   
   public class Stu {
       private String[] courses;
       private List<String> list;
       private Map<String, String> maps;
       private Set<String> sets;
   
       public void setCourses(String[] courses) {
           this.courses = courses;
       }
   
       public void setList(List<String> list) {
           this.list = list;
       }
   
       public void setMaps(Map<String, String> maps) {
           this.maps = maps;
       }
   
       public void setSets(Set<String> sets) {
           this.sets = sets;
       }
   }
   
   ```

   (2)在Spring配置文件中进行配置

   ```html
   <bean id="stu" class="com.spring5.collectiontype.Stu">
       <!--数组类型属性注入-->
       <property name="courses">
           <array>
               <value>java课程</value>
               <value>数据库课程</value>
           </array>
       </property>
       <!--list类型属性注入-->
       <property name="list">
           <list>
               <value>张三</value>
               <value>李四</value>
           </list>
       </property>
       <!--map类型属性注入-->
       <property name="maps">
           <map>
               <entry key="JAVA" value="java"/>
               <entry key="PHP" value="php"/>
           </map>
       </property>
       <!--set类型属性注入-->
       <property name="sets">
           <set>
               <value>MySql</value>
               <value>Redis</value>
           </set>
       </property>
   </bean>
   ```

   (3) 在集合里面设置对象类型值

   ```html
   <property name="courseList">
       <list>
           <ref bean="course1"/>
           <ref bean="course2"/>
       </list>
   </property>
   ```

   ```html
   <bean id="course1" class="com.spring5.collectiontype.testdemo.Course">
       <property name="course_name" value="Spring5"/>
   </bean>
   <bean id="course2" class="com.spring5.collectiontype.testdemo.Course">
       <property name="course_name" value="MyBatis"/>
   </bean>
   ```

   （4）把集合注入部分提取出来

     1. 在Spring配置文件中引入名称空间util

        ```html
        <?xml version="1.0" encoding="UTF-8"?>
        <beans xmlns="http://www.springframework.org/schema/beans"
               xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
               xmlns:util="http://www.springframework.org/schema/util"
               xsi:schemaLocation="http://www.springframework.org/schema/beans
                                   http://www.springframework.org/schema/beans/spring-beans.xsd
                                   http://www.springframework.org/schema/util
                                   http://www.springframework.org/schema/util/spring-util.xsd">
        </beans>
        ```

   2. 使用util标签完成list集合注入提取

      ```html
      <util:list id="bookList">
          <value>易筋经</value>
          <value>九阴真经</value>
          <value>九阳神功</value>
      </util:list>
      
      <bean id="book" class="com.spring5.collectiontype.Book">
          <property name="list_book" ref="bookList"/>
      </bean>
      ```
      
      

## IOC操作管理（FactoryBean）

### 1. 概念

​	Spring有两种类型bean,一种普通bean，另外一种工厂bean （FactoryBean）

​	普通 bean：在配置文件中定义bean类型就是返回类型

​	工厂bean：在配置文件定义bean 类型可以和返回类型不一样

### 2. 工厂bean实现方法

​	第一步创建类，让这个类作为工厂bean，实现接口FactoryBean

​	第二步实现接口里面的方法，在实现的方法中定义返回的bean类型

```java
package com.spring5.collectiontype.factorybean;

import com.spring5.collectiontype.Book;
import org.springframework.beans.factory.FactoryBean;

public class MyBean implements FactoryBean<Book> {
// 下面这个函数定义了返回bean的类型
    @Override
    public Book getObject() throws Exception {
        Book book = new Book();
        return book;
    }

    @Override
    public Class<?> getObjectType() {
        return null;
    }

    @Override
    public boolean isSingleton() {
        return FactoryBean.super.isSingleton();
    }
}
```

```html
<bean id="myBean" class="com.spring5.collectiontype.factorybean.MyBean"/>
```

```java
@Test
public void testFactoryBean() {
    ApplicationContext context = new ClassPathXmlApplicationContext("bean3.xml");
    Book book = context.getBean("myBean", Book.class);
    System.out.println(book);
}
```



## IOC操作Bean管理（bean作用域）

### 概念

1. 在 Spring 里面，设置创建bean实例是单实例还是多实例
2. 在Spring里面，默认情况，bean是单实例对象

### 设置单实例和多实例

(1）在 spring配置文件bean标签里面有属性（scope）用于设置单实例还是多实例

(2） scope属性值
第一个值默认值，singleton，表示是单实例对象
第二个值prototype，表示是多实例对象

```html
<bean id="book" class="com.spring5.collectiontype.Book" scope="prototype">
    <property name="list_book" ref="bookList"/>
</bean>
```

```java
@Test
public void testBook() {
    ApplicationContext context = new ClassPathXmlApplicationContext("bean2.xml");
    Book book1 = context.getBean("book", Book.class);
    Book book2 = context.getBean("book", Book.class);
    System.out.println(book1);
    System.out.println(book2);
}
```

![image-20230307105153598](C:\Users\Alvin\AppData\Roaming\Typora\typora-user-images\image-20230307105153598.png)

(3)singleton 和 prototype区别

第一 singleton 单实例，prototype多实例

第二设置 scope值是 singleton时候，加载 spring配置文件时候就会创建单实例对象(饿汉式)

设置 scope值是prototype时候，不是在加载 spring 配置文件时候创建对象，在调用getBean方法时候创建多实例对象（懒汉式）





## IOC操作Bean管理（bean生命周期）

### 		1.概念

​					1.生命周期：从对象创建到对象销毁的过程

​					2.**bean生命周期**

​							(1)通过构造器创建bean实例（无参数构造)

​							(2)为 bean的属性设置值和对其他bean引用（调用 set方法）

​							(3)调用 bean的初始化的方法（需要进行配置初始化的方法）

​						  （4）bean可以使用了对象获取到了

​						  （5）当容器关闭时候，调用bean的销毁的方法（需要进行配置销毁的方法）

### 		2.演示生命周期操作

​				

```java
package com.spring5.collectiontype.bean;

public class Orders {
    private String oname;

    //无参构造器
    public Orders() {
        System.out.println("第一步，执行无参数构造创建bean实列");
    }

    public void setOname(String oname) {
        this.oname = oname;
        System.out.println("第二步，调用Set方法");
    }

    //创建执行的初始化方法
    public void initMethod() {
        System.out.println("第三步执行初始化方法");
    }

    //创建执行的销毁方法
    public void destroyMethod(){
        System.out.println("第五步执行销毁方法");
    }

}
```

```html
<bean id="orders" class="com.spring5.collectiontype.bean.Orders" init-method="initMethod" destroy-method="destroyMethod">
    <property name="oname" value="电脑"/>
</bean>
```

```java
@Test
public void testOrders() {
    ClassPathXmlApplicationContext context = new ClassPathXmlApplicationContext("bean4.xml");
    Orders orders = context.getBean("orders", Orders.class);
    System.out.println("第四步获取到了对象");
    System.out.println(orders);

    //手动让bean实例销毁
    context.close();
}
```

###  3.加上bean的后置处理器， bean的生命周期一共有七步

​			(1)通过构造器创建bean实例（无参数构造)

​			(2)为 bean的属性设置值和对其他bean引用（调用 set方法）

​			(3)把bean实例传递bean后置处理器的方法

​			(4)调用 bean的初始化的方法（需要进行配置初始化的方法）

​			(5)把bean实例传递bean后置处理器的方法

​		 （6）bean可以使用了对象获取到了

​		（7）当容器关闭时候，调用bean的销毁的方法（需要进行配置销毁的方法）

### 4. 演示添加bean的后置处理器效果

​         (1)创建类,实现接口 BeanPostProcessor，创建后置处理器

```java
package com.spring5.collectiontype.bean;

import org.springframework.beans.BeansException;
import org.springframework.beans.factory.config.BeanPostProcessor;
import org.springframework.lang.Nullable;

public class MyBeanPost implements BeanPostProcessor {
    @Override
    public Object postProcessBeforeInitialization(Object bean, String beanName) throws BeansException {
        System.out.println("初始化之前执行的方法");
        return bean;
    }

    @Override
    public Object postProcessAfterInitialization(Object bean, String beanName) throws BeansException {
        System.out.println("初始化之后执行的方法");
        return bean;
    }
}
```

```html
<bean id="orders" class="com.spring5.collectiontype.bean.Orders" init-method="initMethod" destroy-method="destroyMethod">
    <property name="oname" value="电脑"/>
</bean>

<bean id="myBeanPost" class="com.spring5.collectiontype.bean.MyBeanPost"/>
```

配置好以后，每个对象都会添加后置处理器

![image-20230307114140420](C:\Users\Alvin\AppData\Roaming\Typora\typora-user-images\image-20230307114140420.png)

## IOC操作管理Bean(xml自动装配)

### 1. 根据指定装配规则（属性名称或者属性类型）Spring自动将匹配的属性值注入			

### 2.演示自动装配过程

```html
<!--实现自动装配
    bean标签属性autowire，配置自动装配
    autowire属性常用两个值:byMame根据属性名称注入     注入值bean的id值和类属性名称一样
                        byType根据属性类型注入
-->
<bean id="emp" class="com.spring5.collectiontype.autowire.Emp" autowire="byType">
</bean>

<bean id="dept" class="com.spring5.collectiontype.autowire.Dept"/>
```

## IOC操作Bean管理（外部属性文件）

### 1. 直接配置数据库信息

​	配置德鲁伊连接池，引入druid的jar包

```html
<bean id="dataSource" class="com.alibaba.druid.pool.DruidDataSource">
    <property name="driverClassName" value="com.mysql.jdbc.Driver"/>
    <property name="url" value="jdbc:mysql://localhost:3306/atguigu"/>
    <property name="username" value="root"/>
    <property name="password" value="123456"/>
</bean>
```

### 2. 引入外部属性文件配置数据库连接池

```properties
driverClassName=com.mysql.jdbc.Driver
url=jdbc:mysql://localhost:3306/atguigu
username=root
password=123456
```

 把外部properties属性文件引入到spring配置文件中

 * 引入context名称空间

   ```html
   <beans xmlns="http://www.springframework.org/schema/beans"
          xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
          xmlns:context="http://www.springframework.org/schema/context"
          xsi:schemaLocation="http://www.springframework.org/schema/beans
                              http://www.springframework.org/schema/beans/spring-beans.xsd
                              http://www.springframework.org/schema/context       
                              http://www.springframework.org/schema/context/spring-context.xsd">
   ```

* 在Spring配置文件中使用标签引入外部属性文件

  **注意system-properties-mode标签，在不配置的时候默认是ENVIROMENT，也就是会先检测系统路径，再检测本地路径。而系统路径中有username属性（既电脑系统用户），所以会先引入电脑系统用户名，导致项目本地配置的用户名没有导入成功，访问数据库失败。所以应该将该标签属性值配置为NEVER或FALLLBACK**

  ```html
  <context:property-placeholder location="classpath:jdbc.properties" system-properties-mode="NEVER"/>
  <bean id="dataSource" class="com.alibaba.druid.pool.DruidDataSource">
      <property name="driverClassName" value="${driverClassName}"/>
      <property name="url" value="${url}"/>
      <property name="username" value="${username}"/>
      <property name="password" value="${password}"/>
  </bean>
  ```

# IOC操作Bean管理（注解方式）

## Spring 针对Bean管理中创建对象提供注解

### 1.相关注解

#### 				@Component

#### 				@Service

####   			  @Controller

####             @Repository

上面四个注解的功能一样，都可以用来创建bean实例

### 2. 基于注解的创建对象演示

第一步：引入依赖spring-aop.jar

第二步：开启组件扫描

```html
<?xml version="1.0" encoding="UTF-8"?>
<!--引入context名称空间 -->
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd
                           http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context.xsd">
    <!--开启组件扫描
        1.如果扫描多个包，多个包使用逗号隔开
        2.或者扫描包的上层目录
    -->
    <context:component-scan base-package="com.spring5.testdemo"/>
</beans>
```

```java
@Component(value = "userService")  //<bean id="userService" class="..."/>
public class UserService {
    public void add() {
        System.out.println("Service add.......");
    }
```

```java
@Test
public void testService() {
    ApplicationContext context = new ClassPathXmlApplicationContext("bean1.xml");
    UserService userService = context.getBean("userService", UserService.class);
    System.out.println(userService);
    userService.add();
}
```

组件扫描配置细节

```html
<!--
    use-default-filters="false" 表示现在不使用默认的filter，自己配置filter
    context:include-filter，设置扫描哪些内容
    context:exclude-filter，设置哪些内容不进行扫描
-->
<context:component-scan base-package="com.spring5">
    <context:include-filter type="annotation" expression="org.springframework.stereotype.Controller"/>
    <context:exclude-filter type="annotation" expression="org.springframework.stereotype.Controller"/>
</context:component-scan>
```

## 基于注解方式实现属性注入

### 1.相关注解

####                     1.@Autowired：根据属性类别进行自动装配  

​							第一步吧service和dao对象创建，在service和dao类添加创建对象注解

```java
@Repository(value = "userDaoImpl")
public class UserDaoImpl implements UserDao{

    @Override
    public void add() {
        System.out.println("dap add......");
    }
}
```

​							第二步在service注入dao对象，在service类添加dao类型属性，在属性上面使用注解

```java
@Service(value = "userService")  //<bean id="userService" class="..."/>
public class UserService {
    @Autowired
    private UserDao userDao;

    public void add() {
        System.out.println("Service add.......");
        userDao.add();
    }
}
```

​						第三步测试

```java
@Test
public void testService() {
    ApplicationContext context = new ClassPathXmlApplicationContext("bean1.xml");
    UserService userService = context.getBean("userService", UserService.class);
    System.out.println(userService);
    userService.add();
}
```

####                    2.@Qualifier：根据属性名称进行注入

​						这个注解要和@Auowired一起使用。因为根据类型进行注入，一个接口可能有很多个实现类，所以搭配@Qualifier(value="")来确定注入哪个类型的属性，value后面跟的那个是创建对象注解中的value值。

```java
@Service(value = "userService")  //<bean id="userService" class="..."/>
public class UserService {
    @Autowired
    @Qualifier(value = "userDaoImpl")
    private UserDao userDao;

    public void add() {
        System.out.println("Service add.......");
        userDao.add();
    }
```

####                    3. @Resource：可以根据类型注入，可以根据名称注入

####                   4. @Value：注入普通类型属性

```java
@Value(value = "abc")
private String name;
```



## 完全注解开发

### 1. 创建配置类，替代xml配置文件

```java
@Configuration     //作为配置类，替代xml文件
@ComponentScan(basePackages = {"com.spring5"})
public class SpringConfig {
    
}
```

### 2. 测试

```java
@Test
public void testService2() {
    ApplicationContext context = new AnnotationConfigApplicationContext(SpringConfig.class);
    UserService userService = context.getBean("userService", UserService.class);
    System.out.println(userService);
    userService.add();
}
```

# AOP(底层原理)

## 1. 概念

![image-20230308152253409](C:\Users\Alvin\AppData\Roaming\Typora\typora-user-images\image-20230308152253409.png)

第二种 没有接口情况， 使用CGLIB动态代理

	* 创建子类代理对象，增强类的方法

![image-20230308152641144](C:\Users\Alvin\AppData\Roaming\Typora\typora-user-images\image-20230308152641144.png)

## 2.动态代理实现

0. java中动态代理的实现方式（代码原理）

   ```java
   package DongTaiDaiLi;
   
   
   import java.lang.reflect.InvocationHandler;
   import java.lang.reflect.Method;
   import java.lang.reflect.Proxy;
   
   interface Human{
       String getBelief();
   
       void eat(String food);
   }
   
   //被代理类
   class SuperMan implements Human{
       @Override
       public String getBelief() {
           return "I believe I can fly!";
       }
   
       @Override
       public void eat(String food) {
           System.out.println("我喜欢吃" + food);
       }
   }
   //*************************************************************************************
   /*
   * 实现动态代理要解决的问题？
   * 1. 如何根据加载到内存中的被代理类，动态的创建一个代理类及其对象
   * 2. 当通过代理类的对象调用方法时，如何动态的去调用被代理类中的同名方法a
   * */
   class ProxyFactory{
       //调用此方法，返回一个代理类的对象，解决问题一
       public static Object getProxyInstance(Object obj){//obj：被代理类的对象
           MyInvocationHandler handler = new MyInvocationHandler();
           handler.bind(obj);
           return Proxy.newProxyInstance(obj.getClass().getClassLoader(), obj.getClass().getInterfaces(), handler);
       }
   }
   
   class MyInvocationHandler implements InvocationHandler{
       private Object obj; //需要使用被代理类的对象进行赋值
   
       public void bind(Object obj) {
           this.obj = obj;
       }
       //当我们通过代理类的对象，调用方法a时，就会自动的调用如下的方法：invoke()
       //将被代理类要执行的方法a的功能就声明在invoke()中
       @Override
       public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
           //method:即为代理类对象调用的方法，此方法也就作为被代理类调用的方法
           //obj：被代理类的对象
   /*
           下面这个invoke不是这个重写的invoke()方法，而是Method类中的invoke()方法
           Method类中的invoke()方法，调用者就是方法名，第一个参数是obj，表示拥有该方法的对象
                                                  第二个参数是args，表示该方法所需参数
   */
           Object returnValue = method.invoke(obj, args);
           //上述方法的返回值就作为当前类中的invoke()的返回值
           return returnValue;
       }
   }
   //********************************************************************************
   //测试
   public class ProxyTest {
       public static void main(String[] args) {
           SuperMan superMan = new SuperMan();
           //proxyInstance:代理类对象
            Human proxyInstance = (Human) ProxyFactory.getProxyInstance(superMan);
            /*
            * public Object invoke(Object proxy, Method method, Object[] args) throws Throwable方法中对应的参数
            * proxy -> proxyInstance
            * method -> getBelief()  或者 method -> eat()
            * args -> 无 (因为getBelief()方法中没有参数) 或 args -> "汉堡" （因为eat()函数中传入的参数是"汉堡"）
            * */
            proxyInstance.getBelief();
            proxyInstance.eat("汉堡");
       }
   }
   ```

  1. java.lang.reflect.Proxy调用newProxyInsstance方法

     ![image-20230308153246531](C:\Users\Alvin\AppData\Roaming\Typora\typora-user-images\image-20230308153246531.png)

2. 编写JDK动态代理代码

   1. 创建接口，定义方法

      ```java
      public interface UserDao {
          public int add(int a, int b);
          public String update(String id);
      }
      ```

   2. 创建接口实现类，实现方法

      ```java
      public class UserDaoImpl implements UserDao{
          @Override
          public int add(int a, int b) {
              return a + b;
          }
      
          @Override
          public String update(String id) {
              return id;
          }
      }
      ```

   3. 使用Proxy类创建接口代理对象

​			具体代码参照0，二者区别就是可以在 Object returnValue = method.invoke(obj, args);的前面和后面加入一些加强操作，实现不改变源代码增加新功能的效果！！！！！

# AOP操作术语

## 1.连接点

​			类里面哪些方法可以被增强，这些类就称为连接点

## 2. 切入点

​			实际被真正增强的方法，称为切入点

## 3. 通知（增强）

​			(1)实际增强的逻辑部分称为通知

​			(2)通知有多种类型

​					1.前置通知：在被增强的方法之前执行的通知

​					2.后置通知：在被增强方法之后执行的通知

​					3.环绕通知：在被增强方法之前和之后都做执行的通知

​					4.异常通知：被增强方法出现异常以后进行执行

​					5.最终通知：类似finally模块

## 4.切面

​			是动作，把通知应用到切入点的过程

# AOP准备工作

## 1. Spring框架一般都是基于AspectJ实现AOP操作

​		(1)什么是AspectJ？

​			AspectJ不是Spring的组成部分，是独立的AOP框架，一般把AspectJ和Spring框架一起使用，进行AOP操作

## 2. 基于AspectJ实现AOP操作

​		(1)基于xml配置文件实现

​	  （2）基于注解方式实现

## 3.引入相关依赖jar包

## 4.切入点表达式

​		(1)切入点表达式作用：知道对哪个类型里面的哪个方法进行增强

​		(2)语法结构

​			execution（【权限修饰符】【返回类型】【类全路径】【方法名称】【参数列表】）//权限修饰符可以省略，返回																																					类型不能省略

​			举例：对com.atguigu.dao.BookDao类里的add进行增强

​			execution（* com.com.atguigu.dao.BookDao.add(..)） //*代表任意返回类型

​			举例：对com.atguigu.dao.BookDao类里的所有方法进行增强

​			execution（* com.com.atguigu.dao.BookDao.*(..)）

​			举例：对com.atguigu.dao包里所有的类里的所有方法进行增强

​			execution（* com.com.atguigu.dao.* .*(..)）

# AOP操作(AspectJ注解操作)

(1)创建类，在类里面定义方法



(2)创建增强类（编写增强逻辑）

  1. 在增强类里面，创建方法，让不同方法代表不同通知类型

     

(3)进行通知配置

 1. 在Spring配置文件中，开启注解扫描

    ```html
    <?xml version="1.0" encoding="UTF-8"?>
    <beans xmlns="http://www.springframework.org/schema/beans"
           xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
           xmlns:context="http://www.springframework.org/schema/context"
           xmlns:aop="http://www.springframework.org/schema/aop"
           xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd
                               http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context.xsd
                               http://www.springframework.org/schema/aop http://www.springframework.org/schema/context/spring-aop.xsd">
        <context:component-scan base-package="aoptest"/>
    
    </beans>
    ```

 2. 使用注解创建User和UserProxy对象

    ```java
    @Component
    public class UserProxy {
    }
    ```

 3. 在增强类上添加注解@Aspect

    ```java
    @Component
    @Aspect
    public class UserProxy {
        //前置通知
        public void before() {
            System.out.println("before....");
        }
    }
    ```

 4. 在Spring配置文件中开启生成代理对象

    ```html
    <aop:aspectj-autoproxy/>
    ```

 5. 配置不同类型的通知

    (1)在增强类的里面，在作为通知方法上面添加通知类型注解，使用切入点表达式配置

    ```
    1 Spring AOP
    
    AOP：面向切面（Aspect）编程。AOP并不是Spring框架的特性，只是Spring很好的支持了AOP。
    
    如果需要在处理每个业务时，都执行特定的代码，则可以假设在整个数据处理流程中存在某个切面，切面中可以定义某些方法，当处理流程执行到切面时，就会自动执行切面中的方法。最终实现的效果就是：只需要定义好切面方法，配置好切面的位置（连接点），在不需要修改原有数据处理流程的代码的基础之上，就可以使得若干个流程都执行相同的代码。
    
    2 切面方法
    
    1.切面方法的访问权限是public。
    
    2.切面方法的返回值类型可以是void或Object，如果使用的注解是@Around时，必须使用Object作为返回值类型，并返回连接点方法的返回值；如果使用的注解是@Before或@After等其他注解时，则自行决定。
    
    3.切面方法的名称可以自定义。
    
    4.切面方法的参数列表中可以添加ProceedingJoinPoint接口类型的对象，该对象表示连接点，也可以理解调用切面所在位置对应的方法的对象，如果使用的注解是@Around时，必须添加该参数，反之则不是必须添加。
    ```

    

    ```java
    @Component(value = "userProxy") //注册对象
    @Aspect //生成代理对象
    public class UserProxy {
        //前置通知
        @Before(value = "execution(* aoptest.User.add(..))")
        public void before() {
            System.out.println("before....");
        }
    
        //后置通知 或者 返回通知
        @AfterReturning(value = "execution(* aoptest.User.add(..))")
        public void afterReturning() {
            System.out.println("afterReturning....");
        }
        
        //最终通知
        @After(value = "execution(* aoptest.User.add(..))")
        public void after() {
            System.out.println("after....");
        }
    
        //异常通知
        @AfterThrowing(value = "execution(* aoptest.User.add(..))")
        public void afterThrowing() {
            System.out.println("afterThrowing....");
        }
    
        //环绕通知
        @Around(value = "execution(* aoptest.User.add(..))")
        public void around(ProceedingJoinPoint proceedingJoinPoint) throws Throwable {
            System.out.println("环绕之前.....");
            //被增强的方法执行
            proceedingJoinPoint.proceed();
    
            System.out.println("环绕之后.....");
        }
    ```

6. 测试

   ```java
   @Test
   public void testAop() {
       ApplicationContext context = new ClassPathXmlApplicationContext("bean2.xml");
       User user = context.getBean("user", User.class);
       user.add();
   }
   ```

7. 公共切入点抽取

   ```java
   //相同切入点抽取
   @Pointcut(value = "execution(* aoptest.User.add(..))")
   public void pointDemo() {
       
   }
   
   //前置通知
   @Before(value = "pointDemo()")
   public void before() {
       System.out.println("before....");
   }
   ```

8. 多个增强类对同一个方法进行增强，设置增强类型优先级

   (1)在增强类上面添加注释@Order(数字类型值)，数字类型值越小优先级越高

# 事务操作

## 1概念

一般在Service层进行事务操作

两种方式，编程式事务管理和声明式事务管理(使用)

Spring中声明式事务管理（底层为AOP）分为：1. 基于注解方式   2. 基于xml配置文件方式

提供一个接口，代表事务管理器，这个接口针对不同类型框架提供不同的实现类

![image-20230314122314688](C:\Users\Alvin\AppData\Roaming\Typora\typora-user-images\image-20230314122314688.png)



## 2. 具体操作

1. 在spring配置文件配置事务管理器

   ```html
   <!--创建事务管理器-->
   <bean id="transactionManager" class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
       <!--注入数据-->
       <property name="dataSource" ref="dataSource"/>
   </bean>
   ```

2. 在spring配置文件，开启事务注解

   (1)在spring配置文件引入名称空间tx

   ```html
   <beans xmlns="http://www.springframework.org/schema/beans"
          xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
          xmlns:tx="http://www.springframework.org/schema/tx"
          xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd
                              http://www.springframework.org/schema/tx http://www.springframework.org/schema/tx/spring-tx.xsd">
   ```

   (2)开启事务注解

   ```html
   <tx:annotation-driven transaction-manager="transactionManager"/>
   ```

3. 在Service类上面（或者service类里面方法上面）添加事务注解

   @Transactional

   (1）@Transactional，这个注解添加到类上面，也可以添加方法上面
   (2）如果把这个注解添加类上面，这个类里面所有的方法都添加事务
   (3）如果把这个注解添加方法上面，为这个方法添加事务

## 3. propagation:事务传播行为

当一个事务的方法被另外一个事务方法调用的时候，这个事务方法如何进行

![image-20230316210055818](C:\Users\Alvin\AppData\Roaming\Typora\typora-user-images\image-20230316210055818.png)

![image-20230316210408272](C:\Users\Alvin\AppData\Roaming\Typora\typora-user-images\image-20230316210408272.png)



两个事务互不影响

![image-20230316210638994](C:\Users\Alvin\AppData\Roaming\Typora\typora-user-images\image-20230316210638994.png)



单独运行方法B可以不在事务中运行

![image-20230316210834205](C:\Users\Alvin\AppData\Roaming\Typora\typora-user-images\image-20230316210834205.png)

## 4. ioslation：事务隔离级别

（1）事务有特性成为隔离性，多事务操作之间不会产生影响。不考虑隔离性产生很多问题

（2）没有隔离性会有三个读问题：脏读、不可重复读、虚（幻）读

​		脏读：一个未提交事务读取到另一个未提交事务的数据

![image-20230316211554034](C:\Users\Alvin\AppData\Roaming\Typora\typora-user-images\image-20230316211554034.png)

​		不可重复读：一个未提交事务读取到另一提交事务修改数据

![image-20230316212057257](C:\Users\Alvin\AppData\Roaming\Typora\typora-user-images\image-20230316212057257.png)

虚（幻）读：一个未提交事务读取到另一提交事务添加数据



（3）通过设置事务隔离级别，解决读问题

![image-20230316212408212](C:\Users\Alvin\AppData\Roaming\Typora\typora-user-images\image-20230316212408212.png)

![image-20230316212509597](C:\Users\Alvin\AppData\Roaming\Typora\typora-user-images\image-20230316212509597.png)

mysql默认设置REPEATABLERED隔离级别

## 5. 其他参数

![image-20230316212732978](C:\Users\Alvin\AppData\Roaming\Typora\typora-user-images\image-20230316212732978.png)

![image-20230316212754193](C:\Users\Alvin\AppData\Roaming\Typora\typora-user-images\image-20230316212754193.png)

![image-20230316212831945](C:\Users\Alvin\AppData\Roaming\Typora\typora-user-images\image-20230316212831945.png)

![image-20230316212851850](C:\Users\Alvin\AppData\Roaming\Typora\typora-user-images\image-20230316212851850.png)

![image-20230316212929412](C:\Users\Alvin\AppData\Roaming\Typora\typora-user-images\image-20230316212929412.png)

## 6. 完全注解声明式事务管理

​		1. 创建配置类，使用配置类替代xml配置文件

```java
package config;

import com.alibaba.druid.pool.DruidDataSource;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.ComponentScan;
import org.springframework.context.annotation.Configuration;
import org.springframework.jdbc.core.JdbcTemplate;
import org.springframework.jdbc.datasource.DataSourceTransactionManager;
import org.springframework.transaction.annotation.EnableTransactionManagement;

import javax.sql.DataSource;

@Configuration //配置类
@ComponentScan(basePackages = "com.spring5")
@EnableTransactionManagement //开启事务
public class TxConfig {
    //创建数据库连接池对象
    @Bean
    public DruidDataSource getDruidDataSource() {
        DruidDataSource dataSource =  new DruidDataSource();
        dataSource.setDriverClassName("com.mysql.jdbc.Driver");
        dataSource.setUrl("jdbc:mysql:///user_db");
        dataSource.setUsername("root");
        dataSource.setPassword("123456");
        return dataSource;
    }

    //创建JdbcTemplate对象
    @Bean
    public JdbcTemplate getJdbcTemplate(DataSource dataSource) {
        //到ioc容器中根据类型找到dataSource
        JdbcTemplate jdbcTemplate = new JdbcTemplate();
        //注入dataSource
        jdbcTemplate.setDataSource(dataSource);
        return jdbcTemplate;
    }

    //创建事务管理器
    @Bean
    public DataSourceTransactionManager getDataSourceTransactionManager(DataSource dataSource) {
        DataSourceTransactionManager transactionManager = new DataSourceTransactionManager();
        transactionManager.setDataSource(dataSource);
        return transactionManager;
    }
}
```
