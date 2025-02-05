---
# **知识梳理**

---
> [!TIP]
>  
>  以下内容仅供参考，欢迎批评指正
>
>  作者：蒋光明
>
>  联系方式：gmjiang116@gmail.com

---
# **OpenWRT**
---

## 1. 简介
* OpenWRT 是一个开源项目， 是一个基于 Linux 内核开发的路由系统，由于 Linux 使用 GPL, 因此基于此开发的 WRT 系统也就随之开源，即 OpenWRT.
* OpenWRT 是一个嵌入式 Linux 系统，高度模块化、高度自动化，拥有强大的网络组件和扩展性.

## 2. LEDE 项目
* LEDE Linux Embeded Development Environment 就是 OpenWRT 的一个分支.
* 这个分支会对新的硬件进行支持，许多新功能的代码都贡献到了这个分支，以至于原本的 OpenWRT 都受到了影响
    * 2016 年 LEDE 开始分支
    * 2017 年 第一个稳定版本
    * 2018 年 重新合并入 OpenWRT

## 3. 架构支持
 arch    |  sub arch
---------|--------------------------------------------------------------
ARM      | arm64/generic, layerscape/64b, arc770/generic, archs38/generic, brcm2708/bcm2708, ...
MIPS64   | ar71xx/generic, ar71xx/nand, ar71xx/mikrotik, lantiq/xrx200, lantiq/xway, ...
MIPS     | ar71xx/generic, ramips/mt7628, ramips/mt7688, ...
PowerPC  | ppc44x/generic, ...
X86      | x86/geode, x86/legacy, x86/generic
X86_64   | x86/64

* more inforamtion: https://openwrt.org/zh/docs/techref/targets/start

## 4. OpenWRT 对硬件设备要求
* OpenWrt 支持的架构
* 有足够的闪存来储存 OpenWrt 固件镜像
    * 最少 4MB，无法安装 GUI(LuCI)
    * 推荐 8MB 及以上，能安装GUI和其他应用程序
* 有足够的内存 (RAM) 来保证稳定运行
    * 最少 32MB，推荐 64MB 或以上

## 5. OpenWRT 源代码
* 任何在 openwrt.git 这个主仓库中产生的 LEDE 进化都可以通过 HTTP 和 HTTPS 方式访问:
    * git clone https://git.openwrt.org/openwrt/openwrt.git
* 还可以使用以下命令，找到源代码仓库在 Github 上的一个镜像:
    * git clone https://github.com/openwrt/openwrt.git

## 6. OpenWRT 编译
* 需要确认所有的依赖软件已安装。下面以 Debian/Ubuntu 为例:
  
    ```
    sudo apt-get install subversion g++ zlib1g-dev build-essential git python rsync man-db
    sudo apt-get install libncurses5-dev gawk gettext unzip file libssl-dev wget zip time
    ```

* 接下来，使用以下命名获取 OpenWrt 的源代码:    
    ```
    git clone https://git.openwrt.org/openwrt/openwrt.git/
    cd openwrt
    
    ./scripts/feeds update -a
    ./scripts/feeds install -a
    
    make menuconfig
    ```
* `make menuconfig` 命令将打开一个菜单，如果您想为 “WRTnode2R” 这款无线路由构建固件，您可以这样设置:
    ```
    Target System (MediaTek Ralink MIPS)
    Subtarget (MT76x8 based boards)
    Target Profile (WRTnode WRTnode 2R)
    ```

* 选择保存并退出设置
    ```
    make or make V=s -jN #Change N to your CPU core number
    ```

* 漫长的编译等待...
    * 编译完成后，固件可以在目录 `/openwrt/bin/targets/ramips/mt76x8/` 中找到 `openwrt-ramips-mt76x8-wrtnode_wrtnode2r-squashfs-sysupgrade.bin`

* 固件烧写
    * 方法1： 通过 U-Boot download 固件并烧写到 flash
    * 方法2： 通过 sysupgrade 进行固件升级
    * 方法3： 通过 Luci web server 进行固件升级
    * 方法4： 通过 mtd 命令进行对应的分区烧写 (Linux系统对闪存类存储器是采用 MTD 设备驱动实现的) 

## 7. 软件包编译
* make V=s package/package-name/option
* 
    Option     | Function
    -----------|--------------------------------------------------------
    clean      | Clean the package build directory
    prepare    | Unpack and patch the sources
    configure  | Run configuration scripts if needed
    compile    | Build the package sources
    install    | Copy files from the compiled source into the ipkg


## 8. OpenWRT BuildRoot 环境
* OpenWRT Buildroot 环境是 Makefiles, patches 和一些列脚本组成，通过这些脚本去生成 cross-compilation toolchain，下载 Linux kernel，生成一个根文件系统，管理第三方的软件包，等等。 cross-compilation toolchain 使用 uClibc。OpenWRT buildroot 中是没有 Linux kernel 以及任何第3方软件的源码 tar 包，编译过程中，编译脚本会去决定使用那个版本的 Linux kernel，并且会去指定的 url 下载，同样的，编译脚本也会指定需要下载那个版本的软件包的参与编译。
<br>

## 9. OpenWRT Network
* 硬件框图

```
                +-----------------------------+
                | cpu                         |
                |                      +------|
                |            br0-------| eth1 |---------------------------------------------------------------+
                |            |         +------| (WLAN)                                                        |
                |       vlan1 vlan0           |                                                               |
                |         |     |             |                                                               |
                |      +-----------+          |                                                               |
                |      |  Tagging  |          |                                                               |
                |      +-----------+          |                                                               |
                |           |                 |                                                               |
                |        +------+             |                                                               |
                |        | eth0 |             |                                                               |
                +-----------------------------+                                                               |
                            ||                                                                                |
                        +-----------+                                                                         |
+-----------------------|  Port5    |-------------SWITCH-------------------------------------------+          |
|                       +-----------+                                                              |          |
|                            ||                                                                    |          |
|                       +-----------+                                                              |          |
|      +----vlan1-------|  Tagging  |-----vlan0----+                                               |          |
|      |                +-----------+              |                                               |          |
|      |                                           |                                               |          |
|      |                       +-------------------+-----------------+------------------+          |          |
|      |                       |                   |                 |                  |          |          |
|  +-----------+          +-----------+      +-----------+      +-----------+      +-----------+   |    +-----------+
+--|  Port0    |-------- -|  Port1    |------|  Port2    |------|  Port3    |------|  Port4    |---+    |   Wi-Fi   |
   +-----------+          +-----------+      +-----------+      +-----------+      +-----------+        +-----------+
      WLAN                   LAN1                LAN2                LAN3               LAN4                WLAN


```

* `ifconfig` 查看当前网络配置

