# Spring in action 4

## 第一章 Spring之旅

本章主要内容：

- Spring的bean容器
- Spring的核心模块
- 更为强大的Spring生态系统
- Spring的新功能

### 1.1 简化Java开发

POJO JAVABEAN MOCK

Spring框架为了降低Java开发的复杂性，采取了以下4种策略：

- 基于POJO的轻量级和最小侵入性编程
- 通过依赖注入和面向接口实现松耦合
- 基于切面和惯例进行声明式编程
- 通过切面和模板减少样板式代码

#### 激发POJO的潜能

很多框架强迫应用继承它们的类导致应用与框架绑死，而基于Spring构建的应用中，类通常没有任何使用Spring的痕迹。Spring的非侵入性编程模型意味着这个类在Spring和非Spring应用中都能发挥同样的作用。重要的是Spring是通过DI来装配这些POJO。

~~~java
public class HelloWorldBean{
	public String sayHello(){
        return "Hello World";
    }
}
~~~

#### 依赖注入

实现功能需要多个类，这些类之间需要协作来完成业务。传统方式通过每个对象管理与自己关联的对象，即内部保存关联对象的引用，这种方式实现简单但是也导致代码高耦合难以维护和扩展业务。

~~~java
package java.com.springinaction.knights;

public class DamselRescuingKnight implements Knight {
    private Quest quest;
    public DamselRescuingKnight(Quest quest) {
        this.quest = new RescueQuest();//传统方式对象内部保存关联对象的引用
    }
    @Override
    public void embarkOnQuest() {
        quest.embark();
    }
}
~~~

而通过DI，对象的依赖关系由系统中负责协调个对象的第三方组件在创建对象的时候进行设定。对象无需自行创建或管理它们的依赖关系，依赖关系将被自动注入到需要它们的对象当中去。![依赖注入](C:\Users\ZHOUWEI\Documents\Spring in action 4 imgs\依赖注入.png)

~~~java
package java.com.springinaction.knights;

public class DamselRescuingKnight implements Knight {
    private Quest quest;
    public DamselRescuingKnight(Quest quest) {
        this.quest = quest;
    }
    @Override
    public void embarkOnQuest() {
        quest.embark();
    }
}
~~~

不同于直接创建其他对象，上面代码通过构造器注入的方式把其他对象作为参数传入构造器，以实现松耦合且支持所有该接口的子类，没有与具体的对象耦合，只要传入的参数实现接口即可。

如果想传入特定的Quest实现，比如SlayDragonQuest，那么就需要装配。

~~~java
package java.com.springinaction.knights;

import java.io.PrintStream;

public class SlayDragonQuest implements Quest {
    private PrintStream printStream;
    public SlayDragonQuest(PrintStream printStream) {
        this.printStream = printStream;
    }
    @Override
    public void embark() {
        printStream.println("slay the dragon");
    }
}
~~~

创建组件之间协作的行为通常称为装配。Spring提供了三种装配的方式：XML、Java配置类和自动装配。

XML是很常见的一种装配方式，以下代码就将BraveKnight和SlayDragonQuest装配到一起。XML将BraveKnight和SlayDragonQuest声明为bean，并在BraveKnight构造时传入SlayDragonQuest的引用作为参数。

~~~xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
       http://www.springframework.org/schema/beans/spring-beans.xsd">
    <bean id="knight" class="java.com.springinaction.knights.BraveKnight">
        <constructor-arg ref="quest"/>
    </bean>

    <bean id="quest" class="java.com.springinaction.knights.SlayDragonQuest">
        <constructor-arg value="#{T(System).out}"/>
    </bean>
</beans>
~~~

除了XML方式还可以通过Java配置来实现装配。

~~~java
package java.com.springinaction.knights.config;

import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

import java.com.springinaction.knights.BraveKnight;
import java.com.springinaction.knights.Knight;
import java.com.springinaction.knights.Quest;
import java.com.springinaction.knights.SlayDragonQuest;

@Configuration
public class KnightConfig {
    @Bean
    public Knight knight() {
        return new BraveKnight(quest());
    }

    @Bean
    public Quest quest() {
        return new SlayDragonQuest(System.out);
    }
}
~~~

不管是XML还是基于JAVA配置，依赖注入的效果都是相同的。BraveKnight依赖Quest接口但不关心传入的是哪种Quest实现。只有Spring通过配置能够了解这些组件是如何装配起来的，这样就可以再不改变所依赖的类的情况下，修改依赖关系。

现在已经声明了BraveKnight和Quest的关系，接下来我们需要装载XML配置并启动应用。

Spring通过ApplicationContext装载bean的定义并把他们组装起来。Spring应用上下文ApplicationContext负责对象的创建和组装，ApplicationContext有多种实现，它们之间的主要区别就是如何加载配置。

~~~java
package com.springinaction.knights;

import org.springframework.context.support.ClassPathXmlApplicationContext;

public class KnightMain {

    public static void main(String[] args) {
        ClassPathXmlApplicationContext context = new ClassPathXmlApplicationContext("resource/knight.xml");
        Knight knight = context.getBean(Knight.class);
        knight.embarkOnQuest();
        context.close();
    }
}
~~~

