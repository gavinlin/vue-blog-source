---
title: 向android系统添加ape音频格式（第二部分）
date: 2013-09-10 
category: Android
---

编译出了`libapedec.so`，后需要知道如何用这个库。
<!-- excerpt -->

主要还是看`MACLib.h`这个头文件，因为我们要调的主要接口都在这个头文件中。其中有以下几个

    :::c
        APE::IAPEDecompress * __stdcall CreateIAPEDecompress(const APE::str_utfn * pFilename, int * pErrorCode = NULL);

传入参数是文件名和错误代码的int指针，返回的是这个解码器的句柄`IAPEDecompress`。先获得APE的音频数据及相关信息，就要先调用此函数初始化解码器并获得其句柄。


    :::c
    class IAPEDecompress
    {
    public:
        // destructor (needed so implementation's destructor will be called)
        virtual ~IAPEDecompress() {};
        virtual int GetData(char * pBuffer, int nBlocks, int * pBlocksRetrieved) = 0;
        virtual int Seek(int nBlockOffset) = 0;
        virtual intn GetInfo(APE_DECOMPRESS_FIELDS Field, intn nParam1 = 0, intn nParam2 = 0) = 0;
    };

获得了句柄后，就可以使用里面的函数获取我们需要的信息，例如`GetInfo`可以获得APE文件的相关信息，`GetData`可以获得解码后的数据，`Seek`可以跳转到APE数据的指定位置。

只要用好这些方法，就可以了，问题是如何在stagefright上面用。

<!-- more -->

##StageFright

android 4.2 中音视频处理的代码都集中在 `frameworks/av/media` 这个目录中，可以看到里面有很多目录，主要关注两个 `libmediaplayerservice` 和 `libstagefright`

先看 `MediaPlayerFactory.cpp` 其决定了下层用什么播放器来播放音视频文件。我们增加以下代码。

    :::cpp
    //MediaPlayerFactory.cpp
    class StagefrightPlayerFactory :
        public MediaPlayerFactory::IFactory {
        ......

            // APE
            if (ident == 0x2043414D)
            {
                ALOGV("is APE");
                return 1.0;
            
            }

            return 0.0;
        }

        ......
    };

这样就保证了使用 stagefright 来播放APE。

<br/>

接着到 `libstagefright` 目录。

改造一下 `FileSource.cpp` 使其可以让我们获得文件路径，因为 APE 解码库需要这个路径。

    :::cpp
    //FileSource.cpp
        if( offset == 9223372036854775807UL)
        {
            ALOGE("file uri len %d is %s", mFilenameLength, mFileUri);
            memcpy(data, mFileUri, mFilenameLength);
            return OK;	
        }


