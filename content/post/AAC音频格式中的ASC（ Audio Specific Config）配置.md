---
title: "AAC音频格式中的ASC（ Audio Specific Config）配置"
date: 2020-04-03T10:15:25+08:00
draft: false
tags: [
    "go",
    "aac",
]
---
在解码raw格式的aac音频时，需要传入一个参数asc,通常是一个十六进制数组，例如{0x14, 0x08}，这个参数表示的意思是什么呢。
### ASC配置的结构
 asc为一串二进制数字,结构如下
```
5 bits: object type
if (object type == 31)
    6 bits + 32: object type
4 bits: frequency index //采样频率的索引
if (frequency index == 15)
    24 bits: frequency
4 bits: channel configuration //声道配置
var bits: AOT Specific Config
```


前面的{0x14, 0x08}，转换成2进制就是‭00010 1000 0001 000‬，前五位00010，换成10进制是2，从Audio Object Types列表查询可知表示AAC LC，接下来表示采样频率的4位是1000,等于8，查表可得采样频率是16000k。接下来表示声道的4位是0001，等于1，查表可知表示只有一个声道。 object type，采样频率和声道的列表在下面。
### Audio Object Types 

- 0: Null
- 1: AAC Main
- 2: AAC LC (Low Complexity)
- 3: AAC SSR (Scalable Sample Rate)
- 4: AAC LTP (Long Term Prediction)
- 5: SBR (Spectral Band Replication)
- 6: AAC Scalable
- 7: TwinVQ
- 8: CELP (Code Excited Linear Prediction)
- 9: HXVC (Harmonic Vector eXcitation Coding)
- 10: Reserved
- 11: Reserved
- 12: TTSI (Text-To-Speech Interface)
- 13: Main Synthesis
- 14: Wavetable Synthesis
- 15: General MIDI
- 16: Algorithmic Synthesis and Audio Effects
- 17: ER (Error Resilient) AAC LC
- 18: Reserved
- 19: ER AAC LTP
- 20: ER AAC Scalable
- 21: ER TwinVQ
- 22: ER BSAC (Bit-Sliced Arithmetic Coding)
- 23: ER AAC LD (Low Delay)
- 24: ER CELP
- 25: ER HVXC
- 26: ER HILN (Harmonic and Individual Lines plus Noise)
- 27: ER Parametric
- 28: SSC (SinuSoidal Coding)
- 29: PS (Parametric Stereo)
- 30: MPEG Surround
- 31: (Escape value)
- 32: Layer-1
- 33: Layer-2
- 34: Layer-3
- 35: DST (Direct Stream Transfer)
- 36: ALS (Audio Lossless)
- 37: SLS (Scalable LosslesS)
- 38: SLS non-core
- 39: ER AAC ELD (Enhanced Low Delay)
- 40: SMR (Symbolic Music Representation) Simple
- 41: SMR Main
- 42: USAC (Unified Speech and Audio Coding) (no SBR)
- 43: SAOC (Spatial Audio Object Coding)
- 44: LD MPEG Surround
- 45: USAC
### 采样频率
支持13种采样频率

- 0: 96000 Hz
- 1: 88200 Hz
- 2: 64000 Hz
- 3: 48000 Hz
- 4: 44100 Hz
- 5: 32000 Hz
- 6: 24000 Hz
- 7: 22050 Hz
- 8: 16000 Hz
- 9: 12000 Hz
- 10: 11025 Hz
- 11: 8000 Hz
- 12: 7350 Hz
- 13: Reserved
- 14: Reserved
- 15: frequency is written explictly
### 声道配置

- 0: Defined in AOT Specifc Config
- 1: 1 channel: front-center
- 2: 2 channels: front-left, front-right
- 3: 3 channels: front-center, front-left, front-right
- 4: 4 channels: front-center, front-left, front-right, back-center
- 5: 5 channels: front-center, front-left, front-right, back-left, back-right
- 6: 6 channels: front-center, front-left, front-right, back-left, back-right, LFE-channel
- 7: 8 channels: front-center, front-left, front-right, side-left, side-right, back-left, back-right, LFE-channel
- 8-15: Reserved
### 参考资料
[MPEG-4 Audio](https://wiki.multimedia.cx/index.php?title=MPEG-4_Audio#Audio_Specific_Config)
