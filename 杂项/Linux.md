# 常用命令

`--help` 查询各个命令如何使用
`man`  
`vimtutor` vim 的使用教程

远程链接 `ssh user@host`
上传`ssh`公钥 `ssh-copy-id user@host`
使用`scp`传输文件(双向) `scp /path/to/localFile user@host:/path`

修改密码 `passwd username`

查看当前目录 `pwd`
查看`Linux`版本 `cat /etc/issue`
查看系统内核、系统版本等 `uname -[amnrsv]`

查看当前系统所有用户 `who`
查看最近用户的登陆信息 `last`
查看历史指令 `history [-count][-c]`

查看本地`ip`
- `ip addr show`
- `ifconfig`
> `ifconfig` 用于获取网卡配置与网络状态等信息，第一部分显示状态信息、第二部分显示网络信息。RX表示接收数据包的情况，TX表示发送数据包的情况。lo表示主机的回环网卡

查看内存使用情况 `free [-k|b|m][-h]`

查询指定进程`PID` `pidof service_name [-scox]`
>  `s`只返回进程号 `c`只显示运行在`root`目录下的进程 `o`忽略指定进程号的进程 `x`显示由脚本开启的进程

查看端口占用情况 `netstat -tunlp | grep port`

终止指定`PID`进程 `kill [-s sigspec | -n signum | -sigspec] pid`
终止服务的全部进程 `killall service_name`
## echo
`echo`命令用于在终端输出字符串或变量提取后的值 格式 `echo [字符串 | $变量]

> 在终端显示命令结果使用`${}`包裹 如: `echo pwd`或`echo ${pwd}`

`>` 表示输出重定向，输出到文件如果没有指定文件则创建，有则覆盖

> `echo [字符串 | $变量] > file` 如: `echo "你好" > holle.txt`  
> `>>` 表示输出追加重定向，不会覆盖而是添加在文件末端

## wget
`wget` 在终端下载文件 格式 `wget [参数] 下载地址`  
参数:

1. `-b` 后台下载
2. `-P` 指定下载目录
3. `-t` 最大重试次数
4. `-c` 断点续传
5. `-p`下载界面所有资源(包括图片，视频)
6. `-r` 递归下载

## ps
`ps` 用于查看系统中的进程状态 常用 `ps -aux | grep keyWord`  
参数：

1. `-a` 显示现行终端机下的所有程序，包括其他用户的程序
2. `-u` 以用户为主的格式来显示程序状况
3. `-x` 显示没有控制终端的进程，同时显示各个命令的具体路径
4. `-e` 列出程序时，显示每个程序所使用的环境变量
5. `-f` 显示当前所有的进程
6. `-t` 指定终端机编号，并列出属于该终端机的程序的状况

## top
`top` 命令动态地监视进程活动与系统负载等信息 `q`键退出  
参数解释:

|列名|含义|
|---|---|
|PID|进程ID|
|USER|进程所有者的用户名|
|PR|进程优先级|
|NI|nice值。负值表示高优先级，正值表示低优先级|
|VIRT|进程使用的虚拟内存总量，单位kb|
|RES|进程使用的、未被换出的物理内存大小，单位kb|
|SHR|共享内存大小，单位kb|
|S|进程状态D：不可中断的睡眠状态R：正在运行S：睡眠T：停止Z：僵尸进程|
|%CPU|上次更新到现在的CPU时间占用百分比|
|%MEM|进程使用的物理内存百分比|
|TIME+|进程使用的CPU时间总计，单位1/100秒|
|COMMAND|命令名|
## ufw
`sudo ufw enable` 打开防火墙  
`sudo ufw disable` 关闭防火墙  
`sudo ufw status verbose` 查看防火墙规则  
`sudo ufw default deny incoming` 防火墙为白名单模式

>`incoming`、`outgoing`前者为入站流量，后者为出站流量

端口操作:  
`sudo ufw allow 8080` 开启8080端口，同时放行`tcp`和`udp`  
`sudo ufw allow 8080/tcp` 只放行`tcp`，`udp`同理  
`sudo ufw deny 8080` 关闭8080端口，同上

`sudo ufw allow from ip-Address to any port 8080` 对指定`ip`开放端口

`sudo ufw status` 查看放行端口  
`sudo ufw status numbered` 排序 > `sudo ufw delete 1`

> `Status: inactive` 表示`ufw`没开启

## systemctl
systemctl 运行的服务都在 /usr/lib/systemd/system 目录下的 xxx.service
```shell
systemctl enable --now xxx.service   # 启动服务，并设置开机自启 
systemctl disable xxx.service  # 关闭容器开机自启
systemctl start xxx.service    # 启动服务 
systemctl stop xxx.service     # 关闭服务 
systemctl restart xxx.service  # 重启服务 
systemctl status xxx.service   # 查看服务运行状态
```

## 后台运行
后台运行`bash`脚本 `nohup ./xxx.sh &`

> 自定义重定向输入 `nohup ./xxx.sh > /path/to/custom.out &`  
> 文件后缀可以是 `.out`、`.log`、`.txt`...  
> `&` 将脚本后台运行，不占用终端(终端关闭仍然关闭)  
> `nohup` 终端关闭，脚本不关闭

> 要追踪查看日志:  
> `tail -f xxx.xx` 实时跟踪文件的内容  
> `cat xxx.xxx` 查看文件全部内容

如果`nohup`不能保证运行，使用`screen`

1. `screen` 开启一个伪终端
2. 在`screen`操作
3. `Ctrl+A`后`d`退出`screen`即可

> 如果需要重新进入`screen`，`screen -r` 进入上次进入的`screen`  
> `screen -ls` 即可查看开启的`screen`  
> 可以使用 `kill pid` 关闭 `screen`

## BBR
`BBR`是瓶颈带宽和往返传播时间的缩写，是谷歌开发的一种现代拥塞控制算法。它通过有效地调整网络管道的大小来优先考虑更快的数据传输，以提高整体性能。
```bash
sysctl net.ipv4.tcp_congestion_control    # 查看当前使用算法
sudo modprobe tcp_bbr                     # 查看bbr的兼容性
# 配置bbr
sudo sh -c 'echo "net.core.default_qdisc=fq" >> /etc/sysctl.conf'
sudo sh -c 'echo "net.ipv4.tcp_congestion_control=bbr" >> /etc/sysctl.conf'
sudo sysctl -p  # 重启sysctl已启用配置
```
# 命令别名
在`~/.bashrc`文件可以添加一些复杂命令的别名
```bash
alias ll='ls -alF'   # 输出当前文件夹的详细信息
```
添加后执行`source ~/.bashrc`重新加载终端的配置文件
# 用户权限
用户管理
```bash
sudo useradd -m username   # 创建用户 -m 同时创建HOME目录
sudo userdel -r username   # 删除用户
sudo passwd username       # 修改用户密码

