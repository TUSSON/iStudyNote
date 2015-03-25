# 高通平台Audio调试 (8916平台）  -- 杜松 March 24, 2015

## 源码目录

##### 1. driver

- kernel/sound/soc/codecs/msm8x16-wcd.c ---- CODEC的驱动
- kernel/sound/soc/codecs/wcd-mbhc-v2.c ---- 耳机插入，耳机按键等
- kernel/sound/soc/msm/msm8x16.c ---- 平台驱动，资源整合，前后端DAI连接
- kernel/sound/soc/msm/qdsp6v2/ ---- dsp音效处理

##### 2. TinyALSA

- external/tinyalsa/ ---- ALSA架构应用层接口

##### 3. Audio Route

- system/media/audio_route/ ---- mixer_path.xml文件的解析和使用

##### 4. Audio HAL

- hardware/qcom/audio/hal/ ---- 通过tinyalsa向驱动交互，为audio flinger服务

##### 5. Audio Policy HAL

- hardware/qcom/audio/policy_hal/ ---- 音频策略定制
- hardware/libhardware_legacy/audio/ ---- 音频策略默认方案

## 驱动设备文件

- /dev/snd/ ALSA架构和上层通信的设备节点
- /proc/asound/ 声卡的一些信息
- /sys/class/sound/ 声卡的各种设备、属性

## 调试工具

- system/bin/tinymix 临时进行各种control接口配置
- system/bin/tinycap 可以录音
- system/bin/tinypcminfo 查看PCM设备的配置，频率、通道、格式等
- system/bin/tinyplay 可以播放WAV格式音乐

## 配置文件

##### 1. mixer path

- 源码目录： device/qcom/msm8916_32/mixer_paths_mtp.xml
- 手机目录： /etc/mixer_paths_mtp.xml

这个文件有两个用途：

1. 各种control的初始值设定，比如一些*AD，DA增益*，*开关*，*通道* 等
2. 音频回路的配置，比如speaker打开时，因该打开哪些开关

下面是根据pm8916的codec驱动绘制的**音频回路配置图**, 括号都是开关，打开不同的开关就连通不同的回路
例如最简单的speaker回路打开, 打开如下两个开关，就连通整个播放回路了

```
    mixer_path配置：
    <path name="speaker">
        <ctl name="RX3 MIX1 INP1" value="RX1" />
        <ctl name="SPK DAC Switch" value="1" />
    </path>
```

```
    连通的播放回路：
    SPK_OUT <- SPK PA <- SPK DAC <- RX3 CHAIN <- RX3 MIX1 INP1 <- I2S RX1 <- RX_I2S_CLK <- CDC_CONN
```

```
Playback:

EAR <-- EAR_S <-- (Switch) EAR PA <----------------
                               ^               ^  ^
                |--|- CP       |-- EAR CP      |  |
                |--|- RX_BIAS  |-- RX_BIAS     |  |
                |  v                           |  |
HEADPHONE <--- HPHL PA <--- HPHL <-- (Switch) HPHL DAC <------------------------
           \    |                                 |                            |
            \   v                                 |                            v
             \- HPHR PA <-- HPHR <-- (Switch) HPHR DAC <-- RDAC2 MUX <-- (RX1) RX1 CHAIN
                                                                      \- (RX2) RX2 CHAIN

SPK_OUT <-- SPK PA <- SPK DAC <-------------------------------------- (Switch) RX3 CHAIN
                               \-- SPK_RX_BIAS

               /- RX1,2,3 CLK
RX1,2,3 CHAIN <-- RX1,2,3 MIX1,2 <-- RX1,2,3 MIX1,2 INP1,2,3 <-- (RX1,2,3) I2S RX1,2,3 <-- RX_I2S_CLK <-- CDC_CONN
                                                              \- (IIR1) IIR1 <-- IIR1 INP1 MUX <-- (DEC1) DEC1 MUX
                                                                                                \- (DEC2) DEC2 MUX
```

```
Record:

                       /- (DMIC1) DMIC1
                      /-- (DMIC2) DMIC2
I2S TX1 <-- DEC1 MUX <--- (ADC1) ADC1 <------------------------------------ AMIC1
I2S TX2 <-- DEC2 MUX <--- (ADC2) ADC2 <-- ADC2 MUX <-- (INP2) ADC2_INP2 <-- AMIC2
      ^               \-- (ADC3) ADC3 /             \- (INP3) ADC2_INP3 <-- AMIC3
      |                \- CDC_CONN
      |
      |------------------ TX_I2S_CLK

```

##### 2. audio policy

- 源码目录： device/qcom/msm8916_32/audio_policy.conf
- 手机目录： /etc/audio_policy.conf

这个是配置硬件支持哪些音频设备，每种设备支持的频率、通道、格式等。
音频设备的选择就是根据系统支持的设备，和当前已经连接的设备来抉择，选择算法是在代码写死的。
之前有个平板没有听筒就是在这个文件中把听筒删掉。

##### 3. 音频参数

- 源码目录： vendor/qcom/proprietary/mm-audio/audcal/family-b/acdbdata/
- 手机目录： /etc/acdbdata/

这个是用高通工具QACT调试生成的音频参数，目前由硬件同事负责调试生成




