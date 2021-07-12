adb shell  
setprop service.adb.tcp.port 4444  
exit  
adb tcpip 4444  
adb connect 192.168.202.206:  