sudo groupadd groupname   # 创建用户组
sudo usermod -aG groupname username # 将用户添加近用户组 -a 表示追加 -G 表示指定组
```

> 如果更改了已登录系统账户所属的用户组，该用户必须登出系统后再登录，组关系的更改才能生效。

文件权限
`ls -l` 查看文件权限
`-rwxrwxr-x 1 root rich 4882 12月11日 13:55 myprog`
>`myprog`是个文件
>执行权限和所属于`root`用户(`rwx`)、`rich`用户组(`rwx`)、其他人(`r-x`)
>硬链接`1`个
>大小`4882byte`
>最后修改时间`12月11日 13:55`

改变文件所属
```bash
chmod options mode file
chown options owner[.group] file 
chgrp options group file
```
> `chmod`命令用来改变文件权限;`chown`命令用来改变文件的属主;`chgrp`命令用来改变文件的默认属组
> 只有`root`用户能够改变文件的属主;任何属主都可以改变文件的属组，但前提是属主必须是原属组和目标属组的成员
# Firewall
`redhat`系默认使用的防火墙工具，但底层还是`iptables`
![[firewalld-comparison.png]]
> `firewall-config` GUI配置工具，`firewall-cmd` CLI配置工具 `firewalld` 守护进程

启动: `systemctl start firewalld`
查看状态: `systemctl status firewalld` | `firewall-cmd --state`

查看所有设置: `firewall-cmd --list-all`
## 配置
`firewall`有`Zones`、`Service`

运行时和永久设置
运行时的修改会立刻执行，但当`firewalld`重启设置会充值回永久值
```bash
firewall-cmd --permanent <other options>   # 直接修改永久值

firewall-cmd --runtime-to-permanent        # 将运行时设置写入永久值

firewall-cmd --reload                      # 重启服务

firewall-cmd <other options> --timeout 15m # 15分钟后自动取消设置
```
### zones
其配置文件在`/etc/firewalld/zones/` 模板文件`/usr/lib/firewalld/zones/`
有`block`、`public`、`trusted`...
默认使用`public` 除了开放的服务`incoming`流量皆被禁止

`public.xml`
```xml
<?xml version="1.0" encoding="utf-8"?>
<zone>
  <short>Public</short>
  <description>For use in public areas. You do not trust the other computers on networks to not harm your computer. Only selected incoming connections are accepted.</description>
  <service name="ssh"/>
  <service name="dhcpv6-client"/>
  <service name="cockpit"/>
  <forward/>
