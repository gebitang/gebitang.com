+++
title = "Sonar实践问题记录(八)jacoco-plugin"
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



[link on JianShu](https://www.jianshu.com/p/f2e211407353)

SonarQube 7.4以上版本默认自动jacoco plugin，可以分析jacoco生成的xml格式的测试报告。

在 配置 -- 应用市场 可以看到已安装的——
>JaCoCo：JaCoCo XML report importer Developed by [SonarSource](http://www.sonarsource.com/)

### 使用方式：
1. 首先执行junit测试生成JaCoCo报告。
添加JaCoCo plugin，生成报告的默认位置为`target/site/jacoco/jacoco.xml`，这里位置会被自动检查，不需要手动设置，如需修改默认位置，使用`-Dsonar.coverage.jacoco.xmlReportPaths=report1.xml,report2.xml `定义多个位置，或者在pom文件中使用properties字段定义。

```
   <plugins>
    <plugin>
    <groupId>org.jacoco</groupId>
    <artifactId>jacoco-maven-plugin</artifactId>
    <version>0.8.4</version>
    <executions>
      <execution>
        <id>prepare-agent</id>
        <goals>
         <goal>prepare-agent</goal>
        </goals>
      </execution>
      <execution>
        <id>report</id>
        <goals>
         <goal>report</goal>
        </goals>
      </execution>
    </executions>
    </plugin>
   </plugins>
```
`mvn clean jacoco:prepare-agent install jacoco:report` 


2. 将步骤一种的结果传给SonarQube，使用 `sonar.coverage.jacoco.xmlReportPaths`属于。 在sonarscanner中使用 `-Dsonar.coverage.jacoco.xmlReportPaths=path.to.jacoco.xml`定义，旧的 `sonar.jacoco.reportPaths`属性已经被废弃。

|symbol|name|number|
|---------|---------|---------|
sonar.coverage.jacoco.xmlReportPaths|Project-wide|Comma-separated list of paths to JaCoCo (jacoco.xml) report files.|


示例参考：[sonar-scanning-examples](https://github.com/SonarSource/sonar-scanning-examples)  
>  基础模式 [maven-basic](https://github.com/SonarSource/sonar-scanning-examples/tree/master/sonarqube-scanner-maven/maven-basic)
>  多module方式 [maven-multimodule](https://github.com/SonarSource/sonar-scanning-examples/tree/master/sonarqube-scanner-maven/maven-multimodule)


SonarQube使用JaCoCO的结果，所以需要先确保生成了结果，否则当然无法加载。

另外一个问题需要注意：Line Out of Range，类似[这里](https://stackoverflow.com/questions/50461872/sonarqube-line-out-of-range-since-file-shrinks-after-merge-with-master)和[这里](https://stackoverflow.com/questions/39753072/sonarqube-scan-error-with-line-out-of-range)提到的。生成覆盖率报告的代码和静态扫描的代码**必须一致**。否则在分析时，就可能出现out of range问题。——这也很好理解。

--- 

This plugin (provided by default with SonarQube 7.4+) allows you to load the [JaCoCo](https://www.jacoco.org/jacoco/) data from its **XML format** for all the languages for which you can generate a JaCoCo report.

### Usage

1.  Prior to the SonarQube/SonarCloud analysis, execute your unit tests and generate the **JaCoCo** coverage report in its **XML format**.

2.  Import reports while running the SonarQube/SonarCloud analysis by providing the paths of the *jacoco.xml* files through the following properties. The paths may be absolute or relative to the project base directory.


参考：
[[Coverage & Test Data] Importing JaCoCo coverage report in XML format](https://community.sonarsource.com/t/coverage-test-data-importing-jacoco-coverage-report-in-xml-format/12151)
[JaCoCo Plugin](https://docs.sonarqube.org/display/PLUG/JaCoCo+Plugin)
[Java Unit Tests and Coverage Results Import](https://docs.sonarqube.org/display/PLUG/Java+Unit+Tests+and+Coverage+Results+Import)
[Getting meaningful coverage results in SonarQube when using JaCoCo and Lombok](https://community.sonarsource.com/t/getting-meaningful-coverage-results-in-sonarqube-when-using-jacoco-and-lombok/13821)
[SonarQube and the code coverage](https://community.sonarsource.com/t/sonarqube-and-the-code-coverage/4725)
