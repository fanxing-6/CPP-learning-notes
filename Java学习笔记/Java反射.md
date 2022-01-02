# Java反射

**将类的各个组成部分封装成其它类对象**

![image-20211112212811033](https://lzx-figure-bed.obs.dualstack.cn-north-4.myhuaweicloud.com/Figurebed/202111122128218.png)

## 获取class类对象的三种方式:

```java
 		//1.使用Class.forName("类名");
        Class cls1 = Class.forName("com.company.Reflect");
        System.out.println(cls1);
        //2. 使用 类名.class 获取
        Class cls2= Reflect.class;
        System.out.println(cls2);
        //3. 使用 类对象.getClass();
        Reflect re = new Reflect();
        Class cls3 = re.getClass();
        System.out.println(cls3);
```

**比较内存地址是否相同**

```java
        System.out.println(cls2 == cls1); //true
        System.out.println(cls2 == cls3); //true
```

1. Class对象功能
   - 获取成员变量
   - 