#### 应用切面

DI能够让相互协作的软件组件保持松散耦合，而面向切面编程AOP允许把遍布应用各处的功能分离出来形成可重用的组件。

AOP被定义为促使系统实现关注点的分离。AOP实现系统的核心业务与额外的关注点如日志、事务管理等分离，这些关注点服务经常存在承载核心业务的组件中，因为他们会跨越系统的多个组件，所以被称为横切关注点。这些关注点如果分散在各个组件中会导致这些代码重复维护成本高，同时会使得核心业务代码混乱。而AOP能使这些关注点服务模块化，并以声明的方式应用到组件中，不会影响核心业务。

~~~java
package com.springinaction.knights;

import java.io.PrintStream;

public class Minstrel {
    private PrintStream stream;

    public Minstrel(PrintStream stream) {
        this.stream = stream;
    }

    public void singBefore() {
        stream.println("sing before quest");
    }

    public void singAfter() {
        stream.println("sing after quest");
    }
}
~~~

~~~xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:aop="http://www.springframework.org/schema/aop"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
       http://www.springframework.org/schema/beans/spring-beans.xsd http://www.springframework.org/schema/aop http://www.springframework.org/schema/aop/spring-aop.xsd">
    <bean id="knight" class="com.springinaction.knights.BraveKnight">
        <constructor-arg ref="quest"/>
    </bean>

    <bean id="quest" class="com.springinaction.knights.SlayDragonQuest">
        <constructor-arg value="#{T(System).out}"/>
    </bean>

    <bean id="minstrel" class="com.springinaction.knights.Minstrel">
        <constructor-arg value="#{T(System).out}"/>
    </bean>

    <aop:config>
        <aop:aspect ref="minstrel">
            <!--定义切点-->
            <aop:pointcut id="embark" expression="execution(* *.embarkOnQuest(..))"/>
            <!--声明前置通知-->
            <aop:before method="singBefore" pointcut-ref="embark"/>
            <!--声明后置通知-->
            <aop:after method="singAfter" pointcut-ref="embark"/>
        </aop:aspect>
    </aop:config>
</beans>
~~~

通过XML配置为BraveKnight添加切面，通过aop相关命名空间为BraveKnight添加通知。

#### 使用模板消除代替重复模板代码

利用Spring提供的JdbcTemplate消除模板代码。

### 1.2 容纳你的Bean

> Spring通过应用上下文Application Context装载bean的定义并把它们组装起来。Spring应用上下文全权负责对象的创建和组装。Spring自带多种应用上下文的实现，它们的主要区别仅仅在于如何加载配置。

在Spring应用中，对象存在于Spring容器container中。Spring容器负责创建对象，装配对象，配置对象并管理对象的整个生命周期，从创建到死亡。

容器是Spring框架的核心，Spring容器使用DI管理组件，创建相互协作的组件之间的关联。容器中的对象会更加简单，易于理解便于重用。

Spring容器并不只有一个。Spring自带了多个容器的实现，可以归为两种不同的类型。

- Bean工厂，由`org.springframework.beans.factory.BeanFactory`接口定义。这是最简单的容器，提供基本的DI支持。
- 应用上下文，有`org.springframework.context.ApplicationContext`接口定义。应用上下文基于BeanFactory构建，可以为应用提供框架级别的服务，比如从属性文件解析文本信息以及发布应用实践给监听者。一般代替Bean工厂。

#### 使用应用上下文

Spring自带多种应用上下文，下面介绍几个常见的：

- AnnotationConfigApplicationContext 从一个或多个基于Java的配置类中加载Spring应用上下文
- AnnotationConfigWebApplicationContext 从一个或多个基于Java的配置类中加载Spring Web应用上下文
- ClassPathXmlApplicationContext 从类路径下的一个或多个XML配置文件中加载上下文定义，把应用上下文的定义文件作为类资源
- FileSystemXmlapplicationContext 从文件系统下的一个或多个xml配置文件中加载上下文定义
- XmlWebApplicationContext 从Web应用下的xml配置文件中加载上下文定义

在讨论Spring Web应用时会介绍AnnotationConfigWebApplicationContext 和XmlWebApplicationContext ，现在先通过FileSystemXmlApplicationContext 和ClassPathXmlApplicationContext 来加载上下文。

~~~~java
/* FileSystemXmlApplicationContext方式创建容器 */
ApplicationContext context = new FileSystemXmlApplicationContext("c:/knight.xml");

/* ClassPathXmlApplicationContext方式创建容器 */
ApplicationContext context = new ClassPathXmlApplicationContext("knight.xml")
    
/* AnnotationConfigApplicationContext方式创建 */
ApplicationContext context = new AnnotationConfigApplicationContext(com.springinaction.knights.config.KnightConfig.class)
~~~~

使用FileSystemXmlApplicationContext和ClassPathXmlApplicationContext的区别在于前者是在指定的文件路径下查找xml配置文件，而后者是在所有类路径包含Jar文件下查找xml配置文件。如果不制定从xml配置文件中加载Spring应用上下文，而从Java配置类中加载应用上下文需要使用AnnotationConfigApplicationContext。在应用上下文准备就绪之后，就可以调用上下文的getBean()方法从Spring容器中获取bean。

