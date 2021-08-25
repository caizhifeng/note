```c
$ModLoad imudp # 引用udp协议的模块
$UDPServerRun 514 # 设置udp协议使用端口

$ModLoad imtcp # 引用tcp协议的模块
$InputTCPServerRun 514 # 设置tcp协议使用端口


#### GLOBAL DIRECTIVES ####

# Where to place auxiliary files
$WorkDirectory /var/lib/rsyslog

# Use default timestamp format
$ActionFileDefaultTemplate RSYSLOG_TraditionalFileFormat

$template Remote,"/var/log/remote/%fromhost-ip%.log" # 设置远程日志存放路径和文件格式
:fromhost-ip, !isequal, "127.0.0.1" ?Remote # 如果是本机日志则不记录
```