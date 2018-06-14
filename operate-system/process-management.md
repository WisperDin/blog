## 操作系统-进程管理

## 问题引入

- 进程，线程区别
- 进程通信
- 线程同步
- 内核级线程，用户级线程
- 进程，线程状态
- 死锁
- 用户态 核心态 系统调用
- 进程的地址空间
- 进程调度
- 父子进程，孤儿进程...
- fork进程时操作
- 中断





### 进程 & 线程

进程时程序运行的实例，是系统进行资源调度和分配的基本单位，拥有自己的地址空间

线程是进程的一个实体，是CPU调度的分派的基本单位，线程只拥有一些运行中必不可少的资源(程序计数器，寄存器，栈) 但是线程可以共享其所在进程的资源（地址空间，打开的文件...）

一个进程可以有多个线程，一个线程只能属于一个进程

进程切换的开销大，线程切换的开销小

| **区别**     | **进程**                                                     | **线程**                                                     |
| ------------ | ------------------------------------------------------------ | ------------------------------------------------------------ |
| **概念**     | 进程是表示资源分配的基本单位。                               | 线程是操作系统可识别的最小执行和调度单位。线程体现的特征是可执行的，是CPU资源的分派单位。 |
| **关系**     | 一个进程可以有多个线程，但至少有一个线程。                   | 一个线程线程必定是属于某个进程的。                           |
| **资源分配** | 资源只分配给进程，同一进程的所有线程共享该进程的所有资源。 同一进程中的多个线程共享代码段(代码和常量)，数据段(全局变量和静态变量)，扩展段(堆存储)。当进程结束时，所有的资源被回收。 | 每个线程有自己独立的栈段，栈段又叫运行时段，用来存放所有局部变量和临时变量。当进程结束时，线程作为进程的资源也会被终止。 |
| **系统开销** | 创建或撤消进程时，系统都要为之分配或回收资源，如内存空间、I/O设备等。在进行进程切换时，涉及到整个当前进程CPU环境的保存以及新被调度运行的进程的CPU环境的设置。 | 线程创建和切换只须保存和设置少量寄存器的内容，并 不涉及存储器管理方面的操作。 |
| **通信**     | 进程间通信：管道，信号（事件），消息队列，共享内存，内存映射，信号量，套接字。 | 共享进程资源可直接访问。                                     |
| **同步**     | 进程同步实际上是指不同进程中的线程同步。注：某些同步方式不能跨进程，如临界区。 | 互斥，信号量，事件，条件变量，**临界区**，读写锁等。         |

### 进程通信

- 共享内存

  多个进程可以访问同一块内存空间，往往需要结合其他通信基址达到进程间的同步，互斥

  能够很容易控制容量，速度快

- 管道 Pipe / 有名管道 Named Pipe

  半双工的通信，数据只能单向流动

  - 管道可用于具有亲缘关系进程的通信
  - 有名管道还允许无亲缘关系进程间通信

- 信号 Signal

  通知接受进程某个事件的发生

- 消息队列 Message Queue 

  消息的链接表，克服了信号传递信息少、管道只能承载无格式字节流以及缓冲区大小受限等缺点

  - 直接方式，发送进程将消息挂在接收进程的消息缓冲队列中
  - 间接方式，发送进程将消息送到中间设施信箱中，接收进程从信箱获取消息

- 信号量 semaphore 

  进程间以及同一进程不同线程间的同步手段

  信号量是一个计数器，可以控制多个进程对共享资源的访问，常作为一种锁机制

- 套接字 socket

  更为一般的进程间通信，可用于不同机器间通信

- 管程 monitor




### 线程同步

- 信号量 semaphore

  允许同一时刻多个线程访问同一资源，但需要控制同一时刻访问此资源的最大线程数

  (PV操作)

  如果一个线程试图递减一个信号量，但这个信号量的值 已经为0，则线程将会阻塞。另一个线程“发出(post)”这个信号(semaphore)，使用信号量大于0之后，被阻塞的线程才会被释放。

- 互斥量 mutex

  简化的信号量，不需要计数器

  只有拥有互斥对象的线程才具有访问资源的权限，由于互斥对象只有一个，因此任何情况下共享资源都不会同时被多个线程访问。

  **可以用于同步不同进程的线程**

- 信号(事件)


  一种通知的操作，可以实现不同进程中的线程同步

