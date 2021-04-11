## DVM以及ART对JVM的优化

#### Dalvik

- 它是Google自己设计的用于Android平台的Java虚拟机，我们用Java或者Kotlin写的代码最终都是运行在虚拟机中。
- 在Android5.0之前叫做DVM，5.0之后叫做ART（Android Runtime）
- DVM或者ART称为Android版本的Java虚拟机，这个是不准确的，他俩虽然有Java虚拟机的功能，但是要想成为Java虚拟机需要通过JCM的测试并且授权，他俩都没有被授权。
- 目前市面上的CPU主要分为两大阵营
  - Intel、AMD为首的复杂指令集CPU
    - Intel和AMD的CPU是x86架构
  - IBM、ARM为首的精简指令集CPU
    - IBM的CPU是PowerPC架构
    - ARM是ARM架构

#### Dex文件

- Android中把所有编译好的.class文件进行合并优化，然后生成一个最终class.dex文件。

- 生成的dex文件中去除了class文件中的冗余信息（比如重复字符串常量），并且结构更加紧凑。在dex解析阶段，减少了I/O操作，提高了类的查找速度

- Android这个优化也会伴随着一些副作用就是 Android 65535问题

  - 上述65535问题原因是在DVM源码中的MemberIdsSection.java类中

    ```java
    protected void orderItems(){
    	int idx = 0;
      if(items().size()>DexFormat.MAX_MEMBER_IDX+1){
        throw new DexException(Main.TO_MANY_ID_ERROR_MESSAGE)
      }
      
    }
    ```

    当我们的item个数超过DexFormat.MAX_MEMBER_IDX就会报错，这个DexFormat.MAX_MEMBER_IDX值就是65535。

#### 架构基于寄存器 & 基于栈结构

- JVM是基于栈结构来执行的，而Android却是基于寄存器的。

- 运行在Android虚拟机中的字节码和Java虚拟机中的字节码是完全不同的

  - Android字节码中更多是二地址指令和三地址指令 主要是为了让运行更快
  - 通常来看基于寄存器的指令要比基于栈指令的要少。

- 一张表格来比对基于寄存器和栈结构的指令

  | 栈式 VS 寄存器 |          对比          |
  | :------------: | :--------------------: |
  |    指令条数    |      栈式> 寄存器      |
  |    指令长度    |      寄存器>栈式       |
  |     移植性     |     栈式优于寄存器     |
  |    指令优化    |     栈式更不易优化     |
  | 解释器执行速度 | 寄存器解释器要稍微快点 |
  |  代码生成难度  |        栈式简单        |

#### 内存管理与回收

- DVM与JVM比较明显的不同就是内存结构上，主要体现在堆内存的管理上
- Dalvik虚拟机中堆被划分为2部分，分别是ActiveHeap、和Zygote Heap
- 图表示

#### 说下 Zygote和Active

- Zygote进程是在系统启动时产生的，它会完成虚拟机的初始化，库的加载，以及初始化的操作，当我们的Android系统需要一个新的虚拟机实例的时候，Zygote通过复制自身最快速的提供一个进程
- Dalvik虚拟机的堆最初只有一个，也就是Zygote进程在启动过程中创建的Dalvik虚拟机时，只有一个堆。
  - 当Zyzote进程在fork第一个应用程序进程之前，会把已经使用的堆内存划分为一个区域，把还没有使用的堆内存划分为另外一部分，前者我们成为Zygote堆，后者称为Active堆。
  - 以后无论是Zygote进程还是我们开启的应用程序进程，当他们需要分配对象的时候，都在Active堆上进行的