```
root@LEDE:~# ifconfig
br-lan    Link encap:Ethernet  HWaddr 64:51:7E:80:2D:74  //虚拟设备, LAN 口桥接设备
          inet addr:192.168.1.1  Bcast:192.168.1.255  Mask:255.255.255.0
          inet6 addr: fd40:cf3a:6f76::1/60 Scope:Global
          inet6 addr: fe80::6651:7eff:fe80:2d74/64 Scope:Link
          UP BROADCAST RUNNING MULTICAST  MTU:1500  Metric:1
          RX packets:87647 errors:0 dropped:0 overruns:0 frame:0
          TX packets:18306 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:1000
          RX bytes:125039159 (119.2 MiB)  TX bytes:4009166 (3.8 MiB)

eth0      Link encap:Ethernet  HWaddr 64:51:7E:80:2D:74  //真实设备, CPU 内部到 Switch 之间的一个接口
          inet6 addr: fe80::6651:7eff:fe80:2d74/64 Scope:Link
          UP BROADCAST RUNNING MULTICAST  MTU:1500  Metric:1
          RX packets:87322 errors:0 dropped:1 overruns:0 frame:0
          TX packets:16007 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:1000
          RX bytes:14940946 (14.2 MiB)  TX bytes:3020745 (2.8 MiB)
          Interrupt:5

eth0.1    Link encap:Ethernet  HWaddr 64:51:7E:80:2D:74  //虚拟设备，从 eth0 虚拟出来，有 VLAN 划分的有线 LAN 口, VLAN 编号为 1
          UP BROADCAST RUNNING MULTICAST  MTU:1500  Metric:1
          RX packets:0 errors:0 dropped:0 overruns:0 frame:0
          TX packets:8738 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:1000
          RX bytes:0 (0.0 B)  TX bytes:2098622 (2.0 MiB)

eth0.2    Link encap:Ethernet  HWaddr 64:51:7E:80:2D:75  //虚拟设备，从 eth0 虚拟出来，有 VLAN 划分的有线 WAN 口, VLAN 编号为 2
          inet addr:162.168.10.60  Bcast:162.168.10.255  Mask:255.255.255.0
          inet6 addr: fe80::6651:7eff:fe80:2d75/64 Scope:Link
          UP BROADCAST RUNNING MULTICAST  MTU:1500  Metric:1
          RX packets:87310 errors:0 dropped:11989 overruns:0 frame:0
          TX packets:7250 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:1000
          RX bytes:13368266 (12.7 MiB)  TX bytes:854435 (834.4 KiB)

lo        Link encap:Local Loopback                     //虚拟设备，回环设备
          inet addr:127.0.0.1  Mask:255.0.0.0
          inet6 addr: ::1/128 Scope:Host
          UP LOOPBACK RUNNING  MTU:65536  Metric:1
          RX packets:930 errors:0 dropped:0 overruns:0 frame:0
          TX packets:930 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:1000
          RX bytes:89011 (86.9 KiB)  TX bytes:89011 (86.9 KiB)

wlan0     Link encap:Ethernet  HWaddr 64:51:7E:80:2D:74 //真实设备，启动 Wi-Fi 后将会产生此无线设备
          inet6 addr: fe80::6651:7eff:fe80:2d74/64 Scope:Link
          UP BROADCAST RUNNING MULTICAST  MTU:1500  Metric:1
          RX packets:100128 errors:0 dropped:0 overruns:0 frame:0
          TX packets:27172 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:1000
          RX bytes:126940539 (121.0 MiB)  TX bytes:6626975 (6.3 MiB)

```

* OpenWRT Network 配置查看
1. 通过配置文件查看 `vim /etc/config/network`

```
config interface 'loopback'
        option ifname 'lo'
        option proto 'static'
        option ipaddr '127.0.0.1'
        option netmask '255.0.0.0'

config globals 'globals'
        option ula_prefix 'fd40:cf3a:6f76::/48'

config interface 'lan'
        option type 'bridge'
        option ifname 'eth0.1'
        option proto 'static'
        option ipaddr '192.168.1.1'
        option netmask '255.255.255.0'
        option ip6assign '60'

config device 'lan_dev'
        option name 'eth0.1'
        option macaddr '64:51:7e:80:2d:74'

config interface 'wan'
        option ifname 'eth0.2'
        option proto 'dhcp'  //配置为 dhcp

config device 'wan_dev'
        option name 'eth0.2'
        option macaddr '64:51:7e:80:2d:75'

config interface 'wan6'
        option ifname 'eth0.2'
        option proto 'dhcpv6'

config switch
        option name 'switch0'
        option reset '1'
        option enable_vlan '1'

config switch_vlan
        option device 'switch0'
        option vlan '1'
        option ports '1 2 3 4 6t'  //port1 port2 port3 port4 --> lan, port6 --> cpu

config switch_vlan
        option device 'switch0'
        option vlan '2'
        option ports '0 6t'       //port0 --> wan

```

2. 通过 UCI 查看, `uci show network`

```
network.loopback=interface
network.loopback.ifname='lo'
network.loopback.proto='static'
network.loopback.ipaddr='127.0.0.1'
network.loopback.netmask='255.0.0.0'
network.globals=globals
network.globals.ula_prefix='fd40:cf3a:6f76::/48'
network.lan=interface
network.lan.type='bridge'
network.lan.ifname='eth0.1'
network.lan.proto='static'
network.lan.ipaddr='192.168.1.1'
network.lan.netmask='255.255.255.0'
network.lan.ip6assign='60'
network.lan_dev=device
network.lan_dev.name='eth0.1'
network.lan_dev.macaddr='64:51:7e:80:2d:74'
network.wan=interface
network.wan.ifname='eth0.2'
network.wan.proto='dhcp'
network.wan_dev=device
network.wan_dev.name='eth0.2'
network.wan_dev.macaddr='64:51:7e:80:2d:75'
network.wan6=interface
network.wan6.ifname='eth0.2'
network.wan6.proto='dhcpv6'
network.@switch[0]=switch
network.@switch[0].name='switch0'
network.@switch[0].reset='1'
network.@switch[0].enable_vlan='1'
network.@switch_vlan[0]=switch_vlan
network.@switch_vlan[0].device='switch0'
network.@switch_vlan[0].vlan='1'
network.@switch_vlan[0].ports='1 2 3 4 6t'
network.@switch_vlan[1]=switch_vlan
network.@switch_vlan[1].device='switch0'
network.@switch_vlan[1].vlan='2'
network.@switch_vlan[1].ports='0 6t'

```

3. wireless 配置 `vim /etc/config/wireless`

```
config wifi-device 'radio0'
        option type 'mac80211'
        option hwmode '11g'
        option path 'platform/10300000.wmac'
        option htmode 'HT40'
        option legacy_rates '1'
        option channel '11'
        option txpower '20'
        option country 'CN'
        option cell_density '2'
        option disabled '0'

config wifi-iface 'default_radio0'
        option device 'radio0'
        option network 'lan'
        option mode 'ap'
        option ssid 'OpenWRT'
        option encryption 'psk2'
        option key '12345678'
        option disabled '0'
```

4. ssh 配置 `vim /etc/config/dropbear`

```
config dropbear
        option PasswordAuth 'on'
        option RootPasswordAuth 'on'
        option Port         '22'
#       option BannerFile   '/etc/banner'


```

5. dhcp 配置 `vim /etc/config/dhcp`

