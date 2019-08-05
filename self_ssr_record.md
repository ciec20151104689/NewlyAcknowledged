# 使用一键脚本安装ssr服务端，脚本：

## yum -y install wget

wget -N --no-check-certificate https://raw.githubusercontent.com/ToyoDAdoubi/doubi/master/ssr.sh && chmod +x ssr.sh && bash ssr.sh

## 之后使用一键开启 bbr脚本：

sudo wget --no-check-certificate https://github.com/teddysun/across/raw/master/bbr.sh && chmod +x bbr.sh && ./bbr.sh

## 因为我们在安装BBR成功后替换了内核，需重新配置端口进行通讯 ，检查防火墙是否允许你设定的端口进行通信

iptables -L -n | grep 你的shadowsocks端口

## 如果没有信息的话，就是防火墙不允许该端口进行通信。需设置：

iptables -I INPUT -p tcp --dport 你的shadowsocks端口 -j ACCEPT
