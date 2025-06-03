# Software Developer (Example)

## 1. 

Base64编码

**system**: 

```
I want you to act as a software developer. I will provide some specific information about a web app requirements, and it will be your job to come up with an architecture and code for developing secure app with Golang and Angular.
```

**user**: 

```
现在有新的报错，请你解决，似乎是某个so文件无法正常引入

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
sh-5.1# cd /usr/local/Ascend/ascend-toolkit/latest
sh-5.1# ls
aarch64-linux  arm64-linux  bin       conf       include  mindstudio-toolkit  ops           pyACL   remove_latest_softlink.sh  test-ops  tools
acllib         atc          compiler  fwkacllib  lib64    opp                 platform.ini  python  runtime                    toolkit   version.cfg
sh-5.1# find . -name "libascendcl.so"
./aarch64-linux/devlib/x86_64/libascendcl.so
./aarch64-linux/devlib/aarch64/libascendcl.so
./aarch64-linux/devlib/libascendcl.so
./aarch64-linux/lib64/libascendcl.so
```

**assistant**: 

```
以下是针对 `libascendcl.so` 缺失问题的详细解决方案：

---

### **问题根源分析**
错误 `ImportError: libascendcl.so: cannot open shared object file` 表明：
1. **CANN 环境未正确配置**：未设置 `LD_LIBRARY_PATH` 或 `ASCEND_TOOLKIT_HOME`
2. **架构路径混淆**：系统中存在 x86_64 和 aarch64 两种架构的库文件，但未正确选择 ARM 版本

---

### **解决方案**
按顺序执行以下步骤：

#### **步骤 1：确认系统架构**

uname -m

- 输出应为 `aarch64`（华为昇腾设备通常为 ARM 架构）

#### **步骤 2：设置环境变量**

# 设置 CANN 工具包根目录
export ASCEND_TOOLKIT_HOME=/usr/local/Ascend/ascend-toolkit/latest

# 添加 ARM 架构库路径到链接库搜索路径
export LD_LIBRARY_PATH=$ASCEND_TOOLKIT_HOME/aarch64-linux/lib64:$LD_LIBRARY_PATH

# 添加 Python 插件路径（关键！）
export PYTHONPATH=$ASCEND_TOOLKIT_HOME/pyACL/python/site-packages/acl:$PYTHONPATH

# 永久生效（可选）
echo "export ASCEND_TOOLKIT_HOME=$ASCEND_TOOLKIT_HOME" >> ~/.bashrc
echo "export LD_LIBRARY_PATH=$ASCEND_TOOLKIT_HOME/aarch64-linux/lib64:\$LD_LIBRARY_PATH" >> ~/.bashrc
echo "export PYTHONPATH=$ASCEND_TOOLKIT_HOME/pyACL/python/site-packages/acl:\$PYTHONPATH" >> ~/.bashrc
source ~/.bashrc


#### **步骤 3：验证环境配置**

# 检查库路径是否生效
echo $LD_LIBRARY_PATH

# 直接测试库加载
ldconfig -p | grep libascendcl.so  # 应显示有效路径


#### **步骤 4：运行昇腾环境初始化脚本**
华为 CANN 工具包通常提供环境初始化脚本：

# 执行环境初始化（路径可能不同，根据实际位置调整）
source $ASCEND_TOOLKIT_HOME/aarch64-linux/set_env.sh


#### **步骤 5：以正确权限运行程序**
避免使用 `root` 用户直接运行（可能导致环境变量失效）：

# 切换到普通用户
su - your_username

# 或保留环境变量使用 sudo
sudo -E python -m ais_bench --model ...  # -E 保留用户环境变量


---

### **关键路径说明**
根据你的 `find` 结果，关键文件位置如下：

/usr/local/Ascend/ascend-toolkit/latest
├── aarch64-linux
│   ├── devlib
│   │   ├── x86_64/libascendcl.so  # x86 架构（错误路径）
│   │   └── aarch64/libascendcl.so # ARM 架构（正确路径）
│   └── lib64/libascendcl.so       # 主库路径（需添加到 LD_LIBRARY_PATH）
└── pyACL
    └── python/site-packages/acl   # Python 接口路径（需添加到 PYTHONPATH）


---

### **高级调试技巧**
#### **1. 手动加载库验证**

# 使用 ldd 检查依赖关系
ldd $(python -c "import aclruntime; print(aclruntime.__file__)") | grep ascendcl

# 预期输出应显示正确路径：
# libascendcl.so => /usr/local/Ascend/ascend-toolkit/latest/aarch64-linux/lib64/libascendcl.so


#### **2. 检查 CANN 版本兼容性**

# 查看 CANN 版本
cat $ASCEND_TOOLKIT_HOME/version.cfg

# 确认与 ais_bench 的版本匹配（华为昇腾社区提供版本对照表）


#### **3. 重装 CANN 工具包（终极方案）**
如果问题仍未解决，重新安装 CANN 工具包：
1. 从 [华为昇腾社区](https://ascend.huawei.com) 下载对应版本
2. 按官方文档安装：

./Ascend-cann-toolkit_6.0.0_linux-aarch64.run --install


---

### **华为昇腾环境检查清单**
| 检查项                  | 命令/操作                                                                 |
|-------------------------|--------------------------------------------------------------------------|
| 芯片驱动状态            | `npu-smi info`                                                          |
| CANN 工具包版本         | `cat $ASCEND_TOOLKIT_HOME/version.cfg`                                  |
| 环境变量完整性          | `env | grep -E 'ASCEND|LD_LIBRARY|PYTHON'`                          |
| 用户权限                | 确保非 root 用户运行或使用 `sudo -E`                                    |

---

通过以上步骤，应该能解决 `libascendcl.so` 的加载问题。如果仍然报错，请提供以下信息以便进一步排查：
1. `echo $LD_LIBRARY_PATH` 的输出
2. `npu-smi info` 的结果
3. CANN 工具包的具体版本号
```