```
config dnsmasq
        option domainneeded '1'
        option localise_queries '1'
        option rebind_protection '1'
        option rebind_localhost '1'
        option local '/lan/'
        option domain 'lan'
        option expandhosts '1'
        option authoritative '1'
        option readethers '1'
        option leasefile '/tmp/dhcp.leases'
        option localservice '1'
        option nonwildcard '0'
        option resolvfile '/tmp/resolv.conf.d/resolv.conf.auto'
        list server '114.114.114.114'
        list server '192.168.1.1'

config dhcp 'lan'
        option interface 'lan'
        option start '100'
        option limit '150'
        option leasetime '12h'
        option dhcpv6 'server'
        option ra 'server'

config dhcp 'wan'
        option interface 'wan'
        option ignore '1'

config odhcpd 'odhcpd'
        option maindhcp '0'
        option leasefile '/tmp/hosts/odhcpd'
        option leasetrigger '/usr/sbin/odhcpd-update'


```

* 这里的每一项代表什么意思在 openwrt 官方文档 http://wiki.openwrt.org/doc/uci/dhcp 中有详细说明。

## 10. 配置 DHCP 服务器和 DNS 服务器
* openwrt 使用同一个程序 dnsmasq 来实现 DHCP 服务器和 DNS 服务器。dhcpd 是一个 DHCP 服务器，openwrt 未使用。
dhcpd 和 dnsmasq 在软件包配置中默认都是选中的，既然 dhcpd 没有使用，就可以不编译 dhcpd，配置 Openwrt 如下

```
ase system --->
    <*> dnsmasq
    < > dnsmasq-dhcpv6
    < > dnsmasq-full
Network --->
    < > odhcpd

```

* DHCP 服务器和 DNS 服务器的配置文件为/etc/config/dhcp
默认配置文件中包括一个公用的 section 来配置 DNS 和程序相关的一些 option，以及一个或多个为每个网络接口定义
DHCP 地址池的 section。

## 11. 添加软件包
* 下面用 `<BUILDROOT>` 表示 Openwrt 源码树顶层目录。Openwrt 所有的软件包都都保存在`<BUILDROOT>/package` 目录下，
一个软件包占一个子目录。一个典型的 Openwrt 用户空间软件包，例如 helloworld，如下所示<br>
`<BUILDROOT>/package/helloworld/Makefile`<br>
`<BUILDROOT>/package/helloworld/files/`<br>
`<BUILDROOT>/package/helloworld/patches/`<br>
其中 files 目录是可选的，一般存放配置文件和启动脚本。patches 目录也是可选的，一般存放 bug 修复或者程序优化的
补丁。其中 Makefile 是必须的也是最重要的，它具有和我们所熟悉的 GNU Makefile 完全不同的语法，它定义了如何构建一个Openwrt 软件包，包括从哪里下载源码，怎样编译和安装。

* OpenWRT Makefile example

```Makefile
##############################################
# OpenWrt Makefile for helloworld program
#
#
# Most of the variables used here are defined in
# the include directives below. We just need to
# specify a basic description of the package,
# where to build our program, where to find
# the source files, and where to install the
# compiled program on the router.
#
# Be very careful of spacing in this file.
# Indents should be tabs, not spaces, and
# there should be no trailing whitespace in
# lines that are not commented.
#
##############################################

include $(TOPDIR)/rules.mk

# Name and release number of this package
PKG_NAME:=helloworld
PKG_RELEASE:=1


# This specifies the directory where we're going to build the program. 
# The root build directory, $(BUILD_DIR), is by default the build_mipsel
# directory in your OpenWrt SDK directory
PKG_BUILD_DIR := $(BUILD_DIR)/$(PKG_NAME)

include $(INCLUDE_DIR)/package.mk
include $(INCLUDE_DIR)/kernel.mk

# Specify package information for this program.
# The variables defined here should be self explanatory.
# If you are running Kamikaze, delete the DESCRIPTION
# variable below and uncomment the Kamikaze define
# directive for the description below
define Package/helloworld
    SECTION:=utils
    CATEGORY:=Utilities
    DEPENDS:=+libpthread +libv4l +libjpeg
    TITLE:=helloworld -- this is a helloworld-example
endef


# Uncomment portion below for Kamikaze and delete DESCRIPTION variable above
define Package/helloworld/description
        If you can't figure out what this program does, you're probably
        brain-dead and need immediate medical attention.
endef



# Specify what needs to be done to prepare for building the package.
# In our case, we need to copy the source files to the build directory.
# This is NOT the default.  The default uses the PKG_SOURCE_URL and the
# PKG_SOURCE which is not defined here to download the source from the web.
# In order to just build a simple program that we have just written, it is
# much easier to do it this way.
define Build/Prepare
	mkdir -p $(PKG_BUILD_DIR)
	$(CP) ./src/* $(PKG_BUILD_DIR)/
endef

TARGET_CFLAGS += \
	-I$(LINUX_DIR)/drivers/char \
	$(foreach c, $(PKG_KCONFIG),$(if $(CONFIG_$c),-DCONFIG_$(c)))

# We do not need to define Build/Configure or Build/Compile directives
# The defaults are appropriate for compiling a simple program such as this one


# Specify where and how to install the program. Since we only have one file,
# the helloworld executable, install it by copying it to the /bin directory on
# the router. The $(1) variable represents the root directory on the router running
# OpenWrt. The $(INSTALL_DIR) variable contains a command to prepare the install
# directory if it does not already exist.  Likewise $(INSTALL_BIN) contains the
# command to copy the binary file from its current location (in our case the build
# directory) to the install directory.
define Package/helloworld/install
	$(INSTALL_DIR) $(1)/bin
	$(INSTALL_BIN) $(PKG_BUILD_DIR)/helloworld $(1)/bin/
endef


# This line executes the necessary commands to compile our program.
# The above define directives specify all the information needed, but this
# line calls BuildPackage which in turn actually uses this information to
# build a package.
$(eval $(call BuildPackage,helloworld))

```

## 12. OpenWRT 的软件启动机制

1. init 进程
  * init进程是所有系统进程的父进程，它被内核调用起来并负责调用所有其他的进程
2. OpenWRT 软件启动机制
  * 内核启动完成后读取/etc/inittab文件，然后执行inittab中的sysinit所指的脚本（/etc/init.d/rcS）
  * 如果按照通常的简单做法：我们会将每一个待启动的程序启动命令按行放入rcS文件中，并顺序执行。这种实现方法在软件启动进程列表不变时工作得非常好，如果需要动态修改，则不容易以程序来控制（在OpenWrt下，使用ls命令查看不到/etc/init.d/rcS这个文件）。OpenWrt引入了一个便于控制的启动机制，这种机制是在/etc/rc.d目录下创建每个软件的软链接方式，由rcS脚本在该目录读取启动命令的软链接， 然后启动软链接所指向的程序，由于每一个软链接均包含一个数字，这样就可以按照数字顺序读取并进行启动了
具体执行流程：执行/etc/init.d/rcS脚本时，给脚本传递两个参数（分别为S何boot），接着rcS脚本通过run_scripts函数来启动软件，将每一个以/etc/rc.d/S开头的脚本按照数字传递boot参数并调用
    * S：表示软件启动模块
    * K：软件关闭
    * boot：表示首次启动

