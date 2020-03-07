+++
title = "Sonar实践问题记录(一)"
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


[link on JianShu](https://www.jianshu.com/p/7461c045b1c0)

跟踪了一天的问题，记录一下。 

问题一：  
目前最大的问题为获取Sonar数据时一直处于超时状态。

第一步，怀疑在超时时间内一直请求，相当于对server做了压力测试，导致server端假死。  
- 请求之间添加等待时间
- 后续要为server端添加上监控信息（后面的分析更加验证了这一点的必要性）

问题二：
后面遇到了扫描失败：不识别分支模块——查询了log后，居然是因为使用了中文命名、分支名包含`,`符合  
- 不建议使用中文命名分支名，只是有规范的字符

问题三：  
submit结果时请求超时。  
- 目前原因还不清楚


问题四：  
缺少属性文件`Please provide compiled classes of your project with sonar.java.binaries property` 
- 可[参考](https://stackoverflow.com/questions/46976567/please-provide-compiled-classes-of-your-project-with-sonar-java-binaries)


问题五：
辅助的服务不稳定。kafka服务域名解析失败
- 基础设施问题，需要平台同事协助

--- 

问题一重复多次。第二步计划更新api，从`/api/ce/task?id=AW7WCYZS3Mm3i2nvUcQw`更新到`/api/ce/component?component=x:xx` 

另外，我怀疑是调用时没有使用验证，似乎这是更大的可能性。 


