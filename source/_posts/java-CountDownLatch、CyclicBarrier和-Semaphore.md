title: java -- CountDownLatch、CyclicBarrier和 Semaphore
author: Kenzhaoyihui
tags:
  - JAVA
categories: []
date: 2018-07-22 19:32:00
---
## CountDownLatch

CountDownLatch 在java.util.concurrent包中， 可以用它实现类似于计数器的功能。实现得是等待其他任务执行完毕之后才能继续执行主任务。

CountDownLatch类只提供了一个构造器：
```
public CountDownLatch(int count) {
        if (count < 0) throw new IllegalArgumentException("count < 0");
        this.sync = new Sync(count);
    }
```

然后下面这3个方法是CountDownLatch类中最重要的方法：
```
public void await() throws InterruptedException { };   
//调用await()方法的线程会被挂起，它会等待直到count值为0才继续执行

public boolean await(long timeout, TimeUnit unit) throws InterruptedException { };  
//和await()类似，只不过等待一定的时间后count值还没变为0的话就会继续执行

public void countDown() { }; 
//将count值减1
```

Code:
```
import java.util.concurrent.CountDownLatch;

public class Countdownlatch {
    public static void main(String[] args) {
        final CountDownLatch latch = new CountDownLatch(2);
        new Thread() {
            public void run() {
                try {
                    System.out.println("Child thread: " + Thread.currentThread().getName() + " is working");
                    Thread.sleep(3000);
                    System.out.println("Child thread: " + Thread.currentThread().getName() + " is finished");
                    latch.countDown();
                }catch (InterruptedException e) {
                    e.printStackTrace();
                }
            };
        }.start();

        new Thread() {
            public void run() {
                try {
                    System.out.println("Child thread " + Thread.currentThread().getName() + " is working");
                    Thread.sleep(3000);
                    System.out.println("Child thread " + Thread.currentThread().getName() + " is finished");
                    latch.countDown();
                }catch (InterruptedException e) {
                    e.printStackTrace();
                }
            };
        }.start();

        try {
            System.out.println("Wait for the 2 threads finished...");
            latch.await();
            System.out.println("2 threads are finished");
            System.out.println("Continue to execute the main thread");
        }catch (InterruptedException e) {
            e.printStackTrace();
        }
    }
}
```

Result:
```
Child thread: Thread-0 is working
Wait for the 2 threads finished...
Child thread Thread-1 is working
Child thread: Thread-0 is finished
Child thread Thread-1 is finished
2 threads are finished
Continue to execute the main thread
```

## CyclicBarrier

回环栅栏，通过它可以实现让一组线程等待至某个状态之后再全部同时执行。叫做回环是因为当所有等待线程都被释放以后，CyclicBarrier可以被重用。我们暂且把这个状态就叫做barrier，当调用await()方法之后，线程就处于barrier了。

CyclicBarrier类位于java.util.concurrent包下，CyclicBarrier提供2个构造器：
```
public CyclicBarrier(int parties, Runnable barrierAction) {
}
 
public CyclicBarrier(int parties) {
}
```

参数parties指让多少个线程或者任务等待至barrier状态；参数barrierAction为当这些线程都达到barrier状态时会执行的内容。

然后CyclicBarrier中最重要的方法就是await方法，它有2个重载版本：
```
public int await() throws InterruptedException, BrokenBarrierException { };

public int await(long timeout, TimeUnit unit)throws InterruptedException,BrokenBarrierException,TimeoutException { };
```

第一个版本比较常用，用来挂起当前线程，直至所有线程都到达barrier状态再同时执行后续任务；

第二个版本是让这些线程等待至一定的时间，如果还有线程没有到达barrier状态就直接让到达barrier的线程执行后续任务。

##### 假若有若干个线程都要进行写数据操作，并且只有所有线程都完成写数据操作之后，这些线程才能继续做后面的事情，此时就可以利用CyclicBarrier了：

