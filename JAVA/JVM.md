# JVM

- JVM内存模型
- GC

## JVM如何加载.class文件

JVM主要由Class Loader，Runtime Data Area，Execution Engine，native interface 这四个部分组成。它主要通过ClassLoader将符合其格式要求的class文件加载到内存中，并通过Execution Engine去解析Class文件中的字节码，提交给操作系统去执行。

![image-20200717152427322](C:\Users\92985\AppData\Roaming\Typora\typora-user-images\image-20200717152427322.png)

- Class Loader：根据特定格式，加载class文件到内存
- Execution Engine：对命令进行解析
- Native Interface：融合不同开发语言的原生库为Java所用
- Runtime Data Area：JVM内存结构模型



## 什么是反射

JAVA反射机制是运行状态中，对于**任意一个类**，都能知道这个类的**所有属性和方法**；对于**任意一个对象**，都能**调用**它的任意方法和属性；这种**动态获取信息**以及**动态调用对象**的方法称为java反射机制。



#### 写一个反射例子

```java
Class rc = Class.forname("cn.yongnian.Robot");
Robot r = (Robot) rc.newInstance();
// 类的名字: rc.getName();
Method getHello = rc.getDeclaredMethod("throwHello", String.class);//获取所有方法(除了继承的和实现的接口的方法)
getHello.setAccessible(true);
Object str = getHello.invoke(r,"Bob");
Method sayHi = rc.getMethod("");//public,但:继承的方法和接口的方法
sayHi.invoker(r,"Welcome");
```





## Class Loader

- 工作在Class装载的加载阶段
- 主要作用：从系统外部获得Class二进制数据流，然后交给JVM进行连接、初始化等操作
- 所有的Class都是这样加载的。

#### 类从编译到执行的过程

1. 编译器将Robot.java 源文件编译为Robot.class字节码文件
2. ClassLoader将**字节码**转换为JVM中的Class<Robot>对象
3. JVM利用**Class<Robot>**对象实例化为**Robot对象**





#### ClassLoader的种类

- BootStrapClassLoader：C++编写，加载核心库java.*
- ExtClassLoader：JAVA编写，加载扩展库 javax.*
- AppClassLoader：JAVA编写，加载程序所在目录
- 自定义ClassLoader：JAVA编写，定制化加载

```java
public class MyClassLoader extends ClassLoader{
	// 用于寻找类文件
    @Override
    public Class findClass(String name){
        byte[] b = loadClassData(name);
        return defineClass(name,b,0,b.length); 
        //将二进制流转换为Class。
    	// 利用defineClass，我们可以利用不同的形式去加载
        // 1. 访问远程网络，获取二进制流
        // 2. 对某些敏感文件加密，在findClass中对其进行解密
        // 3. 对生成的二进制流做手脚，添加信息（字节码增强技术）
        // 4. AOP也可以这样实现
    }
    
    //用于加载类文件
    private byte[] loadClassData(String name){
        //FILE OPERATION
    }
}
```



## 双亲(parent)委派机制

![image-20200717165943453](C:\Users\92985\AppData\Roaming\Typora\typora-user-images\image-20200717165943453.png)

#### 为什么要使用双亲委派机制去加载类？

- 避免多份同样字节码的加载



## 类的加载方式

- 隐式加载：new（隐式调用类加载器）
- 显式加载：loadClass，forName等

![image-20200717171246686](C:\Users\92985\AppData\Roaming\Typora\typora-user-images\image-20200717171246686.png)



#### loadClass和forName区别

- Class.forName得到的class是已经**初始化完成的**
- Classloader.loadClass得到的class是**还没有链接的**
  - Spring IOC 延时加载工作





## JAVA内存模型

#### 线程独占部分

- 地址空间的划分
  - 内核空间
  - 用户空间

![image-20200717215215887](C:\Users\92985\AppData\Roaming\Typora\typora-user-images\image-20200717215215887.png)

 

![image-20200717223411760](C:\Users\92985\AppData\Roaming\Typora\typora-user-images\image-20200717223411760.png)

- 程序计数器

  - 当前线程所执行的字节码行号指示器(**逻辑计数器**)
  - Java方法计数, Native方法则计数器值为Undefined
  - 不会发生内存泄漏

- java虚拟机栈

  - java方法执行的内存模型

  - 包含多个帧栈

    - 局部变量表：包括方法执行过程中的所有变量

    - 操作数栈：类似于CPU中的寄存器

    - > 递归java.lang.StackOverflowError
      >
      > 原因：栈帧超出虚拟栈深度

- 本地方法栈

  - 与虚拟机相似，主要作用于标注了native的方法

#### 线程公有

- 元空间（MetaSpace）和永久代（PermGen）的区别

  - 存储Class的相关信息，包括Class的Method和Field
  - 都是方法区的实现（方法区为JVM的规范）
  - 元空间使用**本地内存**，永久代使用的是JVM的内存
  - 元空间的优势
    - **字符串常量池**存在永久代中，容易出现性能问题和内存溢出
    - **类和方法的信息**大小难以确定，给永久代的大小指定带来困难
    - 为**GC**带来不必要的复杂性

