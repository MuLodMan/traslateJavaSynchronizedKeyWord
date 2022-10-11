[原文](http://tutorials.jenkov.com/java-concurrency/synchronized.html)

   <pre>Java同步块用于标注方法或者代码块是同步的。Java同步块可以被用于避免竞争条件的产生。</pre>

## Java synchronized 关键字

在Java中使用synchronized关键字来表明同步块。同步块可以使某对象保持同步（一致性）。在同一个对象中同一时刻一个只允许一个线程访问同步块中的代码。其他尝试访问该代码块的线程将会被阻塞直到当前代码块中的线程将锁释放。

   synchronized关键字可以用在下面的4种代码块上:

    1. 实例方法
    2. 静态方法
    3. 实例方法中的代码块
    4. 静态方法中的代码块

   这些同步块在不同的对象中才会同步。具体需要那种同步块取决于具体情况

## synchronized 实例方法

下面是一个实例方法：
```Java
 public synchronized void add(int value){
      this.count += value;
  }
```
  注意使用synchronized关键字来告诉编译器该方法是同步的。
  一个同步的实例方法在拥有它的对象中是同步的。因此在不同的对象中每一个实例都有一个属于它自己的实例方法。只有一个线程可以访问实例方法。如果多个实例存在,那么在每个实例中某一时刻只有一个线程可以执行同步方法。

 ## 同步的静态方法：
 静态方法使用方法和实例方法一样,使用synchronized关键字。Java的静态同步方法如下。
 ```Java
  public static synchronized void add(int value){
      count += value;
  }
 ```
 同样,这里的静态关键字告诉编译器该方法是静态方法。
 静态同步方法会在该方法所属类的类对象上进行同步,因为每个类在Java虚拟机中只有一个类对象,所以在同一个类中只有一个线程可以执行该方法。

 如果该静态方法在不同的类中被调用,那么在各个类中只有一个线程可以执行该同步静态方法。一个线程只能对应一个类，无论该同步静态方法是被谁调用的。

 ## 实例方法中的同步代码块：
你不必同步整个函数。有时同步函数中的一部分是更好的方案。方法中的Java同步块使其成为可能。
如下是同步代码块在非同步的方法中：
```Java
 public void add(int value){

    synchronized(this){
       this.count += value;   
    }
  }
```
该例子使用synchronized关键字标记该段代码为同步代码。这段代码就可以像同步方法一样运行。

注意到block代码构造体的括号中放置一个对象。在这个例子中"this"代表当前方法被哪个对象锁调用。该block代码构造体括号中的对象被称为监听对象。也就是说该段代码是否同步取决于该对象。一个同步的实例方法使用该方法所属的对象作为监听对象。

对于同一个监听对象而言只有一个线程可以执行代码块。

接下来的两个例子都是都是在调用它们的函数上同步的。因此它们各自的同步方法都是等价的。
```Java
  public class MyClass {
  
    public synchronized void log1(String msg1, String msg2){
       log.writeln(msg1);
       log.writeln(msg2);
    }

  
    public void log2(String msg1, String msg2){
       synchronized(this){
          log.writeln(msg1);
          log.writeln(msg2);
       }
    }
  }
```
因此只有一个线程可以执行这两段代码。

如果此时有其他对象而不是this是监听对象，那么就可以有一个线程访问该代码块。

## 同步的静态代码
以下两个例子作为静态同步方法。这些方法会对该方法所对应的类对象进行同步。
```Java
  public class MyClass {

    public static synchronized void log1(String msg1, String msg2){
       log.writeln(msg1);
       log.writeln(msg2);
    }

  
    public static void log2(String msg1, String msg2){
       synchronized(MyClass.class){
          log.writeln(msg1);
          log.writeln(msg2);  
       }
    }
  }
```
在同一时刻只有一个线程可以执行这两种方法。

如果第二个同步代码块被其他非MyClass.class对象所监听,那么某个线程就可以同时访问这两种方法。

##Java同步示例
下面的实例中会开启两个线程,并且这两个线程都会调用Counter类同一个对象的add方法。那么同时只有一个线程可以调用同一个实例的add方法,因为该方法只对属于它的对象同步。
```Java
  public class Counter{
     
     long count = 0;
    
     public synchronized void add(long value){
       this.count += value;
     }
  }

```
```Java
  public class CounterThread extends Thread{

     protected Counter counter = null;

     public CounterThread(Counter counter){
        this.counter = counter;
     }

     public void run() {
	for(int i=0; i<10; i++){
           counter.add(i);
        }
     }
  }
```
```Java
  public class Example {

    public static void main(String[] args){
      Counter counter = new Counter();
      Thread  threadA = new CounterThread(counter);
      Thread  threadB = new CounterThread(counter);

      threadA.start();
      threadB.start(); 
    }
  }
```
创建两个线程。将Counter类的同一个实例放在这两个实例的构造器中。那么Counter.add方法会在该实例上同步,因为add方法是一个被synchronized标记的实例方法。因此在任意时刻只有一个线程可以调用add方法。其他线程都会等待直到当前线程离开add方法。

如果两个线程使用不同的Counter类的实例，那么调用add方法则没有任何问题。因为这些调用是不同的对象，所以这些方法只对各自的对象同步。因此调用并不会造成阻塞，请看如下代码：
```Java
  public class Example {

    public static void main(String[] args){
      Counter counterA = new Counter();
      Counter counterB = new Counter();
      Thread  threadA = new CounterThread(counterA);
      Thread  threadB = new CounterThread(counterB);

      threadA.start();
      threadB.start(); 
    }
  }
```
注意到线程A和线程B不再使用相同的counter对象。counterA和counterB的add方法只对自身的实例才会同步。因此调用counterA的add方法并不会阻塞调用counterB的add方法。

  
