# SpringBoot加载自定义yml文件

## 新建yml文件

- 在resource根目录下创建自定义yml文件如festival.yml

### yml文件不被Springboot识别

- 在Project Structure界面选择Modules，选择目标模块Spring配置，在右侧界面选择customize spring boot自定义spring boot按钮
- 找到目标yml文件，添加为springboot 文件

## 加载自定义yml文件

- 在resource目录下创建application.yml配置文件
- Springboot v2.2.0.RELEASE
~~~
# yml语法 key: value 属性换行缩进 参考 https://www.jianshu.com/p/97222440cd08
user:
  id: 1001
  name: zhouwei

  # MAP测试
  #方式一
  games: {name: 英雄联盟,ow: 守望先锋,pubg: 绝地求生}
  #方式二
  moba:
    game1: lolm
    game2: dota
  #方式三
  game[LOL]: 英雄联盟
  game[OW]: 守望先锋
  game[PUBG]: 绝地求生

  # List测试
  #方式一
  phones[0]: 小米
  phones[1]: 三星
  phones[2]: 华为
  #方式二
  citys:
    - NanJing
    - Shanghai
    - ShenZhen
~~~

## 方式一：@PropertySource+@Value
- @PropertySource提供了一种方便的声明性机制，用于添加配置文件到Spring环境中。
- @Value指明对应哪个配置参数

- 配置Bean代码
~~~java
package com.zhouwei.ep;

import org.springframework.beans.factory.annotation.Value;
import org.springframework.boot.context.properties.ConfigurationProperties;
import org.springframework.context.annotation.PropertySource;
import org.springframework.stereotype.Component;

import java.util.List;
import java.util.Map;

@Component
@PropertySource(value = "classpath:application.yml")
//@ConfigurationProperties(prefix = "user")
public class User {
    @Value("${user.id}")
    private Integer id;

    @Value("${user.name}")
    private String name;
    
    //不添加getter、setter 属性也可以注入
    public Integer getId() {
        return id;
    }
    public void setId(Integer id) {
        this.id = id;
    }
    public String getName() {
        return name;
    }
    public void setName(String name) {
        this.name = name;
    }
~~~

- 测试代码
~~~java
package com.zhouwei.ep;

import org.junit.Test;
import org.junit.runner.RunWith;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.test.context.junit4.SpringRunner;

@RunWith(SpringRunner.class)
@SpringBootTest
public class TestConfig {

    @Autowired
    private User user;

    @Test
    public void testConfig() {
        System.out.println("user.getName() = " + user.getName());
        System.out.println("user.getId() = " + user.getId());
        System.out.println("user.getGames() = " + user.getGames());
        System.out.println("user.getGame() = " + user.getGame());
        System.out.println("user.getMoba() = " + user.getMoba());
        System.out.println("user.getPhones() = " + user.getPhones());
        System.out.println("user.getCitys() = " + user.getCitys());
    }
}
~~~

## 方式二：@ConfigurationProperties
对于Map/List的配置参数，使用方式一读取不到配置参数，需要配合@ConfigurationProperties来实现。

~~~
//如果用方式一的方法获取games配置，会出现占位符错误

java.lang.IllegalArgumentException: Could not resolve placeholder 'user.games' in value "${user.games}"
~~~

@ConfigurationProperties(prefix="xxx")，设置了前缀，这样配合@Value只需要加具体的参数名称。

~~~java
@Component
@PropertySource(value = "classpath:application.yml")
@ConfigurationProperties(prefix = "user")
public class User {
//    @Value("${user.id}")
    @Value("${code}")//这里读取配置文件中另一个参数，不添加setter方法也可以读取到参数
    private Integer id;
    
    public Integer getId() {
        return id;
    }

//    public void setId(Integer id) {
//        this.id = id;
//    }
~~~


~~~java
package com.zhouwei.ep;

import org.springframework.beans.factory.annotation.Value;
import org.springframework.boot.context.properties.ConfigurationProperties;
import org.springframework.context.annotation.PropertySource;
import org.springframework.stereotype.Component;

import java.util.List;
import java.util.Map;

@Component
@PropertySource(value = "classpath:application.yml")
@ConfigurationProperties(prefix = "user")
public class User {
//    @Value("${user.id}")
    private Integer id;

//    @Value("${user.name}")
    private String name;

//    @Value("${user.games}")
    private Map<String, Object> games;

//    @Value("${user.game}")
    private Map<String, Object> game;

//    @Value("${user.moba}")
    private Map<String, Object> moba;

//    @Value("${user.phones}")
    private List<String> phones;

//    @Value("${user.citys}")
    private List<String> citys;

    //getter、setter方法 保证getBean和配置文件中的参数名对应
}
~~~

在方式一中可以对id属性读取配置文件中的code参数，可以不需要setter方法。
~~~java
@Component
@PropertySource(value = "classpath:application.yml")
//@ConfigurationProperties(prefix = "user")
public class User {
    @Value("${user.code}")
    private Integer id;
    //不需要setter
~~~
输出为
~~~
user.getId() = 2112
~~~
在方式二中对id属性读取配置文件中的code参数，读取参数失败，
~~~java
@Component
@PropertySource(value = "classpath:application.yml")
@ConfigurationProperties(prefix = "user")
public class User {
//    @Value("${user.code}")
    @Value("${code}")
    private Integer id;
~~~

总结：
- 在使用@PropertySource+@Value时，只需要在bean的属性上使用@Value注解指明对应的配置参数，不需要setter方法就能获取。
- 在使用@ConfigurationProperty时，Bean的属性必须与yml配置文件中的字段一致，bean必须有setter方法,且在bean的属性上加@Value注解失效，是无法获取其他的参数的。

参考
https://blog.csdn.net/u010922732/article/details/91048606
https://www.jianshu.com/p/f536b10dd9e7