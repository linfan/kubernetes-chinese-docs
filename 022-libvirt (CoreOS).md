
# libvirt CoreOS的入门指南


## 亮点

1. 超高速启动集群（几秒钟，而不是几分钟的游离状态）
2. 使用COW来节省磁盘使用空间
3. 使用KSM来节省内存占用空间


## 环境准备

1. 安装dnsmasq
2. 安装ebtables
3. 安装qemu
4. 安装libvirt
5. 启用和启动libvirt守护进程，例如：
    * systemctl enable libvirtd
    * systemctl start libvirtd
6. 开启libvirt接入的用户权限
7. 确保qemu用户可以使用$HOME
    在分布式环境中，libvirt的接入默认被拒绝或者每次接入需要密码


8. 