这里算是 cheat 。因为不想增加方法。

    :::cpp
    //ACodec.cpp
    status_t ACodec::configureCodec(
            const char *mime, const sp<AMessage> &msg) {
    ......
                    }
                }
                err = setupFlacCodec(encoder, numChannels, sampleRate, compressionLevel);
            }
        }else if (!strcasecmp(mime, "audio/ape")){
            return OK;
            
        } else if (!strcasecmp(mime, MEDIA_MIMETYPE_AUDIO_RAW)) {

    ......

    :::cpp
    //DataSource.cpp
    void DataSource::RegisterDefaultSniffers() {
    ......
        RegisterSniffer(SniffAPE);
    ......
    }

这个 `SniffAPE` 函数是在 `APEExtractor.cpp` 里面添加的。

做完这些就差不多了。最后是添加我们的解码器


    :::cpp
    //APEExtractor.cpp
    namespace android{


    class APESource : public MediaSource{

        public:
            APESource(
                    const sp<DataSource> &dataSource,
                    const sp<MetaData> &trackMetadata,
                    APE::IAPEDecompress *apeDec);

            virtual status_t start(MetaData *params);
            virtual status_t stop();
            virtual sp<MetaData> getFormat();

            virtual status_t read(
                    MediaBuffer **buffer,
                    const ReadOptions *options = NULL);

        protected:
            virtual ~APESource();

        private:
            sp<DataSource> mDataSource;
            sp<MetaData> mTrackMetadata;
            int32_t mSampleRate;
            int32_t mNumChannels;
            int32_t mBitsPerSample;
            int32_t mBlockSize;
            off64_t mOffset;
            size_t mSize;
            off64_t mCurrentPos;
            MediaBufferGroup *mGroup;
            bool mInitCheck;
            bool mStarted;
            APE::IAPEDecompress *mApeDec;

            status_t init();

            APESource(const APESource&);
            APESource &operator=(const APESource &);
    };

    APESource::APESource(
            const sp<DataSource> &dataSource,
            const sp<MetaData> &trackMetadata,
            APE::IAPEDecompress *apeDec)
        : mDataSource(dataSource),
          mTrackMetadata(trackMetadata),
          mInitCheck(false),
          mStarted(false)
    {
        ALOGI("APESource::APESource");
        mApeDec = apeDec;
        mInitCheck = init();
    }

    APESource::~APESource()
    {
        ALOGI("APESource::~APESource");
        if(mStarted)
        {
            stop();
        }
    }

    status_t APESource::start(MetaData *params)
    {
        ALOGI("APESource::start");

        mGroup = new MediaBufferGroup;
        mGroup->add_buffer(new MediaBuffer(mBlockSize * mApeDec->GetInfo(APE::APE_INFO_BLOCK_ALIGN) ));

        mCurrentPos = mOffset;

        mStarted = true;
        return OK;
    }

    status_t APESource::stop()
    {
        ALOGI("APESource::stop");

        delete mGroup;
        mGroup = NULL;

        mStarted = false;
        return OK;
    }

    sp<MetaData> APESource::getFormat()
    {
        return mTrackMetadata;
    }

    status_t APESource::read(
            MediaBuffer **outBuffer, const ReadOptions *options)
    {
    //	ALOGI("APESource::read");

        *outBuffer = NULL;
        int trueblock;
        int numread = 0;

        int64_t seekTimeUs;
        ReadOptions::SeekMode mode;
        if(options != NULL && options->getSeekTo(&seekTimeUs, &mode))
        {
            //current_block = (currentus / 1000) * SAMPLERATE / 1000
            int64_t pos = seekTimeUs * mSampleRate / 1000000 ;
            mApeDec->Seek((int)pos);

        }

        MediaBuffer *buffer;
        status_t err = mGroup->acquire_buffer(&buffer);
        if (err != OK) {
            return err;
        }

        char *dst = (char *)buffer->data();
        mApeDec->GetData( dst, mBlockSize, &trueblock);
        numread = trueblock * mApeDec->GetInfo(APE::APE_INFO_BLOCK_ALIGN);

    //	ALOGE("get trueblock is %d, numread is %d, mBlockSize is %d"
    //			, trueblock, numread, mBlockSize );

        if(numread <= 0)
        {
            buffer->release();
            buffer = NULL;
            return ERROR_END_OF_STREAM;
        }

        buffer->meta_data()->setInt64(
                kKeyTime,
                mApeDec->GetInfo(APE::APE_DECOMPRESS_CURRENT_MS) * 1000LL
                );
        buffer->meta_data()->setInt32(kKeyIsSyncFrame, 1);

        *outBuffer = buffer;

    /*********************************/
        return OK;
    }

    status_t APESource::init()
    {
        ALOGI("APESource::init");
        mSampleRate = mApeDec->GetInfo(APE::APE_INFO_SAMPLE_RATE);
        mBitsPerSample = mApeDec->GetInfo(APE::APE_INFO_BITS_PER_SAMPLE);
        mNumChannels = mApeDec->GetInfo(APE::APE_INFO_CHANNELS);

        switch(mApeDec->GetInfo(APE::APE_INFO_COMPRESSION_LEVEL))
        {
            case COMPRESSION_LEVEL_FAST:
            case COMPRESSION_LEVEL_NORMAL:
            case COMPRESSION_LEVEL_HIGH:
                mBlockSize = (int32_t)(20 *
                        mApeDec->GetInfo(APE::APE_INFO_SAMPLE_RATE)/1000.0);
                break;
            case COMPRESSION_LEVEL_EXTRA_HIGH:
                mBlockSize = (int32_t)(10 *
                        mApeDec->GetInfo(APE::APE_INFO_SAMPLE_RATE)/1000.0);
                break;
            default:
                mBlockSize = (int32_t)(10 *
                        mApeDec->GetInfo(APE::APE_INFO_SAMPLE_RATE)/1000.0);
                break;
        }


        ALOGV("mSampleRate %d",mSampleRate);
        ALOGV("mBitsPerSample %d",mBitsPerSample);
        ALOGV("mNumChannels %d",mNumChannels);
        ALOGV("mBlockSize %d",mBlockSize);

        mTrackMetadata->setCString(kKeyMIMEType, MEDIA_MIMETYPE_AUDIO_RAW);
        mTrackMetadata->setInt32(kKeyMaxInputSize, mBlockSize);

        return OK;
    }


    APEExtractor::APEExtractor(const sp<DataSource> &source)
        :mDataSource(source)
    {
        mInitCheck = init();
    }

    APEExtractor::~APEExtractor()
    {
        if(mApeDec)
        {
            delete mApeDec;
            mApeDec = NULL;
        }
    }

    sp<MetaData> APEExtractor::getMetaData()
    {
        ALOGI("APEExtractor::getMetaData");
        sp<MetaData> meta = new MetaData;
        if(mInitCheck != OK)
        {
            return meta;
        }

        meta->setCString(kKeyMIMEType, "audio/ape");
        return meta;
    }

    size_t APEExtractor::countTracks()
    {
        ALOGI("APEExtractor::countTracks mInitCheck is %d", mInitCheck == OK ? 1 : 0);
        return mInitCheck == OK ? 1 : 0;
    }

    sp<MediaSource> APEExtractor::getTrack(size_t index)
    {
        ALOGI("APEExtractor::getTrack");
        if(mInitCheck != OK || index > 0)
        {
            return NULL;
        }

        return new APESource(
                mDataSource, mTrackMeta,mApeDec);
    }

    sp<MetaData> APEExtractor::getTrackMetaData(
            size_t index, uint32_t flags)
    {
        ALOGI("APEExtractor::getTrackMetaData");
        if(mInitCheck != OK || index > 0)
        {
            return NULL;
        }
        return mTrackMeta;
    }


    status_t APEExtractor::init()
    {
        int nRetVal;
        ALOGV("APEExtractor::init");
        char mFilePath[1024];
        memset(mFilePath, 0, sizeof(mFilePath));
        mDataSource->readAt(9223372036854775807UL, mFilePath, 0);
        ALOGV("file path is %s", mFilePath);

        mApeDec = CreateIAPEDecompress(APE::CAPECharacterHelper::GetUTF16FromANSI(mFilePath), &nRetVal);

        if(mApeDec == NULL || nRetVal != 0)
        {
            ALOGE("APE init error, nRetVal=%d",nRetVal);
            if(mApeDec)
            {
                delete mApeDec;
                mApeDec = NULL;
            }
            return NO_INIT;
        }

        mSampleRate = mApeDec->GetInfo(APE::APE_INFO_SAMPLE_RATE);
        mBitsPerSample = mApeDec->GetInfo(APE::APE_INFO_BITS_PER_SAMPLE);
        mNumChannels = mApeDec->GetInfo(APE::APE_INFO_CHANNELS);
        mChannelMask = CHANNEL_MASK_USE_CHANNEL_ORDER;

        mTrackMeta = new MetaData;

        mTrackMeta->setCString(kKeyMIMEType, "audio/ape");

        mTrackMeta->setInt32(kKeyChannelCount, mNumChannels);
        mTrackMeta->setInt32(kKeyChannelMask, mChannelMask);
        mTrackMeta->setInt32(kKeySampleRate, mSampleRate);
        mTrackMeta->setInt32(kKeyBitRate, mBitsPerSample);

        int64_t durationUs = 1000LL * mApeDec->GetInfo(APE::APE_DECOMPRESS_LENGTH_MS);
        mTrackMeta->setInt64(kKeyDuration, durationUs);

        return OK;
    }

    bool SniffAPE(
            const sp<DataSource> &source, String8 *mimeType, float *confidence,
            sp<AMessage> *)
    {
        uint8_t header[8];
        long ident;

        if(source->readAt(0, header, sizeof(header)) != sizeof(header))
        {
            return false;
        }

        ident = *((long*)header);

        if(ident != 0x2043414D)
        {
            return false;
        }

        *mimeType = "audio/ape";
        *confidence = 0.5;

        return true;

    }

    }//namespace android
