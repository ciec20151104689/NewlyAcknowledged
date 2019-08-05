# ssr

## Step 1: Upgrade the kernel using the ELRepo RPM repository

In order to use BBR, you need to upgrade the kernel of your CentOS 7 machine to 4.9.0. You can easily get that done using the ELRepo RPM repository.

Before the upgrade, you can take a look at the current kernel:

uname -r
This command should output a string which resembles:

3.10.0-514.2.2.el7.x86_64
As you see, the current kernel is 3.10.0.

### Install the ELRepo repo:

sudo rpm --import https://www.elrepo.org/RPM-GPG-KEY-elrepo.org
sudo rpm -Uvh http://www.elrepo.org/elrepo-release-7.0-2.el7.elrepo.noarch.rpm

### Install the 4.9.0 kernel using the ELRepo repo:

sudo yum --enablerepo=elrepo-kernel install kernel-ml -y

### Confirm the result:

rpm -qa | grep kernel
If the installation is successful, you should see kernel-ml-4.9.0-1.el7.elrepo.x86_64 among the output list:

kernel-ml-4.9.0-1.el7.elrepo.x86_64
kernel-3.10.0-514.el7.x86_64
kernel-tools-libs-3.10.0-514.2.2.el7.x86_64
kernel-tools-3.10.0-514.2.2.el7.x86_64
kernel-3.10.0-514.2.2.el7.x86_64
Now, you need to enable the 4.9.0 kernel by setting up the default grub2 boot entry.

### Show all entries in the grub2 menu:

sudo egrep ^menuentry /etc/grub2.cfg | cut -f 2 -d \'
The result should resemble:

CentOS Linux 7 Rescue a0cbf86a6ef1416a8812657bb4f2b860 (4.9.0-1.el7.elrepo.x86_64)
CentOS Linux (4.9.0-1.el7.elrepo.x86_64) 7 (Core)
CentOS Linux (3.10.0-514.2.2.el7.x86_64) 7 (Core)
CentOS Linux (3.10.0-514.el7.x86_64) 7 (Core)
CentOS Linux (0-rescue-bf94f46c6bd04792a6a42c91bae645f7) 7 (Core)
Indexing starts at 0. This means that the 4.9.0 kernel is located at 1:

sudo grub2-set-default `
Reboot the system:

sudo shutdown -r now
When the server is back online, log back in and rerun the uname command to confirm that you are using the correct Kernel:

uname -r
You should see the result as below:

4.9.0-1.el7.elrepo.x86_64
## Step 2: Enable BBR
In order to enable the BBR algorithm, you need to modify the sysctl configuration as follows:

echo 'net.core.default_qdisc=fq' | sudo tee -a /etc/sysctl.conf
echo 'net.ipv4.tcp_congestion_control=bbr' | sudo tee -a /etc/sysctl.conf
sudo sysctl -p
Now, you can use the following commands to confirm that BBR is enabled:

sudo sysctl net.ipv4.tcp_available_congestion_control
The output should resemble:

net.ipv4.tcp_available_congestion_control = bbr cubic reno
Next, verify with:

sudo sysctl -n net.ipv4.tcp_congestion_control
The output should be:

bbr
Finally, check that the kernel module was loaded:

lsmod | grep bbr
The output will be similar to:

tcp_bbr                16384  0
## Step 3 (optional): Test the network performance enhancement
In order to test BBR's network performance enhancement, you can create a file in the web server directory for download, and then test the download speed from a web browser on your desktop machine.

sudo yum install httpd -y
sudo systemctl start httpd.service
sudo firewall-cmd --zone=public --permanent --add-service=http
sudo firewall-cmd --reload
cd /var/www/html
sudo dd if=/dev/zero of=500mb.zip bs=1024k count=500
Finally, visit the URL http://[your-server-IP]/500mb.zip from a web browser on your desktop computer, and then evaluate download speed.

# CentOS 上开启 BBR 加速
 
BBR 算法需要 Linux 4.9 及以上的内核支持，所以想要使用该方式的需要先升级内核版本。

在 Cent OS 7 上的 Linux 内核是 3.10, 使用 uname -r 查看内核版本

[root@iZ2ze83hhomw2zcf15c3qcZ ~]# uname -r

3.10.0-327.22.2.el7.x86_64
 

## 升级内核版本
### 安装 eprl 的源

sudo rpm --import https://www.elrepo.org/RPM-GPG-KEY-elrepo.org
sudo rpm -Uvh http://www.elrepo.org/elrepo-release-7.0-2.el7.elrepo.noarch.rpm
 

### 安装最新的内核版本，截止目前最新的都到 4.14

sudo yum --enablerepo=elrepo-kernel install kernel-ml -y
 

### 看一下系统现在所有的内核 
rpm -qa | grep kernel


[root@iZ2ze83hhomw2zcf15c3qcZ ~]# rpm -qa | grep kernel
kernel-3.10.0-327.el7.x86_64
kernel-3.10.0-327.22.2.el7.x86_64
kernel-tools-libs-3.10.0-327.el7.x86_64
kernel-tools-3.10.0-327.el7.x86_64
kernel-ml-4.14.3-1.el7.elrepo.x86_64
kernel-headers-3.10.0-514.2.2.el7.x86_64

 

#### 可以看到最新的内核版本 kernel-ml-4.14.3-1.el7.elrepo.x86_64 已经安装好了。

### 现在来修改 grub2 的启动项，设置启动之后选择最新的内核 sudo egrep ^menuentry /etc/grub2.cfg | cut -f 2 -d \' 

[root@iZ2ze83hhomw2zcf15c3qcZ ~]# sudo egrep ^menuentry /etc/grub2.cfg | cut -f 2 -d \'
CentOS Linux (4.14.3-1.el7.elrepo.x86_64) 7 (Core)
CentOS Linux (3.10.0-327.22.2.el7.x86_64) 7 (Core)
CentOS Linux (3.10.0-327.el7.x86_64) 7 (Core)
CentOS Linux (0-rescue-7d26c16f128042a684ea474c9e2c240f) 7 (Core)
 

### 启动顺序已经修改了，但是为了以防万一，我们还是设置一下 sudo grub2-set-default 0，选择第一个为默认启动项。

## 最后就可以重启机器

sudo reboot
### 再次登录机器查看内核版本 uname -r ，已经是最新版本

[root@iZ2ze83hhomw2zcf15c3qcZ ~]# uname -r
4.14.3-1.el7.elrepo.x86_64
 

## 开启 BBR
 
### 为了启用BBR算法，您需要修改sysctl配置，如下所示：

echo 'net.core.default_qdisc=fq' | sudo tee -a /etc/sysctl.conf
echo 'net.ipv4.tcp_congestion_control=bbr' | sudo tee -a /etc/sysctl.conf
sudo sysctl -p

### 现在，您可以使用以下命令确认已启用BBR：

sudo sysctl net.ipv4.tcp_available_congestion_control

### 输出应该类似：

net.ipv4.tcp_available_congestion_control = bbr cubic reno
### 然后：
sudo sysctl -n net.ipv4.tcp_congestion_control
### 输出应该类似：

bbr

最后，检查内核模块是否已加载：

lsmod | grep bbr

### 输出应该类似：

tcp_bbr                16384  0
## 开启对应端口的 网络通讯
## 或 直接使用一步安装脚本

sudo wget --no-check-certificate https://github.com/teddysun/across/raw/master/bbr.sh && chmod +x bbr.sh && ./bbr.sh
