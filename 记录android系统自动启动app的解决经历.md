# 记录android系统自动启动app的解决经历



## 方法1：通过替换Launcher启动

- 步骤：

  - 程序要点：

    - AndroidManifest.xml

      ```
      <?xml version="1.0" encoding="utf-8"?>
      <manifest xmlns:android="http://schemas.android.com/apk/res/android"
          xmlns:tools="http://schemas.android.com/tools"
          package="com.vein.xgzx.veindemo"
          android:sharedUserId="android.uid.system">
      
          <uses-permission android:name="android.permission.WRITE_EXTERNAL_STORAGE" />
          <uses-permission android:name="android.permission.BLUETOOTH_ADMIN" />
          <uses-permission android:name="android.permission.BLUETOOTH" />
          <uses-permission android:name="android.permission.INTERNET" />
          <uses-permission android:name="android.hardware.usb.host" />
          <uses-permission android:name="android.permission.RECEIVE_BOOT_COMPLETED"/>
      
      
          <application
              android:allowBackup="true"
              android:icon="@mipmap/ic_launcher"
              android:label="@string/app_name"
              android:supportsRtl="true"
              android:theme="@style/AppTheme"
              tools:ignore="GoogleAppIndexingWarning">
      
              <receiver android:name="BootReceiver" >
                  <intent-filter android:priority="1000" >
                      <action android:name="android.intent.action.BOOT_COMPLETED" />
                      <category android:name="android.intent.category.LAUNCHER" />
                  </intent-filter>
                  <intent-filter android:priority="1000" >
                      <action android:name="android.intent.action.REBOOT" />
      
                      <category android:name="android.intent.category.LAUNCHER" />
                  </intent-filter>
              </receiver>
      
              <service android:name="com.u1200.service.MainService" android:exported="true" android:enabled="true">
              </service>
              <activity
                  android:name=".MainActivity"
                  android:label="@string/app_name"
                  android:exported="true">
                  <intent-filter>
                      <category android:name="android.intent.category.HOME" />
                      <category android:name="android.intent.category.DEFAULT" />
                      <category android:name="android.intent.category.LAUNCHER" />
                      <action android:name="android.intent.action.MAIN" />
      
                  </intent-filter>
                  <meta-data
                      android:name="android.hardware.usb.action.USB_DEVICE_ATTACHED"
                      android:resource="@xml/device_filter" />
              </activity>
              <activity
                  android:name="com.vein.xgzx.veindemo.VeinInit"
                  android:configChanges="orientation|keyboardHidden|screenSize"
                  android:label="@string/connecting"
                  android:theme="@android:style/Theme.Dialog" >
              </activity>
              <activity
                  android:name="com.vein.xgzx.veindemo.DeviceListActivity"
                  android:configChanges="orientation|keyboardHidden|screenSize"
                  android:label="@string/scanning"
                  android:theme="@android:style/Theme.Dialog" />
          </application>
      
      </manifest>
      ```

  - 是否需要签名未验证



## 方法2： 通过broadcastReceiver接收系统启动广播

- 此办法是可行的，之所以在3328-cc上不行，是因为3328-cc 因为内存小做了优化，不允许系统广播。

  - 检查方法如下：

    ```
    adb root
    adb remount
    adb shell
    getprop ro.mem_optimise.enable  // 如果为true 则为优化过
    ```

    

  - 修改措施如下：

    busybox vi system/build.prop

  ```
  修改： ro.mem_optimise.enable  == false
  ```

  

