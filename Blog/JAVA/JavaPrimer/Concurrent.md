# Concurrent

## 多线程并发基础

资源对象（同步/加锁/...）
线程对象(休眠/等待)

### 一、基本概念

#### 线程基础
-   synchronised
-   使用显示的的Lock对象 lock.lock() finally {lock.unlock()} 只用为了解决特殊问题
-   volatile
-   线程状态：
	-   新建(new)：分配到必需的系统资源，并已初始化。可转为就绪或者阻塞
	-   就绪(Runnable)：只要调度器把时间片分配给线程，线程就可以运行
	-   阻塞(Blocked)：线程能够运行，但是条件不满足，可能是线程进入阻塞的状态
		-   sleep();
		-   wait();
		-   任务再等待某个输入、输出完成
		-   调用对象的同步控制方法，但对象锁不可用
		-   死亡(Dead)：不可再调度，完成或者中断
-   阻塞中断
	-   sleep()可中断
	-   IO阻塞 不可中断，但可以从底层关闭资源以释放锁
	-   synchronised互斥阻塞，同一互斥可以被同一任务多次获得

#### 线程之间的协作（以互斥为基础）
-   wait()/notifyAll()
	-   都属于Object基类的一部分
	-   进入wait()，释放锁
	-   notifyAll() 唤醒所有wait()挂起的任务
-   生产者－消费者 与同步队列
	-   同步队列：在任何情况下只允许一个任务插入或移除元素
	-   BlockingQueue
#### 死锁
- 发生死锁，必须同时满足四个条件
	-    互斥条件，任务使用的资源中至少有一个不能共享
	-   至少有一个任务必须持有一个资源且正在等待获取一个当前被别的任务持有的资源
	-   资源不能被任务抢占，任务必须把资源释放当作普通事件
	-   必须有资源循环等待，锁无法释放

### 二、Thread和Runnable的区别
如果一个类继承Thread，则不适合资源共享。但是如果实现了Runable接口的话，则很容易的实现资源共享。

实现Runnable接口比继承Thread类所具有的优势：
1）：适合多个相同的程序代码的线程去处理同一个资源
2）：可以避免java中的单继承的限制
3）：增加程序的健壮性，代码可以被多个线程共享，代码和数据独立

提醒一下大家：main方法其实也是一个线程。在java中所以的线程都是同时启动的，至于什么时候，哪个先执行，完全看谁先得到CPU的资源。在java中，每次程序运行至少启动2个线程。一个是main线程，一个是垃圾收集线程。因为每当使用java命令执行一个类的时候，实际上都会启动一个JVM，JVM就是在操作系统中启动了一个进程。

### 三、线程状态转换

![](./images/concurrent/线程状态转换.jpg)


1、新建状态（New）：新创建了一个线程对象。

2、就绪状态（Runnable）：线程对象创建后，其他线程调用了该对象的start()方法。该状态的线程位于可运行线程池中，变得可运行，等待获取CPU的使用权。

3、运行状态（Running）：就绪状态的线程获取了CPU，执行程序代码。

4、阻塞状态（Blocked）：阻塞状态是线程因为某种原因放弃CPU使用权，暂时停止运行。直到线程进入就绪状态，才有机会转到运行状态。阻塞的情况分三种：

（一）、等待阻塞：运行的线程执行wait()方法，JVM会把该线程放入等待池中。
（二）、同步阻塞：运行的线程在获取对象的同步锁时，若该同步锁被别的线程占用，则JVM会把该线程放入锁池中。
（三）、其他阻塞：运行的线程执行sleep()或join()方法，或者发出了I/O请求时，JVM会把该线程置为阻塞状态。当sleep()状态超时、join()等待线程终止或者超时、或者I/O处理完毕时，线程重新转入就绪状态。

5、死亡状态（Dead）：线程执行完了或者因异常退出了run()方法，该线程结束生命周期。

### 四、线程的调度
1、调整线程优先级：Java线程有优先级，优先级高的线程会获得较多的运行机会。
Java线程的优先级用整数表示，取值范围是1~10，Thread类有以下三个静态常量：
static int MAX_PRIORITY 
线程可以具有的最高优先级，取值为10。
static int MIN_PRIORITY
线程可以具有的最低优先级，取值为1。
static int NORM_PRIORITY
分配给线程的默认优先级，取值为5。
Thread类的setPriority()和getPriority()方法分别用来设置和获取线程的优先级。
每个线程都有默认的优先级。主线程的默认优先级为Thread.NORM_PRIORITY。
线程的优先级有继承关系，比如A线程中创建了B线程，那么B将和A具有相同的优先级。

JVM提供了10个线程优先级，但与常见的操作系统都不能很好的映射。如果希望程序能移植到各个操作系统中，应该仅仅使用Thread类有以下三个静态常量作为优先级，这样能保证同样的优先级采用了同样的调度方式。

2、线程睡眠：Thread.sleep(long millis)方法，使线程转到阻塞状态。millis参数设定睡眠的时间，以毫秒为单位。当睡眠结束后，就转为就绪（Runnable）状态。sleep()平台移植性好。

3、线程等待：Object类中的wait()方法，导致当前的线程等待，直到其他线程调用此对象的 notify() 方法或 notifyAll() 唤醒方法。这个两个唤醒方法也是Object类中的方法，行为等价于调用 wait(0) 一样。

4、线程让步：Thread.yield() 方法，暂停当前正在执行的线程对象，把执行机会让给相同或者更高优先级的线程。

5、线程加入：join()方法，等待其他线程终止。在当前线程中调用另一个线程的join()方法，则当前线程转入阻塞状态，直到另一个进程运行结束，当前线程再由阻塞转为就绪状态。

6、线程唤醒：Object类中的notify()方法，唤醒在此对象监视器上等待的单个线程。如果所有线程都在此对象上等待，则会选择唤醒其中一个线程。选择是任意性的，并在对实现做出决定时发生。线程通过调用其中一个 wait 方法，在对象的监视器上等待。 直到当前的线程放弃此对象上的锁定，才能继续执行被唤醒的线程。被唤醒的线程将以常规方式与在该对象上主动同步的其他所有线程进行竞争；例如，唤醒的线程在作为锁定此对象的下一个线程方面没有可靠的特权或劣势。类似的方法还有一个notifyAll()，唤醒在此对象监视器上等待的所有线程。

注意：Thread中suspend()和resume()两个方法在JDK1.5中已经废除，不再介绍。因为有死锁倾向。

### 五、常用函数说明

1) sleep(long millis):在指定的毫秒数内让当前正在执行的线程休眠（暂停执行）
2) join():指等待t线程终止。

join是Thread类的一个方法，启动线程后直接调用，即join()的作用是：“等待该线程终止”，这里需要理解的就是该线程是指的主线程等待子线程的终止。也就是在子线程调用了join()方法后面的代码，只有等到子线程结束了才能执行。

为什么要用join()方法
在很多情况下，主线程生成并起动了子线程，如果子线程里要进行大量的耗时的运算，主线程往往将于子线程之前结束，但是如果主线程处理完其他的事务后，需要用到子线程的处理结果，也就是主线程需要等待子线程执行完成之后再结束，这个时候就要用到join()方法了。

- 无join

```
package test;

  

public class Thread1 extends Thread {
       public Thread1(String name){
             super(name);
      }
       @Override
       public void run() {
            System. out.println(Thread. currentThread().getName()+ "主线程运行开始!" );
             for ( int i = 0; i < 5; i++) {
                   try {
                         sleep(1000);
                  } catch (InterruptedException e) {
                  }
                  System. out.println(i);
            }
            System. out.println(Thread. currentThread().getName()+ "主线程运行结束!" );
      }
       public static void main(String[] args) {
            System. out.println(Thread. currentThread().getName()+ "主线程运行开始!" );
            Thread1 thread1 = new Thread1( "A");
            Thread1 thread2 = new Thread1( "B");
            thread1.start();
            thread2.start();
            System. out.println(Thread. currentThread().getName()+ "主线程运行结束!" );
      }

}
```

```
>>> sysout
main主线程运行开始!
main主线程运行结束!
A主线程运行开始!
B主线程运行开始!

0
0
1
1
2
2
3
3
4
B主线程运行结束!
4
A主线程运行结束!
```

