---
title: 为android编译ffmpeg
date: 2013-03-10
category: Android
tags: ffmpeg
---

之前的教程是在桌面系统上实现的，由于系统已经为我们提供好了相关的库，所以我们之需要调用库的接口就可以使用ffmpeg为我们提供的功能了。但是我们现在需要在android系统上面调用这些函数，ffmpeg没有为我们提供可在android上运行的动态链接库。这时候我们需要。
<!-- excerpt -->

此次移植的是`ffmpeg0.11.2`，使用的编译工具是android提供的`android-r8d`，编译脚本与Android.mk参考了abitno。

First of the first,我们需要在ffmpeg主页下载源代码。到目前为止，ffmpeg的版本已经发展到了1.1.3，可我们的目标却是0.11.2，真是个固步自封的熊孩子。说话回来，我们不是在追求最新，只是追求可用。

ffmpeg的链接: [http://www.ffmpeg.org/download.html][1]，其实ffmpeg已经把代码托管到github，把github的版本库拉回来，想用哪个版本就切换到那个版本会更方便点。

<br/>

##config.h

config.h是一个很重要的配置文件，但是这个文件不是手工编写或者默认提供，而是需要通过`configure`得到。configure又是如何使用的？ffmpeg源代码目录敲入命令`./configure --help`，可以列出configure的相关选项。下面我提供了一个脚本，里面记录了configure的选项，只要我们运行脚本，就会配置ffmpeg了。

    :::sh
    #!/bin/sh
    
    ANDROID_NDK=/home/gavin/android-ndk-r8d
    SYSROOT=$ANDROID_NDK/platforms/android-14/arch-arm
    TOOLCHAIN=`echo $ANDROID_NDK/toolchains/arm-linux-androideabi-4.4.3/prebuilt/linux-x86`
    export PATH=$TOOLCHAIN/bin:$PATH
    FLAGS="--target-os=linux --cross-prefix=arm-linux-androideabi- --arch=arm --cpu=armv7-a "
    FLAGS="$FLAGS --sysroot=$SYSROOT"
    FLAGS="$FLAGS --disable-avdevice 
    				--disable-static 
    				--enable-shared 
    				--disable-ffplay 
    				--disable-doc 
    				--disable-ffmpeg 
    				--disable-ffprobe 
    				--disable-ffserver 
    				--disable-avfilter 
    				--disable-encoders 
    				--disable-muxers 
    				--disable-filters 
    				--disable-devices 
    				--enable-version3 
    				--enable-asm  
    				--enable-neon "
    
    EXTRA_CFLAGS="-I$ANDROID_NDK/sources/cxx-stl/system/include"
    
    EXTRA_CFLAGS="$EXTRA_CFLAGS -fPIC -march=armv7-a -mfloat-abi=softfp -mfpu=neon"
    #EXTRA_CFLAGS="$EXTRA_CFLAGS -march=armv7-a -mfloat-abi=softfp -mfpu=vfpv3-d16"
    EXTRA_LDFLAGS="-Wl,--fix-cortex-a8 -nostdlib"
    EXTRA_CXXFLAGS="-Wno-multichar -fno-exceptions -fno-rtti"
    ABI="armeabi-v7a"
    
    echo $FLAGS --extra-cflags="$EXTRA_CFLAGS" --extra-ldflags="$EXTRA_LDFLAGS" --extra-cxxflags="$EXTRA_CXXFLAGS" > info.txt
    ./configure $FLAGS --extra-cflags="$EXTRA_CFLAGS" --extra-ldflags="$EXTRA_LDFLAGS" --extra-cxxflags="$EXTRA_CXXFLAGS" | tee configuration.txt

<span class="label label-info">Info</span> 把libavutil/libm.h里面的static方法注释掉

编译出`config.h`后我们需要一些改动，`#define restrict restrict`改为`#define restrict`后就可以了。android系统有pthread，config.h需要打开pthread，`#define HAVE_PTHREADS 1`。这样就可以了，再在工程目录加入以下Android.mk，这个Android.mk的好处就是可以知道编译了什么文件，方便增删、改参数等操作。最后要生成avcodec.h。

<br/>

##Android.mk

<span class="label label-info">Info</span> codec_names.h如何生成？在ffmpeg源代码目录运行cat libavcodec/avcodec.h | libavcodec/codec_names.sh config.h libavcodec/codec_names.h

    :::c
    LOCAL_PATH:= $(call my-dir)
    
    AVUTIL_FILES := \
    	libavutil/adler32.c   \
    	libavutil/aes.c \
    	libavutil/audioconvert.c \
    	libavutil/audio_fifo.c \
    	libavutil/avstring.c \
    	libavutil/base64.c \
    	libavutil/bprint.c \
    	libavutil/cpu.c \
    	libavutil/crc.c \
    	libavutil/des.c \
    	libavutil/dict.c \
    	libavutil/error.c \
    	libavutil/eval.c \
    	libavutil/fifo.c \
    	libavutil/file.c \
    	libavutil/imgutils.c \
    	libavutil/intfloat_readwrite.c \
    	libavutil/inverse.c \
    	libavutil/lfg.c \
    	libavutil/lls.c \
    	libavutil/log.c \
    	libavutil/lzo.c \
    	libavutil/mathematics.c \
    	libavutil/md5.c \
    	libavutil/mem.c \
    	libavutil/opt.c \
    	libavutil/parseutils.c \
    	libavutil/pixdesc.c \
    	libavutil/random_seed.c \
    	libavutil/rational.c \
    	libavutil/rc4.c \
    	libavutil/samplefmt.c \
    	libavutil/sha.c \
    	libavutil/timecode.c \
    	libavutil/tree.c \
    	libavutil/utils.c \
    	libavutil/arm/cpu.c
    
    
    AVFORMAT_FILES := \
    	libavformat/4xm.c \
    	libavformat/a64.c \
    	libavformat/aacdec.c \
    	libavformat/ac3dec.c \
    	libavformat/act.c \
    	libavformat/adxdec.c \
    	libavformat/aea.c \
    	libavformat/aiffdec.c \
    	libavformat/allformats.c \
    	libavformat/amr.c \
    	libavformat/anm.c \
    	libavformat/apc.c \
    	libavformat/ape.c \
    	libavformat/apetag.c \
    	libavformat/asf.c \
    	libavformat/asfcrypt.c \
    	libavformat/asfdec.c \
    	libavformat/assdec.c \
    	libavformat/au.c \
    	libavformat/avidec.c \
    	libavformat/avio.c \
    	libavformat/aviobuf.c \
    	libavformat/avlanguage.c \
    	libavformat/avs.c \
    	libavformat/bethsoftvid.c \
    	libavformat/bfi.c \
    	libavformat/bink.c \
    	libavformat/bintext.c \
    	libavformat/bit.c \
    	libavformat/bmv.c \
    	libavformat/c93.c \
    	libavformat/caf.c \
    	libavformat/cache.c \
    	libavformat/cafdec.c \
    	libavformat/cavsvideodec.c \
    	libavformat/cdg.c \
    	libavformat/cdxl.c \
    	libavformat/concat.c \
    	libavformat/crypto.c \
    	libavformat/cutils.c \
    	libavformat/daud.c \
    	libavformat/dfa.c \
    	libavformat/diracdec.c \
    	libavformat/dnxhddec.c \
    	libavformat/dsicin.c \
    	libavformat/dtsdec.c \
    	libavformat/dv.c \
    	libavformat/dxa.c \
    	libavformat/eacdata.c \
    	libavformat/electronicarts.c \
    	libavformat/ffmdec.c \
    	libavformat/ffmetadec.c \
    	libavformat/file.c \
    	libavformat/filmstripdec.c \
    	libavformat/flacdec.c \
    	libavformat/flic.c \
    	libavformat/flvdec.c \
    	libavformat/framehash.c \
    	libavformat/g723_1.c \
    	libavformat/g729dec.c \
    	libavformat/gopher.c \
    	libavformat/gsmdec.c \
    	libavformat/gxf.c \
    	libavformat/h261dec.c \
    	libavformat/h263dec.c \
    	libavformat/h264dec.c \
    	libavformat/hls.c \
    	libavformat/hlsproto.c \
    	libavformat/http.c \
    	libavformat/httpauth.c \
    	libavformat/icodec.c \
    	libavformat/id3v1.c \
    	libavformat/id3v2.c \
    	libavformat/idcin.c \
    	libavformat/idroqdec.c \
    	libavformat/ingenientdec.c \
    	libavformat/ivfdec.c \
    	libavformat/iff.c \
    	libavformat/img2.c \
    	libavformat/img2dec.c \
    	libavformat/ipmovie.c \
    	libavformat/isom.c \
    	libavformat/iss.c \
    	libavformat/iv8.c \
    	libavformat/jacosubdec.c \
    	libavformat/jvdec.c \
    	libavformat/lmlm4.c \
    	libavformat/loasdec.c \
    	libavformat/lxfdec.c \
    	libavformat/m4vdec.c \
    	libavformat/matroska.c \
    	libavformat/matroskadec.c \
    	libavformat/md5proto.c \
    	libavformat/metadata.c \
    	libavformat/mgsts.c \
    	libavformat/microdvddec.c \
    	libavformat/mm.c \
    	libavformat/mmf.c \
    	libavformat/mms.c \
    	libavformat/mmsh.c \
    	libavformat/mmst.c \
    	libavformat/mov.c \
    	libavformat/mov_chan.c \
    	libavformat/mp3dec.c \
    	libavformat/mpc.c \
    	libavformat/mpc8.c \
    	libavformat/mpeg.c \
    	libavformat/mpegts.c \
    	libavformat/mpegvideodec.c \
    	libavformat/msnwc_tcp.c \
    	libavformat/mtv.c \
    	libavformat/mvi.c \
    	libavformat/mxg.c \
    	libavformat/mxf.c \
    	libavformat/mxfdec.c \
    	libavformat/ncdec.c \
    	libavformat/network.c \
    	libavformat/nsvdec.c \
    	libavformat/nut.c \
    	libavformat/nutdec.c \
    	libavformat/nuv.c \
    	libavformat/oggdec.c \
    	libavformat/oggparsecelt.c \
    	libavformat/oggparsedirac.c \
    	libavformat/oggparseflac.c \
    	libavformat/oggparseogm.c \
    	libavformat/oggparseskeleton.c \
    	libavformat/oggparsespeex.c \
    	libavformat/oggparsetheora.c \
    	libavformat/oggparsevorbis.c \
    	libavformat/oma.c \
    	libavformat/omadec.c \
    	libavformat/options.c \
    	libavformat/os_support.c \
    	libavformat/pcm.c \
    	libavformat/pcmdec.c \
    	libavformat/pmpdec.c \
    	libavformat/psxstr.c \
    	libavformat/pva.c \
    	libavformat/qcp.c \
    	libavformat/r3d.c \
    	libavformat/rawdec.c \
    	libavformat/rawvideodec.c \
    	libavformat/rdt.c \
    	libavformat/riff.c \
    	libavformat/rl2.c \
    	libavformat/rm.c \
    	libavformat/rmdec.c \
    	libavformat/rpl.c \
    	libavformat/rso.c \
    	libavformat/rsodec.c \
    	libavformat/rtmppkt.c \
    	libavformat/rtmpproto.c \
    	libavformat/rtp.c \
    	libavformat/rtpdec.c \
    	libavformat/rtpdec_amr.c \
    	libavformat/rtpdec_asf.c \
    	libavformat/rtpdec_g726.c \
    	libavformat/rtpdec_h263.c \
    	libavformat/rtpdec_h263_rfc2190.c \
    	libavformat/rtpdec_h264.c \
    	libavformat/rtpdec_xiph.c \
    	libavformat/rtpdec_latm.c \
    	libavformat/rtpdec_mpeg4.c \
    	libavformat/rtpdec_qcelp.c \
    	libavformat/rtpdec_qdm2.c \
    	libavformat/rtpdec_qt.c \
    	libavformat/rtpdec_svq3.c \
    	libavformat/rtpdec_vp8.c \
    	libavformat/rtpproto.c \
    	libavformat/rtsp.c \
    	libavformat/rtspdec.c \
    	libavformat/sbgdec.c \
    	libavformat/sdp.c \
    	libavformat/sapdec.c \
    	libavformat/sauce.c \
    	libavformat/seek.c \
    	libavformat/segafilm.c \
    	libavformat/sierravmd.c \
    	libavformat/siff.c \
    	libavformat/smacker.c \
    	libavformat/smjpeg.c \
    	libavformat/smjpegdec.c \
    	libavformat/sol.c \
    	libavformat/soxdec.c \
    	libavformat/spdif.c \
    	libavformat/spdifdec.c \
    	libavformat/srtdec.c \
    	libavformat/swfdec.c \
    	libavformat/tcp.c \
    	libavformat/thp.c \
    	libavformat/tiertexseq.c \
    	libavformat/tmv.c \
    	libavformat/tta.c \
    	libavformat/tty.c \
    	libavformat/txd.c \
    	libavformat/udp.c \
    	libavformat/utils.c \
    	libavformat/vc1test.c \
    	libavformat/voc.c \
    	libavformat/vocdec.c \
    	libavformat/vorbiscomment.c \
    	libavformat/vqf.c \
    	libavformat/wav.c \
    	libavformat/wc3movie.c \
    	libavformat/westwood_aud.c \
    	libavformat/westwood_vqa.c \
    	libavformat/wtv.c \
    	libavformat/wtvdec.c \
    	libavformat/wv.c \
    	libavformat/xa.c \
    	libavformat/xmv.c \
    	libavformat/xwma.c \
    	libavformat/yuv4mpeg.c \
    	libavformat/yop.c
    
    
    AVCODEC_FILES := \
    	libavcodec/4xm.c \
    	libavcodec/8bps.c \
    	libavcodec/8svx.c \
    	libavcodec/aacdec.c \
    	libavcodec/aac_ac3_parser.c \
    	libavcodec/aac_adtstoasc_bsf.c \
    	libavcodec/aac_parser.c \
    	libavcodec/aacadtsdec.c \
    	libavcodec/aacsbr.c \
    	libavcodec/aactab.c \
    	libavcodec/aacps.c \
    	libavcodec/aacpsdsp.c \
    	libavcodec/aandcttab.c \
    	libavcodec/ansi.c \
    	libavcodec/ass.c \
    	libavcodec/assdec.c \
    	libavcodec/ass_split.c \
    	libavcodec/aasc.c \
    	libavcodec/ac3.c \
    	libavcodec/ac3_parser.c \
    	libavcodec/ac3dec.c \
    	libavcodec/ac3dec_data.c \
    	libavcodec/ac3dsp.c \
    	libavcodec/ac3tab.c \
    	libavcodec/acelp_filters.c \
    	libavcodec/acelp_pitch_delay.c \
    	libavcodec/acelp_vectors.c \
    	libavcodec/adpcm.c \
    	libavcodec/adpcm_data.c \
    	libavcodec/adx.c \
    	libavcodec/adxdec.c \
    	libavcodec/adx_parser.c \
    	libavcodec/alac.c \
    	libavcodec/allcodecs.c \
    	libavcodec/alsdec.c \
    	libavcodec/amrnbdec.c \
    	libavcodec/amrwbdec.c \
    	libavcodec/anm.c \
    	libavcodec/apedec.c \
    	libavcodec/asv1.c \
    	libavcodec/atrac.c \
    	libavcodec/atrac1.c \
    	libavcodec/atrac3.c \
    	libavcodec/audioconvert.c \
    	libavcodec/audio_frame_queue.c \
    	libavcodec/aura.c \
    	libavcodec/avfft.c \
    	libavcodec/avpacket.c \
    	libavcodec/avs.c \
    	libavcodec/avuidec.c \
    	libavcodec/bethsoftvideo.c \
    	libavcodec/bfi.c \
    	libavcodec/bgmc.c \
    	libavcodec/bink.c \
    	libavcodec/binkaudio.c \
    	libavcodec/binkdsp.c \
    	libavcodec/bintext.c \
    	libavcodec/bitstream.c \
    	libavcodec/bitstream_filter.c \
    	libavcodec/bmp.c \
    	libavcodec/bmv.c \
    	libavcodec/c93.c \
    	libavcodec/cabac.c \
    	libavcodec/cavs.c \
    	libavcodec/cavs_parser.c \
    	libavcodec/cavsdec.c \
    	libavcodec/cavsdsp.c \
    	libavcodec/cdgraphics.c \
    	libavcodec/cdxl.c \
    	libavcodec/celp_filters.c \
    	libavcodec/celp_math.c \
    	libavcodec/cga_data.c \
    	libavcodec/chomp_bsf.c \
    	libavcodec/cinepak.c \
    	libavcodec/cljr.c \
    	libavcodec/cook.c \
    	libavcodec/cook_parser.c \
    	libavcodec/cscd.c \
    	libavcodec/cyuv.c \
    	libavcodec/dca.c \
    	libavcodec/dca_parser.c \
    	libavcodec/dcadsp.c \
    	libavcodec/dct.c \
    	libavcodec/dct32_fixed.c \
    	libavcodec/dct32_float.c \
    	libavcodec/dfa.c \
    	libavcodec/dirac.c \
    	libavcodec/dirac_arith.c \
    	libavcodec/diracdec.c \
    	libavcodec/diracdsp.c \
    	libavcodec/dirac_parser.c \
    	libavcodec/dnxhd_parser.c \
    	libavcodec/dnxhddata.c \
    	libavcodec/dnxhddec.c \
    	libavcodec/dpcm.c \
    	libavcodec/dpx.c \
    	libavcodec/dsicinav.c \
    	libavcodec/dsputil.c \
    	libavcodec/dump_extradata_bsf.c \
    	libavcodec/dv.c \
    	libavcodec/dvbsub_parser.c \
    	libavcodec/dvbsubdec.c \
    	libavcodec/dvdata.c \
    	libavcodec/dvdec.c \
    	libavcodec/dvdsub_parser.c \
    	libavcodec/dvdsubdec.c \
    	libavcodec/dv_profile.c \
    	libavcodec/dwt.c \
    	libavcodec/dxa.c \
    	libavcodec/dxtory.c \
    	libavcodec/eac3dec.c \
    	libavcodec/eac3_data.c \
    	libavcodec/eacmv.c \
    	libavcodec/eaidct.c \
    	libavcodec/eamad.c \
    	libavcodec/eatgq.c \
    	libavcodec/eatgv.c \
    	libavcodec/eatqi.c \
    	libavcodec/error_resilience.c \
    	libavcodec/escape124.c \
    	libavcodec/escape130.c \
    	libavcodec/exr.c \
    	libavcodec/faanidct.c \
    	libavcodec/faxcompr.c \
    	libavcodec/fft_fixed.c \
    	libavcodec/fft_float.c \
    	libavcodec/ffv1.c \
    	libavcodec/ffwavesynth.c \
    	libavcodec/flac.c \
    	libavcodec/flacdata.c \
    	libavcodec/flacdec.c \
    	libavcodec/flac_parser.c \
    	libavcodec/flashsv.c \
    	libavcodec/flicvideo.c \
    	libavcodec/flvdec.c \
    	libavcodec/fmtconvert.c \
    	libavcodec/fraps.c \
    	libavcodec/frwu.c \
    	libavcodec/g722.c \
    	libavcodec/g722dec.c \
    	libavcodec/g723_1.c \
    	libavcodec/g726.c \
    	libavcodec/g729dec.c \
    	libavcodec/g729postfilter.c \
    	libavcodec/gifdec.c \
    	libavcodec/golomb.c \
    	libavcodec/golomb-test.c \
    	libavcodec/gsmdec.c \
    	libavcodec/gsmdec_data.c \
    	libavcodec/gsm_parser.c \
    	libavcodec/h261.c \
    	libavcodec/h261data.c \
    	libavcodec/h261_parser.c \
    	libavcodec/h261dec.c \
    	libavcodec/h263.c \
    	libavcodec/h263_parser.c \
    	libavcodec/h263dec.c \
    	libavcodec/h264.c \
    	libavcodec/h264_cabac.c \
    	libavcodec/h264_cavlc.c \
    	libavcodec/h264_direct.c \
    	libavcodec/h264_loopfilter.c \
    	libavcodec/h264_mp4toannexb_bsf.c \
    	libavcodec/h264_parser.c \
    	libavcodec/h264_ps.c \
    	libavcodec/h264_refs.c \
    	libavcodec/h264_sei.c \
    	libavcodec/h264dsp.c \
    	libavcodec/h264idct.c \
    	libavcodec/h264pred.c \
    	libavcodec/huffman.c \
    	libavcodec/huffyuv.c \
    	libavcodec/idcinvideo.c \
    	libavcodec/iff.c \
    	libavcodec/imgconvert.c \
    	libavcodec/imc.c \
    	libavcodec/imx_dump_header_bsf.c \
    	libavcodec/indeo2.c \
    	libavcodec/indeo3.c \
    	libavcodec/indeo4.c \
    	libavcodec/indeo5.c \
    	libavcodec/intelh263dec.c \
    	libavcodec/interplayvideo.c \
    	libavcodec/intrax8.c \
    	libavcodec/intrax8dsp.c \
    	libavcodec/ituh263dec.c \
    	libavcodec/ivi_common.c \
    	libavcodec/ivi_dsp.c \
    	libavcodec/j2k.c \
    	libavcodec/j2k_dwt.c \
    	libavcodec/j2kdec.c \
    	libavcodec/jacosubdec.c \
    	libavcodec/jpegls.c \
    	libavcodec/jpeglsdec.c \
    	libavcodec/jrevdct.c \
    	libavcodec/jvdec.c \
    	libavcodec/kbdwin.c \
    	libavcodec/kgv1dec.c \
    	libavcodec/kmvc.c \
    	libavcodec/lagarith.c \
    	libavcodec/lagarithrac.c \
    	libavcodec/latm_parser.c \
    	libavcodec/lcldec.c \
    	libavcodec/loco.c \
    	libavcodec/lpc.c \
    	libavcodec/lsp.c \
    	libavcodec/lzw.c \
    	libavcodec/mace.c \
    	libavcodec/mdct_fixed.c \
    	libavcodec/mdct_float.c \
    	libavcodec/mdec.c \
    	libavcodec/microdvddec.c \
    	libavcodec/mimic.c \
    	libavcodec/mjpeg.c \
    	libavcodec/mjpeg2jpeg_bsf.c \
    	libavcodec/mjpeg_parser.c \
    	libavcodec/mjpega_dump_header_bsf.c \
    	libavcodec/mjpegbdec.c \
    	libavcodec/mjpegdec.c \
    	libavcodec/mlp.c \
    	libavcodec/mlp_parser.c \
    	libavcodec/mlpdec.c \
    	libavcodec/mlpdsp.c \
    	libavcodec/mmvideo.c \
    	libavcodec/motionpixels.c \
    	libavcodec/movsub_bsf.c \
    	libavcodec/mp3_header_compress_bsf.c \
    	libavcodec/mp3_header_decompress_bsf.c \
    	libavcodec/mpc.c \
    	libavcodec/mpc7.c \
    	libavcodec/mpc8.c \
    	libavcodec/mpeg12.c \
    	libavcodec/mpeg12data.c \
    	libavcodec/mpeg4audio.c \
    	libavcodec/mpeg4video.c \
    	libavcodec/mpeg4video_parser.c \
    	libavcodec/mpeg4videodec.c \
    	libavcodec/mpegaudio.c \
    	libavcodec/mpegaudio_parser.c \
    	libavcodec/mpegaudiodata.c \
    	libavcodec/mpegaudiodec.c \
    	libavcodec/mpegaudiodec_float.c \
    	libavcodec/mpegaudiodecheader.c \
    	libavcodec/mpegaudiodsp.c \
    	libavcodec/mpegaudiodsp_fixed.c \
    	libavcodec/mpegaudiodsp_float.c \
    	libavcodec/mpegvideo.c \
    	libavcodec/mpegvideo_parser.c \
    	libavcodec/mqc.c \
    	libavcodec/mqcdec.c \
    	libavcodec/msgsmdec.c \
    	libavcodec/msmpeg4.c \
    	libavcodec/msmpeg4data.c \
    	libavcodec/msrle.c \
    	libavcodec/msrledec.c \
    	libavcodec/msvideo1.c \
    	libavcodec/mxpegdec.c \
    	libavcodec/nellymoser.c \
    	libavcodec/nellymoserdec.c \
    	libavcodec/noise_bsf.c \
    	libavcodec/nuv.c \
    	libavcodec/options.c \
    	libavcodec/parser.c \
    	libavcodec/pcm-mpeg.c \
    	libavcodec/pcm.c \
    	libavcodec/pcx.c \
    	libavcodec/pgssubdec.c \
    	libavcodec/pictordec.c \
    	libavcodec/png.c \
    	libavcodec/pngdsp.c \
    	libavcodec/png_parser.c \
    	libavcodec/pngdec.c \
    	libavcodec/pnm.c \
    	libavcodec/pnm_parser.c \
    	libavcodec/proresdata.c \
    	libavcodec/pnmdec.c \
    	libavcodec/proresdec_lgpl.c \
    	libavcodec/proresdec2.c \
    	libavcodec/proresdsp.c \
    	libavcodec/pthread.c \
    	libavcodec/ptx.c \
    	libavcodec/qcelpdec.c \
    	libavcodec/qdm2.c \
    	libavcodec/qdrw.c \
    	libavcodec/qpeg.c \
    	libavcodec/qtrle.c \
    	libavcodec/r210dec.c \
    	libavcodec/ra144.c \
    	libavcodec/ra144dec.c \
    	libavcodec/ra288.c \
    	libavcodec/ralf.c \
    	libavcodec/rangecoder.c \
    	libavcodec/raw.c \
    	libavcodec/rawdec.c \
    	libavcodec/rdft.c \
    	libavcodec/remove_extradata_bsf.c \
    	libavcodec/resample.c \
    	libavcodec/resample2.c \
    	libavcodec/rl2.c \
    	libavcodec/roqvideo.c \
    	libavcodec/roqvideodec.c \
    	libavcodec/rpza.c \
    	libavcodec/rtjpeg.c \
    	libavcodec/rv10.c \
    	libavcodec/rv30.c \
    	libavcodec/rv30dsp.c \
    	libavcodec/rv34.c \
    	libavcodec/rv34_parser.c \
    	libavcodec/rv34dsp.c \
    	libavcodec/rv40.c \
    	libavcodec/rv40dsp.c \
    	libavcodec/s302m.c \
    	libavcodec/s3tc.c \
    	libavcodec/sbrdsp.c \
    	libavcodec/sgidec.c \
    	libavcodec/shorten.c \
    	libavcodec/simple_idct.c \
    	libavcodec/sinewin.c \
    	libavcodec/sipr.c \
    	libavcodec/sipr16k.c \
    	libavcodec/smacker.c \
    	libavcodec/smc.c \
    	libavcodec/snow.c \
    	libavcodec/snowdec.c \
    	libavcodec/sonic.c \
    	libavcodec/sp5xdec.c \
    	libavcodec/srtdec.c \
    	libavcodec/sunrast.c \
    	libavcodec/sunrastenc.c \
    	libavcodec/svq1.c \
    	libavcodec/svq1dec.c \
    	libavcodec/svq3.c \
    	libavcodec/synth_filter.c \
    	libavcodec/targa.c \
    	libavcodec/tiertexseqv.c \
    	libavcodec/tiff.c \
    	libavcodec/tmv.c \
    	libavcodec/truemotion1.c \
    	libavcodec/truemotion2.c \
    	libavcodec/truespeech.c \
    	libavcodec/tscc.c \
    	libavcodec/tta.c \
    	libavcodec/twinvq.c \
    	libavcodec/txd.c \
    	libavcodec/ulti.c \
    	libavcodec/utils.c \
    	libavcodec/utvideo.c \
    	libavcodec/v210dec.c \
    	libavcodec/v210x.c \
    	libavcodec/v308dec.c \
    	libavcodec/v408dec.c \
    	libavcodec/v410dec.c \
    	libavcodec/vb.c \
    	libavcodec/vble.c \
    	libavcodec/vc1.c \
    	libavcodec/vc1_parser.c \
    	libavcodec/vc1data.c \
    	libavcodec/vc1dec.c \
    	libavcodec/vc1dsp.c \
    	libavcodec/vcr1.c \
    	libavcodec/vmdav.c \
    	libavcodec/vmnc.c \
    	libavcodec/vorbis.c \
    	libavcodec/vorbis_parser.c \
    	libavcodec/vorbis_data.c \
    	libavcodec/vorbisdec.c \
    	libavcodec/vp3.c \
    	libavcodec/vp3_parser.c \
    	libavcodec/vp3dsp.c \
    	libavcodec/vp5.c \
    	libavcodec/vp56.c \
    	libavcodec/vp56data.c \
    	libavcodec/vp56dsp.c \
    	libavcodec/vp56rac.c \
    	libavcodec/vp6.c \
    	libavcodec/vp6dsp.c \
    	libavcodec/vp8.c \
    	libavcodec/vp8_parser.c \
    	libavcodec/vp8dsp.c \
    	libavcodec/vqavideo.c \
    	libavcodec/wavpack.c \
    	libavcodec/wma.c \
    	libavcodec/wma_common.c \
    	libavcodec/wmadec.c \
    	libavcodec/wmalosslessdec.c \
    	libavcodec/wmaprodec.c \
    	libavcodec/wmavoice.c \
    	libavcodec/wmv2.c \
    	libavcodec/wmv2dec.c \
    	libavcodec/wnv1.c \
    	libavcodec/ws-snd1.c \
    	libavcodec/xan.c \
    	libavcodec/xbmdec.c \
    	libavcodec/xiph.c \
    	libavcodec/xl.c \
    	libavcodec/xsubdec.c \
    	libavcodec/xwddec.c \
    	libavcodec/yuv4dec.c \
    	libavcodec/zerocodec.c \
    	libavcodec/xxan.c \
    	libavcodec/y41pdec.c \
    	libavcodec/yop.c \
    	libavcodec/zmbv.c 
    
    
    ARM_GENERAL_AVCODEC_FILES := \
    	libavcodec/arm/asm.S \
    	libavcodec/arm/aacpsdsp_init_arm.c \
    	libavcodec/arm/aacpsdsp_neon.S \
    	libavcodec/arm/ac3dsp_arm.S \
    	libavcodec/arm/ac3dsp_init_arm.c \
    	libavcodec/arm/dcadsp_init_arm.c \
    	libavcodec/arm/dsputil_arm.S \
    	libavcodec/arm/dsputil_init_arm.c \
    	libavcodec/arm/dsputil_init_armv5te.c \
    	libavcodec/arm/fft_init_arm.c \
    	libavcodec/arm/fft_fixed_init_arm.c \
    	libavcodec/arm/fmtconvert_init_arm.c \
    	libavcodec/arm/h264dsp_init_arm.c \
    	libavcodec/arm/h264pred_init_arm.c \
    	libavcodec/arm/jrevdct_arm.S \
    	libavcodec/arm/mpegaudiodsp_init_arm.c \
    	libavcodec/arm/mpegvideo_arm.c \
    	libavcodec/arm/mpegvideo_armv5te.c \
    	libavcodec/arm/mpegvideo_armv5te_s.S \
    	libavcodec/arm/simple_idct_arm.S \
    	libavcodec/arm/simple_idct_armv5te.S \
    	libavcodec/arm/vp56dsp_init_arm.c \
    	libavcodec/arm/vp8dsp_init_arm.c
    
    
    ARM_VFP_AVCODEC_FILES := \
    	libavcodec/arm/dsputil_init_vfp.c \
    	libavcodec/arm/dsputil_vfp.S \
    	libavcodec/arm/fmtconvert_vfp.S
    
    
    ARM_ARMV6_AVCODEC_FILES := \
    	libavcodec/arm/ac3dsp_armv6.S \
    	libavcodec/arm/dsputil_armv6.S \
    	libavcodec/arm/dsputil_init_armv6.c \
    	libavcodec/arm/mpegaudiodsp_fixed_armv6.S \
    	libavcodec/arm/simple_idct_armv6.S \
    	libavcodec/arm/vp8_armv6.S \
    	libavcodec/arm/vp8dsp_armv6.S \
    	libavcodec/arm/vp8dsp_init_armv6.c 
    
    
    ARM_NEON_AVCODEC_FILES := \
    	libavcodec/arm/ac3dsp_neon.S \
    	libavcodec/arm/dcadsp_neon.S \
    	libavcodec/arm/dsputil_init_neon.c \
    	libavcodec/arm/dsputil_neon.S \
    	libavcodec/arm/fft_neon.S \
    	libavcodec/arm/fft_fixed_neon.S \
    	libavcodec/arm/fmtconvert_neon.S \
    	libavcodec/arm/h264dsp_neon.S \
    	libavcodec/arm/h264idct_neon.S \
    	libavcodec/arm/h264cmc_neon.S \
    	libavcodec/arm/h264pred_neon.S \
    	libavcodec/arm/int_neon.S \
    	libavcodec/arm/mdct_neon.S \
    	libavcodec/arm/mdct_fixed_neon.S \
    	libavcodec/arm/mpegvideo_neon.S \
    	libavcodec/arm/neon.S \
    	libavcodec/arm/rdft_neon.S \
    	libavcodec/arm/rv34dsp_init_neon.c \
    	libavcodec/arm/rv34dsp_neon.S \
    	libavcodec/arm/rv40dsp_init_neon.c \
    	libavcodec/arm/rv40dsp_neon.S \
    	libavcodec/arm/simple_idct_neon.S \
    	libavcodec/arm/synth_filter_neon.S \
    	libavcodec/arm/vp3dsp_neon.S \
    	libavcodec/arm/vp56dsp_neon.S \
    	libavcodec/arm/vp8dsp_neon.S \
    	libavcodec/arm/sbrdsp_init_arm.c \
    	libavcodec/arm/sbrdsp_neon.S \
    	libavcodec/arm/vp8dsp_init_neon.c 
    
    
    
    SWSCALE_FILES := \
    	libswscale/options.c \
    	libswscale/rgb2rgb.c \
    	libswscale/swscale.c \
    	libswscale/output.c \
    	libswscale/input.c \
    	libswscale/swscale_unscaled.c \
    	libswscale/utils.c \
    	libswscale/yuv2rgb.c 
    
    SWRESAMPLE_FILES := \
    	libswresample/audioconvert.c \
    	libswresample/dither.c \
    	libswresample/rematrix.c \
    	libswresample/resample.c \
    	libswresample/swresample.c 
    
    
    # FFMPEG NEON
    include $(CLEAR_VARS)
    
    LOCAL_MODULE := ffmpeg
    LOCAL_LDLIBS += -llog -lz -lm
    LOCAL_CFLAGS += -DHAVE_AV_CONFIG_H -D_FILE_OFFSET_BITS=64 -D_LARGEFILE_SOURCE -DPIC -DHAVE_SYS_UIO_H=1 -DANDROID
    LOCAL_CFLAGS += $(CC_OPTIMIZE_FLAG)
    LOCAL_CFLAGS += -march=armv7-a -mfpu=neon -mfloat-abi=softfp -mvectorize-with-neon-quad
    LOCAL_CXXFLAGS += -D__STDC_CONSTANT_MACROS
    LOCAL_LDFLAGS += -Wl,--fix-cortex-a8
    
    TARGET_ARCH_ABI := armeabi-v7a
    LOCAL_ARM_MODE := arm
    LOCAL_ARM_NEON := true
    
    LOCAL_SRC_FILES += $(AVCORE_FILES) \
    									 $(AVUTIL_FILES) \
    									 $(AVFORMAT_FILES) \
    									 $(AVCODEC_FILES) \
    									 $(ARM_GENERAL_AVCODEC_FILES) \
    									 $(ARM_ARMV6_AVCODEC_FILES) \
    									 $(ARM_VFP_AVCODEC_FILES) \
    									 $(ARM_NEON_AVCODEC_FILES) \
    									 $(SWSCALE_FILES) \
    									 $(SWRESAMPLE_FILES)
    
    LOCAL_C_INCLUDES +=$(LOCAL_PATH) \
    	$(LOCAL_PATH)/libavcodec \
    	$(LOCAL_PATH)/libavformat \
    	$(LOCAL_PATH)/libavutil \
    	$(LOCAL_PATH)/libswscale \
    	$(LOCAL_PATH)/libswresample \
    	$(LOCAL_PATH)/libavutil/arm 
    
    include $(BUILD_SHARED_LIBRARY)




[1]: http://www.ffmpeg.org/download.html
