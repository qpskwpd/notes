### 第一部分

#### 第一章 走进Java

##### 1.2 Java技术体系

* 按组成部分的功能划分：
  * Java程序设计语言
  * 各种硬件平台上的Java虚拟机实现
  * Class文件格式
  * Java类库API
  * 来自商业机构和开源社区的第三方Java类库

* 前三个部分称为JDK（Java Development Kit），是支持Java程序开发的最小环境。

* Java类库API中的Java SE API子集和Java虚拟机这两部分称为JRE（Java Runtime Environment），是支持java程序运行的标准环境。

![Java技术体系](img\java虚拟机\java技术体系.jpg)

* 按服务领域划分：
  * Java Card：支持Java小程序运行在小内存设备（智能卡）上的平台。

  * Java ME（Micro Edition）：支持Java程序运行在移动终端（手机、PDA）上的平台，对Java API有精简，并加入了移动终端的针对性支持。

  * Java SE（Standard Edition）：支持面向桌面级应用的Java平台，提供完整的Java核心API。

  * Java EE（Enterprise Edition）：支持使用多层架构的企业应用的Java平台，除了提供Java SE API外，还做了大量针对性的扩充，并提供相关的部署支持。

##### 1.4 Java虚拟机

* Java虚拟机始祖
  * Sun Classic VM
  * Exact VM
    * 准确式垃圾回收：虚拟机可以知道内存中某个位置的数据具体是什么类型。
* 武林盟主
  * HotSpot VM
* 小家碧玉
  * Mobile/Embedded VM：缩水版的HotSpot、KVM
* 天下第二
  * BEA JRockit/ IBM J9
* 软硬合璧（与硬件平台绑定，软硬件配合工作）
  * BEA Liquid/Azul VM/Azul Zing VM
* 挑战者 
  * Apache Harmony/Google Android Dalvik VM
* 其他
  * Java Card VM
  * Squawk VM
  * JavaInJava
  * Maxine VM
  * Jikes RVM
  * IKVM.NET
### 第二部分 自动内存管理

#### 第二章  Java内存区域与内存溢出异常

##### 2.2 运行时数据区域

<img src="img\java虚拟机\运行时数据区.jpg" style="zoom: 33%;" />

* 程序计数器：当前线程所执行的字节码的行号指示器，分支、循环、跳转、异常处理、线程恢复等基础功能都需要依赖它完成。
  * 线程私有（生命周期与线程相同）：每条线程都需要一个独立的程序计数器，各条线程之间计数器互不影响，独立存储。
  * 执行Java方法，它记录的是正在执行的虚拟机字节码指令的地址；执行本地（Native）方法，它的值为空（Undefined）。
  * 没有规定任何OutOfMemoryError情况。
* 虚拟机栈： Java方法执行的线程内存模型。每个Java方法被执行时，虚拟机会同步创建一个栈帧用于存储局部变量表、操作数栈、动态连接、方法出口等信息。

  * 局部变量表存放编译期可知的各种基本数据类型、对象引用和returnAddress类型。在编译期完成内存分配，方法运行期间不会发生变化。

  * 两类异常：

    * StackOverflowError：请求的栈深度大于虚拟机所允许的深度。

    * OutOfMemoryError：栈扩展时无法申请到足够的内存。HotSpot虚拟机的栈容量是不可以动态扩展的。
* 本地方法栈：与虚拟机栈类似，只是为本地方法服务。
  
  * HotSpot虚拟机直接把本地方法栈和虚拟机栈合二为一。
* Java堆：虚拟机所管理的内存中最大的一块，是垃圾收集器管理的内存区域。用途为存放对象实例。
  * 线程共享（在虚拟机启动时创建）：划分出多个线程私有的分配缓冲区（Thread Local Allocation Buffer，TLAB），以提升对象分配的效率。
  * 可以处于物理上不连续的内存空间，但在逻辑上被视为连续。
  * OutOfMemoryError：没有内存完成实例分配，并且堆无法扩展时。
* 方法区： 用于存储已被虚拟机加载的类型信息、常量、静态变量、即时编译器编译后的代码缓存等数据。

  * 线程共享。

  * 早年使用永久代来实现，JDK8时改用在本地内存中实现的元空间来替代。

  * 该区域的内存回收主要是针对常量池的回收和对类型的卸载。

  * OutOfMemoryError：无法满足新的内存分配需求时。

  * 运行时常量池：方法区的一部分，用于存放编译器生成的各种字面量与符号引用。

    * 相对于Class文件常量池（编译后直接生成的常量）具备动态性，运行期间产生的常量也可以进入运行时常量池，如String类的intern()方法（常量池中存在则直接返回已存在的引用，否则将字符串加入常量池，返回该字符串的引用）。
  * OutOfMemoryError：无法申请到内存时。
    * JDK1.7后字符串常量池被移动至Java堆中。
* 直接内存：不是虚拟机运行时数据区域的一部分。JDK1.4引入的NIO（New Input/Output）类可以使用Native函数直接分配堆外内存，然后通过一个存储在Java堆中的DirectByteBuffer对象作为这块内存的引用进行操作。
  
  * OutOfMemoryError：受限制于本机总内存大小以及处理器寻址空间的限制，动态扩展时可能出现溢出异常。
##### 2.3 虚拟机对象

* 对象创建过程：

  * 当Java虚拟机遇到一条字节码new指令时，首先将去检查这个指令的参数是否能在常量池中定位到一个类的符号引用，并且检查这个符号引用代表的类是否已被加载、解析和初始化过。
  * 当类加载检查通过后，虚拟机将为新生对象分配内存，即把一块确定大小的内存块在Java堆中划分出来。
    * 分配方式通常为“指针碰撞”、“空闲列表”。
    * 为保证线程安全，解决方案为对分配内存的操作进行同步处理、把内存分配的操作按照线程划分在不同的空间中进行。
  * 内存分配完成后，虚拟机将分配的内存空间（除对象头）初始化为零值，对对象头进行必要的设置，如类的元数据信息、对象的哈希码、对象的GC分代年龄等。
  * new指令后会接着执行构造函数，即Class文件中的\<init>()方法，按照程序员的意愿对对象进行初始化。
  
* 对象的内存布局 ：划分为三个部分，对象头、实例数据和对齐填充。
  * 对象头：通常包括两类信息
    * Mark Word：存储对象自身的运行时数据，如哈希码、GC分代年龄、锁状态标志、线程持有的锁、偏向线程ID、偏向时间戳等。
    * 类型指针：对象指向它的类型元数据的指针，虚拟机通过该指针来确定该对象为哪个类的实例，但不是必须实现（使用句柄访问时，类型信息被存储在句柄中）。
    * 数组长度：当对象为数组时，用于记录数组的长度。
  * 实例数据 ：对象真正存储的有效信息。
    * 存储顺序受虚拟机分配策略和字段在Java源码中定义顺序的影响。
    * 默认分配顺序为longs/doubles、ints、shorts/chars、bytes/booleans、oops（Ordinary Object Pointers，OOPs）。
    * 相同宽度的字段被分配到一起存放，父类中定义的变量出现在子类之前。
  * 对齐填充： 不是必然存在，也没有特别含义，仅起占位符的作用。
    * 由于HotSpot虚拟机的自动内存管理系统要求对象起始地址必须是8字节的整数倍。
  
* 对象的访问定位：通常有两种方式

  * 使用句柄访问：Java堆中划分出一块内存作为句柄池，栈上的reference数据存储对象的句柄地址，句柄中包含对象实例数据和类型数据的具体地址。

    ![句柄访问](img\java虚拟机\句柄访问.jpg)

  * 使用直接指针访问：栈上的reference数据存储对象的内存地址。

    ![直接指针访问](img\java虚拟机\直接指针访问.jpg)

  * 优势：

    * 使用句柄的好处是reference中存储的是稳定句柄地址，对象被移动时指绘改变句柄中的实例数据指针，而reference本身不需要被改变。
    * 使用直接指针的好处是速度更快，省略了一次指针定位的时间开销。HotSpot主要使用这种方式。

#### 第三章 垃圾收集器与内存分配策略

##### 3.1 概述

程序计数器、虚拟机栈、本地方法栈三个区域生命周期与线程相同，栈中的栈帧随着方法的进入和退出而入栈和出栈，这几个区域的内存分配和回收都具备确定性。

Java堆和方法区具有不确定性，如一个接口的多个实现类需要的内存可能不同，一个方法执行的不同条件分支需要的内存可能不同，这是处于运行期间才能知道的内容，因此这两个区域的内存分配和回收都是动态的。

##### 3.2 如何判断对象存活

* 引用计数算法：在对象中添加一个引用计数器，每当有一个地方引用它时，计数器加一；当引用失效时，计数器减一；任何时刻计数器为零的对象就是消亡的。
  * 优势：原理简单，判定效率很高。
  * 问题：有很多例外情况需要考虑，必须配合大量的额外处理才能保证正确工作，如对象之间循环引用的问题。
* 可达性分析算法： 通过一系列成为“GC Roots”的根对象作为起始节点集，从这些节点开始，根据引用关系向下搜索，搜索过程所走过的路径称为引用链，如果某个对象到GC Roots间没有任何引用链相连（不可达），则该对象是消亡的。Java中作为GC Roots的对象包括
  * 在虚拟机栈（栈帧中的本地变量表）中引用的对象，如各个线程被调用的方法栈中的使用到的参数、局部变量、临时变量等。
  * 在方法区中类静态属性引用的对象，如Java类的引用类型静态变量。
  * 在方法区中常量引用的对象，如字符串常量池里的引用。
  * 在本地方法栈中JNI（Native方法）引用的对象。
  * Java虚拟机内部的引用，如基本数据类型对应的Class对象，一些常驻的异常对象（NullPointException、OutOfMemoryError等），系统类加载器。
  * 所有被同步锁（synchronized关键字）持有的对象。
  * 反映Java虚拟机内部情况的JMXBean、JVMTI中注册的回调、本地代码缓存等。
* 引用：JDK1.2后Java将引用分为强引用、软引用、弱引用和虚引用4种。
  * 强引用：最传统的引用定义，即类似“Object obj = new Object()"的引用关系，只要引用关系存在，这些对象就不会被回收。
  * 软引用：关联有用但非必须的对象，当发生内存溢出异常前，这些对象会被列入回收范围进行第二次回收，如果回收后还没有足够的内存，才会抛出内存溢出异常。
  * 弱引用：关联非必须的对象，但强度更弱，这些对象只能生存到下一次垃圾收集发生为止，无论当前内存是否足够。
  * 虚引用：关联的对象生存时间不会有任何影响，唯一目的是这个对象被回收时收到一个系统通知。
* finalize方法：有点像遗言方法，只会被执行一次，不推荐使用。
  * 对象在进行可达性分析后发现没有与GC Roots的引用链，则它将会被第一次标记。接着进行一次筛选，条件是该对象是否有必要执行finalize方法。如果对象没有覆盖finalize方法或已经执行过，则虚拟机视为”没有必要执行“，即直接被回收。
  * 如果对象有必要执行finalize方法，则它将进入F-Queue队列，稍后由虚拟机自动建立一个低优先级的线程去执行这些对象的finalize方法。
  * 稍后收集器将进行第二次标记，如果对象在finalize方法种将自己与引用链上任何一个对象关联，则它被移出回收集合，否则它将被回收。
* 回收方法区 ：主要回收两部分内容
  * 废弃的常量，如字面量，类、方法、字段的符号引用，类似于回收Java堆中的对象。
  * 不再使用的类，需要同时满足三个条件
    * 该类（包括子类）所有的实例都已经被回收。
    * 加载该类的类加载器已经被回收。
    * 该类对应的Class对象没有在任何地方被引用，无法在任何地方通过反射访问该类的方法。

##### 3.3 垃圾收集算法

从如何判定对象消亡的角度，划分为“引用计数式垃圾收集”和“追踪式垃圾收集”。

* 分代收集理论：
  * 建立在两个分代假说上：
    * 弱分代假说：绝大多数对象都是朝生夕灭的。
    * 强分代假说：熬过越多次垃圾收集过程的对象就越难以消亡。
    
  * 设计原则：收集器应该将Java堆划分出不同的区域，然后将回收对象依据其年龄（熬过垃圾收集过程的次数）分配到不同的区域中存储。
  
  * 困难之处：对象之间可能存在跨代引用。进行新生代的垃圾收集（Minor GC/Young GC）时不得不再额外遍历整个老年代中所有的对象来确保可达性分析结果的正确性，这样会带来很大的性能负担。因此引入第三条分代假说：
    
    * 跨代引用假说：跨代引用相对于同代引用来说仅占极少数。
    
    这样需要在新生代上建立一个全局的数据结构（记忆集），把老年代划分成若干小块，标识出老年代的哪一块内存存在跨代引用。此后进行新生代的垃圾收集时只会遍历这些内存里的老年代对象。
    
  
* 标记-清除算法 ：最早出现，最基础的垃圾收集算法。

  * 过程：首先标记出所有需要回收的对象，在标记完成后，统一回收掉所有被标记的对象。

  * 缺点：

    * 执行效率不稳定，如果Java堆中包含大量对象，且大部分是需要被回收的，则会进行大量标记和清除操作。

    * 清除后会产生大量不连续的内存碎片，可能导致分配较大对象时无法找到足够的连续内存而提前再次触发垃圾收集。

<img src="img\java虚拟机\标记-清除.png" alt="标记-清除" style="zoom: 50%;" />

* 标记-复制算法：为了解决标记-清除算法面对大量可回收对象执行效率低的问题。

  * 过程：将可用内存按容量划分为大小相等的两块，每次只使用其中一块。当这一块内存用完了，就将还存活着的对象复制到另外一块，然后把已使用过的内存空间一次清理掉。

  * 缺点：将可用内存缩小为原来的一半，空间浪费严重。

  * 回收新生代的优先算法，因为绝大多数对象熬不过第一轮收集，因此不需要按1 : 1划分内存。

    * Appel式回收：将新生代划分为一块较大的Eden空间和两块较小的Survivor空间，每次分配只使用Eden和其中一块Survivor。垃圾收集时，将Eden和Survivor中仍然存活的对象一次性复制到另一块Survivor中，然后直接清理掉Eden和使用过的Survivor。

      当Survivor不足以容纳一次新生代GC后存活的对象时，依赖其他内存区域（大多为老年代）进行分配担保，即直接将这些对象放进老年代。

<img src="img\java虚拟机\标记-复制.jpg" alt="标记-复制" style="zoom:50%;" />

* 标记-整理算法：为了解决标记-复制算法在对象存活率较高时执行效率低的问题，并且需要额外的空间进行分配担保。

  * 过程：标记过程与之前相同，但后续步骤是让所有存活的对象向内存空间一端移动，然后直接清理掉边界以外的内存。

  * 缺点：移动存活对象并更新所有引用这些对象的地方是一种负重的操作，且必须全程暂停用户应用程序。

    但是内存分配和访问比垃圾收集频率要高，因此从整个程序吞吐量（相当于分配内存的效率与垃圾收集的效率之和）来看，移动对象会更划算。

<img src="img\java虚拟机\标记-整理.jpg" alt="标记-整理" style="zoom:50%;" />

##### 3.4 HotSpot的算法细节实现 

* 根节点枚举
  * 所有收集器在此步骤都必须暂停用户线程，避免在分析过程中根节点集合的对象引用关系不断变化的情况。
  * 并不需要一个不漏地检查完所有执行上下文和全局的引用位置，由于使用准确式垃圾收集，虚拟机可以直接得到哪些地方存放着对象引用。
    * HotSpot虚拟机使用一组称为OopMap的数据结构达到这个目的。类加载完成时，虚拟机把对象内什么偏移量上是什么类型的数据计算出来，在即时编译过程中，也会在特定的位置记录下栈里和寄存器里哪些位置是引用。
  
* 安全点
  * OopMap的问题：导致它变化的指令非常多，为每一条指令都生成对应的OopMap需要大量的额外空间，因此只在安全点记录OopMap。
  * 作用：用户程序执行时并非在代码指令流的任意位置都能停顿下来开始垃圾收集，而是强制要求执行到安全点后才能暂停。
  * 选取的标准：是否具有让程序长时间执行的特征，最明显的是指令序列的复用，如方法调用、循环跳转、异常跳转等。
  * 问题：如何在垃圾收集发生时让所有线程都跑到最近的安全点？两种方案：
    * 抢先式中断：首先把所有用户线程全部中断，然后恢复不在安全点的线程，让它跑到安全点上再中断。
    
    * 主动式中断：不直接对线程操作，而是设置一个标志位，各个线程会不停地主动轮询这个标志，一旦发现中断标志为真时，就自己在最近的安全点上主动中断。
    
  - HotSpot虚拟机的一个优化：为了避免安全点过多，认为循环次数较少的话，执行时间应该不会太长，因此使用int或更小的数据类型作为索引值的循环（可数循环）默认不会设置安全点，使用long或范围更大的数据类型作为索引值的循环（不可数循环）将会被设置安全点。
  
* 安全区域
  * 安全点的问题：当用户线程处于Sleep或Blocked状态，无法响应虚拟机的中断请求自动挂起，虚拟机也不可能持续等待线程被激活。
  * 安全区域指能够确保在某一段代码片段中，引用关系不会发生变化，在这个区域中任意地方开始垃圾收集都是安全的。
  * 用户线程执行到安全区域的代码时，会标识自己进入安全区域，这段时间中虚拟机进行垃圾收集时不会管这些线程。用户线程离开安全区域时，需要检查虚拟机是否完成了需要暂停用户线程的阶段，如根节点枚举，如果完成了就继续执行，否则必须一直等待。
  
* 记忆集与卡表
  * 记忆集是一种用于记录从非收集区域指向收集区域的指针集合的抽象数据结构。
  
  * 记录所有含跨代引用的对象的数组，空间占用和维护成本高昂。可选择的更为粗犷的记录精度：
    * 字长精度：每个记录精确到一个机器字长，该字包含跨代指针。
    * 对象精度：每个记录精确到一个对象，该对象里有字段含有跨代指针。
    * 卡精度：每个记录精确到一块内存区域，该区域内有对象含有跨代指针。
    
  * 卡表最简单的形式是一个字节数组，字节数组的每一个元素都对应着其标识的内存区域中一块特定大小的内存块，称为卡页。
  
    * HotSpot的卡表标记逻辑：
  
      ```c
      CARD_TABLE[this address >> 9] = 0;
      ```
  
      HotSpot中使用的卡页是2的9次幂即512字节。
      
    * 只要卡页内有一个对象的字段存在跨代指针，则对应卡表的数组元素值标记为1（变脏）。在垃圾收集发生时，筛选出卡表中标识为1的元素，就能得出哪些卡页内存块中包含跨代指针，然后把它们加入根节点中。
  
* 写屏障

  * 卡表元素何时变脏：其他分代区域对象的引用类型字段赋值本区域对象时。

  * 如何在对象赋值的那一刻更新卡表：解释执行的场景，Java代码变为字节码，虚拟机负责执行字节码则有介入机会，而编译执行的场景，代码变为机器指令，这样必须在机器码层面上把为维护卡表的动作放到每一个赋值操作中。Q：如果是Java代码还会出现编译执行的情况吗？

  * HotSpot虚拟机通过写屏障技术维护卡表。
    
    * 写屏障可以看作虚拟机层面对“引用类型字段赋值”这个动作的AOP切面，在引用对象赋值时会产生一个环绕通知，供程序执行额外的动作（类似动态代理增强方法）。写后屏障增加更新卡表的操作。
    
  * 卡表的问题：

    * 写屏障的开销。

    * 伪共享：当多线程修改互相独立的变量时，如果这些变量恰好共享同一个缓存行，就会彼此影响而导致性能降低。由于一个卡表元素占1个字节，64个卡表元素共享一个缓存行，如果不同线程更新的对象正好处于这个卡表对应的内存区域内，就会导致更新卡表时写入的是同一个缓存行而影响性能。

      解决方法：先检查卡表标记，只有当该卡表元素未被标记过时才将其标记为变脏，即
      
      ```c
      if (CARD_TABLE[this address >> 9] != 0)
      	CARD_TABLE[this address >> 9] = 0;
      ```
      
      Q：这个更新卡表的逻辑为什么是=0？

* 并发的可达性分析

  * 并发过程中发生存活对象消失的情况：

    <img src="img\java虚拟机\对象消失.jpg" alt="对象消失" style="zoom:50%;" />

    * 白色：表示对象尚未被垃圾收集器访问过。
    * 黑色：表示对象已经被垃圾收集器访问过，且这个对象的所有引用都已经扫描过。
    * 灰色：表示对象已经被垃圾收集器访问过，但这个对象上至少有一个引用还没有被扫描过。

  * 对象消失同时满足的条件：

    * 赋值器插入了一条或多条从黑色对象到白色对象的新引用；
    * 赋值器删除了全部从灰色对象到该白色对象的直接引用或间接引用。

  * 解决办法：

    * 增量更新（incremental update）：破坏第一个条件，当黑色对象插入新的到白色对象的引用时，将这个新插入的引用记录下来，当这次扫描结束之后，再将这些记录过的引用关系中的黑色对象为根，再扫描一次（黑色对象变回灰色对象）。
    * 原始快照（snapshot at the beginning，SATB）：破坏第二个条件，当灰色对象删除指向白色对象的引用关系时，将这个要删除的引用记录下来，当这次扫描结束之后，再将这些记录过的引用关系中的灰色对象为根，再扫描一次（无论删除与否都会按照开始扫描时的对象图快照进行扫描）。

    记录操作是通过写屏障实现的。

##### 3.5 经典垃圾收集器

<img src="img\java虚拟机\经典垃圾收集器.jpg" alt="垃圾收集器" style="zoom:50%;" />

* Serial收集器：最基础、历史最久的收集器。

  * Serial/Serial Old运行示意：

    ![](img\java虚拟机\serial收集器.jpg)

  * 新生代收集器，基于标记-复制算法。

  * 单线程工作，进行垃圾收集时暂停其他所有工作线程，直到收集完成。
  
  * 额外内存消耗最小，适用于客户端模式。

* ParNew收集器：Serial收集器的多线程并行版本。
  * ParNew/Serial Old运行示意：

    ![](img\java虚拟机\parnew收集器.jpg)

  * 新生代收集器，基于标记-复制算法。

  * 多线程工作。

  * 除了Serial收集器外，只有它能与CMS收集器配合工作。

  * 适用于服务端模式，但已逐渐退出历史舞台。