- 有join
```
package test;

public class ThreadJoin extends Thread {

       public ThreadJoin(String name){
             super(name);
      }
       @Override
       public void run() {
            System. out.println(Thread. currentThread().getName()+ "主线程运行开始!" );
             for ( int i = 0; i < 5; i++) {
                   try {
                         sleep(1000);
                  } catch (InterruptedException e) {
                  }
                  System. out.println(i);
            }
            System. out.println(Thread. currentThread().getName()+ "主线程运行end!");
      }

       public static void main(String[] args) {

            System. out.println(Thread. currentThread().getName()+ "主线程运行开始!" );
            ThreadJoin threadJoin = new ThreadJoin( "A");
            ThreadJoin threadJoin2 = new ThreadJoin( "B");
            threadJoin.start();
             try {
                  threadJoin.join();
            } catch (InterruptedException e) {
            }
            threadJoin2.start();
             try {
                  threadJoin2.join();
            } catch (InterruptedException e) {
            }
            System. out.println(Thread. currentThread().getName()+ "主线程运行end!");
      }
}
```

```
>>>sysout

main主线程运行开始!
A主线程运行开始!
0
1
2
3
4
A主线程运行end!
B主线程运行开始!
0
1
2
3
4
B主线程运行end!
main主线程运行end!
```

3) yield():暂停当前正在执行的线程对象，并执行其他线程。（重新竞争）

Thread.yield()方法作用是：暂停当前正在执行的线程对象，并执行其他线程。
yield()应该做的是让当前运行线程回到可运行状态，以允许具有相同优先级的其他线程获得运行机会。因此，使用yield()的目的是让相同优先级的线程之间能适当的轮转执行。但是，实际中无法保证yield()达到让步目的，因为让步的线程还有可能被线程调度程序再次选中。

sleep()和yield()的区别
sleep()和yield()的区别):sleep()使当前线程进入停滞状态，所以执行sleep()的线程在指定的时间内肯定不会被执行；yield()只是使当前线程重新回到可执行状态，所以执行yield()的线程有可能在进入到可执行状态后马上又被执行。

sleep方法使当前运行中的线程睡眼一段时间，进入不可运行状态，这段时间的长短是由程序设定的，yield方法使当前线程让出CPU占有权，但让出的时间是不可设定的。实际上，yield()方法对应了如下操作：先检测当前是否有相同优先级的线程处于同可运行状态，如有，则把CPU的占有权交给此线程，否则，继续运行原来的线程。所以yield()方法称为“退让”，它把运行机会让给了同等优先级的其他线程

另外，sleep方法允许较低优先级的线程获得运行机会，但yield()方法执行时，当前线程仍处在可运行状态，所以，不可能让出较低优先级的线程些时获得CPU占有权。在一个运行系统中，如果较高优先级的线程没有调用sleep方法，又没有受到IO阻塞，那么，较低优先级线程只能等待所有较高优先级的线程运行结束，才有机会运行。

4) interrupt():
中断某个线程，这种结束方式比较粗暴，如果t线程打开了某个资源还没来得及关闭也就是run方法还没有执行完就强制结束线程，会导致资源无法关闭。要想结束进程最好的办法就是用sleep()函数的例子程序里那样，在线程类里面用以个boolean型变量来控制run()方法什么时候结束，run()方法一结束，该线程也就结束了。

5) wait()
Obj.wait()，与Obj.notify()必须要与synchronized(Obj)一起使用，也就是wait,与notify是针对已经获取了Obj锁进行操作，从语法角度来说就是Obj.wait(),Obj.notify必须在synchronized(Obj){...}语句块内。从功能上来说wait就是说线程在获取对象锁后，主动释放对象锁，同时本线程休眠。直到有其它线程调用对象的notify()唤醒该线程，才能继续获取对象锁，并继续执行。相应的notify()就是对对象锁的唤醒操作。但有一点需要注意的是notify()调用后，并不是马上就释放对象锁的，而是在相应的synchronized(){}语句块执行结束，自动释放锁后，JVM会在wait()对象锁的线程中随机选取一线程，赋予其对象锁，唤醒线程，继续执行。这样就提供了在线程间同步、唤醒的操作。Thread.sleep()与Object.wait()二者都可以暂停当前线程，释放CPU控制权，主要的区别在于Object.wait()在释放CPU同时，释放了对象锁的控制。

建立三个线程，A线程打印10次A，B线程打印10次B,C线程打印10次C，要求线程同时运行，交替打印10次ABC。这个问题用Object的wait()，notify()就可以很方便的解决
```
package test;

public class ThreadPrinter extends Thread {
      String       name;
      Object       pre;
      Object       self;
       public ThreadPrinter(String name, Object pre, Object self) {
             this. name = name;
             this. pre = pre;
             this. self = self;
      }
       @Override
       public void run() {
             for ( int i = 0; i < 10; i++) {
                   synchronized ( pre) {
                         synchronized ( self) {
                              System. out.print( name+ "->");
                               self.notify(); //唤醒下一个
                        }
                         try {
                               pre.wait(); //等待前一个唤醒
                               sleep(200);
                        } catch (InterruptedException e) {}
                  }
            }
      }

       public static void main(String[] args) throws InterruptedException {
            Object a = new Object();
            Object b = new Object();
            Object c = new Object();
             new ThreadPrinter( "A", c, a).start();
            Thread. sleep(100);
             new ThreadPrinter( "B", a, b).start();
            Thread. sleep(100);
             new ThreadPrinter( "C", b, c).start();
      }
}
```

wait和sleep区别
共同点：
1. 他们都是在多线程的环境下，都可以在程序的调用处阻塞指定的毫秒数，并返回。
2. wait()和sleep()都可以通过interrupt()方法 打断线程的暂停状态 ，从而使线程立刻抛出InterruptedException。
如果线程A希望立即结束线程B，则可以对线程B对应的Thread实例调用interrupt方法。如果此刻线程B正在wait/sleep /join，则线程B会立刻抛出InterruptedException，在catch() {} 中直接return即可安全地结束线程。
需要注意的是，InterruptedException是线程自己从内部抛出的，并不是interrupt()方法抛出的。对某一线程调用 interrupt()时，如果该线程正在执行普通的代码，那么该线程根本就不会抛出InterruptedException。但是，一旦该线程进入到 wait()/sleep()/join()后，就会立刻抛出InterruptedException 。
不同点：
1. Thread类的方法：sleep(),yield()等
Object的方法：wait()和notify()等
2. 每个对象都有一个锁来控制同步访问。Synchronized关键字可以和对象的锁交互，来实现线程的同步。
sleep方法没有释放锁，而wait方法释放了锁，使得其他线程可以使用同步控制块或者方法。
3. wait，notify和notifyAll只能在同步控制方法或者同步控制块里面使用，而sleep可以在任何地方使用
4. sleep必须捕获异常，而wait，notify和notifyAll不需要捕获异常
所以sleep()和wait()方法的最大区别是：
sleep()睡眠时，保持对象锁，仍然占有该锁；
而wait()睡眠时，释放对象锁。
但是wait()和sleep()都可以通过interrupt()方法打断线程的暂停状态，从而使线程立刻抛出InterruptedException（但不建议使用该方法）。
sleep（）方法
sleep()使当前线程进入停滞状态（阻塞当前线程），让出CUP的使用、目的是不让当前线程独自霸占该进程所获的CPU资源，以留一定时间给其他线程执行的机会;
sleep()是Thread类的Static(静态)的方法；因此他不能改变对象的机锁，所以当在一个Synchronized块中调用Sleep()方法是，线程虽然休眠了，但是对象的机锁并木有被释放，其他线程无法访问这个对象（即使睡着也持有对象锁）。
在sleep()休眠时间期满后，该线程不一定会立即执行，这是因为其它线程可能正在运行而且没有被调度为放弃执行，除非此线程具有更高的优先级。
wait（）方法
wait()方法是Object类里的方法；当一个线程执行到wait()方法时，它就进入到一个和该对象相关的等待池中，同时失去（释放）了对象的机锁（暂时失去机锁，wait(long timeout)超时时间到后还需要返还对象锁）；其他线程可以访问；
wait()使用notify或者notifyAlll或者指定睡眠时间来唤醒当前等待池中的线程。
wiat()必须放在synchronized block中，否则会在program runtime时扔出”java.lang.IllegalMonitorStateException“异常。

