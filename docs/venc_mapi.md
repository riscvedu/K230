# K230 VENC MAPI介绍

## 1. 概述

嘉楠发布的K230 AI芯片，提供了媒体处理软件平台MPP，可以支持应用软件的快速开发，通过MPP，应用开发者不需要关注芯片相关的底层处理，直接调用提供的MPI接口即可实现相应功能；但是由于MPI接口太多，调用流程也相对比较繁琐，为了简化用户的使用流程，对MPI接口和功能模块进行了抽象，形成了MAPI接口。对于VENC模块，目前MAPI接口主要在小核上调用，用以获取编码的码流数据。


## 2. 编码数据流图

图1

![mapi encode channel](../test_venc.png)

典型的编码流程包括了输入图像的接收、图像的编码、数据流的跨核传输以及码流的输出等过程。

## 3. API接口

视频编码模块主要提供视频编码通道的创建和销毁、视频编码通道的开启和停止接收图像、设置和获取编码通道属性、注册和解注册获取码流的回调函数等功能。

该功能模块提供以下MAPI：

[kd_mapi_venc_init](#4131-kd_mapi_venc_init)

[kd_mapi_venc_deinit](#4132-kd_mapi_venc_deinit)

[kd_mapi_venc_registercallback](#4133-kd_mapi_venc_registercallback)

[kd_mapi_venc_unregistercallback](#4134-kd_mapi_venc_unregistercallback)

[kd_mapi_venc_start](#4135-kd_mapi_venc_start)

[kd_mapi_venc_stop](#4136-kd_mapi_venc_stop)

[kd_mapi_venc_bind_vi](#4137-kd_mapi_venc_bind_vi)

[kd_mapi_venc_unbind_vi](#4138-kd_mapi_venc_unbind_vi)

[kd_mapi_venc_request_idr](#4139-kd_mapi_venc_request_idr)

[kd_mapi_venc_enable_idr](#41310-kd_mapi_venc_enable_idr)

### 3.1 kd_mapi_venc_init

【描述】

初始化编码通道。

【语法】

k_s32 kd_mapi_venc_init(k_u32 chn_num, [k_venc_chn_attr](#3115-k_venc_chn_attr) \* pst_venc_attr)

【参数】

| 参数名称      | 描述                                        | 输入/输出 |
|---------------|---------------------------------------------|-----------|
| chn_num       | VENC 通道号 取值范围：[0, VENC_MAX_CHN_NUM) | 输入      |
| pst_venc_attr | VENC 通道属性指针             | 输入      |

【返回值】

| 返回值 | 描述               |
|--------|--------------------|
| 0      | 成功               |
| 非0    | 失败，其值为错误码 |

【芯片差异】

无 。

【需求】

头文件：mapi_venc_api.h、k_venc_comm.h
库文件：libmapi.a libipcmsg.a libdatafifo.a

【注意】

- 调用该接口前需要先初始化kd_mapi_sys_init ()和kd_mapi_media_init ()成功，详见“SYS MAPI”章节。
- 重复初始化返回成功

#### 3.1.1 k_venc_chn_attr

【说明】

定义编码通道属性结构体。

【定义】

typedef struct {
&emsp;[k_venc_attr](#3116-k_venc_attr) venc_attr;
&emsp;[k_venc_rc_attr](#3117-k_venc_rc_attr) rc_attr;
} k_venc_chn_attr;

【成员】

| 成员名称  | 描述             |
|-----------|------------------|
| venc_attr | 编码器属性。     |
| rc_attr   | 码率控制器属性。 |

#### 3.1.2 k_venc_attr

【说明】

定义编码器属性结构体。

【定义】

typedef struct {  
&emsp;k_payload_type type;  
&emsp;k_u32 stream_buf_size;  
&emsp;k_u32 stream_buf_cnt;  
&emsp;k_u32 pic_width;  
&emsp;k_u32 pic_height;  
&emsp;k_venc_profile profile;  
} k_venc_attr;

【成员】

| 成员名称 | 描述 |
|---|---|
| type            | 编码协议类型枚。 |
| stream_buf_size | 码流buffer大小。 |
| stream_buf_cnt  | 码流buffer个数。|
| profile         | 编码的等级枚举。|
| pic_width       | 编码图像宽度。 取值范围： `[MIN_WIDTH, MAX_WIDTH]`，以像素为单 位。 必须是MIN_ALIGN的整数倍。   |
| pic_height      | 编码图像高度。 取值范围： `[MIN_HEIGHT, MAX_HEIGHT]`，以像素为 单位。 必须是MIN_ALIGN的整数倍。 |

#### 3.1.3 k_venc_rc_attr

【说明】

定义编码通道码率控制器属性结构体。

【定义】

typedef struct {  
&emsp;[k_venc_rc_mode](#316-k_venc_rc_mode) rc_mode;  
&emsp;union { 

​		[k_venc_cbr](#3118-k_venc_cbr) cbr;  
&emsp;&emsp;[k_venc_vbr](#3119-k_venc_vbr) vbr;  
&emsp;&emsp;[k_venc_fixqp](#3120-k_venc_fixqp) fixqp;  
&emsp;&emsp;[k_venc_mjpeg_fixqp](#3121-k_venc_mjpeg_fixqp) mjpeg_fixqp;  
&emsp;};  
} k_venc_rc_attr;

【成员】

| 成员名称     | 描述                                        |
|--------------|---------------------------------------------|
| rc_mode      | RC模式。                                    |
| cbr          | H.264/H.265协议编码通道固定比特率模式属性。 |
| Vbr          | H.264/H.265协议编码通道可变比特率模式属性。 |
| fixqp        | H.264/H.265协议编码通道固定qp模式属性。     |
| mjpeg_fixqp  | Mjpeg协议编码通道Fixqp模式属性。            |

#### 3.1.4 k_venc_cbr

【说明】

定义H.264/H.265编码通道CBR属性结构体。

【定义】

typedef struct {  
&emsp;k_u32 gop;  
&emsp;k_u32 stats_time;  
&emsp;k_u32 src_frame_rate;  
&emsp;k_u32 dst_frame_rate;  
&emsp;k_u32 bit_rate;  
} k_venc_cbr;

【成员】

| 成员名称        | 描述                                               |
|-----------------|----------------------------------------------------|
| gop             | gop值。                                            |
| stats_time      | CBR码率统计时间，以秒为单位。 取值范围： `[1, 60]`。 |
| src_frame_rate  | 输入帧率，以fps为单位。                            |
| dst_frame_rate  | 编码器输出帧率，以fps为单位。                      |
| bit_rate        | 平均bitrate，以kbps为单位。                        |

【注意事项】

- 如果设置的码率超过芯片手册中规定的最大实时码率，则不能保证实时编码。

#### 3.1.5 k_venc_vbr

【说明】

定义H.264/H.265编码通道VBR属性结构体。

【定义】

typedef struct {  
&emsp;k_u32 gop;  
&emsp;k_u32 stats_time;  
&emsp;k_u32 src_frame_rate;  
&emsp;k_u32 dst_frame_rate;  
&emsp;k_u32 max_bit_rate;  
&emsp;k_u32 bit_rate;  
} k_venc_vbr;

【成员】

| 成员名称        | 描述                                               |
|-----------------|----------------------------------------------------|
| gop             | gop值。                                            |
| stats_time      | VBR码率统计时间，以秒为单位。 取值范围： `[1, 60]`。 |
| src_frame_rate  | 输入帧率，以fps为单位。                            |
| dst_frame_rate  | 编码器输出帧率，以fps为单位。                      |
| max_bit_rate    | 最大bitrate，以kbps为单位。                        |
| bit_rate        | 平均bitrate，以kbps为单位。                        |

#### 3.1.6 k_venc_fixqp

【说明】

定义H.264/H.265编码通道Fixqp属性结构体。

【定义】

typedef struct {  
&emsp;k_u32 gop;  
&emsp;k_u32 src_frame_rate;  
&emsp;k_u32 dst_frame_rate;  
&emsp;k_u32 i_qp; k_u32 p_qp;  
} k_venc_fixqp;

【成员】

| 成员名称        | 描述                                   |
|-----------------|----------------------------------------|
| gop             | gop值。                                |
| src_frame_rate  | 输入帧率，以fps为单位。                |
| dst_frame_rate  | 编码器输出帧率，以fps为单位。          |
| i_qp            | I帧所有宏块Qp值。 取值范围： `[0, 51]`。 |
| q_qp            | P帧所有宏块Qp值。 取值范围： `[0, 51]`。 |

#### 3.1.7 k_venc_mjpeg_fixqp

【说明】

定义MJPEG编码通道Fixqp属性结构体。

【定义】

typedef struct {  
&emsp;k_u32 src_frame_rate;  
&emsp;k_u32 dst_frame_rate;  
&emsp;k_u32 q_factor;  
} k_venc_mjpeg_fixqp;

【成员】

| 成员名称        | 描述                                      |
|-----------------|-------------------------------------------|
| src_frame_rate  | 输入帧率，以fps为单位。                   |
| dst_frame_rate  | 编码器输出帧率，以fps为单位。             |
| q_factor        | MJPEG编码的Qfactor。 取值范围： `[1, 99]`。 |


### 3.2 kd_mapi_venc_deinit

【描述】

编码通道去初始化。

【语法】

k_s32 kd_mapi_venc_deinit(k_u32 chn_num)

【参数】

| 参数名称 | 描述                                        | 输入/输出 |
|----------|---------------------------------------------|-----------|
| chn_num  | VENC 通道号 取值范围：[0, VENC_MAX_CHN_NUM) | 输入      |

【返回值】

| 返回值 | 描述                          |
|--------|-------------------------------|
| 0      | 成功                          |
| 非0    | 失败，其值为[错误码](#5-错误码) |

【芯片差异】

无 。

【需求】

头文件：mapi_venc_api.h、k_venc_comm.h
库文件：libmapi.a libipcmsg.a libdatafifo.a

【注意】

无。

【举例】

无。


### 3.3 kd_mapi_venc_registercallback

【描述】

注册编码通道回调函数，用于编码数据的获取。

【语法】

k_s32 kd_mapi_venc_registercallback(k_u32 chn_num, [kd_venc_callback_s](#4141-kd_venc_callback_s) \*pst_venc_cb);

【参数】

| 参数名称    | 描述                                        | 输入/输出 |
|-------------|---------------------------------------------|-----------|
| chn_num     | VENC 通道号 取值范围：[0, VENC_MAX_CHN_NUM) | 输入      |
| pst_venc_cb | 编码器回调函数结构体指针。                  | 输入      |

【返回值】

| 返回值 | 描述                          |
|--------|-------------------------------|
| 0      | 成功                          |
| 非0    | 失败，其值为[错误码](#5-错误码) |

【芯片差异】

无 。

【需求】

头文件：mapi_venc_api.h、k_venc_comm.h
库文件：libmapi.a libipcmsg.a libdatafifo.a


#### 3.3.1 kd_venc_callback_s

【说明】

编码回调函数结构体。

【定义】

typedef struct  
{  
    pfn_venc_dataproc pfn_data_cb; 
    k_u8 \*p_private_data;  
}kd_venc_callback_s;

【成员】

| 成员名称       | 描述                                                  |
|----------------|-------------------------------------------------------|
| pfn_data_cb    | 回调处理函数。用于获取编码数据                        |
| p_private_data | 私有数据指针。作为参数，在pfn_venc_dataproc中被调用。 |

【注意事项】

- 该结构体被注册后，编码启动后，有编码数据时，pfn_data_cb函数会被调用。用户通过该函数获取编码数据。
- p_private_data为私有数据，用户可选用。

##### 3.3.2 pfn_venc_dataproc

【说明】

定义编码数据回调函数。

【定义】

typedef k_s32 (*pfn_venc_dataproc)(k_u32 chn_num, [kd_venc_data_s](#4144-k_venc_data_pack_s) \*p_vstream_data, k_u8 \*p_private_data);

【成员】

| 成员名称       | 描述         |
|----------------|--------------|
| chn_num        | 编码通道句柄 |
| kd_venc_data_s | 数据指针     |
| p_private_data | 私有数据指针 |

##### 3.3.3 kd_venc_data_s

【说明】

编码后的数据包类型。

【定义】

typedef struct
{
&emsp;[k_venc_chn_status](#3122-k_venc_chn_status) status;
&emsp;k_u32 u32_pack_cnt;
&emsp;[k_venc_data_pack_s](#4144-k_venc_data_pack_s) astPack `[`[KD_VENC_MAX_FRAME_PACKCOUNT](#4145-kd_venc_max_frame_packcount)`]`;
}kd_venc_data_s;

【成员】

| 成员名称     | 描述     |
|--------------|----------|
| status       | 通道状态 |
| u32_pack_cnt | 包的数量 |
| astPack      | 数据包   |

【注意事项】

无。

##### 3.3.4 k_venc_data_pack_s

【说明】

编码后的数据包类型。

【定义】

typedef struct  
{  
&emsp;k_char * vir_addr;  
&emsp;k_u64 phys_addr;  
&emsp;k_u32 len;  
&emsp;k_u64 pts;  
&emsp;k_venc_pack_type type;  
}k_venc_data_pack_s;

【成员】

| 成员名称  | 描述             |
|-----------|------------------|
| vir_addr  | 数据包的虚拟地址 |
| phys_addr | 数据包的物理地址 |
| len       | 数据包的长度     |
| pts       | 时间戳           |
| type      | 码流PACK类型     |


### 3.4 kd_mapi_venc_unregistercallback

【描述】

解注册编码通道回调函数。

【语法】

k_s32 kd_mapi_venc_unregistercallback(k_u32 chn_num, [kd_venc_callback_s](#4141-kd_venc_callback_s) \*pst_venc_cb);

【参数】

| 参数名称    | 描述                                        | 输入/输出 |
|-------------|---------------------------------------------|-----------|
| chn_num     | VENC 通道号 取值范围：[0, VENC_MAX_CHN_NUM) | 输入      |
| pst_venc_cb | 编码器回调函数结构体指针。                  | 输入      |

【返回值】

| 返回值 | 描述                          |
|--------|-------------------------------|
| 0      | 成功                          |
| 非0    | 失败，其值为[错误码](#5-错误码) |

【芯片差异】

无 。

【需求】

头文件：mapi_venc_api.h、k_venc_comm.h

库文件：libmapi.a libipcmsg.a libdatafifo.a

【注意】

pst_venc_cb回调函数结构体内容与注册时的结构体内容完全一致时才能解注册该回调函数。

【举例】

无


### 3.5 kd_mapi_venc_start

【描述】

启动编码通道。

【语法】

k_s32 kd_mapi_venc_start(k_s32 chn_num ,k_s32 s32_frame_cnt);

【参数】

| 参数名称      | 描述                                        | 输入/输出 |
|---------------|---------------------------------------------|-----------|
| chn_num       | VENC 通道号 取值范围：[0, VENC_MAX_CHN_NUM) | 输入      |
| s32_frame_cnt | 期望编码帧的个数                            | 输入      |

【返回值】

| 返回值 | 描述                          |
|--------|-------------------------------|
| 0      | 成功                          |
| 非0    | 失败，其值为[错误码](#5-错误码) |

【芯片差异】

无 。

【需求】

头文件：mapi_venc_api.h、k_venc_comm.h
库文件：libmapi.a libipcmsg.a libdatafifo.a


【举例】

无。


### 3.6 kd_mapi_venc_stop

【描述】

停止编码通道。

【语法】

k_s32 kd_mapi_venc_stop(k_s32 chn_num);

【参数】

| 参数名称 | 描述                                        | 输入/输出 |
|----------|---------------------------------------------|-----------|
| chn_num  | VENC 通道号 取值范围：[0, VENC_MAX_CHN_NUM) | 输入      |

【返回值】

| 返回值 | 描述                          |
|--------|-------------------------------|
| 0      | 成功                          |
| 非0    | 失败，其值为[错误码](#5-错误码) |

【芯片差异】

无。

【需求】

头文件：mapi_venc_api.h、k_venc_comm.h
库文件：libmapi.a libipcmsg.a libdatafifo.a

【注意】

无。

【举例】

无。


### 3.7 kd_mapi_venc_bind_vi

【描述】

编码通道绑定输入源VI

【语法】

k_s32 kd_mapi_venc_bind_vi(k_s32 src_dev, k_s32 src_chn,k_s32 chn_num);

【参数】

| 参数名称 | 描述                                        | 输入/输出 |
|----------|---------------------------------------------|-----------|
| src_dev  | 输入源Device ID                             | 输入      |
| src_chn  | 输入源Channel ID                            | 输入      |
| chn_num  | VENC 通道号 取值范围：[0, VENC_MAX_CHN_NUM) | 输入      |

【返回值】

| 返回值 | 描述                          |
|--------|-------------------------------|
| 0      | 成功                          |
| 非0    | 失败，其值为[错误码](#5-错误码) |

【芯片差异】

无。

【需求】

头文件：mapi_venc_api.h、k_venc_comm.h
库文件：libmapi.a libipcmsg.a libdatafifo.a

【注意】

无。

【举例】

无。


### 3.8 kd_mapi_venc_unbind_vi

【描述】

解绑定编码通道的输入源VI。

【语法】

k_s32 kd_mapi_venc_unbind_vi(k_s32 src_dev, k_s32 src_chn, k_s32 chn_num);

【参数】

| 参数名称 | 描述                                        | 输入/输出 |
|----------|---------------------------------------------|-----------|
| src_dev  | 输入源Device ID                             | 输入      |
| src_chn  | 输入源Channel ID                            | 输入      |
| chn_num  | VENC 通道号 取值范围：[0, VENC_MAX_CHN_NUM) | 输入      |

【返回值】

| 返回值 | 描述                          |
|--------|-------------------------------|
| 0      | 成功                          |
| 非0    | 失败，其值为[错误码](#5-错误码) |

【芯片差异】

无。

【需求】

头文件：mapi_venc_api.h、k_venc_comm.h
库文件：libmapi.a libipcmsg.a libdatafifo.a

【注意】

无。

【举例】

无。


### 3.9 kd_mapi_venc_request_idr

【描述】

请求IDR帧，在调用之后立即产生一个IDR帧。

【语法】

k_s32 kd_mapi_venc_request_idr(k_s32 chn_num);

【参数】

| 参数名称 | 描述                                        | 输入/输出 |
|----------|---------------------------------------------|-----------|
| chn_num  | VENC 通道号 取值范围：`[0, VENC_MAX_CHN_NUM]`   | 输入      |

【返回值】

| 返回值 | 描述                          |
|--------|-------------------------------|
| 0      | 成功                          |
| 非0    | 失败，其值为[错误码](#5-错误码) |

【芯片差异】

无。

【需求】

头文件：mapi_venc_api.h、k_venc_comm.h
库文件：libmapi.a libipcmsg.a libdatafifo.a

【注意】

无。

【举例】

无。


### 3.10 kd_mapi_venc_enable_idr

【描述】

使能IDR帧，根据GOP间隔产生IDR帧。

【语法】

k_s32 kd_mapi_venc_enable_idr(k_s32 chn_num, k_bool idr_enable);

【参数】

| 参数名称 | 描述                                        | 输入/输出 |
|----------|---------------------------------------------|-----------|
| chn_num  | VENC 通道号 取值范围：`[0, VENC_MAX_CHN_NUM]`   | 输入      |
| idr_enable  | 是否是能IDR帧，0:不使能 1:使能              | 输入      |

【返回值】

| 返回值 | 描述                          |
|--------|-------------------------------|
| 0      | 成功                          |
| 非0    | 失败，其值为[错误码](#5-错误码) |

【芯片差异】

无。

【需求】

头文件：mapi_venc_api.h、k_venc_comm.h
库文件：libmapi.a libipcmsg.a libdatafifo.a

【注意】

- 该接口应在[kd_mpi_venc_create_chn](#211-kd_mpi_venc_create_chn)之后，[kd_mpi_venc_start_chn](#213-kd_mpi_venc_start_chn)之前调用。

【举例】

无。


## 4. 错误码

编码 API 错误码
| 错误代码   | 宏定义                   | 描述                                         |
|------------|--------------------------|----------------------------------------------|
| 0xa0098001 | K_ERR_VENC_INVALID_DEVID | 设备ID超出合法范围                           |
| 0xa0098002 | K_ERR_VENC_INVALID_CHNID | 通道ID超出合法范围                           |
| 0xa0098003 | K_ERR_VENC_ILLEGAL_PARAM | 参数超出合法范围                             |
| 0xa0098004 | K_ERR_VENC_EXIST         | 试图申请或者创建已经存在的设备、通道或者资源 |
| 0xa0098005 | K_ERR_VENC_UNEXIST       | 试图使用或者销毁不存在的设备、通道或者资源   |
| 0xa0098006 | K_ERR_VENC_NULL_PTR      | 函数参数中有空指针                           |
| 0xa0098007 | K_ERR_VENC_NOT_CONFIG    | 使用前未配置                                 |
| 0xa0098008 | K_ERR_VENC_NOT_SUPPORT   | 不支持的参数或者功能                         |
| 0xa0098009 | K_ERR_VENC_NOT_PERM      | 该操作不允许，如试图修改静态配置参数         |
| 0xa009800c | K_ERR_VENC_NOMEM         | 分配内存失败，如系统内存不足                 |
| 0xa009800d | K_ERR_VENC_NOBUF         | 分配缓存失败，如申请的数据缓冲区太大         |
| 0xa009800e | K_ERR_VENC_BUF_EMPTY     | 缓冲区中无数据                               |
| 0xa009800f | K_ERR_VENC_BUF_FULL      | 缓冲区中数据满                               |
| 0xa0098010 | K_ERR_VENC_NOTREADY      | 系统没有初始化或没有加载相应模块             |
| 0xa0098011 | K_ERR_VENC_BADADDR       | 地址超出合法范围                             |
| 0xa0098012 | K_ERR_VENC_BUSY          | VENC系统忙                                   |
                                  |

## 5. 调试信息

多媒体内存管理和和系统绑定调试信息，请参考《K230 系统控制 API参考》。

## 6. sample
```
#include <stdio.h>
#include <sys/mman.h>
#include <unistd.h>
#include <fcntl.h>
#include <string.h>
#include <pthread.h>
#include <stdlib.h>
#include <getopt.h>
#include <signal.h>
#include "mapi_sys_api.h"
#include "mapi_vvi_api.h"
#include "mapi_venc_api.h"
#include "mapi_vicap_api.h"
#include "k_vicap_comm.h"

#define MAX_CHN_NUM 3
#define VI_ALIGN_UP(addr, size) (((addr)+((size)-1U))&(~((size)-1U)))

static k_u8 g_exit = 0;
static FILE *fp[MAX_CHN_NUM] = {NULL};
static int stream_count[MAX_CHN_NUM] = {0};
static int is_jpeg_type[MAX_CHN_NUM] = {0};
static char jpeg_out_path[128];

static void sig_handler(int sig) {
    g_exit = 1;
}

k_s32 get_venc_stream(k_u32 chn_num, kd_venc_data_s* p_vstream_data, k_u8 *p_private_data)
{
    int cut = p_vstream_data->status.cur_packs;
    int i = 0;
    k_char * pdata  = NULL;
    
    if (!is_jpeg_type[chn_num]) {
        for(i = 0;i < cut;i++) {
            pdata =  p_vstream_data->astPack[i].vir_addr;
            if(stream_count[chn_num] % 90 == 0)
                printf("recv count:%d, chn:%d, phys_addr:0x%lx, len:%d, data:[%02x %02x %02x %02x %02x]\n",
                stream_count[chn_num], chn_num, p_vstream_data->astPack[i].phys_addr, p_vstream_data->astPack[i].len,pdata[0],pdata[1],pdata[2],pdata[3],pdata[4]);

            if (stream_count[chn_num] <= 1000) {
                fwrite(pdata, 1, p_vstream_data->astPack[i].len, fp[chn_num]);
                fflush(fp[chn_num]);
            }
        }
    } else {
        if (stream_count[chn_num] <= 10) {
            char jpg_name[128];
            snprintf(jpg_name, 128, "%s/chn%d_%d.jpg", jpeg_out_path, chn_num, stream_count[chn_num]);
            FILE *fp_jpg = fopen(jpg_name, "wb");
            for(i = 0; i < cut; i++) {
                pdata = p_vstream_data->astPack[i].vir_addr;
                printf("recv count:%d, chn:%d, phys_addr:0x%lx, len:%d, data:[%02x %02x %02x %02x %02x]\n",
                    stream_count[chn_num], chn_num, p_vstream_data->astPack[i].phys_addr, p_vstream_data->astPack[i].len,pdata[0],pdata[1],pdata[2],pdata[3],pdata[4]);

                fwrite(pdata, 1, p_vstream_data->astPack[i].len, fp_jpg);
                fflush(fp_jpg);
            }
            fclose(fp_jpg);
        }
    }
    stream_count[chn_num]++;
}

typedef struct {
    k_vicap_sensor_type sensor_type;
    k_payload_type type;
    k_u32 chn_num;
    char out_path[128];
} SampleCtx;

struct option long_options[] = {
    {"sensor_type", required_argument, NULL, 's'},
    {"chn_num", required_argument, NULL, 'n'},
    {"type", required_argument, NULL, 't'},
    {"out_path", required_argument, NULL, 'o'},
    {"help", required_argument, NULL, 'h'},
    {NULL, 0, NULL, 0},
};

static k_payload_type get_venc_type(k_u32 vtype_index) {
    switch (vtype_index) {
        case 0:
            return K_PT_H264;
        case 1:
            return K_PT_H265;
        case 2:
            return K_PT_JPEG;
        default:
            printf("encoder type %d is unsupported, use K_PT_H265 default.\n", vtype_index);
            return K_PT_H265;
    }
}

k_u32 parse_option(int argc, char *argv[], SampleCtx *psample_ctx) {
    if (argc > 1) {
        int c;
        int option_index = 0;
        while ((c = getopt_long(argc, argv, "s:n:t:o:h", long_options, &option_index)) != -1) {
            switch (c) {
                case 's': {
                    psample_ctx->sensor_type = (k_vicap_sensor_type)atoi(optarg);
                    printf("sensor type: %d.\n", psample_ctx->sensor_type);
                    break;
                }
                case 'n': {
                    psample_ctx->chn_num = atoi(optarg);
                    if (psample_ctx->chn_num < 1 || psample_ctx->chn_num > MAX_CHN_NUM) {
                        printf("sample not support chn_num > 3 or chn_num < 1, please check by %s -h \n", argv[0]);
                        return -1;
                    }
                    printf("chn_num: %d.\n", psample_ctx->chn_num);
                    break;
                }
                case 't': {
                    psample_ctx->type = get_venc_type(atoi(optarg));
                    printf("encoder payload type: %d.\n", psample_ctx->type);
                    break;
                }
                case 'o': {
                    if (access(optarg, F_OK) != 0) {
                        printf("output path %s is not exist.\n", optarg);
                        return -1;
                    }
                    strcpy(psample_ctx->out_path, optarg);
                    printf("output path: %s.\n", psample_ctx->out_path);
                    break;
                }
                case 'h': {
                    printf("Usage: %s -s 0 -n 2 -t 0\n", argv[0]);
                    printf("          -s or --sensor_type [sensor_index],\n");
                    printf("                                see vicap doc\n");
                    printf("          -n or --chn_num [number], 1, 2, 3.\n");
                    printf("          -t or --type [type_index],\n");
                    printf("                        0: h264 type\n");
                    printf("                        1: h265 type\n");
                    printf("                        2: jpeg type\n");
                    printf("          -o or --out_path [output_path].\n");
                    printf("          -h or --help, will print usage.\n");
                    return 1;
                }
                default: {
                    printf("Invalid option, please check by %s -h\n", argv[0]);
                    return -1;
                }
            }
        }
    }
    return K_SUCCESS;
}

int main(int argc, char *argv[]) {

    signal(SIGINT, sig_handler);

    SampleCtx sample_context = {
        .sensor_type = IMX335_MIPI_2LANE_RAW12_1920X1080_30FPS_LINEAR,
        .type = K_PT_H264,
        .chn_num = 1,
        .out_path = "/tmp/"
    };

    k_u32 ret = parse_option(argc, argv, &sample_context);
    if (ret != K_SUCCESS) {
        if (ret > 0)
            return 0;

        printf("parse_option failed.\n");
        return -1;
    }

    strcpy(jpeg_out_path, sample_context.out_path);

    printf("mapi sample_venc...\n");

    ret = kd_mapi_sys_init();
    if (ret != K_SUCCESS) {
        printf("kd_mapi_sys_init failed, %x.\n", ret);
        return -1;
    }

    // vicap init
    k_vicap_sensor_info sensor_info;
    memset(&sensor_info, 0, sizeof(sensor_info));
    sensor_info.sensor_type = sample_context.sensor_type;
    ret = kd_mapi_vicap_get_sensor_info(&sensor_info);
    if (ret != K_SUCCESS) {
        printf("kd_mapi_vicap_get_sensor_info failed, %x.\n", ret);
        goto venc_deinit;
    }

    k_vicap_dev_set_info dev_attr_info;
    memset(&dev_attr_info, 0, sizeof(dev_attr_info));
    if (sample_context.sensor_type <= OV_OV9286_MIPI_1280X720_60FPS_10BIT_LINEAR_IR_SPECKLE || sample_context.sensor_type >= SC_SC035HGS_MIPI_1LANE_RAW10_640X480_120FPS_LINEAR)
        dev_attr_info.dw_en = K_FALSE;
    else
        dev_attr_info.dw_en = K_TRUE;

    k_u32 pic_width[MAX_CHN_NUM] = {1280, 960, 640 };
    k_u32 pic_height[MAX_CHN_NUM] = {720, 640, 480};
    k_mapi_media_attr_t media_attr;
    memset(&media_attr, 0, sizeof(media_attr));
    media_attr.media_config.vb_config.max_pool_cnt = sample_context.chn_num * 2 + 1;
    for (int i = 0; i < sample_context.chn_num; i++) {
        k_u32 pic_size = pic_width[i] * pic_height[i] * 3 / 2;
        k_u32 stream_size = pic_width[i] * pic_height[i] * 3 / 4;
        media_attr.media_config.vb_config.comm_pool[i * 2].blk_cnt = 6;
        media_attr.media_config.vb_config.comm_pool[i * 2].blk_size = ((pic_size + 0xfff) & ~0xfff);
        media_attr.media_config.vb_config.comm_pool[i * 2].mode = VB_REMAP_MODE_NOCACHE;
        media_attr.media_config.vb_config.comm_pool[i * 2 + 1].blk_cnt = 30;
        media_attr.media_config.vb_config.comm_pool[i * 2 + 1].blk_size = ((stream_size + 0xfff) & ~0xfff);
        media_attr.media_config.vb_config.comm_pool[i * 2 + 1].mode = VB_REMAP_MODE_NOCACHE;
    }

    if (dev_attr_info.dw_en) {
        media_attr.media_config.vb_config.comm_pool[sample_context.chn_num * 2].blk_cnt = 6;
        media_attr.media_config.vb_config.comm_pool[sample_context.chn_num * 2].blk_size = VI_ALIGN_UP(sensor_info.width * sensor_info.height * 3 / 2, 0x400);
        media_attr.media_config.vb_config.comm_pool[sample_context.chn_num * 2].mode = VB_REMAP_MODE_NOCACHE;
    }

    memset(&media_attr.media_config.vb_supp.supplement_config, 0, sizeof(media_attr.media_config.vb_supp.supplement_config));
    media_attr.media_config.vb_supp.supplement_config |= VB_SUPPLEMENT_JPEG_MASK;

    ret = kd_mapi_media_init(&media_attr);
    if (ret != K_SUCCESS) {
        printf("kd_mapi_media_init failed, %x.\n", ret);
        goto sys_deinit;
    }

    dev_attr_info.pipe_ctrl.data = 0xFFFFFFFF;
    dev_attr_info.sensor_type = sample_context.sensor_type;
    dev_attr_info.vicap_dev = VICAP_DEV_ID_0;
    ret = kd_mapi_vicap_set_dev_attr(dev_attr_info);
    if (ret != K_SUCCESS) {
        printf("kd_mapi_vicap_set_dev_attr failed, %x.\n", ret);
        goto sys_deinit;
    }

    k_venc_chn_attr venc_chn_attr;
    memset(&venc_chn_attr, 0, sizeof(venc_chn_attr));
    venc_chn_attr.rc_attr.rc_mode = K_VENC_RC_MODE_CBR;
    venc_chn_attr.rc_attr.cbr.src_frame_rate = 30;
    venc_chn_attr.rc_attr.cbr.dst_frame_rate = 30;
    venc_chn_attr.rc_attr.cbr.bit_rate = 4000;
    venc_chn_attr.venc_attr.type = sample_context.type;
    if (sample_context.type == K_PT_H264) {
        venc_chn_attr.venc_attr.profile = VENC_PROFILE_H264_HIGH;
    } else if (sample_context.type == K_PT_H265) {
        venc_chn_attr.venc_attr.profile = VENC_PROFILE_H265_MAIN;
    } else if (sample_context.type == K_PT_JPEG) {
        venc_chn_attr.rc_attr.rc_mode = K_VENC_RC_MODE_FIXQP;
        venc_chn_attr.rc_attr.mjpeg_fixqp.src_frame_rate = 30;
        venc_chn_attr.rc_attr.mjpeg_fixqp.dst_frame_rate = 30;
        venc_chn_attr.rc_attr.mjpeg_fixqp.q_factor = 45;
    }

    for (int venc_idx = 0; venc_idx < sample_context.chn_num; venc_idx++) {
        char filename[128];
        if (sample_context.type == K_PT_H264) {
            snprintf(filename, 128, "%s/stream_chn%d.264", sample_context.out_path, venc_idx);
        } else if (sample_context.type == K_PT_H265) {
            snprintf(filename, 128, "%s/stream_chn%d.265", sample_context.out_path, venc_idx);
        } else if (sample_context.type == K_PT_JPEG) {
            is_jpeg_type[venc_idx] = 1;
        }
        if (sample_context.type != K_PT_JPEG) {
            fp[venc_idx] = fopen(filename, "wb");
            if (fp[venc_idx] == NULL) {
                printf("stream file %s open failed.\n", filename);
                goto media_deinit;
            }
        }

        k_u32 stream_size = pic_width[venc_idx] * pic_height[venc_idx] * 3 / 4;
        venc_chn_attr.venc_attr.pic_width = pic_width[venc_idx];
        venc_chn_attr.venc_attr.pic_height = pic_height[venc_idx];
        venc_chn_attr.venc_attr.stream_buf_cnt = 30;
        venc_chn_attr.venc_attr.stream_buf_size = ((stream_size + 0xfff) & ~0xfff);
        ret = kd_mapi_venc_init(venc_idx, &venc_chn_attr);
        if (ret != K_SUCCESS) {
            printf("init venc %d failed, %x.\n", venc_idx, ret);
            for (int create_idx = 0; create_idx < venc_idx; create_idx++) {
                ret = kd_mapi_venc_deinit(create_idx);
                if (ret != K_SUCCESS) {
                    printf("deinit venc %d failed, %x.\n", create_idx, ret);
                    return -1;
                }
            }
            goto sys_deinit;
        }
    }

    for (int venc_idx = 0; venc_idx < sample_context.chn_num; venc_idx++) {
        int user_data = venc_idx;
        kd_venc_callback_s venc_cb;
        venc_cb.pfn_data_cb = get_venc_stream;
        venc_cb.p_private_data = (k_u8 *)&user_data;
        kd_mapi_venc_registercallback(venc_idx, &venc_cb);
    }

    for (int vichn_idx = 0; vichn_idx < sample_context.chn_num; vichn_idx++) {
        k_vicap_chn_set_info vi_chn_attr_info;
        memset(&vi_chn_attr_info, 0, sizeof(vi_chn_attr_info));

        vi_chn_attr_info.crop_en = K_FALSE;
        vi_chn_attr_info.scale_en = K_FALSE;
        vi_chn_attr_info.chn_en = K_TRUE;
        vi_chn_attr_info.crop_h_start = 0;
        vi_chn_attr_info.crop_v_start = 0;
        vi_chn_attr_info.out_width = pic_width[vichn_idx];
        vi_chn_attr_info.out_height = pic_height[vichn_idx];
        vi_chn_attr_info.pixel_format = PIXEL_FORMAT_YUV_SEMIPLANAR_420;
        vi_chn_attr_info.vicap_dev = VICAP_DEV_ID_0;
        vi_chn_attr_info.vicap_chn = (k_vicap_chn)vichn_idx;
        vi_chn_attr_info.alignment = 12;
        if (!dev_attr_info.dw_en)
            vi_chn_attr_info.buf_size = VI_ALIGN_UP(pic_width[vichn_idx] * pic_height[vichn_idx] * 3 / 2, 0x400);
        else
            vi_chn_attr_info.buf_size = VI_ALIGN_UP(sensor_info.width * sensor_info.height * 3 / 2, 0x400);
        ret = kd_mapi_vicap_set_chn_attr(vi_chn_attr_info);
        if (ret != K_SUCCESS) {
            printf("vicap chn %d set attr failed, %x.\n", vichn_idx, ret);
            goto venc_deinit;
        }
    }

    // chn start
    for (int venc_idx = 0; venc_idx < sample_context.chn_num; venc_idx++) {
        ret = kd_mapi_venc_start(venc_idx, -1);
        if (ret != K_SUCCESS) {
            printf("venc chn %d start failed, %x.\n", venc_idx, ret);
            for (int create_idx = 0; create_idx < venc_idx; create_idx++) {
                ret = kd_mapi_venc_stop(create_idx);
                if (ret != K_SUCCESS) {
                    printf("venc chn %d stop failed, %x.\n", create_idx, ret);
                    return -1;
                }
            }
            goto venc_deinit;
        }
    }

    k_s32 src_dev = VICAP_DEV_ID_0;
    for (int venc_idx = 0; venc_idx < sample_context.chn_num; venc_idx++) {
        k_s32 src_chn = venc_idx;
        ret = kd_mapi_venc_bind_vi(src_dev, src_chn, venc_idx);
        if (ret != K_SUCCESS) {
            printf("venc chn %d bind vi failed, %x.\n", venc_idx, ret);
            for (int idx = 0; idx < venc_idx; idx++) {
                ret = kd_mapi_venc_unbind_vi(src_dev, idx, idx);
                if (ret != K_SUCCESS) {
                    printf("venc idx %d unbind vi failed, %x.\n", idx, ret);
                    return -1;
                }
            }
            goto venc_stop;
        }
    }

    ret = kd_mapi_vicap_start(VICAP_DEV_ID_0);
    if (ret != K_SUCCESS) {
        printf("kd_mapi_vicap_start failed, %x.\n", ret);
        goto vicap_stop;
    }

    // waiting thread
    while (!g_exit) {
        usleep(50000);
    }

    for (int idx = 0; idx < sample_context.chn_num; idx++) {
        ret = kd_mapi_venc_unbind_vi(src_dev, idx, idx);
        if (ret != K_SUCCESS) {
            printf("venc chn %d unbind vi failed, %x.\n", idx, ret);
            return -1;
        }
    }

vicap_stop:
    ret = kd_mapi_vicap_stop(VICAP_DEV_ID_0);
    if (ret != K_SUCCESS) {
        printf("kd_mapi_vicap_stop failed, %x.\n", ret);
        return -1;
    }

venc_stop:
    for (int venc_idx = 0; venc_idx < sample_context.chn_num; venc_idx++) {
        ret = kd_mapi_venc_stop(venc_idx);
        if (ret != K_SUCCESS) {
            printf("venc chn %d stop failed, %x.\n", venc_idx, ret);
            return -1;
        }
    }

venc_deinit:
    for (int venc_idx = 0; venc_idx < sample_context.chn_num; venc_idx++) {
        ret = kd_mapi_venc_deinit(venc_idx);
        if (ret != K_SUCCESS) {
            printf("venc %d deinit failed, %x.\n", venc_idx, ret);
            return -1;
        }
    }

media_deinit:
    ret = kd_mapi_media_deinit();
    if (ret != K_SUCCESS) {
        printf("kd_mapi_media_deinit failed, %x.\n", ret);
        return -1;
    }

    for (int venc_idx = 0; venc_idx < sample_context.chn_num; venc_idx++) {
        if (fp[venc_idx]) {
            fclose(fp[venc_idx]);
            fp[venc_idx] = NULL;
        }
    }

sys_deinit:
    ret = kd_mapi_sys_deinit();
    if (ret != K_SUCCESS) {
        printf("kd_mapi_sys_deinit failed, %x.\n", ret);
        return -1;
    }

    printf("mapi sample_venc end...\n");
    return 0;
}
```