---
tags: [AV,ps,psm]
create_time: 2022-10-25
---
### Program Stream Map(PSM)
对 psm header 的解析主要是为了，解析出流的映射和对应关系  

**packet_start_code_prefix**  
该字段是值为'0000 0000 0000 0000 0000 0001' (0x000001)的位串。  
**map_stream_id**  
8 位字段，值为 0xBC  

>data 指向第一个字节

##### 计算指向 es_map 的位置
```cpp
int program_stream_info_length = (data[8] << 8) | data[9];
int i = 10 + program_stream_info_length;
```
##### 计算 es_map 的长度
``` cpp
// 直接获取
int element_stream_map_length = (data[i] << 8) | data[i+1];
```
  

``` cpp
// 通过两个长度相减获取，media-server采取此方法
int program_stream_map_length = (data[4] << 8) | data[5];
int program_stream_info_length = (data[8] << 8) | data[9];
int element_stream_map_length = program_stream_map_length - program_stream_info_length - 10;
```
>media-server 采用此种方式，可能直接获取对部分流有误

**流类型字段  stream_type**
8 位字段，根据表 2-29 规定了流的类型。该字段只能标志包含在 PES 分组中的基本流且取值不能为 0x05。
>这里我们暂时根据国标 GB28181 中的定义可以知道
1、MPEG-4 视频流： 0x10； 
2、H.264 视频流： 0x1B；  
3、SVAC 视频流： 0x80；  
4、G.711 音频流： 0x90；  
5、G.722.1 音频流： 0x92；  
6、G.723.1 音频流： 0x93；  
7、G.729 音频流： 0x99；  
8、SVAC 音频流： 0x9B。  
因为节目映射流字段只有在关键帧打包的时候，才会存在，所以如果要判断 PS 打包的流编码类型，就根据这个字段来判断。

**基本流标识字段  elementary_stream_id**
8 位字段，指出该基本流所在 PES 分组的 PES 分组标题中 stream_id 字段的值。
>这个字段的定义，其中 0x(C0~DF)指音频，0x(E0~EF)为视频

```cpp
// 跳过elementary_stream_map_length 字段
i+=2;
for(j = i; 
    j + 4/*element_stream_info_length*/ <= i+element_stream_map_length && 
    // psm->stream 会在之前算出来
    psm->stream_count < sizeof(psm->streams)/sizeof(psm->streams[0]); 
    j += 4 + element_stream_info_length)
{
    element_stream_info_length = (data[j + 2] << 8) | data[j + 3];

    stream_type = data[j];
    elementary_stream_id = data[j+1];

    /*
    descroptor() 的解析
    */
}
```
