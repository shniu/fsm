# 万能钥匙：管程

管程是一种并发访问共享资源的方式，在并发编程领域应用广泛。

## 信号量和管程

最先引入并发概念的软件是操作系统，OS 的出现让计算机进入了一个前所未有的时代，而多任务管理让计算机更进一步；多任务管理的OS中，最大的价值体现是多个任务之间相互配合完成复杂的事情，这势必会出现多个任务访问同一个资源的情况出现，当某个任务访问完了共享资源又如何通知其他的任务来执行呢？这里就引出了并发编程的两个核心问题：互斥和同步。

### 信号量模型

信号量是最先提出的模型，用来解决多线程并发访问共享资源的互斥和同步问题。基本模型如下：

1. 声明一个共享资源的数量 `sem`
2. 提供一个 P 操作可以对 `sem` 进行 `-1` 操作，当 `sem < 0` 时，当前线程进入阻塞等待状态
3. 提供一个 V 操作对 `sem` 进行 `+1` 操作，当 `sem < 0` 时，唤醒一个线程

其中 p 操作和 v 操作都是原语，原语是完成某种功能且不被分割不被中断执行的操作序列，是可由硬件实现的原子操作。信号量的实现模型：

```c
// 信号量数据模型
// 共享资源数和等待资源的进程链表
typedef struct {
  int value;  // 共享资源数目
  struct process * L;  // 进程链表，记录等待该资源的进程
} semaphore;

// 信号量的封装操作
// 对信号量数据模型的两个操作
void wait(semaphore S) {  // 这个就是 P 操作
  S.value--;
  if (S.value < 0) {
    add this process to S.L
    block(S.L);  // block 是阻塞原语，线程或进程进入阻塞状态，由os和硬件配合实现
  }
}

void signal(semaphore S) { // 这个就是 V 操作
  S.value++;
  if (S.value < 0) {
    remove this process from S.L
    wakeup(S.L);  // wakeup 是线程唤醒原语
  }
}
```

以上配合起来就可以解决同步和互斥的问题。

- 解决互斥问题

```c
// 解决互斥问题，可以初始信号量为 1
semaphore S = 1;

P1() {
    // ...
    P(S);
    临界区代码
    V(S);
    // ...
}


P2() {
    // ...
    P(S);
    临界区代码
    V(S);
    // ...
}
```

可见，P1 和 P2 竞争 S，只有通过 P 操作的进程方可执行，另一个进程需要等待，这就实现了互斥执行。

- 解决同步问题

```c
// 解决同步问题，可以初始信号量为 0
semaphore S = 0;

P1() {
  // ...
  x; // 做一些事情
  V(S); // signal(S), wakeup 一个线程
}

P2() {
  // ...
  P(S); // wait(S); 等待资源可访问
  y; // 做一些自己的事情
}
```

可见 P1 执行 sgnal 来通知 P2 可以继续运行了，P2 一直阻塞在 P(S) 处。


信号量的缺点是：

1. p操作和v操作必须成对出现，不正确的使用容易出现bug
2. pv 操作大量分散在各个进程中，不易管理，易发生死锁

### 管程模型

信号量已经很好的解决了同步和互斥的问题，但是为什么还要引入管程呢？从上面分析中我们直到信号量有一些缺点，所以引入管程的目的是：

1、把分散在各进程中的临界区集中起来进行管理；
2、防止进程有意或无意的违法同步操作；
3、便于用高级语言来书写程序，也便于程序正确性验证

其实，简单的理解一下就是管程比信号量封装性更好，更容易编程和管理。总结一下，管程对共享资源和对共享资源的操作进行封装，让它来支持并发，和OO的思想很接近啊。具体是如何使用和如何实现的呢？

管程的基本模型是：

1. 一组共享







