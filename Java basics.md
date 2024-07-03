## AIO、BIO和NIO区别？
BIO(Blocking I/O): **同步阻塞I/O**,是JDK1.4之前的传统I/O模型。线程发起I/O请求后，一直阻塞，直到缓冲区数据就绪后，再进入下一步操作。
NIO(Non-Blocking I/O): **同步非阻塞IO**，线程发起IO请求后，不需要阻塞，可以先进行下一步操作，只需要*定时轮询*[^1]检查IO缓冲区数据是否就绪即可。



[^1]: 定时轮询(Timed Polling)是指在预定的时间间隔内定期检查某个资源或者系统状态的过程，以检测是否有新的事件、消息或状态变化。这种机制常用于客户端与服务器之间的通信，以及实时数据处理和监控场景中。
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
