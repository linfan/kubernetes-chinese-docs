
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
6. [开启libvirt接入的用户权限](https://libvirt.org/aclpolkit.html)
7. 确保qemu用户可以使用$HOME  
    在分布式环境中，libvirt的接入默认被拒绝或者每次接入需要密码。  
    可以使用如下命令进行测试：  
```virsh -c qemu:///system pool-list```  
    如果接入报错，请阅读https://libvirt.org/acl.html 和https://libvirt.org/aclpolkit.html。  
    简而言之，如果libvirt是在包含Polkit的环境下编译（例如：Arch, Fedora 21），可以创建内容如下/etc/polkit-1/rules.d/50-org.libvirt.unix.manage.rules文件来给libvirt接入所有用户的权限  
   sudo /bin/sh -c "cat - > /etc/polkit-1/rules.d/50-org.libvirt.unix.manage.rules" << EOF
polkit.addRule(function(action, subject) {
        if (action.id == "org.libvirt.unix.manage" &&
            subject.user == "$USER") {
                return polkit.Result.YES;
                polkit.log("action=" + action);
                polkit.log("subject=" + subject);
        }
});
EOF  
  ($USER为你的登陆用户名)  
  如果libvirt在不包含Polkit的环境下编译（例如：14.04.1 LTS），检查libvirt unix socket的使用权限：
```$ ls -l /var/run/libvirt/libvirt-sock  
   srwxrwx--- 1 root libvirtd 0 févr. 12 16:03 /var/run/libvirt/libvirt-sock  
   $ usermod -a -G libvirtd $USER              
   $USER needs to logout/login to have the new group be taken into account```  
  Qemu是以一个具体的用户运行，他必须能接入到VMs驱动中。  
  所有的虚拟机都需要磁盘驱动器（CoreOS disk image, Kubernetes binaries, cloud-init files, 等等.），这些驱动都放入到./cluster/libvirt-coreos/libvirt_storage_pool中。
如果你的$HOME是可以读的，那么一起都好。如果你的$HOME是私有的，那么脚本cluster/kube-up.sh将会报如下错误：
```error: Cannot access storage file '$HOME/.../kubernetes/cluster/libvirt-coreos/libvirt_storage_pool/kubernetes_master.img' (as uid:99, gid:78): Permission denied```  
  可以通过如下方法修复这个问题：
  * 在cluster/libvirt-coreos/config-default.sh中设置POOL_PATH到一个目录：
    * 在文件系统找一个有大量空闲的磁盘空间
    * 这个目录你的用户具有可写权限
    * qemu用户可以接入
  * 允许你的qemu用户可以接入到所有的存储池
  设置访问权限：   
    ```setfacl -m g:kvm:--x ~```

## 安装
在默认的情况下，libvirt-coreos将创建一个 Kubernetes master和3个Kubernetes nodes。因为VM的驱动用了COW和由于内存释放和KSM，有许多的资源被过度的分配。
开始运行你的本地cluster，打开一个shell并且运行：  
 ```cd kubernetes
    export KUBERNETES_PROVIDER=libvirt-coreos
    cluster/kube-up.sh```


