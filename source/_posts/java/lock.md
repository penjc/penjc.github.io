---
title: Java 中的锁机制
comment: true
date: 2025-01-17
tags:
  - JAVA
  - Lock
categories:
  - [JAVA]
---


在 Java 中，锁是一种机制，用于协调多个线程对共享资源的访问，防止数据的不一致和竞态条件的发生。Java 提供了多种锁机制来实现同步，这些机制分为基本的 `synchronized` 锁和更高级的锁（如 `ReentrantLock`）。下面，我将详细解释 Java 中的锁机制及其应用场景。

## **1. Java 中的基本锁机制：`synchronized`**

### **1.1 `synchronized` 关键字**

`synchronized` 是 Java 提供的最基本的锁机制，用于确保多个线程在同一时间只能有一个线程访问某个代码块或方法，从而实现线程安全。

`synchronized` 可以用在两种地方：
- **方法**（同步方法）
- **代码块**（同步代码块）

#### **（1）同步方法**

```java
public synchronized void increment() {
    // 只有一个线程可以访问该方法
    counter++;
}
```
- 当一个线程进入 `increment()` 方法时，其他线程必须等待，直到当前线程离开该方法。
- 锁是当前对象（`this`）。

#### **（2）同步代码块**

```java
public void increment() {
    synchronized (this) {
        // 只有一个线程可以访问该代码块
        counter++;
    }
}
```
- 使用 `synchronized(this)`，锁定的是当前对象。
- 这样可以让我们对更小的代码块进行同步，而不是整个方法，提高并发性。

#### **（3）静态同步方法**

```java
public static synchronized void staticMethod() {
    // 锁定的是类对象
}
```
- 如果方法是静态的，那么锁定的是该类的类对象（`Class` 对象），而不是实例。

### **1.2 锁的细节**

- **可重入性**：`synchronized` 是可重入的，意味着一个线程可以多次获得同一把锁。
- **隐式锁释放**：当线程执行完同步代码块或方法后，锁会自动释放。

#### **缺点**
- 可能导致线程等待过长时间（即 **线程饥饿**）。
- 不支持超时尝试获取锁。
- 锁定的范围不能灵活控制。

---

## **2. Java 中的高级锁机制：`java.util.concurrent.locks`**

Java 5 引入了 `java.util.concurrent.locks` 包，提供了更灵活的锁机制。最常用的是 `ReentrantLock`。

### **2.1 `ReentrantLock`**

`ReentrantLock` 是 `Lock` 接口的一个实现，功能类似于 `synchronized`，但提供了更多的锁定机制。

#### **（1）基本用法**

```java
import java.util.concurrent.locks.Lock;
import java.util.concurrent.locks.ReentrantLock;

public class Counter {
    private final Lock lock = new ReentrantLock();
    private int counter = 0;

    public void increment() {
        lock.lock(); // 获取锁
        try {
            counter++;
        } finally {
            lock.unlock(); // 释放锁
        }
    }
}
```
- `lock.lock()`：获取锁。
- `lock.unlock()`：释放锁。在 `finally` 块中确保锁总是被释放。

#### **（2）可重入性**

`ReentrantLock` 是 **可重入的**，类似于 `synchronized`，同一个线程可以多次获得同一把锁。

#### **（3）可中断的锁**

`ReentrantLock` 支持线程在获取锁时中断，这可以避免线程长时间等待：
```java
lock.lockInterruptibly();
```

#### **（4）尝试获取锁**

可以尝试在指定时间内获取锁：
```java
if (lock.tryLock(1, TimeUnit.SECONDS)) {
    try {
        // 处理逻辑
    } finally {
        lock.unlock();
    }
}
```
- `tryLock(long time, TimeUnit unit)`：在给定时间内尝试获取锁。

#### **（5）公平锁与非公平锁**

- **公平锁**：按照线程请求锁的顺序分配锁。
- **非公平锁**：可能会让某些线程先获取锁，即使其他线程已经在等待。

默认情况下，`ReentrantLock` 是非公平的，但可以通过构造函数设置为公平锁：
```java
Lock lock = new ReentrantLock(true); // true 表示公平锁
```

