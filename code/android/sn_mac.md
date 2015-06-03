serial number get flow

1. packages/apps/Settings/src/com/android/settings/deviceinfo/Status.java
    String serial = Build.SERIAL;
2. frameworks/base/core/java/android/os/Build.java
    public static final String SERIAL = getString("ro.serialno");
3. system/core/drmservice/drmservice.c
    if SERIALNO_FROM_IDB //read serialno form idb
        rknand_sys_storage_test_sn();
        property_set("ro.serialno", sn_buf_idb[0] ? sn_buf_idb : "");
    else //auto generate serialno
        generate_device_serialno(10,sn_buf_auto);
        property_set("ro.serialno", sn_buf_auto[0] ? sn_buf_auto : "");

mac address get flow

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
        if (ioctl(sock, SIOCGIFHWADDR, &ifr))   
12. kernel/net/
