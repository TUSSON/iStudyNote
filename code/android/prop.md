
# 屏幕旋转方向
ro.sf.hwrotation 

# 屏幕密度dpi
ro.sf.lcd_density

# adb start with root permission
service.adb.root=1

# connect adb with tcp
stop adbd
setprop service.adb.tcp.port 5555
start adbd