* Parallel Scavenge收集器：

  * 新生代收集器，基于标记-复制算法。

  * 多线程工作。

  * 目标是达到一个可控制的吞吐量。

    * $$
      吞吐量=\frac{运行用户代码时间}{运行用户代码时间+运行垃圾收集时间}
      $$
    
    * 提供两个参数用于精确控制吞吐量。
  
    * 提供一个参数用于自适应的动态调节策略。

* Serial Old收集器：Serial收集器的老年代版本。
  * 老年代收集器，基于标记-整理算法。
  * 单线程工作。
  * 适用于客户端模式，在服务端模式下有两种用途：
    * JDK5及之前版本与Parallel Scavenge收集器搭配使用。
    * 作为CMS收集器发生失败时的后备预案。

* Parallel Old收集器：Parallel Scavenge收集器的老年代版本。

  * Parallel Scavenge/Parallel Old运行示意：

    ![](img\java虚拟机\ParallelOld收集器.jpg)

  * 老年代收集器，基于标记-整理算法。

  * 多线程工作。

  * 与Parallel Scavenge配合使用，适用于注重吞吐量或处理器资源稀缺的场合。

* CMS收集器：Concurrent Mark Sweep，是一种以获取最短回收停顿时间为目标的收集器。
  
  * 运行示意：
    
    ![](img\java虚拟机\CMS收集器.jpg)
    
  * 老年代收集器，基于标记-清除算法，分为四个步骤：
    1. 初始标记：只是标记一下GC Roots能直接关联到的对象，速度很快。
    2. 并发标记：从GC Roots的直接关联对象开始遍历整个对象图的过程，耗时较长但不需要停顿用户线程。
    3. 重新标记：为了修正并发标记期间，因用户程序继续运作而导致标记产生变动的那一部分对象的标记记录，耗时介于初始标记和并发标记之间。
    4. 并发清除：清理掉标记阶段判断为已经消亡的对象，由于不需要移动存活对象，这个阶段也可以与用户线程同时执行。
  
  * 多线程工作。
  
  * 优点：并发收集，低停顿。
  
  * 缺点：
  
    * 对处理器资源非常敏感，默认启动的回收线程数是(核心数量+3)/4，由于占用一部分线程会导致应用程序变慢。
    * 由于无法处理“浮动垃圾”，即标记和清理阶段用户线程新产生的垃圾对象，而用户线程在并行运行，如果CMS运行期间预留的内存无法满足新的对象分配，就可能出现“并发失败”而导致另一次完全收集（Full GC）的产生，即使用Serial Old收集器重新收集老年代。
    * 由于基于标记-清除算法，收集结束时会出现大量内存碎片。

* Garbage First收集器：主要面向服务端应用的全功能垃圾收集器。
  * Mixed GC模式：面向堆内存任何部分来组成回收集进行回收。

  * 基于Region的堆内存设计：
    * 把连续的Java堆划分为多个大小相等的独立区域（Region），每一个Region都可以根据需要扮演新生代的Eden空间、Survivor空间，或老年代空间。收集器对扮演不同角色的Region采用不同的策略去处理。
    * 还有一类特殊的Humongous区域，专门用来存储大对象，即大小超过了一个Region容量一半的对象。G1的大多数行为都把Humongous Region作为老年代的一部分看待。
    
  * 建立可预测的停顿时间模型：将Region作为单次回收的最小单元，跟踪各个Region中垃圾堆积的价值，即回收所获得的空间大小与回收所需要的时间，然后维护一个优先级列表，根据用户设定的收集停顿时间优先处理价值收益最大的哪些Region。

  * 细节问题：

    * 跨Region引用对象如何解决：每个Region维护自己的记忆集（双向的卡表结构）。
    * 收集线程与用户线程互不干扰如何解决：通过原始快照算法实现标记结果的修正，并且把Region中的一部分空间划分出来（设计了两个TAMS指针）用于并发回收过程中的新对象分配。
    * 可靠的停顿预测模型如何建立：通过记录每个Region的回收耗时、每个Region记忆集里的脏卡数量等可测量数据，分析一些统计信息。

  * 运行步骤：

    1. 初始标记：只标记一下GC Roots能直接关联到的对象，并且修改TAMS指针的值，让下一阶段用户线程并发运行时，能正确地在可用的Region中分配新对象。

    2. 并发标记：从GC Roots开始对堆中对象进行可达性分析，递归扫描整个堆里的对象图，找出要回收的对象。

    3. 最终标记：处理并发阶段结束后少量的原始快照记录，修正标记结果。

    4. 筛选回收：更新Region的统计数据，对各个Region的回收价值和成本进行排序，根据用户期望的停顿时间来制定回收计划，需要把回收Region中存活的对象复制到空的Region中，因此必须暂停用户线程。

       ![](img\java虚拟机\G1收集器.jpg)

  * 优点：

    * 可以指定最大停顿时间、分Region的内存布局、按收益动态确定回收集。
    * 基于标记-复制算法，垃圾收集完成后不会产生内存碎片，有利于程序长时间运行。

  * 缺点：

    * 内存占用和执行负载都比CMS要高。

##### 3.6 低延迟垃圾收集器

内存占用、吞吐量、延迟三方面表现都不错的收集器。硬件性能越好，通常会削弱内存占用的问题，并且带来更高的吞吐量，但是收集更多的内存可能会耗费更长的时间。

* 各款收集器的并发情况：

  <img src="img\java虚拟机\各款收集器并发情况.jpg" style="zoom:50%;" />

* 目标：在任意可管理的堆容量下实现垃圾收集停顿不超过10 ms。

* Shenandoah收集器：OpenJDK中才会存在的收集器。

  * 在管理内存方面，比起G1的改进：
    
    1. 支持并发的整理算法。
    
    2. 默认不使用分代收集。
    
    3. 摒弃了记忆集，使用连接矩阵的全局数据结构记录跨Region的引用。
    
         ​						 <img src="img\java虚拟机\连接矩阵.jpg" style="zoom:30%;" />
    
  * 运行步骤：
    
    1. 初始标记：标记与GC Roots直接关联的对象，需要停顿。
    2. 并发标记：遍历对象图，标记出全部可达的对象，与用户线程并发。
    3. 最终标记：处理剩余的原始快照扫描，统计出回收价值最高的Region，组成回收集，需要停顿。
    4. 并发清理：清理那些整个区域内一个存活对象都没有的Region，与用户线程并发。
    5. 并发回收：把回收集里面的存活对象先复制一份到其他未被使用的Region中，通过读屏障和”Brooks Pointers“转发指针来解决，与用户线程并发。
    6. 初始引用更新：把堆中所有指向旧对象的引用修正到复制后的新地址，但初始阶段只是为了建立一个线程集合点，确保所有收集器线程都完成复制对象的任务，需要停顿。
    7. 并发引用更新：真正进行引用更新操作，与用户线程并发。
    8. 最终引用更新：修正存在于GC Roots中的引用，需要停顿。
    9. 并发清理：经过并发回收和引用更新之后，整个回收集中所有的Region都无存活对象，调用一次并发清理。
   - 实现对象移动与用户程序并发的解决方案：
        - 保护陷阱：在被移动对象原有的内存上设置保护陷阱，用户程序访问到这里就会产生自陷中断，进入预设好的异常处理器中，在这里把访问转发到复制后的新对象上。
        - 转发指针：在原有对象布局结构的最前面统一增加一个新的引用阶段，在正常不处于并发移动的情况下，该引用指向对象自己。当对象有新的副本之后，将转发指针指向新对象。
          - 并发访问时，需要同步操作，避免修改了旧对象后又转发到没有修改的新对象上。
          - 同时设置读、写屏障拦截全部的对象访问操作。
        
    - 优点：低延迟时间。缺点：高运行负担使得吞吐量下降，需要与用户线程抢更长时间的cpu资源。
  
* ZGC收集器：基于Region内存布局的，不设分代的，使用了读屏障、染色指针和内存多重映射等技术来实现可并发的标记-整理算法的，以低延迟为首要目标的垃圾收集器。

  - 技术特点：

    1. Region可动态创建和销毁，及动态的区域容量大小。

         <img src="img\java虚拟机\ZGC内存布局.jpg" style="zoom: 50%;" />

    2. 染色指针，直接把标记信息记录在引用对象的指针上。原因：

       - AMD 64架构只支持到52位的地址总线和48位的虚拟地址空间，操作系统也存在约束，Linux 64分别支持47位的虚拟地址空间和46位的物理地址空间，windows 64支持44位的物理地址空间
       - 由于需求仍能满足（2^46 = 64 TB），将指针高4位提取出来存储4个标志信息，这样导致ZGC能够管理的内存为2^42 = 4 TB。<img src="img\java虚拟机\染色指针.jpg" style="zoom:50%;" />
       - 优势：
         1. 指针自愈：一旦某个Region的存活对象被移走后，这个Region立即就能被释放和重用，不必等待引用修正。
         2. 大幅减少在垃圾收集过程中内存屏障的使用数量，只使用了读屏障。
         3. 可以作为一种可扩展的存储结构来记录更多与对象标记、重定位相关的数据，以便进一步提高性能。

    3. 内存多重映射，改变指针中前几位的意义，这样如何在物理地址上寻址？ZGC通过将多个不同的虚拟内存地址映射到同一个物理内存地址上（意思应该是使标记位不影响真正的寻址），来使用染色指针进行正常寻址。
  * 运行步骤：
    1. 并发标记：遍历对象图做可达性分析，更新染色指针中的Marked 0、Marked 1标记位。前后也要经过初始标记、最终标记的短暂停顿。
    2. 并发预备重分配：根据特定的查询条件统计得出本次收集过程要清理哪些Region，组成重分配集。重分配集只是确定哪些对象要被复制，哪些Region要被清理，不同于G1的回收集为了找出收益最高的回收区域。
    3. 并发重分配：把重分配集中的存活对象复制到新的Region上，并为重分配集中的每个Region维护一个转发表，记录从旧对象到新对象的转向关系。
       - 优势1的原因在于：ZGC通过染色指针的标记信息可以得知一个对象是否处于重分配集中，当用户线程并发访问该对象时（旧对象已被清理），预置的读屏障将拦截这个访问，根据Region上的转发表，将访问转发到新复制的对象上，同时修正更新该引用的值，使其直接指向新对象。因此相比转发指针只有第一次转发的开销。
     4. 并发重映射：修正整个堆中指向重分配集中旧对象的所有引用，但并不迫切，因为有自愈的存在。所以它是在下一次垃圾收集的并发标记阶段做的，节省一次遍历对象图/线性扫描的开销。当所有指针被修正后，原来的转发表被释放掉。
    
  * 优点：低延迟时间，同时没有记忆集、写屏障等导致的运行负担。缺点：由于没有分代设计，承受的对象分配速率不会太高（没有专门的针对新生代的频率更高的垃圾收集）。

##### 3.7 选择合适的垃圾收集器

* Epsilon收集器：不进行垃圾收集的收集器，面向微服务应用，如只需运行几分钟，只要正确分配内存，在堆耗尽前就会退出。
* 考虑因素：
  * 应用程序的主要关注点：吞吐量、延迟、内存占用。
  * 运行应用的基础设施：硬件规格、操作系统。
  * JDK发行商及版本。

* 虚拟机及垃圾收集器日志

  * ```
    -Xlog[:[selector][:[output][:[decorators][:output-options]]]]
    ```

#####  3.8 内存分配策略

* 对象优先在Eden分配
* 大对象直接进入老年代
* 长期存活的对象进入老年代
* 动态对象年龄判定：Survivor空间中相同年龄所有对象大小的总和大于Survivor空间的一半，大于等于该年龄的对象直接进入老年代。
* 空间分配担保：进行新生代GC前，判断老年代的连续空间大于新生代对象总大小或历次晋升的平均大小。

#### 第四章 虚拟机性能监控、故障处理工具

##### 4.2 基础故障处理工具

- jps：虚拟机进程状况工具，JVM Process Status Tool，列出正在运行的虚拟机进程、执行主类名称以及这些进程的本地虚拟机唯一id。

  - 命令格式：jps [options] [hostid]

  - 主要选项：

    ![](img\java虚拟机\jps工具.jpg)

- jstat：虚拟机统计信息监视工具，JVM Statistic Monitoring Tool，监视虚拟机各种运行状态信息。

  - 命令格式：jstat [option vmid [interval[s|ms] [count]] ]

  - 主要选项：

    ![](img\java虚拟机\jstat工具.jpg)

- jinfo：Java配置信息工具，Configuration Info for Java，实时查看和调整虚拟机各项参数。
  
- 命令格式：jinfo [option] pid
  
- jmap：Java内存映像工具，Memory Map for Java，生成堆转储快照，查询finalize执行队列、堆和方法区的详细信息。

  - 命令格式：jmap [option] vmid

  - 主要选项：

    ![](img\java虚拟机\jmap工具.jpg)

- jhat：虚拟机堆转储快照分析工具，JVM Heap Analysis Tool。

- jstack：Java堆栈跟踪工具，Stack Trace for Java，生成虚拟机当前时刻的线程快照，定位线程出现长时间停顿的原因，如线程间死锁、死循环、请求外部资源导致的长时间挂起等。

  - 命令格式：jstack [option] vmid

  - 主要选项：

    ![](img\java虚拟机\jstack工具.jpg)

##### 4.3 可视化故障处理工具

- JHSDB：基于服务性代理的调试工具

  - 测试代码

    ```Java
  package cn.dut.test;
    
    public class JHSDB_TestCase {
        static class Test{
            static ObjectHolder staticObj = new ObjectHolder();
            ObjectHolder instanceObj = new ObjectHolder();
    
            void foo(){
                ObjectHolder localObj = new ObjectHolder();
                System.out.println("done");
            }
        }
        private static class ObjectHolder{}
    
        public static void main(String[] args) {
            Test test = new JHSDB_TestCase.Test();
            test.foo();
        }
    }
    ```
    
  - 使用jps -l查看测试程序的进程id。
  
  ![](img\java虚拟机\jvm_case\jps.jpg)
  - 使用jhsdb hsdb -pid xxx进入图形化模式。
  
  - Tools->Heap Parameters查看堆参数：
  
  <img src="img\java虚拟机\jvm_case\heapparameters.jpg" style="zoom:67%;" />
  
	  											   	 	<img src="img\java虚拟机\jvm_case\heapstat.jpg" style="zoom: 50%;" />
	
	- Windows->Console使用scanoops查找堆中的对象。
	  - 报错：No such type。
	- Tools->Inspecotr查看地址中对象的元信息。
	- Tools->Compute Reverse Ptrs根据堆中对象实例地址找出引用它们的指针。
	  - 也可以在console输入revptrs。
	  - 发现静态变量存在于一个Class对象实例中。JDK7后HotSpot虚拟机把静态变量与类型在Java语言一端的映射Class对象存放在一起，存储在Java堆中。实例变量存在于JHSDB_TestCase.Test对象实例中。
	- Java Thread窗口选择main线程，点击Stack Memory查看该线程的栈内存。
	  - 发现局部变量存在于线程的栈内存中。

- JConsole：Java监视与管理控制台，基于JMX的可视化监视、管理工具。

  - 测试代码：

    ```Java
    package cn.dut.test;
    
    import java.io.BufferedReader;
    import java.io.InputStreamReader;
    
    public class ThreadWating {
        public static void createBusyThread() {
            Thread thread = new Thread(new Runnable() {
                @Override
                public void run() {
                    while (true) ;
                }
            }, "testBusyThread");
            thread.start();
        }
    
        public static void createLockThread(final Object lock) {
            Thread thread = new Thread(new Runnable() {
                @Override
                public void run() {
                    synchronized (lock) {
                        try {
                            lock.wait();
                        } catch (InterruptedException e) {
                            e.printStackTrace();
                        }
                    }
                }
            }, "testLockThread");
            thread.start();
        }
    
        public static void main(String[] args) throws Exception{
            BufferedReader br = new BufferedReader(new InputStreamReader(System.in));
            br.readLine();
            createBusyThread();
            br.readLine();
            Object obj = new Object();
            createLockThread(obj);
        }
    }
    ```

    ```java
  package cn.dut.test;
    
    public class DeadThread implements Runnable {
        int a, b;
        public DeadThread(int a, int b) {
            this.a = a;
            this.b = b;
        }
        @Override
        public void run() {
            synchronized (Integer.valueOf(a)) {
                synchronized (Integer.valueOf(b)) {
                    System.out.println(a + b);
                }
            }
        }
        public static void main(String[] args) {
            for (int i = 0; i < 100; i++) {
                new Thread(new DeadThread(1, 2)).start();
                new Thread(new DeadThread(2, 1)).start();
            }
        }
    }
    ```
  
  - 概览：
  
    <img src="G:\2019黑马Java(IDEA)\md\img\java虚拟机\jvm_case\jconsole_概览.jpg" style="zoom:50%;" />
    
  - 内存页：
  
    <img src="G:\2019黑马Java(IDEA)\md\img\java虚拟机\jvm_case\jconsole_内存.jpg" style="zoom:50%;" />
  
  - 线程：
  
    <img src="G:\2019黑马Java(IDEA)\md\img\java虚拟机\jvm_case\jconsole_main线程.jpg" style="zoom:50%;" />
  
    - runnable状态，readBytes方法等待输入，但检测到没有输入流时归还执行令牌。
  
    <img src="G:\2019黑马Java(IDEA)\md\img\java虚拟机\jvm_case\jconsole_busy线程.jpg" style="zoom:50%;" />
    
    - runnable状态，但执行死循环。
    
    <img src="img\java虚拟机\jvm_case\jconsole_wating线程.jpg" style="zoom:50%;" />
    
    - waiting状态，不会被分配执行时间。
    
    <img src="img\java虚拟机\jvm_case\jconsole_死锁.jpg" style="zoom:50%;" />
    
    - 死锁：只有两个integer对象，当某个线程的两个synchronized块之间发生一次线程切换（只能保证一个synchronized块是同步的，嵌套无法保证），将会出现线程A等待线程B持有的一个integer对象，而线程B等待线程A持有的另一个integer对象。
    
  - VM概要：
  
    <img src="img\java虚拟机\jvm_case\jconsole_VM概要.jpg" style="zoom:50%;" />

- VisualVM：多合-故障处理工具，提供运行监控、故障处理、性能分析等功能。
- Java Mission Control：生产环境中可持续在线的监控工具。

##### 4.4 HotSpot虚拟机插件及工具

* HSDIS：即时编译代码的反汇编插件。
* JITWatch：可视化的编译日志分析工具。

#### 第五章 调优案例分析与实战

##### 5.2 案例

- 大内存硬件上的程序部署策略
  - 通过一个单独的虚拟机实例来管理大量的Java堆内存。关键是控制Full GC的频率。
  - 同时使用若干个Java虚拟机，建立逻辑集群来利用硬件资源。

- 集群间同步导致的内存溢出
  - 被集群共享的数据要使用类似JBossCache这种非集中式的集群缓存来同步的话，可以允许读操作频繁，因为数据在本地内存有一份副本，读取的操作不会耗费多少资源，但不应当有过于频繁的写操作，会带来很大的网络同步开销。

- 堆外内存导致的溢出错误
  - 在处理小内存或者32位的应用问题时，除了Java堆和方法区之外，其他部分的内存，如直接内存、线程堆栈、Socket缓存区、JNI代码（占用Java虚拟机本地方法栈和本地内存）、虚拟机和垃圾收集器，总和受操作系统进程最大内存的限制。

- 外部命令导致系统缓慢
  - 通过Java的Runtime.getRuntime().exec()方法调用外部命令，会复制一个新的线程去执行外部命令，频繁的创建和销毁进程带来处理器和内存上的消耗。

- 服务器虚拟机进程崩溃
  - 使用异步方式调用web服务，由于服务速度的不对等导致累积了越多web服务没有调用完成，在等待的线程和socket连接越来越多，最终导致虚拟机进程崩溃。

- 不恰当数据结构导致内存占用过大
  - 使用类似HashMap<Long, Long>的结构存储数据空间效率过低。
  - 该对象过大，导致复制算法成本很高，治标方法为将Survivor空间去掉，让新生代存活的对象在第一次Minor GC时就进入老年代。

- 由Windows虚拟内存导致的长时间停顿
  - Java的GUI程序最小化时，工作内存被自动交换到磁盘的页面文件中，这样发生垃圾收集时就可能因为恢复页面文件的操作导致不正常的垃圾收集停顿。

- 由安全点导致长时间停顿
  - 使用int循环清理超时连接，当循环体执行时间较长，而int循环不会被设置安全点，这样导致垃圾收集时其他线程必须等待这个线程进入int循环的线程执行完毕才能开始垃圾收集，导致长时间停顿的现象。

### 第三部分 虚拟机执行子系统

#### 第六章 类文件结构

##### 6.2 平台和语言无关性

- Java虚拟机不与包括Java语言在内的任何程序语言绑定，它只与Class文件这种特定的二进制文件格式所关联（平台无关的基石），Class文件中包含了Java虚拟机指令集、符号表以及若干其他辅助信息。

##### 6.3 Class类文件的结构

- 一个Class文件通常对应着唯一一个类或接口的定义信息，但类或接口不一定都得定义在文件里，也可以动态生成，直接送入类加载器中。

- Class文件是一组以8个字节为基础单位的二进制流，各个数据项目严格按照顺序紧凑地排列在文件中，中间没有任何分隔符。遇到需要占用8个字节以上空间的数据项时，按照高位在前（大端模式，高位字节在低位地址）的方式分割成若干个8个字节进行存储。
  - 采用一种伪结构存储数据，包含两种数据类型：无符号数和表。
    - 无符号数：基础数据类型，以u1、u2、u4、u8分别代表1个字节、2个字节、4个字节和8个字节的无符号数，用来描述数字、索引引用、数量值或按照UTF-8编码构成字符串值。
    
    - 表：由多个无符号数或者其他表作为数据项构成的复合数据类型，用来描述有层次关系的复合结构的数据。
    
      <img src="img\java虚拟机\class文件格式.jpg" style="zoom:67%;" />
  - 需要描述同一类型但数量不定的多个数据时，使用一个前置的容量计数器加若干个连续的数据项的形式。

- 示例代码：

  ```Java
  package cn.dut.test;
  
  public class TestClass {
      private int m;
      public int inc() {
          return m + 1;
      }
  }
  ```