关于路径的问题  https://blog.csdn.net/lizhen1114/article/details/80317068 

#### bean的生命周期

传统的Java应用中，bean的生命周期一般从new开始，然后这个bean就可以使用了，当这个bean不在被使用，就会被Java自动进行垃圾回收。

Spring容器中的bean的生命周期相对复杂，正确了解Spring bean的生命周期非常重要，因为可以利用Spring提供的扩展点来自定义bean的创建过程。在bean准备就绪之前，bean工厂执行了若干启动步骤。下面介绍Spring bean的生命周期：

- Spring对bean进行实例化
- Spring对bean的属性注入值或者引用
- 如果bean实现了BeanNameAware接口，Spring将bean的id传递给setBeanName()方法
- 如果bean实现了BeanFactoryAware接口，Spring将调用setBeanFactory()方法，将BeanFactory容器实例传入
- 如果bean实现了ApplicationContextAware接口，Spring将调用setApplicationContext()方法，将bean所在的应用上下文的引用传进来
- 如果bean实现了BeanPostProcessor接口，Spring将调用它们的postProcessBeforeInitialization()方法
- 如果bean实现了InitializingBean接口，Spring将调用它们的afterPropertiesSet()方法。类似的，如果bean使用init-method声明了初始化方法，该方法也会被调用
- 如果bean实现了BeanPostProcessor接口，Spring将调用它们的postProcessAfterInitialization()方法
- 此时，bean已经准备就绪，可以被应用程序使用了，它们将一直存在应用上下文中，知道该上下文被销毁
- 如果bean实现了DisposableBean接口，Spring将调用它的destroy()接口方法。同样如果bean使用destroy-method声明了销毁的方法，该方法也会被调用。

至此，如何创建和加载一个Spring容器已经介绍完，但是一个空的容器是没有价值，需要借助DI将对象装配到容器中。

### 1.3 Spring的体系结构

Spring框架关注与通过DI、AOP&模板代码来简化开发，在此之外，Spring还有多种方式简化开发。在框架之外也形成了一个基于核心框架的Spring生态圈，涉及Web服务、REST、移动开发以及NoSQL。

##  第二章 装配Bean

本章内容：

- 声明bean
- 构造器注入和setter方法注入
- 装配bean
- 控制bean的创建和销毁

传统方法创建对象之间的关联关系通常会导致代码结构变得越来越复杂，而在Spring中，对象无需自己查找或创建关联对象，容器负责把需要相互协作的对象引用赋予各个对象。创建对象之间协作关系的行为通常称为装配，这也是DI的本质。

### 2.1 Spring配置的可选方案

Spring容器负责创建bean并通过DI来协调对象间的关系，至于Spring要创建哪些bean并且如何装配起来需要我们来指定。Spring装配bean提供了三种主要的机制：

- 在XML中进行显式配置
- 在Java中进行显式配置
- 隐式的bean发现机制和自动装配

建议使用自动装配>java显式配置>xml配置

### 2.2 自动化装配bean

利用Java和XML进行Spring装配bean比较繁琐，Spring还提供了自动装配，从两个角度来实现自动化装配：

- 组件扫描(component scan)：Spring会自动发现上下文创建的bean
- 自动装配(autowiring)：Spring自动满足bean之间的依赖

#### 创建可被发现的bean

`@Component`注解表明这个类会作为组件类，Spring应用上下文会为这个类创建bean。不过，组件默认扫描是不启用的，需要显式配置。

~~~java
package com.springinaction.soundsystem;

import org.springframework.stereotype.Component;

@Component
public class SgtPeppers implements CompactDisc {

    private String title = "Sgt. Pepper's Lonely Hearts Club Band ";
    private String artist = "The Beatles";

    @Override
    public void display() {
        System.out.println("Playing" + title + "by" + artist);
    }
}
~~~

`CDPlayerConfig`定义了Spring的装配规则，这个类使用了`@ComponentScan`注解，该注解能够在Spring中启用组件扫描。该注解默认扫描此配置类所在包及其子包中用`@Component`注解的类，这样就能发现CompactDisc，并为其创建bean。

~~~java
package com.springinaction.soundsystem;

import org.springframework.context.annotation.ComponentScan;
import org.springframework.context.annotation.Configuration;

@Configuration
@ComponentScan
public class CDPlayerConfig {
}
~~~

也可以通过XML方式进行组件扫描

~~~xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context.xsd">
    
    <context:component-scan base-package="com.springinaction.soundsystem"/>

</beans>
~~~

为了测试组件扫描，我们通过一个Junit测试，他会创建Spring上下文，并判断CompactDisc是不是真的创建出来了。

~~~java
package com.springinaction.soundsystem;

import org.junit.Test;
import org.junit.runner.RunWith;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.test.context.ContextConfiguration;
import org.springframework.test.context.junit4.SpringJUnit4ClassRunner;

import static org.junit.Assert.assertNotNull;

@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration(classes = CDPlayerConfig.class)
public class CDPlayerTest {

