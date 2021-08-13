# Java内部类

如果一个事物内部包含另一个事物,那么这就是一个类内部包含另一个类 分类
- 成员内部类
- 局部内部类

成员内部类的定义格式: 修饰符 class 外部类名称{ 修饰符 class 内部类名称{ } }

注意: 内用外 随意;外用内 需要内部类对象

如何使用成员内部类,有两种方式:
1. 间接方式: 在外部类的方法中,使用内部类,然后`main`调用外部类方法

2. 直接方式: 外部类名称.内部类名称 对象名 = `new` 外部类名称().new 内部类名称();

   ```java
      Body.Heart heart = new Body("直接调用的内部类").new Heart();  
   ```

   

   

      如何访问重名的外部类成员变量
   ```java
   public class Outer
   {
       int num = 10;
       public  class Inner
       {
           int num = 20;
           public  void methodInner()
           {
               int num = 30;
               System.out.println(num); //30
               System.out.println(this.num); //20
               System.out.println(Outer.this.num); //10
               
           }
       }
   }
   ```

## 局部内部类

定义格式:

修饰符 class 外部类名称{

​	修饰符 返回值类型 外部类方法名称(参数列表){

​			class 局部类内部名称{

​							//...					

​				}

}

## 匿名内部类

如果接口的实现类(或者是父类的子类)只需要使用一次
那么这种情况下,就可以省略该类的定义,而改为使用 匿名内部类

格式:

```
匿名内部类:
    接口名称 对象名 = new 接口名称(){
    //...覆盖重写接口中所有抽象方法
    };
```

```java

public interface MyInterface
{
    public abstract void method();

}

```



```java
 MyInterface obj = new MyInterface()
        {
            @Override
            public void method()
            {
                System.out.println("匿名内部类实现了方法");
            }
        };
        obj.method();
```



对格式进行解析:
1.    new 代表创建对象的动作
2.   接口名称就是匿名内部类要实现的接口
2.  {...} 是匿名内部类实现的具体内容

注意:

1. **匿名内部类,在创建对象的时候们,只能使用一次**
2. 2.匿名对象,在调用方法时候只能调用唯一一次 