### 六、线程同步
1、synchronized关键字的作用域有二种：
1）是某个对象实例内，synchronized aMethod(){}可以防止多个线程同时访问这个对象的synchronized方法（如果一个对象有多个synchronized方法，只要一个线程访问了其中的一个synchronized方法，其它线程不能同时访问这个对象中任何一个synchronized方法）。这时，不同的对象实例的synchronized方法是不相干扰的。也就是说，其它线程照样可以同时访问相同类的另一个对象实例中的synchronized方法；
2）是某个类的范围，synchronized static aStaticMethod{}防止多个线程同时访问这个类中的synchronized static 方法。它可以对类的所有对象实例起作用。

2 synchronized 的二种用法
当 synchronized 用来修饰一个方法或者一个代码块的时候，能够保证在同一时刻最多只有一个线程执行该段代码
(1)synchronized 方法
- synchronized 方法控制对类成员变量的访问：每个类实例对应一把锁，每个 synchronized 方法都必须获得调用该方法的类实例的锁方能执行，否则所属线程阻塞，方法一旦执行，就独占该锁，直到从该方法返回时才将锁释放，此后被阻塞的线程方能获得该锁，重新进入可执行状态。这种机制确保了同一时刻对于每一个类实例，其所有声明为 synchronized 的成员函数中至多只有一个处于可执行状态（因为至多只有一个能够获得该类实例对应的锁），从而有效避免了类成员变量的访问冲突（只要所有可能访问类成员变量的方法均被声明为 synchronized）
- synchronized 方法的缺陷：若将一个大的方法声明为ynchronized 将会大大影响效率，典型地，若将线程类的方法 run() 声明synchronized ，由于在线程的整个生命期内它一直在运行，因此将导致它对本类任何 synchronized 方法的调用都永远不会成功
(2)synchronized 块
- synchronized关键字可以作为函数的修饰符，也可作为函数内的语句，也就是平时说的同步方法和同步语句块。如果再细的分类，synchronized可作用于instance变量、object reference（对象引用）、static函数和class literals(类名称字面常量)
- 无论synchronized关键字加在方法上还是对象上，它取得的锁都是对象，而不是把一段代码或函数当作锁――而且同步方法很可能还会被其他线程的对象访问
- 每个对象只有一个锁（lock）与之相关联

3、除了方法前用synchronized关键字，synchronized关键字还可以用于方法中的某个区块中，表示只对这个区块的资源实行互斥访问。用法是: synchronized(this){/*区块*/}，它的作用域是当前对象；

4、synchronized关键字是不能继承的，也就是说，基类的方法synchronized f(){} 在继承类中并不自动是synchronized f(){}，而是变成了f(){}。继承类需要你显式的指定它的某个方法为synchronized方法；
Java对多线程的支持与同步机制深受大家的喜爱，似乎看起来使用了synchronized关键字就可以轻松地解决多线程共享数据同步问题。到底如何？——还得对synchronized关键字的作用进行深入了解才可定论。
总的说来，synchronized关键字可以作为函数的修饰符，也可作为函数内的语句，也就是平时说的同步方法和同步语句块。如果再细的分类，synchronized可作用于instance变量、object reference（对象引用）、static函数和class literals(类名称字面常量)身上。
在进一步阐述之前，我们需要明确几点：
A．无论synchronized关键字加在方法上还是对象上，它取得的锁都是对象，而不是把一段代码或函数当作锁——而且同步方法很可能还会被其他线程的对象访问。
B．每个对象只有一个锁（lock）与之相关联。
C．实现同步是要很大的系统开销作为代价的，甚至可能造成死锁，所以尽量避免无谓的同步控制。



## Callable和Future
Callable接口类似于Runnable，从名字就可以看出来了，但是Runnable不会返回结果，并且无法抛出返回结果的异常，而Callable功能更强大一些，被线程执行后，可以返回值，这个返回值可以被Future拿到，也就是说，Future可以拿到异步执行任务的返回值

```
package test;

import java.util.concurrent.Callable;
import java.util.concurrent.ExecutionException;
import java.util.concurrent.FutureTask;

public class CallableAndFuture1 {

       public static void main(String[] args) {

             for ( int i = 0; i < 5; i++) {
                   final int n = i;
                  Callable<Integer> callable = new Callable<Integer>() {
                         @Override
                         public Integer call() throws Exception {
                               return n;
                        }
                  };
                  FutureTask<Integer> futureTask = new FutureTask<Integer>(callable);
                   new Thread(futureTask).start();
                   try {
                        System. out.println(futureTask.get());
                  } catch (InterruptedException e) {
                        e.printStackTrace();
                  } catch (ExecutionException e) {
                        e.printStackTrace();
                  }
            }
      }
}
```

```
package test;

import java.util.concurrent.Callable;
import java.util.concurrent.ExecutionException;
import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;
import java.util.concurrent.Future;

public class CallableAndFuture2 {

       public static void main(String[] args) {

            ExecutorService service = Executors. newCachedThreadPool();
             for ( int i = 0; i < 5; i++) {
                   final int n = i;
                  Callable<Integer> callable = new Callable<Integer>() {
                         @Override
                         public Integer call() throws Exception {
                               return n;
                        }
                  };
                  Future<Integer> futureTask = service.submit(callable);
                   try {
                        System. out.println(futureTask.get());
                  } catch (InterruptedException e) {
                        e. printStackTrace();
                  } catch (ExecutionException e) {
                        e.printStackTrace();
                  }
            }
      }
}
```

```
package test;

import java.util.concurrent.Callable;
import java.util.concurrent.CompletionService;
import java.util.concurrent.ExecutionException;
import java.util.concurrent.ExecutorCompletionService;
import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;

public class CallableAndFuture3 {

    public static void main(String[] args) {

        ExecutorService tPool = Executors.newCachedThreadPool();
        CompletionService<Integer> completionService = new ExecutorCompletionService<Integer>(tPool);
        for (int i = 0; i < 5; i++) {
            final int n = i;
            Callable<Integer> callable = new Callable<Integer>() {
                @Override
                public Integer call() throws Exception {
                    return n;
                }
            };
            completionService.submit(callable);
        }
        for (int i = 0; i < 5; i++) {
            try {
                System.out.println(completionService.take().get());
            } catch (InterruptedException e) {
                e.printStackTrace();
            } catch (ExecutionException e) {
                e.printStackTrace();
            }
        }
    }
}
```

## semaphore
Semaphore也是一个线程同步的辅助类，可以维护当前访问自身的线程个数，并提供了同步机制。使用Semaphore可以控制同时访问资源的线程个数，例如，实现一个文件允许的并发访问数。
Semaphore的主要方法摘要：
　　void acquire():从此信号量获取一个许可，在提供一个许可前一直将线程阻塞，否则线程被中断。
　　void release():释放一个许可，将其返回给信号量。
　　int availablePermits():返回此信号量中当前可用的许可数。
　　boolean hasQueuedThreads():查询是否有线程正在等待获取

Semaphore就是一个信号量，它的作用是限制某段代码块的并发数。Semaphore有一个构造函数，可以传入一个int型整数n，表示某段代码最多只有n个线程可以访问，如果超出了n，那么请等待，等到某个线程执行完毕这段代码块，下一个线程再进入。由此可以看出如果Semaphore构造函数中传入的int型整数n=1，相当于变成了一个synchronized了

```
package test;

import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;
import java.util.concurrent.Semaphore;

public class SemaphoreTest extends Thread {
      private Pool pool;
      SemaphoreTest(Pool pool){
            this.pool = pool;
      }
      @Override
      public void run() {
            try {
                  Integer i = pool.getItem();
                  sleep((i+1)*2000);
                  System.out.println((i+1)*2000);
                  pool.putItem(i);
            } catch (InterruptedException e1) {
                  e1.printStackTrace();
            }
      }

      public static void main(String[] args) {
            Pool pool = new Pool();
            ExecutorService executorService = Executors.newCachedThreadPool();
            for (int i = 0; i <20; i++) {
                  executorService.execute(new SemaphoreTest(pool));
            }
      }
}

class Pool {
      private static final int      MAX_AVAILABLE     = 11;
      private final Semaphore       available         = new Semaphore(MAX_AVAILABLE, true);

      public Integer getItem() throws InterruptedException {
            available.acquire();
            return getNextAvailableItem();
      }

      public void putItem(Integer x) {
            if (markAsUnused(x))
                  available.release();
      }

      // Not a particularly efficient data structure; just for demo

      protected int[]         items = { 0, 1, 2, 3, 4, 5, 6, 7, 8, 9, 10 };
      protected boolean[]     used  = new boolean[MAX_AVAILABLE];

      protected synchronized Integer getNextAvailableItem() {
            for (int i = 0; i < MAX_AVAILABLE; ++i) {
                  if (!used[i]) {
                        used[i] = true;
                        System.out.println("get" + i);
                        return items[i];
                  }
            }
            return null; // not reached
      }

      protected synchronized boolean markAsUnused(int item) {
            for (int i = 0; i < MAX_AVAILABLE; ++i) {
                  if (item == items[i]) {
                        if (used[i]) {
                              used[i] = false;
                              System.out.println("put" + i);
                              return true;
                        } else
                              return false;
                  }
            }
            return false;
      }

}
```

