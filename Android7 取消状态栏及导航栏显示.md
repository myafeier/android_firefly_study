# Android7 取消状态栏及导航栏显示

### 1、取消状态栏显示

修改：/frameworks/base/packages/SystemUI/src/com/android/systemui/statusbar/phone/PhoneStatusBar.java  （Linearl：3677）

```
mStatusBarView.setVisibility(View.GONE);
```

### 2、取消导航栏显示

修改： /device/rockchip/rk3328/system.prop （Line：48）

```
qemu.hw.mainkeys=1
```

