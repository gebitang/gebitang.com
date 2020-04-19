+++
title = "Sonar实践问题：支持go工程的覆盖率"
description = "go test for SonarQube"
tags = [
]
date = "2020-03-19"
topics = [
    "US"
]
+++

[Go for SonarQube](https://dev.to/remast/go-for-sonarqube-4iho)  
[Go for SonarCloud with Github Actions](https://dev.to/remast/go-for-sonarcloud-with-github-actions-3pmn)

[示例工程 service_sonar](https://github.com/remast/service_sonar)

[go testing](https://golang.org/pkg/testing/)

---

步骤——  

- go build生成可执行程序
- 执行 `go test -coverprofile=bin/cov.out service-sonar` 生成覆盖率文件
- 执行`sonar-scanner `

关键时如何生成覆盖率文件，将这个文件传递给sonar-scanner即可。

```
.PHONY: default
default: build

all: clean get-deps build test

version := "0.1.0"

build:
	mkdir -p bin
	go build -o bin/service-sonar main.go

test: build
	go test -short -coverprofile=bin/cov.out `go list ./... | grep -v vendor/`
	go tool cover -func=bin/cov.out

clean:
	rm -rf ./bin

sonar: test
	sonar-scanner -Dsonar.projectVersion="$(version)"

start-sonar:
	docker run --name sonarqube -p 9000:9000 sonarqube

```

实践版本——

使用`go help command`可以查看对应command的用法，或直接执行command，不传递参数，也会显示对应的用法。例如：`go help test`、`go tool cover`

sonar-scanner默认会在当前目录下查找`sonar-project.properties`文件检查对应的参数值。关键的参数为`sonar.go.coverage.reportPaths`
```
sonar.sources=.
sonar.exclusions=**/*_test.go,**/vendor/**,**/testdata/*
 
sonar.tests=.
sonar.test.inclusions=**/*_test.go
sonar.test.exclusions=**/vendor/**
sonar.go.coverage.reportPaths=./bin/cov.out

```

这些值都可以使用`-Dsonar.go.coverage.reportPaths=./bin/cov.out`的方式传递。

---

本地有[Basic SonarQube SonarScanner](https://www.jianshu.com/p/0b94bc843357)搭建的server，完成扫描后，可以看到跟文章相同的覆盖率数据。

![](https://upload-images.jianshu.io/upload_images/3296949-989274cb23e3d01e.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