## 并发锁

### 自旋锁

自旋锁： 线程挂起和恢复需转入内核态中完成，给系统并发性能带来了很大的压力，自旋锁让后面请求锁的线程等待，执行一个忙循环（自旋），不放弃CPU的执行时间，避免线程切换带来的性能开销。对于自旋锁，如果持有锁的线程占用时间短，自选等待的效果好，反之会浪费很多的CPU时间。所以自旋次数需要限制，默认10次。
很多synchronized里面的代码只是一些很简单的代码，执行时间非常快，此时等待的线程都加锁可能是一种不太值得的操作，因为线程阻塞涉及到用户态和内核态切换的问题。既然synchronized里面的代码执行得非常快，不妨让等待锁的线程不要被阻塞，而是在synchronized的边界做忙循环，这就是自旋。如果做了多次忙循环发现还没有获得锁，再阻塞，这样可能是一种更好的策略。
自适应自旋锁：自旋的时间由前一次在同一个锁上自旋时间及锁的拥有者状态决定。如果在同一个锁的对象上，自旋等待刚成功获得过锁，JVM认为这次自旋成功的概率大，允许自旋等待时间更长。如果自旋很少成功，那么以后获得锁的自旋过程可能会被忽略。

### 轻量级锁 

轻量级锁的能提升同步性能的依据是：对于绝大部分的锁，在整个同步周期内都是不存在竞争的。如果没有竞争，轻量级锁使用CAS避免互斥的开销。如果存在竞争，除了互斥量开销还要额外的CAS操作，会比传统的锁更慢。

### 偏向锁

在无竞争的的情况下去除整个同步，连CAS都不做。偏向于第一个获取它的线程，如果接下来的执行过程，该锁没有被其他线程获取，则持有偏向锁的线程将不再进行同步。当另一个线程需要获取这个锁，偏向模式结束。


## AbstractQueuedSychronizer(AQS)

AQS全称为AbstractQueuedSychronizer，翻译过来应该是抽象队列同步器。
如果说java.util.concurrent的基础是CAS的话，那么AQS就是整个Java并发包的核心了，ReentrantLock、CountDownLatch、Semaphore等等都用到了它。AQS实际上以双向队列的形式连接所有的Entry，比方说ReentrantLock，所有等待的线程都被放在一个Entry中并连成双向队列，前面一个线程使用ReentrantLock好了，则双向队列实际上的第一个Entry开始运行。
AQS定义了对双向队列所有的操作，而只开放了tryLock和tryRelease方法给开发者使用，开发者可以根据自己的实现重写tryLock和tryRelease方法，以实现自己的并发功能

## FutureTask

FutureTask表示一个异步运算的任务。FutureTask里面可以传入一个Callable的具体实现类，可以对这个异步运算的任务的结果进行等待获取、判断是否已经完成、取消任务等操作。当然，由于FutureTask也是Runnable接口的实现类，所以FutureTask也可以放入线程池中。

## ConcurrentHashMap

JDK1.7的分段锁机制Hashtable之所以效率低下主要是因为其实现使用了synchronized关键字对put等操作进行加锁，而synchronized关键字加锁是对整个对象进行加锁。也就是说在进行put等修改Hash表的操作时，锁住了整个Hash表，从而使得其表现的效率低下。
因此，在JDK1.5到1.7版本，Java使用了分段锁机制实现ConcurrentHashMap。
简而言之，ConcurrentHashMap在对象中保存了一个Segment数组，即将整个Hash表划分为多个分段。而每个Segment元素，即每个分段则类似于一个Hashtable。在执行put操作时首先根据hash算法定位到元素属于哪个Segment，然后使用ReentrantLock对该Segment加锁即可。因此，ConcurrentHashMap在多线程并发编程中可是实现多线程put操作。
Segment类是ConcurrentHashMap中的内部类，继承于ReentrantLock类。ConcurrentHashMap与Segment是组合关系，一个ConcurrentHashMap对象包含若干个Segment对象，ConcurrentHashMap类中存在“Segment数组”成员。
HashEntry也是ConcurrentHashMap的内部类，是单向链表节点，存储着key-value键值对。Segment与HashEntry是组合关系，Segment类中存在“HashEntry数组”成员，“HashEntry数组”中的每个HashEntry就是一个单向链表。

JDK1.8的改进在JDK1.7的版本，ConcurrentHashMap是通过分段锁机制来实现的，所以其最大并发度受Segment的个数限制。因此，在JDK1.8中，ConcurrentHashMap的实现原理摒弃了这种设计，而是选择了与HashMap类似的数组+链表+红黑树的方式实现，而加锁则采用CAS原子更新、volatile关键字、synchronized可重入锁实现。
JDK1.8的实现降低锁的粒度，JDK1.7版本锁的粒度是基于Segment的，包含多个HashEntry，而JDK1.8锁的粒度就是HashEntry（首节点）。
JDK1.8版本的数据结构变得更加简单，使得操作也更加清晰流畅，因为已经使用synchronized来进行同步，所以不需要分段锁的概念，也就不需要Segment这种数据结构了，由于粒度的降低，实现的复杂度也增加了。
JDK1.8版本的扩容操作支持多线程并发。在之前的版本中如果Segment正在进行扩容操作，其他写线程都会被阻塞，JDK1.8改为一个写线程触发了扩容操作，其他写线程进行写入操作时，可以帮助它来完成扩容这个耗时的操作。
JDK1.8使用红黑树来优化链表，基于长度很长的链表的遍历是一个很漫长的过程，而红黑树的遍历效率是很快的，代替一定阈值的链表。重要属性
sizeCtl：标志控制符。这个参数非常重要，出现在ConcurrentHashMap的各个阶段，不同的值也表示不同情况和不同功能：
负数代表正在进行初始化或扩容操作。-1表示正在进行初始化操作。-N表示有N-1个线程正在进行扩容操作。
其为0时，表示hash表还未初始化。
正数表示下一次进行扩容的大小，类似于扩容阈值。它的值始终是当前容量的0.75倍，如果hash表的实际大小>=sizeCtl，则进行扩容。

### 构造方法
需要说明的是，在构造方法里并没有对集合进行初始化操作，而是等到了添加元素的时候才进行初始化，属于懒汉式的加载方式。
而且loadFactor参数在JDK1.8中也不再有加载因子的意义，仅为了兼容以前的版本，加载因子由sizeCtl来替代。
同样，concurrencyLevel参数在JDK1.8中也不再有多线程运行的并发度的意义，仅为了兼容以前的版本。

```
// 空参构造器。
 public ConcurrentHashMap() {
 }

 // 指定初始容量的构造器。
 public ConcurrentHashMap(int initialCapacity) {
 // 参数有效性判断。
 if (initialCapacity < 0)
 throw new IllegalArgumentException();
 int cap = ((initialCapacity >= (MAXIMUM_CAPACITY >>> 1)) ?
 MAXIMUM_CAPACITY :
 tableSizeFor(initialCapacity + (initialCapacity >>> 1) + 1));
 // 设置标志控制符。
this.sizeCtl = cap;
 }

 // 指定初始容量，加载因子的构造器。
 public ConcurrentHashMap(int initialCapacity, float loadFactor) {
 this(initialCapacity, loadFactor, 1);
 }

 // 指定初始容量，加载因子，并发度的构造器。
 public ConcurrentHashMap(int initialCapacity, float loadFactor, int concurrencyLevel) {
 // 参数有效性判断。
 if (!(loadFactor > 0.0f) || initialCapacity < 0 || concurrencyLevel <= 0)
 throw new IllegalArgumentException();
 // 比较初始容量和并发度的大小，取最大值作为初始容量。
 if (initialCapacity < concurrencyLevel)
 initialCapacity = concurrencyLevel;
 // 计算最大容量。
 long size = (long)(1.0 + (long)initialCapacity / loadFactor);
 int cap = (size >= (long)MAXIMUM_CAPACITY) ?
 MAXIMUM_CAPACITY : tableSizeFor((int)size);
 // 设置标志控制符。
 this.sizeCtl = cap;
 }

 // 包含指定Map集合的构造器。
 public ConcurrentHashMap(Map<? extends K, ? extends V> m) {
 // 设置标志控制符。
 this.sizeCtl = DEFAULT_CAPACITY;
 // 放置指定的集合。
 putAll(m);
}
```