- 魔数与Class文件的版本

  - 头4个字节称为魔数，唯一作用是确定这个文件是否为一个能被虚拟机接受的Class文件。Class文件的魔数为0xCAFEBABE。

  - 接着的4个字节是Class文件的版本号，包括次版本号和主版本号。主版本号从45开始，每个大版本发布主版本号加1。虚拟机拒绝执行超过其版本号的Class文件。

    ![](img\java虚拟机\TestClass_1.jpg)

- 常量池

  - Class文件里的资源仓库，是Class文件结构中与其他项目关联最多的数据。

  - 由于常量池中常量的数量不固定，在常量池入口放置一项u2类型数据，表示常量池容量计数值。它从1开始计数，将“不引用任何一个常量池项目”以索引值设为0表示。

    ![](img\java虚拟机\TestClass_2.jpg)

    - 19表明常量池中有18项常量，索引为1-18。

  - 常量池主要存放字面量（文本字符串，声明为final的常量值等）和符号引用（被模块导出或开放的包、类和接口的全限定名、字段的名称和描述符、方法的名称和描述符、方法句柄和方法类型、动态调用点和动态常量）。

  - 常量池中每一项常量都是一个表，表结构起始的第一位是个u1类型标志位，表明当前常量属于哪种常量类型。

    ![](img\java虚拟机\常量池的项目类型.png)

    - CONSTANT_Class_info类型常量表示一个类或接口的符号引用。name_index是常量池的索引，它指向常量池中一个CONSTANT_Utf8_info类型常量，此常量代表这个类或接口的全限定名。

      ![](img\java虚拟机\constant_class_info.jpg)

    - CONSTANT_Utf8_info类型常量，编码实际内容，如类名、方法名、字段名等。length说明这个utf-8编码的字符串长度是多少字节，后面紧跟长度为length字节的连续数据，是一个使用utf-8缩略编码表示的字符串。

      ![](G:\2019黑马Java(IDEA)\md\img\java虚拟机\constant_utf8_info.jpg)

      ![](img\java虚拟机\TestClass_3.jpg)
    
  - 使用javap -verbose工具分析字节码：
  
    ```
    public class cn.dut.test.TestClass
      minor version: 0
      major version: 55
      flags: (0x0021) ACC_PUBLIC, ACC_SUPER
      this_class: #3                          // cn/dut/test/TestClass
      super_class: #4                         // java/lang/Object
      interfaces: 0, fields: 1, methods: 2, attributes: 1
    Constant pool:
       #1 = Methodref          #4.#15         // java/lang/Object."<init>":()V
       #2 = Fieldref           #3.#16         // cn/dut/test/TestClass.m:I
       #3 = Class              #17            // cn/dut/test/TestClass
       #4 = Class              #18            // java/lang/Object
       #5 = Utf8               m
       #6 = Utf8               I
       #7 = Utf8               <init>
       #8 = Utf8               ()V
       #9 = Utf8               Code
      #10 = Utf8               LineNumberTable
      #11 = Utf8               inc
      #12 = Utf8               ()I
      #13 = Utf8               SourceFile
      #14 = Utf8               TestClass.java
      #15 = NameAndType        #7:#8          // "<init>":()V
      #16 = NameAndType        #5:#6          // m:I
      #17 = Utf8               cn/dut/test/TestClass
      #18 = Utf8               java/lang/Object
    {
      public cn.dut.test.TestClass();
        descriptor: ()V
        flags: (0x0001) ACC_PUBLIC
        Code:
          stack=1, locals=1, args_size=1
             0: aload_0
             1: invokespecial #1                  // Method java/lang/Object."<init>":()V
             4: return
          LineNumberTable:
            line 3: 0
    
      public int inc();
        descriptor: ()I
        flags: (0x0001) ACC_PUBLIC
        Code:
          stack=2, locals=1, args_size=1
             0: aload_0
             1: getfield      #2                  // Field m:I
             4: iconst_1
             5: iadd
             6: ireturn
          LineNumberTable:
            line 7: 0
    }
    SourceFile: "TestClass.java"
    ```
  
  - 常量池17种数据类型的结构表
  
    ![](img\java虚拟机\常量池的项目类型结构.png)

- 访问标志

  - 常量池结束后的2个字节表示类的访问标志，用于识别一些类或接口层次的访问信息，包括

    ![](img\java虚拟机\访问标志.jpg)

  - 没有使用到的标志位为0。

    ![](img\java虚拟机\TestClass_4.jpg)

- 类索引、父类索引与接口索引集合

  - 类索引、父类索引为一个u2类型的数据，接口索引集合是一组u2类型的数据集合，用于确定当前类的继承关系。

  - 类索引用于确定当前类的全限定名，父类索引用于确定父类的全限定名，接口索引集合用于描述当前类实现的接口。

    - Java.lang.Object的父类索引为0。

  - 类索引和父类索引各自指向一个CONSTANT_Class_info类型的类描述符常量，通过它找到定义在CONSTANT_Utf8_info常量中的全限定名字符串。

  - 接口索引集合的入口为u2类型的接口计数器。

    ![](img\java虚拟机\TestClass_5.jpg)

    ```
    Constant pool:
       #1 = Methodref          #4.#15         // java/lang/Object."<init>":()V
       #2 = Fieldref           #3.#16         // cn/dut/test/TestClass.m:I
       #3 = Class              #17            // cn/dut/test/TestClass
       #4 = Class              #18            // java/lang/Object
    ```