```
import java.util.concurrent.BrokenBarrierException;
import java.util.concurrent.CyclicBarrier;

public class Cyclicbarrier {

    public static void main(String[] args) {
        int N = 4;
        CyclicBarrier barrier = new CyclicBarrier(N);
        for (int i = 0; i < N; i++) {
            new Writer(barrier).start();
        }
    }

    static class Writer extends Thread{
        private CyclicBarrier cyclicBarrier;
        public Writer(CyclicBarrier cyclicBarrier) {
            this.cyclicBarrier = cyclicBarrier;
        }

        @Override
        public void run() {
            System.out.println("线程"+Thread.currentThread().getName()+"正在写入数据...");
            try {
                Thread.sleep(5000);      //以睡眠来模拟写入数据操作
                System.out.println("线程"+Thread.currentThread().getName()+"写入数据完毕，等待其他线程写入完毕");
                cyclicBarrier.await();
            } catch (InterruptedException e) {
                e.printStackTrace();
            }catch(BrokenBarrierException e){
                e.printStackTrace();
            }
            System.out.println("所有线程写入完毕，继续处理其他任务...");
        }
    }
}
```
Result:
```
线程Thread-0正在写入数据...
线程Thread-1正在写入数据...
线程Thread-2正在写入数据...
线程Thread-3正在写入数据...
线程Thread-0写入数据完毕，等待其他线程写入完毕
线程Thread-1写入数据完毕，等待其他线程写入完毕
线程Thread-2写入数据完毕，等待其他线程写入完毕
线程Thread-3写入数据完毕，等待其他线程写入完毕
所有线程写入完毕，继续处理其他任务...
所有线程写入完毕，继续处理其他任务...
所有线程写入完毕，继续处理其他任务...
所有线程写入完毕，继续处理其他任务...
```

##### 如果说想在所有线程写入操作完之后，进行额外的其他操作可以为CyclicBarrier提供Runnable参数：
```
import java.util.concurrent.BrokenBarrierException;
import java.util.concurrent.CyclicBarrier;

public class Cyclicbarrier1 {
    public static void main(String[] args) {
        int N = 4;
        CyclicBarrier barrier  = new CyclicBarrier(N,new Runnable() {
            @Override
            public void run() {
                System.out.println("当前线程"+Thread.currentThread().getName());
            }
        });

        for(int i=0;i<N;i++)
            new Writer(barrier).start();
    }
    static class Writer extends Thread{
        private CyclicBarrier cyclicBarrier;
        public Writer(CyclicBarrier cyclicBarrier) {
            this.cyclicBarrier = cyclicBarrier;
        }

        @Override
        public void run() {
            System.out.println("线程"+Thread.currentThread().getName()+"正在写入数据...");
            try {
                Thread.sleep(5000);      //以睡眠来模拟写入数据操作
                System.out.println("线程"+Thread.currentThread().getName()+"写入数据完毕，等待其他线程写入完毕");
                cyclicBarrier.await();
            } catch (InterruptedException e) {
                e.printStackTrace();
            }catch(BrokenBarrierException e){
                e.printStackTrace();
            }
            System.out.println("所有线程写入完毕，继续处理其他任务...");
        }
    }
}
```

Result:
```
线程Thread-0正在写入数据...
线程Thread-1正在写入数据...
线程Thread-3正在写入数据...
线程Thread-2正在写入数据...
线程Thread-0写入数据完毕，等待其他线程写入完毕
线程Thread-1写入数据完毕，等待其他线程写入完毕
线程Thread-3写入数据完毕，等待其他线程写入完毕
线程Thread-2写入数据完毕，等待其他线程写入完毕
当前线程Thread-2
所有线程写入完毕，继续处理其他任务...
所有线程写入完毕，继续处理其他任务...
所有线程写入完毕，继续处理其他任务...
所有线程写入完毕，继续处理其他任务...
```

##### 利用await() 的时间限制， 即await的第二次方法去执行
```
import java.util.concurrent.BrokenBarrierException;
import java.util.concurrent.CyclicBarrier;
import java.util.concurrent.TimeUnit;
import java.util.concurrent.TimeoutException;

public class Cyclicbarrier2 {
    public static void main(String[] args) {
        int N = 4;
        CyclicBarrier barrier  = new CyclicBarrier(N);

        for(int i=0;i<N;i++) {
            if(i<N-1)
                new Writer(barrier).start();
            else {
                try {
                    Thread.sleep(5000);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
                new Writer(barrier).start();
            }
        }
    }
    static class Writer extends Thread{
        private CyclicBarrier cyclicBarrier;
        public Writer(CyclicBarrier cyclicBarrier) {
            this.cyclicBarrier = cyclicBarrier;
        }

        @Override
        public void run() {
            System.out.println("线程"+Thread.currentThread().getName()+"正在写入数据...");
            try {
                Thread.sleep(5000);      //以睡眠来模拟写入数据操作
                System.out.println("线程"+Thread.currentThread().getName()+"写入数据完毕，等待其他线程写入完毕");
                try {
                    cyclicBarrier.await(2000, TimeUnit.MILLISECONDS);
                } catch (TimeoutException e) {
                    // TODO Auto-generated catch block
                    e.printStackTrace();
                }
            } catch (InterruptedException e) {
                e.printStackTrace();
            }catch(BrokenBarrierException e){
                e.printStackTrace();
            }
            System.out.println(Thread.currentThread().getName()+"所有线程写入完毕，继续处理其他任务...");
        }
    }
}
```