### 初始化方法
集合并不会在构造方法里进行初始化，而是在用到集合的时候才进行初始化，在初始化的同时会设置集合的阈值。
初始化方法主要应用了关键属性sizeCtl，如果sizeCtl小于0，表示其他线程正在进行初始化，就放弃这个操作，在这也可以看出初始化只能由一个线程完成。如果获得了初始化权限，就用CAS方法将sizeCtl置为-1，防止其他线程进入。初始化完成后，将sizeCtl的值改为0.75倍的集合容量，作为阈值。
在初始化的过程中为了保证线程安全，总共使用了两步操作：
1）通过CAS原子更新方法将sizeCtl设置为-1，保证只有一个线程进入。
2）线程获取初始化权限后内部通过 if ((tab = table) == null || tab.length == 0) 二次判断，保证只有在未初始化的情况下才能执行初始化。
```
// 初始化集合，使用CAS原子更新保证线程安全，使用volatile保证顺序和可见性。
 private final Node<K,V>[] initTable() {
 Node<K,V>[] tab; int sc;
 // 死循环以完成初始化。
 while ((tab = table) == null || tab.length == 0) {
 // 如果sizeCtl小于0则表示正在初始化，当前线程让步。
 if ((sc = sizeCtl) < 0)
 Thread.yield();
 // 如果需要初始化，并且使用CAS原子更新。判断SIZECTL保存的sizeCtl值是否和sc一致，一致则将sizeCtl更新为-1。
 else if (U.compareAndSwapInt(this, SIZECTL, sc, -1)) {
 try {
 // 第一个线程初始化之后，第二个线程还会进来所以需要重新判断。类似于线程同步的二次判断。
 if ((tab = table) == null || tab.length == 0) {
 // 如果没有指定容量则使用默认容量16。
 int n = (sc > 0) ? sc : DEFAULT_CAPACITY;
 // 初始化一个指定容量的节点数组。
 @SuppressWarnings("unchecked")
 Node<K,V>[] nt = (Node<K,V>[])new Node<?,?>[n];
 // 将节点数组指向集合。
 table = tab = nt;
 // 扩容阀值，获取容量的0.75倍的值，写法略叼更高端比直接乘高效。
 sc = n - (n >>> 2);
 }
 } finally {
 // 将sizeCtl的值设为阈值。
 sizeCtl = sc;
 }
 break;
 }
 }
 return tab;
 }
```

### 添加方法
1）校验数据。判断传入一个key和value是否为空，如果为空就直接报错。ConcurrentHashMap是不可为空的（HashMap是可以为空的）。
2）是否要初始化。判断table是否为空，如果为空就进入初始化阶段。
3）如果数组中key指定的桶是空的，那就使用CAS原子操作把键值对插入到这个桶中作为头节点。
4）协助扩容。如果这个要插入的桶中的hash值为-1，也就是MOVED状态（也就是这个节点是ForwordingNode），那就是说明有线程正在进行扩容操作，那么当前线程就进入协助扩容阶段。
5）插入数据到链表或者红黑树。如果这个节点是链表节点，那么就遍历这个链表，如果有相同的key值就更新value值，如果没有发现相同的key值，就在链表的尾部插入该数据。如果这个节点是红黑树节点，那就需要按照树的插入规则进行插入。
6）转化成红黑树。插入结束之后判断如果是链表节点，并且个数大于8，就需要把链表转化为红黑树存储。
7）添加结束之后，需要给增加已存储的数量，并判断是否需要扩容。
```
// 添加元素。
2 public V put(K key, V value) {
3 return putVal(key, value, false);
4 }
5
6 // 添加元素。
7 final V putVal(K key, V value, boolean onlyIfAbsent) {
8 // 排除null的数据。
9 if (key == null || value == null) throw new NullPointerException();
10 // 计算hash，并保证hash一定大于零，负数表示在扩容或者是树节点。
11 int hash = spread(key.hashCode());
12 // 节点个数。0表示未加入新结点，2表示TreeBin或链表结点数，其它值表示链表结点数。主要用于每次加入结点后查看是否要由链表转为红黑树。
13 int binCount = 0;
14 // CAS经典写法，不成功无限重试。
15 for (Node<K,V>[] tab = table;;) {
16 // 声明节点、集合长度、对应的数组下标、节点的hash值。
17 Node<K,V> f; int n, i, fh;
18 // 如果没有初始化则进行初始化。除非构造时指定集合，否则默认构造不初始化，添加时检查是否初始化，属于懒汉模式初始化。
19 if (tab == null || (n = tab.length) == 0)
20 // 初始化集合。
21 tab = initTable();
22 // 如果已经初始化了，并且使用CAS根据hash获取到的节点为null。
23 else if ((f = tabAt(tab, i = (n - 1) & hash)) == null) {
24 // 使用CAS比较该索引处是否为null防止其它线程已改变该值，null则插入。
25 if (casTabAt(tab, i, null, new Node<K,V>(hash, key, value, null)))
26 // 添加成功，跳出循环。
27 break;
28 }
29 // 如果获取到节点不为null，并且节点的hash为-1，则表示节点在扩容。
30 else if ((fh = f.hash) == MOVED)
31 // 帮助扩容。
32 tab = helpTransfer(tab, f);
33 // 产生hash碰撞，并且没有扩容操作。
34 else {
35 V oldVal = null;
36 // 锁住节点。
37 synchronized (f) {
38 // 这里volatile获取首节点与节点对比判断节点还是不是首节点。
39 if (tabAt(tab, i) == f) {
40 // 判断是否是链表节点。
41 if (fh >= 0) {
42 // 记录节点个数。
43 binCount = 1;
44 // 循环完成添加节点到链表。
45 for (Node<K,V> e = f;; ++binCount) {
46 K ek;
47 if (e.hash == hash && ((ek = e.key) == key || (ek != null && key.equals(ek)))) {
48 oldVal = e.val;
49 if (!onlyIfAbsent)
50 e.val = value;
51 break;
52 }
53 Node<K,V> pred = e;
54 if ((e = e.next) == null) {
55 pred.next = new Node<K,V>(hash, key, value, null);
56 break;
57 }
58 }
59 }
60 // 如果是红黑树节点。
61 else if (f instanceof TreeBin) {
62 Node<K,V> p;
63 binCount = 2;
64 if ((p = ((TreeBin<K,V>)f).putTreeVal(hash, key, value)) != null) {
65 oldVal = p.val;
66 if (!onlyIfAbsent)
67 p.val = value;
68 }
69 }
70 }
71 }
72 // 如果添加到了链表节点，需要进一步判断是否需要转为红黑树。
73 if (binCount != 0) {
74 // 如果链表上的节点数大于等于8。
75 if (binCount >= TREEIFY_THRESHOLD)
76 // 尝试转为红黑树。
77 treeifyBin(tab, i);
78 if (oldVal != null)
79 // 返回原值。
80 return oldVal;
81 break;
82 }
83 }
84 }
85 // 集合容量加一并判断是否要扩容。
86 addCount(1L, binCount);
87 return null;
88 }
```
### ### 修改容量并判断是否需要扩容

1）尝试对baseCount和CounterCell进行增加的操作，这些操作基于CAS原子操作，同时使用volatile保证顺序和可见性。备用方法fullAddCount()则会死循环插入。

