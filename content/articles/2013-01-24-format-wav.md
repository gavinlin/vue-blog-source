---
title: wav音频文件格式详解
category: 多媒体
tags: linux, wav
---

##利用ffmpeg做音频转换

大多时候我们得到的都是mp3格式的音频文件，在linux下我们想要转换音频格式可以使用ffmpeg ，非常方便，没有ffmpeg就安装一个吧 `sudo apt-get install ffmpeg` 感觉ubuntu真心的方便。
<!-- excerpt -->

    :::java
    ffmpeg -i track8.mp3 -ar 44100 -ac 2 -acodec pcm_s32le track8.wav

根据上面的命令，我们把track8.mp3转换为采样率44100，采样精度32位，双声道的wav音频文件。 

<br/>

##文件的十六进制表示

我们选择vim来看其十六进制数据，或许你不相信vim的强大，但事实就是如此，我们用vim打开一个wav是这样的：

<span class="label label-info">info</span> 加上 -b 让vim知道你是打开了一个二进制文件，例如:vim track8.wav -b

![vim before](http://lingavin.com/wp-content/uploads/2013/01/vim-before-1.png)

不用急，只要用命令模式运行如下命令，将会有奇迹发生。`:%!xxd`

![vim after](http://lingavin.com/wp-content/uploads/2013/01/vim-after-1.png)

我们需要的信息就在眼前了。现在可以用来分析了。

<br/>

##wav框架

终于进入主题了。

分析头部信息前，首先要明确wav文件存储的头部信息是包含在trunk中的，wav由trunk组成的，一个wav大致有以下几个trunk

<br/>

|||
:------:|:------:|:------:
RIFF WAVE Chunk|ID='RIFF'|RiffType='WAVE'
Format Chunk | ID = 'fmt'
Fact Chunk(optional) | ID = 'fact'
Data Chunk | ID = 'data' 

<br/>

##具体分析

*RIFF WAVE Chunk*

<br/>

头|所占字节数|内容|例子
:------:|:------:|:------:|:------:
ID |4|'RIFF'|0x52494646
Size|4||0x0037296c
Type|4|'WAVE'|0x57415645

<br/>

可以看到首4个字节是'RIFF'，接着的四个字节是文件的总大小减去头8个字节，最后四个字节是格式类型'WAVE'。

*Format Chunk*

<br/>

名称|所占字节数|内容|例子
:------:|:------:|:------:
ID|4|'fmt'|0x666d7420
Size|4|本chunk大小|0x00000010
Formattag|2|编码方式，一般为0x0001|0x0001
Channels|2|声道数,1-单声道；2-二声道|0x0002
SamplesPerSec|4|采样频率|0x0000ac44
AvgBytesPerSec|4|每秒所需字节数|0x00056220
BlockAlign|2|数据块对齐单位|0x0008
BitPerSample|2|采样精度|0x0020
|2|附加信息（可选，有则size为18）|无

<br/>

这个trunk的大小为16个字节，没有附加信息。上面这些数据有什么关系呢？首先，有一些数据是直接可以得到的，例如声道数2，采样频率44100，采样精度32，而每秒所需的字节数可通过计算得到（44100 * 2 * 32 / 8）352800byte/s。

*Fact Chunk*

本文件没有fact chunk信息

*Data Chunk*

下面的就是数据了，数据根据不同的采样精度和通道数有不同的排列。

<br/>

|||||
:------:|:------:|:------:|:------:|:------:
单声道|取样1|取样2|取样3|取样4
8bit量化|声道0|声道0|声道0|声道0
-|-|-|-|-
双声道|取样1|取样2|取样3|取样4
8bit量化|声道0（左）|声道1（右）|声道0（左）|声道1（右）
-|-|-|-|-
单声道|取样1|取样2|取样3|取样4
16bit量化|声道0（低位）|声道0（高位）|声道0（低位）|声道0（高位）
-|-|-|-|-
双声道|取样1|取样2|取样3|取样4
16bit量化|声道0（左低位）|声道0（左高位）|声道1（右低位）|声道1（右高位）

<br/>

基本上，wav格式就分析完了，其对数据没有压缩，所以即使播放时间很短，音频数据也会很大。

下面有个网址详细地说了各个标签，真的很详细，所以诚意推荐[http://www.sonicspot.com/guide/wavefiles.html#cue](http://www.sonicspot.com/guide/wavefiles.html#cue)

最后贴一段代码，代码丑陋，仅供参考

    :::c
    struct WAVE_FORMAT
    {
        uint16_t wFormattag;
        uint16_t wChannels;
        uint32_t dwSamplesPerSec;
        uint32_t dwAvgBytesPerSec;
        uint16_t wBlockAlign;
        uint16_t wBitsPerSample;
    //  uint16_t pack; overhead information
    };
    
    struct FMT_BLOCK
    {
        uint8_t szFmtID[4];
        uint32_t dwFmtSize;
        struct WAVE_FORMAT wavFormat;
    };
    
    struct FACT_BLOCK
    {
        uint8_t szFactID[4];
        uint32_t dwFactSize;
    };
    
    struct CUE_BLOCK
    {
        uint8_t szCueID[4];
        uint32_t dwCueSize;
    };
    
    struct LIST_BLOCK
    {
        uint8_t szListID[4];
        uint32_t dwListSize;
    };
    
    struct DATA_BLOCK
    {
        uint8_t szDataID[4];
        uint32_t dwDataSize;
    }
    
    :::c
    static uint32_t get_wav_header(int fd, unsigned *rate, unsigned *channels, unsigned *bps)
    {
        struct RIFF_HEADER riffHeader;
        struct FMT_BLOCK fmtBlock;
        struct FACT_BLOCK factBlock;
        struct DATA_BLOCK dataBlock;
        struct CUE_BLOCK cueBlock;
        off_t currPos;
        //start read header
        if(read(fd,&riffHeader,sizeof(riffHeader)) == -1)
        {
            return -1;
        }</pre>
    
    trace("riff info size is %d",riffHeader.dwRiffSize);
        //read fmt_block ,important info here and we should handle that if it has overhead info.
        if(read(fd,&fmtBlock,sizeof(fmtBlock)) == -1)
        {
            return -1;
        }
    
    *rate = fmtBlock.wavFormat.dwSamplesPerSec;
        *channels = fmtBlock.wavFormat.wChannels;
        *bps = fmtBlock.wavFormat.wBitsPerSample;
        trace("rate is %d channels is %d bps is %d",
                *rate,*channels,*bps);
    
    if(fmtBlock.dwFmtSize == 18)
        {
            // 2 bytes offset
            currPos = lseek(fd,2,SEEK_CUR); 
        }else{
            currPos = lseek(fd,0,SEEK_CUR);
        }
    
    if(read(fd,&factBlock,sizeof(factBlock)) == -1)
        {
            return -1;
        }
        if(memcmp(factBlock.szFactID,"fact",4) == 0)
        {
            //skip fack
            trace("have hact");
            lseek(fd,factBlock.dwFactSize,SEEK_CUR);
        }else{
            trace("don't have fact");
            lseek(fd,currPos,SEEK_SET);
        }
    
    if(read(fd,&cueBlock,sizeof(cueBlock)) == -1)
        {
            return -1;
        }
    
    if(memcmp(cueBlock.szCueID,"cue ",4) == 0)
        {
            struct LIST_BLOCK listBlock;
            lseek(fd,cueBlock.dwCueSize,SEEK_CUR);
            read(fd,&listBlock,sizeof(listBlock));
            if(memcmp(listBlock.szListID,"LIST",4) == 0)
            {
                lseek(fd,listBlock.dwListSize,SEEK_CUR);
            }
        }else{
            lseek(fd,currPos,SEEK_SET);
        }
    
    trace("currPos is %d",currPos);
        if(read(fd,&dataBlock,sizeof(dataBlock)) == -1)
        {
            return -1;
        }
        if(memcmp(dataBlock.szDataID,"data",4) != 0)
        {
            return -1;
        }
        trace("data id is %s and size is %d",dataBlock.szDataID,
                dataBlock.dwDataSize);
        return dataBlock.dwDataSize;
    }