</zone>
```
### service
其配置文件在`/etc/firewalld/services/` 模板文件`/usr/lib/firewalld/services/`

`ssh.xml`
```xml
<?xml version="1.0" encoding="utf-8"?>
<service>
  <short>SSH</short>
  <description>Secure Shell (SSH) is a protocol for logging into and executing commands on remote machines. It provides secure encrypted communications. If you plan on accessing your machine remotely via SSH over a firewalled interface, enable this option. You need the openssh-server package installed for this option to be useful.</description>
  <port protocol="tcp" port="22"/>
</service>
```

除了添加配置文件也可使用`firewall-cmd`添加服务
> 注意`firewalld`未激活时才能添加
```bash
firewall-cmd --new-service=service-name 
```
### port
直接操作开放端口
```bash
firewall-cmd --add-port=port-number/port-type      # 添加端口/协议

firewall-cmd --remove-port=port-number/port-type   # 删除端口/协议
```
> `port-type`可选: `tcp`、`udp`、`sctp`、`dccp`
# WSL
当linux不经如你心意，回归windows吧
## 基础命令
安装&卸载
```bash
wsl --install    # 会自动下载ubuntu作为wsl
wsl --list --online  # 查看当前官方支持的linux发行版  wsl -l -o
wsl --install -d <DistroName>  # 安装指定的linux发行版(需官方支持)

wsl --unregister <DistroName>  # 将从 WSL 取消注册该发行版，以便可以重新安装或清理
```
> 还可以像卸载任何其他应用商店应用程序一样卸载 Windows 计算机上的 Linux 发行版应用

运行状态
```bash
wsl --list --verbose # 列出已安装的发行版，查看wsl运行状态 
wsl -l -v # 简写版
```
>还可以配合其他参数 `--all` `--running`

优雅关机
```bash
wsl --shutdown   # 立即终止所有正在运行的发行版和 WSL 2 轻量级实用工具虚拟机
wsl --terminate <DistroName> # 终止指定的发行版或阻止其运行
```

查看IP
```bash
# 安装
wsl hostname -i # 查看windows -> linux IP
cat /etc/resolv.conf # 查看linux -> windows IP
```

导入导出
```bash
wsl --import <Distribution Name> <InstallLocation> <FileName>
wsl --import-in-place <Distribution Name> <FileName>  # 从当前目录导入

wsl --export <Distribution Name> <FileName>
```

## .wslconfig
保存位置：C:\\Users\\${username}\\.wslconfig

> 必须等到运行你的 Linux 发行版的子系统完全停止运行并重启，配置设置更新才会显示。 这通常需要关闭发行版 shell 的所有实例后大约 8 秒。

wsl.conf和.wslconfig?  
前者用于为在 WSL 1 或 WSL 2 上运行的每个 Linux 发行版按各个发行版配置**本地设置**；后者用于在 WSL 2 上运行的所有已安装发行版中配置**全局设置**

[.wslconfig可配置项](https://learn.microsoft.com/zh-cn/windows/wsl/wsl-config#wslconfig)

我暂时就配这个
```conf
[wsl2]
memory=4GB
swap=4GB
```

# windows GRUB
windows和linux装双系统时，一般使用grub作为引导启动程序。当删除linux后windows的系统分区可能会残存linux的引导文件，这里记录下如何删除系统分区中的引导文件

```bash
# 管理员模式打开终端
diskpart # 打开磁盘管理工具
list disk # 查看当前挂载的磁盘
sel disk n # 选择win的系统盘

list vol # 查看当前磁盘的分区情况
sel vol n # 选择EFI分区(就是系统分区，一般为FAT格式)
```

选择到系统分区后使用`assign letter=L:`将其映射到L盘(需要避开已存在的盘符)
```bash
# 再次以管理员模式打开终端
L:  # 进入 L:
cd EFI # 进入 EFI 文件夹
dir # 查看目前存在的引导文件
rmdir /S linux # 删除linux的引导文件
```

# ArchLinux 配置
Arch 及其衍生版的使用配置

## 开发环境

魔法
```bash
sudo pacman -S v2ray
yay -S v2raya-bin
sudo systemctl enable --now v2raya
```
> google dns ipv4:`8.8.8.8,8.8.4.4` ipv6:`2001:4860:4860::8888,2001:4860:4860::8844`

## 字体
```
yay ttf-maple       # 安装等宽字体
yay lxgw-wenkai     # 安装阅读字体(霞鹜)
```

基本软件
```bash
yay -S google-chrome
sudo pacman -S extra/jdk17-openjdk
sudo pacman -S npm
sudo pacman -S docker
sudo pacman -S docker-compose
yay -S jetbrains-toolbox
yay -S visual-studio-code-bin
yay -S apifox-bin
sudo pacman -S obsidian
yay -S linuxqq
yay -S dingtalk-bin
```

## swapfile
ext4格式(不支持btrfs文件格式)
```bash
sudo fdisk -l     # 查看挂载的磁盘
#创建交换文件
sudo dd if=/dev/nvme1n1p2 of=swapfile bs=1GB count=5 status=progress
#为交换文件设置权限(尽量不要设置为全局可用)
sudo chmod 0600 /swapfile
# 格式化
sudo mkswap -U clear /swapfile
# 启用
sudo swapon /swapfile

