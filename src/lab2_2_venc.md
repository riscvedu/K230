## 音视频录制实验

[venc_mapi API文档](docs/venc_mapi.md)
- [视频讲解](https://riscv-edu.cn/course/230/replay/6376)
- [实验指导部分](https://github.com/kendryte/k230_docs/blob/main/zh/01_software/board/examples/K230_SDK_CanMV_Board_Demo%E4%BD%BF%E7%94%A8%E6%8C%87%E5%8D%97.md#22-venc_demo)

### 实验步骤

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

5. 在小核，执行
```bash
cd /mnt
./sample_venc -s 24 -n 2 -o /tmp -t 0
```

6. 在VLC中打开网络串流

VLC -> 媒体 -> 网络 -> 输入网络URL -> 播放

> 实验成功，可以在视频播放器中播放摄像头拍摄到的视频

<img src="https://github.com/riscvedu/K230/assets/53103747/ad66e74a-93b3-46c7-b32e-67cbcbf3a3e9" width="300">
