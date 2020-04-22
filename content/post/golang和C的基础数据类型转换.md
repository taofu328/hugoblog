---
title: "golang和C的基础数据类型转换"
date: 2020-03-31T09:25:21+08:00
draft: false
tags: [
    "go",
    "编程",
]
---

| C语言类型 | CGO类型 | Go语言类型 |
| --- | --- | --- |
| char | C.char | byte |
| singed char | C.schar | int8 |
| unsigned char | C.uchar | uint8 |
| short | C.short | int16 |
| unsigned short | C.ushort | uint16 |
| int | C.int | int32 |
| unsigned int | C.uint | uint32 |
| long | C.long | int32 |
| unsigned long | C.ulong | uint32 |
| long long int | C.longlong | int64 |
| unsigned long long int | C.ulonglong | uint64 |
| float | C.float | float32 |
| double | C.double | float64 |
| size_t | C.size_t | uint |