- 临界区 critical section

  对独占性资源的保护

  通过对多线程的串行化来访问 公共资源或一段代码，速度快，适合控制数据访问。

  在任意时刻只允许一个线程对共享资源进行访问。如果有多个线程试图同时访问临界区，那么在有一个线程进入后其他所有试图访问此临界区的线程将 被挂起，并一直持续到进入临界区的线程离开。

  临界区在被释放后，其他线程可以继续抢占，并以此达到用原子方式操作共享资源的目的。

  **只能用于同步本进程的线程**

- 自旋锁

  专为多处理器并发引入，在内核中大量应用与中断处理等部分

  自旋锁最多只能被一个内核任务占有，如果一个内核任务尝试请求一个已被持有的自旋锁，那么这个任务线程就会一直**忙等待**（循环检查锁的情况） 直至锁被释放(特别耗费处理器时间，适用范围在短期间)

- 读写锁

  特殊的自旋锁，读模式锁定意味着对共享资源可以进行共享读，写模式锁定意味着独占着资源，不允许其他读写操作

  如果当前没有读者写者锁定，写者可以立刻获得读写锁，否则写者必须自旋，等待没有任何写者或读者

  如果读写锁没有写者，那么读者可以立刻获得该锁，否则读者必须自旋，直到写者释放该锁

- 条件变量





### 管程 monitor

管程是一个由过程，变量及数据结构组成的集合

提出原因：**不当的使用信号量非常容易引发死锁，所以提出更高级的同步原语**

实现管程通常用一个互斥量或者二元信号量，而且由实现管程的编译器实现而非程序员（对比信号量/互斥量，相当于替程序员管理信号量，实现互斥），减少出错概率法

管程提供了实现互斥的一种简单途径

**管程结构确保了同时只能有一个进程在管程内活动**

此外，为了解决管程中的同步问题，引入了条件变量，以及其相关的wait & signal操作

条件变量存在在管程内部，对同一个条件变量调用操作的进程将和条件变量建立一定的联系



### 条件变量

相比于轮询耗费cpu时间去忙等待，条件变量节省了cpu资源

> 对管程内部的条件变量x，当线程调用x.wait() 时将自身**挂起** 到条件变量x中
>
> 当另一个线程调用x.signal()时，在x上挂起的线程会被唤醒，如果此时没有线程在x上，操作将忽略

> **条件变量的作用并不是保证在同一时刻仅有一个线程访问某一个共享数据，而是在对应的共享数据的状态发生变化时，通知其他因此而被阻塞的线程。**



### Golang 中采用 条件变量+锁 实现读者写者问题

