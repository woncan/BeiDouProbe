# BeiDouProbe

北斗探针SDK说明

##### 方式一： Gradle集成SDK

######  1.1在Project的build.gradle文件中配置repositories，添加maven仓库地址（新版本Android studio需要在settings.gradle里面配置）
```
    repositories {
            ...
            maven { url "https://jitpack.io" }
    }
```

###### 在主工程的build.gradle文件配置dependencies
[![](https://jitpack.io/v/woncan/BeidouProbe.svg)](https://jitpack.io/#woncan/BeidouProbe)
```
    android {
            compileOptions {
    		    sourceCompatibility JavaVersion.VERSION_1_8
		    targetCompatibility JavaVersion.VERSION_1_8
	    }
    }
    dependencies {
	       implementation 'com.github.woncan:BeiDouProbe:latest.release' //其中latest.release指代最新SDK版本号
    }
```
##### 方式二：通过拷贝添加
###### 1.添加 jar 文件：
    将下载的 SDK 的 jar包复制到工程（此处截图以官方示例Demo为例子）的 libs 目录下，如果有老版本 jar 包在其中，请删除

###### 2.添加 so 库：
        使用默认配置，不需要修改build.gradle。在 main 目录下创建文件夹 jniLibs (如果有就不需要创建了)，将下载文件的 armeabi 文件夹复制到这个目录下,如果已经有这个目录，将下载的 so 库复制到这个目录即可。
        使用自定义配置，将下载文件的 armeabi 文件夹复制到 libs 目录，如果有这个目录,请将下载的 so 库复制到这个目录，然后打开build.gradle，找到 sourceSets 标签，在里面增加一项配置
```
   sourceSets {
        main {
            jniLibs.srcDirs = ['libs']
        }
    }
```
[点击下载文件](https://github.com/woncan/BeidouProbe/releases)

##### 添加混淆配置

-keep public class com.woncan.device.**{*;}

###  使用

##### 使用拷贝方式添加的需要配置AndroidManifest.xml
```
    <!--允许程序使用网络-->
    <uses-permission android:name="android.permission.INTERNET" />
    <!--允许程序获取粗略的位置-->
    <uses-permission android:name="android.permission.ACCESS_COARSE_LOCATION" />
```
##### 搜索设备
```
    //搜索设备
    List<UsbSerialDriver> serialDeviceList = DeviceManager.getSerialDeviceList(MainActivity.this);
    UsbSerialDriver usbSerialDriver = serialDeviceList.get(0);
    //判断权限
    if (((UsbManager) getSystemService(Context.USB_SERVICE)).hasPermission(usbSerialDriver.getDevice())) {
            connectDevice(usbSerialDriver);
    } else {
            DeviceManager.requestPermission(MainActivity.this, usbSerialDriver, isGranted -> {
            if (isGranted) {
                    connectDevice(usbSerialDriver);
                }
            });
    }
    //注册设备连接监听
    DeviceManager.registerDeviceAttachListener(MainActivity.this, new OnDeviceAttachListener() {
            @Override
            public void onDeviceAttach(UsbSerialDriver usbSerialDriver) {
                //设备连接手机时回调
                //需要判断权限
                connectDevice(usbSerialDriver);
            }
    });
```

##### 连接设备
```
    private void connectDevice(UsbSerialDriver driver){
        Device device = DeviceManager.connectDevice(getApplicationContext(), driver);
        //监听定位信息
        device.setLocationListener(new LocationListener() {
            @Override
            public void onError(int errCode, String msg) {
                //错误信息    修改部分
                Log.i("TAG", "errCode: "+errCode+"   msg:" + msg);
            }

            @Override
            public void onLocationChanged(WLocation location) {
               //位置信息
            }
        });
        //监听卫星数据
        device.setSatelliteListener(new SatelliteListener() {
            @Override
            public void onSatelliteListener(Satellite[] satellites) {
                //卫星信息
            }
        });
	//NMEA数据
        device.setNMEAListener(new NMEAListener() {
            @Override
            public void onNMEAReceiver(String s) {
                //NMEA回调
            }
        });
    }
```
##### 获取设备信息
```
   Device device = DeviceManager.getInstance().getDevice();
        if (device!=null){
            DeviceInfo deviceInfo = device.getDeviceInfo();
        }
```

##### 断开连接
```
    DeviceManager.disconnect();

```
##### 日志
```
    LogUtil.setIsDebug(BuildConfig.DEBUG);
```
##### onError状态

| 错误码 | 错误信息 |
| --- | --- |
| 1001 | 串口断开       |
| 1101 | 初始化失败     |
| 1102 | 初始化失败     |
| 1121 | 账号出错       |
| 1122 | 账号出错       |
| 1123 | 账号出错       |
| 1124 | 账号出错       |
| 1126 | 账号出错       |
| 1127 | 账号出错       |
| 1201 | 差分服务无响应  |
| 1211 | 网络错误       |
| 1212 | 密码错误       |
| 1213 | 账号过期       |
| 1214 | 账号待激活     |
| 1215 | 账号无效       |
| 1216 | 服务端认证失败  |
| 1217 | ID错误         |
| 1218 | 账号已登录      |
| 1219 | 账号被禁用       |
| 1220 | 挂载点无效       |
| 1221 | 失败次数超限       |
| 1222 | 服务覆盖，但无差分       |
| 1223 | DNS解析异常       |
| 1224 | 账号池鉴权入参为空       |
| 1225 | 频繁登录       |
| 1226 | 登录类型错误       |
| 1227 | 账号过期       |
| 1228 | Cors鉴权失败       |
| 1229 | 设备ID无效       |
| 1230 | 账号类型错误       |
| 1231 | SDK启动失败       |
| 2001 | 获取账号失败   |
| 2002 | 获取星历失败   |
| 3101 | 差分账号或者密码错误   |

##### WLocation

| 方法名 | 说明| 备注|
| :--| :-- | :-- |
| getFixStatus| 获取定位状态 |0=无法定位；1=单点定位；5=浮点定位；4=固定定位|
| getDiffAge| 差分延时 |单位：秒|
|getLatitude| 纬度 |单位：度|
| getLongitude|经度  |单位：度|
| getAltitude|海拔高 |单位：米|
| getAltitudeCorr|高程异常值 |大地高=海拔高+高程异常值|
| getnSatsInView| 卫星数 |
| getnSatsInUse| 参与解算的卫星数 |
| getTime| 时间戳 |
| getSpeed| 速度 |单位：米/秒|
| getBearing| 速度方向 |
| getAccuracy| 定位水平精度|单位：米|
| getmVerticalAccuracy| 定位垂直精度|
| gethDOP| hdop|
| getvDOP| vdop|
| getpDOP| pdop|