    @Autowired
    private CompactDisc cd;

    @Test
    public void cdShouldNotBeNull() {
        assertNotNull(cd);
    }
}
~~~

该JUnit使用了SpringJUnit4ClassRunner，以便在测试开始的时候自动创建Spring的应用上下文。`@ContextConfiguration`注解会告诉它需要在`CDPlayerConfig`中加载配置。因为`CDPlayerConfig`中使用了`@ComponentScan`注解，Spring应用上下文会在扫描到组件后创建CompactDisc类的bean。测试类中使用`@Autowired`注解自动从Spring容器中获取CompactDisc类的bean，并通过断言判断bean是否创建。

#### 为组件扫描的bean命名

Spring应用上下文中所有的bean都有一个id，默认为类名首字母小写如sgtPeppers， **特殊情况，即类名超过2个字符，且前两个字符均大写，则直接用类名作为bean的id。** 如果要自定义bean id，可以通过以下两种方式

- `@Component(id)`，如`@Component("lonelyHeartClub")`，lonelyHeartClub就是自定义bean id
- @Named(id)，Spring支持`@Named()`作为`@Component()`的替代方案，两者功能几乎一样，同上

#### 设置组件扫描的基础包

组件扫描注解`@Component`默认以配置类所在包及其子包作为base package进行扫描，而通过`@Component`注解的value属性可以配置其他包进行扫描。也可以通过basePackages属性指定多个包为base package

~~~java
@Configuration
@ComponentScan(value = "com.springinaction.soundsystem")
public class CDPlayerConfig {
}

------------------------------------------------------------------------------
@Configuration
@ComponentScan(basePackages = "com.springinaction.soundsystem")
public class CDPlayerConfig {
}
~~~

上述使用String类型设置基础包，但是在重构代码时，所指定的基础包可能就会出错。`@ComponentScan`还提供了另一种方法，将base package指定为包中所包含的类或接口。如下，`CDPlayerConfig`和`CompactDisc`这两个类所在的包将会作为base package被扫描。

~~~java
@Configuration
@ComponentScan(basePackageClasses = {CDPlayerConfig.class, CompactDisc.class})
public class CDPlayerConfig {
}
~~~

#### 实现自动装配

自动装配就是让Spring自动实现bean依赖，在满足依赖的过程中，会在Spring上下文中寻找匹配某个bean需求的其他bean，借助`@Autowired`注解实现自动装配。`@Autowired`注解可以使用在类的属性，构造器，方法上，在Spring初始化bean之后，会自动装配满足bean的依赖。如果该bean不存在Spring容器中，Spring会抛出异常。为了避免异常，可以将`@Autowired`的required属性设置为false，这样在没有合适的bean装配的时候回赋值为null。

`@Autowired`注解可以用`javax.inject.Inject`包中的`@Inject`注解代替。

### 2.3 通过Java代码装配bean

尽管在很多场景下通过组件扫描和自动装配实现Spring的自动化配置是更为推荐的方式，但有时候这种方案行不通，因此需要明确配置Spring。比如说，将第三方库的组件装配到自己的应用中，这种情况是无法在第三方库的类加@Component和@Autowired注解。所以，在这种情况下就需要采用显式装配的方式。显式装配有Java和XML两种方案。

在进行显式配置时，JavaConfig更强大、类型安全并且对重构友好。JavaConfig中只应该包含配置代码，不应该包含业务代码，通常将它放在单独的包中用以区分。

#### 创建配置类

Java通过为类添加`@Configuration`注解使该类成为配置类，该类应该包含在Spring上下文中如何创建bean的细节。在此为了演示纯显式配置，去掉了`@ComponentScan`注解，以另一种方式装配bean。

#### 声明简单的bean

去掉扫包要在JavaConfig中声明bean，我们需要`@Bean`注解，该注解告诉Spring指定方法会返回一个对象，该对象要注册为Spring上下文中的bean。默认情况bean的id与带有`@Bean`注解的**方法名**是一样的。也可以通过`@Bean`注解的name属性指定一个方法名。

~~~java
package com.springinaction.soundsystem;

import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

/**
 * Java显式配置
 * @author ZHOUWEI
 */

@Configuration
public class JCDPlayerConfig {

    @Bean//bean id为sgtPeppers
    public CompactDisc sgtPeppers() {
        return new SgtPeppers();
    }
	
    @Bean(name = "peppers")//bean id为peppers
    public CompactDisc sgtPeppers() {
        return new SgtPeppers();
    }
}
~~~

#### 借助JavaConfig实现注入

JavaConfig实现DI有两种方案：

- 引用JavaConfig中创建bean的方法
- 参数注入

第一种方案通过cdPlayer()方法创建CDPlayer类的bean，该方法体调用CDPlayer的有参构造器来创建实例，而参数为调用sgtPeppers()方法得到的实例，注意sgtPeppers()方法添加了`@Bean`注解，Spring会拦截对sgtPeppers()的调用，所以默认情况下是单例的。

```java
@Bean
public CDPlayer cdPlayer() {
    return new CDPlayer(sgtPeppers());
}
```

