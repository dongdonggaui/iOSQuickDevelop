# Concurrency Programming Guide

[文档地址](https://developer.apple.com/library/ios/documentation/General/Conceptual/ConcurrencyProgrammingGuide)

## Introduction

## Concurrency and Application Design

### The Move Away from Threads

#### Dispatch Queues

#### Dispatch Sources

#### Operation Queues

### Asynchronous Design Techniques

#### Define Your Application's Exp...

### Performance Implications

### Concurrency and Other Technology

## Operation Queues

## Dispatch Queues

### About Dispatch Queues

> You can create as many serial queues as you need, and each queue operates concurrently with respect to all other queues. In other words, if you create four serial queues, each queue executes only one task at a time but up to four tasks could still execute concurrently, one from each queue

### Queue-Related Technologies

Dispatch semaphores 介绍：

> A dispatch semaphore is similar to a traditional semaphore but is generally more efficient.

> Dispatch semaphores call down to the kernel only when the calling thread needs to be blocked because the semaphore is unavailable. If the semaphore is available, no kernel call is made.

百度百科补充：

> Semaphore,是负责协调各个线程, 以保证它们能够正确、合理的使用公共资源，也是操作系统中用于控制进程同步互斥的量。

> Semaphore分为单值和多值两种，前者只能被一个线程获得，后者可以被若干个线程获得。

> 以一个停车场是运作为例。为了简单起见，假设停车场只有三个车位，一开始三个车位都是空的。这时如果同时来了五辆车，看门人允许其中三辆不受阻碍的进入，然后放下车拦，剩下的车则必须在入口等待，此后来的车也都不得不在入口处等待。这时，有一辆车离开停车场，看门人得知后，打开车拦，放入一辆，如果又离开两辆，则又可以放入两辆，如此往复。

> 在这个停车场系统中，车位是公共资源，每辆车好比一个线程，看门人起的就是信号量的作用。

> 更进一步，信号量的特性如下：信号量是一个非负整数（车位数），所有通过它的线程（车辆）都会将该整数减一（通过它当然是为了使用资源），当该整数值为零时，所有试图通过它的线程都将处于等待状态。在信号量上我们定义两种操作： Wait（等待） 和 Release（释放）。 当一个线程调用Wait（等待）操作时，它要么通过然后将信号量减一，要么一直等下去，直到信号量大于一或超时。Release（释放）实际上是在信号量上执行加操作，对应于车辆离开停车场，该操作之所以叫做“释放”是因为加操作实际上是释放了由信号量守护的资源。

### Implementing Tasks Using Blocks

### Creating and Managing Dispatch Queues

#### Getting the Global Concurrent Dispatch Queues

#### Creating Serial Dispatch Queues

主要用途：

> Serial queues are useful when you want your tasks to execute in a specific order.You might use a serial queue instead of a lock to protect a shared resource or mutable data structure. 

优点：

> Unlike a lock, a serial queue ensures that tasks are executed in a predictable order. And as long as you submit your tasks to a serial queue asynchronously, the queue can never deadlock. 

创建方式：

> Unlike concurrent queues, which are created for you, you must explicitly create and manage any serial queues you want to use.

避免：

> avoid creating large numbers of serial queues solely as a means to execute as many tasks simultaneously as you can. If you want to execute large numbers of tasks concurrently, submit them to one of the global concurrent queues. 

用途须明确：

> When creating serial queues, try to identify a purpose for each queue, such as protecting a resource or synchronizing some key behavior of your application.

创建方法参数说明：

> The `dispatch_queue_create` function takes two parameters: **the queue name** and **a set of queue attributes**.

> The debugger and performance tools display the queue name to help you track how your tasks are being executed. 

> The queue attributes are reserved for future use and should be NULL.

创建例子：

~~~objc
dispatch_queue_t queue;
queue = dispatch_queue_create("com.example.MyQueue", NULL);
~~~

主线程顺序队列：

> the system automatically creates a serial queue and binds it to your application’s main thread.

### Adding Tasks to a Queue

#### Adding a Single Task to a Queue

两种添加方法：

> There are two ways to add a task to a queue: asynchronously or synchronously.

优先使用异步分发：

> When possible, asynchronous execution using the `dispatch_async` and `dispatch_async_f` functions is preferred over the synchronous alternative.

优先异步分发的原因：

> When you add a block object or function to a queue, there is no way to know when that code will execute. As a result, adding blocks or functions asynchronously lets you schedule the execution of the code and continue to do other work from the calling thread. This is especially important if you are scheduling the task from your application’s main thread—perhaps in response to some user event.

使用同步分发：

> when you need to add a task synchronously to prevent race conditions or other synchronization errors.

> In these instances, you can use the `dispatch_sync` and `dispatch_sync_f` functions to add the task to the queue. These functions block the current thread of execution until the specified task finishes executing.

避免分发任务到分发此任务的队列：

> You should never call the `dispatch_sync` or `dispatch_sync_f` function from a task that is executing in the same queue that you are planning to pass to the function. This is particularly important for serial queues, which are guaranteed to deadlock, but should also be avoided for concurrent queues.

### Using Dispatch Semaphores to Regulate the Use of Finite Resources

应用场景：

> If the tasks you are submitting to dispatch queues access some finite resource, you may want to use a dispatch semaphore to regulate the number of tasks simultaneously accessing that resource. 

对系统调用的优化：

> A dispatch semaphore works like a regular semaphore with one exception. When resources are available, it takes less time to acquire a dispatch semaphore than it does to acquire a traditional system semaphore.

> This is because Grand Central Dispatch does not call down into the kernel for this particular case. The only time it calls down into the kernel is when the resource is not available and the system needs to park your thread until the semaphore is signaled.

使用步骤：

> 1. When you create the semaphore (using the `dispatch_semaphore_create` function), you can specify a positive integer indicating the number of resources available.
> 2. In each task, call `dispatch_semaphore_wait` to wait on the semaphore.
> 3. When the wait call returns, acquire the resource and do your work.
> 4. When you are done with the resource, release it and signal the semaphore by calling the `dispatch_semaphore_signal` function.

简要工作原理：

> When you create the semaphore, you specify the number of available resources. This value becomes the initial count variable for the semaphore. 

> Each time you wait on the semaphore, the `dispatch_semaphore_wait` function decrements that count variable by 1. If the resulting value is negative, the function tells the kernel to block your thread. 

> On the other end, the `dispatch_semaphore_signal` function increments the count variable by 1 to indicate that a resource has been freed up. If there are tasks blocked and waiting for a resource, one of them is subsequently unblocked and allowed to do its work.

使用例子：

> consider the use of file descriptors on the system. Each application is given a limited number of file descriptors to use. If you have a task that processes large numbers of files, you do not want to open so many files at one time that you run out of file descriptors. Instead, you can use a semaphore to limit the number of file descriptors in use at any one time by your file-processing code.

~~~objc
// Create the semaphore, specifying the initial pool size
dispatch_semaphore_t fd_sema = dispatch_semaphore_create(getdtablesize() / 2);
 
// Wait for a free file descriptor
dispatch_semaphore_wait(fd_sema, DISPATCH_TIME_FOREVER);
fd = open("/etc/services", O_RDONLY);
 
// Release the file descriptor when done
close(fd);
dispatch_semaphore_signal(fd_sema);
~~~