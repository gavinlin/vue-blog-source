---
title: ffmpeg教程二：屏幕显示
date: 2013-12-31
category: 多媒体
tags: ffmpeg, tutorial
---

## SDL and Video

显示图像的方法有很多，这里我们使用 SDL 。SDL 英文全称为 Simple Direct Layer 。是一个跨平台的多媒体库，也就是你可以在 windonws linuxe 和 mac 里面使用这个库。所谓的多媒体库，就是提供了图像显示，声音播放，和线程相关接口的库。在 linux 上安装这个库很简单，示例代码的 README 已经有写了。
<!-- excerpt -->

SDL 提供了很多显示图片的方法，SDL显示图片是通过把数据显示在 YUV 层实现的。YUV 是显示原始数据的一种方法，类似于 RGB，通俗地说， Y 代表亮度系数， U 和 V 代表颜色系数。SDL的工作方式是把 YUV数据传进 YUV 层并显示。它接受4种 YUV 格式，但其中 YV12 是最快的。另一种格式叫 YUV420P 和 YV12 差不多，只是 U 和 V 调转了。420 的意思是每个样本的比例是 4:2:0 ，也就是每4个亮度分量共享一个颜色分量。这样可以节省带宽，而且人类对于颜色分量不太敏感，即使去掉几个也看不出差别。"P" 的意思是数据是 Y U V 在不同的数组中。ffmpeg 能够把图像转为 YUV420P， 其实很多视频已经使用这种格式存储图像，或者很容易就可以转为到这种格式。

现在如果想要显示图像，我们要把之前的代码中的 SaveFrame 函数替换掉。但是首先，我们先要知道怎么用 SDL 。

首先需要把库包含进来并初始化 SDL：

    :::c
    #include <SDL.h>
    #include <SDL_thread.h>

    if(SDL_Init(SDL_INIT_VIDEO | SDL_INIT_AUDIO | SDL_INIT_TIMER)) {
      fprintf(stderr, "Could not initialize SDL - %s\n", SDL_GetError());
      exit(1);
    }

SDL_Init 告诉库我们先要用什么功能。SDL_GetError 显然是用来处理 debug 信息的函数。

##Creating a Display

现在我们需要一块屏幕来显示东西。SDL 中显示图像的区域叫做 surface：

    :::c
    SDL_Surface *screen;

    screen = SDL_SetVideoMode(pCodecCtx->width, pCodecCtx->height, 0, 0);
    if(!screen) {
      fprintf(stderr, "SDL: could not set video mode - exiting\n");
      exit(1);
    }

这里给定了屏幕的宽和高。下一个参数是屏幕位数，0的意思是使用现在的显示值。

这样我们就创建了 YUV 层了，接着就可以把视频放进去了。

    :::c
    SDL_Overlay     *bmp;

    bmp = SDL_CreateYUVOverlay(pCodecCtx->width, pCodecCtx->height,
                               SDL_YV12_OVERLAY, screen);

就像之前说的，我们使用 YV12 来显示图像。

##Displaying the Image

是不是很简单？接着我们只需要显示图片了。一直去到得到 frame的地方。 我们可以把之前的代码都删掉，换为显示图像的代码。要显示图像，我们需要一个 AVPicture 结构体，然后设置数据指针和 linesize 到我们的 YUV 层。

    :::c
    if(frameFinished) {
        SDL_LockYUVOverlay(bmp);

        AVPicture pict;
        pict.data[0] = bmp->pixels[0];
        pict.data[1] = bmp->pixels[2];
        pict.data[2] = bmp->pixels[1];

        pict.linesize[0] = bmp->pitches[0];
        pict.linesize[1] = bmp->pitches[2];
        pict.linesize[2] = bmp->pitches[1];

        // Convert the image into YUV format that SDL uses
        img_convert(&pict, PIX_FMT_YUV420P,
                        (AVPicture *)pFrame, pCodecCtx->pix_fmt, 
                pCodecCtx->width, pCodecCtx->height);
        
        SDL_UnlockYUVOverlay(bmp);
      }    

首先我们需要把显示层锁上，因为我们要往里面写数据。这样做可以避免后面可能出现的问题。AVPicture 结构体，就像之前看到的，有4个指向数据的指针。由于我们处理 YUV420P 格式，所以只用到其中三个通道，因此只需其中3套数据。其他格式可能会用到第四个数据指针来存储透明值或者其他东西。linesize 就像它的名字那样。对于 YUV 层有 pitches与 linesize 对应。所以我们只要把指针指向 pitches，然后当我们往 pict 写数据时，实际上是往 overlay层写数据。

##Drawing the Image

虽然图像数据已经在 Overlay 层了，但是我们还是需要告诉 SDL 显示数据。我们还传一个矩型到函数里面告诉 SDL 图像应该显示的地方和缩放比例。在这里，SDL 帮助我们做了缩放，以获得较高的处理速度。

    :::c
    SDL_Rect rect;

      if(frameFinished) {
        /* ... code ... */
        // Convert the image into YUV format that SDL uses
        img_convert(&pict, PIX_FMT_YUV420P,
                        (AVPicture *)pFrame, pCodecCtx->pix_fmt, 
                pCodecCtx->width, pCodecCtx->height);
        
        SDL_UnlockYUVOverlay(bmp);
        rect.x = 0;
        rect.y = 0;
        rect.w = pCodecCtx->width;
        rect.h = pCodecCtx->height;
        SDL_DisplayYUVOverlay(bmp, &rect);
      }

这样视频就开始显示了。

下面说一下 SDL 另一个特性，它的事件处理系统。当你在 SDL 程序中打字，移动鼠标或者发送信号，它都会产生一个事件。你的程序可以截获这些事件和处理先要处理的事件。你的程序也可以向 SDL 事件系统发送事件。这个在多线程程序中很有用。这个会在教程4中有所体现。现在我们只用它来处理程序的退出事件。

    :::c
    SDL_Event       event;

        av_free_packet(&packet);
        SDL_PollEvent(&event);
        switch(event.type) {
        case SDL_QUIT:
          SDL_Quit();
          exit(0);
          break;
        default:
          break;
        }

暂时到这里，后面的更精彩。

源代码可以在 [https://github.com/gavinlin/ffmpeg_tutorial](https://github.com/gavinlin/ffmpeg_tutorial) 代码会有些出入，因为源码中使用了新版本的 ffmpeg 。