第二种方案更为简单，cdPlayer方法请求CompactDisc作为参数，然后以构造器或者setter方法进行注入。

~~~java
@Bean
public CDPlayer cdPlayer(CompactDisc cd){
    return new CDPlayer(cd);
}
~~~

验证，CDPlayerConfig配置了扫包，只能扫到CompactDisc不能扫到CDPlayer，JCDPlayerConfig使用`@Bean`装配CompactDisc到CDPlayer中，生成的CDPlayer类bean id 不影响测试中自动装配。

~~~java
package com.springinaction.soundsystem;

import org.junit.Test;
import org.junit.runner.RunWith;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.test.context.ContextConfiguration;
import org.springframework.test.context.junit4.SpringJUnit4ClassRunner;

@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration(classes = {CDPlayerConfig.class, JCDPlayerConfig.class})
public class JCDPlayerTest {

    @Autowired
    private CDPlayer cd;

    @Test
    public void play() {
        cd.play();//输出 DI CompactDisc 
    }
}
~~~

### 2.4 通过XML装配bean

通过XML创建以\<beans>元素为根的文件，相当于JavaConfig中`@Configuration`注解，涉及元素有\<bean>\<constructor-arg>\<null>\<property>等等。

### 2.5 导入和混合配置

- JavaConfig导入XML配置

通过`@Import`注解导入其他JavaConfig文件，通过`@ImportResource`导入XML配置文件

- XML中导入JavaConfig

在XML中将JavaConfig声明为bean，再通过\<impot>元素引用

## 第三章 高级装配

- Spring profile
- 条件化的bean声明
- 自动装配与歧义性
- bean的作用域
- Spring表达式语言

### 3.1 环境与profile

上一章的装配的主角是bean，而这一章高级装配上升到环境配置。在完整的开发周期中，应用会跑在不同的环境中，若使用单一的配置就会出现问题。Spring通过提供profile使应用在运行时在决定所创建的bean。

#### 配置profile bean

Spring引入bean profile功能，使用前需要把所有不同的bean定义在一个或多个profile文件中，在应用部署到某个环境前，保证该环境的profile文件处于激活状态。通过使用`@Profile`注解指定某个bean属于哪个profile。

- `@Profile`注解用在**类**上时只有dev profile激活时才会创建bean

~~~java
package com.springinaction.advancedwire;

import org.springframework.context.annotation.Configuration;
import org.springframework.context.annotation.Profile;

@Configuration
@Profile(value = "dev")
public class DevProfileConfig {
    @Bean
    public CDPlayer cdPlayer(){
        ...
    }
}
~~~

- `@Profile`注解用在**方法**，可以将不同profile创建的bean放在同一JavaConfig中

~~~java
@Bean
@Profile("test")
public Object object() {
    return null;
}
~~~

#### XML中配置profile

XML中通过\<bean>元素的profile属性配置profile bean，也可以配置多个profile bean

~~~xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
       http://www.springframework.org/schema/beans/spring-beans.xsd"
        profile="dev">

    <beans profile="test"/>

    <beans profile="qa"/>

</beans>
~~~

#### 激活profile

Spring通过spring.profiles.active和spring.profiles.default来确定某个profile处于激活状态。如果设置了spring.profiles.active属性，那设置的该profile就是激活的。如果没设置就默认为spring.profiles.default的profile，如果都没有设置，那就没有激活的profile。有多个方式来设置这两个属性：

- 作为DispatcherServlet的初始化参数
- 作为Wev应用的上下文参数
- 作为JNDI条目
- 作为环境变量
- 作为JVM的系统属性
- 在集成测试类上，使用`@ActiveProfiles`注解设置激活的profile

在web应用的web.xml文件中设置默认的proile

~~~xml
# 为上下文设置默认的profile
<context-param>
	<param-name>spring.profiles.default</param-name>
	<param-value>dev</param-value>
</context-param>

# 为Servlet设置默认的profile
<init-param>
	<param-name>spring.profiles.default</param-name>
	<value>dev</value>
</init-param>
~~~

### 3.2 条件化的bean

如果希望某个bean在特定条件下才会创建，那么就需要借助`@Conditional`注解，他可以用到带有`@Bean`注解的方法上，如果给定的条件计算结果为true，就会创建这个bean，否则这个bean就会被忽略。

例如，MagicBean只有设置了magic环境属性的时候，Spring才会实例化这个类。

~~~java
@Bean
@Conditional(MagicExistsCondition.class)
public MagicBean magicBean(){
	return new MagicBean();
}
~~~

`@Conditional`给定了一个Class作为条件，`@Conditional`将会通过Condition接口进行条件对比

设置给`@Conditional`的类可以是任意实现了Condition接口的类型。

### 3.3 处理自动装配的歧义性

在某个接口有多个实现类的时候，采用自动装配就会出现歧义从而抛出异常。

~~~java
Caused by: org.springframework.beans.factory.NoUniqueBeanDefinitionException: No qualifying bean of type 'com.springinaction.advancedwire.Phone' available: expected single matching bean but found 2: HUAWEI,MIPhone
~~~

#### 标示首选的bean

