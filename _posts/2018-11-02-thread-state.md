---
layout: post
title: 线程和进程
date: 2018-11-02
tags: [thread]
---

看到一个关于线程状态的问题：

> 有一个4*50的二维数组，用4个线程去分5个阶段去填满它，也就说，第一阶段大家一起填0-9，当大家都填满了0-9，再一起去填10-19，以此类推，先填满的线程要等着其他线程都填好了，再继续。

## 线程和进程
进程就是一个程序的执行流程，内部保存程序运行所需的资源,所以是系统资源调度和分配的最小单位。
线程是系统分配处理器时间的基本单元，或者说进程之内独立执行的一个单元执行流。是程序执行的最小单位。

## 线程的状态
```java
public enum State {
    /**
     * Thread state for a thread which has not yet started.
     */
    NEW,

    /**
     * Thread state for a runnable thread.  A thread in the runnable
     * state is executing in the Java virtual machine but it may
     * be waiting for other resources from the operating system
     * such as processor.
     */
    RUNNABLE,

    /**
     * Thread state for a thread blocked waiting for a monitor lock.
     * A thread in the blocked state is waiting for a monitor lock
     * to enter a synchronized block/method or
     * reenter a synchronized block/method after calling
     * {@link Object#wait() Object.wait}.
     */
    BLOCKED,

    /**
     * Thread state for a waiting thread.
     * A thread is in the waiting state due to calling one of the
     * following methods:
     * <ul>
     *   <li>{@link Object#wait() Object.wait} with no timeout</li>
     *   <li>{@link #join() Thread.join} with no timeout</li>
     *   <li>{@link LockSupport#park() LockSupport.park}</li>
     * </ul>
     *
     * <p>A thread in the waiting state is waiting for another thread to
     * perform a particular action.
     *
     * For example, a thread that has called <tt>Object.wait()</tt>
     * on an object is waiting for another thread to call
     * <tt>Object.notify()</tt> or <tt>Object.notifyAll()</tt> on
     * that object. A thread that has called <tt>Thread.join()</tt>
     * is waiting for a specified thread to terminate.
     */
    WAITING,

    /**
     * Thread state for a waiting thread with a specified waiting time.
     * A thread is in the timed waiting state due to calling one of
     * the following methods with a specified positive waiting time:
     * <ul>
     *   <li>{@link #sleep Thread.sleep}</li>
     *   <li>{@link Object#wait(long) Object.wait} with timeout</li>
     *   <li>{@link #join(long) Thread.join} with timeout</li>
     *   <li>{@link LockSupport#parkNanos LockSupport.parkNanos}</li>
     *   <li>{@link LockSupport#parkUntil LockSupport.parkUntil}</li>
     * </ul>
     */
    TIMED_WAITING,

    /**
     * Thread state for a terminated thread.
     * The thread has completed execution.
     */
    TERMINATED;
}
```
## 状态间变化
![policy_state_machine](https://raw.githubusercontent.com/BeanHaHa/BeanHaHa.github.io/master/assets/images/2018/thread_state.jpg)

## 项目
[Github地址](https://github.com/BeanHaHa/day-day-up/blob/main/src/main/java/com/daobin/learning/daydayup/thread/ThreadState.java)
