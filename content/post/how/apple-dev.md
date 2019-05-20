+++
title = "How to develop on MacOS for Apple platform"
description = "xcode"
tags = [
    "mac",
    "apple"
]
date = "2019-05-15"
topics = [
    "mac",
    "apple",
    "dev"
]
draft = false
toc=true
+++

### xip format
[xip format install](https://ktatsiu.wordpress.com/2016/10/28/how-to-download-xcode-8-x-with-xip-format/)  

To verify that a package is signed, execute the following command on a computer with the package:
```
pkgutil –check-signature /path/to/package.pkg
```
If the output is “Status: signed by a certificate trusted by Mac OS X”, the package is signed. If the output is “Status: no signature”, the package is not signed.

### 运行多个Xcode版本
[from Baidu](https://jingyan.baidu.com/article/f0062228091dd1fbd3f0c8de.html)  
显示当前版本：`xcode-select --print-path`   
更换版本 `sudo xcode-select --switch /Applications/Xcode-beta.app/Contents/Developer`