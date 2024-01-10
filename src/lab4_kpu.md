# KPU加速模型推理实验

- [nncase Github主页](https://github.com/kendryte/nncase)
- [课程回放](https://riscv-edu.cn/course/230/replay/6399)

    - nncase是一个为 AI 加速器设计的神经网络编译器, 目前支持的 target有cpu/K210/K510/K230等.

    - nncase提供的功能：
        - 支持多输入多输出网络，支持多分支结构
        - 静态内存分配，不需要堆内存
        - 算子合并和优化
        - 支持 float 和uint8/int8量化推理
        - 支持训练后量化，使用浮点模型和量化校准集
        - 平坦模型，支持零拷贝加载

### 实验步骤

1. 进入docker环境
```bash
cd /path/to/k230_sdk
```
```bash
sudo docker run -u root -it -v $(pwd):$(pwd) -v $(pwd)/toolchain:/opt/toolchain -w $(pwd) ghcr.io/kendryte/k230_sdk /bin/bash
```

2. 安装nncase
```bash
pip install -i https://pypi.org/simple nncase==2.5.1 nncase-kpu==2.5.0
```

3. 验证nncase是否安装成功
```bash
pip list | grep nncase

nncase            2.5.1  
nncase-kpu        2.5.0
```

```bash
python3

Python 3.8.10 (default, May 26 2023, 14:05:08) 
[GCC 9.4.0] on linux
Type "help", "copyright", "credits" or "license" for more information.
>>> import _nncase
>>> print(_nncase.__version__)
2.5.1
>>> quit()
```

4. 编译模型

```bash
cd src/big/nncase/examples/
./build_model.sh
```

```bash
ls -l tmp/

total 12
drwxr-xr-x 11 root root 4096 Oct 31 11:23 mbv2_tflite
drwxr-xr-x 11 root root 4096 Oct 31 11:26 mobile_retinaface
drwxr-xr-x 11 root root 4096 Oct 31 11:24 yolov5s_onnx
```

5. 编译App

| App               | 备注                                                         |
| ----------------- | ------------------------------------------------------------ |
| image_classify    | 图片分类demo, 输入是RGB图片, 推理结果打印到串口              |
| object_detect     | 目标检测demo, 输入是RGB图片, 推理结果打印到串口              |
| image_face_detect | 人脸检测demo, 输入是RGB图片, 推理结果会输出到画了人脸box和landmark的图片 |

```bash
./build_app.sh
```

编译app结束后, 默认会将demo及其运行所需文件拷贝到当前目录下的k230_bin子目录

```bash
ls -l k230_bin/
```
```bash
total 12
drwxrwxr-x 2 1000 1000 4096 Oct 31 12:41 image_classify
drwxrwxr-x 2 1000 1000 4096 Oct 31 12:42 image_face_detect
drwxrwxr-x 2 1000 1000 4096 Oct 31 12:42 object_detect
```

### 上板运行 

**前置条件**

- gnne频率: 800MHZ
- noc限速: 3.2GB/s

- 先进入sharefs
```bash
cd sharefs/
```

#### 图片分类demo

```bash
/image_classify>./cpp.sh
```
```bash
case ./image_classify.elf built at Oct 31 2023 11:54:35
interp.run() took: 2.19156 ms
image classify result: tabby(0.453891)
```

- 实验成功

<img src="https://github.com/riscvedu/K230/assets/53103747/de7dc79d-5d97-4514-ac3f-822d3b4724d9" width="300">

#### 目标检测demo

```bash
/object_detect>./cpp.sh
```
```bash
case ./object_detect.elf built at Oct 31 2023 11:54:35
od set_input took 0.176074 ms
od run took 16.4843 ms
od get output took 12.759 ms
post process took 55.0952 ms
text = truck:0.300000
text = dog:0.250000
text = bicycle:0.230000
draw result took 31.1012 ms
```

- 实验成功

<img src="https://github.com/riscvedu/K230/assets/53103747/190fbdfc-a4c2-4758-bc34-ca21bf3d4b26" width="300">

#### 人脸检测demo

```bash
/image_face_detect>./cpp.sh
```
```bash
case ./image_face_detect.elf built at Oct 31 2023 11:54:47
Press 'q + enter' to exit!!!
g
```

推理结果(人脸box和landmark)会生成到face_500x500_result_x.jpg

- 实验成功

<img src="https://github.com/riscvedu/K230/assets/53103747/98133780-a3a8-4410-ac5e-2a91b40fce8f" width="300">
