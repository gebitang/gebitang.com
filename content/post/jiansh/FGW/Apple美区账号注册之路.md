+++
title = "Apple美区账号注册之路"
tags = [
    "jiansh"
]
date = "2020-03-07"
topics = [
    "jiansh",
    "FGW"
]
toc = true
+++



[link on JianShu](https://www.jianshu.com/p/7d00f6cef828)

使用[一枝红杏出墙来](https://www.jianshu.com/p/9fc3c7b94f90)里提到的[AgentNEO](https://agneo.co/?rc=8d50vj3e)服务，可以在Mac、Windows、Android设备上自由出入。

但iPhone上使用这个服务需要：先用美区账号登录，付费购买支持的软件Shadowrocket(2.99)、Quantumult(4.99)、Kitsunebi(4.99)才能使用。这不变成了鸡(生蛋)蛋(生鸡)问题了嘛。

只能先曲线救国了，先注册美区账号再说。这里有[官方教程：Create or use an Apple ID without a payment method](https://support.apple.com/en-us/HT204034)、[How to create a new Apple ID](https://support.apple.com/en-us/HT204316)


### 注册账号
- 挂上美区的IP地址，访问[whatismyipaddress.com](https://whatismyipaddress.com/)检查当前网络IP是否正确
- 访问[https://appleid.apple.com](https://appleid.apple.com/)开始注册，上一步正确的话，这里可以看到注册时默认是美区账号
- 填写密保问题、有效邮件地址（尽量使用gmail、hotmail等国外服务的邮箱）。然后激活邮件（因为是电脑端访问，所以正好直接打卡邮件激活）

### 激活账号
- 注册成功后，从[https://appleid.apple.com](https://appleid.apple.com/)登录，填写账单地址和支付方式（支付方式选择为None）、手机号码

- 如果不使用美区IP，注册的时候可能无法选择None选项，参考这里[If you can't remove your last payment method or use no payment method with your Apple ID](https://support.apple.com/en-us/HT203905)

- 工具：[美国地址http://haoweichi.com/](http://haoweichi.com/)、[生成随机假地址](https://www.fakeaddressgenerator.com/World_Address/get_us_address)，或者利用google map选择一个你的ip地址(通过第一步可以看到大概位置)附近的生活区域的地址
- 使用虚拟手机号接受短信 [SMS Receive Free](https://smsreceivefree.com/)，不一定必须收到短信，这里可以看到有效的手机号码，或者google搜索`disposable number`获取有效的手机号。需要真实的号码，可以参考[这个攻略](https://blog.shuziyimin.org/348)

### 使用账号
账号注册成功后，首次在App Store使用时会走一遍review流程（同意协议、确认付款方式）。可以下载一个免费的应用走这个流程。

在下面这个确认付款方式页面时，一直提示`For assistance, contact iTunes Support at www.apple.com/support/itunes/ww/.`
![image](https://upload-images.jianshu.io/upload_images/3296949-aa1a03dc772c6d82.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

~~一些无效的尝试~~
绑定国内的双币信用卡、尝试注册美版Paypal（实践上难度更高）、iPhone设备上使用国内的wifi走流程（干脆就没有None选项）、iPhone设备上使用美区IP走流程（依然无效）

以上操作耗费了更多的精力，但都没有成功。最后老实进行反馈吧（尽管有点心虚）。

- 访问 [www.apple.com/support/itunes/ww/](www.apple.com/support/itunes/ww/)选择Unite States地区，进入 [iTunes页面：https://support.apple.com/itunes](https://support.apple.com/itunes)，选择右下角都 get supports链接，选择问题类型，描述问题，提供反馈邮箱，提交反馈。

![](https://upload-images.jianshu.io/upload_images/3296949-8a1a0738d7b00a3d.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

- 运气不错，大概两小时后，收到反馈（反馈时间也正好是美区的工作时间端），官方后台做了操作，让我再次走下流程。

![](https://upload-images.jianshu.io/upload_images/3296949-eeea3ae57bd6b78f.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

---
### Android共享代理
上面的无效尝试也并非没有成果，搜到这篇[借助Proxy Server实现Android设备免root共享VPN连接](https://eugenegeng.blogspot.com/2018/10/share-vpn-with-proxy-server-on-Android-no-root.html)文章，尽管评论说v2rayng无效，还是尝试了一下我的。
- 从google play搜索下载proxy server
- Android、iPhone设备使用相同的局域网
- Proxy Server根据说明操作，必须的是指定端口，启动后在Info栏可以看到IP
- iPhone端的无线链里的代理修改为手动，使用上一步的IP+端口
- iPhone端使用浏览器访问[whatismyipaddress.com](https://whatismyipaddress.com/)检查当前网络IP是否正确

挂上相同的代理，iPhone端从App Store下载一个免费的App，走到确认支付手段的地方，选择None选项（如果使用国内的wifi，是没有None选项的；如果使用国内信用卡，也始终无法通过Next），Next终于成功。

---

首次走完上面的流程之后，再次使用就不需要必须使用美国IP了，因为不需要再走激活流程。付款方式采用none之后，可以使用礼品卡进行充值，可参考[这篇文章](https://www.hurbai.com/iOS/182)关注一下注意事项。

访问[礼品卡页面](https://www.apple.com/shop/gift-cards/itunes-electronic)，直接按照流程操作即可。

- 两个有效有效（送礼人、接收人）
- 一个有效美国地址（作为付款但账单地址）
- 使用Guest方式支付（可以使用国内的双币信用卡）

大概4小时之后可以收到，官方承诺的为24小时发货。
![](https://upload-images.jianshu.io/upload_images/3296949-e942a3d746a96c5e.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
