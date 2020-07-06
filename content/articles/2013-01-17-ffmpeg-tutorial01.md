---
title: 'ffmpeg教程一：视频截图（Tutorial 01: Making Screencaps'
category: 多媒体
tags: ffmpeg, tutroial
---

教程原地址为：[http://dranger.com/ffmpeg/tutorial01.html][1]，本人只是做了翻译和部分接口更新工作，保证其能正常工作。需要做一些预备工作
<!-- excerpt -->

    :::java
    sudo apt-get install libavcodec-dev libavformat-dev libswscale-dev libavdevice-dev libavfilter-dev libavutil-dev libpostproc-dev 

首先我们需要了解视频文件的一些基本概念，视频文件本身被称作容器，例如avi或者是quicktime，容器的类型确定了文件的信息。然后，容器里装的东西叫流（stream），通常包括视频流和音频流（“流”的意思其实就是“随着时间推移的一段连续的数据元素”）。流中的数据元素叫做“帧”。每个流由不同的编解码器来编码，编解码器定义了数据如何编码（COded）和解码（DECoded），所以叫做编解码器（CODEC）。编解码器的例子有Divx和mp3。包（Packets），是从流中读取的，通过解码器解包，得到原始的帧，我们便可以对这些数据进行播放等的处理。对于我们来说，每个包包含完整的帧，或者多个音频帧。

在初级的水平，处理音视频流是非常简单的：

* 从video.avi中获得视频流
* 从视频流中解包得到帧
* 如果帧不完整，重复第2步
* 对帧进行相关操作
* 重复第2步

用ffmpeg来处理多媒体就像上面的步骤那么简单，即使你的第4步可能很复杂。所以在本教程，我们先打开一个视频，读取视频流，获得帧，然后第4步是把帧数据存储为PPM文件。

<br/>

##打开文件

我们先来看一下怎么打开一个视频文件，首先把头文件包含进来

    :::c
    #include <libavcodec/avcodec.h>
    #include <libavformat/avformat.h>
    #include <libswscale/swscale.h>
    ...
    int main(int argc, char *argv[]){
      av_register_all();

`av_register_all`只需要调用一次，他会注册所有可用的文件格式和编解码库，当文件被打开时他们将自动匹配相应的编解码库。如果你愿意，可以之注册个别的文件格式和编解码库。

现在我们真的要打开一个文件了

    :::c
    AVFormatContext *pFormatCtx;

    if(av_open_input_file(&pFormatCtx,argv[1],NULL,0,NULL)!=0)
      return -1;

我们从传入的第一个参数获得文件路径，这个函数会读取文件头信息，并把信息保存在我们存入的`pFormatCtx`结构体当中。这个函数后面三个参数分别是指定文件格式，缓存大小，和格式化选项，当我们设置为NULL或0时，libavformat会自动完成这些工作。

这个函数仅仅是获得了头信息，下一步我们要得到流信息

    :::c
    if(av_find_steam_info(pFormatCtx)<0)
      return -1

这个函数填充了`pFormatCtx->streams`流信息，

我们可以通过`dump_format`把信息打印出来

    :::c
    dump_format(pFormatCtx, 0, argv[1], 0);
    
`pFromatCtx->streams`只是大小为`pFormateCtx->nb_streams`的一系列的点，我们要从中得到视频流

    :::c
    int i;
    AVCodecContext *pCodecCtx;
    // Find the first video stream
    videoStream=-1;
    for(i=0; i<pformatctx->nb_streams; i++)
      if(pFormatCtx->streams[i]->codec->codec_type==CODEC_TYPE_VIDEO) 
      {
        videoStream=i;
        break;
      }   
    if(videoStream==-1)
      return -1; // Didn't find a video stream</pformatctx-></pre>
    
    // Get a pointer to the codec context for the video stream
    pCodecCtx=pFormatCtx->streams[videoStream]->codec;

pCodecCtx包含了这个流在用的编解码的所有信息，但我们仍需要通过他获得特定的解码器然后打开他。

    :::c
    AVCodec *pCodec;
    pCodec=avcodec_find_decoder(pCodecCtx->codec_id);
    if(pCodec==NULL) {
      fprintf(stderr, "Unsupported codec!\n");
      return -1; // Codec not found
    }
    // Open codec
    if(avcodec_open(pCodecCtx, pCodec)<0)
      return -1; // Could not open codec

<br/>

##存储数据

现在我们需要一个地方来存储一帧

    :::c
    AVFrame *pFrame;
    pFrame=avcodec_alloc_frame();

我们计划存储的PPM文件，其存储的数据是24位RGB，我们需要把得到的一帧从本地格式转换为RGB，ffmpeg可以帮我们完成这个工作。在很多工程里，我们都希望把原始帧转换到特定格式。现在就让我们来完成这个工作吧。

    :::c
    AVFrame *pFrameRGB;
    pFrameRGB=avcodec_alloc_frame();
    if(pFrameRGB==NULL)
      return -1;

即使我们分配了帧空间，我们仍然需要空间来存放转换时的raw数据，我们用`avpicture_get_size`来得到我们需要的空间，然后手动分配。

    :::c
    uint8_t *buffer;
    int numBytes;
    numBytes=avpicture_get_size(PIX_FMT_RGB24, pCodecCtx->width,
                    pCodecCtx->height);
    buffer=(uint8_t *)av_malloc(numBytes*sizeof(uint8_t));

av_malloc是ffmpeg简单封装的一个分配函数，意在确保内存地址的对齐等，它不会保护你的内存泄漏，二次释放，或其他malloc问题。

现在，我们使用avpicture_fill来关联我们新分配的缓冲区的帧。AVPicture结构体是AVFrame结构体的一个子集，开始的AVFrame是和AVPicture相同的。

    :::c
    // Assign appropriate parts of buffer to image planes in pFrameRGB
    // Note that pFrameRGB is an AVFrame, but AVFrame is a superset
    // of AVPicture
    avpicture_fill((AVPicture *)pFrameRGB, buffer, PIX_FMT_RGB24,
           pCodecCtx->width, pCodecCtx->height);

我们准备读取流了！

<br/>

##读取数据

我们要做的是通过包来读取整个视频流，然后解码到帧当中，一但一帧完成了，我们将转换并保存它。这里跟教程的接口调用有不一样的地方。

    :::c
    int frameFinished;
    AVPacket packet;
    
    i=0;
    while(av_read_frame(pFormatCtx, &packet)>=0) {
      // Is this a packet from the video stream?
      if(packet.stream_index==videoStream) {
        // Decode video frame
    //    avcodec_decode_video(pCodecCtx, pFrame, &frameFinished, 
    //           packet.data, packet.size);
          int result;
          avcodec_decode_video2(pCodecCtx,pFrame,&frameFinished,
                  &packet);
    
    // Did we get a video frame?
        if(frameFinished) {
      //  Convert the image from its native format to RGB
    //    img_convert((AVPicture *)pFrameRGB, PIX_FMT_RGB24, 
    //                  (AVPicture*)pFrame, pCodecCtx->pix_fmt,pCodecCtx->width, 
    //                    pCodecCtx->height);
    
    img_convert_ctx = sws_getContext(pCodecCtx->width,
                                          pCodecCtx->height,
                                          pCodecCtx->pix_fmt,
                                          pCodecCtx->width,
                                          pCodecCtx->height,
                                          PIX_FMT_RGB24,
                                          SWS_BICUBIC,NULL,
                                          NULL,NULL);
        result = sws_scale(img_convert_ctx,
                  (const uint8_t* const*)pFrame->data,
                  pFrame->linesize,
                  0,
                  pCodecCtx->height,
                  pFrameRGB->data,
                  pFrameRGB->linesize);
        printf("get result is %d~~~~~~~~~\n",result);
        // Save the frame to disk
        printf("i is %d \n",i);
        if(++i<=5)
          SaveFrame(pFrameRGB, pCodecCtx->width, pCodecCtx->height,
                i);
          }
        }
    
    // Free the packet that was allocated by av_read_frame
       av_free_packet(&packet);
    }

现在我们需要做的事情就是写SaveFrame函数来保存数据到PPM文件。

    :::c
    void SaveFrame(AVFrame *pFrame, int width, int height, int iFrame) {
      FILE *pFile;
      char szFilename[32];
      int  y;</pre>
    
    printf("start sws_scale\n");
      // Open file
      sprintf(szFilename, "frame%d.ppm", iFrame);
      pFile=fopen(szFilename, "wb");
      if(pFile==NULL)
      {
        printf("pFile is null");
        return;
      }
    
    // Write header
      fprintf(pFile, "P6\n%d %d\n255\n", width, height);
    
    // Write pixel data
      for(y=0; y<height; y++)
        fwrite(pFrame->data[0]+y*pFrame->linesize[0], 1, width*3, pFile);
    
    // Close file
      fclose(pFile);
    }

我们做了一些标准文件打开，然后写RGB数据，我们一次写一行文件，PPM文件就是简单地把RGB信息保存为一长串。头部记录着宽和高，和RGB的最大尺寸。

现在回到main函数，读完视频流后，我们需要释放一切

    :::c
    // Free the RGB image
      av_free(buffer);
      av_free(pFrameRGB);
    
    // Free the YUV frame
      av_free(pFrame);
    
    // Close the codec
      avcodec_close(pCodecCtx);
    
    // Close the video file
      av_close_input_file(pFormatCtx);
    
    return 0;

这些就是全部代码来，现在你需要linux系统编译和运行

    :::java
    gcc -o tutorial01 tutorial01.c -lavformat -lavcodec -lswscale -lz

得到tutorial01，执行以下语句可得到同级目录下的5个PPM文件

    :::java
    ./tutorial01 hello.mp4 

需要源码可留言。

[1]: http://dranger.com/ffmpeg/tutorial01.html
