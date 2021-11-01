### java中的关键字
- transient ：被该关键字修饰的属性不会序列化
- navicat   : 用来修饰方法，表明该方法是java底层源码，C/C++实现
- class     : 定义类
- interface : 定义接口
- abstract  : 定义抽象类，也可以用在方法上，表明该方法是抽象方法
- enum      : 定义枚举类
- extends   : 继承类，可以继承父类的属性，只能集成一个类，不能多继承,但是可以多层集成
- implements: 实现接口，可以多实现
- instanceof: 返回boolean，判断是否是一个特定类或者是它的子类
- assert    : 断言，如果结果不符则抛出异常
- final     : 被修饰的属性不能改变，被修饰的类不能被继承
- static    : 修饰的方法和属性可以直接被调用

### 访问修饰符
| 修饰符 | 当前类 | 同一包内 | 子孙类(同一包) | 子孙类(不同包) | 其他包 |
|:-----:|:------:|:-------:|:-------------:|:-------------:|:-----:|
|public   |true  |true     |true           |true           |true   |
|protected|true  |true     |true           |不同情况而异    |false  |
|default  |true  |true     |true           |false          |false  |
|private  |true  |false    |false          |false          |false  |

### 异常相关
- throw  : 抛出方法给异常本身
- throws : 抛出方法中的异常给调用者
- try    : 捕获代码中的异常
- catch  : 捕获到异常进行操作
- finally: 不管有没有出现异常都会执行的代码块

```
public class MineDefinedException extends RuntimeException{
    public MineDefinedException(String message) {
        super(message);
    }
}
```
```
public class ....{
  public .. method(String message) throws MineDefinedException{
      throw new MineDefinedException(.....);
  }
}
```

### 线程相关

- volatile : 被修饰的属性取值赋值会直接使用内存，不会使用缓存。达到线程间的可见.性详情如下:
- synchronized : 同步代码块解决了线程间可见性，原子性问题

```
private static boolean valve = true;

public static void main(String[] args) throws InterruptedException {
    new Thread(() -> {
        while(valve){

        }
    }).start();
    TimeUnit.SECONDS.sleep(1);
    new Thread(() -> {
        valve=false;
        System.out.println("valve is false");
    }).start();
}
```
运行上面代码你会发现和你想象的不太一样,试一试如果给valve加上volatile再试试，是不是就达到了你想要的结果
但是对于原子性的问题需要使用synchronized修饰代码块
