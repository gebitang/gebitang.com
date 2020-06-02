+++
title = "kill -TERM Spring Boot应用"
description = "how to shutdown spring boot app gracefully "
tags = [
    "spring boot",
    "tech"
]
date = "2020-05-09"
topics = [
    "tech"
]
toc = true
+++

### 有关signals

`kill -INT $pid` sends the "interrupt" signal to the process with process ID `pid`.  
`kill -9 $pid` sends the "kill" signal which cannot be caught or ignored.   

`kill -INT $pid` is the same as `kill -2 $pid`.  
`kill -9 $pid` is the same as `kill -KILL $pid`

[常见信号](https://unix.stackexchange.com/a/399440) 、[wiki signals](https://en.wikipedia.org/wiki/Signal_(IPC)#POSIX_signals)


```
➜  ~ /bin/kill -l
HUP INT QUIT ILL TRAP ABRT BUS FPE KILL USR1 SEGV USR2 PIPE ALRM TERM STKFLT
CHLD CONT STOP TSTP TTIN TTOU URG XCPU XFSZ VTALRM PROF WINCH POLL PWR SYS


 1 HUP      2 INT      3 QUIT     4 ILL      5 TRAP     6 ABRT     7 BUS
 8 FPE      9 KILL    10 USR1    11 SEGV    12 USR2    13 PIPE    14 ALRM
15 TERM    16 STKFLT  17 CHLD    18 CONT    19 STOP    20 TSTP    21 TTIN
22 TTOU    23 URG     24 XCPU    25 XFSZ    26 VTALRM  27 PROF    28 WINCH
29 POLL    30 PWR     31 SYS     

```
[信号列表](https://www.pearsonitcertification.com/articles/article.aspx?p=1246308&seqNum=4)  
Table Signals Available Under Solaris  

|Signal|Number|Description|
|---------|---------|---------|
|SIGHUP|1|Hangup. Usually means that the controlling terminal has been disconnected.|
|SIGINT|2|Interrupt. The user can generate this signal by pressing Ctrl+C or Delete.|
|SIGQUIT|3|Quits the process and produces a core dump.|
|SIGILL|4|Illegal instruction.|
|SIGTRAP|5|Trace or breakpoint trap.|
|SIGABRT|6|Abort.|
|SIGEMT|7|Emulation trap.|
|SIGFPE|8|Arithmetic exception. Informs a process of a floating-point error.|
|SIGKILL|9|Killed. Forces the process to terminate. This is a sure kill.|
|SIGBUS|10|Bus error.|
|SIGSEGV|11|Segmentation fault.|
|SIGSYS|12|Bad system call.|
|SIGPIPE|13|Broken pipe.|
|SIGALRM|14|Alarm clock.|
|SIGTERM|15|Terminated. A gentle kill that gives processes a chance to clean up.|
|SIGUSR1|16|User signal 1.|
|SIGUSR2|17|User signal 2.|
|SIGCHLD|18|Child status changed.|
|SIGPWR|19|Power fail or restart.|
|SIGWINCH|20|Window size change.|
|SIGURG|21|Urgent socket condition.|
|SIGPOLL|22|Pollable event.|
|SIGSTOP|23|Stopped (signal). Pauses a process.|
|SIGTSTP|24|Stopped (user).|
|SIGCONT|25|Continued.|
|SIGTTIN|26|Stopped (tty input).|
|SIGTTOU|27|Stopped (tty output).|
|SIGVTALRM|28|Virtual timer expired.|
|SIGPROF|29|Profiling timer expired.|
|SIGXCPU|30|CPU time limit exceeded.|
|SIGXFSZ|31|File size limit exceeded.|
|SIGWAITING|32|Concurrency signal reserved by threads library.|
|SIGLWP|33|Inter-LWP signal reserved by threads library.|
|SIGFREEZE|34|Checkpoint freeze.|
|SIGTHAW|35|Checkpoint thaw.|
|SIGCANCEL|36|Cancellation signal reserved by the threads library.|

`docker stop` 实际是发送 SIGTERM 指令即kill -15。该命令会把此信号发送给容器内`pid`为`1`的进程，因此需要保证服务的主进程是第一个进程。如果使用脚本启动服务或者在启动服务之前有其他动作，那么第一个进程是很可能不是服务主进程，从而导致无法优雅下线。这种情况下需要在主进程启动命令前面加上 `exec` 关键字来主进程的 id 为 1，例如: `cd /data/www/app && exec java -jar app.jar`

### spring boot shutdown

当 Java 主线程接收到 Kill -15 信号时，SpringBoot 的 MVC 框架本身会保证服务的平滑关闭。内建的Tomcat最多等待5秒，如果5秒内处理不完将会强制关闭。可以做如下修改以延长时间。

```java
    /** 下面三个方法将会修改内置Tomcat的关闭信号量等到时间到30s **/
    @Bean
    public GracefulShutdown gracefulShutdown() {
        return new GracefulShutdown();
    }

    @Bean
    public ConfigurableServletWebServerFactory webServerFactory(final GracefulShutdown gracefulShutdown) {
        TomcatServletWebServerFactory factory = new TomcatServletWebServerFactory();
        factory.addConnectorCustomizers(gracefulShutdown);
        return factory;
    }

    public class GracefulShutdown implements TomcatConnectorCustomizer, ApplicationListener<ContextClosedEvent> {
        private Connector connector;

        @Override
        public void customize(Connector connector) {
            this.connector = connector;
        }

        @Override
        public void onApplicationEvent(ContextClosedEvent event) {
            this.connector.pause();
            Executor executor = this.connector.getProtocolHandler().getExecutor();
            if (executor instanceof ThreadPoolExecutor) {
                try {
                    ThreadPoolExecutor threadPoolExecutor = (ThreadPoolExecutor) executor;
                    threadPoolExecutor.shutdown();
                    if (!threadPoolExecutor.awaitTermination(30, TimeUnit.SECONDS)) {
                        log.warn("Tomcat thread pool did not shut down gracefully within "
                                + "30 seconds. Proceeding with forceful shutdown");
                    }
                } catch (InterruptedException ex) {
                    Thread.currentThread().interrupt();
                }
            }
        }
    }
```

--- 

>`docker stop` 实际是发送 SIGTERM 指令即kill -15。

主程序中可以添加ShutdownHook。

```java
Thread mainThread = Thread.currentThread();
//register a hook for Shutdown (signal term same as kill -15)
ShutdownHook thread = new ShutdownHook(mainThread, 5000L);
Runtime.getRuntime().addShutdownHook(thread);
SpringApplication.run(Application.class, args);
```
- 为虚拟机JVM注册一个shutdown类型的钩子hook
- 钩子的含义为：初始化但未执行的线程
- 此线程会在JVM关闭时执行

JVM关闭的场景：  
- 正常关闭，例如最后一个非守护类型的线程退出时，或是System.exit调用时
- JVM收到了用户的打断interrupt，比如 `ctrl+C` (=`^C`)或 `SIGTERM`

hook注意事项：符合shutdown的场景，例如尽量执行可以很快结束的任务、非人机交互的任务等。

--- 

官方doc：

Registers a new virtual-machine shutdown hook. 

The Java virtual machine shuts down in response to two kinds of events:
- The program exits normally, when the last non-daemon thread exits or when the exit (equivalently, System.exit) method is invoked, or 
- The virtual machine is terminated in response to a user interrupt, such as typing ^C, or a system-wide event, such as user logoff or system shutdown.

A shutdown hook is simply an initialized but unstarted thread. 

When the virtual machine begins its shutdown sequence it will start all registered shutdown hooks in some **unspecified order** and let them run concurrently. When all the hooks have finished it will then run all uninvoked finalizers if finalization-on-exit has been enabled. 

Finally, the virtual machine will halt. Note that daemon threads will continue to run during the shutdown sequence, as will non-daemon threads if shutdown was initiated by invoking the exit method.

Once the shutdown sequence has begun it can be stopped only by invoking the halt method, which forcibly terminates the virtual machine.

Once the shutdown sequence has begun it is impossible to register a new shutdown hook or de-register a previously-registered hook.

Attempting either of these operations will cause an IllegalStateException to be thrown.

Shutdown hooks run at a delicate time in the life cycle of a virtual machine and should therefore be coded defensively. 

They should, in particular, be written to be thread-safe and to avoid deadlocks insofar as possible. They should also not rely blindly upon services that may have registered their own shutdown hooks and therefore may themselves in the process of shutting down. Attempts to use other thread-based services such as the AWT event-dispatch thread, for example, may lead to deadlocks.

Shutdown hooks should also **finish their work quickly**. When a program invokes exit the expectation is that the virtual machine will promptly shut down and exit. When the virtual machine is terminated due to user logoff or system shutdown the underlying operating system may only allow a fixed amount of time in which to shut down and exit. It is therefore inadvisable to attempt any user interaction or to perform a long-running computation in a shutdown hook.

Uncaught exceptions are handled in shutdown hooks just as in any other thread, by invoking the uncaughtException method of the thread's ThreadGroup object. The default implementation of this method prints the exception's stack trace to System.err and terminates the thread; it does not cause the virtual machine to exit or halt.

In rare circumstances the virtual machine may abort, that is, stop running without shutting down cleanly. This occurs when the virtual machine is terminated externally, for example with the `SIGKILL` signal on Unix or the `TerminateProcess` call on Microsoft Windows. The virtual machine may also abort if a native method goes awry by, for example, corrupting internal data structures or attempting to access nonexistent memory. If the virtual machine aborts then no guarantee can be made about whether or not any shutdown hooks will be run.

Params:
- hook – An initialized but unstarted Thread object

Throws:

- IllegalArgumentException – If the specified hook has already been registered, or if it can be determined that the hook is already running or has already been run
- IllegalStateException – If the virtual machine is already in the process of shutting down
- SecurityException – If a security manager is present and it denies RuntimePermission("shutdownHooks")

