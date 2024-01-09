# K230 GUI实战 - LVGL移植教程
    
- [官方文档](https://github.com/kendryte/k230_docs/blob/main/zh/02_applications/tutorials/K230_GUI%E5%AE%9E%E6%88%98_LVGL%E7%A7%BB%E6%A4%8D%E6%95%99%E7%A8%8B.md)
- [视频讲解](https://riscv-edu.cn/course/230/replay/6381)

## 概述

k230使用DRM作为显示驱动，DRM(Direct Rendering Manager) 是 Linux 内核中的一个子系统，相比过时的Framebuffer 可以支持复杂的 GPU 操作，如硬件加速的图形渲染。lvgl可以基于libdrm提供的接口进行GUI的绘制。

SDK中已经移植好lvgl组件，且默认编译了一个可以运行的demo，位于/usr/bin/lvgl_demo_widgets。开机后在小核linux终端输入命令lvgl_demo_widgets回车即可体验。

### 实验步骤

1. linux内核中DRM驱动加载成功后会出现以下节点/dev/dri/card0
在小核侧，执行
```bash
cd /dev/dri
ls | grep card0
```

#### DRM驱动源码介绍

lvgl的drm驱动程序位于src/little/buildroot-ext/package/lvgl/port_src/lv_drivers/display/drm.c。 对DRM的操作需要先open该文件节点。
lvgl使用中主要用到以下几个函数：
- drm_init() drm初始化。
- drm_get_sizes() 获取显示器分辨率信息。
- drm_flush() 显示绘制回调接口。

- 该drm驱动默认开启了双buffer，但目前在没有硬件加速的情况下，双缓冲刷新率低，可以注释掉下面代码以单缓冲方式刷新，显示效果会更好：
```bash
void drm_flush(lv_disp_drv_t *disp_drv, const lv_area_t *area, lv_color_t *color_p)
{
    // 注释以下代码 
    ...
    if (!drm_dev.cur_bufs[0])
        drm_dev.cur_bufs[1] = &drm_dev.drm_bufs[1];
    else
        drm_dev.cur_bufs[1] = drm_dev.cur_bufs[0];
        drm_dev.cur_bufs[0] = fbuf;
    ...
}
```

#### lvgl的使用

lvgl的主程序文件位于src/little/buildroot-ext/package/lvgl/port_src/main.c

lvgl配置文件位于：src/little/buildroot-ext/package/lvgl/port_src/lv_conf.h

##### 实验步骤

1. 编译lvgl_demo

在k230_sdk根目录，执行：
```bash
make # 默认编译全部
```

2. 拷贝至SD卡

3. 开发板上电启动
运行lvgl_demo_widgets
```bash
./lvgl_demo_widgets
```