**user**: 

```
[HwHiAiUser@davinci-mini /]$ echo $LD_LIBRARY_PATH
/usr/local/Ascend/ascend-toolkit/latest/lib64:/usr/local/Ascend/ascend-toolkit/latest/lib64/plugin/opskernel:/usr/local/Ascend/ascend-toolkit/latest/lib64/plugin/nnengine:/usr/local/Ascend/ascend-toolkit/latest/opp/built-in/op_impl/ai_core/tbe/op_tiling:/var/davinci/driver/lib64:/var/davinci/driver/lib64/common:/var/davinci/driver/lib64/driver:
[HwHiAiUser@davinci-mini /]$ ldconfig -p | grep libascendcl.so
```

**assistant**: 

```
根据你提供的信息，问题出在 **LD_LIBRARY_PATH 路径配置错误** 和 **系统库缓存未正确注册**。以下是针对性解决方案：

---

### **问题诊断**
1. **路径错误**  
   你的 `LD_LIBRARY_PATH` 中配置的是 `/usr/local/Ascend/ascend-toolkit/latest/lib64`，但实际 `libascendcl.so` 位于子目录 `aarch64-linux/lib64` 中：
   
   # 当前错误路径
   /usr/local/Ascend/ascend-toolkit/latest/lib64

   # 正确路径应为
   /usr/local/Ascend/ascend-toolkit/latest/aarch64-linux/lib64
   

2. **库未注册到系统缓存**  
   `ldconfig -p` 未找到库，说明路径未添加到系统库配置中。

---

### **解决方案**
分两步修复：

#### **步骤 1：修正环境变量**

# 清除旧路径（避免干扰）
export LD_LIBRARY_PATH=$(echo $LD_LIBRARY_PATH | tr ':' '\n' | grep -v "/usr/local/Ascend/ascend-toolkit/latest/lib64" | tr '\n' ':' | sed 's/:$//')

# 添加正确路径
export LD_LIBRARY_PATH=/usr/local/Ascend/ascend-toolkit/latest/aarch64-linux/lib64:$LD_LIBRARY_PATH

# 验证路径
echo $LD_LIBRARY_PATH


#### **步骤 2：永久注册库路径**

# 创建配置文件
sudo tee /etc/ld.so.conf.d/ascend.conf <<EOF
/usr/local/Ascend/ascend-toolkit/latest/aarch64-linux/lib64
/usr/local/Ascend/driver/lib64
EOF

# 更新系统库缓存
sudo ldconfig

# 验证库是否可发现
ldconfig -p | grep libascendcl.so


---

### **补充操作**
#### **1. 检查华为昇腾环境初始化脚本**
华为 CANN 工具包通常提供 `set_env.sh`，直接运行可自动配置环境：

# 执行环境初始化脚本（路径可能不同）
source /usr/local/Ascend/ascend-toolkit/latest/aarch64-linux/set_env.sh

# 验证环境变量
env | grep -E 'ASCEND|LD_LIBRARY|PYTHON'


#### **2. 检查用户权限**
确保以 **HwHiAiUser** 用户运行程序（华为昇腾默认用户）：

# 切换用户
su - HwHiAiUser

# 重新运行程序
python -m ais_bench --model ...


#### **3. 验证昇腾驱动状态**

# 检查昇腾芯片状态
npu-smi info

# 预期输出示例
+----------------------------------------------------------------------------------------+
| npu-smi 21.0.4                 Version: 21.0.4                                       |
+----------------------+---------------+------------------------------------------------+
| NPU   Name           | Health        | Power(W)  Temp(C)  HBM-Usage(MB)  Utilization(%) |
| Chip                 |               |                                                  |
+======================+===============+================================================+
| 0     Ascend910      | OK            | 65.8      45       0/32768         0%            |
+----------------------+---------------+------------------------------------------------+


---

### **关键配置总结**
| 配置项                  | 正确值                                                                 |
|-------------------------|----------------------------------------------------------------------|
| `LD_LIBRARY_PATH`        | `/usr/local/Ascend/ascend-toolkit/latest/aarch64-linux/lib64:...`    |
| `ASCEND_TOOLKIT_HOME`    | `/usr/local/Ascend/ascend-toolkit/latest`                            |
| 系统库配置文件           | `/etc/ld.so.conf.d/ascend.conf`                                      |
| 运行用户                 | `HwHiAiUser`（非 root）                                              |

---

### **仍不成功的终极方案**
如果问题依旧，可能是 **CANN 工具包安装不完整**，需重新安装：
1. 从华为昇腾社区下载 CANN 工具包（确保选择 **aarch64** 版本）
2. 卸载旧版本：
   
   sudo /usr/local/Ascend/ascend-toolkit/latest/uninstall.sh
   
3. 安装新版本：
   
   sudo ./Ascend-cann-toolkit_6.0.0_linux-aarch64.run --install
   

---

通过以上步骤，`libascendcl.so` 的加载问题应得到解决。如果仍有异常，请提供以下信息：
1. `npu-smi info` 的完整输出
2. `ls -l /usr/local/Ascend/ascend-toolkit/latest/aarch64-linux/lib64/libascendcl.so`
```