```go
// Go 之 条件变量 & 锁使用
//
// Copyright (c) 2015 - Batu <1235355@qq.com>
//
// 创建一个文件存放数据,在同一时刻,可能会有多个Goroutine分别进行对此文件的写操作和读操作.
// 每一次写操作都应该向这个文件写入若干个字节的数据,作为一个独立的数据块存在,这意味着写操作之间不能彼此干扰,写入的内容之间也不能出现穿插和混淆的情况
// 每一次读操作都应该从这个文件中读取一个独立完整的数据块.它们读取的数据块不能重复,且需要按顺序读取.
// 例如: 第一个读操作读取了数据块1,第二个操作就应该读取数据块2,第三个读操作则应该读取数据块3,以此类推
// 对于这些读操作是否可以被同时执行,不做要求. 即使同时进行,也应该保持先后顺序.
package main

import (
	"fmt"
	"sync"
	"time"
	"os"
	"errors"
	"io"
)

//数据文件的接口类型
type DataFile interface {
	// 读取一个数据块
	Read() (rsn int64, d Data, err error)
	// 写入一个数据块
	Write(d Data) (wsn int64, err error)
	// 获取最后读取的数据块的序列号
	Rsn() int64
	// 获取最后写入的数据块的序列号
	Wsn() int64
	// 获取数据块的长度
	DataLen() uint32
}

//数据类型
type Data []byte

//数据文件的实现类型
type myDataFile struct {
	f *os.File	//文件
	fmutex sync.RWMutex //被用于文件的读写锁
	rcond   *sync.Cond   //读操作需要用到的条件变量
	woffset int64 // 写操作需要用到的偏移量
	roffset int64 // 读操作需要用到的偏移量
	wmutex sync.Mutex // 写操作需要用到的互斥锁
	rmutex sync.Mutex // 读操作需要用到的互斥锁
	dataLen uint32 //数据块长度
}

//初始化DataFile类型值的函数,返回一个DataFile类型的值
func NewDataFile(path string, dataLen uint32) (DataFile, error){
	//f, err := os.OpenFile(path, os.O_APPEND|os.O_RDWR|os.O_CREATE, 0666)
	f,err := os.Create(path)
	if err != nil {
		fmt.Println("Fail to find", f, "cServer start Failed")
		return nil, err
	}

	if dataLen == 0 {
		return nil, errors.New("Invalid data length!")
	}

	df := &myDataFile{
		f : f,
		dataLen:dataLen,
	}
	//创建一个可用的条件变量(初始化),返回一个*sync.Cond类型的结果值,我们就可以调用该值拥有的三个方法Wait,Signal,Broadcast
	df.rcond = sync.NewCond(df.fmutex.RLocker())
	return df, nil
}

//获取并更新读偏移量,根据读偏移量从文件中读取一块数据,把该数据块封装成一个Data类型值并将其作为结果值返回

func (df *myDataFile) Read() (rsn int64, d Data, err error){
	// 读取并更新读偏移量
	var offset int64
	// 读互斥锁定
	df.rmutex.Lock()
	offset = df.roffset
	// 更改偏移量, 当前偏移量+数据块长度
	df.roffset += int64(df.dataLen)
	// 读互斥解锁
	df.rmutex.Unlock()

	//读取一个数据块,最后读取的数据块序列号
	rsn = offset / int64(df.dataLen)
	bytes := make([]byte, df.dataLen)
	//读写锁:读锁定
	df.fmutex.RLock()
	defer df.fmutex.RUnlock()

	for {
		_, err = df.f.ReadAt(bytes, offset)
		if err != nil {
			if err == io.EOF {
				//暂时放弃fmutex的 读锁,并等待通知的到来
				df.rcond.Wait()
				continue
			}
		}
		break
	}
	d = bytes
	return
}

func (df *myDataFile) Write(d Data) (wsn int64, err error){
	//读取并更新写的偏移量
	var offset int64
	df.wmutex.Lock()
	offset = df.woffset
	df.woffset += int64(df.dataLen)
	df.wmutex.Unlock()

	//写入一个数据块,最后写入数据块的序号
	wsn = offset / int64(df.dataLen)
	var bytes []byte
	if len(d) > int(df.dataLen){
		bytes = d[0:df.dataLen]
	}else{
		bytes = d
	}
	df.fmutex.Lock()
	defer df.fmutex.Unlock()
	_, err = df.f.Write(bytes)
	//发送通知
	df.rcond.Signal()
	return
}

func (df *myDataFile) Rsn() int64{
	df.rmutex.Lock()
	defer df.rmutex.Unlock()
	return df.roffset / int64(df.dataLen)
}

func (df *myDataFile) Wsn() int64{
	df.wmutex.Lock()
	defer df.wmutex.Unlock()
	return df.woffset / int64(df.dataLen)
}

func (df *myDataFile) DataLen() uint32 {
	return df.dataLen
}

func main(){
	//简单测试下结果
	var dataFile DataFile
	dataFile,_ = NewDataFile("./mutex_2015_1.dat", 10)

	var d=map[int]Data{
		1:[]byte("batu_test1"),
		2:[]byte("batu_tstt2"),
		3:[]byte("batu_test3"),
	}


	//写入数据
	for i:= 1; i < 4; i++ {
		go func(i int){
			wsn,_ := dataFile.Write(d[i])
			fmt.Println("write i=", i,",wsn=",wsn, ",success.")
		}(i)
	}
    
    //读取数据
	for i:= 1; i < 4; i++ {
		go func(i int){
			rsn,d,_ := dataFile.Read()
			fmt.Println("Read i=", i,",rsn=",rsn,",data=",d, ",success.")
		}(i)
	}
    
	time.Sleep(10 * time.Second)
}
```



### 内核级线程/用户级线程

- 用户级线程

内核不知道进程内的线程情况，线程在用户空间中管理，每个进程内维护其线程表，记录进程内各个线程的程序计数器，堆栈指针，寄存器，状态等