# 编辑 /etc/fstab 为交换文件添加一个条目(配置开机自启)
# /swapfile none swap defaults 0 0
# 可以使用 echo 添加
sudo bash -c "echo /swapfile none swap defaults 0 0 >> /etc/fstab"

# 删除
sudo swapoff /swapfile
rm -f /swapfile
# 最后记得删除 /etc/fstab 里的条目
```

## 中文输入法
安装`fcitx5`
```bash
sudo pacman -S fcitx5-im #基础包组
sudo pacman -S fcitx5-chinese-addons #官方中文输入引擎
sudo pacman -S fcitx5-anthy #日文输入引擎
sudo pacman -S fcitx5-pinyin-zhwiki #中文维基百科词库
sudo pacman -S fcitx5-material-color #主题
```

设置环境变量：`sudo vim /etc/environment`
```bash
GTK_IM_MODULE=fcitx
QT_IM_MODULE=fcitx
XMODIFIERS=@im=fcitx
SDL_IM_MODULE=fcitx
```
启用`fcitx5`后注销或重启系统

> 此方法已过时，请参考 [Using Fcitx5 on Wayland](https://fcitx-im.org/wiki/Using_Fcitx_5_on_Wayland)

## Gnome
安装插件需要先安装 `gnome-browser-connector`  
推荐插件 `Dash to Dock`、`netspeed`、 `Clipboard history` 、`AppIndicator and KStatusNotifierItem Support`

安装`gnome-tweak`来配置应用图标、鼠标图标、字体  
[壁纸下载]([Awesome Wallpapers - wallhaven.cc](https://wallhaven.cc/))、[gnome主题下载](https://www.gnome-look.org/browse/)

> gnome console binary is named `kgx`


# 各发行版包管理器
## pacman & yay
```bash
pacman -Syu                # 升级系统
pacman -S package          # 安装软件  有多个版本在不同仓库时 pacman -S repository/package
pacman -U package.pkg.tar.gz  # 安装本地软件包
pacman -R package          # 删除软件  同时删除无用依赖 pacman -Rs package
pacman -Ss string          # 在仓库中查询软件包
pacman -Qs string          # 查询已安装的软件包
pacman -Sc                 # 清理未安装的包文件  /var/cache/pacman/pkg
pacman -Scc                # 清理所有的缓存文件

yay    #  等价于   yay -Syu   会更新pacman 和 aur 的包
yay package  # 在源里查询 package 
yay -S package  # 安装 package 
yay -R package  # 删除 package 
yay -Yc  # 清理无用依赖
```

> 修改arch镜像 `sudo vim /etc/pacman.d/mirrorlist`
> 清华源 `Server = https://mirrors.ustc.edu.cn/archlinux/$repo/os/$arch`

## apt
```bash
#apt [选项] 命令
apt search|install|remove  [package]  #搜索安装移除
apt reinstall [package]  #重新安装

apt update  #- 更新可用软件包列表
apt upgrade  #- 通过 安装/升级 软件来更新系统
apt full-upgrade  #- 通过 卸载/安装/升级 来更新系统
apt list --upgradeable   #显示可升级的软件包
apt list --installed     #显示已安装的软件包
sudo apt update && sudo apt upgrade

apt autoremove  #卸载所有自动安装且不再使用的软件包

apt edit-sources #- 编辑软件源信息文件
apt show [package]   #显示软件包具体信息例如：版本号，安装大小，依赖关系，bug报告等等
apt show git   #显示软件信息
```

> apt 是 apt-get 的新版本可以完全代替

## dnf
~~等待填坑~~

# 杂项
## docker 添加用户组
```bash
sudo usermod -aG docker ${USER}
# 或
su - ${USER}
groups
sudo usermod -aG docker username
```

