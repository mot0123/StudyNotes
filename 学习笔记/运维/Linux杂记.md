# Linux

## 系统设置

### 防火墙

1. 防火墙状态

```shell
systemctl status firewalld
```

2. 停止/关闭防火墙

```shell
systemctl stop/disable firewalld
```



### 时间同步ntp

> NTP即Network Time Protocol（网络时间协议），是一个互联网协议，用于同步计算机之间的系统时钟。timedatectl程序可以自动同步Linux系统时钟到使用NTP的远程服务器（大部分linux系统自带ntp服务而不带ntp client，所以纠结了半天找不到为什么都没装NTP每次改了时间又被同步回网络时间去了）。

1. 显示系统的当前时间和日期

```
timedatectl  status
```

2. 开启关闭ntp

```
timedatectl set-ntp true／false
```

#### 时间同步

安装

```shell
yum install ntpdate -y
```

同步

```
ntpdate time.windows.com
```



### SELinux

> 安全增强型Linux（SELinux）是一个Linux内核的功能，它提供支持访问控制的安全政策保护机制。本文介绍如何开启或关闭SELinux，并且避免系统无法启动的问题。

1. 编辑SELinux的`config`文件。

   ```
   vi /etc/selinux/config
   ```

2. 找到`SELINUX=disabled`，按`i`进入编辑模式，通过修改该参数开启SELinux。

   ![1](https://help-static-aliyun-doc.aliyuncs.com/assets/img/zh-CN/6818987851/p76895.png)

   您可以根据需求修改参数，开启SELinux有以下两种模式：

   - 强制模式`SELINUX=enforcing`：表示所有违反安全策略的行为都将被禁止。
   - 宽容模式`SELINUX=permissive`：表示所有违反安全策略的行为不被禁止，但是会在日志中作记录。

3. 修改完成后，按下键盘`Esc`键，执行命令`:wq`，保存并退出文件。

   > **说明** 修改`config`文件后，需要重启实例，但直接重启实例将会出现系统无法启动的错误。因此在重启之前需要在根目录下新建`autorelabel`文件。

   > 执行命令`setenforce 0`临时关闭SELinux。

4. 在根目录下新建隐藏文件`autorelabel`，实例重启后，SELinux会自动重新标记所有系统文件。

   ```
   touch /.autorelabel
   ```

5. 重启ECS实例。

   ```
   shutdown -r now
   ```

### 关闭Swap

```shell
swapoff -a  # 临时
sed -ri 's/.*swap.*/#&/' /etc/fstab    # 永久
```

### 设置主机名

```shell
# 根据规划设置主机名

hostnamectl set-hostname <hostname>
```



## 软件安装

### 1、centOS环境下：

不能用apt-get，要用yum命令更新或下载

更新：

```shell
yum -y update
```


下载(比如下载的是gcc）：

```shell
yum -y install gcc
```

### 2、ubuntu环境下：

1）下载源

```shell
wget  http://security.ubuntu.com/ubuntu/pool/main/a/apt/apt_1.9.3_amd64.deb
```

2）安装包

```
sudo dpkg -i apt_1.4_amd64.deb
```

3）运行apt-get

```
sudo apt-get install XXX
```

### 3、如何确定linux系统的版本

1)	先使用 yum 安装 redhat-lsb

```shell
yum install -y redhat-lsb
```

2)	使用命令

```shell
lsb_release -a
```


## Java编程相关

1、Linux环境下，在程序中文件路径写为

```shell
# 从程序运行的目录开始（比如jar运行目录在 /bin/java）
/test/a.txt  # 则目录在/bin/java/test/a.txt
# 从根目录
test/a.txt  # 则目录在/test/a.txt
```

2、注意该程序是否运行在容器中，则路径会相应在容器中



