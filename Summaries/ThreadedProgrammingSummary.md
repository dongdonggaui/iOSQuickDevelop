# Threaded Programming

## Synchronization

### Synchronization Tools

#### Atomic Operations

介绍：

> Atomic operations are a simple form of synchronization that work on simple data types. 

优势：

> The advantage of atomic operations is that they do not block competing threads. For simple operations, such as incrementing a counter variable, this can lead to much better performance than taking a lock.

系统提供的原子操作：

> OS X and iOS include numerous operations to perform basic mathematical and logical operations on 32-bit and 64-bit values. Among these operations are atomic versions of the compare-and-swap, test-and-set, and test-and-clear operations.

### Using Atomic Operations

尽量避免使用锁：

> Nonblocking synchronization is a way to perform some types of operations and avoid the expense of locks. Although locks are an effective way to synchronize two threads, acquiring a lock is a relatively expensive operation, even in the uncontested case. 

更有效率的“锁”：

> many atomic operations take a fraction of the time to complete and can be just as effective as a lock.

原理：

> These operations rely on special hardware instructions (and an optional memory barrier) to ensure that the given operation completes before the affected memory is accessed again. 

建议：

> In the multithreaded case, you should always use the atomic operations that incorporate a memory barrier to ensure that the memory is synchronized correctly between threads.