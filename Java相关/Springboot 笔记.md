



# Springboot 笔记

## 自动配置

无须配置包扫描:(默认)与主程序同级或低级群都可以被扫描

●默认的包结构
○主程序所在包及其下面的所有子包里面的组件都会被默认扫描进来
○无需以前的包扫描配置
○想要改变扫描路径，@SpringBootApplication(scanBasePackages="com.atguigu")
■ 或者@ComponentScan 指定扫描路径



## @Configuration

```
/*
1. 配置类里面使用也是使用@Bean 方法 默认是单例的
2. 本身也是组件
3. @Configuration(proxyBeanMethods== ...) 如果为true 就是代理,是单利模式 否则就不是单例模式
 */

@Configuration(proxyBeanMethods = false) //这是一个配置类 == 配置文件
public class MyConfig
{
    @Bean //以方法名作为组件ID 返回值就是Bean
    public User user01()
    {
        return new User("李锐", 12);
    }

    @Bean
    public Pet pet01()
    {
        return new Pet("你好");
    }

}
```

## @Import

```java
 * 4、@Import({User.class, DBHelper.class})
 *      给容器中自动创建出这两个类型的组件、默认组件的名字就是全类名
 *
 *
 *
 */

@Import({User.class, DBHelper.class})
@Configuration(proxyBeanMethods = false) //告诉SpringBoot这是一个配置类 == 配置文件
public class MyConfig {
}



```

## @Conditional

条件装配：满足Conditional指定的条件，则进行组件注入

![](https://lzx-figure-bed.obs.dualstack.cn-north-4.myhuaweicloud.com/Figurebed/202201280031479.png)





