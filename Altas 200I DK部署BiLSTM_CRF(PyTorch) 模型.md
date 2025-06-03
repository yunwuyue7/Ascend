# Altas 200I DK部署BiLSTM_CRF(PyTorch) 模型

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

#### a.启动

可参考[Windows系统制卡-Atlas 200I DK A2开发者套件-昇腾社区](https://www.hiascend.com/document/detail/zh/Atlas200IDKA2DeveloperKit/23.0.RC2/qs/qs_0005.html) “连接启动开发者套件 -- 本机显示模式”章节内容（左侧选择章节）。

- 将SD卡插入卡槽（见下图）
- 调整拨码开关的值
- HDMI接口连接显示器
- 上电

<img src=".\images\3.png" alt="1" style="zoom:40%;" />

正常情况下能看到图形化界面：

**注意**：用户名为“HwHiAiUser”，密码为“Mind@123”

<img src=".\images\4.png" alt="1" style="zoom:67%;" />

#### b.登录

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

### 3. 部署模型

主要参考[Ascend/ModelZoo-PyTorch - Gitee.com](https://gitee.com/ascend/ModelZoo-PyTorch/tree/master/ACL_PyTorch/built-in/nlp/BiLSTM_CRF_PyTorch#快速上手)的“快速上手”章节，也有一些与文档中不同的操作，在下面一一列出。

#### a.获取源码

修改`requirements.txt` 中的torch版本：`torch==2.0.1`（修改前为1.13.0）

#### b.准备数据集

1. 文档中以下列命令下载数据集，但链接好像挂了（HTTP Error 403）

   ```bash
   cd CLUENER2020/bilstm_crf_pytorch/
   python download_clue_data.py --data_dir=./dataset --tasks=cluener 
   cd ../../
   ```

   可以从[yunwuyue7/Ascend](https://github.com/yunwuyue7/Ascend#)仓库中获取`archive.zip`压缩包，解压后得到所需的文件，放在`BiLSTM_CRF_PyTorch/CLUENER2020/bilstm_crf_pytorch/dataset/cluener`路径下。

   生成的数据目录结构与文档中一致：

   ```bash
   ├── CLUENER2020/
    ├── bilstm_crf_pytorch/
        ├── dataset/
            ├── cluener/
                ├── README.md
                ├── cluener_predict.json
                ├── dev.json
                ├── test.json
                └── train.json
   ```

2.  执行`bilstm_preprocess.py`程序时，可能会有import Error。

   可以按下述方式修改导入，也可以用其他方式，只要能保证正确导入即可。

   后续执行其他几个python程序时，也可能有类似的问题，以相同的方法解决即可。

     ```python
     import os
     import sys
     import json
     import stat
     import argparse
     from pathlib import Path
     
     import numpy as np
     import torch
     import torch.nn as nn
     import tqdm
     import config
     
     # 获取当前脚本所在目录
     current_dir = Path(__file__).resolve().parent
     # 计算 CLUENER2020/bilstm_crf_pytorch 的绝对路径
     voca_dir = current_dir / "CLUENER2020" / "bilstm_crf_pytorch"
     # 添加至 Python 搜索路径
     sys.path.insert(0, str(voca_dir))
     from vocabulary import Vocabulary
     from common import seed_everything
     ```

#### c.模型转换

1. 执行gitee文档中`1.PyTroch 模型转 ONNX 模型`时，`run_lstm_crf.py`程序默认跑50个epoch，如果要先跑通流程，可改为跑两三个epoch（修改`run_lstm_crf.py` line209），可以大幅缩短耗时。

2. 执行 gitee文档`2.ONNX 模型转 OM 模型`时，`chip_name`设置为`310B4`

   ```bash
   chip_name=310B4
   ```

#### 推理验证

文档中提到先安装`aclruntime`、`ais_bench`两个工具，可以先执行`find . -name "libascendcl.so"`，能找到so文件就不需要安装。

之前遇到以下问题，可能是预置的`aclruntime`库与新安装的`aclruntime`库有冲突，无法正确导入。

```bash
sh-5.1# python -m ais_bench     --model ./inference/bilstm_bs${bs}.om     --input inference/prep_data/inputs/input_ids,inference/prep_data/inputs/input_mask     --output ./inference/     --output_dirname results_bs${bs}     --outfmt NPY
Traceback (most recent call last):
  File "/usr/lib64/python3.9/runpy.py", line 197, in _run_module_as_main
    return _run_code(code, main_globals, None,
  File "/usr/lib64/python3.9/runpy.py", line 87, in _run_code
    exec(code, run_globals)
  File "/usr/local/lib/python3.9/site-packages/ais_bench/__main__.py", line 3, in <module>
    exec(open(os.path.join(cur_path, "infer/__main__.py")).read())
  File "<string>", line 13, in <module>
  File "/usr/local/lib/python3.9/site-packages/ais_bench/infer/interface.py", line 3, in <module>
    import aclruntime
ImportError: libascendcl.so: cannot open shared object file: No such file or directory
```

若出现相同的error，可参考[yunwuyue7/Ascend](https://github.com/yunwuyue7/Ascend#)仓库中的`LLM-help`文档，实测此文档提供的方案可以解决问题。

**精度验证**结果如下：

<img src=".\images\5.png" alt="1" style="zoom:67%;" />

**性能验证**结果如下：

<img src=".\images\7.png" alt="1" style="zoom:67%;" />

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