在实现类上用`@Primary`注解Spring会在自动装配的时候首选该实现类，搭配`@Component`或者`@Bean`注解使用

~~~java
package com.springinaction.advancedwire;

import org.springframework.context.annotation.Primary;
import org.springframework.stereotype.Component;

@Component
@Primary
public class MIPhone implements Phone {
    @Override
    public void call() {
        System.out.println("are you ok?");
    }
}
~~~

#### 限定自动装配的bean

在自动注入中可选bean较多时，通过`@Qualifier`注解指定id注入bean

~~~java
package com.springinaction.advancedwire;

import org.junit.Test;
import org.junit.runner.RunWith;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.beans.factory.annotation.Qualifier;
import org.springframework.test.context.ContextConfiguration;
import org.springframework.test.context.junit4.SpringJUnit4ClassRunner;

import static org.junit.Assert.assertNotNull;

@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration(classes = DevProfileConfig.class)
public class PhoneTest {

    @Autowired
    @Qualifier("HUAWEI")
    private Phone phone;

    @Test
    public void phoneCall() {
        phone.call();
        assertNotNull(phone);
    }
}
~~~

### 3.4 bean的作用域

默认情况下，Spring应用上下文中bean都是以单例形式创建的，但是单例不适合某些状态会变的类，Spring定义了多种作用域，可以基于这些作用域创建bean，包括：

- 单例(Singleton)：在整个应用中，只创建bean的一个实例
- 原型(Prototype)：每次注入或通过Spring应用上下文获取的时候，都会创建一个新的bean实例
- 会话(session)：在web应用中，为每个会话创建一个bean实例
- 请求(request)：在web应用中，为每个请求创建一个bean实例

通过`@Scope`注解将bean声明为其他作用域

```xml
# 注解声明作用域代理
//@Scope(ConfigurableBeanFactory.SCOPE_PROTOTYPE)//作为原型bean

//@Scope("prototype")//使用字符串方式声明为原型bean

@Scope(value = WebApplicationContext.SCOPE_SESSION, proxyMode = ScopedProxyMode.INTERFACES)//声明作用域为web会话,作用域代理模式为接口代理，代理对象需要实现接口

# xml中配置作用域代理 
在<bean>元素中的scope属性声明bean的作用域;在<aop:scoped-proxy proxy-target-class=false/>声明代理类型，默认使用cglib创建目标类代理，可以通过设置proxy-target-class=false生成基于接口的代理
```

#### 使用会话和请求作用域

会话和请求作用域能在会话和请求范围内实例化共享的bean，单例和原型都不能满足bean与给定会话绑定。proxy-target-class指定的接口或者类会作为目标类的代理，Spring会把实际的bean注入到这个代理中，在调用方法时代理会对其进行懒解析并将调用委托给会话作用域内真正的bean。

### 3.5 运行时注入值

DI除了bean之间的注入，还有运行时注入值的方式：

- 属性占位符(Property placrholder)
- Spring语言式表达(SpEL)

#### 属性占位符

`@PropertySource`注解引用加载properties文件，在通过Environment获取参数。

~~~
# 声明属性源
@PropertySource("classpath:/com/soundsystem/app.properties")
#检索属性值
env.getProperties("disc.title")
~~~

在Spring装配中，占位符的形式为使用${a.b}包装的属性名称，在XML中通常使用这种方式。

如果依赖组件扫描和自动装配来创建和初始化组件的话，那就没有指定占位符的配置文件或类了。在这种情况下可以使用`@Value`注解

~~~java
@Bean
public Phone phone(
        @Value("${x.y}") String name
) {
    return null;
}
~~~

为了使用占位符，还需要配置一个PropertySourcePlaceholderConfigurer 类的bean。

~~~java
@Bean
public PropertySourcesPlaceholderConfigurer placeholderConfigurer() {
    return new PropertySourcesPlaceholderConfigurer();
}
~~~

#### SpEL语言



## 第4章 面向切面的Spring

本章内容

- 面向切面编程的基本原理
- 通过POJO创建切面
- 使用@AspectJ注解
- 为AspectJ切面注入依赖

软件系统中业务往往都由核心业务功能代码支撑，但是除此之外，也有一些其他的功能代码比如日志安全管理等等，这些功能代码往往是和业务逻辑相分离的，被称为横切关注点。把这些横切关注点和业务逻辑相分离正是面向切面编程AOP要解决的问题。DI有助于应用对象之间的耦合，而AOP是关注点和应用的解耦。

### 4.1 什么是面向切面编程

切面能帮助我们模块化横切关注点，横切关注点可以被描述为影响应用多处的功能，如安全、日志。切面就是在系统中定义日志的通用类，但是可以通过声明的方式定义日志要以何种方式在何处应用，而无需修改受影响的类。横切关注点可以被模块化为特殊的类，这个类就被称为切面，比如创建日志服务的切面。

相关术语 转载  https://www.jianshu.com/p/5015f212e4e4 

- 连接点 Joinpoint

程序执行的某个特定位置， 如类开始初始化前、类初始化后、类某个方法调用前、调用后、方法抛出异常后。 

- 切点 Pointcut

