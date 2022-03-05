<h1 align="center">adb root</h1>

## adb root(仅限mtk手表)

```
- adb shell settings put global device_provisioned 1
# boot to Home Screen
# go to setting -> system -> Developer options -> OEM unlocking
– adb reboot bootloader
– fastboot flashing unlock
  (fastboot getvar unlocked)
# press volume up key(不用操作)
– fastboot reboot
– adb root
– adb disable-verity
– adb reboot
– adb root
– adb remount
```