2）判断是否要扩容操作，并且支持多个线程协助进行扩容。
```
// 修改容量并判断是否要扩容。
2 private final void addCount(long x, int check) {
3 CounterCell[] as; long b, s;
4 // counterCells不为null，或者使用CAS对baseCount增加失败了，说明产生了并发，需要进一步处理。
5 // counterCells初始为null，如果不为null，说明产生了并发。
6 // 如果counterCells仍然为null，但是在使用CAS对baseCount增加的时候失败，表示产生了并发。
7 if ((as = counterCells) != null || !U.compareAndSwapLong(this, BASECOUNT, b = baseCount, s = b + x)) {
8 CounterCell a; long v; int m;
9 boolean uncontended = true;
10 // 如果counterCells是null的，或者counterCells的个数小于0。
11 // 或者counterCells的每一个元素都是null。
12 // 或者用counterCells数组中随机位置的值进行累加也失败了。
13 if (as == null || (m = as.length - 1) < 0 ||
14 (a = as[ThreadLocalRandom.getProbe() & m]) == null ||
15 !(uncontended = U.compareAndSwapLong(a, CELLVALUE, v = a.value, v + x))) {
16 // 继续更新counterCells和baseCount。
17 fullAddCount(x, uncontended);
18 return;
19 }
20 // 删除或清理节点时是-1，插入索引首节点0，第二个节点是1。
21 if (check <= 1)
22 return;
23 // 计算map元素个数。
24 s = sumCount();
25 }
26 // 如果check的值大于等于0，需要检查是否要扩容。删除或清理节点时是-1，此时不检查。
27 if (check >= 0) {
28 Node<K,V>[] tab, nt; int n, sc;
29 // 当元素个数大于阈值，并且集合不为空，并且元素个数小于最大值。循环判断，防止多线程同时扩容跳过if判断。
30 while (s >= (long)(sc = sizeCtl) && (tab = table) != null && (n = tab.length) < MAXIMUM_CAPACITY) {
31 // 生成与n有关的标记，且n不变的情况下生成的一定是一样的。
32 int rs = resizeStamp(n);
33 // sc在单线程时是大于等于0的，如果小于0说明有其他线程正在扩容。
34 // 如果小于0说明有线程执行了else里面的判断，导致rs左移16位并在低位+2赋值给sc。
35 if (sc < 0) {
36 // 在第一次左移16位的sc，经过第二次右移16位之后，还和rs相同，说明已经扩容完成。
37 // 线程执行扩容，会使用CAS让sc自增，如果sc和右移并累加后的rs相等，说明已经扩容完成。
38 // 线程执行扩容，会使用CAS让sc自增，如果sc和右移并累加最大值后的rs相等，说明已经扩容完成。
39 // 如果下个节点是null，说明已经扩容完成。
40 // 如果transferIndex小于等于0，说明集合已完成扩容，无法再分配任务。
41 if ((sc >>> RESIZE_STAMP_SHIFT) != rs ||
42 sc == rs + 1 ||// 此处应为 sc == (rs << RESIZE_STAMP_SHIFT) + 1
43 sc == rs + MAX_RESIZERS ||// 此处应为 sc == (rs << RESIZE_STAMP_SHIFT) + MAX_RESIZERS
44 (nt = nextTable) == null ||
45 transferIndex <= 0)
46 // 跳出循环。
47 break;
48 // 使用CAS原子累加sc的值。
49 if (U.compareAndSwapInt(this, SIZECTL, sc, sc + 1))
50 // 扩容。
51 transfer(tab, nt);
52 }
53 // 如果sizeCtl大于或等于0，说明第一次扩容，并且使用CAS设置sizeCtl为rs左移后的负数，并且低位+2表示有2-1个线程正在扩容。
54 else if (U.compareAndSwapInt(this, SIZECTL, sc, (rs << RESIZE_STAMP_SHIFT) + 2))
55 // 进行扩容操作。
56 transfer(tab, null);
57 // 计算map元素个数，baseCount和counterCells数组存的总和。
58 s = sumCount();
59 }
60 }
61 }
```

### 帮助扩容方法
1）判断集合已经完成初始化，并且节点是ForwordingNode类型（表示正在扩容），并且当前节点的子节点不为null，如果不成立则不需要扩容。
2）循环判断是否扩容成功，如果没有就使用CAS原子操作累加扩容的线程数，并且进行协助扩容。
```
// 帮助扩容。
2 final Node<K,V>[] helpTransfer(Node<K,V>[] tab, Node<K,V> f) {
3 Node<K,V>[] nextTab; int sc;
4 // 如果表不为null，并且是fwd类型的节点，并且节点的子节点也不为null。
5 if (tab != null && (f instanceof ForwardingNode) && (nextTab = ((ForwardingNode<K,V>)f).nextTable) != null) {
6 // 得到标识符。
7 int rs = resizeStamp(tab.length);
8 // 如果nextTab没有被并发修改，并且tab也没有被并发修改，并且sizeCtl小于0说明还在扩容。
9 while (nextTab == nextTable && table == tab && (sc = sizeCtl) < 0) {
10 // 在第一次左移16位的sc，经过第二次右移16位之后，还和rs相同，说明已经扩容完成。
11 // 线程执行扩容，会使用CAS让sc自增，如果sc和右移并累加后的rs相等，说明已经扩容完成。
12 // 线程执行扩容，会使用CAS让sc自增，如果sc和右移并累加最大值后的rs相等，说明已经扩容完成。
13 // 如果transferIndex小于等于0，说明集合已完成扩容，无法再分配任务。
14 if ((sc >>> RESIZE_STAMP_SHIFT) != rs ||
15 sc == rs + 1 ||// 此处应为 sc == (rs << RESIZE_STAMP_SHIFT) + 1
16 sc == rs + MAX_RESIZERS ||// 此处应为 sc == (rs << RESIZE_STAMP_SHIFT) + MAX_RESIZERS
17 transferIndex <= 0)
18 // 跳出循环。
19 break;
20 // 使用CAS原子累加sc的值。
21 if (U.compareAndSwapInt(this, SIZECTL, sc, sc + 1)) {
22 // 扩容。
23 transfer(tab, nextTab);
24 break;
25 }
26 }
27 return nextTab;
28 }
29 return table;
30 }
```

### 扩容方法

1）根据CPU核心数平均分配给每个CPU相同大小的区间，如果不够16个，默认就是16个。

2）有且只能由一个线程构建一个nextTable，这个nextTable是扩容后的数组（容量已经扩大）。

3）外层使用for循环处理每个区间里的根节点，内层使用while循环让线程领取未扩容的区间。

4）处理每个区间的头节点，如果头结点为空，则直接放置一个ForwordingNode，通知其他线程帮助扩容。

5）处理每个区间的头节点，如果头结点不为空，并且hash不为-1，那么就同步头节点，开始扩容。判断头结点是链表还是红黑树：如果是链表，则拆分为高低两个链表。如果是红黑树，拆分为高低两个红黑树，并判断是否需要转为链表。

