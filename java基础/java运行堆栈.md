# java运行堆栈分析
> 在Java虚拟机规范中制定了虚拟机字节码执行引擎的概念模型，这个概念模型成为各种虚拟机执行引擎的统一外观。

虚拟机实现的执行引擎，主要分为下面两种：
- 解释器执行
- 即时编译器执行本地代码执行

如sun classic VM内部只存在解释器，只能通过解释执行；而BEA JRockit内部只存在即时编译器，只能使用编译执行；而HotSpot中通过将两种方式混合设计，进而提高其性能。所有的执行引擎目的都是一致的 **输入字节码文件，输出执行结果**。

当虚拟机运行时，每个方法从调用开始至执行结束都对应着java虚拟机栈中的增加和删除一个栈帧，每个栈桢都包含了局部变量表、操作数栈、动态连接、方法返回地址和一些附加信息。

![线程&栈帧结构](https://upload-images.jianshu.io/upload_images/15579402-8594a04b422efbd0.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/350)

```
public class Demo {
    static int a = 10;
    int b = 100;

    public static void main(String[] args) {
        Demo demo = new Demo();
        int c = 1000;
        int d = demo.add(a, demo.b);
        demo.add(c, d);
    }

    public int add(int x, int y){
        return x+y;
    }
}
```

**1. 局部变量表**
java编译阶段就可以确定局部变量表的大小，局部变量表是储存一组变量的空间，用于储存方法参数和方法内部定义的变量；局部变量表可以容纳boolean、byte、char、short、int、float、reference、returnAddress这8种数据类型。

**2. 操作数栈**
操作数栈也称为操作栈，是一个后入先出的栈（LIFO），同局部变量表一样其最大深度在编译时就已经确定。
在程序运行时会对操作数栈进行数据的写入和提取，也就是入栈和出栈操作；程序就是通过这种方式实现参数的传递和计算。
大多数程序运行期间会对操作数栈进行优化操作，当方法A的局部变量value作为方法B的参数时，会将方法A的局部 变量表和方法B的操作数栈进行重叠处理形成一块共享区域，这样当方法B调用时无需对value进行复制和传递。

**3. 动态连接**
程序运行时当前栈帧中会包含一个指向常量池中当前栈帧所对应的方法的引用，上层方法调用指令的参数就是这个引用。
我们将静态方法、私有方法、实例构造器、父类方法这4个在类加载的时候就可以确定且运行期间不会改变（加上final方法）的方法称之为非虚方法；除了前面5种非虚方法之外的其他方法都称为虚方法。
像需要创建对象之后才能确定调用者的实例方法，在运行时将调用者与方法绑定的操作称之为**动态连接**。

**4. 方法返回地址**
方法退出有2种方式，第一种是正常完成出口，当程序遇到方法返回指令，会将可能会有的返回值传递给调用者；另一种是异常完成出口，当程序执行过程中遇到一个没有处理方法的异常操作时会导致退出。
方法正常退出时程序计时器的值可以作为返回地址，栈帧中很可能会保存这个计数器的值。
方法异常退出时返回地址要通过异常处理器表确定。

当方法退出时，相当于当前栈帧进行出栈操作，正常情况下需要恢复上层局部变量表和操作数栈，将返回值压入操作数栈中，调整程序计数器指向下一条指令。

**5. 附加信息**
虚拟机规范允许具体的虚拟机实现里面在栈帧中增加一些规范里面没有描述的信息，如调试信息。
在实际开发中，一般把动态连接、方法返回地址和其他附加信息称为栈帧信息。


```
Constant pool:
   #1 = Methodref          #7.#22         // java/lang/Object."<init>":()V
   #2 = Fieldref           #3.#23         // com/test/Demo.b:I
   #3 = Class              #24            // com/test/Demo
   #4 = Methodref          #3.#22         // com/test/Demo."<init>":()V
   #5 = Fieldref           #3.#25         // com/test/Demo.a:I
   #6 = Methodref          #3.#26         // com/test/Demo.add:(II)I
   #7 = Class              #27            // java/lang/Object
   #8 = Utf8               a
   #9 = Utf8               I
  #10 = Utf8               b
  #11 = Utf8               <init>
  #12 = Utf8               ()V
  #13 = Utf8               Code
  #14 = Utf8               LineNumberTable
  #15 = Utf8               main
  #16 = Utf8               ([Ljava/lang/String;)V
  #17 = Utf8               add
  #18 = Utf8               (II)I
  #19 = Utf8               <clinit>
  #20 = Utf8               SourceFile
  #21 = Utf8               Demo.java
  #22 = NameAndType        #11:#12        // "<init>":()V
  #23 = NameAndType        #10:#9         // b:I
  #24 = Utf8               com/test/Demo
  #25 = NameAndType        #8:#9          // a:I
  #26 = NameAndType        #17:#18        // add:(II)I
  #27 = Utf8               java/lang/Object
{
  static int a;
    descriptor: I
    flags: ACC_STATIC

  int b;
    descriptor: I
    flags:

  public com.bing.test.Demo();
    descriptor: ()V
    flags: ACC_PUBLIC
    Code:
      stack=2, locals=1, args_size=1
         0: aload_0
         1: invokespecial #1                  // Method java/lang/Object."<init>":()V
         4: aload_0
         5: bipush        100
         7: putfield      #2                  // Field b:I
        10: return
      LineNumberTable:
        line 3: 0
        line 5: 4

  public static void main(java.lang.String[]);
    descriptor: ([Ljava/lang/String;)V
    flags: ACC_PUBLIC, ACC_STATIC
    Code:
      stack=3, locals=4, args_size=1
         0: new           #3                  // class com/test/Demo
         3: dup
             4: invokespecial #4                  // Method "<init>":()V
         7: astore_1
         8: sipush        1000
        11: istore_2
        12: aload_1
        13: getstatic     #5                  // Field a:I
        16: aload_1
        17: getfield      #2                  // Field b:I
        20: invokevirtual #6                  // Method add:(II)I
        23: istore_3
        24: aload_1
        25: iload_2
        26: iload_3
        27: invokevirtual #6                  // Method add:(II)I
        30: pop
        31: return
      LineNumberTable:
        line 8: 0
        line 9: 8
        line 10: 12
        line 11: 24
        line 12: 31

  public int add(int, int);
    descriptor: (II)I
    flags: ACC_PUBLIC
    Code:
      stack=2, locals=3, args_size=3
         0: iload_1
         1: iload_2
         2: iadd
         3: ireturn
      LineNumberTable:
        line 15: 0

  static {};
    descriptor: ()V
    flags: ACC_STATIC
    Code:
      stack=1, locals=0, args_size=0
         0: bipush        10
         2: putstatic     #5                  // Field a:I
         5: return
      LineNumberTable:
        line 4: 0
}
SourceFile: "Demo.java"
```

推荐书籍：[《深入理解java虚拟机》](https://book.douban.com/subject/24722612/)