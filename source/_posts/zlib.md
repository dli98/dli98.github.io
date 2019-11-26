---
title: zlib，gzip快速识别
date: 2019-11-25 17:40:38
tags:
- zlib
- gzip
---

这里只是快速识别压缩算法，不讲实现原理
<!-- more -->
## zlib压缩数据格式
![](/images/zlib.png)

### CMF（Compression Method and flags）:  
0~3bit CM 压缩方法。  
4~7bit CNIFO 压缩信息。  
目前只有deflate(CM=8)算法。对于CM=8，CNIFO几乎不会设置7以外的任何值，因此CMF常见数值为0x78

### FLG (FLaGs) :  
0~4bit FCHECK 检测位。FLEVEL和FDICT确定之后计算FCHECK。最后（CMF * 256 + FLG）是31的倍数  
5bit  FDICT  
6~7bit FLEVEL 压缩等级。  

### FLEVEL (compress level)  
不同的压缩算法有不同的等级。deflate（CM=8）算法压缩等级如下  
 　　0 - 最快压缩  
 　　1 - 快压缩  
 　　2 - 默认压缩  
 　　3 - 最大压缩率，最慢速度  

### 常用头部
| CMF   |             FLG            |
| :---: | :------------------------: |
|0x78   | 0x01 - No Compression/low  |
|0x78   | 0x5e - Fast Compression    |
|0x78   | 0x9C - Default Compression |
|0x78   | 0xDA - Best Compression    |

<font color=red> \x78\x79 </font>最常见

## gzib 压缩数据格式

![](/images/gzip.png)

### ID1和ID2
确定一个文件为gzip格式的标识，因此ID1和ID2为固定值：ID1 = 31(0x1f)，ID2 = 139(0x8b)

### CM
目前只有deflate(CM=8)算法

### 总结
因此根据 <font color=red> \x1f\x8b\x08 </font> 就可以确定这个算法为gzib压缩算法。


## python zlib模块
`zlib.compress`对数据进行 zlib 格式压缩，好像压缩的时候只支持这一种格式，不过你可以通过把压缩后的数据去掉两字节 zlib 头和末尾 4 字节的 CRC 校验得到 deflate 格式的压缩数据（zlib_compressed[2:-4]）

对于 gzip 压缩格式可以通过 gzip 模块得到。  

`zlib.decompress(string[,wbits[,bufsize]])` 解压更折腾，压缩格式是通过 `wbits` 参数控制的：

- [8, 15]：解压 zlib 格式的数据, 必须有头和尾
- [-8, -15]：解压 deflate 格式数据
- [24, 31] = 16 + (8 to 15)：解压 gzip 格式数据
- [40, 47] = 32 + (8 to 15)：zip和gzip都可以
 
## 参考：
https://tools.ietf.org/html/rfc1950  
https://tools.ietf.org/html/rfc1952
https://my.oschina.net/u/658658/blog/306777