ROC_rk3328_cc GPIO控制办法

1. Android下针脚对应

   - pin number 15 : 86
   - pin number 16 : 3
   - pin number 18 : 101
   - pin number 22 : 2

2. 添加一个开机后启动AM的指令

   /vendor/rockchip/common/phone/etc/prestart.sh(new file)

   ```
   #!/system/bin/sh
   echo 2 > /sys/class/gpio/export
   echo out > /sys/class/gpio/gpio2/direction
   chmod a+w /sys/class/gpio/gpio2/value
   ```

3. 修改设备配置，编译时复制脚本到系统system目录

   /device/rockchip/rk3328/rk3328.mk （Line: 43）

   ```
   PRODUCT_COPY_FILES +=vendor/rockchip/common/phone/etc/prestart.sh:system/bin/prestart.sh
   ```

4. 修改设备启动项，监听启动事件，运行指令

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

5、android 程序

```
private void IOCtl(int pin,int level){
    String path;
    path="/sys/class/gpio/gpio"+pin+"/value";

    try {
        FileOutputStream out=new FileOutputStream(path);
        out.write(String.valueOf(level).getBytes());
        out.flush();
        out.close();
    } catch (FileNotFoundException e) {
        e.printStackTrace();
    } catch (IOException e) {
        e.printStackTrace();
    }
}
```

