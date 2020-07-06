---
title: 向android frameworks 添加自定义系统服务
date: 2013-09-25
category: android
tags: android
---

以添加控制LED的服务为例子简述 android 系统服务的编写。
<!-- excerpt -->

<br/>

##分析

首先需要知道的是android 之所以能够提供系统服务，是因为进程间通信，系统服务一直在后台运行。当程序想使用这些服务的时候，只要申请使用就可以了。android使用自己的Binder实现了这种进程间通行。

我想实现一个控制LED的系统服务，但是不需要过度了解 Binder, 只需要在系统服务的框架中添加相应的代码就可以了。

![structure]({{BASE_PATH}}/images/system_service_manager.png)

<br/>

##实现

<br/>

###定义接口

aidl文件定义了服务端提供的接口

    :::java
    //ILEDManager.aidl
    package android.app;

    interface ILEDManager {
        void setWifiLED(boolean flag);
        void setApLED(boolean flag);
    }

定义好好需要在编译文件添加文件。

    :::java
    //Android.mk
        core/java/android/app/ILEDManager.aidl \


<br/>

###接口实现代码

定义了接口后，就需要实现服务端逻辑了。

    :::java
    //LEDManagerService.java
    package com.android.server;

    import android.util.Slog;
    import android.os.IBinder;
    import android.os.Binder;
    import android.content.Context;

    import android.app.LEDManager;
    import android.app.ILEDManager;

    public class LEDManagerService extends ILEDManager.Stub{
        private static final String TAG = "LEDManagerService";

        private LightsService mLightsService;
        private Context mContext;
        private final LightsService.Light mWifiLight;
        private final LightsService.Light mApLight;

        public LEDManagerService(Context context, LightsService lights){
            this.mContext = context;
            this.mLightsService = lights;
            mWifiLight = mLightsService.getLight(LightsService.LIGHT_ID_WIFI);
            mApLight = mLightsService.getLight(LightsService.LIGHT_ID_AP);
        }


        public void setWifiLED(boolean flag){
            if(mWifiLight != null){
                if(flag){
                    mWifiLight.setColor(1);
                }else{
                    mWifiLight.turnOff();
                }
            }
        }

        public void setApLED(boolean flag){
            if(mApLight != null){
                if(flag){
                    mApLight.setColor(1);
                }else{
                    mApLight.turnOff();
                }
            }

        }

    }

<br/>

###加载

服务端的代码实现了，但是还需要在系统启动的时候启动服务，所以需要加上下面代码。

    :::java
    //SystemServer.java
    try{
        Slog.i(TAG, "LEDManager Service");
        led = new LEDManagerService(context, lights);
        ServiceManager.addService("led", led);
    } catch (Throwable e){
        reportWtf("starting LED service",e);
    }

<br/>

###客户端实现

    :::java
    //LEDManager.java
    package android.app;

    import android.os.RemoteException;

    public class LEDManager
    {

        private final ILEDManager mService;
        LEDManager (ILEDManager mService){
            mService = service;
        }

        public void setWifiLED(boolean flag){
            try {
                mService.setWifiLED(flag);
            } catch (RemoteException ex) {
            }
        }

        public void setApLED(boolean flag){
            try {
                mService.setApLED(flag);
            } catch (RemoteException ex) {
            }
        }
    }

<br/>

    :::java
    //ContexImpl.java
    registerService("led", new StaticServiceFetcher(){
            public Object createStaticService(){
            IBinder b = ServiceManager.getService("led");
            ILEDManager service = ILEDManager.Stub.asInterface(b);
            return new LEDManager(service);
            }
            });

只要加上了服务端的接口，我们就可以很方便的使用系统服务了。

    :::java
    LEDManager lm = (LEDManader) getSystemService("led");
    lm.setWifiLED(true);

