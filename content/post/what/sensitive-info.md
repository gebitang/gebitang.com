+++
title = "信息规则：银行卡、手机号、身份证、车牌号、车辆识别号"
description = "rules of sensitive info"
tags = [
    "info"
]
date = "2020-07-29"
topics = [
    "info",
    "regex"
]
toc = true
+++

### 银行卡号编码规则

[银行卡号编码规则及其应用](https://zhuanlan.zhihu.com/p/21399490)

详细介绍[银行卡号的编码规则及校验](https://www.jianshu.com/p/13fb9e3556da)

[Payment card number](https://en.wikipedia.org/wiki/Payment_card_number) [Luhn algorithm](https://en.wikipedia.org/wiki/Luhn_algorithm)


卡号最长19位，包括：

- 6位发行者识别号码（IIN, Issuer Identification Number）
- 最长12位的账户号码（Primary Account Number, PAN）
- 1位校验码，以Luhn算法计算

<!--more-->

[发卡行识别码](https://zh.wikipedia.org/wiki/%E5%8F%91%E5%8D%A1%E8%A1%8C%E8%AF%86%E5%88%AB%E7%A0%81)


```
[1-9]\d{12,18}

^([1-9]{1})(\d{14}|\d{18})$
```

校验码为银行卡号最后一位，采用LUHN算法，亦称模10算法。计算方法如下：

第一步：从右边第1个数字开始每隔一位乘以2；

第二步： 把在第一步中获得的乘积的各位数字相加，然后再与原号码中未乘2的各位数字相加；

第三步：对于第二步求和值中个位数求10的补数，如果个位数为0则该校验码为0。

举例：6259 6508 7177 209（不含校验码的银行卡号）

第一步：6*2=12，5*2=10，6*2=12，0*2=0，7*2=14，7*2=14，2*2=4，9*2=18

第二步：1+2 + 1+0 + 1+2 + 0 + 1+4 + 1+4 + 4 + 1+8 = 30

30 + 2+9+5+8+1+7+0 = 62

第三步：10-2=8

所以，校验码是8，完整的卡号应该是6259650871772098。

[Luhn algorithm Java](https://www.geeksforgeeks.org/luhn-algorithm/ ) [Luhn algorithm Golang](https://github.com/theplant/luhn)



### 身份证号码的编码规则

[身份证号码的编码规则](https://www.jianshu.com/p/ead5b08e9839)

>身份证号码共18位，由17位本体码和1位校验码组成。  
>  
>前6位是地址码，表示登记户口时所在地的行政区划代码，依照《中华人民共和国行政区划代码》国家标准（GB/T2260）的规定执行； 
>  
>7到14位是出生年月日，采用YYYYMMDD格式；  
>  
>15到17位是顺序码，表示在同一地址码所标识的区域范围内，对同年、同月、同日出生的人编订的顺序号，顺序码的奇数分配给男性，偶数分配给女性，即第17位奇数表示男性，偶数表示女性；  
>
>第18位是校验码，采用ISO 7064:1983, MOD 11-2校验字符系统，计算规则下一章节说明。

一代身份证与二代身份证的区别在于：

- 一代身份证是15位，二代身份证是18位；
- 一代身份证出生年月日采用YYMMDD格式，二代身份证出生年月日采用YYYYMMDD格式；
- 一代身份证无校验码，二代身份证有校验码。

```
# 18 位
^\d{6}(18|19|([23]\d))\d{2}((0[1-9])|(10|11|12))(([0-2][1-9])|10|20|30|31)\d{3}[0-9Xx]$
# 15位
^\d{6}\d{2}((0[1-9])|(10|11|12))(([0-2][1-9])|10|20|30|31)\d{3}$
```

### 手机号规则

知识点好文 [史上最全：手机号段分配完全揭秘](https://news.mydrivers.com/1/496/496684.htm)

- 前3位是网络识别号
- 4-7位是地区编码(HLR归属位置寄存器)
- 8-11位是用户号码(随机分配)
- 国内的手机号码是由国家信息产业部统一规划


```
^(\(\d{3,4}\)|\d{3,4}-|\s)?\d{7,14}$

^1[3-9]\d{9}$
```

### 车牌号码规则

[车牌号码规则](https://cloud.tencent.com/developer/article/1361209)

[中国车牌号码规则-民用、警队、军队等](https://my.oschina.net/chenyoca/blog/1571062)

[wiki: Vehicle identification number](https://en.wikipedia.org/wiki/Vehicle_identification_number)

#### 传统车牌

- 第1位为省份简称（汉字）
- 第二位为发牌机关代号（A-Z的字母）
- 第3到第7位为序号（由字母或数字组成，但不存在字母I和O，防止和数字1、0混淆，另外最后一位可能是“挂学警港澳使领”中的一个汉字）。

#### 新能源车牌

- 第1位和第2位与传统车牌一致，第3到第8位为序号（比传统车牌多一位）。

- 小型车：第1位只能是字母D或F，第2为可以是数字或字母，第3到6位必须是数字。

- 大型车：第1位到第5位必须是数字，第6位只能是字母D或F。

```
^(([京津沪渝冀豫云辽黑湘皖鲁新苏浙赣鄂桂甘晋蒙陕吉闽贵粤青藏川宁琼使领][A-Z](([0-9]{5}[DF])|([DF]([A-HJ-NP-Z0-9])[0-9]{4})))|([京津沪渝冀豫云辽黑湘皖鲁新苏浙赣鄂桂甘晋蒙陕吉闽贵粤青藏川宁琼使领][A-Z][A-HJ-NP-Z0-9]{4}[A-HJ-NP-Z0-9挂学警港澳使领]))$
```

### 车架号规则

[PDF: 车辆识别代号（VIN）编制规则](https://www.leopaard.com/uploadfile/2018/0506/20180506045547367.pdf)

[17位车架号/VIN码含义](https://www.douban.com/group/topic/127367158/)

“车辆识别号码”，英文名称“Vehicle Identification Number”，简称“VIN”。由17位由英文（不含'I'、'O'、'Q'，避免与'1'，'0'混淆）和数字（0-9）组成的号码。

- 1 ~ 3 位为世界制造商标识码（简称“WMI”）
- 4 ~ 9 位为车辆说明部分（简称“VDS”）
- 10 ~ 17位为车辆指示码（简称“VIS”）

![](https://upload-images.jianshu.io/upload_images/3296949-4cdb52b7d433f73f.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

```
/^[A-HJ-NPR-Z\d]{17}$/
```

[Vehicle identification number](https://en.wikipedia.org/wiki/Vehicle_identification_number)

第9位为校验码，只能是数字0-9或X，它是由其他16位字码对应数值乘以其所位置权数的和除以11所得的余数；当余数为0-9时候，余数就是检验数字；当余数为10时，X为检验代码。校验码的目的就是核对数字，检验VIN填写是否正确，并能防止假冒产品。

各个位置的字母或数字的值、每个位置的权重、最终的校验码计算方式——

![](https://s3-img.meituan.net/v1/mss_3d027b52ec5a4d589e68050845611e68/ff/n0/0k/q1/x8_435850.jpg)