- 步骤：

  - 程序要点

    - AndroidManifest.xml

      ```
      <manifest xmlns:android="http://schemas.android.com/apk/res/android"
          xmlns:tools="http://schemas.android.com/tools"
          package="com.vein.xgzx.veindemo"
          android:sharedUserId="android.uid.system">
      
          <uses-permission android:name="android.permission.WRITE_EXTERNAL_STORAGE" />
          <uses-permission android:name="android.permission.BLUETOOTH_ADMIN" />
          <uses-permission android:name="android.permission.BLUETOOTH" />
          <uses-permission android:name="android.permission.INTERNET" />
          <uses-permission android:name="android.hardware.usb.host" />
          <uses-permission android:name="android.permission.RECEIVE_BOOT_COMPLETED"/>
      
      
          <application
              android:allowBackup="true"
              android:icon="@mipmap/ic_launcher"
              android:label="@string/app_name"
              android:supportsRtl="true"
              android:theme="@style/AppTheme"
              tools:ignore="GoogleAppIndexingWarning">
      
              <receiver android:name="BootReceiver" >
                  <intent-filter android:priority="1000" >
                      <action android:name="android.intent.action.BOOT_COMPLETED" />
                      <category android:name="android.intent.category.LAUNCHER" />
                  </intent-filter>
                  <intent-filter android:priority="1000" >
                      <action android:name="android.intent.action.REBOOT" />
      
                      <category android:name="android.intent.category.LAUNCHER" />
                  </intent-filter>
              </receiver>
      
              <service android:name="com.u1200.service.MainService" android:exported="true" android:enabled="true">
              </service>
              <activity
                  android:name=".MainActivity"
                  android:label="@string/app_name"
                  android:exported="true">
                  <intent-filter>
                      <category android:name="android.intent.category.LAUNCHER" />
                      <action android:name="android.intent.action.MAIN" />
                      <category android:name="android.intent.category.DEFAULT" />
                      <action android:name="android.hardware.usb.action.USB_DEVICE_ATTACHED" />
            
                  </intent-filter>
              </activity>
          
          </application>
      
      </manifest>
      ```

  - BootReceiver.java

    ```
    public class BootReceiver extends BroadcastReceiver {
    
        private static final String TAG="shon";
        @Override
        public void onReceive(Context context, Intent intent) {
    
            String action=intent.getAction();
            Log.i(TAG, "onReceive broadcast: "+action);
            Intent intent1=new Intent(context,MainActivity.class);
            intent1.addFlags(Intent.FLAG_ACTIVITY_NEW_TASK);
            context.startActivity(intent1);
        }
    }
    ```

  - MainServie.java

    ```
    public class MainService extends Service {
        public static final String TAG="MainService";
        @Nullable
        @Override
        public IBinder onBind(Intent intent) {
            return null;
        }
    
        @Override
        public void onCreate() {
            super.onCreate();
        }
    
        @Override
        public void onStart(Intent intent, int startId) {
            super.onStart(intent, startId);
            Log.i(TAG, "onStart: ");
        }
    
        @Override
        public int onStartCommand(Intent intent, int flags, int startId) {
            Log.i(TAG, "onStartCommand: ");
            return START_STICKY;
        }
    
        @Override
        public void onDestroy() {
            super.onDestroy();
        }
    }
    ```

- 验证方法：通过adb模拟系统启动广播

  - adb root
  - adb remount
  - adb shell
    - am broadcast -a android.intent.action.BOOT_COMPLETED 

- 系统签名办法：

  ###### 参考文档：<https://blog.csdn.net/cxq234843654/article/details/51557025>

  - as 生成签名 .jks

    - build
      - Generate Signed Bundle or APK
        - Create New 

  - 用系统证书进行签名

    - 下载 签名工具文件 https://github.com/getfatday/keytool-importkeypair

    - 获取andriod源码中的证书文件： 位置： ../build/target/product/security

      - platform.x509.pem
      - platform.pk8

    - 对以上三个文件进行签名：

      ```
      ./keytool-importkeypair -k [jks文件名] -p [jks的密码] -pk8 platform.pk8 -cert platform.x509.pem -alias [jks的别名]
      ```

    - 修改 build.gradle以支持run签名(未验证)

      ```
      android {
          compileSdkVersion 21
          buildToolsVersion '28.0.2'
      
          defaultConfig {
              applicationId "com.android.com.veindemo"
              minSdkVersion 15
              targetSdkVersion 21
              versionCode 1
              versionName "1.0"
          }
      
          signingConfigs {
              release {
                  storeFile file("../signApk/SignDemo.jks")
                  storePassword '123456'
                  keyAlias 'SignDemo'
                  keyPassword '123456'
              }
      
              debug {
                  storeFile file("../signApk/SignDemo.jks")
                  storePassword '123456'
                  keyAlias 'SignDemo'
                  keyPassword '123456'
              }
          }
          buildTypes {
              release {
                  //在这里添加：
                  lintOptions {
                      checkReleaseBuilds false
                      abortOnError false
                  }
                  minifyEnabled false
                  proguardFiles getDefaultProguardFile('proguard-android.txt'), 'proguard-rules.pro'
              }
          }
      }
      ```

       



## 方法3：修改android源文件

###### 	通过添加一个开机启动的AM指令解决

### 步骤：

- 添加一个开机后启动AM的指令

  /vendor/rockchip/common/phone/etc/prestart.sh(new file)

  ```
  #!/system/bin/sh
  
  am start com.android.com.veindemo
  ```

- 修改设备配置，编译时复制脚本到系统system目录

  /device/rockchip/rk3328/rk3328.mk  （Line: 43）

  ```
  PRODUCT_COPY_FILES +=vendor/rockchip/common/phone/etc/prestart.sh:system/bin/prestart.sh
  ```

- 修改设备启动项，监听启动事件，运行指令

  /device/rockchip/common/init.rk30board.rc（Line：170， **on boot 段落中**）

  ```
  chmod 0755 /system/bin/prestart.sh
  ```

  /device/rockchip/common/init.rk30board.rc（Line： 317）

  ```
  on property:sys.boot_completed=1
         chmod 755 /system/bin/preinstall.sh
         stop  preinstall
         start  preinstall
  
  service preinstall /system/bin/preinstall.sh
         seclabel u:r:preinstall:s0
      class main
      oneshot
      user root
  ```

  