每个程序类都拥有多个连接点，如一个拥有两个方法的类，这两个方法都是连接点，即连接点是程序类中客观存在的事物。AOP通过“切点”定位特定的连接点。连接点相当于数据库中的记录，而切点相当于查询条件。切点和连接点不是一对一的关系，一个切点可以匹配多个连接点。

**连接点是一个比较空泛的概念，就是定义了哪一些地方是可以切入的，也就是所有允许你通知的地方。**

  **切点就是定义了通知被应用的位置 （配合通知的方位信息，可以确定具体连接点）**

- 通知Advice

通知是织入到目标类连接点上的一段程序代码，在Spring中，通知除用于描述一段程序代码外，还拥有另一个和连接点相关的信息，这便是执行点的方位。结合执行点方位信息和切点信息，我们就可以找到特定的连接点。

Spring切面可以应用5种类型的通知：

​    前置通知（Before）：在目标方法被调用之前调用通知功能；

​    后置通知（After）：在目标方法完成之后调用通知，此时不会关心方法的输出是什么；

​    返回通知（After-returning）：在目标方法成功执行之后调用通知；

​    异常通知（After-throwing）：在目标方法抛出异常后调用通知；

​    环绕通知（Around）：通知包裹了被通知的方法，在被通知的方法调用之前和调用之后执行自定义的行为。

​    **通知就定义了，需要做什么，以及在某个连接点的什么时候做。 上面的切点定义了在哪里做。**

- 引入  **Introduction** 

引介是一种特殊的通知，它为类添加一些属性和方法。这样，即使一个业务类原本没有实现某个接口，通过AOP的引介功能，我们可以动态地为该业务类添加接口的实现逻辑，让业务类成为这个接口的实现类。 

  **允许向现有类中添加方法和属性（通知中定义的）**

- 织入 Weaving

织入是将通知添加对目标类具体连接点上的过程。AOP像一台织布机，将目标类、通知或引介通过AOP这台织布机天衣无缝地编织到一起。根据不同的实现技术，AOP有三种织入的方式：

  a、编译期织入，这要求使用特殊的Java编译器。

  b、类装载期织入，这要求使用特殊的类装载器。

  c、动态代理织入，在运行期为目标类添加通知生成子类的方式。

  **把切面应用到目标对象来创建新的代理对象的过程，Spring采用动态代理织入，而AspectJ采用编译期织入和类装载期织入。**

- **代理（Proxy）**

一个类被AOP织入通知后，就产出了一个结果类，它是融合了原类和通知逻辑的代理类。根据不同的代理方式，代理类既可能是和原类具有相同接口的类，也可能就是原类的子类，所以我们可以采用调用原类相同的方式调用代理类。

-  **切面（Aspect）** 

切面由切点和通知组成，它既包括了横切逻辑的定义，也包括了连接点的定义，Spring AOP就是负责实施切面的框架，它将切面所定义的横切逻辑织入到切面所指定的连接点中。

  **切点的通知的结合，切面知道所有它需要做的事：何时/何处/做什么**

#### Spring对AOP的支持

Spring提供4中类型的AOP支持

- 基于代理的经典Spring AOP
- 纯POJO切面
- @AspectJ注解驱动的切面
- 注入式AspectJ切面

前三种都是基于Spring AOP，Spring AOP构建在动态代理基础之上，因此，Spring对AOP的支持局限于方法拦截。Spring借鉴了AspectJ的切面，以提供注解驱动的AOP。本质上，它依然是Spring基于代理的AOP，但是风格和AspectJ注解完全一致。AspectJ提供了更强大的功能，支持构造器和属性拦截。

Spring通知是JAVA编写的，所以可以在一样的IDE中完成。而且，定义通知所应用的切点通常会使用注解或XML。而AspectJ有自己的AOP语言，强大的功能也需要额外的学习成本。

Spring在运行时通知对象，通过在代理类中包裹切面，在运行时把切面织入到bean中，此时代理类封装了目标类，拦截被通知方法的调用，再把调用转发给真正的目标bean。当代理拦截到方法调用时，在调用目标bean方法之前，会执行切面逻辑。

SpringAOP只支持方法级别的连接点，不支持属性和构造器。

### 4.2 通过切点来选择连接点

Spring AOP 所支持的AspectJ切点指示器



#### 编写切点

Performance接口来定义切面的切点

~~~java
package com.springinaction.aop;

public interface Performance {
    void perform();
}
~~~

下面是一个切点表达式，该表达式会在指定方法执行时触发 。*位置表示匹配返回任意类型 然后依次为包.类.方法(参数)，同时还可以使用within()指示器来限制匹配包。"&&"操作符将两者组成与关系，类似"||"和"!"分别为 或、非。

切入点表达式 

 execution(【modifiers-pattern?】 访问修饰符 ret-type-pattern 返回值类型【declaring-type-pattern?】 全限定性类名 name-pattern(param-pattern) 方法名(参数名)【throws-pattern?】) 抛出异常类型 

~~~
execution(* aop.Performance.perfom(..))&&within(aop.*)and bean("Performance")
~~~

### 4.3 使用注解创建切面