**user**: 

```
当我运行bilstm_postprocess.py这个文件时，有以下报错，如何解决？

sh-5.1# python bilstm_postprocess.py     --infer_results inference/results_bs1     --annotations inference/prep_data/label.txt
0it [00:00, ?it/s]
Traceback (most recent call last):
  File "/home/HwHiAiUser/Desktop/ModelZoo-PyTorch/ACL_PyTorch/built-in/nlp/BiLSTM_CRF_PyTorch/bilstm_postprocess.py", line 125, in <module>
    main()
  File "/home/HwHiAiUser/Desktop/ModelZoo-PyTorch/ACL_PyTorch/built-in/nlp/BiLSTM_CRF_PyTorch/bilstm_postprocess.py", line 111, in main
    features = torch.from_numpy(np.load(str(path)))
  File "/usr/local/lib64/python3.9/site-packages/numpy/lib/npyio.py", line 462, in load
    raise ValueError("Cannot load file containing pickled data "
ValueError: Cannot load file containing pickled data when allow_pickle=False
```

**assistant**: 

```
以下是针对该错误的详细解决方案：

---

### **错误原因分析**
错误 `ValueError: Cannot load file containing pickled data when allow_pickle=False` 表明：
1. **numpy 加载策略限制**：代码试图加载包含 Python pickle 序列化数据的 `.npy` 文件
2. **安全机制触发**：numpy 默认禁用 `allow_pickle` 以防止潜在的安全风险（恶意序列化数据）

---

### **解决方案**
修改代码中所有 `np.load()` 调用，添加 `allow_pickle=True` 参数：

#### **修改点 1：CRF 转移矩阵加载**
在 `Postprocessor.build_crf` 方法中：

# 原代码
state_dict = {
    'transitions': torch.from_numpy(np.load(transitions_npy))
}

# 修改后
state_dict = {
    'transitions': torch.from_numpy(np.load(transitions_npy, allow_pickle=True))
}


#### **修改点 2：推理结果文件加载**
在 `main` 函数循环中：

# 原代码
features = torch.from_numpy(np.load(str(path)))

# 修改后
features = torch.from_numpy(np.load(str(path), allow_pickle=True))


---

### **完整修改后的关键代码段**

class Postprocessor:
    def build_crf(self, transitions_npy):
        crf = CRF(tagset_size=len(self.label2id), 
                  tag_dictionary=self.label2id, 
                  device=torch.device("cpu"))
        state_dict = {
            'transitions': torch.from_numpy(np.load(transitions_npy, allow_pickle=True))  # 添加参数
        }
        crf.load_state_dict(state_dict)
        crf.eval()
        return crf

def main():
    # ...其他代码...
    for path in tqdm.tqdm(Path(args.infer_results).iterdir()):
        features = torch.from_numpy(np.load(str(path), allow_pickle=True))  # 添加参数
        data_index = int(path.stem.split('_')[1])
        info = anno_info[data_index]
        pred_tags, label_entities = postprocessor(features, info['lens'])
        metric.update(pred_paths=pred_tags, label_paths=info['tags'])


---

### **附加建议**
1. **检查输入文件安全性**  
   确保 `.npy` 文件来源可信（pickle 可能执行任意代码）
   
   # 查看文件是否包含 pickle 数据
   head -c 100 inference/results_bs1/*.npy | grep -a 'numpy.core.numeric'
   

2. **升级 numpy 版本**  
   使用新版 numpy 获得更清晰的错误提示：
   
   pip install --upgrade numpy
   

3. **替代序列化方案**  
   建议改用更安全的格式（需修改前处理代码）：
   
   # 保存时使用
   np.savez_compressed("data.npz", arr=features)  # 避免 pickle

   # 加载时使用
   data = np.load("data.npz")
   features = data['arr']
   

---

### **操作验证步骤**
1. 修改代码后重新运行：
   
   python bilstm_postprocess.py \
       --infer_results inference/results_bs1 \
       --annotations inference/prep_data/label.txt
   

2. 预期输出应显示进度条和评估指标：
   
   100%|██████████| 128/128 [00:03<00:00, 35.6it/s]
   metrics:
   {'precision': 0.85, 'recall': 0.83, 'f1': 0.84}
   

---

### **可能延伸问题**
如果修改后出现 `UnicodeDecodeError`，说明 pickle 数据存在版本兼容问题：

# 添加指定编码格式
np.load(file, allow_pickle=True, encoding='latin1')


---

通过以上修改，应该能解决该 pickle 加载错误。如果问题仍然存在，请提供以下信息：
1. 生成 `.npy` 文件的前处理代码片段
2. `np.__version__` 的输出结果
3. 文件头信息：`file inference/results_bs1/*.npy`
```

**user**: 

```
Traceback (most recent call last):
  File "/usr/local/lib64/python3.9/site-packages/numpy/lib/npyio.py", line 465, in load
    return pickle.load(fid, **pickle_kwargs)
_pickle.UnpicklingError: invalid load key, '\x00'.

The above exception was the direct cause of the following exception:

Traceback (most recent call last):
  File "/home/HwHiAiUser/Desktop/ModelZoo-PyTorch/ACL_PyTorch/built-in/nlp/BiLSTM_CRF_PyTorch/bilstm_postprocess.py", line 125, in <module>
    main()
  File "/home/HwHiAiUser/Desktop/ModelZoo-PyTorch/ACL_PyTorch/built-in/nlp/BiLSTM_CRF_PyTorch/bilstm_postprocess.py", line 111, in main
    features = torch.from_numpy(np.load(str(path), allow_pickle=True))
  File "/usr/local/lib64/python3.9/site-packages/numpy/lib/npyio.py", line 467, in load
    raise pickle.UnpicklingError(
_pickle.UnpicklingError: Failed to interpret file 'inference/results_bs1/data_00001122_0.bin' as a pickle
```

**assistant**: 

```
---

### **问题根源分析**
新的错误 `UnpicklingError: invalid load key, '\x00'` 表明：
1. **文件格式不匹配**：`data_00001122_0.bin` 文件不是合法的 `.npy` 格式
2. **工具链输出错误**：华为昇腾的推理工具 `ais_bench` 可能未正确生成输出文件
3. **文件内容损坏**：文件头信息缺失或二进制内容异常

---

### **解决方案**
分三步排查和修复：

#### **步骤 1：验证文件格式**

# 检查文件类型
file inference/results_bs1/data_00001122_0.bin

# 预期输出（如果是合法的numpy文件）：
# data_00001122_0.bin: NumPy array, version 1.0, header...

# 如果输出显示为 "data" 或 "empty"，说明文件格式错误


#### **步骤 2：修正 ais_bench 输出格式**
确保 `ais_bench` 运行时指定正确的输出格式：

# 原命令（可能导致二进制格式错误）
python -m ais_bench --outfmt NPY ...

# 改为强制使用纯二进制格式（华为昇腾专用格式需特殊处理）
python -m ais_bench --outfmt BIN ...

# 或尝试原始数据模式
python -m ais_bench --outfmt RAW ...


#### **步骤 3：手动解析华为昇腾二进制文件**
如果文件是华为昇腾专用格式，需使用华为提供的解析方法：


# 修改 bilstm_postprocess.py 的加载逻辑
import acl

def load_acl_bin(file_path, dtype=np.float32):
    # 华为昇腾二进制文件读取接口
    buf = acl.util.create_host_buffer_from_file(file_path)
    data = np.frombuffer(buf, dtype=dtype)
    return data

# 替换原代码中的 np.load
features = torch.from_numpy(load_acl_bin(str(path)))


---

### **关键操作验证**

#### **1. 快速测试文件完整性**

# 临时调试脚本 test_file.py
import numpy as np

path = "inference/results_bs1/data_00001122_0.bin"
try:
    data = np.load(path, allow_pickle=True)
    print(f"File loaded successfully. Shape: {data.shape}")
except Exception as e:
    print(f"Error: {str(e)}")
    with open(path, 'rb') as f:
        header = f.read(10)
        print(f"File header: {header}")


#### **2. 华为昇腾二进制文件规范**
华为昇腾推理输出文件通常具有以下特征：
- 文件名格式：`输出名称_批次编号.bin`
- 数据布局：`NCHW` 或华为自定义格式
- 数据类型：`float32` 或 `int32`

---

### **备选方案**
如果问题仍未解决，可尝试以下方法：

#### **1. 使用华为 ModelZoo 官方后处理工具**

# 从 ModelZoo 下载专用后处理脚本
wget https://gitee.com/ascend/modelzoo/raw/master/built-in/ACL_PyTorch/nlp/BiLSTM_CRF_PyTorch/bilstm_postprocess.py

# 对比文件差异
diff your_bilstm_postprocess.py official_bilstm_postprocess.py


#### **2. 重新生成推理结果**

# 清理旧结果
rm -rf inference/results_bs1/*

# 重新运行推理（确保输出目录为空）
python -m ais_bench \
    --model ./inference/bilstm_bs1.om \
    --output_dirname results_bs1 \
    --outfmt NPY  # 或 BIN 根据工具版本调整


---

### **华为昇腾环境检查清单**
| 检查项                  | 命令/操作                                                                 |
|-------------------------|--------------------------------------------------------------------------|
| 推理工具版本            | `python -m ais_bench --version`                                         |
| 模型输入输出格式        | 使用 `omg` 工具解析模型：`omg --model=bilstm_bs1.om --mode=analyse`     |
| 文件权限                | `ls -l inference/results_bs1/data_00001122_0.bin`                       |

---

### **错误场景模拟**
若需复现问题，可手动创建错误文件：

# 生成伪文件（模拟错误）
with open("inference/results_bs1/error.bin", 'wb') as f:
    f.write(b'\x00\x00\x00')  # 写入非法内容


---

请通过以上步骤定位具体原因。如果问题仍然存在，请提供：
1. `file inference/results_bs1/data_00001122_0.bin` 的输出
2. `ais_bench` 的完整运行命令
3. 模型转换时使用的 `atc` 命令参数
```
--------------------


<a href="https://chatboxai.app" style="display: flex; align-items: center;">
<img src='https://chatboxai.app/icon.png' style='width: 40px; height: 40px; padding-right: 6px'>
<b style='font-size:30px'>Chatbox AI</b>
</a>
