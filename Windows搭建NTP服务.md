# Windows 搭建 NTP 服务

## 修改注册表

1、新建文件 ntp.reg

	Windows Registry Editor Version 5.00
	
	[HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services\W32Time\Config]
	"AnnounceFlags"=dword:00000005
	
	[HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services\W32Time\Parameters]
	"Type"="NTP"
	
	[HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services\W32Time\TimeProviders\NtpServer]
	"Enabled"=dword:00000001

2、双击“ntp.reg”，并选择信任

3、将“Windows Time”服务的启动方式设为“自动”

4、连接ntp服务器

	[root@localhost ~]# ntpdate 192.168.1.59
	26 Jun 16:20:16 ntpdate[6197]: step time server 192.168.1.59 offset 0.832311 sec