### **2.2 `ReadWriteLock`**

`ReadWriteLock` 允许多个线程同时读取资源，但在写资源时必须独占。这对于 **读多写少** 的场景可以显著提高性能。

```java
import java.util.concurrent.locks.ReadWriteLock;
import java.util.concurrent.locks.ReentrantReadWriteLock;

public class DataContainer {
    private final ReadWriteLock rwLock = new ReentrantReadWriteLock();
    private int data = 0;

    public void read() {
        rwLock.readLock().lock();
        try {
            System.out.println("Reading data: " + data);
        } finally {
            rwLock.readLock().unlock();
        }
    }

    public void write(int value) {
        rwLock.writeLock().lock();
        try {
            data = value;
        } finally {
            rwLock.writeLock().unlock();
        }
    }
}
```
- `readLock()`：获取读锁，多个线程可以同时获取。
- `writeLock()`：获取写锁，只能由一个线程获取，且读写是互斥的。

### **2.3 `StampedLock`**

`StampedLock` 是 Java 8 引入的一种改进的锁，适用于读多写少的场景。与 `ReadWriteLock` 相比，它提供了更高的吞吐量。

```java
import java.util.concurrent.locks.StampedLock;

public class Point {
    private double x, y;
    private final StampedLock lock = new StampedLock();

    public void move(double deltaX, double deltaY) {
        long stamp = lock.writeLock();
        try {
            x += deltaX;
            y += deltaY;
        } finally {
            lock.unlockWrite(stamp);
        }
    }

    public double distanceFromOrigin() {
        long stamp = lock.tryOptimisticRead();
        double currentX = x, currentY = y;
        if (!lock.validate(stamp)) {
            stamp = lock.readLock();
            try {
                currentX = x;
                currentY = y;
            } finally {
                lock.unlockRead(stamp);
            }
        }
        return Math.sqrt(currentX * currentX + currentY * currentY);
    }
}
```
- **乐观读锁**：`tryOptimisticRead()`，在数据没有被修改的情况下，可以直接读取。
- **写锁**：`writeLock()`，需要排他访问。

---

## **3. 条件变量：`Condition`**

`Condition` 接口允许线程在某个条件不满足的情况下等待，并且在条件满足后得到通知。它是 `Object.wait()` 和 `Object.notify()` 的更灵活替代。

```java
import java.util.concurrent.locks.Condition;
import java.util.concurrent.locks.Lock;
import java.util.concurrent.locks.ReentrantLock;

public class BoundedBuffer {
    private final Lock lock = new ReentrantLock();
    private final Condition notFull = lock.newCondition();
    private final Condition notEmpty = lock.newCondition();
    private final Object[] items = new Object[10];
    private int putIndex, takeIndex, count;

    public void put(Object x) throws InterruptedException {
        lock.lock();
        try {
            while (count == items.length) {
                notFull.await();
            }
            items[putIndex] = x;
            if (++putIndex == items.length) putIndex = 0;
            ++count;
            notEmpty.signal();
        } finally {
            lock.unlock();
        }
    }

    public Object take() throws InterruptedException {
        lock.lock();
        try {
            while (count == 0) {
                notEmpty.await();
            }
            Object x = items[takeIndex];
            if (++takeIndex == items.length) takeIndex = 0;
            --count;
            notFull.signal();
            return x;
        } finally {
            lock.unlock();
        }
    }
}
```
- **`await()`**：线程等待条件。
- **`signal()`**：通知等待的线程条件满足。

---

## **总结**

- **`synchronized`** 是 Java 中的基本锁机制，简单易用，适合大多数简单的同步场景。
- **`ReentrantLock`** 提供了更高级的锁功能，如可中断锁、超时锁、公平锁。
- **`ReadWriteLock`** 和 **`StampedLock`** 适合读多写少的场景。
- **`Condition`** 允许实现更复杂的线程间通信，代替传统的 `wait/notify` 机制。

根据具体场景选择合适的锁机制，可以提高并发程序的效率和可靠性。