优点：

> 当线程本地阻塞时，在用户空间内进行线程的切换，不需要陷入内核，线程切换速度快
>
> 允许每个进程有自己定制的调度算法

缺点：

> 发生系统调用时，操作系统由于对进程内的线程一无所知，则整个进程陷入了阻塞，此时即进程内某一线程发生的系统调用阻塞了整个进程

- 内核级线程

内核中维护一个线程表，操作系统负责内核级线程的调度(抢占式)

优点：

> 发生系统调用时，内核会检查是否有其他可以运行的线程进行切换，不会阻塞其他线程

缺点：

> 线程切换需要陷入内核，速度比用户级线程要慢

- 混合实现

使用内核线程的前提下，其中内核线程会被多个用户级线程复用，Go中goroutine就是使用这个设计思路



#### 扩展：Golang中的goroutine设计实现

首先，Go的调度所使用的模型是**内核级线程与用户级线程 M:N 的映射关系**

即多个 Goroutine (用户级线程)(协程) 在多个内核线程上跑，同时具有使用内核级线程和用户级线程的优点，但是其调度的难度也增加

Go中调度器重要的三个结构 

![img](https://pic2.zhimg.com/80/2f5c6ef32827fb4fc63c60f4f5314610_hd.jpg)

- M 真正的内核线程
- P 调度的上下文 一个局部的调度器，负责调度Goroutine 与 内核线程间的运行，是实现 多对多映射的关键
- G 表示Goroutine ，有自己的栈，以及用户级线程相关信息



下图中，每个内核线程都有一个正在运行的Goroutine，所有正在运行的Goroutine 时系统真正的并发度，由`GOMAXPROCS()`设置

![img](https://pic3.zhimg.com/80/67f09d490f69eec14c1824d939938e14_hd.jpg)

#### 某个内核线程在执行G的过程中执行了系统调用阻塞时处理？

此时M被内核调度器调度出CPU，陷入阻塞，此时Go的运行时系统的监控线程(sysmon线程)探测到这样的M, 将调度器P剥离，寻找其他空闲或者新建M接管调度器P，继续运行其中的G

等待原来被阻塞的线程M回复是，则需要找一个空闲的P继续执行原来的G，若此时系统没有空闲的P，则把G放到全局队列中等待执行



![img](https://user-gold-cdn.xitu.io/2017/10/10/a8c3f94708df85808b14c7f6b82d1c28?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)



### 进程/线程 状态

#### 进程（线程类似）



- 运行

  该时刻进程占用CPU 

- 就绪

  进程已获得除处理机外其他所需资源，等待分配处理机资源

- 阻塞

  进程等待某个事件的发生，在事件发生前无法运行

![img](https://upload-images.jianshu.io/upload_images/1485056-efde09b1217348ee.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/700)





### 死锁

- 定义：如果一个进程集合中的每个进程都在等待只能由该进程集合中的其他进程才能引发的事件，那么，该进程集合就是死锁的
- 死锁的四个必要条件：
  - 互斥条件 -- 资源要么可用，要么被某一进程独占
  - 占有且申请 -- 占有了某个资源的进程还能申请新的资源
  - 不可抢占 -- 被进程占有的资源只能等待该进程自行释放，其他进程不能强行抢占
  - 环路等待 -- 死锁发生时，系统中存在两个或以上的进程组成的环路，环路中每个进程都在等待下一个进程所占有的资源
- 处理死锁的四种策略
  - 忽略
  - 检测并恢复 使用专门的机构检测死锁的发生并在发生时破坏死锁
  - 避免死锁 动态分配资源，在资源分配过程中避免死锁  -- 银行家算法 -- 保证在资源分配过程中时刻处于安全状态
  - 预防死锁，在申请资源时遵循某种协议，破坏死锁四个必要条件中的一个或多个



> 扩展：
>
> - 两阶段锁协议
>
>   在第一阶段，进程试图对所有所需记录进行加锁，加锁成功后
>
>   第二阶段，完成更新然后释放锁
>
>   通常应用在数据库系统中
>
> - 活锁
>
>   在某些情况，进程发现不能获取所需要的下一个锁时，会尝试短暂释放自己所拥有的锁，然后再尝试，若此时有另一进程在相同时刻做相同的操作，（等待相同的时间），这样导致两个进程一直都无法使用资源
>
> - 饥饿
>
>   在某种资源分配策略下（短作业优先调度算法，一直有新的短作业加入，后面较长时间的作业一直得不到机会运行），有可能使得一些进程永远得不到服务
>
>   比如使用高响应比调度算法，动态优先级调度等..可以避免饥饿现象
>
> - 银行家算法
>
>   设计思想：当用户申请一组资源时，系统判断如果这些资源分出去，系统是否还处于安全状态，是则分出，否则不满足申请
>
>   系统情况：
>
>   - available 可用的各类资源
>
>   对考察的进程定义向量
>
>   - max 进程最多需要的资源
>   - allocation 已分配的每类资源
>   - need 由 max-allocation 求出
>
>   ​
>
>   开始进行资源分配，当某个进程需要分配的资源可被当前系统available满足时，将资源分配出去
>
>   检查系统是否处于安全状态(通过找到一个当前的安全序列) -- **检查是否存在一个安全序列，使得按照这个顺序分配资源，每次分配都能满足至少一个进程的最大资源需求，然后进程再释放资源，标记为finish，直至所有进程都标记为finish，这个顺序则称为安全序列**
>
>   是则可以继续分配
>



### 用户态 内核态 系统调用

当一个进程执行**系统调用**陷入内核时，我们称进程处于**内核态**，其他时候则称为**用户态**

用户态：可执行非特权指令

内核态：可执行全部质量

**用户态-->核心态的方式:**

- **系统调用**

  用户态主动要求切换到内核态的一种方式，用户态进程通过系统调用申请使用操作系统提供的服务程序完成工作

- 异常

- 外部设备中断




### 进程的地址空间(Linux)

- 代码段 -- 编译后的机器指令

  绝大多数Linux系统支持共享代码段

- 数据段 -- 所有程序变量，字符串，数字和其他数据

  - 符号起始块 BSS (未初始化数据)
  - 初始化数据 

- **栈段**

  存储局部，临时变量，在程序块开始时编译器分配内存，结束时编译器自动释放内存，存储函数返回指针

- **堆段**

  存储动态内存分配，分配和释放由程序员实现（C/C++中）

![img](http://hi.csdn.net/attachment/201110/4/0_1317730654069L.gif)



### 调度

调度方法一般用于进程或线程(内核线程)的CPU调度

- 先来先服务 FCFS

- 最短作业优先 SJF

- 最短剩余时间优先 SRTF （其实是抢占式的最短作业优先）

- 时间片轮转 RR 

- 优先级调度 

  在静态优先级下可能低优先级进程会产生饥饿现象

- 多级队列

  高优先级的进程运行后移到次高优先级的队列

  （越高优先级的队列运行一次所需要时间片越少(轮转快)）

- 彩票调度



### Linux下各种称呼的进程

在unix/linux 下 ，一般情况子进程是由父进程创建的，子进程的结束和父进程的运行是一个异步过程

- 孤儿进程

  孤儿进程因为在其父进程退出时被init进程(内核启动的进程，pid=1) 收养，由init进程对这些进程完成状态收集工作



​	通过两次fork生成孤儿进程,父进程在等待其儿子进程退出后即退出，由于等待了3秒，然后孙子进程的原父亲已经退出，所以此时该孙子进程的父亲变为init进程

```c
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>
#include <errno.h>

int main()
{
    pid_t  pid;
    //创建第一个子进程
    pid = fork();
    if (pid < 0)
    {
        perror("fork error:");
        exit(1);
    }
    //第一个子进程
    else if (pid == 0)
    {
        //子进程再创建子进程
        printf("I am the first child process.pid:%d\tppid:%d\n",getpid(),getppid());
        pid = fork();
        if (pid < 0)
        {
            perror("fork error:");
            exit(1);
        }
        //第一个子进程退出
        else if (pid >0)
        {
            printf("first procee is exited.\n");//孙子进程称为孤儿进程
            exit(0);
        }
        //第二个子进程
        //睡眠3s保证第一个子进程退出，这样第二个子进程的父亲就是init进程里
        sleep(3);
        printf("I am the second child process.pid: %d\tppid:%d\n",getpid(),getppid());
        exit(0);
    }
    //父进程处理第一个子进程退出
    if (waitpid(pid, NULL, 0) != pid)
    {
        perror("waitepid error:");
        exit(1);
    }
    exit(0);
    return 0;
}
```



- 僵尸进程

  僵尸进程是子进程执行完成 exit 之后，如果此时父进程不调用wait/waitpid 获取子进程exit 的状态，那么子进程的进程控制块还存在在进程表中，且占用着进程号，这种进程称为僵尸进程

  当父进程通过这种方法一直产生子进程后，最终可能导致系统没有可用的进程号，从而没有新的进程产生

  僵尸进程在可以通过命令 `ps` 查看 状态为 Z(defunct)的进程

  ​

  示例代码：

  ```c
  #include <stdio.h>
  #include <unistd.h>
  #include <errno.h>
  #include <stdlib.h>
  #include <signal.h>
  #include <sys/types.h>    //NOTE: add this to avoid pid_t undefied
  static void sig_child(int signo);

  int main()
  {
      pid_t pid;
      //创建捕捉子进程退出信号
      signal(SIGCHLD,sig_child);
      pid = fork();
      if (pid < 0)
      {
          perror("fork error:");
          exit(1);
      }
      else if (pid == 0)
      {
          printf("I am child process,pid id %d.I am exiting.\n",getpid());
          exit(0);
      }
      printf("I am father process.I will sleep two seconds\n");
      //等待子进程先退出
      sleep(2);
      //输出进程信息
      system("ps -o pid,ppid,state,tty,command");
      printf("father process is exiting.\n");
      return 0;
  }

  static void sig_child(int signo)
  {
       pid_t        pid;
       int        stat;
       //处理僵尸进程
       while ((pid = waitpid(-1, &stat, WNOHANG)) >0)
              printf("child %d terminated.\n", pid);
  }
  ```

  ​

  ```
  # 以下 23808为父进程 23809为僵尸进程
   PID  PPID S TT       COMMAND
  22965 22946 S pts/2    /bin/bash
  23808 22965 S pts/2    ./a.out
  23809 23808 Z pts/2    [a.out] <defunct>
  23810 23808 S pts/2    sh -c ps -o pid,ppid,state,tty,command
  23811 23810 R pts/2    ps -o pid,ppid,state,tty,command
  ```

- 守护进程

守护进程(daemon)，是一种运行在后台 的特殊进程，它独立与控制终端 ，并 周期性地执行某项任务或等待处理某些发生的事件。 。守护进程可以由一个普通的进程按照守护进程的特性改造而来。

![è¿éåå¾çæè¿°](http://img.blog.csdn.net/20170826190719713?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvdTAxMzYxNjk0NQ==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)



>[僵尸进程 WIKI](https://en.wikipedia.org/wiki/Zombie_process)
>
>[僵尸进程/孤儿进程/守护进程](https://www.waitig.com/%E9%9D%A2%E8%AF%95%E6%80%BB%E7%BB%93%E4%B9%8B%E5%AE%88%E6%8A%A4%E8%BF%9B%E7%A8%8B%EF%BC%8C%E5%83%B5%E5%B0%B8%E8%BF%9B%E7%A8%8B%E5%92%8C%E5%AD%A4%E5%84%BF%E8%BF%9B%E7%A8%8B.html)



### fork 进程

当进程调用fork后，进程陷入内核，执行

- 分配新的内存块和内核数据给子进程
- 将父进程部分数据结构内容拷贝至子进程
- 添加子进程到进程表中
- fork返回(从内核空间返回用户空间)，进行进程调度

从fork返回后，父子进程共享进程的代码，并且都从fork函数调用之后运行



### 中断

在计算机执行程序过程中，由于发生某些事件，使CPU暂停当前程序的执行，转而去执行处理这一事件的程序，等这些事件处理完之后再回到执行之前的程序恢复现场

中断过程：

- 中断响应
  - 中止当前程序的执行
  - 保存原程序断点信息(PC,寄存器...)
  - 转到相应中断的处理程序
- 中断处理
  - 保存现场 -- 保存中断程序的现场
  - 分析原因 -- 分析中断原因
  - 处理中断 -- 调用相应的中断处理程序
  - 中断返回 -- 回到原来执行的进程，恢复现场

中断类型：

- 内部异常中断

- 软中断

  程序中执行了引起中断的指令，

- 外部中断

  因外部设备请求触发