#### 定义切面

定义Audience类为Performance接口的切面。具体代码如下

在环绕通知中需要ProceedingJoinPoint来调用被通知的方法，通过ProceedingJoinPoint的proceed()方法将控制权转移到被调用的方法，否则环绕通知会阻塞被调用的方法。

~~~java
package com.springinaction.aop;

import org.aspectj.lang.ProceedingJoinPoint;
import org.aspectj.lang.annotation.*;
import org.springframework.stereotype.Component;

@Aspect
@Component
public class Audience {

    @Pointcut(value = "execution(public * *..Performance.perform(..))")
    public void myPointcut() {
        System.out.println("====>已找到切点");
    }

    @Before(value = "execution(public * *..Performance.perform(..))")
//    @Before("myPointcut()")
    public void myBefore() {
        System.out.println("====>开始前置通知");
    }

    @After(value = "execution(public * *..Performance.perform(..))")
    public void myAfter() {
        System.out.println("====>开始后置通知");
    }

    @AfterReturning(value = "execution(public * *..Performance.log(..))")
    public void myAfterReturning() {
        System.out.println("====>返回值已打印");
    }

    @AfterThrowing(value = "execution(public * *..Performance.withException(..))")
    public void myAfterThrowing() {
        System.out.println("====>异常已抛出");
    }

    @Around(value = "execution(public * *..Performance.perform(..))")
    public void myAround(ProceedingJoinPoint pjp) throws Throwable {
        System.out.println("====>环绕前置增强");

        pjp.proceed();//若不调用该方法会阻塞被调用的方法

        System.out.println("====>环绕后置增强");
    }

}
~~~

目标类

~~~java
package com.springinaction.aop;

public interface Performance {

    String log();

    void perform();

    void withException();
}
~~~

~~~java
package com.springinaction.aop;

import org.springframework.stereotype.Component;

@Component
public class MusicPerformance implements Performance {

    @Override
    public String log() {
        String log = "print log...";
        System.out.println(log);
        return log;
    }

    @Override
    public void perform() {
        System.out.println("start music performance");
    }

    @Override
    public void withException() {
        int i = 1 / 0;
    }
}
~~~

JAVAConfig

~~~java
package com.springinaction.aop;

import org.springframework.context.annotation.ComponentScan;
import org.springframework.context.annotation.Configuration;
import org.springframework.context.annotation.EnableAspectJAutoProxy;

@Configuration
@EnableAspectJAutoProxy//启用Aspect注解
@ComponentScan
public class AspectConfig {

}
~~~

测试类

~~~~java
package com.springinaction.aop;

import org.junit.Test;
import org.junit.runner.RunWith;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.test.context.ContextConfiguration;
import org.springframework.test.context.junit4.SpringJUnit4ClassRunner;

@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration(classes = AspectConfig.class)
public class PerformanceTest {

    @Autowired
    private Performance performance;

    @Test
    public void aopPerform() {
        performance.perform();
        performance.log();
        performance.withException();
    }
}
~~~~

输出

~~~text
====>环绕前置增强
====>开始前置通知
start music performance
====>环绕后置增强
====>开始后置通知
print log...
====>返回值已打印
====>异常已抛出
~~~

#### 利用XML来配置Aspect

~~~xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:aop="http://www.springframework.org/schema/aop"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd http://www.springframework.org/schema/aop http://www.springframework.org/schema/aop/spring-aop.xsd">

    <bean id="audience" class="com.springinaction.aop.Audience"/>
    
    <!--自动代理-->
    <aop:aspectj-autoproxy/>
    
    <aop:config>
        <aop:pointcut id="myPointcut" expression="execution(public * *..Performance.perform(..))"/>
        <aop:aspect ref="audience">
            <aop:before method="myBefore" pointcut-ref="myPointcut"/>
            <aop:after method="myAfter" pointcut-ref="myPointcut"/>
            <aop:after-returning method="myAfterReturning" pointcut-ref="myPointcut"/>
            <aop:after-throwing method="myAfterThrowing" pointcut-ref="myPointcut"/>
            <aop:around method="myAround" pointcut-ref="myPointcut"/>
        </aop:aspect>
    </aop:config>
</beans>
~~~

#### 通过切面引入新功能(#RH)

===>

### 4.3 注入AspectJ切面(#RH)

===>

## 第五章 构建Spring Web应用



## 第六章 渲染Web视图

### 6.4 使用Thymeleaf

为了在Spring中使用Thymeleaf，需要配置启用Thymeleaf与Spring集成的三个bean：

- ThymeleafViewResolver：将逻辑视图名称解析为Thymeleaf模板视图
- SpringTemplateEngine：Spring模板引擎
- TemplateResolver：加载Thymeleaf模板，模板解析器。

## 第七章 SpringMVC 高级技术

#### 7.3 处理异常

- 特定的Spring异常将会自动映射为HTTP状态码
- 异常类上可以加`@ResponseStatus`注解，将其映射为状态码
- 通过`@ExceptionHandler`注解为指定异常添加异常处理器
- 通过`@ControllerAdvice`注解为处理器添加全局异常处理



