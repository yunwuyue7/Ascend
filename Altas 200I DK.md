# Altas 200I DK

## 基本信息

- 操作系统版本：openEuler release 22.03 LTS
- 芯片型号：310B
- python版本：3.9

## 部署流程

### 1. 制卡

准备一张SD卡（64G足够）以安装OpenEuler系统镜像。

参考文档：[Windows系统制卡-Atlas 200I DK A2开发者套件-昇腾社区](https://www.hiascend.com/document/detail/zh/Atlas200IDKA2DeveloperKit/23.0.RC2/qs/qs_0005.html)

注意：制卡工具中的镜像版本选择openEuler22.03版本，相关的信息如下图所示，与参考文档中图片不一致。

![1](.\images\1.png)

按照参考文档进行操作，镜像烧录完成后，显示如下：

![1](.\images\2.png)

### 2. 启动、登录开发板

#### 启动

可参考[Windows系统制卡-Atlas 200I DK A2开发者套件-昇腾社区](https://www.hiascend.com/document/detail/zh/Atlas200IDKA2DeveloperKit/23.0.RC2/qs/qs_0005.html) “连接启动开发者套件 -- 本机显示模式”章节内容（左侧选择章节）。

- 将SD卡插入卡槽（见下图）
- 调整拨码开关的值
- HDMI接口连接显示器
- 上电

<img src=".\images\3.png" alt="1" style="zoom:40%;" />

正常情况下能看到图形化界面：

注意：用户名为“HwHiAiUser”，密码为“Mind@123”

<img src=".\images\4.png" alt="1" style="zoom:67%;" />

#### 登录

1. 参考“登录开发者套件 - 远程登录模式（Windows系统）- 以太网口远程登录（SSH方式）”章节：

   [快速开始-Atlas 200I DK A2开发者套件23.0.RC3-昇腾社区](https://www.hiascend.com/document/detail/zh/Atlas200IDKA2DeveloperKit/23.0.RC2/qs/qs_0017.html)

2. 如果不想修改本机的ip，还有另一种方式：

   - 将Atlas开发板的eth1网口（两个之中靠上的）接入路由器

   - 本机连接路由器的wifi，查看本机ip

   - 修改开发板`/etc/sysconfig/network-scripts`路径下的`ifcfg-eth1`文件（格式见下述代码块），将`IPADDR`字段修改为与本机ip网段相同的ip，同时修改网关、DNS等。保存后在开发板终端执行**nmcli con reload**重加载配置文件，再执行**nmcli con up eth1**或**nmcli con up /etc/sysconfig/network-scripts/ifcfg-eth1**命令激活配置。

     ```bash
     TYPE=Ethernet
     BOOTPROTO=static
     DEFROUTE=yes
     NAME=eth1
     DEVICE=eth1
     ONBOOT=yes
     IPADDR=192.168.1.161
     PREFIX=24
     GATEWAY=192.168.1.1
     DNS1=223.5.5.5
     DNS2=223.6.6.6
     ```

按上述方法之一完成设置后，可以远程登陆开发板：

<img src=".\images\6.png" alt="1" style="zoom:67%;" />

### 3. 转换模型



## Tips

#### 查看GPU设备状态

```bash
sh-5.1# npu-smi info
+----------------------------------------------------------------------------------------+
| npu-smi 23.0.rc3                                 Version: 23.0.rc3                     |
+--------------------+------------+------------------------------------------------------+
| NPU     Name       | Health     | Power(W)     Temp(C)           Hugepages-Usage(page) |
| Chip    Device     | Bus-Id     | AICore(%)    Memory-Usage(MB)                        |
+====================+============+======================================================+
| 0       310B4      | OK         | 7.7          54                15    / 15            |
| 0       0          | NA         | 0            2166 / 3513                             |
+====================+============+======================================================+
```

#### 查询CANN软件包版本

参考：[Altas产品查询CANN软件包版本的方法 - 华为](https://support.huawei.com/enterprise/zh/knowledge/EKB1100100366)

```bash
[HwHiAiUser@davinci-mini arm64-linux]$ cat ascend_toolkit_install.info
package_name=Ascend-cann-toolkit
version=7.0.RC1
innerversion=V100R001C13SPC005B246
compatible_version=[V100R001C29],[V100R001C30],[V100R001C13],[V100R003C10],[V100R003C11]
arch=aarch64
os=linux
path=/usr/local/Ascend/ascend-toolkit/7.0.RC1/aarch64-linux
```

