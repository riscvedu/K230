# k230实验班技术资料

- 课程主页：https://riscv-edu.cn/course/230

- [k230文档及相关资源Github主页](https://github.com/kendryte/k230_docs)
    - [K230 SDK CanMV Board Demo使用指南](https://github.com/kendryte/k230_docs/blob/main/zh/01_software/board/examples/K230_SDK_CanMV_Board_Demo%E4%BD%BF%E7%94%A8%E6%8C%87%E5%8D%97.md)

- [RT-Thread社区论坛](https://club.rt-thread.org/index.html)
    - [RT-Smart在riscv中的初始化流程](https://club.rt-thread.org/ask/article/c994a22a0cf2bb76.html)
    - [RT-Smart riscv64汇编注释](https://club.rt-thread.org/ask/article/cb935a6d9794d770.html)

## 1.2

- [从零开始玩转K230](从零开始玩转K230.pdf)
    
    1. 环境搭建与镜像烧写
    2. 通过串口操作开发板

## 1.3

- [K230 SDK基础实验 hello_world](https://github.com/kendryte/k230_docs/blob/main/zh/02_applications/tutorials/K230_%E5%AE%9E%E6%88%98%E5%9F%BA%E7%A1%80%E7%AF%87_hello_world.md)
    - 编译适用于小核linux/大核rt-smart的可执行程序
        - 在拷贝hello、hello.elf至SD卡时，需要首先对SD卡的分区(3)进行取消隐藏\
        下图为使用DiskGenius取消隐藏的示例：\
        <img src="./hello_world/show_hidden.png" width="300">

## 1.4

- [K230视频编解码API参考文档](https://github.com/kendryte/k230_docs/blob/main/zh/01_software/board/mpp/K230_%E8%A7%86%E9%A2%91%E7%BC%96%E8%A7%A3%E7%A0%81_API%E5%8F%82%E8%80%83.md)
    - [视频讲解](https://riscv-edu.cn/course/230/replay/6374)
- [K230 编码实战 - rtsp sever搭建和推流文档](https://github.com/kendryte/k230_docs/blob/main/zh/02_applications/tutorials/K230_%E7%BC%96%E7%A0%81%E5%AE%9E%E6%88%98_rtsp_server%E6%90%AD%E5%BB%BA%E5%92%8C%E6%8E%A8%E6%B5%81.md)
    - [视频讲解](https://riscv-edu.cn/course/230/replay/6375)
- [venc_mapi文档](docs/venc_mapi.md)
    - [视频讲解](https://riscv-edu.cn/course/230/replay/6376)
