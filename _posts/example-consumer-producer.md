---
name: consumer-producer
title: 生产者-消费者(有界缓冲区)
date: 2025-05-16
categories: 设计模式
---

```java
class BoundedBuffer<E> {
    final ReentrantLock lock = new ReentrantLock();
    final Condition notFull  = lock.newCondition(); 
    final Condition notEmpty = lock.newCondition(); 
 
    final Object[] items = new Object[100];
    int putptr, takeptr, count;
 
    public void put(E x) throws InterruptedException {
      lock.lock();
      try {
        while (count == items.length)
          notFull.await();
        items[putptr] = x;
        if (++putptr == items.length) putptr = 0;
        ++count;
        notEmpty.signal();
      } finally {
        lock.unlock();
      }
    }
 
    public E take() throws InterruptedException {
      lock.lock();
      try {
        while (count == 0)
          notEmpty.await();
        E x = (E) items[takeptr];
        if (++takeptr == items.length) takeptr = 0;
        --count;
        notFull.signal();
        return x;
      } finally {
        lock.unlock();
      }
    }
}
```

