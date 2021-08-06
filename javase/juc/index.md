# JUC
**JUC：**在并发编程中使用的工具类

```java

class Ticket{
    private int number = 30;

    private Lock lock = new ReentrantLock();

    public void saleTicket(){
        try{
            lock.lock();
            if(number>0){
                System.out.println(Thread.currentThread().getName()+"\t卖出第"+（number--）);
            }
        }finally{
            lock.unlock();
        }
        
    }
}

/**
    在高内聚低耦合的前提下，线程操作资源类
*/
public class SaleTicket{
    public static void main(String[] args){
        Ticket tiket = new Ticket();
        new Thread(()->{tick.saleTicket()},"Thread1").start();
        new Thread(()->{tick.saleTicket()},"Thread2").start();
        new Thread(()->{tick.saleTicket()},"Thread3").start();
    }
}

```

Java多线程有几种状态：NEW,RUNNABLE,BLOCKED,WAITING,TIME_WAITING,TERMINATED

wait/sleep的区别？
功能都是当前线程暂停，wait放开手里的锁，Sleep是不放开手里的锁。

## 线程中的通信

* 生产者与消费者
 
 ```java
//Asynchronized
Class AirConditioner{
    private int number = 0;
    public synchronized void increment(){
        // 判断
        while(number!=0){
            this.wait();
        }
        number++;
        // 通知
        this.notifyAll();
    }
    public synchronized void decrement(){
        while(number==0){
            this.wait();
        }
        number--;
        this.notifyAll();
    }
}

Class AirConditioner{
    private int number = 0;
    private Lock lock = new ReentrantLock();
    private Condition condition = lock.newCondition();

    public void increment(){
        while(number!=0){
            condition.await();
        }
        number++;
        condition.signalAll();
    }
    public void decrement(){
        while(number==0){
            condition.await();
        }
        number--;
        condition.singalAll();
    }
}




/**
    现在两个线程，可以操作初始值为零的一个变量，实现一个线程对该变量加1，一个线程对该变量减1，实现交替，来10变量初始值为0
    判断/干活/通知
    多线程交互中，必须防止多线程中的虚假唤醒，也就是说判断只能使用（while而不是if）
*/
public class ThreadWaitNotifyDemo{
    public static void main(String[] args)throws Excpetion{
             AirConditioner aircondition = new AirConditioner();
             new Thread(()->{
                 for(int i=1;i<10;i++){
                     aircondition.increment();
                 }
             },"A").start();
            new Thread(()->{
                for(int i=1;i<=10;i++){
                    aircondition.decrement();
                }
            },"B").start();
    }
}

 ```

 ```java

/**
    精确控制。

    多个线程之间按顺序调用，实现A->B->C
    三个线程启动，要求如下：
    * AA打印5次，BB打印10次，CC打印15次
    这样来10轮

    注意标志位的修改
*/

public class print{
    private int number = 1; // 1:A 2:B 3:C
    private Lock lock = new ReentrantLock();
    private Condition condition1 = lock.newCondition();
    private Condition condition2 = lock.newCondition();
    private Condition condition3 = lock.newCondition();

    public void pritnA(){
        lock.lock();
        try{
            while(number!=1){
                considtion1.await();
            }   

            for(int i =1;i<=5;i++){
                System.out.println();
            }
            number = 2;
            consition2.signal();
        }finally{
            lock.unlock();
        }
    }
        public void pritnB(){
        lock.lock();
        try{
            while(number!=2){
                considtion1.await();
            }   

            for(int i =1;i<=10;i++){
                System.out.println();
            }
            number = 3;
            consition3.signal();
        }finally{
            lock.unlock();
        }
    }
        public void pritnB(){
        lock.lock();
        try{
            while(number!=3){
                considtion1.await();
            }   

            for(int i =1;i<=15;i++){
                System.out.println();
            }
            number = 1;
            consition1.signal();
        }finally{
            lock.unlock();
        }
    }
}

 ```

 # 锁
  TimeUtil //新的执行类

  ```java
/**
    1. 标准访问，请求先打印邮件还是短信 （不一定）
    2. 邮件方法暂停4秒钟，请求先打印邮件还是短信 ()
    3. 新增一个普通方法hello，请求先打印邮件还是hell
    4. 两部手机，请问先打印邮件还是短信
    5. 两个静态同步方法，同一部手机，请问先打印邮件还是短信
    6. 两个静态同步方法，2部手机，请问先打印邮件还是短信
    7. 1个普通同步方法，1个静态同步方法，1部手机，请问先打印邮件还是短信
    8. 1个普通同步方法，1个静态不同方法，2部手机，请问先打印邮件还是短信

    一个对象里面如果有多个synchronized方法，某一个时刻内，只要一个线程去调用其中的一个synchronized方法了，其他的线程都只能等待，换句话说，某个时刻内，只能有唯一一个线程去访问这些synchronized方法，锁的是当前对象this，被锁定之后，其他的线程都不能进入到当前对象的其他的synchronized方法。

    加个普通方法后发现和同步锁无关，换成两个对象后，不是同一把锁，情况立即变化

    都换成静态同步方法后，情况又改变了。


    
    所有的非静态同步方法用的都是同一把锁-实例对象本身

*/

    class Phone{
        public synchronized void sendEmail()throws Exeption{
            //TimeUnit.SECONDS.sleep(4);
            System.out.println("----------sendEmail");
        }
        public synchronized void sendSMS()trhows Exception{
            System.out.println("----------sendSMS");
        }
        public void hello(){
            System.out.println("----------hello");
        }
    }
    public class lock8{
        public static void main(String[] args) throws Exception{
            Phone phone = new Phone();
            //Phone phone2 = new Phone();

            new Thread(()->{
                try{
                    phone.sendEmail();
                }catch(Exception e){
                    e.printStackTrace();
                }
            },"A").start();

            new Thread(()->{
                try{
                    phone.sendSMS();
                    //phone2.sendSMS();
                }catch(Exception e){
                    e.printStackTrace();
                }
            },"B").start();
        }
    }
  ```


  ```java
    /**
        故障：java.util.concurrentModificationException

        导致原因

        解决方案：
            vector
            Collection.synchronziedList(new ArrayList<>());
            CopyOnWriteArrayList 
        举例说明集合类是不安全的
        vector线程是安全的，ArrayList是不安全的
    */
    public class NotSafeDemo{
        public static void main(String[] args){
            List<String> list = Arrays.asList("a","b","c");
            list.forEach(System.out::println);
        }
    }
    /**
        * 写时复制
        CopyOnWrite容器即写时复制的容器，往一个容器添加元素的时候，不直接往当前容器Object[]添加，而是先将当前容器Object[]进行copu，在讲原容易的引用指向新的容器setArray。这样做的好处是可以对CopyOnWrite容器进行并发的读，而不需要加锁，因为当前容器不会添加任何元素，所以CopyOnWrite容器也是一种读写分离的思想，读和写不同的容器
    */
    public boolean add(E e){
        final ReentrantLock lock = this.lock;
        lock.lock();
        try{
            object[] elements = getArray();
            int len = elemets.length;
            Object[] newElements = Arrays.copyof(elements,len+1);
            newElements[len] = e;
            setArray(newElements);
            return true;
        }finally{
            lock.unlock();
        }
    }
    /**
        CopyOnWriteArraySet
    */
  ```

  ```java
    class MyThread implements Runnable{
        public void run(){

        }
    }
    class MyThread2 implements Callable<Integer>{
        @Overrible
        public void call() throws Exception{
            System.out.println("*******come in here");
            return 1024;
        }
    }
    /**
        多线程中，第3种获得多线程的方法
    */
    public class CallableDemo{
        public static void main(String[] args) throws InterruptedException,ExecutionException{
            FutureTask futureTask = new FutureTask(new MyThread());
            new Thread(futureTask,"A").start();
            // 获得返回值 get会阻塞当前线程，直到计算线程计算完全。
            futureTask.get();
        }
    }
  ```