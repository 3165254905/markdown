# 2022.12.25 - 2022.12.27

## 目录

## [类和对象](#jump1)

1. 什么是类？什么是对象？
2. 如何创建类?如何创建对象?----代码操作
3. 引用类型之间画等号:指向同一个对象
4. null 和 NullPointerException

## [方法重载、构造方法和引用类型数组](#jump2)

1. 方法签名: 方法名+参数列表
2. 方法的重载(Overload):
3. 构造方法:
4. this:指代当前对象，哪个对象调指的就是哪个对象
5. 引用类型数组
6. 重载样例
   <br><br>

## <span id="jump1">类和对象</span>

1. 什么是类？什么是对象？

   - 现实世界是由很多很多对象组成的  
     基于这些对象，抽出了类
   - 类:类型、类别，代表一类个体  
     对象:一个独立的个体
   - 类是一种数据类型(引用类型)
   - 类中可以包含:
     - 所有对象所共有的属性/特征----变量
     - 所有对象所共有的行为---------方法
   - 一个类可以创建多个对象

     - 同一个类创建的对象，结构相同，数据不同

       | 对象   | 类(类型) |
       | ------ | -------- |
       | 蔡志荣 | 老师     |
       | 陈平生 | 老师     |
       | 班主任 | 老师     |

       | 对象 | 类   |
       | ---- | ---- |
       | 张三 | 学生 |
       | 李四 | 学生 |
       | 王五 | 学生 |

       | 对象 | 类  |
       | ---- | --- |
       | 小花 | 狗  |
       | 小黑 | 狗  |
       | 小强 | 狗  |

   - 类是对象的模板，对象是类的具体的实例

2. 如何创建类?如何创建对象?----代码操作

   ```java
   修饰词 Class 类名{

   }
   Public class Cell{

   }
   数据类型   引用类型变量       对象
   Cell      c1             = new Cell();
   Cell      c2             = new Cell();

   Dog d1 = new Dog();
   Dog d2 = new Dog();

   Car car1 = new Car();
   Car car2 = new Car();

   class Teacher{  //老师类
       String name;
       int age;
       String address;
       double salary;  //共有的属性
       void eat(){}
       void sleep(){}
       void teach(){}  //共有的行为
   }
   class Student{  //学生类
       String name;
       int age;
       String address;

       void eat(){}
       void sleep(){}
       void study(){}
   }
   class Car{
       String color;
       double price;
       String type;
       void run(){}
       void stop(){}
   }

   class Dog{
       String name;
       String type;
       String color;
       double price;
       void eat(){}
       void sleep(){}
       void run(){}
   }

   创建对象
   Student zs = new Student(); //创建对象
   Student ls = new Student(); //创建对象
   Student ww = new Student(); //创建对象

   zs.name = "zhangsan";
   zs.age = 37;
   zs.address = "河北廊坊";
   zs.eat();
   zs.sleep();
   zs.study();

   ls.name = "lisi";
   ls.age = 26;
   ls.address = "黑龙江佳木斯";
   ls.eat();
   ls.sleep();
   ls.study();
   ```

3. 引用类型之间画等号:指向同一个对象  
   房子-----对象  
   钥匙-----引用  
   对其中的一个修改，会影响另外一个  
   eg:家门钥匙  
   基本类型之间画等号:赋值  
   对其中的一个修改，不会影响另外一个  
   eg:身份证复印件  
   配一把钥匙给你  
   房子-----------  
   房子这个对象只有一个  
   钥匙这个引用有两个
4. null 和 NullPointerException  
    null:空，意为不再指向任何对象  
    引用值为 null，则不能再进行操作了  
    若操作，则发生空指针异常(NullPointerException)
   <br><br>

## <span id="jump2">方法重载、构造方法和引用类型数组</span>

1. 方法签名: 方法名+参数列表
2. 方法的重载(Overload):
   - 方法名相同，参数列表不同，称为方法的重载
   - 编译器在编译时根据签名绑定调用不同的方法
3. 构造方法:
   - 常常用于给成员变量赋初值
   - 与类同名，没有返回值类型
   - 在创建对象时被自动调用
   - 若自己不写构造，则编译器默认提供一个无参构造  
     若自己写了构造，则编译器不再默认提供
   - 构造方法可以重载
4. this:指代当前对象，哪个对象调指的就是哪个对象  
   在方法中访问成员变量，默认有个 this.  
   this 用法:
   1. ```java
      this.成员变量名---访问成员变量
      ```
   2. ```java
      this.方法名()-----调用方法
      ```
   3. ```java
      this()------------调用构造方法
      ```
5. 引用类型数组
   1. ```java
      Cell[] cells = new Cell[4];
      cells[0] = new Cell(1,2);
      cells[1] = new Cell(3,4);
      cells[2] = new Cell(5,6);
      cells[3] = new Cell(7,8);
      ```
   2. ```java
      Cell[] cells = new Cell[]{
        new Cell(1,2),
        new Cell(3,4),
        new Cell(5,6),
        new Cell(7,8)
      };
      ```
   3. ```java
      int[][] arr = new int[3][];
      arr[0] = new int[2];
      arr[1] = new int[3];
      arr[2] = new int[2];
      arr[1][0] = 100; //arr中第2个元素中的第1个元素
      ```
   4. ```java
      int[][] arr = new int[3][4];
      for(int i=0;i<arr.length;i++){
        for(int j=0;j<arr[i].length;j++){
            arr[i][j] = 100;
        }
      }
      ```
6. 重载样例

   ```java
   //重载的演示

   public class OverloadDemo {
       public static void main(String[] args) {
           Aoo o = new Aoo();
           o.show();
           o.show("zhangsan");
           o.show(25);
           o.show("zhangsan",25);
           o.show(25,"zhangsan");
       }
   }

   class Aoo{
       void show(){}
       void show(String name){}
       void show(int age){}
       void show(String name,int age){}
       void show(int age,String name){}
       //int show(){} //编译错误，与返回值类型无关
       //void show(String address){} //编译错误，与参数名称无关
   }
   ```