```
// 进行扩容操作。
2 private final void transfer(Node<K,V>[] tab, Node<K,V>[] nextTab) {
3 int n = tab.length, stride;
4 // 根据cpu个数找出扩容时的最小分组，最小是16。
5 if ((stride = (NCPU > 1) ? (n >>> 3) / NCPU : n) < MIN_TRANSFER_STRIDE)
6 stride = MIN_TRANSFER_STRIDE;
7 // 表示第一次扩容，因为在addCount()方法中，第一次扩容的时候传入的nextTab的值是null。
8 if (nextTab == null) {
9 try {
10 // 创建新的扩容后的节点数组。
11 @SuppressWarnings("unchecked")
12 Node<K,V>[] nt = (Node<K,V>[])new Node<?,?>[n << 1];
13 // 将新的数组赋值给nextTab。
14 nextTab = nt;
15 } catch (Throwable ex) {
16 // 扩容失败，设置sizeCtl为最大值。
17 sizeCtl = Integer.MAX_VALUE;
18 return;
19 }
20 // 将新的数组赋值给nextTable。
21 nextTable = nextTab;
22 // 记录要扩容的区间最大值，说明是逆序迁移，从高位向低位迁移。
23 transferIndex = n;
24 }
25 // 设置扩容后的容量。
26 int nextn = nextTab.length;
27 // 创建一个fwd节点，用于占位，fwd节点的hash默认为-1。当别的线程发现这个槽位中是fwd类型的节点，则跳过这个节点。
28 ForwardingNode<K,V> fwd = new ForwardingNode<K,V>(nextTab);
29 // 如果是false，需要处理区间上的当前位置，如果是true，说明需要处理区间上的下一个位置。
30 boolean advance = true;
31 // 完成状态，如果是true，就结束此方法。
32 boolean finishing = false;
33 // 死循环，i表示最大下标，bound表示最小下标。
34 for (int i = 0, bound = 0;;) {
35 Node<K,V> f; int fh;
36 // 循环判断是否要处理区间上的下一个位置，每个线程都会在这个循环里获取区间。
37 while (advance) {
38 int nextIndex, nextBound;
39 // i自减一并判断是否大于等于bound，以及是否已经完成了扩容。
40 // 如果i自减后大于等于bound并且未完成扩容，说明需要处理当前i位置上的节点，跳出while循环。
41 // 如果i自减后小于bound并且未完成扩容，说明区间上没有节点需要处理，在while循环里继续判读。
42 // 如果已经完成扩容，跳出while循环。
43 if (--i >= bound || finishing)
44 // 跳出while循环。
45 advance = false;
46 // 如果要扩容的区间最大值小于等于0，说明没有区间需要扩容了。
47 else if ((nextIndex = transferIndex) <= 0) {
48 // i会在下面的if块里判断，从而进入完成状态判断。
49 i = -1;
50 // 跳出while循环。
51 advance = false;
52 }
53 // 首次while循环进入，CAS判断transferIndex和nextIndex是否一致，将transferIndex修改为最大值。
54 else if (U.compareAndSwapInt(this, TRANSFERINDEX, nextIndex,
55 nextBound = (nextIndex > stride ? nextIndex - stride : 0))) {
56 // 当前线程处理区间的最小下标。
57 bound = nextBound;
58 // 初次对i赋值，当前线程处理区间的最大下标。
59 i = nextIndex - 1;
60 // 跳出while循环。
61 advance = false;
62 }
63 }
64 // 判读是否完成扩容。
65 // 如果i小于0，表示已经处理了最后一段空间。
66 // 如果i大于等于原容量，表示超过下标最大值。
67 // 如果i加上原容量大于等于新容量，表示超过下标最大值。
68 if (i < 0 || i >= n || i + n >= nextn) {
69 int sc;
70 // 如果完成扩容，finishing为true，表示最后一个线程完成了扩容。
71 if (finishing) {
72 // 删除成员变量。
73 nextTable = null;
74 // 更新集合。
75 table = nextTab;
76 // 更新阈值。
77 sizeCtl = (n << 1) - (n >>> 1);
78 return;
79 }
80 // 如果没完成扩容，当前线程完成这段区间的扩容，将sc的低16位减1。
81 if (U.compareAndSwapInt(this, SIZECTL, sc = sizeCtl, sc - 1)) {
82 // 如果判断是否是最后一个扩容线程，如果不等于，说明还有其他线程在扩容，当前线程返回。
83 if ((sc - 2) != resizeStamp(n) << RESIZE_STAMP_SHIFT)
84 return;
85 // 如果相等，说明当前最后一个线程完成扩容，扩容结束，并再次进入while循环检查一次。
86 finishing = advance = true;
87 // 再次循环检查一下整张表。
88 i = n;
89 }
90 }
91 // 正常处理区间，如果原数组i位置是null，就使用fwd占位。
92 else if ((f = tabAt(tab, i)) == null)
93 // 如果成功写入fwd占位，进入while循环，继续处理区间的下一个节点。
94 advance = casTabAt(tab, i, null, fwd);
95 // 正常处理区间，如果原数组i位置不是null，并且hash值是-1，说明别的线程已经处理过了。
96 else if ((fh = f.hash) == MOVED)
97 // 进入while循环，继续处理区间的下一个节点。
98 advance = true;
99 // 到这里，说明这个位置有实际值了，且不是占位符。
100 else {
101 // 对这个节点上锁，防止添加元素的时候向链表插入数据。
102 synchronized (f) {
103 // 判断i下标处的桶节点是否和f相同，二次校验。
104 if (tabAt(tab, i) == f) {
105 // 声明高位桶和低位桶。
106 Node<K,V> ln, hn;
107 // 如果f的hash值大于0，表示是链表结构。红黑树的hash默认是-2。
108 if (fh >= 0) {
109 // 获取原容量最高位同节点hash值的与运算结果，用来判断将该节点放到高位还是低位。
110 int runBit = fh & n;
111 // 定义尾节点，暂时取f节点，后面会更新。
112 Node<K,V> lastRun = f;
113 // 遍历这个节点。
114 for (Node<K,V> p = f.next; p != null; p = p.next) {
115 // 获取原容量最高位同节点hash值的与运算结果，用来判断将该节点放到高位还是低位。
116 int b = p.hash & n;
117 // 如果节点的hash值和首节点的hash值，同原容量最高位与运算的结果不同。
118 if (b != runBit) {
119 // 更新runBit，用于下面判断lastRun该赋值给ln还是hn。
120 runBit = b;
121 // 更新lastRun，保证后面的节点与自己的取于值相同，避免后面没有必要的循环。
122 lastRun = p;
123 }
124 }
125 // 如果最后更新的runBit是0，设置低位节点。
126 if (runBit == 0) {
127 ln = lastRun;
128 hn = null;
129 }
130 // 如果最后更新的runBit是1，设置高位节点。
131 else {
132 hn = lastRun;
133 ln = null;
134 }
135 // 再次循环，生成两个链表，lastRun作为停止条件，这样就是避免无谓的循环。
136 for (Node<K,V> p = f; p != lastRun; p = p.next) {
137 int ph = p.hash; K pk = p.key; V pv = p.val;
138 // 如果与运算结果是0，那么创建低位节点。
139 if ((ph & n) == 0)
140 ln = new Node<K,V>(ph, pk, pv, ln);
141 // 如果与运算结果是1，那么创建高位节点。
142 else
143 hn = new Node<K,V>(ph, pk, pv, hn);
144 }
145 // 设置低位链表，放在新数组的i位置。
146 setTabAt(nextTab, i, ln);
147 // 设置高位链表，放在新数组的i+n位置。
148 setTabAt(nextTab, i + n, hn);
149 // 将旧的链表设置成fwd占位符。
150 setTabAt(tab, i, fwd);
151 // 继续处理区间的下一个节点。
152 advance = true;
153 }
154 // 如果是红黑树结构。
155 else if (f instanceof TreeBin) {
156 TreeBin<K,V> t = (TreeBin<K,V>)f;
157 TreeNode<K,V> lo = null, loTail = null;
158 TreeNode<K,V> hi = null, hiTail = null;
159 int lc = 0, hc = 0;
160 // 遍历。
161 for (Node<K,V> e = t.first; e != null; e = e.next) {
162 int h = e.hash;
163 TreeNode<K,V> p = new TreeNode<K,V>(h, e.key, e.val, null, null);
164 // 与运算结果为0的放在低位。
165 if ((h & n) == 0) {
166 if ((p.prev = loTail) == null)
167 lo = p;
168 else
169 loTail.next = p;
170 loTail = p;
171 ++lc;
172 }
173 // 与运算结果为1的放在高位。
174 else {
175 if ((p.prev = hiTail) == null)
176 hi = p;
177 else
178 hiTail.next = p;
179 hiTail = p;
180 ++hc;
181 }
182 }
183 // 如果树的节点数小于等于6，那么转成链表，反之，创建一个新的树。
184 ln = (lc <= UNTREEIFY_THRESHOLD) ? untreeify(lo) : (hc != 0) ? new TreeBin<K,V>(lo) : t;
185 // 如果树的节点数小于等于6，那么转成链表，反之，创建一个新的树。
186 hn = (hc <= UNTREEIFY_THRESHOLD) ? untreeify(hi) : (lc != 0) ? new TreeBin<K,V>(hi) : t;
187 // 设置低位树，放在新数组的i位置。
188 setTabAt(nextTab, i, ln);
189 // 设置高位数，放在新数组的i+n位置。
190 setTabAt(nextTab, i + n, hn);
191 // 将旧的树设置成fwd占位符。
192 setTabAt(tab, i, fwd);
193 // 继续处理区间的下一个节点。
194 advance = true;
195 }
196 }
197 }
198 }
199 }
200 }
```