3. `/etc/init.d/*` 脚本

	| 函 数    | 含 义                                                                 |
	| -------- | --------------------------------------------------------------------- |
	| start	   | 启动服务 相当于 C++ 语言中的虚函数, 通常情况下每一个服务均需重写该函数  |
	| stop	   | 关闭服务 相当于 C++ 语言中的虚函数, 通常情况下每一个服务均需重写该函数  |
	| restart  | 重启服务 调用 stop 函数退出进程, 然后再调用 start 函数启动进程          |
	| reload   | 重新读取配置，如果读取配置失败则调用 restart 函数重启进程               |
	| enable   | 打开服务自启动, 即将启动脚本软链接文件放在/etc/rc.d 目录下              |
	| disable  | 关闭服务自启动, 删除在/etc/rc.d 的软链接文件                           |
	| enabled  | 提供服务自启动的状态查询                                               |
	| boot     | 调用 start 函数                                                        |
	| shutdown | 调用 stop 函数                                                         |
	| help     | 输出帮助信息                                                           |

* 当我们使用enable设置一个软件为开机自启动时，将自动在/etc/rc.d/目录下创建一个软链接指向/etc/init.d/目录下的软件
* 当我们使用disable关闭一个软件开机自启动时，软件在/etc/rc.d/目录下的软链接将删除

4. 自定义软件启动脚本设计

