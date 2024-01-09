## rtsp sever搭建和推流实验

- [官方文档](https://github.com/kendryte/k230_docs/blob/main/zh/01_software/board/examples/K230_SDK_CanMV_Board_Demo%E4%BD%BF%E7%94%A8%E6%8C%87%E5%8D%97.md#212-rtsp%E6%8E%A8%E6%B5%81demo)
- [视频讲解](https://riscv-edu.cn/course/230/replay/6375)

通过该实验，我们将学会调用K230开发板上的音视频编码器，将其推流到rtsp server上。其中通过mapi venc&aenc接口实现对音视频的编码；推流之后在VLC中通过url进行拉取，目前该demo支持3路url推拉流。

#### 依赖资源

- 网线x1，用于将开发板连接到局域网

#### 源码介绍

K230 SDK采用双核架构，小核运行linux系统，实现网络控制服务。大核运行RTT系统，实现对音视频硬件的控制。小核在启动过程中负责引导大核。大小核通过核间通信进行消息通信和内存共享。
小核网络服务移植了live源码，路径为k230_sdk/src/common/cdk/user/thirdparty/live 音频使用G711编码。 音视频推流中，视频默认最大分辨率与配置的sensor_type相关，默认最大分辨率为1920x1080。

#### 程序主要步骤

##### 音视频推流程序

1. `new StreamingPlayer`
   - 初始化核间通信。
   - 配置video buffer。
1. `InitVicap`：初始化摄像头。
1. `CreateSession`：
   - 创建server session。
   - 创建并开启音频输入通道，音频采样率为44.1k，采样宽度16bit。
   - 初始化音频编码，音频采用G711A编码。
   - 创建视频编码通道，使能IDR帧。
1. `Start`：
   - 开启视频编码通道并绑定到vo。
   - 开启视频编码通道，并将视频输入绑定到视频编码。
   - 开始音视频推流。

##### 语音对讲程序

1. `Init`：
   - 创建server session。
   - 初始化核间通信。
   - 配置video buffer。
   - 创建并开启音频输入通道，音频采样率为8k，采样宽度为16bit。
   - 创建音频编码通道，音频采用G711U编码。
   - 创建音频输出通道和音频解码通道，并将音频解码绑定到音频输出。
1. `Start`：
   - 开始推音频流。
   - 开启视频编码通道，并将视频输入绑定到视频编码。

### 实验步骤

-. 在k230_sdk目录下执行make cdk-user，在k230_sdk/src/common/cdk/user/out/little/目录下生成rtsp_demo

    - 相关源码

        在SDK中包含的rtsp相关demo位于k230_sdk/src/common/cdk/user/samples目录下，其中：\

        - rtsp_demo：语音推音视频流程序
        - rtsp_server：语音对讲服务器端程序
        - backchannel_client：语音对讲客户端程序

-. 启动开发板

1. 用网线将开发板连接局域网

2. 检查k_ipcm模块是否加载
```bash
lsmod # 列出已加载的内核模块
insmod k_ipcm.ko # 若未加载，进行k_ipcm加载
```
通过 lsmod 检查小核侧是否加载k_ipcm模块，如未加载，执行 insmod k_ipcm.ko 加载k_ipcm模块\

3. 在大核侧，执行
```bash
cd sharefs/app
./sample_sys_inif.elf #启动核间通信进程
```

- sample_sys_inif.elf 使用说明

```bash
# 执行./sample_venc.elf -h后，输出demo的使用说明，如下：
Usage : ./sample_venc.elf [index] -sensor [sensor_index] -o [filename]
index:
    0) H.265e.
    1) JPEG encode.
    2) OSD + H.264e.
    3) OSD + Border + H.265e.

sensor_index: see vicap doc
```

4. 在小核，执行

```bash
cd /mnt
./rtsp_demo -s 24 -n 2 -t h265 -w 1280 -h 720 -a 0
```
#### 参数说明

| 参数名 | 描述 |参数范围 | 默认值 |
|:--|:--|:--|:--|
| help | 打印命令行参数信息 | - | - |
| n | session个数 | `[1, 3]` | 1 |
| t | 编码类型 | h264、h265、mjpeg | h264 |
| w | 视频编码宽度 | `[640, 1920]` | 1280 |
| h | 视频编码高度 | `[480, 1080]` |720 |
| s | sensor类型| 查看camera sensor文档 | 7 |

5. 在VLC中打开网络串流

VLC -> 媒体 -> 网络 -> 输入网络URL -> 播放


> 实验成功，可以在VLC网络串流中实时看到摄像头拍摄的画面

<img src="https://github.com/riscvedu/K230/assets/53103747/301931e8-075d-4d4f-8cef-7bf8726a7dd4" width="300">