### 获取方法
根据指定的键，返回对应的键值对，由于是读操作，所以不涉及到并发问题，步骤如下：
1）判断查询的key对应数组的首节点是否null。
2）先判断数组的首节点是否为寻找的对象。
3）如果首节点不是，并且是红黑树结构，另做处理。
4）如果是链表结构，遍历整个链表查询。
5）如果都不是，那就返回null值。
```
// 获取元素。
2 public V get(Object key) {
3 Node<K,V>[] tab; Node<K,V> e, p; int n, eh; K ek;
4 // 计算hash，并保证hash一定大于零，负数表示在扩容或者是树节点。
5 int h = spread(key.hashCode());
6 // 如果集合不为null，并且集合长度大于0，并且指定位置上的元素不为null。
7 if ((tab = table) != null && (n = tab.length) > 0 && (e = tabAt(tab, (n - 1) & h)) != null) {
8 // 如果hash相等。
9 if ((eh = e.hash) == h) {
10 // 如果首节点是要找的元素。
11 if ((ek = e.key) == key || (ek != null && key.equals(ek)))
12 return e.val;
13 }
14 // 如果正在扩容或者是树节点。
15 else if (eh < 0)
16 // 尝试查找元素，找到返回元素，找不到返回null。
17 return (p = e.find(h, key)) != null ? p.val : null;
18 // 如果不是首节点，则遍历集合查找。
19 while ((e = e.next) != null) {
20 if (e.hash == h && ((ek = e.key) == key || (ek != null && key.equals(ek))))
21 return e.val;
22 }
23 }
24 return null;
25 }
```

### 删除方法
删除操作，可以看成是用null替代原来的节点，因此合并在这个方法中，由这个方法一起实现删除操作和替换操作。
replaceNode()方法中的三个参数，key表示想要删除的键，value表示想要替换的元素，cv表示想要删除的key对应的值。
```
// 删除元素。
2 public V remove(Object key) {
3 return replaceNode(key, null, null);
4 }
5
6 // 删除元素
7 final V replaceNode(Object key, V value, Object cv) {
8 // 计算hash，并保证hash一定大于零，负数表示在扩容或者是树节点。
9 int hash = spread(key.hashCode());
10 // CAS经典写法，不成功无限重试。
11 for (Node<K,V>[] tab = table;;) {
12 Node<K,V> f; int n, i, fh;
13 // 如果集合是null，或者集合长度是0，或者指定位置上的元素是null。
14 if (tab == null || (n = tab.length) == 0 || (f = tabAt(tab, i = (n - 1) & hash)) == null)
15 // 跳出循环。
16 break;
17 // 如果获取到节点不为null，并且节点的hash为-1，则表示节点在扩容。
18 else if ((fh = f.hash) == MOVED)
19 // 帮助扩容。
20 tab = helpTransfer(tab, f);
21 // 产生hash碰撞，并且没有扩容操作。
22 else {
23 V oldVal = null;
24 // 是否进入了同步代码。
25 boolean validated = false;
26 // 锁住节点。
27 synchronized (f) {
28 // 这里volatile获取首节点与节点对比判断节点还是不是首节点。
29 if (tabAt(tab, i) == f) {
30 // 判断是否是链表节点。
31 if (fh >= 0) {
32 validated = true;
33 // 循环查找指定元素。
34 for (Node<K,V> e = f, pred = null;;) {
35 K ek;
36 // 找到元素了。
37 if (e.hash == hash && ((ek = e.key) == key || (ek != null && key.equals(ek)))) {
38 V ev = e.val;
39 // 如果cv为null，或者cv不为null时cv和指定元素上的值相同，才更新或者删除节点。
40 if (cv == null || cv == ev || (ev != null && cv.equals(ev))) {
41 oldVal = ev;
42 // 如果新值不为null，替换。
43 if (value != null)
44 e.val = value;
45 // 如果新值是null，并且当前节点非首结点，删除。
46 else if (pred != null)
47 pred.next = e.next;
48 // 如果新值是null，并且当前节点是首结点，删除。
49 else
50 setTabAt(tab, i, e.next);
51 }
52 break;
53 }
54 pred = e;
55 // 如果遍历集合也没有找到。
56 if ((e = e.next) == null)
57 // 跳出循环。
58 break;
59 }
60 }
61 // 如果是红黑树节点。
62 else if (f instanceof TreeBin) {
63 validated = true;
64 TreeBin<K,V> t = (TreeBin<K,V>)f;
65 TreeNode<K,V> r, p;
66 // 找到元素了。
67 if ((r = t.root) != null && (p = r.findTreeNode(hash, key, null)) != null) {
68 V pv = p.val;
69 // 如果cv为null，或者cv不为null时cv和指定元素上的值相同，才更新或者删除节点。
70 if (cv == null || cv == pv || (pv != null && cv.equals(pv))) {
71 oldVal = pv;
72 if (value != null)
73 p.val = value;
74 else if (t.removeTreeNode(p))
75 setTabAt(tab, i, untreeify(t.first));
76 }
77 }
78 }
79 }
80 }
81 // 如果进入了同步代码。
82 if (validated) {
83 // 如果更新或者删除了节点。
84 if (oldVal != null) {
85 // 如果value为null，说明是删除操作。
86 if (value == null)
87 // 将数组长度减一。
88 addCount(-1L, -1);
89 return oldVal;
90 }
91 break;
92 }
93 }
94 }
95 return null;
96 }
```

### 计算集合容量
ConcurrentHashMap中baseCount用于保存tab中元素总数，但是并不准确，因为多线程同时增删改，会导致baseCount修改失败，此时会将元素变动存储于counterCells数组内。
当需要统计当前的size的时候，除了要统计baseCount之外，还需要加上counterCells中的每个桶的值。
值得一提的是即使如此，统计出来的依旧不是当前tab中元素的准确值，在多线程环境下统计前后并不能暂停线程操作，因此无法保证准确性。
```
// 计算集合容量。
2 public int size() {
3 long n = sumCount();
4 return ((n < 0L) ? 0 : (n > (long)Integer.MAX_VALUE) ? Integer.MAX_VALUE : (int)n);
5 }
6
7 // 计算集合容量，baseCount和counterCells数组存的总和。
8 final long sumCount() {
9 CounterCell[] as = counterCells; CounterCell a;
10 long sum = baseCount;
11 if (as != null) {
12 for (int i = 0; i < as.length; ++i) {
13 if ((a = as[i]) != null)
14 sum += a.value;
15 }
16 }
17 return sum;
18 }
```

Hashtable、Collections.synchronizedMap()、ConcurrentHashMap之间的区别Hashtable是线程安全的哈希表，它是通过synchronized来保证线程安全的；即，多线程通过同一个“对象的同步锁”来实现并发控制。Hashtable在线程竞争激烈时，效率比较低（此时建议使用ConcurrentHashMap）。当一个线程访问Hashtable的同步方法时，其它线程如果也在访问Hashtable的同步方法时，可能会进入阻塞状态。
Collections.synchronizedMap()使用了synchronized同步关键字来保证对Map的操作是线程安全的。
ConcurrentHashMap是线程安全的哈希表。在JDK1.7中它是通过“锁分段”来保证线程安全的，本质上也是一个“可重入的互斥锁”（ReentrantLock）。多线程对同一个片段的访问，是互斥的；但是，对于不同片段的访问，却是可以同步进行的。在JDK1.8中是通过使用CAS原子更新、volatile关键字、synchronized可重入锁实现的。

## 实践指南

高并发、任务执行时间短的业务怎样使用线程池？并发不高、任务执行时间长的业务怎样使用线程池？并发高、业务执行时间长的业务怎样使用线程池？
这是我在并发编程网上看到的一个问题，把这个问题放在最后一个，希望每个人都能看到并且思考一下，因为这个问题非常好、非常实际、非常专业。关于这个问题，个人看法是：
（1）高并发、任务执行时间短的业务，线程池线程数可以设置为CPU核数+1，减少线程上下文的切换
（2）并发不高、任务执行时间长的业务要区分开看：
　　a）假如是业务时间长集中在IO操作上，也就是IO密集型的任务，因为IO操作并不占用CPU，所以不要让所有的CPU闲下来，可以加大线程池中的线程数目，让CPU处理更多的业务
　　b）假如是业务时间长集中在计算操作上，也就是计算密集型任务，这个就没办法了，和（1）一样吧，线程池中的线程数设置得少一些，减少线程上下文的切换
（3）并发高、业务执行时间长，解决这种类型任务的关键不在于线程池而在于整体架构的设计，看看这些业务里面某些数据是否能做缓存是第一步，增加服务器是第二步，至于线程池的设置，设置参考（2）。最后，业务执行时间长的问题，也可能需要分析一下，看看能不能使用中间件对任务进行拆分和解耦。