Result:
```
线程Thread-0正在写入数据...
线程Thread-1正在写入数据...
线程Thread-2正在写入数据...
线程Thread-0写入数据完毕，等待其他线程写入完毕
线程Thread-1写入数据完毕，等待其他线程写入完毕
线程Thread-2写入数据完毕，等待其他线程写入完毕
线程Thread-3正在写入数据...
java.util.concurrent.BrokenBarrierException
Thread-1所有线程写入完毕，继续处理其他任务...
	at java.util.concurrent.CyclicBarrier.dowait(CyclicBarrier.java:250)
Thread-0所有线程写入完毕，继续处理其他任务...
	at java.util.concurrent.CyclicBarrier.await(CyclicBarrier.java:435)
Thread-2所有线程写入完毕，继续处理其他任务...
	at thread.redhat.com.ccs.Cyclicbarrier2$Writer.run(Cyclicbarrier2.java:39)
java.util.concurrent.BrokenBarrierException
	at java.util.concurrent.CyclicBarrier.dowait(CyclicBarrier.java:250)
	at java.util.concurrent.CyclicBarrier.await(CyclicBarrier.java:435)
	at thread.redhat.com.ccs.Cyclicbarrier2$Writer.run(Cyclicbarrier2.java:39)
java.util.concurrent.TimeoutException
	at java.util.concurrent.CyclicBarrier.dowait(CyclicBarrier.java:257)
	at java.util.concurrent.CyclicBarrier.await(CyclicBarrier.java:435)
	at thread.redhat.com.ccs.Cyclicbarrier2$Writer.run(Cyclicbarrier2.java:39)
线程Thread-3写入数据完毕，等待其他线程写入完毕
Thread-3所有线程写入完毕，继续处理其他任务...
java.util.concurrent.BrokenBarrierException
	at java.util.concurrent.CyclicBarrier.dowait(CyclicBarrier.java:207)
	at java.util.concurrent.CyclicBarrier.await(CyclicBarrier.java:435)
	at thread.redhat.com.ccs.Cyclicbarrier2$Writer.run(Cyclicbarrier2.java:39)

Process finished with exit code 0
```

上面的代码在main方法的for循环中，故意让最后一个线程启动延迟，因为在前面三个线程都达到barrier之后，等待了指定的时间发现第四个线程还没有达到barrier，就抛出异常并继续执行后面的任务。

##### 另外CyclicBarrier是可以重用的，看下面这个例子：
```
import java.util.concurrent.BrokenBarrierException;
import java.util.concurrent.CyclicBarrier;

public class Cyclicbarrier3 {
    public static void main(String[] args) {
        int N = 4;
        CyclicBarrier barrier  = new CyclicBarrier(N);

        for(int i=0;i<N;i++) {
            new Writer(barrier).start();
        }

        try {
            Thread.sleep(25000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }

        System.out.println("CyclicBarrier重用");

        for(int i=0;i<N;i++) {
            new Writer(barrier).start();
        }
    }
    static class Writer extends Thread{
        private CyclicBarrier cyclicBarrier;
        public Writer(CyclicBarrier cyclicBarrier) {
            this.cyclicBarrier = cyclicBarrier;
        }

        @Override
        public void run() {
            System.out.println("线程"+Thread.currentThread().getName()+"正在写入数据...");
            try {
                Thread.sleep(5000);      //以睡眠来模拟写入数据操作
                System.out.println("线程"+Thread.currentThread().getName()+"写入数据完毕，等待其他线程写入完毕");

                cyclicBarrier.await();
            } catch (InterruptedException e) {
                e.printStackTrace();
            }catch(BrokenBarrierException e){
                e.printStackTrace();
            }
            System.out.println(Thread.currentThread().getName()+"所有线程写入完毕，继续处理其他任务...");
        }
    }
}
```

Result:
```
线程Thread-0正在写入数据...
线程Thread-1正在写入数据...
线程Thread-2正在写入数据...
线程Thread-3正在写入数据...
线程Thread-0写入数据完毕，等待其他线程写入完毕
线程Thread-1写入数据完毕，等待其他线程写入完毕
线程Thread-2写入数据完毕，等待其他线程写入完毕
线程Thread-3写入数据完毕，等待其他线程写入完毕
Thread-3所有线程写入完毕，继续处理其他任务...
Thread-0所有线程写入完毕，继续处理其他任务...
Thread-2所有线程写入完毕，继续处理其他任务...
Thread-1所有线程写入完毕，继续处理其他任务...
CyclicBarrier重用
线程Thread-4正在写入数据...
线程Thread-5正在写入数据...
线程Thread-6正在写入数据...
线程Thread-7正在写入数据...
线程Thread-4写入数据完毕，等待其他线程写入完毕
线程Thread-6写入数据完毕，等待其他线程写入完毕
线程Thread-5写入数据完毕，等待其他线程写入完毕
线程Thread-7写入数据完毕，等待其他线程写入完毕
Thread-7所有线程写入完毕，继续处理其他任务...
Thread-4所有线程写入完毕，继续处理其他任务...
Thread-5所有线程写入完毕，继续处理其他任务...
Thread-6所有线程写入完毕，继续处理其他任务...

Process finished with exit code 0
```
从执行结果可以看出，在初次的4个线程越过barrier状态后，又可以用来进行新一轮的使用。而CountDownLatch无法进行重复使用。

