---
title: "Golang对AAC音频进行编码解码"
date: 2020-04-03T9:37:25+08:00
draft: false
tags: [
    "go",
    "aac",
]
---


使用了开源项目[go-fdkaac](https://github.com/winlinvip/go-fdkaac)，作者的[srs](https://github.com/ossrs/srs)是一款非常好用直播服务器软件。go-fdkaac是对[fdk-aac](https://github.com/mstorsjo/fdk-aac)的封装。
### 获取源码
```shell
go get -d github.com/winlinvip/go-fdkaac
```
### 编译fdk-aac
```shell
cd $GOPATH/src/github.com/winlinvip/go-fdkaac
git clone https://github.com/winlinvip/fdk-aac.git fdk-aac-lib
cd fdk-aac-lib
bash autogen.sh
```
clone下来的文件是windows格式的，运行 `bash autogen.sh` 会出错。需要先进行格式转换，这里使用 `dos2unix` 进行转换
```shell
sudo apt-get install dos2unix
dos2unix *.ac
dos2unix *.sh
dos2unix *.am
bash autogen.sh
./configure --prefix=`pwd`/objs 
make && make install
```
### 运行测试代码
```shell
cd $GOPATH/src/github.com/winlinvip/go-fdkaac
go test
```
出现一堆错误
```shell
../github.com/winlinvip/go-fdkaac/fdkaac/../fdk-aac-lib/objs/lib/libfdk-aac.a(genericStds.o)：在函数‘FDKsqrt’中：
/mnt/hgfs/linux/go/src/github.com/winlinvip/go-fdkaac/fdk-aac-lib/libSYS/src/genericStds.cpp:358：对‘sqrt’未定义的引用
../github.com/winlinvip/go-fdkaac/fdkaac/../fdk-aac-lib/objs/lib/libfdk-aac.a(genericStds.o)：在函数‘FDKpow’中：
/mnt/hgfs/linux/go/src/github.com/winlinvip/go-fdkaac/fdk-aac-lib/libSYS/src/genericStds.cpp:357：对‘pow’未定义的引用
../github.com/winlinvip/go-fdkaac/fdkaac/../fdk-aac-lib/objs/lib/libfdk-aac.a(genericStds.o)：在函数‘FDKatan’中：
/mnt/hgfs/linux/go/src/github.com/winlinvip/go-fdkaac/fdk-aac-lib/libSYS/src/genericStds.cpp:359：对‘atan’未定义的引用
../github.com/winlinvip/go-fdkaac/fdkaac/../fdk-aac-lib/objs/lib/libfdk-aac.a(genericStds.o)：在函数‘FDKlog’中：
/mnt/hgfs/linux/go/src/github.com/winlinvip/go-fdkaac/fdk-aac-lib/libSYS/src/genericStds.cpp:360：对‘log’未定义的引用
../github.com/winlinvip/go-fdkaac/fdkaac/../fdk-aac-lib/objs/lib/libfdk-aac.a(genericStds.o)：在函数‘FDKsin’中：
/mnt/hgfs/linux/go/src/github.com/winlinvip/go-fdkaac/fdk-aac-lib/libSYS/src/genericStds.cpp:361：对‘sin’未定义的引用
../github.com/winlinvip/go-fdkaac/fdkaac/../fdk-aac-lib/objs/lib/libfdk-aac.a(genericStds.o)：在函数‘FDKcos’中：
/mnt/hgfs/linux/go/src/github.com/winlinvip/go-fdkaac/fdk-aac-lib/libSYS/src/genericStds.cpp:362：对‘cos’未定义的引用
../github.com/winlinvip/go-fdkaac/fdkaac/../fdk-aac-lib/objs/lib/libfdk-aac.a(genericStds.o)：在函数‘FDKexp’中：
/mnt/hgfs/linux/go/src/github.com/winlinvip/go-fdkaac/fdk-aac-lib/libSYS/src/genericStds.cpp:363：对‘exp’未定义的引用
../github.com/winlinvip/go-fdkaac/fdkaac/../fdk-aac-lib/objs/lib/libfdk-aac.a(genericStds.o)：在函数‘FDKatan2’中：
/mnt/hgfs/linux/go/src/github.com/winlinvip/go-fdkaac/fdk-aac-lib/libSYS/src/genericStds.cpp:364：对‘atan2’未定义的引用
../github.com/winlinvip/go-fdkaac/fdkaac/../fdk-aac-lib/objs/lib/libfdk-aac.a(genericStds.o)：在函数‘FDKacos’中：
/mnt/hgfs/linux/go/src/github.com/winlinvip/go-fdkaac/fdk-aac-lib/libSYS/src/genericStds.cpp:365：对‘acos’未定义的引用
../github.com/winlinvip/go-fdkaac/fdkaac/../fdk-aac-lib/objs/lib/libfdk-aac.a(genericStds.o)：在函数‘FDKtan’中：
/mnt/hgfs/linux/go/src/github.com/winlinvip/go-fdkaac/fdk-aac-lib/libSYS/src/genericStds.cpp:366：对‘tan’未定义的引用
collect2: error: ld returned 1 exit status

```
这是用了math库，但是没有静态链接进去导致的，使用math库编译需要加-lm参数。将dec.go和enc.go文件的LDFLAGS增加-lm参数,go-fdkaac编译测试通过
```shell
#cgo CFLAGS: -I${SRCDIR}/../fdk-aac-lib/objs/include/fdk-aac
#cgo LDFLAGS: ${SRCDIR}/../fdk-aac-lib/objs/lib/libfdk-aac.a -lm
#include "aacdecoder_lib.h"
```
### 编码解码函数的调用
#### 解码raw格式的aac帧
我的项目中使用的是不含ADTS头部的raw格式的aac帧，解码方法如下所示。这里要注意的是一定要设置正确的ASC。ASC的计算方法会在另一篇文章里讲。
```go
func AacDecoder_RAW() {
	var err error
	d := fdkaac.NewAacDecoder()

	asc := []byte{0x12, 0x10}
	if err := d.InitRaw(asc); err != nil {
		fmt.Println("init decoder failed, err is", err)
		return
	}
	defer d.Close()

	// directly decode the frame to pcm.
	var pcm []byte
	if pcm, err = d.Decode([]byte{
		0x21, 0x17, 0x55, 0x35, 0xa1, 0x0c, 0x2f, 0x00, 0x00, 0x50, 0x23, 0xa6, 0x81, 0xbf, 0x9c, 0xbf,
		0x13, 0x73, 0xa9, 0xb0, 0x41, 0xed, 0x60, 0x23, 0x48, 0xf7, 0x34, 0x07, 0x12, 0x53, 0xd8, 0xeb,
		0x49, 0xf4, 0x1e, 0x73, 0xc9, 0x01, 0xfd, 0x16, 0x9f, 0x8e, 0xb5, 0xd5, 0x9b, 0xb6, 0x49, 0xdb,
		0x35, 0x61, 0x3b, 0x54, 0xad, 0x5f, 0x9d, 0x34, 0x94, 0x88, 0x58, 0x89, 0x33, 0x54, 0x89, 0xc4,
		0x09, 0x80, 0xa2, 0xa1, 0x28, 0x81, 0x42, 0x10, 0x48, 0x94, 0x05, 0xfb, 0x03, 0xc7, 0x64, 0xe1,
		0x54, 0x17, 0xf6, 0x65, 0x15, 0x00, 0x48, 0xa9, 0x80, 0x00, 0x38}); err != nil {
		fmt.Println("decode failed, err is", err)
		return
	}

	fmt.Println("SampleRate:", d.SampleRate())
	fmt.Println("FrameSize:", d.FrameSize())
	fmt.Println("NumChannels:", d.NumChannels())
	fmt.Println("AacSampleRate:", d.AacSampleRate())
	fmt.Println("Profile:", d.Profile())
	fmt.Println("AudioObjectType:", d.AudioObjectType())
	fmt.Println("ChannelConfig:", d.ChannelConfig())
	fmt.Println("Bitrate:", d.Bitrate())
	fmt.Println("AacSamplesPerFrame:", d.AacSamplesPerFrame())
	fmt.Println("AacNumChannels:", d.AacNumChannels())
	fmt.Println("ExtensionAudioObjectType:", d.ExtensionAudioObjectType())
	fmt.Println("ExtensionSamplingRate:", d.ExtensionSamplingRate())
	fmt.Println("NumLostAccessUnits:", d.NumLostAccessUnits())
	fmt.Println("NumTotalBytes:", d.NumTotalBytes())
	fmt.Println("NumBadBytes:", d.NumBadBytes())
	fmt.Println("NumTotalAccessUnits:", d.NumTotalAccessUnits())
	fmt.Println("NumBadAccessUnits:", d.NumBadAccessUnits())
	fmt.Println("SampleBits:", d.SampleBits())
	fmt.Println("PCM:", len(pcm))

	// Output:
	// SampleRate: 44100
	// FrameSize: 1024
	// NumChannels: 2
	// AacSampleRate: 44100
	// Profile: 1
	// AudioObjectType: 2
	// ChannelConfig: 2
	// Bitrate: 31352
	// AacSamplesPerFrame: 1024
	// AacNumChannels: 2
	// ExtensionAudioObjectType: 0
	// ExtensionSamplingRate: 0
	// NumLostAccessUnits: 0
	// NumTotalBytes: 91
	// NumBadBytes: 0
	// NumTotalAccessUnits: 1
	// NumBadAccessUnits: 0
	// SampleBits: 16
	// PCM: 4096
}
```
#### 解码带有ADTS头部的aac帧
```go
func ExampleAacDecoder_ADTS() {
	var err error
	d := fdkaac.NewAacDecoder()

	if err := d.InitAdts(); err != nil {
		fmt.Println("init decoder failed, err is", err)
		return
	}
	defer d.Close()

	var pcm []byte
	if pcm, err = d.Decode([]byte{0xff, 0xf1, 0x50, 0x80, 0x0c, 0x40, 0xfc,
		0x21, 0x17, 0x55, 0x35, 0xa1, 0x0c, 0x2f, 0x00, 0x00, 0x50, 0x23, 0xa6, 0x81, 0xbf, 0x9c, 0xbf,
		0x13, 0x73, 0xa9, 0xb0, 0x41, 0xed, 0x60, 0x23, 0x48, 0xf7, 0x34, 0x07, 0x12, 0x53, 0xd8, 0xeb,
		0x49, 0xf4, 0x1e, 0x73, 0xc9, 0x01, 0xfd, 0x16, 0x9f, 0x8e, 0xb5, 0xd5, 0x9b, 0xb6, 0x49, 0xdb,
		0x35, 0x61, 0x3b, 0x54, 0xad, 0x5f, 0x9d, 0x34, 0x94, 0x88, 0x58, 0x89, 0x33, 0x54, 0x89, 0xc4,
		0x09, 0x80, 0xa2, 0xa1, 0x28, 0x81, 0x42, 0x10, 0x48, 0x94, 0x05, 0xfb, 0x03, 0xc7, 0x64, 0xe1,
		0x54, 0x17, 0xf6, 0x65, 0x15, 0x00, 0x48, 0xa9, 0x80, 0x00, 0x38}); err != nil {
		fmt.Println("decode failed, err is", err)
		return
	}

	fmt.Println("SampleRate:", d.SampleRate())
	fmt.Println("FrameSize:", d.FrameSize())
	fmt.Println("NumChannels:", d.NumChannels())
	fmt.Println("AacSampleRate:", d.AacSampleRate())
	fmt.Println("Profile:", d.Profile())
	fmt.Println("AudioObjectType:", d.AudioObjectType())
	fmt.Println("ChannelConfig:", d.ChannelConfig())
	fmt.Println("Bitrate:", d.Bitrate())
	fmt.Println("AacSamplesPerFrame:", d.AacSamplesPerFrame())
	fmt.Println("AacNumChannels:", d.AacNumChannels())
	fmt.Println("ExtensionAudioObjectType:", d.ExtensionAudioObjectType())
	fmt.Println("ExtensionSamplingRate:", d.ExtensionSamplingRate())
	fmt.Println("NumLostAccessUnits:", d.NumLostAccessUnits())
	fmt.Println("NumTotalBytes:", d.NumTotalBytes())
	fmt.Println("NumBadBytes:", d.NumBadBytes())
	fmt.Println("NumTotalAccessUnits:", d.NumTotalAccessUnits())
	fmt.Println("NumBadAccessUnits:", d.NumBadAccessUnits())
	fmt.Println("SampleBits:", d.SampleBits())
	fmt.Println("PCM:", len(pcm))

	// Output:
	// SampleRate: 44100
	// FrameSize: 1024
	// NumChannels: 2
	// AacSampleRate: 44100
	// Profile: 1
	// AudioObjectType: 2
	// ChannelConfig: 2
	// Bitrate: 33764
	// AacSamplesPerFrame: 1024
	// AacNumChannels: 2
	// ExtensionAudioObjectType: 0
	// ExtensionSamplingRate: 0
	// NumLostAccessUnits: 0
	// NumTotalBytes: 98
	// NumBadBytes: 0
	// NumTotalAccessUnits: 1
	// NumBadAccessUnits: 0
	// SampleBits: 16
	// PCM: 4096
}

```


