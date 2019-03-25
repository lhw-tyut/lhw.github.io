---
layout:     post
title:     windows下使用cloudbase-init
subtitle:   修改密码/ip/hostname
date:       2019-03-17
author:     永泉狂客
header-img: img/post-bg-keybord.jpg
catalog: true
tags:
    - cloudbase-init
---

使用VMware虚拟机做测试示例
## 一.安装windows server 2016
略
## 二.做一些简单配置
- 设置密码策略为 disable
Win+R 打开【运行】对话框，输入 gpedit.msc
![](https://s2.ax1x.com/2019/03/25/AtUyrt.jpg)

## 三.下载cloudbase-init

- 网址: https://cloudbase.it/cloudbase-init/#download
- 设置配置文件cloudbase-init.conf
本例使用openstack(configuration drive),从cdrom,vfat或原始磁盘/分区中检索数据,所以需要创建一个label为config-2,大小为64MB，文件类型为vfat(fat32)的分区.
reference: [openstack-configuration-drive](https://cloudbase-init.readthedocs.io/en/latest/services.html#openstack-configuration-drive)
```
[DEFAULT]
username=Administrator
groups=Administrators
inject_user_password=true
config_drive_raw_hhd=true
config_drive_cdrom=true
config_drive_vfat=true
bsdtar_path=C:\Program Files\Cloudbase Solutions\Cloudbase-Init\bin\bsdtar.exe
mtools_path=C:\Program Files\Cloudbase Solutions\Cloudbase-Init\bin\
verbose=true
debug=true
logdir=C:\Program Files\Cloudbase Solutions\Cloudbase-Init\log\
logfile=cloudbase-init.log
default_log_levels=comtypes=INFO,suds=INFO,iso8601=WARN,requests=WARN
logging_serial_port_settings=COM1,115200,N,8
mtu_use_dhcp_config=true
ntp_use_dhcp_config=true
local_scripts_path=C:\Program Files\Cloudbase Solutions\Cloudbase-Init\LocalScripts\
# Services that will be tested for loading until one of them succeeds.
metadata_services=cloudbaseinit.metadata.services.configdrive.ConfigDriveService,
#cloudbaseinit.metadata.services.httpservice.HttpService,
#cloudbaseinit.metadata.services.ec2service.EC2Service,
#cloudbaseinit.metadata.services.maasservice.MaaSHttpService
# What plugins to execute.
plugins=cloudbaseinit.plugins.common.sethostname.SetHostNamePlugin,
        cloudbaseinit.plugins.common.userdata.UserDataPlugin,
        cloudbaseinit.plugins.common.networkconfig.NetworkConfigPlugin,
		    cloudbaseinit.plugins.common.setuserpassword.SetUserPasswordPlugin
# Miscellaneous.
allow_reboot=false
# allow the service to reboot the system
stop_service_on_exit=false
```
- 修改文件setuserpassword.py
```
# C:\Program Files\Cloudbase Solutions\Cloudbase-Init\Python\Lib\site-packages\cloudbaseinit\plugins\common\setuserpassword.py
self._change_logon_behaviour(user_name, password_injected=injected)   #将此行注释
```

## 四. 创建元数据分区

1. Win+R 打开【运行】对话框，输入 diskmgmt.msc
2. 创建config-2分区
![](https://s2.ax1x.com/2019/03/25/AtURIS.jpg)
![](https://s2.ax1x.com/2019/03/25/AtU2a8.jpg)
![](https://s2.ax1x.com/2019/03/25/AtU6qP.jpg)
在该分区下新建目录/openstack/latest,用于存放元数据，如meta_data.json

3. 写入元数据
```
# meta_data.json
{"hostname": "test.local", "meta": {"admin_pass": "123456"}, "uuid": "83679162-1378-4288-a2d4-70e13ec132aa"}
```
```
# network_data.json
{
    "networks": [
        {
            "netmask": "255.255.255.0",
            "link": "Ethernet0",
            "type": "ipv4",
            "ip_address": "10.177.178.35",
            "routes":   [
                {
                    "network": "0.0.0.0",
                    "netmask": "0.0.0.0",
                    "gateway": "10.177.178.254"
                }
                        ],
            "services": [
                {
                    "address": "114.114.114.114",
                    "type": "dns"
                }
                        ]
        },
        {
            "link": "vlan0",
            "netmask": "255.255.255.0",
            "ip_address": "12.12.12.12",
            "type": "ipv4",
            "routes":  [
                {
                    "network": "0.0.0.0",
                    "netmask": "0.0.0.0",
                    "gateway": "12.12.12.1"
                }
                        ],
            "services": [
                {
                    "address": "114.114.114.114",
                    "type": "dns"
                }
                        ]
        }
                ],
    "links":    [
        {
            "type": "phy",
            "ethernet_mac_address": "6c:92:bf:84:59:6a",
            "id": "Ethernet0"
        },
        {
            "type": "phy",
            "ethernet_mac_address": "6c:92:bf:62:ab:9e",
            "id": "Ethernet1"
        },
        {
            "type": "phy",
            "ethernet_mac_address": "6c:92:bf:62:ab:9f",
            "id": "Ethernet2"
        },
        {
            "ethernet_mac_address": "6c:92:bf:62:ab:9e",
            "bond_mode": "balance-alb",
            "bond_links": ["Ethernet1", "Ethernet2"],
            "type": "bond",
            "id": "bond0"
        },
        {
            "vlan_mac_address": "6c:92:bf:62:ab:9e",
            "vlan_link": "bond0",
            "type": "vlan",
            "id": "vlan0",
            "vlan_id": 2000
        }
                ],
    "services": [
        {
            "address": "114.114.114.114",
            "type": "dns"
        }
                ]
}
```

## 五. 启动cloudbase-init
- 执行日志
```
2019-03-25 16:22:34.493 800 INFO cloudbaseinit.init [-] Metadata service loaded: 'ConfigDriveService'
2019-03-25 16:22:34.493 800 DEBUG cloudbaseinit.init [-] Instance id: None configure_host c:\program files\cloudbase solutions\cloudbase-init\python\lib\site-packages\cloudbaseinit\init.py:202
2019-03-25 16:22:34.509 800 DEBUG cloudbaseinit.utils.classloader [-] Loading class 'cloudbaseinit.plugins.common.sethostname.SetHostNamePlugin' load_class c:\program files\cloudbase solutions\cloudbase-init\python\lib\site-packages\cloudbaseinit\utils\classloader.py:27
2019-03-25 16:22:34.509 800 DEBUG cloudbaseinit.utils.classloader [-] Loading class 'cloudbaseinit.plugins.common.networkconfig.NetworkConfigPlugin' load_class c:\program files\cloudbase solutions\cloudbase-init\python\lib\site-packages\cloudbaseinit\utils\classloader.py:27
2019-03-25 16:22:34.509 800 DEBUG cloudbaseinit.utils.classloader [-] Loading class 'cloudbaseinit.plugins.common.setuserpassword.SetUserPasswordPlugin' load_class c:\program files\cloudbase solutions\cloudbase-init\python\lib\site-packages\cloudbaseinit\utils\classloader.py:27
2019-03-25 16:22:34.509 800 INFO cloudbaseinit.init [-] Executing plugins for stage 'MAIN':
2019-03-25 16:22:34.509 800 INFO cloudbaseinit.init [-] Executing plugin 'SetHostNamePlugin'
2019-03-25 16:22:34.509 800 DEBUG cloudbaseinit.utils.classloader [-] Loading class 'cloudbaseinit.osutils.windows.WindowsUtils' load_class c:\program files\cloudbase solutions\cloudbase-init\python\lib\site-packages\cloudbaseinit\utils\classloader.py:27
2019-03-25 16:22:34.509 800 DEBUG cloudbaseinit.metadata.services.base [-] Using cached copy of metadata: 'openstack/latest/meta_data.json' _get_cache_data c:\program files\cloudbase solutions\cloudbase-init\python\lib\site-packages\cloudbaseinit\metadata\services\base.py:73
2019-03-25 16:22:34.509 800 INFO cloudbaseinit.utils.hostname [-] Setting hostname: test1
2019-03-25 16:22:34.539 800 INFO cloudbaseinit.init [-] Executing plugin 'NetworkConfigPlugin'
2019-03-25 16:22:34.539 800 DEBUG cloudbaseinit.utils.classloader [-] Loading class 'cloudbaseinit.osutils.windows.WindowsUtils' load_class c:\program files\cloudbase solutions\cloudbase-init\python\lib\site-packages\cloudbaseinit\utils\classloader.py:27
2019-03-25 16:22:34.539 800 DEBUG cloudbaseinit.plugins.common.networkconfig [-] Enable network adapter "Ethernet0": True _process_link_common c:\program files\cloudbase solutions\cloudbase-init\python\lib\site-packages\cloudbaseinit\plugins\common\networkconfig.py:185
2019-03-25 16:22:34.726 800 INFO cloudbaseinit.plugins.common.networkconfig [-] Setting static IP configuration on network adapter "Ethernet0". IP: 192.168.179.222, prefix length: 24, gateway: 192.168.179.2, dns: ['114.114.114.114']
2019-03-25 16:22:34.851 800 DEBUG cloudbaseinit.osutils.windows [-] Removing existing IP address "192.168.179.222" from adapter "Ethernet0" _set_static_network_config c:\program files\cloudbase solutions\cloudbase-init\python\lib\site-packages\cloudbaseinit\osutils\windows.py:917
2019-03-25 16:22:34.898 800 DEBUG cloudbaseinit.osutils.windows [-] Removing existing route "0.0.0.0/0" from adapter "Ethernet0" _set_static_network_config c:\program files\cloudbase solutions\cloudbase-init\python\lib\site-packages\cloudbaseinit\osutils\windows.py:926
2019-03-25 16:22:34.976 800 INFO cloudbaseinit.init [-] Executing plugin 'SetUserPasswordPlugin'
2019-03-25 16:22:34.976 800 DEBUG cloudbaseinit.utils.classloader [-] Loading class 'cloudbaseinit.osutils.windows.WindowsUtils' load_class c:\program files\cloudbase solutions\cloudbase-init\python\lib\site-packages\cloudbaseinit\utils\classloader.py:27
2019-03-25 16:22:34.992 800 DEBUG cloudbaseinit.metadata.services.base [-] Using cached copy of metadata: 'openstack/latest/meta_data.json' _get_cache_data c:\program files\cloudbase solutions\cloudbase-init\python\lib\site-packages\cloudbaseinit\metadata\services\base.py:73
2019-03-25 16:22:34.992 800 WARNING cloudbaseinit.plugins.common.setuserpassword [-] Using admin_pass metadata user password. Consider changing it as soon as possible
2019-03-25 16:22:34.992 800 INFO cloudbaseinit.plugins.common.setuserpassword [-] Password succesfully updated for user Administrator
```

## 六. 问题
1. 配置bond
初始化时可以成功配置bond，执行修改操作时会出现报错
```
2019-03-20 01:05:13.646 96 INFO cloudbaseinit.init [-] Executing plugin 'NetworkConfigPlugin'
2019-03-20 01:05:13.646 96 DEBUG cloudbaseinit.utils.classloader [-] Loading class 'cloudbaseinit.osutils.windows.WindowsUtils' load_class c:\program files\cloudbase solutions\cloudbase-init\python\lib\site-packages\cloudbaseinit\utils\classloader.py:27
2019-03-20 01:05:13.677 96 DEBUG cloudbaseinit.plugins.common.networkconfig [-] Enable network adapter "Ethernet0": True _process_link_common c:\program files\cloudbase solutions\cloudbase-init\python\lib\site-packages\cloudbaseinit\plugins\common\networkconfig.py:185
2019-03-20 01:05:14.021 96 ERROR cloudbaseinit.init [-] plugin 'NetworkConfigPlugin' failed with error 'Multiple network interfaces with MAC address "00:00:00:00:00:00" exist': cloudbaseinit.exception.CloudbaseInitException: Multiple network interfaces with MAC address "00:00:00:00:00:00" exist
2019-03-20 01:05:14.052 96 ERROR cloudbaseinit.init [-] Multiple network interfaces with MAC address "00:00:00:00:00:00" exist: cloudbaseinit.exception.CloudbaseInitException: Multiple network interfaces with MAC address "00:00:00:00:00:00" exist
2019-03-20 01:05:14.052 96 ERROR cloudbaseinit.init Traceback (most recent call last):
2019-03-20 01:05:14.052 96 ERROR cloudbaseinit.init   File "c:\program files\cloudbase solutions\cloudbase-init\python\lib\site-packages\cloudbaseinit\init.py", line 67, in _exec_plugin
2019-03-20 01:05:14.052 96 ERROR cloudbaseinit.init     shared_data)
2019-03-20 01:05:14.052 96 ERROR cloudbaseinit.init   File "c:\program files\cloudbase solutions\cloudbase-init\python\lib\site-packages\cloudbaseinit\plugins\common\networkconfig.py", line 305, in execute
2019-03-20 01:05:14.052 96 ERROR cloudbaseinit.init     return self._process_network_details_v2(network_details)
2019-03-20 01:05:14.052 96 ERROR cloudbaseinit.init   File "c:\program files\cloudbase solutions\cloudbase-init\python\lib\site-packages\cloudbaseinit\plugins\common\networkconfig.py", line 294, in _process_network_details_v2
2019-03-20 01:05:14.052 96 ERROR cloudbaseinit.init     osutils, network_details)
2019-03-20 01:05:14.052 96 ERROR cloudbaseinit.init   File "c:\program files\cloudbase solutions\cloudbase-init\python\lib\site-packages\cloudbaseinit\plugins\common\networkconfig.py", line 196, in _process_physical_links
2019-03-20 01:05:14.052 96 ERROR cloudbaseinit.init     link.mac_address)
2019-03-20 01:05:14.052 96 ERROR cloudbaseinit.init   File "c:\program files\cloudbase solutions\cloudbase-init\python\lib\site-packages\cloudbaseinit\osutils\windows.py", line 756, in get_network_adapter_name_by_mac_address
2019-03-20 01:05:14.052 96 ERROR cloudbaseinit.init     mac_address)
2019-03-20 01:05:14.052 96 ERROR cloudbaseinit.init cloudbaseinit.exception.CloudbaseInitException: Multiple network interfaces with MAC address "00:00:00:00:00:00" exist
```
