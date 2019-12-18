# 深入理解java虚拟机
	Java 内存区域与内存溢出异常
##  1.1 运行时的数据区域
### 1.1.1程序计数器Program Counter Register
改变程序计数器的值来选取下一条需要执行的指令的字节码

每个线程都要有一个独立的程序计数器，称为线程私有内存，互不影响

此内存区域是唯一一个没有规定任何OutOfMemoryError情况的区域

### 1.1.2Java虚拟机栈Java Vitual Machine Stack
虚拟机栈描述的是Java方法执行的内存模型：每个方法在执行的同时都会创建一个栈帧Stack Frame 用于存储局部变量表等

64位长度的long 和 double 数据会占用两个局部变量空间Slot

在方法运行期间不会改变局部变量表的大小

生命周期和线程相同，会抛出两种异常：
1，StackOverflowError 栈深度太大
2，OutOfMemoryError 扩展时无法申请到足够的内存


### 1.1.3本地方法栈Native Method Stack
类似于虚拟机栈

### 1.1.4 Java 堆Java Heap
所有线程共享的一块内存区域，在虚拟机启动时创建，唯一目的就是存放对象实例，所有实例都要在这里分配内存

也被称为GC堆（Garbage Collected Heap）
可以细分为新生代和老年代
还可以划分为线程私有分配缓存区TLAB（Thread Local Allocation Buffer)

逻辑上是连续的，物理上可以不连续

### 1.1.5 方法区Method Area
各个线程共享
存储已经被虚拟机加载的类信息，常量，静态变量等

也被称为永久代（Permanent Generation)
主要是针对常量池的回收和对类型的卸载

### 1.1.6 运行时常量池Runtime Constant Pool
存放运行时的各种常量

### 1.1.7 直接内存Direct Memory
本机直接分配，经常被使用

## 1.2 HotSpot虚拟机对象
### 1.2.1 对象的创建
遇到一个new指令
->检查这个指令的参数是否已经在常量池中，检查这个类是否已经被加载和解析，如果没有就先执行类的加载和解
->为新生对象分配内存（指针碰撞Bump the Pointer or空闲列表 Free List
但是这个过程不是线程安全的，有两种解决方案：
1.对分配内存空间的动作进行同步处理
2.把内存分配的动作按照线程划分在不同的空间之中进行，在java堆里先分配一块内存称为TLAB，本地线程分配缓冲（Thread Local Allocation Buffer）
->将分配的内存空间都初始化为0值（不包括对象头）
->对对象进行必要的设置，存放在对象头中（Object Header）

### 1.2.2 对象的内存布局
可以分为3个区域 ：
**对象头（header）：**
1，存储对象自身运行时的数据（Mark Word）
2，类型指针，确定这个对象时哪个类的实例

**实例数据（Instance Data）：**
数据的顺序会受到虚拟机分配策略参数FieldAllocationStyle和字段在源码中定义顺序的影响
父类定义的变量会出现在子类之前

**对齐填充（Padding）：**
对象的起始地址必须时8的整倍数，所以需要对齐