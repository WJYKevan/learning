## AIO、BIO和NIO区别？
BIO(Blocking I/O): **同步阻塞I/O**,是JDK1.4之前的传统I/O模型。*线程*[^2]发起*I/O请求*[^3]后，一直阻塞，直到缓冲区数据就绪后，再进入下一步操作。
NIO(Non-Blocking I/O): **同步非阻塞IO**，线程发起IO请求后，不需要阻塞，可以先进行下一步操作，只需要*定时轮询*[^1]检查IO缓冲区数据是否就绪即可。
AIO(Asynchronous I/O): **异步非阻塞IO**,线程发起IO请求后，不需要阻塞，立即返回，也不需要定时轮询检查结果，异步IO操作之后会回调通知调用方。

![bio'nio'aio](https://cdn.nlark.com/yuque/0/2024/png/5378072/1705133708567-49955e01-446a-4fef-b441-4356180eac5c.png?x-oss-process=image%2Fwatermark%2Ctype_d3F5LW1pY3JvaGVp%2Csize_22%2Ctext_SkFWQeWFq-iCoeaWh-ahow%3D%3D%2Ccolor_FFFFFF%2Cshadow_50%2Ct_80%2Cg_se%2Cx_10%2Cy_10)


### 为什么NIO需要轮询？
在NIO中，线程在发起i/o请求后不会被阻塞，这个线程可以继续执行其他的任务，但由于不知道i/o操作何时完成，线程需要定期的检查(轮询)
i/o操作的状态。这种轮询可以手动的视线，也可以通过事件驱动机制自动完成,如Java的Selector机制。轮询的目的是让线程能够及时响应i/o操作的完成情况，从而继续后续的操作。

### Java中的BIO、NIO、AIO分别适用哪些场景？
BIO方式适用于连接数目比较小且固定的架构，这种方式对服务器资源要求比较高，并发局限于应用中，由于每个i/o操作都会阻塞处理它的线程，服务器必须为每个客户端连接分配一个线程，以处理该连接上的所有i/o操作，意味着服务器的并发能力直接受限于系统能够提供的线程数目，JDK1.4以前的唯一选择，但是程序直观简单易理解。

NIO方式适用于连接数目比较多且连接比较短(轻操作)的架构，它允许多个线程共享一个或几个Selector(选择器)，Selector可以监听多个Channeld的I/O事件，不需要为每个连接分配一个独立的线程，从而减少线程上下文切换带来的开销，比如聊天服务器，并发局限于应用中。

AIO方式适用于连接数目比较多且连接比较长(重操作)的架构，比如相册服务器,充分调用os参与操作，编程比较复杂，jdk7开始支持。



[^1]:定时轮询(Timed Polling)是指在预定的时间间隔内定期检查某个资源或者系统状态的过程，以检测是否有新的事件、消息或状态变化。这种机制常用于客户端与服务器之间的通信，以及实时数据处理和监控场景中。
实现定时轮询的常见方法：
    - 使用`java.util.Timer`和`TimerTask`:这是Java标准库中提供的一个简单的计时器，可以通过创建Timer对象和实现TimerTask来安排周期性的任务。
    `Timer timer = new Timer();
    timer.schedule(new TimerTask(){
        public void run(){
            //轮询逻辑
        }
    },0,5000);//5秒执行一次`
    - 使用`ScheduledExecutorService`:这是Java并发包`java.util.concurrent`，提供更加强大的调度和更好的性能。
    `ScheduledExectorService scheduler = Exector.newSingleThreadScheduledExector();
    scheduler.scheduledAtfixedRate(()->{
        //轮询逻辑
    },0,5,TimeUnit.SECONDS);`
    - 使用Spring框架的定时任务:使用@Scheduled注解来定义定时任务。
    `@Component
    public class PollingTask{
        @Scheduled(fixedRate = 5000)
        public void pollData(){
            //轮询逻辑
        }
    }`
    定时轮询的缺点：可能产生不必要的网络请求和cpu消耗，尤其是在没有新数据的情况下频繁轮询;另外，如果目标系统的响应时间比较长，可能会导致轮询间隔的实际执行时间变长，影响轮询的准确性和效率。

[^2]:线程是操作系统能够进行运算调度的最小单位，它被包含在进程之中，是进程中实际运作单位。一个进程中可以并发多个线程，每条线程并行执行不同的任务。在多核处理器中，多个线程可以同时执行，这被称为真正的并行执行；而在单核处理器中，多个线程则是通过时间片轮换(系统将处理器的执行时间分割成一系列小的时间段，每个活动的线程都会获得一定数量的时间片，在这段时间是独占处理器资源；而超线程是一种硬件级别的多线程技术，它允许一个物理核心同时处理两个线程;任务管理器的CPU，内核数量代表它的核心，逻辑处理器数量代表线程)来实现并发执行的。

[^3]:IO请求是指输入输出设备(如磁盘、网络接口卡等)与内存之间的数据传输请求。例如，当你访问一个网站时，你的浏览器向服务器发送请求，这就是一个网络i/o请求;当你的程序读取硬盘上的文件时，这也是一个磁盘io请求。


 