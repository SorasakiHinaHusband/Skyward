# 首次测试清单

> 以下操作会写 recovery 分区，不会清 data。请确认 bootloader 已解锁，并保留
> `stock/` 下的原厂 recovery 以便回滚。

## 刷入

```sh
# 先校验
sha256sum -c out/SHA256SUMS

# piano
fastboot flash recovery_a out/skyward-0.3.1-piano-OS4.0.6.0.XPYCNXM.img
fastboot flash recovery_b out/skyward-0.3.1-piano-OS4.0.6.0.XPYCNXM.img

# waffle
fastboot flash recovery_a out/skyward-0.3.1-waffle-PJD110_17.0.0.100.img
fastboot flash recovery_b out/skyward-0.3.1-waffle-PJD110_17.0.0.100.img

fastboot reboot recovery
```

首次启动可能需要 30–60 秒（等待 data 解密和 key 服务）。

## 检查项

按顺序测，哪一步失败就停在那里：

1. **显示**：Skyward 首页是否正常亮度、无花屏。piano 的面板原生是竖屏，
   系统在 Android 里旋转 270°；Skyward 没有旋转支持，所以画面可能是竖着的，
   把平板转 90° 使用即可。
2. **触摸**：能否点按、滑动；piano 依赖 `/odm/bin/touch_report_debug` 把原始
   帧转成触摸事件。piano 当前配置 `[touch] swap_xy = true`、
   `invert_y = true`；若左右反了再加 `invert_x = true`，若上下反了去掉
   `invert_y`，改完重新构建即可。
3. **adb**：`adb devices` 应看到设备；`adb shell id` 应为 `uid=0(root)`。
4. **data 解密**：如果设置了锁屏密码，进入解密页面输入；`/data`、`/sdcard`
   应可读写。失败时记录屏幕上显示的 KeyMint/Weaver 错误。
5. **Wi-Fi**：打开 Wi-Fi 页面扫描并连接；waffle 已经在实机上拿到 DHCP
   地址并能通信。若失败，记录 `rfkill` 与 `dmesg`，见下方。
6. **屏幕方向**：设置 → 屏幕方向，在横屏/竖屏之间切换；Skyward 会重启
   恢复模式应用选择。竖屏下画面应正向，触摸位置应与点按一致。
7. **安装 zip**：先拿一个无关紧要的 ROM/补丁包测试，不要拿主力系统冒险。
8. **备份/还原**：备份 boot 分区即可，不必第一次就备份 data。
9. **电池/振动**：状态栏应有电池图标；设置里的触摸振动开关和振动强度应可调，
   点按应有振动。waffle 的 Wi-Fi 首次出现约需 70 秒（CNSS 等 DMS 超时）。
10. **重启**：`重启到系统` 应能回到已安装系统；再 `adb reboot recovery`
    应能回到 Skyward。

## 失败时抓日志

```sh
adb shell id                         > id.txt
adb shell getprop                    > props.txt
adb shell dmesg                      > dmesg.txt
adb shell cat /tmp/recovery.log      > recovery.log 2>/dev/null
adb logcat -d                        > logcat.txt
```

另外拍下屏幕上的错误页。Skyward 的失败原因会写到 kernel log（`dmesg`），
以及 `/tmp/recovery.log`。

## 回滚

```sh
# piano
fastboot flash recovery_a stock/piano/recovery.img
fastboot flash recovery_b stock/piano/recovery.img

# waffle（ColorOS 17 原厂）
fastboot flash recovery_a stock/waffle/recovery.img
fastboot flash recovery_b stock/waffle/recovery.img

fastboot reboot recovery
```

`stock/waffle16/recovery.img` 是旧的 ColorOS 16 底包，仅在需要回到 16 时使用。

## 已知问题（首测时重点看）

- **waffle 基带**：`mss` 没有启动。在这台 Milk 内核上启动它会在一分钟内
  把手机重置到 EDL，所以只保留 Wi-Fi；recovery 不需要蜂窝网络，EFS/IMEI
  分区仍可读取和备份。
- **waffle Wi-Fi**：通过写 `/sys/class/rfkill/rfkill0/soft=0` 解锁（OnePlus
  的 `state` 语义是反的）。实机已拿到 `10.148.42.x` 并能通信。
- **waffle MTP**：当前 `mtp = false`，文件传输页可能不可用；先确认 adb 正常，
  再考虑打开 MTP。
- **piano 旋转**：如上，UI 不跟随系统旋转。
- **piano 解密**：已经验证可用（UI 显示“已解密”，`/data`、`/sdcard` 可挂载）。
