# 开发环境搭建

- [配置文档](./docs/从零开始玩转K230.pdf)
- [课程回放](https://riscv-edu.cn/course/230/replay/6364)
    
## 环境搭建与镜像烧写

- 编译K230 SDK

    1. 安装必要软件
    ```bash
    sudo apt-get install git docker.io net-tools openssh-server make curl -y
    ```

    2. 下载SDK
    ```bash
    git clone https://github.com/kendryte/k230_sdk.git
    ```

    3. 拉取docker镜像
    ```bash
    docker pull ghcr.io/kendryte/k230_sdk
    docker images | grep k230_sdk # 确认拉取成功
    ```
    > 说明： docker镜像中默认不包含toolchain，下载源码后，使用命令'make prepare_sourcecode'命令会自动下载toolchain至当前编译目录中。

    4. （可选）配置原生Linux进行编译
    根据[Dockerfile](./docs/Dockerfile)配置编译环境

    5. 编译SDK
    ```bash
    cd k230_sdk
    make prepare_sourcecode
    ```
    > make prepare_sourcecode 会自动下载Linux和RT-Smart toolchain, buildroot package, AI package等. 请确保该命令执行成功并没有Error产生，下载时间和速度以实际网速为准。

    确认当前目录为k230_sdk源码根目录，
    使用如下命令进入docker
    ```bash
    docker run -u root -it -v $(pwd):$(pwd) -v $(pwd)/toolchain:/opt/toolchain -w $(pwd) ghcr.io/kendryte/k230_sdk /bin/bash
    ```
    根据不同开发板或软件功能，选择不同的配置config进行编译\
    编译命令格式：make CONF=xxx，如：\
    编译K230-USIP-LP3-EVB板子镜像，执行make CONF=k230_evb_defconfig 命令开始编译\
    编译CanMV-K230板子的镜像，执行 make CONF=k230_canmv_defconfig 命令开始编译
    ```bash
    make CONF=k230_canmv_defconfig
    ```
    编译结束后会得到一个镜像文件\

    6. 烧录镜像文件

    - 将已插入TF卡的U盘插入电脑
    
    如使用Linux烧录TF卡,需要先确认TF卡在系统中的名称/dev/sdx, 并替换如下命令中的/dev/sdx
    ```bash
    sudo dd if=sysimage-sdcard.img of=/dev/sdx bs=1M oflag=sync
    ```
    如使用Windows烧录, 建议使用[the balena Etcher](https://etcher.balena.io/)工具

## 通过串口操作开发板

将烧录完成的TF卡插入开发板TF卡槽中
系统上电后，默认会有两个串口设备，可分别用于访问小核Linux和大核RTSmart

可以使用例如MobaXterm的工具进行连接

小核Linux默认用户名**root**，密码为空。大核RTSmart系统中开机会自动启动一个应用程序，可按**q**键退出至命令提示符终端。