## Semaphore

Semaphore翻译成字面意思为 信号量，Semaphore可以控同时访问的线程个数，通过 acquire() 获取一个许可，如果没有就等待，而 release() 释放一个许可。

Semaphore类位于java.util.concurrent包下，它提供了2个构造器：
```
public Semaphore(int permits) {          //参数permits表示许可数目，即同时可以允许多少线程进行访问
    sync = new NonfairSync(permits);
}

public Semaphore(int permits, boolean fair) {    //这个多了一个参数fair表示是否是公平的，即等待时间越久的越先获取许可
    sync = (fair)? new FairSync(permits) : new NonfairSync(permits);
}
```

下面说一下Semaphore类中比较重要的几个方法，首先是acquire()、release()方法：
```
public void acquire() throws InterruptedException {  }     //获取一个许可

public void acquire(int permits) throws InterruptedException { }    //获取permits个许可

public void release() { }          //释放一个许可

public void release(int permits) { }    //释放permits个许可
```

acquire()用来获取一个许可，若无许可能够获得，则会一直等待，直到获得许可。

release()用来释放许可。注意，在释放许可之前，必须先获获得许可。

这4个方法都会被阻塞，如果想立即得到执行结果，可以使用下面几个方法：

```
public boolean tryAcquire() { };    //尝试获取一个许可，若获取成功，则立即返回true，若获取失败，则立即返回false

public boolean tryAcquire(long timeout, TimeUnit unit) throws InterruptedException { };  //尝试获取一个许可，若在指定的时间内获取成功，则立即返回true，否则则立即返回false

public boolean tryAcquire(int permits) { }; //尝试获取permits个许可，若获取成功，则立即返回true，若获取失败，则立即返回false

public boolean tryAcquire(int permits, long timeout, TimeUnit unit) throws InterruptedException { }; //尝试获取permits个许可，若在指定的时间内获取成功，则立即返回true，否则则立即返回false
```

另外还可以通过availablePermits()方法得到可用的许可数目。

下面通过一个例子来看一下Semaphore的具体使用：

假若一个工厂有5台机器，但是有8个工人，一台机器同时只能被一个工人使用，只有使用完了，其他工人才能继续使用。那么我们就可以通过Semaphore来实现：
```
import java.util.concurrent.Semaphore;

public class Semaphore1 {
    public static void main(String[] args) {
        int N = 8;            //工人数
        Semaphore semaphore = new Semaphore(5); //机器数目
        for(int i=0;i<N;i++)
            new Worker(i,semaphore).start();
    }

    static class Worker extends Thread{
        private int num;
        private Semaphore semaphore;
        public Worker(int num,Semaphore semaphore){
            this.num = num;
            this.semaphore = semaphore;
        }

        @Override
        public void run() {
            try {
                semaphore.acquire();
                System.out.println("工人"+this.num+"占用一个机器在生产...");
                Thread.sleep(2000);
                System.out.println("工人"+this.num+"释放出机器");
                semaphore.release();
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
    }
}
```

Result:
```
工人0占用一个机器在生产...
工人1占用一个机器在生产...
工人2占用一个机器在生产...
工人3占用一个机器在生产...
工人5占用一个机器在生产...
工人0释放出机器
工人1释放出机器
工人4占用一个机器在生产...
工人6占用一个机器在生产...
工人2释放出机器
工人7占用一个机器在生产...
工人3释放出机器
工人5释放出机器
工人4释放出机器
工人6释放出机器
工人7释放出机器
```

### Conclusion

下面对上面说的三个辅助类进行一个总结：

##### CountDownLatch和CyclicBarrier都能够实现线程之间的等待，只不过它们侧重点不同：

* CountDownLatch一般用于某个线程A等待若干个其他线程执行完任务之后，它才执行；

* 而CyclicBarrier一般用于一组线程互相等待至某个状态，然后这一组线程再同时执行；

* 另外，CountDownLatch是不能够重用的，而CyclicBarrier是可以重用的。

##### Semaphore其实和锁有点类似，它一般用于控制对某组资源的访问权限。












