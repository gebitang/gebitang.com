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