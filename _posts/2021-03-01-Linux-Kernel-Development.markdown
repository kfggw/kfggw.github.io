---
layout: post
title: Linux Kernel Development
---

[TOC]

## 进程管理

1. 什么是进程

2. 内核如何描述进程

3. 进程的生命周期

4. 进程之间的关系

5. 如何创建进程

6. 内核中的线程如何实现

7. 如何终结进程

## 进程调度

1. 解决什么问题
   保证系统的响应时间, 提高CPU的利用率

2. 不同类型的进程
   - IO密集
   - CPU密集

3. 不同的调度算法

## 系统调用

1. 为什么需要系统调用?
   - 提供给上层的应用程序抽象的接口, 避免直接面对硬件的复杂性
   - 把应用程序和内核分离开, 保证系统的安全卡靠

2. 系统调用的工作过程
   1) 用户请求系统调用
      - 系统调用号  --> 体系架构相关的寄存器
      - 系统调用参数 --> 体系架构相关的寄存器
   2) 产生软中断
   3) 进入系统调用中断处理程序
   4) 根据系统调用号转到对应的处理程序
   5) 取出寄存器中的参数, 执行系统调用

---

## 内核同步
1. 什么是临界区, 竞争条件?

2. 自旋锁, 信号量, 互斥体这三种类型的锁的原理和区别


---

## 国防科技大学操作系统课程

### 进程和进程调度

    进程是程序在运行时CPU的状态, 内存的状态, 占用的相关资源的集合, 是
    程序运行的活动. 进程在执行过程中总是在, "New" "Ready" "Running"
    "Block" "Exit"这些状态之间进行转换的. 操作系统的调度实际上是在合适
    的时机, 按照某种原则选择处于就绪状态的进程占用CPU, 选取进程的原则
    实际上体现了调度算法, 当有新就绪的进程或者有新的阻塞状态的进程, 说
    明出现了满足调度发生的条件, 实际的调度动作是在内核返回用户程序之前
    进行的.

### 内存管理
1. 如何"放"的问题:
   - 单道程序连续分配
     每次只把一个完整程序文件读入到内存, 并连续存放
     
   - 多道程序连续分配
     首先把内存划分成空闲块队列, 然后按照进程的调度顺序, 以此从空闲块
     队列中为进程分配内存空间, 每次分配可以采取不同的分配策略(首次满足
     /最优满足/最大满足), 程序文件同样是连续的分配到空闲块中的. 注意,
     每次分配之后需要更空闲块队列, 空闲块队列可以用BitMap数组描述, 因
     为每个空闲块的大小相同, 可以用0/1表示块的使用状态, 当程序的内存空
     间被回收时, 需要更新BitMap的状态.
     
   - 页式内存分配
     将程序的逻辑地址空间和实际的物理地址空间分开, 两种地址空间被划分
     成大小相等的页(Page)/页桢(Frame), 这里的逻辑地址空间大小只受寻址
     范围的限制, 程序可以认为有一个独占的地址空间. 操作系统为每个进程
     维护一个页表, 用来映射逻辑页号和物理页号, 然后根据页号以及页内偏
     移计算实际的物理地址, 页的大小通常设置成2的幂次, 方便硬件进行地址
     转换操作, 通过页式内存分配的方式可以方便实现内存的共享和保护机制,
     当需要共享时只需要让两个逻辑页对应相同的物理页即可, 在访问内存时
     通过检查页号范围, 可以避免内存访问非法的物理内存区域, 起到保护的
     作用. CPU提供了"快表", MMU相关的寄存器, 可以缓存进程的一部分页表
     项目, 这样就不必每次查OS维护的页表, 提高了访问内存的速度. 由于进
     程在运行过程中可能有些页暂时不在内存中, 而是在磁盘的交换分区中,
     因此页表项要提供必要的字段去维护进程中的页的状态, 告知操作系统在
     产生缺页错误时该从哪里得到页面的内容, 当物理内存不足时, 需要OS执
     行某中页淘汰策略, 这时页表项需要记录被淘汰的页在交换分区的位置.
     
   - 段式内存分配

   - 段页式内存分配 

   - 改进的页式内存分配

   - 多级页表内存分配


### 文件和文件系统

1. 为什么需要文件和文件系统?

   - 提供了一种按照逻辑名引用大容量存储设备上数据的方法
   - 文件可以看成是一种虚拟化的存储设备
   - 用户不必关心存储设备的细节, 不用关心数据在设备上的保存位置, 而只
   需要通过逻辑上的文件名就可以实现对数据的读写访问

2. 文件系统的功能

   - 按文件名访问数据
   - 对数据访问的权限控制
   - 对存储设备的分配

3. 其他关于文件和文件系统的理解

   - 文件只是用户在逻辑上使用设备的方式, 从用户的角度看, 文件可以被视
   为字节流序列, 但实际操作系统在存储文件数据内容时可以分散到磁盘空间
   的不同块(假设访问的是磁盘文件)

   - 文件可以有不同的访问控制权限, 确保数据不会被恶意破坏, 最暴力的方
     法是维护一个所有用户对所有文件的访问权限矩阵, 矩阵中每个项记录用
     户对特定文件的权限, 但这种方式不具有实用价值, 因为系统中的用户和
     文件会随时发生变化, 维护这个权限矩阵的代价很大, 实际上是采用一种
     更灵活的方式, 对每个文件来说, 将系统中的用户可以分成3类, 即所有者,
     所属组以及其他人, 针对这三类用户设置不同的文件访问权限就不必考虑
     用户动态变化的问题了, 权限可以保存在描述文件的数据结构中

   - 系统中的文件被组织成一个树形结构, 文件通过文件路径可以进行标识,
     系统中的文件或者目录实际上都是文件, 不过它们具有不同的文件类型,
     为了首先不同的路径访问相同的文件, 可以把树形结构改造成有向无环图
     的形式, 实际的做法是用符号链接, 通过符号链接文件记录实际文件的路
     径, 在访问文件时用符号链接内的数据即可找到目标文件

   - 为了提高IO的速度, 通常可以采用预读, 滞后写, 以及DMA的方式. 预读假
     设用户读取文件是线性的, 提前将之后的数据进行读取; 滞后写, 即不是
     立即把数据写入到文件, 而是等待缓冲区满了之后再进行写入; DMA的方式
     建立在IO控制器的功能已经足够强大的基础上, IO控制器可以直接和内存
     进行数据交换, 而不必通过CPU进行中转

   - 操作系统的整体设计依然是分层的, 最上层的用户程序通过调用系统调用,
     享受操作系统提供的服务, 为了应对不同类型的设备, 操作系统会包含一
     个和硬件无关的抽象层, 而抽象层又依赖设备相关的下层提供服务, 总之
     只要定义好个层之间的接口, 各层那边的实现能满足接口定义的语义, 那
     么整个系统就是可控的, 各层可以在满足接口的前提下用不同的实现, 这
     种分层的思想应该是贯穿整个计算机科学的