## 13. 驱动开发示例
* USB 驱动 [source code](https://github.com/gm-jiang/openwrt-driver)

---
# **USB 知识梳理**
---

## 1. USB 协议标准

| USB 协议标准                   | 主要特点                                                     | 速度等级         |
| ------------------------------ | ------------------------------------------------------------ | ---------------- |
| USB 2.0 Full Speed （USB 1.1） | 规范了 USB 低全速传输                                        | 1.5 Mbps~12 Mbps |
| USB 2.0 High Speed （USB 2.0） | 规范了 USB 高速传输                                          | 480 Mbps         |
| USB 3.2 gen1 （USB 3.0）       | 采用 8b/10b 编码，增加一对超高速差分线，供电 5V/0.9A         | 5 Gbps           |
| USB 3.2 gen2 （USB 3.1）       | 采用 128b/132b 编码，速度提高 1 倍，供电 20V/5A，同时增加了 A/V 影音传输标准 | 10 Gbps          |
| USB 3.2 gen2*2 （USB 3.2）     | 增加一对超高速传输通道，速度再次翻倍，只能在 C 型接口上运行  | 20 Gbps          |


## 2. USB 协议基本概念
* 一个传输（Transfer） (控制、批量、中断、等时)：由多个事务(Transaction)组成
* 一个事务 (IN、OUT、SETUP)：由多个 Packet 组成

## 3. 包 (Packet)
* 包是 USB 信息传输过程中基本单元，所有数据都是经过打包后在总线上传输，包只能在帧内传输，高速USB 总线的帧周期为125us，全速以及低速 USB 总线的帧周期为 1ms。帧的起始由一个特定的包（SOF 包）表示，帧尾为 EOF。EOF不是一个包，而是一种电平状态，EOF期间不允许有数据传输
* 包是USB总线上数据传输的最小单位，不能被打断或干扰，否则会引发错误。若干个数据包组成一次事务传输，一次事务传输也不能打断，属于一次事务传输的几个包必须连续，不能跨帧完成。一次传输由一次到多次事务传输构成，可以跨帧完成
* USB包由五部分组成，即同步字段（SYNC）、包标识符字段（PID）、数据字段、循环冗余校验字段（CRC）和包结尾字段（EOP），包的基本格式如下图：

```
+-------+-----+------+--------------+-----------------------+-----+
| SYNC  | PID | ADDR | Frame Number | DATA                  | CRC |
+-------+-----+------+--------------+-----------------------+-----+
```


* SYNC: 8 bit for full/low-speed. 32bit for high-speed
* PID : 8 bit 



PID Type   | PID Num  | PID <3-0> | Descriptor 
-----------|----------|-----------|------------------------------------------------
Token      | OUT      | 0001B     | ADDR + EP Num in host-to-function transaction
Token      | IN       | 1001B     | ADDR + EP Num in function-to-host transcation
Token      | SOF      | 0101B     | Start-of-Frame marker and frame number
Token      | SETUP    | 1101B     | ADDR + EP Num in host-to-function transaction for SETUP to a control pipe
Data       | DATA0    | 0011B     | Data packet PID even
Data       | DATA1    | 1011B     | Data packet PID odd
Data       | DATA2    | 0111B     | Data packet PID high-speed, high bandwidth isochronous transaction in micro frame
Data       | MDATA    | 1111B     | Data packet PID high-speed, for split and high bandwidth isochronous transaction
Handshake  | ACK      | 0010B     | Receiver accepts error-free data packet
Handshake  | NAK      | 1010B     | Receiving device cannot accept data or transmitting device cannot send data
Handshake  | STALL    | 1110B     | Endpoint is hatled or a control pipe request is not support
Handshake  | NYET     | 0110B     | No response yet from receiver


## 4. 事务 (Transaction)

* **输入(IN) 事务处理**：表示 USB 主机从总线上的某个 USB 设备接收一个数据包的过程
* •【正常】的输入事务处理
* 
    Direction       | Packet type  | packet
    ----------------|--------------|---------------------------------------
    Host -> Device  | Token        | SYNC + IN + ADDR + ENDP  + CRC5
    Device -> Host  | Data         | SYNC + DATA0 + DATA      + CRC16
    Host -> Device  | Handshake    | SYNC + ACK

* •【设备忙】时的输入事务处理  
* 
    Direction       | Packet type  | packet
    ----------------|--------------|---------------------------------------
    Host -> Device  | Token        | SYNC + IN + ADDR + ENDP  + CRC5
    Host -> Device  | Handshake    | SYNC + NAK

* •【设备出错】时的输入事务处理
* 
    Direction       | Packet type  | packet
    ----------------|--------------|---------------------------------------
    Host -> Device  | Token        | SYNC + IN + ADDR + ENDP  + CRC5
    Host -> Device  | Handshake    | SYNC + STALL


* **输出 (OUT) 事务处理**：表示 USB 主机把一个数据包输出到总线上的某个 USB 设备接收的过程

* •【正常】的输出事务处理
* 
    Direction       | Packet type  | packet
    ----------------|--------------|---------------------------------------
    Host -> Device  | Token        | SYNC + OUT + ADDR + ENDP  + CRC5
    Host -> Device  | Data         | SYNC + DATA0 + DATA      + CRC16
    Device -> Host  | Handshake    | SYNC + ACK

* •【设备忙】时的输出事务处理
* 
    Direction       | Packet type  | packet
    ----------------|--------------|---------------------------------------
    Host -> Device  | Token        | SYNC + OUT + ADDR + ENDP  + CRC5
    Host -> Device  | Data         | SYNC + DATA0 + DATA      + CRC16
    Device -> Host  | Handshake    | SYNC + NCK

* •【设备出错】时的输出事务处理
* 
    Direction       | Packet type  | packet
    ----------------|--------------|---------------------------------------
    Host -> Device  | Token        | SYNC + OUT + ADDR + ENDP  + CRC5
    Host -> Device  | Data         | SYNC + DATA0 + DATA      + CRC16
    Device -> Host  | Handshake    | SYNC + STALL

* **设置（SETUP）事务处理**
* •【正常】的设置事务处理
* 
    Direction       | Packet type  | packet
    ----------------|--------------|---------------------------------------
    Host -> Device  | Token        | SYNC + SETUP + ADDR + ENDP  + CRC5
    Host -> Device  | Data         | SYNC + DATA0 + 8Byte SETUP Data + CRC16
    Device -> Host  | Handshake    | SYNC + ACK

* •【设备忙】时的设置事务处理
* 
    Direction       | Packet type  | packet
    ----------------|--------------|---------------------------------------
    Host -> Device  | Token        | SYNC + SETUP + ADDR + ENDP  + CRC5
    Host -> Device  | Data         | SYNC + DATA0 + 8Byte SETUP Data + CRC16
    Device -> Host  | Handshake    | SYNC + NCK

* •【设备出错】时的设置事务处理
* 
    Direction       | Packet type  | packet
    ----------------|--------------|---------------------------------------
    Host -> Device  | Token        | SYNC + SETUP + ADDR + ENDP  + CRC5
    Host -> Device  | Data         | SYNC + DATA0 + 8Byte SETUP Data + CRC16
    Device -> Host  | Handshake    | SYNC + STALL

## 5. 传输
* 在USB的传输中，定义了4种传输类型：
* 控制传输 (Control Transfer)  (SETUP Stage + DATA Stage (optional) + STATUS Stage) 
* 中断传输 (Interrupt Transfer)
* 批量传输 (Bulk Transfer)
* 同步传输 (Isochronous)
* https://www.usbmadesimple.co.uk/ums_3.htm

* 
    |                | **Control**                     | **Bulk**                                         | **Interrupt**                                     | **Isoch**                                   |
    | -------------- | ------------------------------- | ------------------------------------------------ | ------------------------------------------------- | ------------------------------------------- |
    | 带宽           | HS: 20% <br>FS: 10% <br>LS: 10% | HS: 没有保证 <br/>FS: 没有保证 <br/>LS: 没有保证 | HS: 23.4 MB/s <br/>FS: 62.5 KB/s <br/>LS: 800 B/s | HS: N/A <br/>FS: 999 KB/s <br/>LS: 2.9 MB/s |
    | 最大数据包长度 | HS: 64 <br>FS: 64 <br>LS: 8     | HS: 512 <br/>FS: 64 <br/>LS: N/A                 | HS: 1024 <br/>FS: 64 <br/>LS: 8                   | HS: 1024 <br/>FS: 1023 <br/>LS: N/A         |
    | 传输错误管理   | 握手包、PID 翻转                | 握手包、PID 翻转                                 | 握手包、PID 翻转                                  | 无错误纠察                                  |
    | 组成           | Setup + Data (option) + Status  | 一个或多个 Data transaction (IN or OUT)          | 一个 Data transaction (IN)                        | 一个或多个 Data transaction (IN or OUT)     |

## 6. USB 设备描述
* 地址
* 设备
* 配置
* 接口
* 端点


## 7. USB Device 枚举
* 多配置USB设备枚举过程和多字符串描述符的枚举是相同的，过程如下：
1. 总线复位；
2. 获取设备描述符；
3. 总线复位；
4. 设置地址；
5. 获取设备描述符；
6. 获取配置描述符1；
7. 获取配置描述符2；
8. …
9. 获取字符串描述符1；
10. 获取字符串描述符2；
11. …
12. 设置配置；

## 8. SETUP Transaction Data

* The meaning of the 8 bytes of the SETUP transaction data, which are divided into five named fields.
* ![avatar](setup_transaction_data.jpg)

* Here is a table which contains all the standard requests which a host can send. The first 5 columns are the SETUP transaction fields in order, and the last column describes any accompanying data stage data which will have the length wLength.
* ![avatar](standard_request.jpg)

* More information [USB Made simple](https://www.usbmadesimple.co.uk/ums_4.htm)


---
# **安全启动**
---

## 1. 背景
1. `Security boot` 可以保证 `ROM code` 只运行签名被验证成功的 `Bootloader.bin`（在加载时进行验证）。 
2. 从 `ESP32-S2` 和 `ESP32-ECO3` 开始，新的基于 `RSA-PSS` (RSA-概率签名方案) 的安全启动验证方案（Secure Boot V2）被引入。
3. `ROM code` 验证 `Bootloader.bin`，并在验证成功后执行。经过验证的 `Bootloader.bin ` 会验证 `App.bin` 的RSA-PSS签名。

## 2. 背景知识
1. SHA
2. RSA
3. AES

## 3. Security Boot V2 特点
* Security Boot V2 优点
    1. `RSA` 公钥存储在设备上。 相应的 `RSA` 私钥被保密，并且永远不会被设备访问。在 `ESP32-S2` 上可最多可以存储 3 个公钥 (digest of public key)
    2. 应用程序和 `bootloader` 具有相同的签名验证流程
    3. 设备上不会存储任何机密。 因此不受被动侧信道攻击（时序或功率分析等）的影响

* Security Boot V2 不足
    1. 
    2. 
    3. 

## 4. 硬件支持
* SHA  --> Hash calculation
* RSA  --> RSA-PSS verfiy
* EFUSE --> save digest of public key
* AES --> flash encrypt

## 5. 安全启动过程 v2
* Pre verification
    1. verify the hash value of append the bootloader tail.
    2. Becasue the signed bootloader.bin will padding the image to next sector boundary. So contuine update the hash value of sha256 to the all bootloader.bin with padd 0xff to sector boundary. 
    3. Compare the the calculate hash value with the hash value in the signature block.
    4. Calculate the signature block crc with the crc value in the signature block.

* Start do important thing next
* Verification part 1, is this a public key we trust?
    1. verify the pub key in signture block.
    2. calc the hash value of public key in signature block
    3. compare the this hash value with the hash value in efuse block. 

* Verification part 2, verify RSA-PSS signature of image_digest
    1. if verfiy failed, we can revoke the flag that save in efuse.
    2. if just one signature block was verfiy success, than return success.


## 6. RSA-PSS signature
RSA signature (first learn the RSA signature principle)
* Signature
    * Calculation the digest value of bootloader.bin (Hash)
    * Encrypt the digest with the private key --> signature.
    * Append the signature to the bootloader.bin

* Unsigature
    * Use the pub key decrypt the signature --> digest
    * Calculation the digest value of bootloader.bin
    * compare the digest

## 7. 签名块格式

1. Bootloader bin format

```c
struct flash_hdr {
    uint8_t magic;
    uint8_t blocks;
    uint8_t read_mode;  //flag of flash read mode in unpackage and usage in future 
    uint8_t pad;  // used for clock and flash mode now
    uint32_t entry_addr;
} __ATTRIB_PACK;


struct flash_ext_config_hdr {
    uint8_t wp_gpio_num;
    uint8_t drvs[3];
    uint16_t chip_id; // Should equal SOC_CHIP_ID if image is for this chip
    uint8_t reserves[9];
    uint8_t appended_digest;
} __ATTRIB_PACK;


struct block_hdr {
    uint32_t load_addr;
    uint32_t data_len;
} __ATTRIB_PACK;

```


```
0                   8                  16
++++++++++++++++++++++++++++++++++++++++   --------> 0x00000000
|boolader_header ......................|
|..................| load_addr | size  |
++++++++++++++++++++++++++++++++++++++++
|data..................................|
|......................................|
++++++++++++++++++++++++++++++++++++++++
|..................| load_addr | size  |   calc_hash
++++++++++++++++++++++++++++++++++++++++
|data..................................|
|......................................|
++++++++++++++++++++++++++++++++++++++++
| load_addr | size | data..............|
|......................................|
|............| pad(00).............|crc|
++++++++++++++++++++++++++++++++++++++++   --------->
|sha256 hash(32Byte) ..................|
++++++++++++++++++++++++++++++++++++++++   ---------> 0x00007970

```

2. Bootloader bin with signature block format

```c
struct ets_secure_boot_sig_block {
    uint8_t magic_byte;
    uint8_t version;
    uint8_t _reserved1;
    uint8_t _reserved2;
    uint8_t image_digest[32]; // bootloader image of 32KB not include secure boot sig block
    ets_rsa_pubkey_t key; // public eky
    uint8_t signature[384]; // how is this signature value generated ?????
    uint32_t block_crc;
    uint8_t _padding[16];
};
```

```
0                   8                  16
++++++++++++++++++++++++++++++++++++++++   --------> 0x00000000 (0KB)
|boolader_header ......................|
|..................| load_addr | size  |
++++++++++++++++++++++++++++++++++++++++
|data..................................|
|......................................|
++++++++++++++++++++++++++++++++++++++++
|..................| load_addr | size  |   calc_hash
++++++++++++++++++++++++++++++++++++++++
|data..................................|
|......................................|
++++++++++++++++++++++++++++++++++++++++
| load_addr | size | data..............|
|......................................|
|............| pad(00).............|crc|
++++++++++++++++++++++++++++++++++++++++   --------->
|sha256 hash(32Byte) ..................|
++++++++++++++++++++++++++++++++++++++++   ---------> 0x00007970
| pad (0xff)...........................|
| pad (0xff)...........................|
| pad (0xff)...........................|
| pad (0xff)...........................|
| pad (0xff)...........................|
| pad (0xff)...........................|
++++++++++++++++++++++++++++++++++++++++   ---------> 0x00008000 (32KB)
| secure_boot_sig_block                |
|                                      |
|                                      |
|                                      |
+++++++++++++++++++++++++++++++++++++++|   ---------> 0x00009000 (36KB)
```

3. signature block info

offset       | size              | discrption
-------------|-------------------|---------------------------------------------------------------------------------
Offset 0     | (1 byte):         | Magic byte (0xe7)
Offset 1     | (1 byte):         | Version number byte (currently 0x02), 0x01 is for Secure Boot V1.
Offset 2     | (2 bytes):        | Padding bytes, Reserved. Should be zero.
Offset 4     | (32 bytes):       | SHA-256 hash of only the image content, not including the signature block.
Offset 36    | (384 bytes):      | RSA Public Modulus used for signature verification. (value ‘n’ in RFC8017).
Offset 420   | (4 bytes):        | RSA Public Exponent used for signature verification (value ‘e’ in RFC8017).
Offset 424   | (384 bytes):      | Precalculated R, derived from ‘n’.
Offset 808   | (4 bytes):        | Precalculated M’, derived from ‘n’
Offset 812   | (384 bytes):      | RSA-PSS Signature result (section 8.1.1 of RFC8017) of image content, computed using following PSS parameters: SHA256 hash, MFG1 function, 0 length salt, default trailer field (0xBC).
Offset 1196: | (32bytes)         | CRC32 of the preceding 1095 bytes.
Offset 1200  | (16 bytes):       | Zero padding to length 1216 bytes.

4. this signature block can up to three.

```
m：明文
e，n：RSA参数（公钥）
d：RSA参数（私钥）
c：网络传输密文
加密方加密m：c = m^e mod n，传输c

解密方解密c：m = c^d mod n，还原m

c'：篡改密文
k：篡改码
由于c在网络上传输，如果网络上有人对其进行c' = c*k^e mod n，这样的替换

那么解密方将得到的结果是

(c*k^e)^d mod n

= （c^d mod n）* （k^ed mod n）

= m*k

即中间人有办法控制m。
```

* 如果一个 attacker 在 signature block 把公钥 换成自己的公钥，然后用自己的私钥进行签名，后面跟确实可以用这个公钥解到这个签名，但是，boot 过程中有一个验证公钥的过程，因此 。。。。。

## 8. Linux 文件切割工具
* split -a 0 -b 15904 bootloader.bin bootloader_split.bin (切除 32byte 的 hash)
* sha256sum bootloader_split.binaa  （计算 hash）
413a560d438014c686a3faa6da59d430127361ba3eb430612fdff78d0ce586c8


---
# **ZSH**
---

## zsh and oh-my-zsh with p10k theme

```
sudo apt-get install zsh 
```

```
install oh-my-zsh  refer: https://github.com/ohmyzsh/ohmyzsh (gitee for no VPN)
sh -c "$(curl -fsSL https://raw.githubusercontent.com/ohmyzsh/ohmyzsh/master/tools/install.sh)"
```

```
git clone --depth=1 https://github.com/romkatv/powerlevel10k.git ${ZSH_CUSTOM:-$HOME/.oh-my-zsh/custom}/themes/powerlevel10k
```

```
set ZSH_THEME="powerlevel10k/powerlevel10k" in ~/.zshrc
```

```
git clone https://github.com/zsh-users/zsh-autosuggestions ${ZSH_CUSTOM:-~/.oh-my-zsh/custom}/plugins/zsh-autosuggestions
设置~/.zshrc，把zsh-autosuggestions添加到 Oh My Zsh 要加载的插件列表中

# ubuntu 显示乱码问题
打开Terminal 点击preference ->选中描述文件，更改字体即可

# auto-suggestion issues with tmux
export TERM=xterm-256color
```

---
# **VIM**
---

## 1. 插件管理器 Vundle

```
git clone http://github.com/VundleVim/Vundle.vim.git ~/.vim/bundle/Vundle.vim

```

## 2. 按装 libs
```
sudo apt-get install exuberant-ctags ack

```
## 3. 配置文件 `.vimrc`

```
syntax enable
set ts=4
set expandtab
set nocompatible              " be iMproved, required
set tabstop=4
set softtabstop=4
set shiftwidth=4
set autoindent
filetype off                  " required
set cul

" highlight VertSplit ctermbg=100 ctermfg=100
" highlight VertSplit ctermbg=white ctermfg=black

" set the runtime path to include Vundle and initialize
set rtp+=~/.vim/bundle/Vundle.vim
call vundle#begin()
" alternatively, pass a path where Vundle should install plugins
"call vundle#begin('~/some/path/here')

" let Vundle manage Vundle, required
Plugin 'VundleVim/Vundle.vim'

" The following are examples of different formats supported.
" Keep Plugin commands between vundle#begin/end.
" plugin on GitHub repo
Plugin 'tpope/vim-fugitive'
" Install L9 and avoid a Naming conflict if you've already installed a
" different version somewhere else.
" Plugin 'ascenator/L9', {'name': 'newL9'}

Plugin 'scrooloose/nerdtree'

Plugin 'vim-scripts/taglist.vim'

Plugin 'kien/ctrlp.vim'

Plugin 'itchyny/lightline.vim'

Plugin 'mileszs/ack.vim'

" All of your Plugins must be added before the following line
call vundle#end()            " required
filetype plugin indent on    " required
" To ignore plugin indent changes, instead use:
"filetype plugin on
"
" Brief help
" :PluginList       - lists configured plugins
" :PluginInstall    - installs plugins; append `!` to update or just :PluginUpdate
" :PluginSearch foo - searches for foo; append `!` to refresh local cache
" :PluginClean      - confirms removal of unused plugins; append `!` to auto-approve removal
"
" see :h vundle for more details or wiki for FAQ
" Put your non-Plugin stuff after this line
"

" NERDTree config
map <F2> :NERDTreeToggle<CR>
let g:NERDTreeDirArrowExpandable = '+'
let g:NERDTreeDirArrowCollapsible = '-'
let g:NERDTreeWinPos = 'left'
let g:NERDTreeSize = 30
let g:NERDTreeShowLineNumbers = 1
" let g:NERDTreeHidden=0

let Tlist_Use_Right_Window = 1
let Tlist_WinWidth = 30

" Resize window
nnoremap <S-Up> :resize -1<CR>
nnoremap <S-Down> :resize +1<CR>
nnoremap <S-Left> :vertical resize -1<CR>
nnoremap <S-Right> :vertical resize +1<CR>

" Configure Ack
nnoremap <c-u> :Ack<space>

" Configure Git blame
nnoremap <c-g> :Git blame<CR>

set laststatus=2
let g:lightline = {
      \ 'colorscheme': 'powerline',
      \ 'active': {
      \   'left': [ [ 'mode', 'paste' ],
      \             [ 'gitbranch', 'readonly', 'filename', 'modified' ] ]
      \ },
      \ 'component_function': {
      \   'gitbranch': 'FugitiveHead'
      \ },
      \ }
```

## 4. YouCompleteMe For VIM v8.1
```
Plugin 'Valloric/YouCompleteMe'
# git clone https://github.com/Valloric/YouCompleteMe.git

# git checkout 3ededaed2f9923d50bf3860ba8dace0f7d2724cd

# git submodule update --init --recursive

# ./install.sh

//.vimrc config
let g:ycm_global_ycm_extra_conf = '~/.vim/bundle/YouCompleteMe/third_party/ycmd/cpp/ycm_extra_conf.py'
let g:ycm_python_binary_path = '/usr/bin/python3'
```

## 5. VIM 插件 `NERDTree`


| Key  |  Function                                          |      |
| ---- | -------------------------------------------------- | ---- |
| o    | 在已有窗口中打开文件、目录或书签，并跳到该窗口     |      |
| go   | 在已有窗口 中打开文件、目录或书签，但不跳到该窗口  |      |
| t    | 在新 Tab 中打开选中文件/书签，并跳到新 Tab         |      |
| T    | 在新 Tab 中打开选中文件/书签，但不跳到新 Tab       |      |
| i    | split 一个新窗口打开选中文件，并跳到该窗口         |      |
| gi   | split 一个新窗口打开选中文件，但不跳到该窗口       |      |
| s    | vsplit 一个新窗口打开选中文件，并跳到该窗口        |      |
| gs   | vsplit 一个新 窗口打开选中文件，但不跳到该窗口     |      |
| !    | 执行当前文件                                       |      |
| O    | 递归打开选中 结点下的所有目录                      |      |
| x    | 合拢选中结点的父目录                               |      |
| X    | 递归 合拢选中结点下的所有目录                      |      |
| D    | 删除当前书签                                       |      |
| P    | 跳到根结点                                         |      |
| p    | 跳到父结点                                         |      |
| K    | 跳到当前目录下同级的第一个结点                     |      |
| J    | 跳到当前目录下同级的最后一个结点                   |      |
| k    | 跳到当前目录下同级的前一个结点                     |      |
| j    | 跳到当前目录下同级的后一个结点                     |      |
| C    | 将选中目录或选中文件的父目录设为根结点             |      |
| u    | 将当前根结点的父目录设为根目录，并变成合拢原根结点 |      |
| U    | 将当前根结点的父目录设为根目录，但保持展开原根结点 |      |
| r    | 递归刷新选中目录                                   |      |
| R    | 递归刷新根结点                                     |      |
| m    | 显示文件系统菜单                                   |      |
| I    | 切换是否显示隐藏文件                               |      |
| F    | 切换是否显示文件                                   |      |
| f    | 切换是否使用文件过滤器                             |      |
| B    | 切换是否显示书签                                   |      |
| Crtl+W+R | 切换当前窗口左右布局                           |      |




---
# **RISC-V 基础**
---

## 1. 函数调用规范
1. 将参数存储到函数能够访问的位置
2. 跳转到函数开始的位置 (使用 RV32I 的 jal 指令)
3. 获取函数需要的局部存储资源，按需保存寄存器
4. 执行函数中的指令
5. 将返回值存储到调用者能够访问到的位置，恢复寄存器，释放局部存储资源
6. 返回调用函数的位置 (使用 ret 指令)

* 为了获得良好的性能，变量应该尽量保存在寄存器而不是内存中，但同时也要尽量避免频繁的保存和恢复寄存器，因为它们同样会访问内存
* Xtensa 通过寄存器窗口旋转的方式 (逻辑寄存器，物理寄存器)，避免寄存器的频繁保存和恢复
* RISC-V 有足够多的寄存器来达到两全齐美的结果：既能将操作数存在寄存器中，同时也能减少保存和恢复的次数。其中的关键在于，在函数调用的过程中不保留部分寄存器存储的值，称它们为临时寄存器；另一些寄存器则对应地称为保存寄存器。不再调用其它函数的函数称为叶函数。当一个叶函数只有少量的参数和局部变量时，它们可以都被存储在寄存器中，而不会“溢出（spilling）”到内存中。但如果函数参数和局部变量很多，程序还是需要把寄存器的值保存在内存中，不过这种情况并不多见。

## 2. 通用寄存器组
* 
    Register     |ABI name    | Discription           | Saver    | 在调用中是否保留 
   --------------|------------|-----------------------|----------|-------------------
    x0           | zero       | hardware zero         | -        | -
    x1           | ra         | return address        | Caller   | No
    x2           | sp         | stack ptr             | Callee   | Yes
    x3           | gp         | global ptr            | -        | -
    x4           | tp         | thread ptr            | -        | -
    x5           | t0         | temp/alternate link   | Caller   | No
    x6 ~ x7      | t0 ~ t2    | temp reg              | Caller   | No
    x8           | s0/fp      | save reg/frame ptr    | Callee   | Yes
    x9           | s1         | save reg              | Callee   | Yes
    x10 ~ x11    | a0 ~ a1    | func arg/return value | Caller   | No
    x12 ~ x17    | a2 ~ a7    | func arg              | Caller   | No
    x18 ~ x27    | s2 ~ s11   | save reg              | Callee   | Yes
    x28 ~ x32    | t3 ~ t6    | temp reg              | Caller   | No

* 函数调用时保留的寄存器
    * 被调用的函数一般不会使用这些寄存器，即便使用也会提前保存好原值，可以信任。这些寄存器有： sp, gp, tp 和 s0 ~ s11 寄存器
* 函数调用时不保存的寄存器
    * 有可能被调用函数使用更改，需要 Caller 在调用前对自己使用到的寄存器进行保存。这些寄存器有： 参数与返回地址寄存器 a0 ~ a7, 返回地址寄存器 ra, 临时寄存器 t0 ~ t6

---
# **ESP32-S2 无线网卡**
---

## 1. 硬件

* Beaglebone black
* ESP32-S2
* ![image](https://user-images.githubusercontent.com/15644391/178235511-815ddc81-30f3-44eb-ac97-3a2799fda902.png)


## 2. 系统

* Beaglebone Black

  * 系统版本：`bone-debian-10.3-iot-armhf-2020-04-06-4gb.img`
  * 系统安装: 烧写系统到 SD卡 (推荐4GB 以上) `sudo dd bs=4M if=xxx.img of=/dev/sdd`
  * SD 卡系统启动： 按住 S2 按键， 接入电源，等待 4 个 LED 灯全亮，松开 S2 按键
  * 内核版本: `Linux beaglebone 4.19.94-ti-r42`
  * GCC 版本：`gcc (Debian 8.3.0-6) 8.3.0`
  * 软件包支持: `iperf`
  * 驱动编译支持：
  * Wi-Fi 配置管理工具：wpa_supplicant
  * 系统登录: UAR0 / ssh
  * UART0 硬件连接
    * ![image](https://user-images.githubusercontent.com/15644391/178180945-79077622-6f2b-48f8-a737-1d6eff7bf131.png)

* ESP32-S2 Saola-1_v1.2

  * esp-idf branch： esp32_usb_80211_wifi 
  * commit：e9ee1bb876dec6fdda9e646b68ccf92025c861a4
  * toolchain： xtensa-esp32s2-elf-gcc  gcc version 8.4.0 (crosstool-NG esp-2021r1)

## 3. ESP32-S2 固件编译

* esp-idf branch： esp32_usb_80211_wifi 
* idf.py build
* idf.py flash

## 4. linux 驱动编译

* 编译环境：
  * 接入网络：配置 BBB 连接网络
  * 更新源： `sudo apt-get update`
  * 目录缺失：缺少 `/lib/module/build` 问题，需要更新一下软件列表，然后 `sudo apt-get install linux-headers-$(uname -r) `
  * 内核版本: `Linux beaglebone 4.19.94-ti-r42`
  * GCC 版本：`gcc (Debian 8.3.0-6) 8.3.0`

* 下载驱动源码：source code: https://github.com/gm-jiang/beagleboneblack_src
* 编译： `make`
  * ![image](https://user-images.githubusercontent.com/15644391/178183889-90450dea-049f-4c9f-9117-77470e5a366b.png)
* 加载驱动模块： `modprobe cfg80211`
* 加载驱动模块： `insmod xxx.ko`
  * ![image](https://user-images.githubusercontent.com/15644391/178231259-97dc7ca0-31f6-409d-ac76-ed8c7ffa752c.png)

* 安装 `wpa_supplicant`
  * wpa_supplicant的安装方法有两种：第一种是在 ubuntu 中使用命令：`sudo apt-get install wpasupplicant` 该命令可将wpa_supplicant、wpa_cli、wpa_passphrase直接安装
  * 第二种是直接在官网下载源代码，编译、安装 [refer to link](https://blog.csdn.net/u012503786/article/details/79541811)
  
## 5. 测试

* 修改 `/etc/wpasupplicant/wpa_supplicant.conf` 文件，添加如下内容：

```
network={
　　ssid="[网络ssid]"
　　psk="[密码]"
　　priority=1
}
```

* 连接

```
 sudo wpa_supplicant -i xwifi0 -c /etc/wpa_supplicant/wpa_supplicant.conf & (后台运行)
 log：debian@beaglebone:~$ Successfully initialized wpa_supplicant
	xwifi0: Trying to associate with 18:31:bf:4b:8b:68 (SSID='ESP-Audio' freq=2437 MHz)
	Failed to add supported operating classes IE
	xwifi0: Associated with 18:31:bf:4b:8b:68
	xwifi0: CTRL-EVENT-CONNECTED - Connection to 18:31:bf:4b:8b:68 completed [id=0 id_str=]
	xwifi0: CTRL-EVENT-SUBNET-STATUS-UPDATE status=0
```

* DHCP Client 

```
 sudo dhclient & (后台运行)
```

* PING Test
  * ![image](https://user-images.githubusercontent.com/15644391/178230668-743f9456-b2e9-4fd9-9f70-6c1d70ca53a3.png)

* PING 指定网卡
  * ping -S 192.168.50.97 www.baidu.com

* Iperf Test
  * ![image](https://user-images.githubusercontent.com/15644391/178234798-b136f644-7c5d-4ee9-8e0e-79cfa5a713cd.png)


## 6. 遗留问题

* Run on OpenWRT that linux kernel version >= 5.4.0 (done)
* Run on Ubuntu18.04 that linux kernel version == 5.4.0-122-generic (done)

## Device 端软件架构

* ![USB_20220810111722](https://user-images.githubusercontent.com/15644391/183869446-4ad25e24-5625-4870-9df4-ea4e809ccf84.jpg)
* ![USB_WIFI](https://user-images.githubusercontent.com/15644391/184282611-f63dc9a1-85ac-4d87-b6b2-805290de7a5a.jpg)
* ![USB_20220810111707](https://user-images.githubusercontent.com/15644391/183870237-b9a0a34b-cced-4727-86ee-e19f53dec871.jpg)


---
# **RTMP FFMPEG**
---

## 1. RTMP 服务器安装
  * apt install nginx
  * apt install libnginx-mod-rtmp
  * nginx.conf
	  ```
	  rtmp {
	     server {
		    listen 1935;
		    application myapp {
			live on;
		    }
	      }
	  }
	  ```
  * nginx restart

## 2. FFMPEG 推流
  * `ffmpeg  -y -t 50000 -f video4linux2 -re -i /dev/video0  -f flv -flvflags no_duration_filesize -g 5 -b 700000 rtmp://106.14.10.223:1935/myapp/test`
  
## 3. VLC 拉流
  * rtmp://IP:port/myapp/test 点击播放
  
## 4. Aliyun Login: ssh gmjiang@106.14.10.223