- **Java堆**

  - 对象实例的分配区域

  - GC管理的主要区域

    ![image-20200717223458825](C:\Users\92985\AppData\Roaming\Typora\typora-user-images\image-20200717223458825.png)

#### 常考问题

- JVM三大性能调优参数的含义
- JAVA内存模型中**堆和栈**的区别-内存分配策略
  - 静态存储：编译时确定每个数据目标在运行时的存储空间需求。
  - 栈式存储：数据区需求在编译时未知，运行时模块入口前确定。
  - 堆式存储：编译时或者运行时模块入口都无法确定，动态分配。 
  - 联系：引用对象、数组时，栈里定义变量保存堆中目标的首地址
  - 区别
    - 管理方式：栈自动释放，堆需要GC
    - 空间大小：栈比堆小
    - 碎片相关：栈产生的碎片远小于堆
    - 分配方式：栈支持静态和动态分配，而堆仅支持动态分配
    - 效率：栈的效率比堆高





# GC垃圾回收

#### 标记垃圾算法

- 引用计数算法

  - 判断对象的引用数量来决定对象是否可被回收
  - 每个对象实例都有一个引用计数器,被引用则+1,完成引用-1
  - 任何引用计数为0的对象实例可以被当做辣鸡收集
    - 优点：执行效率高
    - 确定：循环引用---内存泄漏

- 可达性分析算法

  - 判断对象的引用链是否可达
  - GC Root 开始（图）
    - 虚拟机栈中引用的对象（栈帧中的本地变量表）
    - 方法区中的常量引用的对象
    - 方法中类静态属性引用的对象
    - 本地方法栈中JNI的引用对象
    - 活跃线程的引用对象

  

#### 垃圾回收算法

- 标记-清除算法（Mark and Sweep)

  - 产生大量碎片化空间

    ![image-20200718114214993](C:\Users\92985\AppData\Roaming\Typora\typora-user-images\image-20200718114214993.png)

- 复制算法

  - 对象面和空闲面

  - 对象在对象面上创建

  - 存活的对象被从对象面复制到空闲面

  - 将对象面所有对象内存清除 

    ![image-20200718114727694](C:\Users\92985\AppData\Roaming\Typora\typora-user-images\image-20200718114727694.png)

- 标记-整理算法

  - 清除：移动所有存活对象，按照内存地址次序依次排列，然后将末端内存地址以后的内存全部回收。
  - 优点
    - 避免内存的不连续行
    - 不用设置两块内存互换
    - 适用于存活率高的场景

- 分代收集算法

  - 组合拳
  - 按照对象生命周期的不同划分区域以采用不同垃圾回收算法，提高JVM回收效率

  - 年轻代，存活率低，复制算法
  - 老年代，存活率高，标记清除 or 标记整理

#### 分代收集算法

- 年轻代：尽可能快速收集生命周期短的对象
  - 存活率低---赋值算法： from <-> to 
  - MINOR GC
  - 如何晋升到老年代
    - 经过一定Minor次数依然存活的对象
    - Survivor区中放不下的对象
    - 新生的大对象
  - 常用的调优参数

```
-XX:SurviroRatio:Eden和Survivor的比值,默认8:1
-XX:NewRatio:老年代和年轻代内存大小的比例
-XX:MaxTenuringThreshold:对象从年轻代晋升到老年代经过GC次数的最大域值
```

- 老年代：生命周期较长的对象

  ![image-20200718120004270](https://raw.githubusercontent.com/YN-Zheng/YN-Zheng.github.io/master/typora202008/24/124956-322002.png)

- Full GG  (Major GC)

  - 比Minor GC慢，执行频率低
  - 触发条件
    - 老年代空间不足
    - 永久代空间不足（JDK7)
      - 元空间取代永久代：降低FULL GC频率
    - Minor GC 晋升到老年代的平均大小大于老年代的剩余空间
    - 调用System.gc();



#### 常见辣鸡收集器

![image-20200718123004781](C:\Users\92985\AppData\Roaming\Typora\typora-user-images\image-20200718123004781.png)





#### 面试题

- 强引用，软引用，弱引用，虚引用有什么用

  - ![image-20200718124605918](https://raw.githubusercontent.com/YN-Zheng/YN-Zheng.github.io/master/typora202008/24/124957-611517.png)
  - 强引用：最普遍
    - Object obj = new Object()
    - 抛出OutOfMemoryError程序也不会回收具有强引用的对象
    - 通过将对象设置为null来弱化引用，使其被回收
  - 软引用
    - 有用但非必须的状态
    - 只有当内存空间不足，GC会回收该引用对象的内存
    - 可以实现高速缓存

  ![image-20200718124335486](E:\视频学习\并发\image-20200718124335486.png)

  - 弱引用
    - GC时会被回收
  - 虚引用
    - 不会决定对象的生命周期
    - 任何时候都可能被垃圾回收期回收
    - 跟踪对象被垃圾回收器的活动，起哨兵作用