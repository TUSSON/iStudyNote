# SN号、MAC地址自定义读写说明  -- 杜松 dusong@rock-chips.com Jun 5, 2015

SN号和MAC地址的查看，可以在系统设置->设备信息->状态

## 系统获取SN号，MAC地址流程

下面分别简要说明设置APK获取SN号、MAC地址的流程

##### 1. SN号获取流程

SN号的获取流程比较简单, 上层就是从 property "ro.serialno" 里读取

```
    1. packages/apps/Settings/src/com/android/settings/deviceinfo/Status.java
        String serial = Build.SERIAL;
    2. frameworks/base/core/java/android/os/Build.java
        public static final String SERIAL = getString("ro.serialno");
```

"ro.serialno"是在**system/core/drmservice/drmservice.c**中设置的,根据文件中宏 *SERIALNO_FROM_IDB* 的配置来选择是根据MAC地址自动生成SN号，还是从idb扇区中读取

```
    if SERIALNO_FROM_IDB //read serialno form idb
        rknand_sys_storage_test_sn();
        property_set("ro.serialno", sn_buf_idb[0] ? sn_buf_idb : "");
    else //auto generate serialno
        generate_device_serialno(10,sn_buf_auto);
        property_set("ro.serialno", sn_buf_auto[0] ? sn_buf_auto : "");
```

**drmservice** 会在开机的时候运行一次，在 **device/rockchip/rksdk/init.rc**

```
    service drmservice /system/bin/drmservice
        class main
        oneshot
```

##### 2. MAC地址获取流程

MAC地址的获取流程相对复杂,从WifiManager->wifi hal->wpa_supplicant_8->kernel/net

```
1. packages/apps/Settings/src/com/android/settings/deviceinfo/Status.java
    WifiInfo wifiInfo = wifiManager.getConnectionInfo();
    String macAddress = wifiInfo == null ? null : wifiInfo.getMacAddress();
2. frameworks/base/wifi/java/android/net/wifi/WifiInfo.java
    void setMacAddress(String macAddress)
        this.mMacAddress = macAddress;
    public String getMacAddress()
        return mMacAddress;
3. frameworks/base/wifi/java/android/net/wifi/WifiManager.java
    public WifiInfo getConnectionInfo()
        return mService.getConnectionInfo();
4. frameworks/base/services/java/com/android/server/wifi/WifiService.java
    public WifiInfo getConnectionInfo()
        return mWifiStateMachine.syncRequestConnectionInfo();
5. frameworks/base/wifi/java/android/net/wifi/WifiStateMachine.java
    mWifiInfo.setMacAddress(mWifiNative.getMacAddress());

    public WifiInfo syncRequestConnectionInfo()
        return mWifiInfo;
6. frameworks/base/wifi/java/android/net/wifi/WifiNative.java
    public String getMacAddress()
        String ret = doStringCommand("DRIVER MACADDR");
            String result = doStringCommandNative(mInterfacePrefix + command);
7. frameworks/base/core/jni/android_net_wifi_WifiNative.cpp
    { "doStringCommandNative", "(Ljava/lang/String;)Ljava/lang/String;",
                    (void*) android_net_wifi_doStringCommand },
        return doStringCommand(env,javaCommand);
            doCommand(env, javaCommand, reply, sizeof(reply));
                ::wifi_command(command.c_str(), reply, &reply_len)
8. hardware/libhardware_legacy/wifi/wifi.c
    int wifi_command(const char *command, char *reply, size_t *reply_len)
        return wifi_send_command(command, reply, reply_len);
            wpa_ctrl_request(ctrl_conn, cmd, strlen(cmd), reply, reply_len, NULL);
9. external/wpa_supplicant_8/src/common/wpa_ctrl.c
    wpa_ctrl_request()
10. external/wpa_supplicant_8/src/drivers/driver_nl80211.c
    if (os_strcasecmp(cmd, "MACADDR") == 0)
        linux_get_ifhwaddr(drv->global->ioctl_sock, bss->ifname, macaddr);
11. external/wpa_supplicant_8/src/drivers/linux_ioctl.c
    int linux_get_ifhwaddr(int sock, const char *ifname, u8 *addr)
        ioctl(sock, SIOCGIFHWADDR, &ifr)
12. kernel/net/core/dev_ioctl.c
   static int dev_ifsioc_locked(struct net *net, struct ifreq *ifr, unsigned int cmd) 
        case SIOCGIFHWADDR:
            memcpy(ifr->ifr_hwaddr.sa_data, dev->dev_addr,
                    min(sizeof ifr->ifr_hwaddr.sa_data, (size_t) dev->addr_len));
```

dev->dev_addr就是wifi驱动需要填写的MAC地址，而wifi驱动会在启动过程中从idb扇区中读取MAC地址， 如果读取到合法的MAC地址就更新, 如下：

```
1.kernel/net/rfkill/rfkill-wlan.c
    int rockchip_wifi_mac_addr(unsigned char *buf)
        GetSNSectorInfo(tempBuf);
            for (i = 445; i <= 450; i++)
                wifi_custom_mac_addr[i-445] = tempBuf[i];
```

##### 3. 如何向idb扇区写入SN、MAC addr等

向idb扇区写入SN号，MAC地址等需要进入bootloader，然后用PC工具UpgradeDllTool写入,详见该工具文档


## 利用vendor扇区自定义SN/MAC地址读写

##### 1.补丁说明

**rk_vendor_read_write_sn_mac_patch/** 目录下共有3个补丁、一个工具，分别为:

- system/core/0001-read-serial-number-form-vendor-sector.patch
   修改drmservice从vendor扇区读取SN号
- kernel/0001-wifi-read-mac-address-form-vendor-sector.patch
    修改wifi驱动从vendor扇区读取MAC地址
- device/rockchip/rk312x/0001-install-drmboot.ko-for-read-write-vendor-sector-thro.patch
    使用emmc存储的需要添加drmboot.ko模块，以支持通过/dev/rknand_sys_storage节点读写vendor扇区
- vendor/tools/rk_vendor_rw.zip
    用户空间向vendor扇区读写SN、MAC addr示例工具

到补丁的目录git am合入补丁，工具直接解压在相关目录，然后重新编译整个系统即可


##### 2.工具说明

工具编译后会在系统的system/bin/目录，可以直接 **adb shell rk_vendor_rw** 运行, *-h* 显帮助

```
>> adb shell rk_vendor_rw -h
Error:Unknown option: ?
vendor sector read/write tools $Revision: v1.1 $

rk_vendor_rw  [-s <sn number>] [-m <mac addr>]

    default read sn number and mac addr
    -s <sn number>  serial number to write in vendor
    -m <mac addr>   mac addr to write in vendor
    Examples:
    rk_vendor_rw -s "32XSDF32ASD" -m "54:32:10:ab:cd:ef"

```

*-s,-m* 可以用来写入SN号和MAC地址，不带参数会读取vendor扇区中存储的SN号和MAC地址

用工具写入成功后需要重新启动才能在设置里面看到最新的SN号和MAC地址
