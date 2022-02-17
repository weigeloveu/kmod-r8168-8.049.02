# kmod-r8168-8.049.02
升级内核3.10到5.6后，无法识别网卡。表现:ip addr后无显示相关网卡
原因： 内核默认使用centos的网卡驱动r8169,但实际网卡是r8168，驱动不匹配.
解决方案：

临时加载r8169驱动
命令如下： 1. #rmmod r8169 2. #modprobe r8169 3. #systemctl restart network
重启后会失效
到官网下载匹配的最新的网卡驱动： https://www.realtek.com/zh-tw/component/zoo/category/network-interface-controllers-10-100-1000m-gigabit-ethernet-pci-express-software
下载后解压，直接运行autorun.sh
如果安装爆错/lib/modules/xxx/kernel/build No such file or directory, 安装kernel-devel包和kernel-headers包
命令：

 # yum --disablerepo=’*’ --enablerepo=elrepo-kernel install kernel-ml-devel
 # yum remove kernel-headers 
 # yum --disablerepo=’*’ --enablerepo=elrepo-kernel install kernel-ml-headers
 
安装相关kernel包后再次运行autorun.sh,可能会爆错gcc : Command not found。 这时候安装gcc
命令： 
Install CentOS SCLo RH repository:
# yum install centos-release-scl-rh
Install devtoolset-9-gcc rpm package:
# yum install devtoolset-9-gcc
source /opt/rh/devtoolset-9/enable
再次运行auutorun.sh,成功
重启network 服务
命令： systemctl restart network
————————————————
参考链接：https://blog.csdn.net/hnufun/article/details/106409560
1、查看系统版本
cat /etc/redhat-release
2、查看当前内核版本
uname -r
3、检查是否安装ELRepo
yum  --disablerepo="*"  --enablerepo="elrepo-kernel"  list  available

    已加载插件：fastestmirror


    Error getting repository data for elrepo-kernel, repository not found

# 看到error说明没有安装ELRepo
4、升级安装ELRepo：http://elrepo.org/tiki/HomePage
#更新yum源仓库
yum -y update
#载入ELRepo仓库的公共密钥
rpm --import https://www.elrepo.org/RPM-GPG-KEY-elrepo.org
或者：

#安装ELRepo仓库的yum源
yum install -y https://www.elrepo.org/elrepo-release-7.el7.elrepo.noarch.rpm

# 或升级
rpm -Uvh https://www.elrepo.org/elrepo-release-7.el7.elrepo.noarch.rpm
5、再次查看可用安装包
#查看可用的系统内核包
yum  --disablerepo="*"  --enablerepo="elrepo-kernel"  list  available

# 长期维护版本为lt，最新主线稳定版为ml
6、安装最新的内核
# 我这里选择的是长期维护版本kernel-lt  如需更新最新稳定版选择kernel-ml
yum  --enablerepo=elrepo-kernel  install  -y  kernel-lt
......
正在安装    : kernel-lt-5.4.108-1.el7.elrepo.x86_64
......
已安装:
  kernel-lt.x86_64 0:5.4.108-1.el7.elrepo
7、查看可用内核版本及启动顺序
awk -F\' '$1=="menuentry " {print i++ " : " $2}' /boot/grub2/grub.cfg

0 : CentOS Linux (5.4.108-1.el7.elrepo.x86_64) 7 (Core)
1 : CentOS Linux (3.10.0-1160.11.1.el7.x86_64) 7 (Core)
2 : CentOS Linux (3.10.0-1160.el7.x86_64) 7 (Core)
3 : CentOS Linux (0-rescue-20210128140208453518997635111697) 7 (Core)
8、安装辅助工具（非必须，有些系统自带该工具）：grub2-pc
yum install -y grub2-pc
9、设置内核默认启动顺序
grub2-set-default 0
或者

grub-set-default 'CentOS Linux (5.4.96-1.el7.elrepo.x86_64) 7 (Core)'
# CentOS Linux (5.4.96-1.el7.elrepo.x86_64) 7 (Core)  这个是具体的版本
10、vim /etc/default/grub
#编辑/etc/default/grub文件
vim /etc/default/grub 
设置   GRUB_DEFAULT=0   # 只需要修改这里即可

GRUB_TIMEOUT=1
GRUB_DISTRIBUTOR="$(sed 's, release .*$,,g' /etc/system-release)"
GRUB_DEFAULT=saved  #---将这里的saved修改成0----
GRUB_DISABLE_SUBMENU=true
GRUB_TERMINAL_OUTPUT="console"
GRUB_CMDLINE_LINUX="crashkernel=auto spectre_v2=retpoline rhgb quiet net.ifnames=0 console=tty0 console=ttyS0,115200n8 noibrs"
GRUB_DISABLE_RECOVERY="true"
11、生成grub 配置文件
# 运行grub2-mkconfig命令来重新创建内核配置
grub2-mkconfig -o /boot/grub2/grub.cfg
12、重启系统
reboot
# 或者
shutdown -r now

#重启完成后，查看内核版本是否正确
uname -r
13、查看系统中已安装的内核
rpm -qa | grep kernel

kernel-devel-3.10.0-1160.11.1.el7.x86_64
kernel-tools-3.10.0-1160.11.1.el7.x86_64
kernel-3.10.0-1160.el7.x86_64
kernel-3.10.0-1160.11.1.el7.x86_64
kernel-lt-5.4.108-1.el7.elrepo.x86_64
kernel-tools-libs-3.10.0-1160.11.1.el7.x86_64
kernel-headers-3.10.0-1160.11.1.el7.x86_64
14、删除旧内核，这一步是【可选】的
yum remove -y  kernel-devel-3.10.0   kernel-3.10.0  kernel-headers-3.10.0 

# 查看已安装内核
rpm -qa | grep kernel

# 也可以安装 yum-utils 工具，当系统安装的内核大于3个时，会自动删除旧的内核版本
yum install -y  yum-utils
13 、升级内核工具包
# 删除旧版本工具包--可选
yum remove kernel-tools-libs.x86_64 kernel-tools.x86_64
# 安装新版本工具包
yum --disablerepo=\* --enablerepo=elrepo-kernel install -y kernel-lt-tools.x86_64

# 查看已安装内核
rpm -qa | grep kernel