- 字段表集合：包含多个字段表，入口是一个u2类型的字段计数器。

  - 字段表用于描述接口或者类中声明的变量，包括类变量和实例变量，不包括局部变量。一个字段包含的信息有：

    - 字段的作用域，public、protected、private。
    - 实例变量还是类变量，static。
    - 可变性，final。
    - 并发可见性，volatile。
    - 能否序列化，transient。
    - 字段数据类型，基本类型、对象、数组。
    - 字段名称。

  - 字段表结构：

    ![](img\java虚拟机\字段表结构.jpg)

    - 字段访问标志：用于存放字段修饰符，包括

      ![](img\java虚拟机\字段访问标志.jpg)

    - name_index和descriptor_index分别代表字段的简单名称以及字段的描述符。

      - 全限定名：类全名中的“.”替换成“/”，“cn/dut/test/TestClass”。

      - 简单名称：没有类型和参数修饰的方法或字段名称，如“inc”和“m”。

      - 描述符：描述字段的数据类型、方法的参数列表和返回值，标识字符包括![](G:\2019黑马Java(IDEA)\md\img\java虚拟机\描述符标识字符.jpg)

        对于数组类型，每一维度使用一个前置的“[”字符描述，例如：

        1. “java.lang.String\[][]” -> “[[Ljava/lang/String;”
        2. "int[]" -> "[I"

        描述方法时，按照先参数列表、后返回值的顺序描述。参数列表按照参数的顺序放在一组小括号内，例如

        1. void inc() -> ()V
        2. java.lang.String toString() -> ()Ljava/lang/String;
        3. int indexOf(char[] source, int sourceOffset, int sourceCount, char[] target, int targetOffset, int targetCount, int fromIndex) -> ([CII[CIII)I

    - 属性表集合，用于存储一些额外的信息，如字段定义为常量，可能会在这里存放指向常量的地址。

  - 字段表集合不会出现继承而来的字段，但可能出现编译器自动添加的字段，如内部类指向外部类实例的字段。Java中字段名无法重载，但对Class文件而言，只要两个字段的描述符不完全相同，则重名是合法的（Q：这是说明Class文件的语言描述能力要更强？那用的时候怎么区分呢？）。

  - ![](img\java虚拟机\TestClass_6.jpg)

    00 01：字段表集合入口，字段计数器为1。

    00 02：字段访问标志，该字段为private。

    00 05：name_index，指向索引为5的常量。

    00 06：descriptor_index，指向索引为6的常量。

    00 00：属性表集合计数器为0。

    ```
    Constant pool:
       #1 = Methodref          #4.#15         // java/lang/Object."<init>":()V
       #2 = Fieldref           #3.#16         // cn/dut/test/TestClass.m:I
       #3 = Class              #17            // cn/dut/test/TestClass
       #4 = Class              #18            // java/lang/Object
       #5 = Utf8               m
       #6 = Utf8               I
    ```

    根据这些信息可以判断源码定义为private int m；

- 方法表集合：包含多个方法表，入口是一个u2类型的方法计数器。
  - 方法表结构：同字段表结构相同。

    ![](img\java虚拟机\字段表结构.jpg)

    - 方法访问标志：用于存放方法修饰符，包括

      ![](img\java虚拟机\方法访问标志.jpg)

    - name_index和descriptor_index分别代表方法的简单名称以及方法的描述符。

    - 属性表集合，用于存储一些额外的信息，如方法定义的代码，存放在一个名为“Code”的属性里面。

  - 父类方法在子类中没有被重写，则方法表集合中不会出现父类的方法信息。但可能出现编译器自动添加的方法，如类构造器“\<clinit>()”方法和实例构造器“\<init>()”方法。Java中重载一个方法，需要与原方法有不同的特征签名，而返回值不包含在特征签名中，所以无法仅依靠返回值进行方法重载。但对Class文件而言，只要两个方法的描述符不完全一致，即有相同的名称和特征签名，但返回值不同，也可以合法共存的（Q：这是说明Class文件的语言描述能力要更强？那用的时候怎么区分呢？）。

    - 特征签名：各个参数在常量池中的字段符号引用的集合。Java方法的特征签名包括方法名称、参数顺序、参数类型，而字节码的特征签名还包括返回值及受查异常表。

  - ![](img\java虚拟机\TestClass_7.jpg)

    00 02：方法表集合入口，方法计数器为2。

    00 01：方法访问标志，该方法为public。

    00 07：name_index，指向索引为7的常量。

    00 08：descriptor_index，指向索引为8的常量。

    00 01：属性表集合计数器为1。

    00 09：属性表的name_index，指向索引为9的常量。

    ```
    Constant pool:
       #1 = Methodref          #4.#15         // java/lang/Object."<init>":()V
       #2 = Fieldref           #3.#16         // cn/dut/test/TestClass.m:I
       #3 = Class              #17            // cn/dut/test/TestClass
       #4 = Class              #18            // java/lang/Object
       #5 = Utf8               m
       #6 = Utf8               I
       #7 = Utf8               <init>
       #8 = Utf8               ()V
       #9 = Utf8               Code
    ```

- 属性表集合：用于描述某些场景专有的信息，不要求各个属性表具有严格的顺序。Java12中预定义的属性有29项。
  - 属性表结构：![](img\java虚拟机\属性表结构.jpg)

    对每一个属性，名称从常量池中引用一个CONSTANT_Utf8_info类型的常量表示，属性值的占用位数通过一个u4的长度属性确定，属性值的结构完全自定义。

  - Code属性：方法体里面的代码经过编译后变成字节码指令存储在Code属性内，一般出现在方法表的属性集合中，接口或抽象方法表不存在该属性。结构为

    ![](img\java虚拟机\Code属性结构.jpg)

    - attribute_name_index指向一个CONSTANT_Utf8_info常量，此常量值固定为Code。

    - attribute_length指定属性值的长度。

    - max_stack代表操作数栈深度的最大值，虚拟机根据这个值分配操作栈深度。

    - max_locals代表局部变量表所需的存储空间。单位是变量槽（slot），虚拟机为局部变量分配内存所使用的最小单位，一个变量槽存放32位数据。编译器会根据变量的作用域分配变量槽，根据同时生存的最大局部变量数量和类型计算出该值大小。

    - code_length和code存储代码编译后生成的字节码指令。 前者代表字节码长度，虽然为u4长度，但实际上不允许超过65535条指令。后者为u1长度，每个字节表示一条指令。

    - ![](img\java虚拟机\TestClass_8.jpg)

      \<init>()方法的属性表

      00 01：属性表集合计数器为1。

      00 09：属性表的name_index，指向索引为9的常量。

      00 00 00 1D：属性表长度，29。

      00 01：操作数栈的最大深度，1。

      00 01：局部变量表的容量，1。

      00 00 00 05：字节码指令长度，5。

      2A B7 00 01 B1：5条字节码指令。

      00 00：LineNumberTable属性，Java源码的行号与字节码指令的对应关系。

      ```
      {
        public cn.dut.test.TestClass();
          descriptor: ()V
          flags: (0x0001) ACC_PUBLIC
          Code:
            stack=1, locals=1, args_size=1
               0: aload_0
               1: invokespecial #1                  // Method java/lang/Object."<init>":()V
               4: return
            LineNumberTable:
              line 3: 0
              
        //inc方法
      }
      ```

      args_size=1与locals=1的原因：尽管init方法没有参数，但Java中任何实例方法都可以通过“this”关键字获取此方法所属对象，实现方法为把对this关键字的访问变为对普通方法参数的访问，即this会是每个方法的一个默认参数，调用实例方法时会自动传入该参数。因此局部变量表预留一个变量槽来存放对象实例的引用。

    - exception_table代表显式异常处理表，非必须。格式为

      ![](img\java虚拟机\显式异常处理表.jpg)

      含义：当字节码从start_pc偏移量到end_pc偏移量间出现了类型为catch_type或其子类的异常，则跳转到handler_pc偏移量处继续处理。catch_type指向一个CONSTANT_Class_info类型的常量，值为0时代表任意异常情况都需要跳转到handler_pc处执行。

      一个异常示例：

      ```java
      public int inc2() {
          int x;
          try {
              x = 1;
              return x;
          } catch (Exception e) {
              x = 2;
              return x;
          } finally {
              x = 3;
          }
      }
      ```

      ```
      public int inc2();
          descriptor: ()I
          flags: (0x0001) ACC_PUBLIC
          Code:
            stack=1, locals=5, args_size=1
               0: iconst_1//try中的x=1
               1: istore_1
               2: iload_1//保存x到returnValue中，此时x=1
               3: istore_2
               4: iconst_3//finally中的x=3
               5: istore_1
        ***    6: iload_2$//将returnValue中的值放到栈顶，准备给ireturn返回
               7: ireturn
               8: astore_2//给catch中的Exception e赋值，存储在变量槽2
               9: iconst_2//catch块中的x=2
              10: istore_1
              11: iload_1//保存x到returnValue中，此时x=2
              12: istore_3
              13: iconst_3//finally中的x=3
              14: istore_1
       ***    15: iload_3//将returnValue中的值放到栈顶，准备给ireturn返回
              16: ireturn
              17: astore 4//出现了不属于java.lang.Exception及其子类的异常才会走到这里
              19: iconst_3//finally中的x=3
              20: istore_1
              21: aload 4//将异常放到栈顶，并抛出
              23: athrow
            Exception table:
               from    to  target type
                   0     4     8   Class java/lang/Exception
                   0     4    17   any
                   8    13    17   any
                  17    19    17   any
      ```

      这个异常示例的异常表有4条，比书上多了一条17-19，可能是Java的新特性。

      具体的指令还是不太懂，但可以看出执行顺序是这样的：不管有无异常发生，x赋值之后并非直接返回，而是保存在returnValue变量槽中，然后继续执行finally代码，然后返回，由于finally没有返回，这时返回的值是之前赋的值，但变量确实被修改为finally中的值。

      学完指令后：

      ```
      正常流程：
      栈：x=1	x=1->返回
      局部变量槽：x=1->x=3	x=1
      异常流程，假设第4条指令x=3处出现异常：
      栈：x=1	x=2		x=2->返回
      局部变量槽：x=1->x=2->x=3	x=1->异常对象	x=2
      出现了其他类型的异常，可以看出存了一个异常对象，将变量槽1的x修改为3，然后抛出了异常对象，没有正常返回值。
      ```
      
      当finally中加上return语句后，字节码指令变为
      
      ```
      public int inc2();
          descriptor: ()I
          flags: (0x0001) ACC_PUBLIC
          Code:
          正常流程：
          栈：x=1	x=3->返回
          局部变量槽：x=1->x=3	x=1
            stack=1, locals=5, args_size=1
               0: iconst_1
               1: istore_1
               2: iload_1
               3: istore_2
               4: iconst_3
               5: istore_1
        ***    6: iload_1
               7: ireturn
               8: astore_2
               9: iconst_2
              10: istore_1
              11: iload_1
              12: istore_3
              13: iconst_3
              14: istore_1
        ***   15: iload_1
              16: ireturn
          17: astore        4
              19: iconst_3
            20: istore_1
              21: iload_1
            22: ireturn
            Exception table:
             from    to  target type
                   0     4     8   Class java/lang/Exception
                 0     4    17   any
                 8    13    17   any
                17    19    17   any
      ```
    ```
      
    可以看出finally中的return应该是会覆盖掉其他块的return，在return前的两条指令都发生了变化，取出来的值都是finally中新赋的值，但具体的指令还是不太懂。
      
    学完指令后：
      
    ```
      正常流程：
    栈：x=1	x=3->返回
      局部变量槽：x=1->x=3	x=1
      异常流程，假设第4条指令x=3处出现异常：
      栈：x=1	x=2		x=3->返回
    局部变量槽：x=1->x=2->x=3	x=1->异常对象	x=2
      出现了其他类型的异常，可以看出存了一个异常对象，将变量槽1的x修改为3，然后将变量槽1的值加载到操作数栈返回。
    
    ```
    
    ```
  
- Exceptions属性：列举出方法中可能抛出的受查异常，即方法描述时throws关键字后面列举的异常。结构为
  
  ![](img\java虚拟机\Exceptions属性结构.jpg)
  
  - number_of_exceptions表明该方法抛出的异常有多少种，每一种受查异常使用一个exception_index_table表示，它指向常量池中一个CONSTANT_Class_info类型。
  
- LineNumberTable属性：位于Code属性内，描述Java源码行号与字节码偏移量之间的对应关系。非必需属性，但不生成该项信息会导致不显示出错行号以及无法按照源码行设置断点调试。结构为
  
  ![](img\java虚拟机\LineNumberTable属性结构.jpg)
  
  - 数量为line_number_table_length、类型为line_number_info的集合，line_number_info表包含start_pc和line_number两个u2类型的数据项。
  
- LocalVariableTable及LocalVariableTypeTable属性：前者是Code属性的子属性，用于描述栈帧中局部变量表的变量与Java源码中定义的变量之间的关系，非必需属性，但不生成该项信息会导致其他人引用该方法时，所有的参数名称会丢失。结构为
  
    ![](img\java虚拟机\LocalVariableTable属性结构.jpg)
  
    - local_variable_info表示栈帧与源码中局部变量的关联。结构为
    
    
  ![](img\java虚拟机\local_variable_info表结构.jpg)
  
    - start_pc和length代表了该局部变量生命周期开始的字节码偏移量及作用范围覆盖长度。
      - name_index和descriptor_index均指向常量池中CONSTANT_Utf8_info类型，分别代表了该局部变量的名称和描述符。
    - index代表该局部变量在栈桢的局部变量表中变量槽的位置。当该局部变量为64位时，它占用的变量槽为index和index+1两个。
  
  LocalVariableTypeTable属性在JDK5引入泛型时新增，与LocalVariableTable属性结构类似，只是把descriptor_index替换成了signature（特征签名）。这是由于引入泛型后，描述符中泛型的参数化类型在编译生成字节码文件后会被擦除掉，描述符就不能准确描述泛型类型了（见10.3节）。
  
  - SourceFile及SourceDebugExtension属性：前者用于记录生成这个Class文件的源码文件名称。非必需属性，但不生成该信息时，不会显示出错代码所属的文件名。结构为![](img\java虚拟机\SourceFile属性结构.jpg)
    - sourcefile_index指向常量池中CONSTANT_Utf8_info型常量，常量值为源码文件的文件名。
    
    SourceDebugExtension属性在JDK5时新增，用于存储额外的代码调试信息，如应用于定位JSP文件出现问题的行号。结构为![](img\java虚拟机\SourceDebugExtension属性结构.jpg)
    
    - debug_extension存储额外的调试信息，是一组通过变长UTF-8格式来表示的字符串。
    
  - ConstantValue属性：通知虚拟机自动为静态变量赋值。对实例变量的赋值是在实例构造器\<init>()方法中进行的，对于类变量的赋值可以在类构造器\<clinit>()方法中或使用ConstantValue属性。
    
    - 同时用final和static来修饰一个变量，且数据类型为基本类型或String，则生成ConstantValue属性来进行初始化。
    - 如果没有被final修饰，或非基本类型及String，则会在\<clinit>()方法中进行初始化。
  
    结构为![](img\java虚拟机\ConstantValue属性结构.jpg)
  
    - 可以看出该属性是一个定长属性，attribute_length必须为2。constantvalue_index为常量池中一个字面量常量的引用，所以它实际上也只能支持基本类型和String类型。
  
  - InnerClasses属性：用于记录内部类与宿主类之间的关联。结构为
  
    ![](img\java虚拟机\InnerClasses属性结构.jpg)
  
    - number_of_classes代表有多少个内部类信息，每个内部类信息由一个inner_classes_info描述，结构为
  
      ![](img\java虚拟机\inner_classes_info表结构.jpg)
  
      - inner_class_info_index和outer_class_info_index都指向常量池中CONSTANT_Class_info类型，分别代表内部类和宿主类的符号引用。
      - inner_name_index指向常量池中CONSTANT_Utf8_info型常量，代表该内部类的名称，匿名内部类为0。
      - inner_class_access_flags为内部类的访问标志，与类的access_flags类似。
  
  - Deprecated和Synthetic属性：都属于标志类型的布尔属性，只存在有和没有的区别。前者用于表示某个类、字段或方法已经不推荐使用，使用”@deprecated“注解进行设置。后者表示此字段或方法由编译器自动添加，也可以设置ACC_SYNTHETIC标志位来标识。结构为
  
    ![](img\java虚拟机\deprecated属性结构.jpg)
  
  - StackMapTable属性：位于Code属性的属性表中，会在虚拟机类加载的字节码验证阶段被新类型检查验证器使用。包含零至多个栈映射帧，每个栈映射帧都显示或隐式地代表一个字节码偏移量，用于表示执行到该字节码时局部变量表和操作数栈的验证类型，类型检查验证器会依据此验证类型确定该字节码指令是否符合逻辑约束。结构为
  
    ![](img\java虚拟机\StackMapTable属性结构.jpg)
  
  - Signature属性：在JDK5引入泛型时新增，可以出现在类、字段表和方法表结构的属性表中，用于记录泛型签名信息，Signature属性的出现使得Java的反射API能够获取泛型类型。
    
    - Java的泛型为采用擦除法实现的伪泛型，源码中所有的泛型信息（类型变量、参数化类型）在编译成字节码后都统统被擦除掉。好处是实现简单（主要修改Javac编译器），非常容易实现backport，运行期也能够节省一些类型所占的内存空间。坏处是运行期无法将泛型类型与用户定义的普通类型同等对待，如运行期做反射时无法获得泛型信息。
    
    结构为![](img\java虚拟机\Signature属性结构.jpg)
    
    - signature_index是一个对常量池CONSTANT_Utf8_info型常量的索引，表示类签名或方法类型签名或字段类型签名。
    
  - BootstrapMethods属性：在JDK7时新增，位于类文件的属性表中，用于保存invokedynamic指令引用的引导方法限定符（见后续invokedynamic指令的运作原理）。结构为![](img\java虚拟机\BootstrapMethods属性结构.jpg)
  
    - num_bootstrap_methods表示引导方法限定符的数量。
    - bootstrap_method表结构为![](img\java虚拟机\bootstrapmethod表结构.jpg)
      - bootstrap_method_ref指向常量池中CONSTANT_MethodHandle_info类型常量。
      - num_bootstrap_arguments表示bootstrap_arguments数组的成员数量。
      - bootstrap_arguments每个成员指向常量池中一个特定类型的常量。
  
  - MethodParameters属性：在JDK8时新增，位于方法表的属性表中，用于记录方法的各个形参名称和信息。结构为![](img\java虚拟机\MethodParameters属性结构.jpg)
  
    - parameter表结构为![](img\java虚拟机\parameter表结构.jpg)
      - name_index指向常量池中CONSTANT_Utf8_info类型常量，代表该参数的名称。
      - access_flags是参数的状态指示器，包含一种或多种：
        1. 0x0010（ACC_FINAL）：参数被final修饰。
        2. 0x1000（ACC_SYNTHETIC）：参数由编译器自动生成。
        3. 0x8000（ACC_MANDATED）：参数在源文件中隐式定义，如this关键字。
  
  - 模块化相关属性：在JDK9时新增了Module、ModulePackages和ModuleMainClass三个属性用于支持模块化相关功能。
  
    Module属性：结构为![](img\java虚拟机\Module属性结构.jpg)
  
    - model_name_index指向常量池中CONSATNT_Utf8_info类型常量，代表该模块的名称。
    - module_flags是模块的状态指示器，包含一种或多种：
      1. 0x0020（ACC_OPEN）：模块是开放的。
      2. 0x1000（ACC_SYNTHETIC）：模块由编译器自动生成。
      3. 0x8000（ACC_MANDATED）：模块在源文件中隐式定义。
  
    - module_version_index指向常量池中CONSATNT_Utf8_info类型常量，代表该模块的版本号。
    - exports每个表都代表一个被模块所导出的包，export表结构为![](img\java虚拟机\export表结构.jpg)
      - exports_index指向常量池中CONSATNT_Package_info类型常量，代表被该模块导出的包。
      - exports_flag是该导出包的状态指示器，包含一种或多种：0x1000，0x8000。
      - exports_to_count是该导出包的限定计数器，为零表明该包无限定，完全开放。不为零，则exports_to_index数组的每个成员指向常量池中CONSATNT_Module_info类型常量，代表只有这些模块才能访问该导出包的内容（类型应该是u2，而非export）。
  
    ModulePackages属性：用于描述该模块中所有的包，结构为![](img\java虚拟机\ModulePackages属性结构.jpg)
  
    - package_index数组每个成员指向常量池中CONSATNT_Package_info类型常量，代表当前模块中的一个包。
  
    ModuleMainClass属性：用于确定该模块的主类，结构为![](img\java虚拟机\ModuleMainClass属性结构.jpg)
  
    - main_class_index指向常量池中CONSATNT_Class_info类型常量，代表该模块的主类。
  
  - 运行时注解相关属性：为了存储源码中的注解信息，JDK5和JDK8时新增了RuntimeVisibleAnnotations、RuntimeInvisibleAnnotations、RuntimeVisibleParameterAnnotations、RuntimeInvisibleParameterAnnotations、RuntimeVisibleTypeAnnotations、RuntimeInvisibleTypeAnnotations六个属性。
  
    RuntimeVisibleAnnotations，记录类、字段或方法的声明上记录的运行时可见注解，使用反射API来获取注解时就是通过该属性得到的。结构为![](img\java虚拟机\RuntimeVisibleAnnotations属性结构.jpg)
  
    - annotations数组每个成员代表一个运行时可见注解，annotation表结构为![](img\java虚拟机\annotation表结构.jpg)
      - type_index指向常量池中CONSTANT_Utf8_info类型常量，该常量以字段描述符的形式表示一个注解。
      - element_value_pairs数组每个成员都是一个键值对，代表该注解的参数和值。

##### 6.4 字节码指令简介

- Java虚拟机的指令由一个字节长度的、代表着某种特定操作含义的数字（操作码）以及跟随其后的零至多个代表此操作所需的参数（操作数）构成。

- 字节码指令集的缺点：长度为一个字节意味着指令数量不超过256条，而Class文件格式放弃了操作数长度对齐，因此虚拟机在处理超过一个字节的数据时必须在运行时从字节中重建出具体数据的结构，导致解释执行字节码时将损失一些性能。优点：操作码短小精干，而放弃操作数的长度对齐也可以节省大量的填充和间隔符号。

- Java虚拟机指令集所支持的数据类型：![](img\java虚拟机\虚拟机指令集所支持的数据类型.jpg)
  - 大部分指令都没有支持byte、char、short，没有任何指令支持boolean。
  - 编译器会在编译期或运行期将byte、short类型的数据带符号扩展为相应的int类型数据，将boolean、char类型的数据零位扩展为相应的int类型数据。

- 加载和存储指令：用于将数据在栈帧中的局部变量表和操作数栈之间传输。包括

  - 将一个局部变量加载到操作数栈：iload、iload_\<n>、lload、lload_\<n>、fload、fload_\<n>、dload、dload_\<n>、aload、aload_\<n>

  - 将一个数值从操作数栈存储到局部变量表：istore、istore_\<n>、lstore、lstore_\<n>、fstore、fstore_\<n>、dstore、dstore\<n>、astore、astore_\<n>

  - 将一个常量加载到操作数栈：bipush、sipush、ldc、ldc_w、ldc2_w、aconst_null、iconst_m1、iconst_\<i>、lconst_\<l>、fconst_\<f>、dconst_\<d>

  - 扩充局部变量表的访问索引：wide

    带有\<n>的指令为带有一个操作数的通用指令的特殊形式，它们省略掉了显式的操作数，不需要进行取操作数的动作，因为操作数就隐含在指令中。

- 运算指令：用于对两个操作数栈上的值进行某种运算，并把结果重新存入操作数栈顶。分两种：对整形数据运算和对浮点型数据运算。包括
  - 加法指令：iadd、ladd、fadd、dadd
  
  - 减法指令：isub、lsub、fsub、dsub
  
  - 乘法指令：imul、lmul、fmul、dmul
  
  - 除法指令：idiv、ldiv、fdiv、ddiv
  
  - 求余指令：irem、lrem、frem、drem
  
  - 取反指令：ineg、lneg、fneg、dneg
  
- 位移指令：ishl、ishr、iushr、lshl、lshr、lushr

  - 按位或指令：ior、lor
  
  - 按位与指令：iand、land
  
  - 按位异或指令：ixor、lxor
  
  - 局部变量自增指令：iinc
  
  - 比较指令：dcmpg、dcmpl、fcmpg、fcmpl、lcmp
  
  运算指令不会导致虚拟机抛出运行时异常，除了处理整形数据时的除数为零。
  
- 类型转换指令：将两种不同的数值类型相互转换。Java虚拟机直接支持小范围类型向大范围类型的安全转换，如int->long、float、double，long->float、double或float->double。处理窄化类型转换时必须显式地使用转换指令完成，包括i2b、i2c、i2s、l2i、f2i、f2l、d2i、d2l、d2f。

  数值类型的窄化转换指令不会导致虚拟机抛出运行时异常。

- 对象创建与访问指令：Java虚拟机对类实例和数组的创建与操作使用不同的字节码指令。包括
  - 创建类实例的指令：new
  - 创建数组的指令：newarray、anawarray、multianewarray
  - 访问类字段和实例字段的指令：getfield、putfield、getstatic、putstatic
  - 把一个数组元素加载到操作数栈的指令：baload、caload、saload、iaload、laload、faload、daload、aaload
  - 将一个操作数栈的值存储到数组元素中的指令：bastore、castore、sastore、iastore、lastore、fastore、dastore、aastore
  - 取数组长度的指令：arraylength
  - 检查类实例类型的指令：instanceof、checkcast

- 操作数栈管理指令：用于直接操作操作数栈的指令，包括
  - 将操作数栈的栈顶一个或两个元素出栈：pop、pop2
  - 复制栈顶一个或两个数值并将复制值或双份的复制值重新压入栈顶：dup、dup2、dup_x1、dup2_x1、dup_x2、dup2_x2
  - 将栈最顶端的两个数值互换：swap

- 控制转移指令：让虚拟机有条件或无条件地从指定位置指令的下一条指令继续执行程序，包括

  - 条件分支：ifeq、iflt、ifle、ifne、ifgt、ifge、ifnull、ifnonnull、if_icmpeq、if_icmpne、if_icmplt、if_icmpgt、if_icmple、if_icmpge、if_acmpeq和if_acmpne
  - 符合条件分支：tableswitch、lookupswitch
  - 无条件分支：goto、goto_w、jsr、jsr_w、ret

  对于boolean、byte、char和short类型的条件分支比较操作，都使用int类型的比较指令完成，对于long、float和double类型的条件分支比较操作，会先执行相应类型的比较运算指令（dcmpg、dcmpl、fcmpg、fcmpl、lcmp），返回一个整型值到操作数栈中，再执行int类型的条件分支比较操作完成整个分支跳转。

  jsr和ret在JDK7时被禁止。

- 方法调用和返回指令：见后续，方法调用的一些指令如

  - invokevirtual指令：用于调用对象的实例方法，根据对象的实际类型进行分派（虚方法分派）。
  - invokeinterface指令：用于调用接口方法，它会在运行时搜索一个实现了这个接口方法的对象，找出适合的方法进行调用。
  - invokespecial指令：用于调用一些需要特殊处理的实例方法，包括实例初始化方法、私有方法和父类方法。
  - invokestatic指令：用于调用类静态方法。
  - invokedynamic指令：用于在运行时动态解析出调用点限定符所引用的方法，并执行该方法。该指令的分派逻辑是由用户所设定的引导方法决定的。

  方法返回指令根据返回值类型区分，包括ireturn、lreturn、freturn、dreturn和areturn，以及return用于void方法、实例初始化方法、类和接口的类初始化方法使用。

- 异常处理指令：显式抛出异常的操作（throw语句）都由athrow指令来实现，其他一些运行时异常会在Java虚拟机检测到时自动抛出，如除数为零时抛出的ArithmeticException。处理异常的操作（catch语句）不再由字节码指令实现，而是采用异常表来完成。

- 同步指令：Java虚拟机支持方法级的同步和方法内部一段指令序列的同步，这两种同步结构都是使用管程（Monitor，锁）来实现。

  方法级的同步是隐式的，无需通过字节码指令来控制，它实现在方法调用和返回操作中。方法调用时，首先检查方法表中访问标志的ACC_SYNCHRONIZED是否被设置，如果设置了，执行线程要求先持有锁，然后才能执行方法，当方法完成时释放锁。

  同步一段指令集序列是由synchronized语句块来表示的，monitorenter和monitorexit两条指令来支持该关键字的语义。

  ```Java
public void onlyMe(TestClass f){
	    synchronized (f){
        doSomething();
      }
  }
  ```
  
  ```
  public void onlyMe(cn.dut.test.TestClass);
    descriptor: (Lcn/dut/test/TestClass;)V
    flags: (0x0001) ACC_PUBLIC
    Code:	//栈：f对象		this	f引用
    		//局部变量槽：this	f对象		f引用		异常对象
      stack=2, locals=4, args_size=2
         0: aload_1	//将对象f入栈
         1: dup	//复制栈顶元素（f的引用）
         2: astore_2	//将栈顶元素存储到局部变量表变量槽2中
         3: monitorenter	//以栈顶元素f作为锁，开始同步
         4: aload_0	//将局部变量槽0，即this指针入栈
         5: invokevirtual #4	// Method doSomething:()V
         8: aload_2	//将局部变量槽2的元素（即f）入栈
         9: monitorexit	//退出同步
        10: goto 18	//方法正常结束，跳到18
        13: astore_3	//这里是异常路径
        14: aload_2	//将局部变量槽2的元素（即f）入栈
        15: monitorexit	//退出同步
	      16: aload_3	//将局部变量槽3的元素（即异常对象）入栈
	      17: athrow	//抛出异常对象
	      18: return	//方法正常返回
	    Exception table:
	       from    to  target type
	           4    10    13   any
	          13    16    13   any
	```
	
	无论方法是否正常完成，monitorenter指令必须对应一条monitorexit指令。虽然方法没有处理异常的代码，但编译器会自动生成一个异常处理程序，可以处理所有的异常，确保monitorexit指令会执行。

##### 6.6 Class文件结构的发展

- Class文件格式的改进，都集中在访问标志、属性表这些原本就是可扩展的数据结构中添加内容。
- 新增的属性大部分是用于支持Java中新出现的语言特性，如枚举、变长参数、泛型、动态注解等；还有一些是为了支持性能改进和调试信息，如新类型校验器用到的StackMapTable属性及对非Java代码调试用到的SourceDebugExtension属性。

#### 第七章 虚拟机类加载机制

##### 7.1 概述

- Java虚拟机把描述类的数据从Class文件加载到内存，并对数据进行校验、转换解析和初始化，最终形成可以被虚拟机直接使用的Java类型，这个过程被称作虚拟机的类加载机制。

- 运行期动态加载和动态连接是Java动态扩展的基础。

##### 7.2 类加载的时机

- 一个类型从被加载到虚拟机内存中开始，到卸载出内存为止，它的整个生命周期将会经历加载、验证、准备、解析、初始化、使用和卸载七个阶段，其中验证、准备、解析三个部分统称为连接。![](img\java虚拟机\类的生命周期.jpg)

  - 加载、验证、准备、初始化、卸载这五个阶段开始的顺序是确定的，而解析阶段在某些情况下可以在初始化阶段之后再开始，这是为了支持Java的动态绑定特性。

  - 加载阶段何时开始由虚拟机的具体实现来自由把握。

  - 初始化阶段有且只有六种情况必须开始，称为对一个类型的主动引用：

    1. 遇到new、getstatic、putstatic或invokestatic这四条字节码指令时，如果类型没有进行过初始化，则需要先触发其初始化。
    2. 使用java.lang.reflect包的方法对类型进行反射调用的时候，如果类型没有进行过初始化，则需要先触发其初始化。
    3. 当初始化类的时候，如果发现其父类还没有进行过初始化，则需要先触发其父类的初始化。
    4. 当虚拟机启动时，用户需要指定一个要执行的主类（包含main()方法），虚拟机会先初始化这个主类。
    5. 当使用JDK7新加入的动态语言支持时，如果一个java.lang.invoke.MethodHandle实例最后的解析结果为REF_getStatic、REF_putStatic、REF_invokeStatic、REF_newinvokeSpecial四种类型的方法句柄，并且这个方法句柄对应的类没有进行过初始化，则需要先触发其初始化。
    6. 当一个接口中定义了JDK8新加入的默认方法时，如果有这个接口的实现类发生了初始化，那么该接口要在其之前被初始化。

  - 被动引用的情况：

    ```java
    package cn.dut.test;
    
    public class ClassLoading {
        public static void main(String[] args) {
            System.out.println(SubClass.value);
        }
    }
    
    class SuperClass {
        static {
            System.out.println("SuperClass init!");
        }
    
        public static int value = 123;
    }
    
    class SubClass extends SuperClass{
        static {
            System.out.println("SubClass init!");
        }
    }
    ```

    运行结果只有“SuperClass init!”及123，这是因为对于静态字段，只有直接定义这个字段的类才会被初始化，因此通过子类引用父类中的静态字段，只会触发其父类的初始化。

    ```java
    public class ClassLoading {
        public static void main(String[] args) {
            SuperClass[] sca = new SuperClass[10];
        }
    }
    ```

    运行结果为空，说明SuperClass并没有被初始化，即通过数组定义引用类，不会触发该类的初始化。

    但这段代码触发了另一个名为“[Lcn.dut.test.SuperClass”的类的初始化，它是一个由虚拟机自动生成的、直接继承于java.lang.Object的子类，创建动作由字节码指令newarray触发。该类代表了一个元素类型为“cn.dut.test.SuperClass”的一维数组，数组中的属性和方法都实现在该类中。

    ```java
    package cn.dut.test;
    
    public class ClassLoading {
        public static void main(String[] args) {
            System.out.println(ConstClass.HELLOWORLD);
        }
    }
    class ConstClass {
        static {
            System.out.println("ConstClass init!");
        }
        public static final String HELLOWORLD = "hello world!";
    }
    ```

    运行结果只有”hello world!“，说明ConstClass并没有被初始化，这是因为常量在编译阶段会存入调用类的常量池中，本质上没有直接引用到定义常量的类，因此不会触发定义常量的类的初始化。

  - 接口的初始化过程，与类的区别在于，一个类在初始化时，要求其父类全部已经初始化过。而一个接口在初始化时，并不要求其父接口全部完成初始化，只有在真正使用到父接口（如引用父接口中定义的常量）的时候才会初始化。接口的实现类在初始化时也不会执行接口的初始化方法。

##### 7.3 类加载的过程

- 加载：
  - 加载阶段，虚拟机完成三件事情
    1. 通过一个类的全限定名来获取定义此类的二进制字节流。
       - 该动作非常灵活，如从ZIP压缩包中读取、从网络中获取、运行时计算生成（动态代理）、由其他文件生成（如JSP）、从数据库中读取、从加密文件中获取等等。
    2. 将这个字节流所代表的静态存储结构转化为方法区的运行时数据结构。
       - 方法区的数据存储格式同样完全由虚拟机自行定义。
       - 该步在文件格式的验证阶段结束后才能完成。
    3. 在堆内存中生成一个代表这个类的java.lang.Class对象，作为方法区这个类的各种数据的访问入口。
  - 数组类本身不通过类加载器创建，它是由虚拟机直接在内存中动态构造出来的，但数组类的元素类型（去掉所有维度）最终还是要靠类加载器完成加载。一个数组类的创建过程遵循以下规则：
    - 如果数组的组件类型（去掉第一个维度）是引用类型，那就递归采用加载过程去加载该组件类型，该数组将被标识在加载该组件类型的类加载器的类名称空间上。
    - 如果数组的组件类型不是引用类型（如int），该数组将被标记为与引导类加载器关联。
    - 数组类的可访问性与它的组件类型的可访问性一致，如果组件类型不是引用类型，它的数组类的可访问性默认为public。

- 验证：确保Class文件的字节流中包含的信息符合《Java虚拟机规范》的约束要求，这是因为Java代码层面做不到的事情在字节码层面都是可以实现的。Q：编译后的结果应该不会出现这些情况吧，防止的是手动篡改？
  - 验证阶段，虚拟机大致完成四个阶段的校验

    1. 文件格式验证：字节流是否符合Class文件格式的规范，并能被当前版本的虚拟机处理。可能包括

       - 是否以魔数0xCAFEBABE开头。

       - 主、次版本号是否在当前虚拟机的接受范围以内。

       - 常量池的常量中是否有不被支持的常量类型（常量tag标志）。

       - 指向常量的各种索引值中是否有指向不存在的常量或不符合类型的常量。

       - CONSTANT_Utf8_info型的常量中是否有不符合UTF-8编码的数据。

       - Class文件中各个部分及文件本身是否有被删除的或附加的其他信息。
  
       该阶段是基于二进制字节流，验证通过后才能存储进方法区中，后面三个验证阶段是基于方法区的存储结构进行验证。
  
    2. 元数据验证：对类的元数据信息进行语义校验，以保证不存在与《Java语言规范》定义相悖的元数据信息。可能包括

       - 该类是否有父类（除了java.lang.Object外）。
   - 该类的父类是否继承了不允许被继承的类（final修饰的类）。Q：为什么说该类的父类？
       - 如果该类不是抽象类，是否实现其父类或接口中要求实现的所有方法。
       - 类中的字段、方法是否与父类产生矛盾（如覆盖了父类的final字段，出现不符合规则的方法重载，如方法参数相同返回值类型不同等）。
  
    3. 字节码验证：对类的方法体字节码进行校验分析，以保证方法运行时不会危害到虚拟机的安全。可能包括：
    
       - 保证任意时刻操作数栈的数据类型与指令代码序列都能配合工作，如避免”在操作数栈放置了一个int类型数据，使用时却按long类型加载入本地变量表“。
       - 保证任何跳转指令都不会跳转到方法体以外的字节码指令上。
       - 保证方法体中的类型转换总是有效的，如可以把子类对象赋值给父类引用，但不能把父类对象赋值给子类引用，或其他毫无继承关系的数据类型。
       
       为了避免过多的执行时间消耗在该阶段，JDK6之后方法体Code属性表新增了一项”StackMapTable“属性，它描述了方法体所有的基本块开始时本地变量表和操作数栈应有的状态，在字节码验证期间，不再需要根据程序推导这些状态的合法性，而是检查StackMapTable属性中的记录是否合法即可。
       
    4. 符号引用验证：对类自身以外的各类信息进行匹配性校验，即该类是否缺少或被禁止访问它依赖的某些外部类、方法、字段等资源，以保证解析阶段能正常执行。可能包括：
  
       - 符号引用中通过字符串描述的全限定名是否能找到对应的类。
       - 在指定类中是否存在符合方法的字段描述符及简单名称所描述的方法和字段。
       - 符号引用中的类、字段、方法的可访问性，是否可被当前类访问。
  
          该阶段发生在虚拟机将符号引用转化为直接引用时，该转化动作将在解析阶段中发生。
  
  - 如果程序运行的全部代码已经被反复使用和验证过，可以考虑关闭验证阶段，以缩短虚拟机类加载的时间。
  
- 准备：为类中定义的变量（静态变量）分配内存并设置初始值的阶段。JDK7之前它们都被分配在方法区内，JDK8及之后，它们会随着Class对象一起存放在Java堆中。

  - 此时进行内存分配的仅包括类变量，不包括实例变量。

  - 另外这里所说的初始值通常情况为0值。![](img\java虚拟机\基本数据类型的零值.jpg)

    ```java
    public static int value = 123;
    ```

    准备阶段会将value初始化为0，而赋值为123的动作要到类的初始化阶段才会被执行。

    特殊情况为该字段的属性表中存在ConstantValue属性，则准备阶段就会被初始化为该属性所指定的初始值。

    ```java
    public static final int value = 123;
    ```

    编译时javac将会为value生成ConstantValue属性，在准备阶段虚拟机就会将value初始化为123。

- 解析：将常量池内的符号引用替换为直接引用的阶段。注意符号引用为该类依赖的某些外部类、方法、字段等资源，区别于本类中的字段表集合、方法表集合。

  - 符号引用和直接引用：

    - 符号引用：以一组符号来描述所引用的目标，符号可以是任何形式的字面量，如CONSTANT_Class_info符号引用（#3）通过name_index（#17）找到一个CONSTANT_Utf8_info常量（cn/dut/test/TestClass）。

    	```
    Constant pool:
       #1 = Methodref          #4.#15         // java/lang/Object."<init>":()V
       #2 = Fieldref           #3.#16         // cn/dut/test/TestClass.m:I
       #3 = Class              #17            // cn/dut/test/TestClass
       #4 = Class              #18            // java/lang/Object
       #5 = Utf8               m
       #6 = Utf8               I
       #7 = Utf8               <init>
       #8 = Utf8               ()V
       #9 = Utf8               Code
      #10 = Utf8               LineNumberTable
      #11 = Utf8               inc
      #12 = Utf8               ()I
      #13 = Utf8               SourceFile
      #14 = Utf8               TestClass.java
      #15 = NameAndType        #7:#8          // "<init>":()V
      #16 = NameAndType        #5:#6          // m:I
      #17 = Utf8               cn/dut/test/TestClass
      #18 = Utf8               java/lang/Object
    	```

    - 直接引用：可以直接指向目标的指针、相对偏移量或一个能间接定位到目标的句柄。

  - 解析阶段发生的具体时间依赖于虚拟机实现，可能在类加载阶段时就进行解析，也可能等到一个符号引用被使用前才解析。但执行用于操作符号引用的字节码指令前必须对它们所使用的符号引用进行解析。

  - 解析动作主要针对类或接口（CONSTANT_Class_info）、字段（CONSTANT_Fieldref_info）、类方法（CONSTANT_Methodref_info）、接口方法（CONSTANT_InterfaceMethodref_info）、方法类型（CONSTANT_MethodType_info）、方法句柄（CONSTANT_MethodHandle_info）和调用点限定符（CONSTANT_Dynamic_info和CONSTANT_InvokeDynamic_info）这七类符号引用进行。
  
    1. 类或接口的解析：当前代码所处的类为D，把一个从未解析过的符号引用N解析为一个类或接口C的直接引用，需要三个步骤：
  
       - 如果C不是一个数组类型，则虚拟机将把代表N的全限定名传递给D的类加载器去加载C。
       - 如果C是一个数组类型，且数组的元素类型为对象，则虚拟机正常加载元素类型，然后生成一个数组对象。
       - 以上两步没有异常，表示C已经成为一个有效的类或接口了，但在D真正访问C之前还要进行符号引用验证，确保D具有对C的访问权限。Q：意思是先解析然后验证？
  
       在JDK9引入模块化之后，还需要检查模块间的访问权限，如果D拥有对C的访问权限意味着：
  
       - 被访问类C是public的，并且与访问类D处于同一个模块。
       - 被访问类C是public的，不与访问类D处于同一个模块，但C的模块允许D的模块进行访问。
       - 被访问类C不是public的，但它与访问类D处于同一个包中。
  
       至少一条成立。
  
    2. 字段解析：解析一个未被解析过的字段符号引用，首先对字段表（CONSTANT_Fieldref_info）内class_index项中索引的CONSTANT_Class_info符号引用进行解析，然后对该类或接口C进行后续搜索：
    
       - 如果C本身包含了简单名称和字段描述符都与目标相匹配的字段，则返回这个字段的直接引用，查找结束。
       - 否则，如果C中实现了接口，将会按照继承关系从下往上递归搜索各个接口和它的父接口，如果接口中包含了简单名称和字段描述符都与目标相匹配的字段，则返回这个字段的直接引用，查找结束。
       - 否则，如果C不是java.lang.Object，将会按照继承关系从下往上递归搜索其父类，如果在父类中包含了简单名称和字段描述符都与目标相匹配的字段，则返回这个字段的直接引用，查找结束。
       - 否则，查找失败。
       - 查找成功后将会对该字段进行权限验证。
    
       同名字段在父类和接口同时出现，或在多个接口中同时出现时，编译器可能会拒绝编译。
    
    3. 方法解析：与字段解析类似，首先对方法表（CONSTANT_Methodref_info）内class_index项中索引的方法所属的类或接口的符号引用进行解析，然后对该类或接口C进行后续搜索：
    
       - 如果类的方法表中发现C为接口，直接抛出异常。
       - 否则，如果类C中有简单名称和方法描述符都与目标相匹配的方法，则返回这个方法的直接引用，查找结束。
       - 否则，在类C的父类中递归查找，如果有简单名称和方法描述符都与目标相匹配的方法，则返回这个方法的直接引用，查找结束。
       - 否则， 在类C实现的接口列表及它们的父接口中递归查找是否有简单名称和方法描述符都与目标相匹配的方法，如果存在匹配的方法，说明类C是一个抽象类，查找结束，抛出异常。
       - 否则，查找失败。
       - 查找成功后会对该方法进行权限验证。
    
    4. 接口方法解析：与方法解析类似，首先对接口方法表（CONSTANT_InterfaceMethodref_info）内class_index项中索引的方法所属的类或接口的符号引用进行解析，然后对该类或接口C进行后续搜索：
    
       - 如果接口的方法表中发现C为类，直接抛出异常。
       - 否则，如果接口C中有简单名称和方法描述符都与目标相匹配的方法，则返回这个方法的直接引用，查找结束。
       - 否则，在接口C的父接口中递归查找，直到java.long.Object类为止，如果有简单名称和方法描述符都与目标相匹配的方法，则返回这个方法的直接引用，查找结束。由于Java的接口允许多重继承，如果存在多个匹配方法，将会从这多个方法中返回一个并结束查找，与字段类似，编译器可能拒绝编译。
       - 否则，查找失败。
       - JDK9之前的接口方法均为public，也没有模块化的访问约束，因此不存在访问权限问题；JDK9之后接口方法增加了静态私有方法，也有了模块化的访问约束，因此需要对该方法进行权限验证。

- 初始化：根据程序代码中的内容去初始化类变量和其他资源，即执行类构造器\<clinit>()方法的过程。

  - \<clinit>()方法是由javac自动收集类中的所有类变量的赋值动作和静态语句块中的语句合并产生的。静态语句块中只能访问定义在它之前的变量，定义在它之后的变量可以赋值但不能访问。

    ```java
    public class Test {
    	static {
    		i = 0;	//编译通过
    		System.out.print(i);	//illegal foward reference
    	}
    	static int i = 1;
    }
    ```

  - \<clinit>()方法与构造函数（即实例构造器\<init>()）不同，它不需要显式地调用父类构造器，虚拟机会确保父类的\<clinit>()方法先执行。
  
  - 父类中定义的静态语句块优先于子类的变量赋值操作。
  
    ```java
    static class Parent { //static只能修饰内部类，使其不需要外部类对象就能被new
    	public static int A = 1;
    	static {
    		A = 2;
    	}
    }
    static class Sub extends Parent {
    	public static int B = A;
    }
    public static void main(String[] args) {
    	System.out.println(Sub.B);	//结果为2
    }
    ```
  
  - \<clinit>()方法并非必需，如果一个类没有静态语句块也没有对变量的赋值操作，编译器可以不生成它。
  
  - 接口中不能使用静态语句块，但仍然可以有变量初始化的赋值操作，因此接口也会生成\<clinit>()方法，与类的不同在于接口的\<clinit>()方法不需要先执行父接口的\<clinit>()方法，且接口的实现类在初始化时也不会执行接口的\<clinit>()方法。
  
  - Java虚拟机必须保证一个类的\<clinit>()方法在多线程环境中被正确地加锁同步，因此多个线程同时初始化一个类时，只会有一个线程去执行该类的\<clinit>()方法，其他线程都需要阻塞等待。

##### 7.4 类加载器

- 它是在Java虚拟机外部实现的，应用程序自己决定如何获取所需的类。实现“通过一个类的全限定名来获取描述该类的二进制字节流”动作的代码被称为“类加载器”。

- 类与类加载器：对于任意一个类，都必须由加载它的类加载器和这个类本身共同确立其在虚拟机中的唯一性。对于两个来源于同一个Class文件、由同一个虚拟机加载的类，只要加载它们的类加载器不同，这两个类就不相等，包括Class对象的equals()方法、isAssignableFrom()方法、isInstance()方法的返回结果，以及instanceof关键字做对象所属关系判定等各种情况。

  ```java
  package cn.dut.test;
  
  import java.io.IOException;
  import java.io.InputStream;
  
  public class ClassLoaderTest {
      public static void main(String[] args) throws Exception {
          ClassLoader myLoader = new ClassLoader() {
              @Override
              public Class<?> loadClass(String name) throws ClassNotFoundException {
                  try {
                      String fileName = name.substring(name.lastIndexOf('.') + 1) + ".class";
                      InputStream is = getClass().getResourceAsStream(fileName);
                      if (is == null) {
                          return super.loadClass(name);
                      }
                      byte[] b = new byte[is.available()];
                      is.read(b);
                      return defineClass(name, b, 0, b.length);
                  } catch (IOException e) {
                      throw new ClassNotFoundException(name);
                  }
              }
          };
  
          Object obj = myLoader.loadClass("cn.dut.test.ClassLoaderTest").newInstance();
          System.out.println(obj.getClass());	//class cn.dut.test.ClassLoaderTest
          System.out.println(obj instanceof cn.dut.test.ClassLoaderTest);	//false
      }
  }
  ```

  可以看到该对象确实为类cn.dut.test.ClassLoaderTest的实例，但instanceof的结果为false。这是因为虚拟机中同时存在两个ClassLoaderTest类，它们由两个类加载器加载。

- 双亲委派模型：

  - 从虚拟机的角度看只有两种不同的类加载器：启动类加载器，是虚拟机自身的一部分，C++所写；其他所有的类加载器，存在于虚拟机外部，全部继承自抽象类java.lang.ClassLoader。

  - 从Java的角度看，在JDK9之前，类加载架构为三层类加载器、双亲委派模型。

    - 启动类加载器：负责加载存放在\<JAVA_HOME>\lib目录、或者被-Xbootclasspath参数所指定的路径中存放的，而且是Java虚拟机能够识别的类库加载到虚拟机内存。无法被Java程序直接引用，编写自定义类加载器时，如果需要把加载请求委派给启动类加载器处理，直接使用null替代即可。

    	```java
      /** Returns the class loader for the class.  Some implementations may use
    	null to represent the bootstrap class loader. This method will return
    	null in such implementations if this class was loaded by the bootstrap
    	class loader. */
    		public ClassLoader getClassLoader() {
    	        ClassLoader cl = getClassLoader0();
    	        if (cl == null)
    	            return null;
    	        SecurityManager sm = System.getSecurityManager();
    	        if (sm != null) {
    	            ClassLoader.checkClassLoaderPermission(cl, Reflection.getCallerClass());
    	        }
    	        return cl;
    	    }      
    	```
    	
    - 扩展类加载器：负责加载存放在\<JAVA_HOME>\lib\ext目录、或者被java.ext.dirs系统变量所指定的路径中所有的类库。JDK9之后，这种扩展机制被模块化带来的天然的扩展能力所取代。
      
    - 应用程序类加载器（系统类加载器）：负责加载用户类路径上所有的类库。它一般是程序中默认的类加载器。

  - JDK9之前的应用都是由这三种类加载器互相配合来完成加载的，还可以加入自定义的类加载器进行拓展。它们的关系为
    
    <img src="img\java虚拟机\双亲委派模型.jpg" style="zoom: 50%;" />
    
    该层次关系被称为类加载器的双亲委派模型。除了顶层的启动类加载器，其他的类加载器都应该有自己的父类加载器，它们通常使用组合关系来复用父类加载器的代码。
    
    工作过程：如果一个类加载器收到了类加载的请求，它会先把该请求委派给父类加载器去完成，每一层的类加载器均如此，即所有的加载请求最终都会传送到启动类加载器，只有当父类加载器无法完成加载请求时，子类加载器才会尝试自己去完成加载。
    
    优点：Java中的类随它的类加载器一起具备了一种带有优先级的层次关系，对于保证Java程序的稳定运作极为重要。如java.lang.Object类无论哪一个类加载器都需要加载它，这样由最顶端的启动类加载器进行加载就能保证所有类加载器加载到的Object类都是同一个类。
    
      ```java
      protected Class<?> loadClass(String name, boolean resolve)
          throws ClassNotFoundException
      {
          synchronized (getClassLoadingLock(name)) {
              // First, check if the class has already been loaded
              Class<?> c = findLoadedClass(name);//首先检查请求加载的类型是否已经被加载过
              if (c == null) {
                  long t0 = System.nanoTime();
                  try {
                      if (parent != null) {//父加载器不为空，则调用父加载器的加载方法
                          c = parent.loadClass(name, false);
                      } else {//最终到达启动类加载器进行加载
                          c = findBootstrapClassOrNull(name);
                      }
                  } catch (ClassNotFoundException e) {
                      // ClassNotFoundException thrown if class not found
                      // from the non-null parent class loader
                  }
      		//父加载器抛出异常，则由当前加载器尝试加载，只要当前加载器没找到就逐层由子加载器加载
                  if (c == null) {
                      // If still not found, then invoke findClass in order
                      // to find the class.
                      long t1 = System.nanoTime();
                      c = findClass(name);
      
                      // this is the defining class loader; record the stats
                      PerfCounter.getParentDelegationTime().addTime(t1 - t0);
                      PerfCounter.getFindClassTime().addElapsedTimeFrom(t1);
                      PerfCounter.getFindClasses().increment();
                  }
              }
              if (resolve) {
                  resolveClass(c);
              }
              return c;
          }
      }
      ```

- 破坏双亲委派模型
  - 基础的类中需要调用回用户的代码，如JNDI服务需要调用由其他厂商实现并部署在应用程序的类路径下的JNDI服务提供者接口（service provider interface）的代码，这相当于需要启动类加载器加载应用程序的类。解决办法是Java中设计了**线程上下文类加载器**，它默认是应用程序类加载器。JDNI服务通过它来加载SPI代码，这相当于父类加载器请求子类加载器完成加载的行为。JDK6时的**java.util.ServiceLoader**是一种更好的解决方案。
  - OSGi通过类加载器实现热部署：每一个程序模块都有一个自己的类加载器，当需要更换一个模块时，就把模块连同它的来加载其一起换掉以实现代码的热替换。在OSGi环境下类加载器不再是双亲委派模型的树形结构，而是更复杂的网状结构。

##### 7.5 Java模块化系统

- JDK9引入模块化系统，为了实现可配置的封装隔离机制，类加载架构也作出了相应的变动调整。
  - 解决JDK9之前基于类路径来查找依赖的可靠性问题。
  - 解决JDK9之前类路径上跨JAR文件的public类型的可访问性问题。
- 模块化系统的兼容性：
  - 通过3条规则保证使用传统类路径依赖的Java程序升级到JDK9以后依然可用。
  - 模块化系统目前不支持在模块定义中加入版本号来管理和约束依赖，只能在编译打包时人工选择好版本。

- 模块化下的类加载器：对比传统三层类加载器和双亲委派架构的变动：

  - 扩展类加载器被平台类加载器取代。基于模块化构建，模块中的Java类库自然满足可扩展的需求。
  - 平台类加载器和应用程序类加载器都不再派生自java.net.URLClassLoader，现在启动类加载器、平台类加载器、应用程序类加载器全都继承于jdk.internal.loader.BuiltinClassLoader。

  <img src="img\java虚拟机\类加载器架构1.jpg" style="zoom:50%;" />

  <img src="img\java虚拟机\类加载器架构2.jpg" style="zoom:50%;" />
  - 委派关系发生变动，当平台和应用程序类加载器收到类加载请求，在委派给父加载器前，先判断该类是否能够归属到某一个系统模块中，如果可以则优先委派给负责那个模块的加载器完成加载。

    <img src="img\java虚拟机\JDK9之后的委派模式.jpg" style="zoom: 50%;" />

    三个类加载器负责各自加载的模块部分。

#### 第八章 虚拟机字节码执行引擎

##### 8.1 概述

- 物理机的执行引擎是直接建立在处理器、缓存、指令集和操作系统层面上的，而虚拟机的执行引擎是由软件自行实现，因此可以不受物理条件制约地定制指令集与执行引擎的结构体系，执行不被硬件直接支持的指令集。

- 执行方式通常有解释执行和编译执行两种。

##### 8.2 运行时栈帧结构

- 栈帧是运行时数据区中虚拟机栈的栈元素，用于支持虚拟机的方法调用和方法执行，一个栈桢对应一个方法调用。每一个栈帧都包括局部变量表、操作数栈、动态连接、方法返回地址和一些额外的附加信息。

- 在活动线程中只有栈顶的方法（当前方法）才是在运行的，栈顶的栈帧（当前栈帧）才是生效的。

- 概念结构为

  <img src="img\java虚拟机\栈帧的概念模型.jpg" style="zoom: 67%;" />

  - 局部变量表：是一组变量值的存储空间，用于存放方法参数和方法内部定义的局部变量。

    - Code属性的max_locals数据项确定了该方法所需分配的局部变量表最大容量。
    - 容量以变量槽为最小单位，每个变量槽都应当能存放一个boolean、byte、char、short、int、float、reference或returnAddress类型的数据，但它的具体长度可以随着处理器、操作系统或虚拟机实现的不同而发生变化。因此一个变量槽可以存放一个32位以内的数据，对于long、double两种64位的数据类型，虚拟机会以高位对齐的方式为其分配两个连续的变量槽。
    - 虚拟机通过索引定位的方式使用局部变量表，索引值的范围从0开始至最大变量槽数量。

    - 执行实例方法时，索引为0的变量槽默认存放方法所属对象实例的引用，即this关键字。其余参数按照参数列表顺序排列，占用从1开始的局部变量槽。参数列表分配完毕后，再根据方法体内部定义的变量顺序和作用域分配其余的变量槽。

    - 依据局部变量的作用域，变量槽可以复用。这可以节省栈帧消耗的内存空间，但它对垃圾收集会产生一定影响。

      ```java
      package cn.dut.test;
      
      public class SlotGC {
          public static void main(String[] args) {
              byte[] placeholder = new byte[64*1024*1024];
              System.gc();
          }
      }
      
      [0.019s][info][gc] Using G1
      [0.141s][info][gc] GC(0) Pause Full (System.gc()) 67M->65M(227M) 2.678ms
      ```

      ```java
      package cn.dut.test;
      
      public class SlotGC {
          public static void main(String[] args) {
              {
                  byte[] placeholder = new byte[64 * 1024 * 1024];
              }
              System.gc();
          }
      }
      [0.017s][info][gc] Using G1
      [0.145s][info][gc] GC(0) Pause Full (System.gc()) 67M->65M(227M) 2.748ms
      ```

      ```java
      package cn.dut.test;
      
      public class SlotGC {
          public static void main(String[] args) {
              {
                  byte[] placeholder = new byte[64 * 1024 * 1024];
              }
              int a = 0;
              System.gc();
          }
      }
      [0.018s][info][gc] Using G1
      [0.148s][info][gc] GC(0) Pause Full (System.gc()) 67M->0M(10M) 8.591ms
      ```

      第一次因为placeholder还在作用域内，无法回收；但第二次placeholder出了作用域进行回收，发现没有回收成功；第三次定义了一个新的局部变量，回收就成功了。这是因为第二段代码中placeholder占用的变量槽还没有复用，依然存放着对数组对象的引用，则作为GC Roots一部分的局部变量表仍然保持着对数组对象的关联。但注意：这是解释执行才会有的现象，第二段代码经过即时编译可以正常回收。

    - 类变量和实例变量会分别在准备阶段和对象创建阶段初始化为零值，因此不赋值也可以使用，但局部变量必须显式赋值进行初始化。

  - 操作数栈：是一个后入先出栈。
  
    - Code属性的max_stacks数据项确定了该方法所需分配的操作数栈的最大深度。
  
    - 每个元素都可以是包括long和double在内的任意数据类型，32位数据类型所占的栈容量为1，64位数据类型所占的栈容量为2。Q：占2是什么意思，还是分开存储的？
  
    - 在概念模型中，两个不同栈帧作为不同方法的虚拟机栈元素，是完全独立的。但实际上两个栈桢会出现部分重叠，即下面栈帧的部分操作数栈与上面栈帧的部分局部变量表重叠。
  
      <img src="img\java虚拟机\栈帧重叠部分.jpg" style="zoom: 33%;" />
  
  - 动态连接：每个栈帧都包含一个指向运行时常量池中该栈帧所属方法的引用，持有这个引用是为了支持方法调用过程中的动态连接，即运行期间将Class文件中的符号引用转化为直接引用。Q：动态连接和这个引用有什么关系？
  
  - 方法返回地址：当一个方法执行后，只有两种方式退出这个方法，即正常调用完成和异常调用完成（遇到异常但没有妥善处理的代码逻辑）。在方法退出后，必须返回到最初方法被调用的位置程序才能继续执行，因此方法正常退出时，栈帧中可能会保存主调方法的PC计数器值作为返回地址，而异常退出时，返回地址通过异常处理器来确定，栈帧中不会保存返回地址的信息。
  
    方法退出时可能的操作：恢复上层方法的局部变量表和操作数栈，把返回值压入调用者栈帧的操作数栈，调整PC计数器的值以指向方法调用指令后面一条指令等。
  
  - 附加信息：与调试、性能收集相关的信息，完全取决于具体的虚拟机实现。

##### 8.3 方法调用

- 方法调用阶段是确定被调用方法的版本，未涉及方法内部的具体运行过程。
- 解析调用：方法在程序真正运行之前就有一个可确定的调用版本，并且这个方法的调用版本的在运行期不可改变，这类方法的调用称为解析，即类加载的阶段符号引用被转化为直接引用。
  - 字节码指令：
    - invokestatic，用于调用静态方法。
    - invokespecial，用于调用实例构造器\<init>()方法、私有方法和父类中的方法。
    - invokevirtual，用于调用所有的虚方法。
    - invokeinterface，用于调用接口方法，会在运行时再确定一个实现该接口的对象。
    - invokedynamic，先在运行时动态解析出调用点限定符所引用的方法，然后再执行该方法。
  - 只要能被invokestatic和invokespecial指令调用的方法，都可以在解析阶段确定唯一的调用方法，共有静态方法、私有方法、实例构造器、父类方法4种，加上特例被final修饰的方法（虽然使用invokevirtual指令调用），这些方法统称为“非虚方法”。

- 分派调用：分为静态单分派、静态多分派、动态单分派、动态多分派，它与多态的实现如重载（多个同名方法）、重写（覆盖父类方法）密切相关。

  - 静态分派（重载）：

    - 静态类型和实际类型：	

      ```java
      package cn.dut.test;
      
      public class StaticDispatch {
          
          static abstract class Human{}
          static class Man extends Human{}
          static class Woman extends Human{}
          
          public void sayHello(Human guy){
              System.out.println("hello, guy!");
          }
          public void sayHello(Man guy){
              System.out.println("hello, gentleman!");
          }
          public void sayHello(Woman guy){
              System.out.println("hello, lady!");
          }
      
          public static void main(String[] args) {
              Human man = new Man();
              //Human称为静态类型/外观类型，Man称为实际类型/运行时类型
              Human woman = new Woman();
              StaticDispatch sr = new StaticDispatch();
              sr.sayHello(man);//hello, guy!
              sr.sayHello(woman);//hello, guy!
          }
      }
      ```

      静态类型和实际类型都可能发生变化，但静态类型的变化在使用时发生，编译时一定可知，而实际类型的变化在运行期才可确定。

        ```java
  //实际类型变化
        Human human = (new Random()).nextBoolean()?new Man():new Woman();
        //静态类型变化        
        sr.sayHello((Man) human);
        sr.sayHello((Woman) human);
        ```
    
    - 重载时是通过参数的静态类型而不是实际类型作为判断依据，这种依赖静态类型来决定方法执行版本的分派动作，称为静态分派。它发生在编译阶段，实际上并非由虚拟机执行。
    
    - 重载的优先级：char->int->long->float->double->Character->Serializable->Object->char ...
    
      基本类型->装箱类型->装箱类型的实现的接口->父类->变长参数。
    
    - 重载与解析并不排他，多个重载的静态方法也是通过静态分派来完成的。
    
  - 动态分派（重写）：

    - invokevirtual指令：

        ```java
        package cn.dut.test;
        
        public class DynamicDispatch {
        
            static abstract class Human{
                protected abstract void sayHello();
            }
            static class Man extends Human{
                @Override
                protected void sayHello() {
                    System.out.println("man say hello");
                }
            }
            static class Woman extends Human{
                @Override
                protected void sayHello() {
                    System.out.println("woman say hello");
                }
            }
        
            public static void main(String[] args) {
                Human man = new Man();
                Human woman = new Woman();
                man.sayHello();//man say hello
                woman.sayHello();//woman say hello
                man = new Woman();
                man.sayHello();//woman say hello
            }
        }
        ```

        ```
        Code:
              stack=2, locals=3, args_size=1
             0: new           #2                  // class cn/dut/test/DynamicDispatch$Man
                 3: dup
                 4: invokespecial #3                  // Method cn/dut/test/DynamicDispatch$Man."<init>":()V
                 7: astore_1
                 8: new           #4                  // class cn/dut/test/DynamicDispatch$Woman
                11: dup
                12: invokespecial #5                  // Method cn/dut/test/DynamicDispatch$Woman."<init>":()V
                15: astore_2
                16: aload_1
                17: invokevirtual #6                  // Method cn/dut/test/DynamicDispatch$Human.sayHello:()V
                20: aload_2
                21: invokevirtual #6                  // Method cn/dut/test/DynamicDispatch$Human.sayHello:()V
                24: new           #4                  // class cn/dut/test/DynamicDispatch$Woman
                27: dup
                28: invokespecial #5                  // Method cn/dut/test/DynamicDispatch$Woman."<init>":()V
                31: astore_1
                32: aload_1
                33: invokevirtual #6                  // Method cn/dut/test/DynamicDispatch$Human.sayHello:()V
                36: return
        ```

        该段代码的字节码指令显示，man和woman两个对象执行的指令和参数都相同，因此关键在于invokevirtual指令，它的运行时解析过程如下：

        1. 找到操作数栈顶的第一个元素所指向的对象的实际类型，记为C。

        2. 如果在类型C中找到与常量中的描述符和简单名称都相符的方法，则进行访问权限校验，如果通过则返回这个方法的直接引用，查找结束；不通过则抛出异常。

        3. 否则，按照继承关系从下往上依次对C的各个父类进行第二步的搜索和验证过程。

        4. 如果始终没找到合适的方法则抛出异常。

        可以看到invokevirtual指令并非直接将常量池中的符号引用DynamicDispatch$Human.sayHello:()V解析为直接引用，而是找到执行方法对象的实际类型，选择方法版本，这个过程就是重写的本质。

        - 另外该指令是用于确定方法执行版本，因此Java中的字段不会参与多态。

          ```java
          package cn.dut.test;
          
          public class FieldHasNoPolymorphic {
          
              static class Father {
                  public int money = 1;
                  public Father(){
                      money=2;
                      showMeTheMoney();
                  }
                  public void showMeTheMoney() {
                      System.out.println("I am Father, i hava $" + money);
                  }
              }
          
              static class Son extends Father {
                  public int money = 3;
                  public Son(){
                      money=4;
                      showMeTheMoney();
                  }
                  public void showMeTheMoney() {
                      System.out.println("I am Son, i hava $" + money);
                  }
              }
          
              public static void main(String[] args) {
                  Father guy = new Son();
                  System.out.println("This guy has $" + guy.money);
              }
          }
          //I am Son, i hava $0
          //I am Son, i hava $4
          //This guy has $2
          ```

          Son在创建时隐式调用Father的构造函数，而Father构造函数中的showMeTheMoney()方法是public权限的实例方法，因此实际执行的版本是Son中的showMeTheMoney()方法，此时Son的money还没有初始化，因此输出I am Son, i hava $0。而最后一句通过Father静态类型访问父类中的money。

          这里将实例方法的权限改为private，则会输出

          ```
          I am Father, i hava $2
          I am Son, i hava $4
          This guy has $2
          ```

    - 重写时是通过参数的实际类型而不是静态类型作为判断依据，这种在运行期依赖实际类型来决定方法执行版本的分派动作，称为动态分派。这是虚拟机完成的。

  - 单分派与多分派：方法的接收者与方法的参数统称为方法的宗量，根据分派基于多少种宗量，分派划分为单分派和多分派，即根据一个还是多个宗量对目标方法进行选择。

    Java中的静态分派属于多分派类型，即通过静态类型以及方法参数两个宗量生成方法执行指令（invokevirtual）的参数（但运行时会动态分派为实际类型的方法）。

    Java中的动态分派为单分派类型，即运行时只会寻找是否有对应的实际类型的方法，而不管传递过来的方法参数。 

  - 虚拟机动态分派的实现：基于执行性能的考虑，实际中的一种优化手段是为类型在方法区中建立一个虚方法表，使用虚方法表索引来代替元数据查找以提高性能。但它只在解释执行时使用，仍然是最慢的一种分派，即时编译时有更多的性能优化技术。

    <img src="img\java虚拟机\方法表结构.jpg" style="zoom:50%;" />

    - 虚方法表中存放着各个方法的实际入口地址。如果某个方法在子类中没有被重写，那该方法在虚方法表中的地址入口与父类中的地址入口一致，都指向父类的实现入口，如Object类型的方法；如果子类中重写了该方法，则它在子类虚方法表的地址入口会被替换为子类的实现入口。

    - 虚方法表一般在类加载的连接阶段（准备阶段）进行初始化。

##### 8.4 动态类型语言支持

- 动态类型语言：类型检查的主体过程在运行期而不是编译期；变量无类型，而变量的值才有类型。

- java.lang.invoke包：目的是除了单纯依靠符号引用来确定调用的目标方法以外，提供一种新的动态确定目标方法的机制，称为方法句柄。

  ```java
  package cn.dut.test;
  
  import java.lang.invoke.MethodHandle;
  import java.lang.invoke.MethodHandles;
  import java.lang.invoke.MethodType;
  
  public class MethodHandleTest {
      static class ClassA {
          public void println(String s) {
              System.out.println(s);
          }
      }
  
      public static void main(String[] args) throws Throwable {
          Object obj = System.currentTimeMillis() % 2 == 0 ? System.out : new ClassA();
          getPrintlnMH(obj).invokeExact("hello methodhandle");
          obj.getClass().getMethod("println", String.class).invoke(obj,"hello reflection");
      }
  
      private static MethodHandle getPrintlnMH(Object receiver) throws Throwable {
          //方法类型，包括返回值类型和参数类型
          MethodType mt = MethodType.methodType(void.class, String.class);
          //lookup，在指定类中查找符合给定的方法名称、方法类型，并且符合调用权限的方法句柄。
          return MethodHandles.lookup().findVirtual(receiver.getClass(), "println", mt).bindTo(receiver);
          //bindto传递该方法的接收者，即this指向的对象。
      }
  }
  ```

  getPrintMH()方法模拟了invokevirtual指令的执行过程，但这个分派逻辑是很动态的，由用户程序所实现。它的返回值可以视为对最终调用方法的一个“引用”。

  - 与反射机制的区别：
    - 反射是在模拟Java代码层次的方法调用，而方法句柄是在模拟字节码层次的方法调用。
    - 反射中的java.lang.reflect.Method对象远比方法句柄中的java.lang.invoke.MethodHandle对象包含的信息多。
    - 反射调用方法几乎不可能直接实施各类调用点优化措施，而方法句柄是字节码层次的指令模拟，因此虚拟机的优化措施也可以用在方法句柄上。
    - 反射专为Java服务，而方法句柄可以为所有运行于Java虚拟机上的语言服务。

- invokedynamic指令：每一处含有invokedynamic指令的位置都被称作“动态调用点”，它的第一个参数变为CONSTANT_InvokeDynaimc_info常量，它存放着引导方法、方法类型和名称三项信息。虚拟机通过执行引导方法，获得一个java.lang.invoke.CallSite对象，最终调用要执行的目标方法。

  ```java
  package cn.dut.test;
  
  import java.lang.invoke.*;
  
  public class InvokeDynamicTest {
      public static void main(String[] args) throws Throwable {
          INDY_BootstrapMethod().invokeExact("hello");
      }
  
      public static void testMethod(String s) {
          System.out.println("heelo String: " + s);
      }
  
      public static CallSite BootstrapMethod(MethodHandles.Lookup lookup, String name, MethodType mt) throws Throwable {
          return new ConstantCallSite(lookup.findStatic(InvokeDynamicTest.class, name, mt));
      }
  
      public static MethodType MT_BootstrapMethod() {
          return MethodType.fromMethodDescriptorString("(Ljava/lang/invoke/MethodHandles$Lookup;Ljava/lang/String;Ljava/lang/invoke/MethodType;)Ljava/lang/invoke/CallSite;", null);
      }
  
      private static MethodHandle MH_BootstrapMethod() throws Throwable {
          return MethodHandles.lookup().findStatic(InvokeDynamicTest.class, "BootstrapMethod", MT_BootstrapMethod());
      }
  
      private static MethodHandle INDY_BootstrapMethod() throws Throwable {
          CallSite cs = (CallSite) MH_BootstrapMethod().invokeWithArguments(MethodHandles.lookup(), "testMethod", MethodType.fromMethodDescriptorString("(Ljava/lang/String;)V", null));
          return cs.dynamicInvoker();
      }
  }
  ```

  这段代码纯粹是为了让字节码生成invokedynamic指令，因为它主要是为其他动态语言服务的，在JDK8引入lambda表达式和接口默认方法后，Java的字节码才有了invokedynamic指令。

##### 8.5 基于栈的字节码解释执行引擎

- 解释执行：两种执行过程

  <img src="img\java虚拟机\编译过程.jpg" style="zoom: 50%;" />

  Java的编译过程到指令流，解释器在虚拟机内部，因此Java的编译是半独立的。

  解释执行是运行期将字节码指令逐条翻译为机器码指令。

- 基于栈的指令集与基于寄存器的指令集
  - 字节码指令流基本上是一种基于栈的指令集架构，大部分指令均为零地址指令，依赖操作数栈中的数据工作。如

    ```
    iconst_1 //将常量1放入操作数栈
    iconst_1
    iadd //取出操作数栈的两个1进行相加
    istore_0 //将运算结果存入局部变量表0号槽
    ```

    优点：可移植，代码相对更加紧凑（每个字节就是一条指令，而多地址指令还需要存放参数），编译器实现更加简单（不需要考虑空间分配的问题，所需空间都在栈上，使用寄存器得考虑移动数据等操作）。

    缺点：如果采用解释执行，执行速度相对慢一些，因为出栈、入栈操作较多，而且栈实现在内存中，频繁的内存访问一定慢于直接操作寄存器。

  - 基于寄存器的指令集为物理硬件直接支持的指令集架构，如x86的二地址指令，依赖寄存器进行工作。如

    ```
    mov eax, 1
    add eax, 1
    ```

- 基于栈的解释器执行过程

  ```java
  public int calc() {
          int a = 100;
          int b = 200;
          int c = 300;
          return (a + b) * c;
      }
      Code:
        stack=2, locals=4, args_size=1
           0: bipush        100 //byte int push
           2: istore_1
           3: sipush        200 //short int push
           6: istore_2
           7: sipush        300
          10: istore_3
          11: iload_1
          12: iload_2
          13: iadd
          14: iload_3
          15: imul
          16: ireturn
  ```

  概念模型：

  <img src="img\java虚拟机\解释执行1.jpg" style="zoom: 32%;" /><img src="img\java虚拟机\解释执行2.jpg" style="zoom:32%;" /><img src="img\java虚拟机\解释执行3.jpg" style="zoom:32%;" />

  <img src="img\java虚拟机\解释执行4.jpg" style="zoom:32%;" /><img src="img\java虚拟机\解释执行5.jpg" style="zoom:32%;" /><img src="img\java虚拟机\解释执行6.jpg" style="zoom:31.5%;" /><img src="img\java虚拟机\解释执行7.jpg" style="zoom:32%;" />

  - push指令将一个常量放入操作数栈顶。
  - store指令将操作数栈顶元素出栈并放入对应的局部变量槽中。
  - load指令将对应的局部变量槽中的元素复制一份到操作数栈顶。
  - add指令将操作数栈顶两个元素取出并相加，然后将结果重新入栈。
  - mul指令将操作数栈顶两个元素取出并相乘，然后将结果重新入栈。
  - return将操作数栈顶元素返回。

#### 第九章 类加载及执行子系统的案例与实战

##### 9.2 案例

- Tomcat：正统的类加载器架构。

  - Tomcat6以前，目录结构中可以设置3组目录，/common/\*、/server/\*、/shared/\*，加上Web应用程序自身的/WEB-INF/\*目录，共4组目录存放Java类库，它们的含义是：

    - /common：类库可被Tomcat和所有的Web应用共同使用。
    - /server：类库可被Tomcat使用，对所有的Web应用都不可见。
    - /shared：类库可被所有的Web应用共同使用，对Tomcat不可见。
    - /WebApp/WEB-INF：类库仅被该Web应用使用，对Tomcat和其他Web应用都不可见。

  - 类加载机制：

    <img src="img\java虚拟机\tomcat类加载机制.png" style="zoom:50%;" />

    - 上面三个为JDK（9之前）默认的类加载器，下面四个分别对应四个目录的类加载器，每一个WebApp对应一个WebApp类加载器，每一个JSP文件对应一个Jasper类加载器。

    - Common类加载器能加载的类都可以被Catalina类加载器和Shared类加载器使用，而Catalina类加载器和Shared类加载器自己能加载的类则与对方相互隔离。

    - WebApp类加载器可以使用Shared类加载器加载到的类，但各个WebApp类加载器实例之间相互隔离。

    - JasperLoader的加载范围仅仅是该JSP文件编译出的Class文件，当服务器检测到JSP文件被修改时，就替换掉当前的JasperLoader实例，再建立一个新的JSP类加载器来实现JSP文件的HotSwap功能。
    - 注意委派的内涵：优先由父加载器进行加载更基础的类，父加载器找不到时还是会由子加载器进行加载的，例如WebApp中的类库加载，虽然一层一层委派给父加载器，但是父加载器找不到最后还是会由当前的WebApp类加载器加载，这样才能起到隔离作用。

  - Tomcat6及之后简化了默认的目录结构，将/common、/server、/shared合并成一个/lib目录，相当于/common目录中类库的作用，并且不会建立Catalina类加载器和Shared类加载器的实例。可以通过修改配置文件指定server.loader和share.loader重新启用原来完整的加载器架构。

    <img src="img\java虚拟机\tomcat配置文件内容.jpg" style="zoom:50%;" />

- OSGi：灵活的类加载架构。
  - OSGi（Open Service Gateway Initative）是一个基于Java语言的动态模块化规范。它的模块依赖关系、类库的可见性控制精确，并且基于它可能实现模块级的热插拔功能。
  - 类加载机制：模块的类加载器之间只有规则，没有固定的委派关系。
    - 某个Package被其他模块依赖时，发布它的模块类加载器负责完成类加载动作。
    - 一个模块的类加载器会根据Export-Package列表严格控制访问范围，某个Package没有被Export，那就不会由该模块的类加载器来加载这个Package。

### 第四部分 程序编译与代码优化

#### 第十章 前端编译与优化

##### 10.1概述

- 前端编译：把java文件转换为class文件。
- 后端编译：生成本地机器码。
  - 即时编译：运行期（以方法为单位）将字节码指令直接翻译为机器码指令后执行。
  - 提前编译：直接把java文件翻译为机器码指令。

##### 10.2 Javac编译器

- 编译过程分为一个准备过程和三个处理过程：

  1. 准备过程：初始化插入式注解处理器。
  2. 解析与填充符号表过程，包括
     - 词法、语法分析。将源代码的字符流转变为标记集合，构造出抽象语法树。
     - 填充符号表。产生符号地址和符号信息。
  3. 插入式注解处理器的注解处理过程。
     - 读取、修改、添加抽象语法树中的任意元素。
  4. 语义分析与字节码生成过程，包括：
     - 标注检查。对语法的静态信息进行检查，如变量使用前是否已被声明、变量与赋值的数据类型是否能够匹配等等。
     - 数据流及控制流分析。对程序动态进行过程进行检查，如局部变量在使用前是否已被赋值、方法的每条路径是否都有返回值、是否所有的受查异常都被正确处理等等。
     - 解语法糖。将简化代码编写的语法糖还原成原有的形式，如泛型、变长参数、自动装箱拆箱等等。
     - 字节码生成。将前面各个步骤所生成的信息转化成字节码。在字节码生成之前会将实例构造器\<init>()方法和类构造器方法\<clinit>()方法添加到语法树上，并优化一些代码逻辑，如将字符串的加操作替换为StringBuilder的append操作等等。

  ![](img\java虚拟机\javac的编译过程.jpg)

##### 10.3 语法糖

- 泛型：将操作的数据类型指定为方法签名中的一种特殊参数，这种参数类型用在类、接口和方法的创建中，分别构成泛型类、泛型接口和泛型方法。
  - Java的泛型实现方式为“类型擦除式泛型”，它只在程序源码中存在，编译成字节码后，全部泛型类型都被替换为原来的裸类型，并在相应位置插入了强制转型代码。因此对于运行期的Java语言，ArrayList\<Integer>和ArrayList\<String>其实是同一个类型（裸类型）。

     - Java选择将已有类型泛型化，因此出现了裸类型的概念：所有该类型泛型化实例的共同父类型，如ArrayList为ArrayList\<Integer>、ArrayList\<String>等等的裸类型，为了保证以前使用ArrayList的代码在泛型新版本中还能继续使用。

     - Java选择使用类型擦除来实现裸类型：一种选择是由Java虚拟机来自动地、真实地构造出ArrayList\<Integer>这样的类型，并且自动实现从ArrayList\<Integer>派生自ArrayList的继承关系来满足裸类型的定义。另一种是直接在编译时把ArrayList\<Integer>还原回ArrayList，只在元素访问、修改时自动插入一些强制类型转换和检查指令（对方法中Code属性中的字节码进行擦除，使用泛型定义的字段等只是一个静态类型的修改，另外Signature属性在元数据层面确实保留了泛型信息）。

       ```java
       public static void main(String[] args) {//泛型擦除前
       	Map<String, String> map = new HashMap<>();
       	map.put("hello", "你好");
       	map.put("how are you?", "吃了没？");
       	System.out.println(map.get("hello"));
       	System.out.println(map.get("how are you?"));
       }
       public static void main(String[] args) {//编译成字节码再反编译，泛型被擦除
       	Map map = new HashMap();
       	map.put("hello", "你好");
       	map.put("how are you?", "吃了没？");
       	System.out.println((String) map.get("hello"));
       	System.out.println((String) map.get("how are you?"));
       }
       ```

     - 类型擦除的缺陷

        - 不支持原始数据类型的泛型，因为不支持int、long与Object之间的强制转换。因此只能使用ArrayList\<Integer>这样的定义，填入、取出数据时做自动装箱、拆箱，因此慢。

        - 运行期无法取到泛型类型信息，会让一些代码变得啰嗦。

          ```java
          public class TypeErasureGenerics<E> {
              public void doSomething(Object item) {
                  if (item instanceof E) {//不合法，无法对泛型进行实例判断
                      ...
                  }
                  E newItem = new E();//不合法，无法使用泛型创建对象
                  E[] itemArray = new E[10];//不合法，无法使用泛型创建数组
              }
          }
          ```

          Q：如何解释？E是一个类型（Class），它的信息在编译成字节码后并没存储，那就无法做以上几件事，但是强制转型的代码不也需要有这个类型信息吗？如果没有E的类型信息，强制转型又是怎么做到的呢？可能是因为强制类型转换只是修改一下变量的静态类型描述，并不需要真正的类型信息？

          Q：Signature属性似乎记录了泛型信息？那为什么还是无法做到呢？
        
          ```java
          //泛型List转换为数组，由于不能从List<T>中得到参数化类型T，只能额外传入数组的组件类型。
          public static <T> T[] convert(List<T> list, Class<T> componentType) {
              //运行期通过反射创建实例
              T[] array = (T[]) Array.newInstance(componentType, list.size());
              ...
     }
          ```
     
        - 带来了一些模糊性。
        
          ```java
          //这段代码无法通过编译，类型擦除后这两个方法特征签名完全一样
          public class GenericTypes {
          	public static void method(List<String> list) {
          		System.out.println("invoke method(List<String> list)");
          	}
          	public static void method(List<Integer> list) {
          		System.out.println("invoke method(List<Integer> list)");
          	}
          }
          ```
        
          ```java
          //这段代码在作者的运行的JDK6版本下javac可以通过编译，原因是Class文件层面只要描述符不完全一致的两个方法就可以共存，实际上并不是由于返回值参与了重载过程，只是javac编译器的锅。JDK11下javac也无法通过编译了。
          public class GenericTypes {
          	public static String method(List<String> list) {
          		System.out.println("invoke method(List<String> list)");
                  return "";
          	}
          	public static int method(List<Integer> list) {
          		System.out.println("invoke method(List<Integer> list)");
                  return 1;
          	}
              public static void main(String[] args) {
                  method(new ArrayList<String>());
                  method(new ArrayList<Integer>());
              }
          }
          ```
        
          
  
  - C#的泛型实现方式为“具现化式泛型”，它在源码、中间码、运行期都是存在的，List\<int>和List\<string>就是两个不同的类型，它们由系统在运行期生成，有着自己独立的虚方法表和类型数据。

- 自动装箱、拆箱与遍历循环

  ```java
  	public static void main(String[] args) {//编译前
          List<Integer> list = Arrays.asList(1, 2, 3, 4);
          int sum = 0;
          for (int i : list) {
              sum += i;
          }
          System.out.println(sum);
      }
  	public static void main(String[] args) {//编译后
          List list = Arrays.asList(new Integer[]{Integer.valueOf(1),
                                                  Integer.valueOf(2),
                                                  Integer.valueOf(3),
                                                  Integer.valueOf(4)});
          int sum = 0;
          for (Iterator localIterator = list.iterator(); localIterator.hasNext()) {
              int i = ((Integer)localIterator.next()).intValue();
              sum += i;
          }
          System.out.println(sum);
      }
  	public static void main(String[] var0) {//本地的反编译结果
          List var1 = Arrays.asList(1, 2, 3, 4);
          int var2 = 0;
  
          int var4;
          for(Iterator var3 = var1.iterator(); var3.hasNext(); var2 += var4) {
              var4 = (Integer)var3.next();
          }
  
          System.out.println(var2);
      }
  ```

  自动装箱的陷阱：包装类的“==”运算在不遇到算术运算的情况下不会自动拆箱；equals()方法不处理数据转型。

  ```java
      public static void main(String[] args) {//JVM会自动维护八种基本类型的常量池，int常量池中初始化-128~127的范围，所以当为Integer i=127时，在自动装箱过程中是取自常量池中的数值，而当Integer i=128时，128不在常量池范围内，所以在自动装箱过程中需new 128，所以地址不一样。
          Integer a = 1;
          Integer b = 2;
          Integer c = 3;
          Integer d = 3;
          Integer e = 321;
          Integer f = 321;
          Long g = 3L;
          System.out.println(c == d);//true 取常量池中同一个数
          System.out.println(e == f);//false 各自创建新的对象，因此地址不一样
          System.out.println(c == (a + b));//true 遇到算术运算，自动拆箱成基本类型比较
          System.out.println(c.equals(a + b));//true 先判断类型是否满足，再比较值
          System.out.println(g == (a + b));//true 遇到算术运算，自动拆箱成基本类型，并向上转型为long进行比较
          System.out.println(g.equals(a + b));//false 类型不满足，直接返回false
      }
  ```

- 条件编译：使用条件为常量的if语句实现语句基本块级别的条件编译。

  ```java
  	public static void main(String[] args) {//编译前
          if (true) {
              System.out.println("block1");
          } else {
              System.out.println("block2");
          }
      }
  	public static void main(String[] args) {//反编译结果
  		System.out.println("block1");
      }
  ```

- 其他语法糖：内部类、枚举类、断言语句、数值字面量、对枚举和字符串的switch支持、try语句中定义和关闭资源、lambda表达式等等。

#### 第十一章 后端编译与优化

##### 11.2 即时编译器

- Java程序最初都是通过解释器进行解释执行的，当某个方法或代码块运行特别频繁，它们就会被认定为热点代码，接下来虚拟机将会把这些代码编译成本地机器码，并以各种手段尽可能地进行代码优化，完成这个任务的后端编译器称为即时编译器。

- 解释器与编译器

  - 解释器与编译器相辅相成配合工作，交互关系为

    <img src="img\java虚拟机\解释器与编译器的交互.jpg" style="zoom: 33%;" />

    HotSpot虚拟机中内置了三个即时编译器，其中两个存在已久，分别为“客户端编译器”和“服务端编译器”，或简称C1编译器和C2编译器。第三个是JDK10出现的Graal编译器。

  - 为了在程序启动响应速度和运行效率之间达到最佳平衡，HotSpot虚拟机在编译子系统中加入了分层编译的功能，其中包括

    - 第0层。程序纯解释执行，并且解释器不开启性能监控功能。
    - 第1层。使用客户端编译器将字节码编译为本地代码运行，进行简单可靠的稳定优化，不开启性能监控功能。
    - 第2层。仍然使用客户端编译器执行，仅开启方法及回边次数统计等有限的性能监控功能。
    - 第3层。仍然使用客户端编译器执行，开启全部性能监控，除了第2层的统计信息外，还会收集如分支跳转、虚方法调用版本等全部的统计信息。
    - 第4层。使用服务端编译器将字节码编译为本地代码，相比客户端编译器，服务端编译器会启用更多编译耗时更长的优化，还会根据性能监控信息进行一些不可靠的激进优化。

    各层次编译之间的交互、转换关系为

    <img src="img\java虚拟机\分层编译的交互关系.jpg" style="zoom: 33%;" />

    使用分层编译后，解释器、C1、C2会同时工作。

- 编译对象与触发条件

  - 热点代码主要有两类，包括被多次调用的方法以及被多次执行的循环体。对于这两种情况，编译的目标对象都是整个方法体。

  - 目前主流的热点探测判定方式有两种：

    - 基于采样的热点探测。周期性地检查各个线程的调用栈顶，如果某个方法经常出现在栈顶，那这个方法就是热点方法。
    - 基于计数器的热点探测。为每个方法建立计数器，统计方法的执行次数，如果执行次数超过一定的阈值就认为它是热点方法。

    HotSpot为每个方法准备了两类计数器：方法调用计数器和回边计数器（回边指在循环边界往回跳转）。

    - 方法调用计数器触发即时编译：

      <img src="img\java虚拟机\方法调用计数器.jpg" style="zoom:50%;" />

      方法调用计数器默认情况下统计的是一段时间内方法被调用的次数。当超过一定的时间限度，如果方法的调用次数仍然不足以让它提交给即时编译器编译，那该方法的调用计数器就会被减少一半，称为方法调用计数器热度的衰减。热度衰减的动作是在虚拟机进行垃圾收集时顺便进行的。

    - 回边计数器触发即时编译：

      <img src="img\java虚拟机\回边计数器.jpg" style="zoom:50%;" />

      回边计数器用于触发栈上的替换编译（On Stack Replace）。 它统计的就是该方法循环执行的绝对次数。

- 编译过程

  - 客户端编译器是一个相对简单快速的三段式编译器：
    - 第一个阶段，一个平台独立的前端将字节码构造成一种高级中间代码表示（HIR），在此之前编译器会在字节码上完成一部分基础优化，如方法内联、常量传播等。
    - 第二个阶段，一个平台相关的后端从HIR中生成低级中间代码表示（LIR），在此之前编译器会在HIR上完成另外一些优化，如空值检查消除、范围检查消除等。
    - 最后阶段，在平台相关的后端使用线性扫描算法在LIR上分配寄存器，并在LIR上做窥孔优化，然后产生机器代码。

  <img src="img\java虚拟机\C1编译器架构.jpg" style="zoom: 33%;" />

  - 服务端编译器通常为服务端的性能配置进行过针对性调整，可以容忍很高的优化复杂度。它会执行大部分经典的优化动作，如无用代码消除、循环展开、循环表达式外提、消除公共子表达式、常量传播、基本块重排序等，还会实施一些与Java语言特性相关的优化技术，如范围消除检查、空值消除检查等。另外还能根据性能监控信息进行一些不稳定的预测性激进优化，如守护内联、分支频率预测等。

- 相比于提前编译，优点：
  - 性能分析制导优化：通过收集性能监控信息，可以获取一些代码运行的偏好性，而静态编译通常无法获得这些信息。如条件判断通常走那条分支、方法调用通常选择哪个版本等等。
  - 激进预测性优化：通过收集性能监控信息，可以做出一些正确的可能性很大但无法保证绝对正确的预测判断，按这些假设进行优化，如果真的执行错误就退回到低级编译器或解释器上执行代码。而静态编译必须保证优化后的代码能够正确执行。如通过类继承关系分析等一系列激进的猜测做去虚拟化，以保证大部分有内联价值的虚方法都可以顺利内联，降低方法分派、调用本身的开销。
  - 链接时优化：虚拟机可以在运行期加载Class文件并进行即时优化，而静态编译的不同库之间完全独立，一个程序调用它们时，其实没有一个整体上的优化。 

##### 11.3 提前编译器

- 两条分支：一是在程序运行之前把程序代码编译成机器码的静态翻译工作；二是把原本即时编译器在运行期要做的编译工作提前做好并保存下来，下次运行到这些代码时直接把它加载进来使用，称为动态提前编译或即时编译缓存。

- 相比于即时编译，优点即为不需要占用程序运行时间和资源，没有执行时间和资源限制的压力。缺点为破坏平台中立性，与目标机器相关；字节膨胀，本地二进制码的体积明显大于字节码的体积；不能在外部动态加载新的字节码。

##### 11.4 编译器优化技术

- HotSpot即时编译器采用的优化技术列表

  ![](img\java虚拟机\编译器优化技术1.jpg)

  ![](img\java虚拟机\编译器优化技术2.jpg)

- Java语言示意的优化过程

  原始代码：

  ```java
  static class B {
  	int value;
  	final int get() {
  		return value;
  	}
  }
  public void foo() {
  	y = b.get();
  	// ... do stuff...
  	z = b.get();
  	sum = y + z;
  }
  ```

  1. 方法内联：
     - 去除方法调用的成本，如查找方法版本、建立栈帧等；
     - 为其他优化建立良好的基础，便于在更大范围上进行后续的优化手段。

     内联后的代码：

     ```java
     public void foo() {
     	y = b.value;
     	// ... do stuff...
     	z = b.value;
     	sum = y + z;
     }
     ```

  2. 冗余访问消除：假设do stuff部分不会改变b.value的值，则z=b.value就可以替换为z=y。如果把b.value看作一个表达式，也可以把这项优化看作一种公共子表达式消除。

     冗余存储消除后的代码：

     ```java
     public void foo() {
     	y = b.value;
     	// ... do stuff...
     	z = y;
     	sum = y + z;
     }
     ```

   3. 复写传播：因为这段代码的逻辑中没有必要使用一个额外的变量z，它与变量y是完全等价的，因此使用y来替代z。
  
      复写传播后的代码：
  
      ```java
      public void foo() {
      	y = b.value;
      	// ... do stuff...
      	y = y;
      	sum = y + y;
      }
      ```
  
    4. 无用代码消除：无用代码可能是永远不会被执行的代码，也可能是完全没有意义的代码。
  
       ```java
       public void foo() {
       	y = b.value;
       	// ... do stuff...
       	sum = y + y;
       }
       ```

- 经典优化技术

  - 方法内联：把目标方法的代码复制到发起调用的方法中，避免真实的方法调用。

    - Java中的方法内联问题：对于一个虚方法，编译器静态去做内联时很难确定应该使用哪个方法版本，而Java对象默认的方法就是虚方法。
    - 为了解决该问题，Java虚拟机首先引入了“类型继承关系分析（Class Hierarchy Analysis，CHA）”技术，这是整个应用程序范围内的类型分析技术，用于确定在目前已加载的类中，某个接口是否有多于一种的实现、某个类是否存在子类、某个子类是否覆盖了父类的某个虚方法等信息。依赖CHA技术，编译器在进行内联时分不同情况采取不同的处理：
      - 如果是非虚方法，直接进行内联；
      - 如果是虚方法，则向CHA查询此方法在当前程序状态下是否有多个目标版本可供选择，
        - 如果查询到只有一个版本，那就假设“应用程序的全貌就是现在运行的这样”来进行内联，被称为守护内联，它属于激进预测性优化。因为后续的动态加载类型可能会改变CHA的结论，如果虚拟机一直没有加载到导致该方法的接收者继承关系发生变化的类，则这个内联优化的代码可以一直使用，否则必须重新编译或退回到解释状态执行。
        - 如果查询到多个版本，那就使用内联缓存的方式来缩减方法调用的开销。这种状态下方法调用是真正发生的。

  - 逃逸分析：分析对象动态作用域。当一个对象在方法里面定义后，它可能被外部方法所引用，如作为调用参数传递到其他方法中，称为方法逃逸。还可能被外部线程访问到，如赋值给可以在其他线程中访问的实例变量，称为线程逃逸。

    - 如果能证明一个对象不会逃逸到方法或线程之外，或者逃逸程度比较低，则可能为这个对象实例采取不同程度的优化：

      - 栈上分配：通常对象是分配在堆上的，如果一个对象不会逃逸出线程之外，那可以让该对象分配在栈上，这样栈帧出栈就可以自动销毁该对象。栈上分配支持方法逃逸（调用其他方法时，该栈帧一定会等待），但不能支持线程逃逸。
      - 标量替换：原始数据类型被称为标量，一个数据可以被分解为更小的数据来表示，它被称为聚合量。将一个对象拆散，根据程序访问的情况，将其用到的成员变量恢复为原始数据类型来访问，这个过程称为标量替换。如果一个对象不会逃逸出方法之外，并且这个对象可以被拆散，那可以不创建该对象，而是直接创建它的若干个被这个方法使用的成员变量来替代。标量替换要求对象不逃逸。
      - 同步消除：如果一个对象不会逃逸出线程之外，那可以消除对这个对象实施的同步措施，因为不会产生线程安全问题。

    - 逃逸分析示例：

      原始伪代码：

      ```java
      public int test(int x) {
      	int xx = x + 2;
      	Point p = new Point(xx, 42);
      	return p.getX();
      }
      ```

      第一步，将Point的构造函数和getX()方法进行内联优化：

      ```java
      public int test(int x) {
      	int xx = x + 2;
      	Point p = point_memory_alloc();
      	//构造函数内联
      	p.x = xx;
      	p.y = 42;
      	return p.x;//getX()方法内联
      }
      ```

      第二步，经过逃逸分析发现Point对象实例不会发生任何逃逸，可以进行标量替换优化，将其内部的x和y直接置换出来，分解为test()方法的局部变量，避免Point实例对象被创建。

      ```java
      public int test(int x) {
      	int xx = x + 2;
      	int px = xx;
      	int py = 42;
      	return px;
      }
      ```

      第三步，通过数据流分析，发现py的值对方法不会造成任何影响，可以进行无效代码消除优化。

      ```java
      public int test(int x) {
      	return x + 2;
      }
      ```

  - 公共子表达式消除：如果一个表达式E之前已经被计算过了，并且从先前的计算到现在E中所有变量的值都没有发生变化，那么E的这次出现就成为公共子表达式。对于这种表达式直接使用计算过的结果替代E即可。

    - 示例：

      ```java
      int d = (c * b) * 12 + a + (a + b * c);
      ```

      字节码：

      ```
      6: iload_3 //c
      7: iload_2	//b
      8: imul	//c * b
      9: bipush 12	//12
      11: imul //(c * b) * 12
      12: iload_1	// a
      13: iadd	//(c * b) * 12 + a
      14: iload_1	// a
      15: iload_2	// b
      16: iload_3	// c
      17: imul	// b * c
      18: iadd	// a + b * c
      19: iadd	// (c * b) * 12 + a + (a + b * c)
      20: istore 4
      ```

      编译器发现c\*b与b\*c是一样的表达式，且计算期间b与c的值不变，则这条表达式被视为

      ```
      int d = E * 12 + a + (a + E);
      ```

      编译器还可能进行代数化简优化，在E本来就有乘法运算的前提下，把表达式变为

      ```
      int d = E * 13 + a + a;
      ```

  - 数组边界检查消除

    - 在编译器对数组索引或循环变量进行数据流分析，判定没有越界就可以在运行期将边界检查消除掉。
    - 隐式异常处理，通过注册一个Segment Fault信号的异常处理器，当不产生异常时不会进行任何判断，而产生异常时就会抛出异常并处理。

### 第五部分 高效并发

#### 第十二章 Java内存模型与线程

##### 12.1 概述

- 衡量一个服务性能的高低，每秒事务处理数（Transactions Per Second，TPS）是重要的指标之一，它代表着一秒内服务端平均能响应的请求总数。

##### 12.2 硬件的效率与一致性

- 共享内存多核系统：

  <img src="img\java虚拟机\处理器与高速缓存与主存.jpg" style="zoom: 33%;" />

- 处理器的乱序执行，只保证计算结果一致，但有中间结果被其他计算任务依赖时，就可能出现不一致问题。

##### 12.3 Java内存模型

- 主要目的是定义程序中各种变量（实例字段、静态字段、数组对象的元素）的访问规则，同时保证高效的执行速度和并发时的数据一致性（原子性、可见性与有序性）。

- 主内存与工作内存

  - Java内存模型规定了所有的变量都存储在主内存中，每条线程拥有自己的工作内存，其中保存被该线程使用的变量的主内存副本，线程对变量的所有操作都必须在工作内存中进行，而不能直接读写主内存中的数据。不同线程之间无法直接访问对方工作内存中的变量，线程间变量值的传递均需要通过主内存完成。

    <img src="img\java虚拟机\线程与工作内存与主内存.jpg" style="zoom:33%;" />

  - Java内存模型可以看作从底层硬件的角度对内存的划分，即主内存直接对应于物理硬件的内存，而工作内存可能会优先存储于寄存器和高速缓存中。第二章的Java运行时数据区是从Java语言的角度对内存的划分。

- 内存间交互操作
  - Java内存模型定义了8种操作来完成主内存与工作内存之间的交互：
    - lock（锁定）：作用于主内存的变量，它把一个变量表示为一条线程独占的状态。
    - unlock（解锁）：作用于主内存的变量，它把一个处于锁定状态的变量释放出来，释放后的变量才可以被其他线程锁定。
    - read（读取）：作用于主内存的变量，它把一个变量的值从主内存传输到线程的工作内存中，以便随后的load动作使用。
    - load（载入）：作用于工作内存的变量，它把read操作从主内存中得到的变量值放入工作内存的变量副本中。
    - use（使用）：作用于工作内存的变量，它把工作内存中一个变量的值传递给执行引擎，每当虚拟机遇到一个需要使用变量的值的字节码指令时将会执行这个操作。
    - assign（赋值）：作用于工作内存的变量，它把一个从执行引擎接收的值赋给工作内存的变量，每当虚拟机遇到一个给变量赋值的字节码指令时执行这个操作。
    - store（存储）：作用于工作内存的变量，它把工作内存中一个变量的值传送到主内存中，以便随后的write操作使用。
    - write（写入）：作用域主内存的变量，它把store操作从工作内存中得到的变量值放入主内存的变量中。
  - 这8种操作满足的一些规则：
    - read和load，store和write必须按顺序执行，但不要求是连续执行。
    - 不允许read和load、store和write操作之一单独出现，即不允许一个变量从主内存读取了但工作内存不接受，或者工作内存发起回写了但主内存不接受的情况。
    - 不允许一个线程丢弃它最近的assign操作，即变量在工作内存中改变了之后必须把该变化同步回主内存。
    - 不允许一个线程无原因地（没有发生过任何assign操作）把数据从线程的工作内存同步回主内存中。
    - 一个新的变量只能在主内存中诞生，不允许在工作内存中直接使用一个未被初始化（load或assign）的变量，即对一个变量实use、store操作之前，必须先执行load、assign操作。
    - 一个变量在同一个时刻只允许一条线程对其进行lock操作，但lock操作可以被同一条线程重复执行多次，多次执行lock后，只有执行相同次数的unlock操作，变量才会被解锁。
    - 如果对一个变量执行lock操作，那会清空工作内存中此变量的值，在执行引擎使用这个变量前，需要重新执行load或assign操作以初始化变量的值。
    - 如果一个变量事先没有被lock操作锁定，那就不允许对它执行unlock操作，也不允许去unlock一个被其他线程锁定的变量。
    - 对一个变量执行unlock操作之前，必须先把此变量同步回主内存中。

- 对于volatile型变量的特殊规则

  - 当一个变量被定义成volatile之后，它将具备两项特性：

    - 保证此变量对所有线程的可见性，即一条线程修改了此变量的值，新值对于其他线程来说是可以立即得知的（普通变量不能保证新值立即刷新到内存中）。

      但这并不能保证volatile变量在并发下的安全性，如

      ```java
      package cn.dut.test;
      
      public class VolatileTest {
          public static volatile int race = 0;
          public static void increase() {
              race++;
          }
          private static final int THREADS_COUNT=20;
      
          public static void main(String[] args) {
              Thread[] threads = new Thread[THREADS_COUNT];
              for (int i = 0; i < THREADS_COUNT; i++) {
                  threads[i] = new Thread(new Runnable() {
                      @Override
                      public void run() {
                          for (int i = 0; i < 10000; i++) {
                              increase();
                          }
                      }
                  });
                  threads[i].start();
              }
              while (Thread.activeCount() > 2)
                  Thread.yield();
              System.out.println(race);//一个小于200000的不确定的数
          }
      }
      ```

      这是因为Java的运算操作符并非原子操作，如race++的字节码指令为

      ```
      public static void increase();
          descriptor: ()V
          flags: (0x0009) ACC_PUBLIC, ACC_STATIC
          Code:
            stack=2, locals=0, args_size=0
               0: getstatic     #2                  // Field race:I
               3: iconst_1
               4: iadd
               5: putstatic     #2                  // Field race:I
               8: return
      ```

      在getstatic把race的值取到操作栈顶时，volatile关键字保证race的值在此时是正确的，但在执行iconst_1、iadd指令时，其他线程可能已经把race的值改变了，操作栈顶的值就是过期的数据了。

      另外一条字节码指令也可能需要多行代码或多条机器码指令表示。

    - 禁止指令重排序优化。会在赋值后多执行一条带有Lock前缀的空操作（内存屏障，意味着之前所有的操作已经执行完成），它会将当前处理器缓存行的数据写回到内存，同时使其他处理器里缓存了该内存地址的数据无效。这样就会使该数据对其他线程立即可见。

  - 第二个特性其实保证了当前线程下指令的执行顺序，实现对其他线程可见性，但是不能保证多个线程同时修改数据时的一致性，volatile变量的适用场景：

    - 运算结果并不依赖变量的当前值，或者能够确保只有单一的线程修改变量的值。
    - 变量不需要与其他的状态变量共同参与不变约束。

  - 对volatile变量的规则：

    - read、load、use连续且一起出现。
    - assign、store、write连续且一起出现。
    - 如果对变量V的use或assign操作先于对变量W的use或assign操作，那对变量V的read或write操作先于对变量W的read或write操作。

- long与double数据的非原子性协定（分两次32位的操作）。
- 原子性、可见性与有序性
  - 原子性：由Java内存模型直接保证的原子性变量操作包括read、load、assign、use、store、write这六个，基本数据类型的访问读写都是具备原子性的。通过synchronized关键字翻译的字节码指令monitorenter和monitorexit来隐式执行Java内存模型的lock和unlock操作。
  - 可见性：volatile保证多线程操作时变量的可见性，普通变量无法保证。synchronized保证对于一个变量unlock前，必须先把此变量同步回主内存中。final保证的可见性指被final修饰的字段在构造器中一旦被初始化完成，并且构造器没有把this的引用传递出去，那么在其他线程中就能看见final字段的值（final字段应该在堆内存的Class对象中）。
  - 有序性：volatile本身包含禁止指令重排序的语义，synchronized保证持有同一个锁的两个同步块只能串行执行。

- 先行发生原则

  - 先行发生是Java内存模型中定义的两项操作之间的偏序关系，操作A先行发生于操作B，就是操作B发生之前，操作A产生的影响能被操作B观察到。

  - Java内存模型的一些先行发生原则：

    - 程序次序规则：在一个线程内，按照控制流顺序，书写在前面的操作先行发生于书写在后面的操作。
    - 管程锁定规则：一个unlock操作先行发生于后面（时间上）对同一个锁的lock操作。
    - volatile变量规则：对一个volatile变量的写操作先行发生于后面（时间上）对这个变量的读操作。
    - 线程启动规则：Thread对象的start()方法先行发生于此线程的每一个动作。
    - 线程终止规则：线程中的所有操作都先行发生于对此线程的终止检测。通过Thread::join()方法、Thread::isAlive()方法检测线程是否已终止。
    - 线程中断规则：对线程interrupt()方法的调用先行发生于被中断线程（的代码）检测到中断事件的发生。通过Thread::interrupted()方法检测是否有中断发生。
    - 对象终结规则：一个对象的初始化完成先行发生于它的finalize()方法的开始。
    - 传递性：如果操作A先行发生于操作B，操作B先行发生于操作C，那么操作A先行发生于操作C。

  - 例子：

    ```java
    private int value = 0;
    public void setValue(int value) {
    	this.value = value;
    }
    public int getValue() {
    	return value;
    }
    ```

    如果线程A先（时间上）调用了setValue(1)，然后线程B调用同一个对象的getValue()，那么线程B得到的结果是不确定的。即时间上先发生并不代表先行发生。

    这个例子中所有的先行发生原则都不满足，因此上述操作不是线程安全的。有两个方案：

    - 把getter/setter方法定义为synchronized方法，套用管程锁定规则。
    - 把value定义为volatile变量，套用volatile变量规则。由于setter方法对value的修改不依赖value的原值，满足volatile使用场景。

    ```java
    int i = 1;
    int j = 2;
    ```

    这两条赋值语句在同一个线程中，根据程序次序规则，“int i = 1;”的操作先行发生于“int j = 2;“，但是”int j = 2;“的代码完全可能先被处理器执行，这并不影响先行发生原则的正确性，因为我们在这条线程之中没有办法感知这一点（也就是说先行发生原则正确，但这段代码其实没有办法体现，因为i和j的赋值操作没有任何关系，实际上谁先发生是不确定的）。即先行发生也不代表时间上先发生。

  - 判断并发安全问题时以先行发生原则为准。

##### 12.4 Java与线程

- 线程的实现：

  - 实现线程主要有三种方式：

    - 内核线程实现：也被称为1：1实现。内核线程（Kernel-Level Thread，KLT）就是直接由操作系统内核支持的线程，这种线程由内核来完成线程切换，内核通过操纵调度器对线程进行调度，并负责将线程的任务映射到各个处理器上。

      程序一般不会直接使用内核线程，而是使用内核线程的一种高级接口，轻量级进程（Light Weight Process，LWP），即通常意义上所讲的线程，由于每个轻量级进程都有一个内核线程支持，因此只有先支持内核线程，才能有轻量级进程。

      <img src="img\java虚拟机\轻量级进程与内核线程.jpg" style="zoom: 50%;" />

      优点：每个轻量级进程都是一个独立的调度单元，整个进程受阻塞影响较小。

      缺点：基于内核线程实现，各种线程操作需要调用代价较高的系统调用，且会消耗一定的内核资源，可创建数量有限。

      - 内核线程的调度成本主要来自于用户态和核心态之间的状态切换，而这两种状态转换的开销主要来自于响应中断、保护和恢复执行现场的成本。

    - 用户线程实现：也被称为1：N实现。狭义上的用户线程指完全建立在用户空间的线程库上，系统内核不能感知到用户线程的存在及如何实现的。

      <img src="img\java虚拟机\进程与用户线程.jpg" style="zoom:50%;" />

      优点：不需要系统内核支援，操作可以快速且低消耗，可创建规模更大的线程数量。

      缺点：所有的线程操作都需要用户程序自己处理。

    - 混合实现：也被称为N：M实现。用户线程完全建立在用户空间中，用户线程的创建、切换、析构等操作依然廉价，并且可以支持大规模的用户线程并发。而操作系统支持的轻量级进程作为用户线程和内核线程之间的桥梁，可以使用内核提供的线程调度功能及处理器映射，并且用户线程的系统调用通过轻量级进程完成，大大降低整个进程被完全阻塞的风险。

      <img src="img\java虚拟机\用户线程与轻量级进程.jpg" style="zoom:50%;" />

  - Java线程的实现：HotSpot的每一个线程都是直接映射到一个操作系统原生线程来实现的，而且中间没有额外的间接结构。

- Java线程调度

  - 线程调度是指系统为线程分配处理器使用权的过程，调度主要方式有两种：

    - 协同式线程调度：线程的执行时间由线程本身来控制，线程把自己的工作执行完成后，要主动通知系统切换到另外一个线程上。

      优点：实现简单，一般没有什么线程同步问题。

      缺点：线程执行时间不可控制，可能会出现阻塞问题。

    - 抢占式线程调度：各个线程由系统来分配执行时间，线程的切换不由线程本身决定。Java使用该方式，一共设置了10个级别的线程优先级用于给系统的调度提供“建议”。

- 状态转换：Java定义了6种线程状态，在任意一个时间点中，一个线程只能有且只有其中一种状态，并且可以通过特定的方法在不同状态之间转换。

  - 新建（New）：创建后尚未启动的线程处于这种状态。
  - 运行（Runnable）：包括操作系统线程状态中Running和Ready，也就是处于此状态的线程有可能正在执行，也有可能正在等待着操作系统为它分配执行时间。
  - 无限期等待（Waiting）：处于这种状态的线程不会被分配处理器执行时间，需要等待被其他线程显式唤醒。
  - 限期等待（Timed Waiting）：处于这种状态的线程不会被分配处理器执行时间，不过一定时间之后他们会由系统自动唤醒。
  - 阻塞（Blocked）：线程在等待着获取一个排它锁，这个事件将在另外一个线程放弃这个锁的时候发生。程序进入同步区域的时候，线程会进入这种状态。
  - 结束（Terminated）：线程已经结束执行。

  它们的转换关系：

  <img src="img\java虚拟机\线程状态转换.jpg" style="zoom:50%;" />

#### 第十三章 线程安全与锁优化

##### 13.2 线程安全

- 线程安全：当多个线程同时访问一个对象时， 如果不用考虑这些线程在运行时环境下的调度和交替执行，也不需要进行额外的同步，或者在调用方进行任何其他的协调操作，调用这个对象的行为都可以获得正确的结果，那就称这个对象是线程安全的。
- Java语言中的线程安全
  - 按照线程安全的“安全程度”由强至弱来排序，各种共享数据可以分为五类：
    - 不可变：不可变的数据一定是线程安全的。如果是基本数据类型，在定义时使用final关键字修饰就可以保证它是不可变的。如果是一个对象，需要对象自行保证其行为不会对其状态产生任何影响，如String类的对象，调用substring()、replace()等方法都只会返回一个新的字符串对象，最简单的方式为将对象内带有状态的变量都声明为final。
    - 绝对线程安全：一个类要做到绝对线程安全，就必须在它内部维护一组一致性的快照访问。
    - 相对线程安全：保证对这个对象单次的操作是线程安全的。对于一些特定顺序的连续调用，需要在调用端使用额外的同步手段，Java中大部分声称线程安全的类都属于这种类型。
    - 线程兼容：指对象本身并不是线程安全的，但是可以通过在调用端正确使用同步手段来保证对象在并发环境中可以安全地使用。
    - 线程对立：不管调用端是否采取了同步措施，都无法在多线程环境中并发使用代码。

- 线程安全的实现方法

  - 互斥同步/阻塞同步：使用互斥手段实现同步。

    - Java中最基本的互斥同步手段就是synchronized关键字。monitorenter指令执行时，尝试获取对象的锁，如果该对象没被锁定，或者当前线程已经持有了该对象的锁，就把锁的计数器的值加一。monitorexit指令执行时，将锁计数器的值减一，一旦计数器的值为零，锁就立即被释放。如果获取对象锁失败，当前线程就进入阻塞等待状态，知道请求锁定的对象被持有它的线程释放为止。
      - 被synchronized修饰的同步块对同一条线程来说是可重入的，即同一条线程反复进入同步块也不会出现自己把自己锁死的情况。
      - 被synchronized修饰的同步块在持有锁的线程执行完毕并释放锁之前，会无条件地阻塞后面其他线程的进入，即无法强制已获取锁的线程释放锁，也无法强制正在等待锁的线程中断等待或超时退出。
    - java.util.concurrent包提供的Lock接口。重入锁是Lock接口最常见的一种实现，与synchronized相比增加了一些高级功能：
      - 等待可中断：持有锁的线程长期不释放锁的时候，正在等待的线程可以选择放弃等待，改为处理其他事情。
      - 公平锁：多个线程在等待同一个锁时，必须按照申请锁的时间顺序来依次获取锁。
      - 锁绑定多个条件：一个重入锁对象可以同时绑定多个Condition对象。

    - 重入锁与synchronized的选择：

      - 只需要基础的同步功能时，synchronized更加清晰简单。

      - Lock应该由程序员确保在finally块中释放锁，synchronized由虚拟机确保自动释放。

    ```java
  package cn.dut.test;
    
  public class VolatileTest {
        public static volatile int race = 0;
        public static void increase() {
            synchronized (VolatileTest.class){
                race++;
            }
        }
        private static final int THREADS_COUNT=20;
    
        public static void main(String[] args) {
            Thread[] threads = new Thread[THREADS_COUNT];
            for (int i = 0; i < THREADS_COUNT; i++) {
                threads[i] = new Thread(new Runnable() {
                    @Override
                    public void run() {
                        for (int i = 0; i < 10000; i++) {
                            increase();
                        }
                    }
                });
                threads[i].start();
            }
    
            while (Thread.activeCount() > 2)
                Thread.yield();
            System.out.println(race);//200000
        }
    }
    ```
  
  - 非阻塞同步：基于冲突检测的乐观并发策略，不管风险先进行操作，如果没有其他线程争用共享数据，那操作就成功了，如果共享数据被争用，那就进行其他的补偿措施。这种不需要把线程阻塞挂起的同步操作称为非阻塞同步。
  
    - 需要硬件保证某些语义上需要多次操作的行为可以通过一条处理器指令就能完成，如测试并设置（Test-and-Set）、获取并增加（Fetch-and-Increment）、交换（Swap）、比较并交换（Compare-and-Swap，CAS）、加载链接/条件存储（Load-Linked/Store-Conditional，LL/SC）。
    
    ```java
    package cn.dut.test;
    
    import java.util.concurrent.atomic.AtomicInteger;
    
    public class AtomicTest {
        public static AtomicInteger race = new AtomicInteger(0);
    
        public static void increase() {
            race.incrementAndGet();//CAS操作
        }
    
        private static final int THREADS_COUNT = 20;
    
        public static void main(String[] args) {
            Thread[] threads = new Thread[THREADS_COUNT];
            for (int i = 0; i < THREADS_COUNT; i++) {
                threads[i] = new Thread(new Runnable() {
                    @Override
                    public void run() {
                        for (int i = 0; i < 10000; i++) {
                            increase();
                        }
                    }
                });
                threads[i].start();
            }
    
            while (Thread.activeCount() > 2)
                Thread.yield();
            System.out.println(race);//200000
        }
    }
    ```
    
  - 无同步方案：同步只是保障存在共享数据争用时正确性的手段，如果能让一个方法本来就不涉及共享数据，那它自然就不需要任何同步措施去保证其正确性。
  
    - 可重入代码/纯代码：可以在代码执行的任何时刻中断它，转而去执行另外一段代码，而在控制权返回后，原来的程序不会出现任何错误，也不会对结果有所影响。
  
      特征：不依赖全局变量、存储在堆上的数据和公用的系统资源，用到的状态量都由参数中传入，不调用非可重用的方法等。
  
      具备可重用性：如果一个方法的返回结果是可以预测的，只要输入了相同的数据，就能返回相同的结果。
  
    - 线程本地存储：如果一段代码中所需要的数据必须与其他代码共享，那就尽量使这些共享数据的代码在同一个线程中执行。通过java.lang.ThreadLocal类实现线程本地存储。

##### 13.3 锁优化

- 自旋锁与自适应自旋
  - 不让线程等待锁时就立即进入挂起（阻塞）状态，避免用户态和核心态的转换代价，让线程执行一个忙循环（自旋），即自旋锁。
  - 当等待时间很短时，自旋的效果非常好，否则自旋的线程只是白白消耗处理器资源，因此需要设定自旋等待的时间。
  - JDK6中引入了自适应的自旋，由前一次在同一个锁上的自旋时间及锁的拥有者的状态来决定的。如果在同一个锁对象上，自旋等待刚刚成功获得过锁，并且持有锁的线程正在运行中，那虚拟机就会认为这次自旋也很有可能成功，进而允许自旋等待持续更长时间。如果对于某个锁，自旋很少成功获得过锁，那以后获取这个锁时就可能直接省略掉自旋过程。

- 锁消除
  - 虚拟机即时编译器在运行时，对一些代码要求同步，但是对被检测到不可能存在共享数据竞争的锁进行消除。依赖于逃逸分析，如果判断到一段代码中堆上的所有数据都不会被其他线程访问到，那就无需同步加锁。

- 锁粗化
  - 如果一串零碎的操作都对同一个对象加锁，将会把加锁同步的范围扩展到整个操作序列的外部。

- 轻量级锁

  - 对象头Mark Word（32位示例）：

    <img src="img\java虚拟机\对象头MarkWord.jpg" style="zoom: 50%;" />

  - 工作过程：在代码即将进入同步块的时候，如果此同步对象没有被锁定，虚拟机首先将在当前线程的栈帧中建立一个名为锁记录的空间，用于存储锁对象目前的Mark Word拷贝。

    <img src="img\java虚拟机\轻量级锁CAS操作之前状态.jpg" style="zoom: 33%;" />

    然后虚拟机将使用CAS操作尝试把对象的Mark Word更新为指向Lock Record的指针（注意Mark Word本身就是一个指针长度的数据，但是这个指针只占了前30位）。

    如果更新成功，即代表该线程拥有了这个对象的锁，并且将Mark Word的锁标志位转变为“00”，表示此对象处于轻量级锁定状态。

    <img src="img\java虚拟机\轻量级锁CAS操作之后状态.jpg" style="zoom: 33%;" />

    如果更新失败，意味着至少存在一条线程与当前线程竞争该对象的锁。虚拟机会首先检查对象的Mark Word释放指向当前线程的栈帧，如果是，说明当前线程已经拥有了这个对象的锁，那直接进入同步块继续执行（Q：指的是已经获得锁的线程又尝试获得锁的行为，如同步块内又有同步块，而且是同一个对象？）。否则说明这个锁对象已经被其他线程抢占，如果出现两条以上的线程争用同一个锁的情况，那轻量级锁就不再有效，必须要膨胀为重量级锁，即Mark Word指向重量级锁，锁标志位变为“10”，后面等待锁的线程进入阻塞状态。

    - 重量级锁是一个ObjectMonitor对象，可以存储更多信息。

    解锁时，如果对象的Mark Word仍然指向线程的锁记录，那就用CAS操作把对象当前的Mark Word和线程中复制的Mark Word替换回来。

    如果成功替换，那整个同步过程就完成了。如果替换失败，说明有其他线程尝试过获取该锁，就要在释放锁的同时，唤醒被挂起的线程。

  - 轻量级锁提升同步性能建立在“对于绝大部分锁，在整个同步周期内都不存在竞争”的经验法则。如果没有锁竞争，轻量级锁通过CAS操作避免了使用互斥的开销（意思是没有锁竞争或锁竞争并不激烈（线程接近于交替执行），竞争锁的线程先使用自旋等待一下，成功了就执行同步块，这样避免直接申请重量级锁？）；但如果有锁竞争，除了互斥的开销，还增加了CAS操作的开销。

- 偏向锁：

  - 这个锁会偏向于第一个获得它的线程，如果在接下来的执行过程中，该锁一直没有被其他线程获取，则持有偏向锁的线程将永远不需要再进行同步。

  - 工作过程：当锁对象第一次被线程获取时，虚拟机把对象头中的标志位设置为“01”，偏向模式设置为“1”。同时使用CAS操作把取到这个锁的线程的ID记录在对象头Mark Word中。如果CAS操作成功，持有偏向锁的线程以后每次进入这个锁相关的同步块时，虚拟机都可以不再进行任何同步操作（加锁、解锁、对Mark Word的更新操作等）。

    一旦出现另外一个线程去尝试获取这个锁，偏向模式马上结束。根据锁对象目前是否处于被锁定的状态撤销偏向，撤销后标志位恢复到未锁定或轻量级锁定的状态。 

    <img src="img\java虚拟机\偏向锁与轻量级锁的状态转换.jpg" style="zoom: 50%;" />

  - 如何保存哈希码：当一个对象已经计算过一致性哈希码后，它就再也无法进入偏向锁状态了。而当一个对象当前正处于偏向锁状态，又收到需要计算其一致性哈希码的请求时，它的偏向状态会被立即撤销，并且锁会膨胀为重量级锁。在重量级锁的实现中，它具有记录非加锁状态下Mark Word的字段，可以存储原来的哈希码。

- 一个关于偏向锁、轻量级锁、重量级锁的解释：

  作者：absfree
  链接：https://www.zhihu.com/question/53826114/answer/160222185
  来源：知乎
  著作权归作者所有。商业转载请联系作者获得授权，非商业转载请注明出处。

  

  首先简单说下先偏向锁、轻量级锁、重量级锁三者各自的应用场景：

  - 偏向锁：只有一个线程进入临界区；
  - 轻量级锁：多个线程交替进入临界区**；**
  - 重量级锁：多个线程同时进入临界区。

  还要明确的是，偏向锁、轻量级锁都是JVM引入的锁优化手段，目的是降低线程同步的开销。比如以下的同步代码块：

  ```java
  synchronized (lockObject) {
      // do something
  }
  ```

  上述同步代码块中存在一个临界区，假设当前存在Thread#1和Thread#2这两个用户线程，分三种情况来讨论：

  - 情况一：只有Thread#1会进入临界区；
  - 情况二：Thread#1和Thread#2交替进入临界区；
  - 情况三：Thread#1和Thread#2同时进入临界区。

  上述的情况一是偏向锁的适用场景，此时当Thread#1进入临界区时，JVM会将lockObject的对象头Mark Word的锁标志位设为“01”，同时会用CAS操作把Thread#1的线程ID记录到Mark Word中，此时进入偏向模式。所谓“偏向”，指的是这个锁会偏向于Thread#1，若接下来没有其他线程进入临界区，则Thread#1再出入临界区无需再执行任何同步操作。也就是说，若只有Thread#1会进入临界区，实际上只有Thread#1初次进入临界区时需要执行CAS操作，以后再出入临界区都不会有同步操作带来的开销。

  然而情况一是一个比较理想的情况，更多时候Thread#2也会尝试进入临界区。若Thread#2尝试进入时Thread#1已退出临界区，即此时lockObject处于未锁定状态，这时说明偏向锁上发生了竞争（对应情况二），此时会撤销偏向，Mark Word中不再存放偏向线程ID，而是存放hashCode和GC分代年龄，同时锁标识位变为“01”（表示未锁定），这时Thread#2会获取lockObject的轻量级锁。因为此时Thread#1和Thread#2交替进入临界区，所以偏向锁无法满足需求，需要膨胀到轻量级锁。

  再说轻量级锁什么时候会膨胀到重量级锁。若一直是Thread#1和Thread#2交替进入临界区，那么没有问题，轻量锁hold住。一旦在轻量级锁上发生竞争，即出现“Thread#1和Thread#2同时进入临界区”的情况，轻量级锁就hold不住了。 （根本原因是轻量级锁没有足够的空间存储额外状态，此时若不膨胀为重量级锁，则所有等待轻量锁的线程只能自旋，可能会损失很多CPU时间）