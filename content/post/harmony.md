+++
title = "harmonyOS"
description = "about harmony"
tags = [
    "harmony",
    "os"
]
date = "2024-06-13"
topics = [
    "harmony"
]
toc = true
draft = true
+++

```shell
# 查看设备
`path/to/sdk/base/toolchains/hdc list targets` 
# 获取设备 udid
hdc -t xxx shell bm get --udid

```

### 查看hap签名信息

直接使用NotePad++打开hap应用，搜索 version-name 回找到json字符串，包含了 bundle-info，其中包含 证书信息；bundle-name；debug-info，其中包含 设备列表信息
