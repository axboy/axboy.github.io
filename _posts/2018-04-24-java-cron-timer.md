---
layout: post
title: 定时任务--timer详解
description: 定时任务--timer详解
categories: java
tags: java
---

Timer可实现定时任务的调度，用一个后台线程执行指定任务一次或多次。 Timer主要涉及两个类，TimerTask, Timer。

### demo演示

首先先来一个demo来演示timer的使用。[源码地址](https://github.com/axboy/zcw.learn/tree/master/java/cron)

- 自定义任务

```java
//简单任务，输出当前时间
public class MySimpleTimerTask extends TimerTask {
    private String taskName;

    public MySimpleTimerTask(String taskName) {
        this.taskName = taskName;
    }
    public String getTaskName() {
        return taskName;
    }
    public void setTaskName(String taskName) {
        this.taskName = taskName;
    }
    @Override
    public void run() {
        System.out.printf("%s, %s\n", this.taskName, new Date());
    }
}
```

- Main函数

```java
//输出当前时间
System.out.printf("Now time: %s\n", new Date());
Timer timer = new Timer();

MySimpleTimerTask simpleTask = new MySimpleTimerTask("Simple Task");

//将任务添加进任务队列，并延时2s执行，之后每3s执行一次
timer.schedule(simpleTask, 2000L, 3000L);
```

### timer类方法详解

- 先敬上源码

```java
public void schedule(TimerTask task, long delay){
    if (delay < 0)
            throw new IllegalArgumentException("Negative delay.");
        sched(task, System.currentTimeMillis()+delay, 0);
}
public void schedule(TimerTask task, Date time) {
    sched(task, time.getTime(), 0);
}
public void schedule(TimerTask task, long delay, long period) {
    if (delay < 0)
        throw new IllegalArgumentException("Negative delay.");
    if (period <= 0)
        throw new IllegalArgumentException("Non-positive period.");
    sched(task, System.currentTimeMillis()+delay, -period);
}
public void schedule(TimerTask task, Date firstTime, long period) {
    if (period <= 0)
        throw new IllegalArgumentException("Non-positive period.");
    sched(task, firstTime.getTime(), -period);
}
public void scheduleAtFixedRate(TimerTask task, long delay, long period) {
    if (delay < 0)
        throw new IllegalArgumentException("Negative delay.");
    if (period <= 0)
        throw new IllegalArgumentException("Non-positive period.");
    sched(task, System.currentTimeMillis()+delay, period);
}
public void scheduleAtFixedRate(TimerTask task, Date firstTime, long period) {
    if (period <= 0)
        throw new IllegalArgumentException("Non-positive period.");
    sched(task, firstTime.getTime(), period);
}
```

可以发现，每个方法最后都调用sched()方法，参数稍微转换了。唯一差别就是schedule(...)的period传入的负数，
scheduleAtFixedRate(...)的period传入的正数，按方法命名来看，scheduleAtFixedRate(...)为固定频率执行。
接下来汇报我的测试结果。

- 代码测试结果 [源码地址](https://github.com/axboy/zcw.learn/tree/master/java/cron)

    - schedule(TimerTask task, long delay, long period)

    指定的任务将在delay毫秒后首次执行，若不传入period，则只执行一次，否则，间隔period毫秒循环执行。
    但是，若任务比较复杂，执行时间超过period，则会连续执行，相当于period为任务执行时间。

    - schedule(TimerTask task, Date firstTime, long period)

    指定任务在firstTime时执行，若当前时间超过firstTime，则会立即执行，之后间隔period毫秒执行，任务时间超过周期时间则连续执行。

    - scheduleAtFixedRate(TimerTask task, long delay, long period)

    同schedule(TimerTask task, long delay, long period)

    - scheduleAtFixedRate(TimerTask task, Date firstTime, long period)

    当前时间未超过firstTime的话，效果和schedule(...)的同参方法一样。

    若设定的时间早于当前时间，scheduleAtFixedRate(...)方法会立即执行，之后连续执行几次把从指定时间的到现在缺少的执行次数补上，补齐之后按firstTime所对应的时间周期执行。任务执行时间比周期时间长，当然是连续执行，相当于一直在把之前缺少的次数补上。

### 分析

不难发现，sched(...)方法的period参数。
若为0，则任务只执行一次；
若小于0，则按周期执行，时间早于当前时间，则立即执行，不把之前缺少的补上；
若大于0，按周期执行，时间早于当前时间，连续执行多次把之前缺少的次数补上，之后按预定义的第一次和周期执行；
任务耗时比周期长的，都是连续执行。

查看源码发现sched(...)方法并不重要，这里就不列出了。

首先来看一下Timer类

```java
public class Timer{
    //数组实现的任务队列，sched(...)方法就是把任务添加到该队列中，这里不展开。
    private final TaskQueue queue = new TaskQueue();
    //调度器的后台的主线程，下文展开源码。
    private final TimerThread thread = new TimerThread(queue);

    //省略构造函数
    //省略上文提到的schedule(...)、scheduleAtFixedRate(...)和sched(...)方法

    //取消所有任务
    public void cancel(){...}

    //移除所有已完成的任务，并返回移除任务数
    public int purge(){...}
}
```

重点来了

```java
class TimerThread extends Thread {
    //标记线程状态
    boolean newTasksMayBeScheduled = true;
    //Timer中的任务队列，构造函数传入
    private TaskQueue queue;
    //构造函数
    TimerThread(TaskQueue queue) {
        this.queue = queue;
    }
    //实现Thread的run()方法
    public void run() {
        try {
            mainLoop();
        } finally {
            // Someone killed this Thread, behave as if Timer cancelled
            synchronized(queue) {
                newTasksMayBeScheduled = false;
                queue.clear();  // Eliminate obsolete references
            }
        }
    }
    //主循环
    private void mainLoop() {
        while (true) {
            try {
                TimerTask task;
                boolean taskFired;
                synchronized(queue) {
                    // Wait for queue to become non-empty
                    while (queue.isEmpty() && newTasksMayBeScheduled)
                        queue.wait();
                    if (queue.isEmpty())
                        break; // Queue is empty and will forever remain; die

                    // Queue nonempty; look at first evt and do the right thing
                    long currentTime, executionTime;
                    task = queue.getMin();          //取得一个任务
                    synchronized(task.lock) {
                        //若任务已完成，则移除并进入下重循环
                        if (task.state == TimerTask.CANCELLED) {
                            queue.removeMin();
                            continue;  // No action required, poll queue again
                        }
                        currentTime = System.currentTimeMillis();
                        executionTime = task.nextExecutionTime;

                        //已过任务计划执行时间
                        if (taskFired = (executionTime<=currentTime)) {
                            //period为0，非周期任务
                            if (task.period == 0) { // Non-repeating, remove
                                queue.removeMin();
                                task.state = TimerTask.EXECUTED;
                            } else { // Repeating task, reschedule
                                //period < 0, 当前时间 + 周期时间
                                //period > 0, 计划时间 + 周期时间
                                //这里就是schedule(...) 和 scheduleAtFixedRate(...)方法的区别了
                                queue.rescheduleMin(
                                  task.period<0 ? currentTime   - task.period
                                                : executionTime + task.period);
                            }
                        }
                    }
                    //未到当前任务的执行时间，等待
                    if (!taskFired) // Task hasn't yet fired; wait
                        queue.wait(executionTime - currentTime);
                }
                if (taskFired)  // Task fired; run it, holding no locks
                    task.run();
            } catch(InterruptedException e) {
            }
        }
    }
}
```

- 最后的最后

通过源码发现，Timer和TimerTask并不是多线程的，是在内部封装一层Thread和一个Task队列，无限循环队列，当时间符合条件则执行任务。
若队列为空或者cancel操作，从而跳出死循环。
多线程如何实现呢，一个Timer是一个线程，定义多个Timer即可。