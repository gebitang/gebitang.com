+++
title = "JaCoCo--skipping-jacoco-due-to-missing-data-file"
tags = [
    "jiansh"
]
date = "2020-03-07"
topics = [
    "jiansh",
    "US"
]
toc = true
+++



[link on JianShu](https://www.jianshu.com/p/f15864b65c1f)

昨天开始有一个多module的项目一直无法获取到单测覆盖率结果。

初步跟踪了一下执行log，发现多个提示："skipping jacoco due to missing data file"。

今天从本地环境执行这个项目，也是相同的结果。

搜索这个问题：[stackoverflow：getting-skipping-jacoco-execution-due-to-missing-execution-data-file-upon-exec](https://stackoverflow.com/questions/18107375/getting-skipping-jacoco-execution-due-to-missing-execution-data-file-upon-exec)

>The truth is, this error is returned for many, many reasons. We experimented with the different solutions on Stack Overflow, but found this [resource](http://www.ffbit.com/blog/2014/05/21/skipping-jacoco-execution-due-to-missing-execution-data-file/) to be best. It tears down the many different potential reasons why Jacoco could be returning the same error.

[https://code-examples.net/en/q/229f799](https://code-examples.net/en/q/229f799)

[http://www.ffbit.com/blog/2014/05/21/skipping-jacoco-execution-due-to-missing-execution-data-file/](http://www.ffbit.com/blog/2014/05/21/skipping-jacoco-execution-due-to-missing-execution-data-file/)

按照上面两个提示，这个项目的配置没有问题。 

使用IDE倒入这个工程，发现多个module里时间是只有一个包含了单测代码，那么其他module在执行JaCoCo插件时提示“skipping jacoco due to missing data file”是正确的结果。

实验式排查，`mvn help:effective-pom`查看完整的module内容。关键是项目的内容，这意味着，只计算包含`impl`路径的单测结果（？），而实际的单测代码并没有包含这个路径，从结果看，测试代码执行了，但没有生成报告。

按照下面但方式修改后，可以看到聚合后但xml文件了。

![](https://upload-images.jianshu.io/upload_images/3296949-2db404b17ccdebe0.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
