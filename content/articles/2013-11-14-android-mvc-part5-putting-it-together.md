---
title: Android MVC Part5 Putting It Together
date: 2013-11-14
category: android
tags: mvc
Parts: mvc
---

之前我们已经讨论了模型（Models），视图（Views）和控制器（Controllers）。现在我们需要把他们组合在一次成为真正符合 MVC 规范。
<!-- excerpt -->

Activity 是我们程序的每一个新视图的入口， 我们需要在 Activity 的 `onCreate` 方法实例化模型和控制器。其实一个典型的 MVC 模型视图是不会去实例化模型和控制器的，但是..android 有点特殊。模型一旦创建了，我们需要把 activity 注册到模型中。再者，如果你想从控制器获取消息，你也需要向控制器注册。让我们看看他们长什么模样的。

##Activity:

    :::java
    @Override
    public void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.main);
     
        counter = new CounterVo();
        counter.addListener(this);
        controller = new TapController(counter);
        controller.addOutboxHandler(new Handler(this));
        // ... other set up code like referencing widgets/views ...
    }

这里视图在模型和控制器上都注册了。让我们看看模型对于视图的回调函数。

    :::java
    @Override
    public void onChange(CounterVo counter) {
        mHandler.sendEmptyMessage(UPDATE_VIEW);
    }

一旦模型通知视图有改变，我们就在 UI 线程修改 UI控件。这就是你数据的绑定。现在让我们来看看处理从控制器发来的信息。

    :::java
    @Override
    public boolean handleMessage(Message msg) {
        switch(msg.what) {
            case TapListController.MESSAGE_MODEL_UPDATED:
                runOnUiThread(new Runnable() {
                    @Override
                    public void run() {
                        adapter.notifyDataSetChanged();
                    }
                });
                return true;
        }
        return false;
    }

就跟之前讨论的那样，这个实现很像控制器处理消息的实现。这里视图选择性地接收 MESSAGE_MODEL_UPATED 消息，然后通知列表适配器去更新显示。如果有其他非 MESSAGE_MODEL_UPDATED 的消息传进来，它会忽略掉的。也就是说它不是处理所有的信息。你可以增加或减少视图可接收的信息。

> 我可以在 Activity 中开线程或者创建后台服务吗？

记住视图的特性。我们一直努力让视图变得足够笨，它只是做我们叫它做的事情。一旦 Activity 很不幸要做很多其他的事情，我们应该保持他们的逻辑足够简单。在 `onRestart` ， `onResume` ， `onDestory`中放代码是可以的，但是如果逻辑太多和复杂，我们应该把他们抽到控制器去实现，只要写个代理就可以了。

##Controller:

控制器向模型传递了引用，然后与视图用消息来沟通。它是 MVC 的大脑。我们已经讨论了控制器如何收发消息了。接着我们讨论它如何处理这些信息。答案就是：想怎么办就怎么办！一个好的控制器内部如何处理这些信息的逻辑需要封闭（开放封闭原则）。现在就讨论一下技术上如何实现。在我们的例子中，当接收到 `TapListController.MESSAGE_INCREMENT_COUNTER` ，控制器就把模型的 `count` 加一，然后通过 DAO 代理来实现数据的持久化（这个之后会说），最后通知外部有更新。所有这些都在一个独立的线程里面完成。所有这些实现都是外部不可见的。让我们看看代码：

    :::java
    package com.musselwhizzle.tapcounter.controllers;
     
    import java.util.ArrayList;
     
    import android.os.Handler;
    import android.os.HandlerThread;
     
    import com.musselwhizzle.tapcounter.daos.CounterDao;
    import com.musselwhizzle.tapcounter.vos.CounterVo;
     
    public class TapListController extends Controller {
        private static final String TAG = TapListController.class.getSimpleName();
        private HandlerThread workerThread;
        private Handler workerHandler;
     
        public static final int MESSAGE_GET_COUNTERS = 1;
        public static final int MESSAGE_MODEL_UPDATED = 2;
        public static final int MESSAGE_DELETE_COUNTER = 3;
        public static final int MESSAGE_INCREMENT_COUNTER = 4;
        public static final int MESSAGE_DECREMENT_COUNTER = 5;
     
        private ArrayList model;
        public ArrayList getModel() {
            return model;
        }
     
        public TapListController(ArrayList model) {
            this.model = model;
            workerThread = new HandlerThread("Worker Thread");
            workerThread.start();
            workerHandler = new Handler(workerThread.getLooper());
        }
     
        @Override
        public void dispose() {
            super.dispose();
            workerThread.getLooper().quit();
        }
     
        @Override
        public boolean handleMessage(int what, Object data) {
            switch(what) {
                case MESSAGE_GET_COUNTERS:
                    getCounters();
                    return true;
                case MESSAGE_DELETE_COUNTER:
                    deleteCounter((Integer)data);
                    getCounters();
                    return true;
                case MESSAGE_INCREMENT_COUNTER:
                    changeCount(1, (CounterVo)data);
                    getCounters();
                    return true;
                case MESSAGE_DECREMENT_COUNTER:
                    changeCount(-1, (CounterVo)data);
                    getCounters();
                    return true;
            }
            return false;
        }
     
        private void changeCount(final int amount, final CounterVo counter) {
            workerHandler.post(new Runnable() {
                @Override
                public void run() {
                    synchronized (counter) {
                        counter.setCount(counter.getCount() + amount);
                        CounterDao dao = new CounterDao();
                        dao.update(counter);
                    }
                }
            });
     
        }
     
        private void getCounters() {
            workerHandler.post(new Runnable() {
                @Override
                public void run() {
                    CounterDao dao = new CounterDao();
                    ArrayList counters = dao.getAll();
                    synchronized (model) {
                        while(model.size() > 0) {
                            model.remove(0);
                        }
                        for (CounterVo counter : counters) {
                            model.add(counter);
                        }
                        notifyOutboxHandlers(MESSAGE_MODEL_UPDATED, 0, 0, null);
                    }
                }
            });
        }
     
        private void deleteCounter(final int itemId) {
            workerHandler.post(new Runnable() {
                @Override
                public void run() {
                    CounterDao dao = new CounterDao();
                    dao.delete(itemId);
                }
            });
        }
    }

##其他一些思考:

在这个程序中，每一个 activity/view 都有特定的 controller 和 model 。TapActivity 有 TapController 和 TapListActivity 有 TapListController 。有时我会允许当 activity 销毁的时候 model 也销毁。在另外一些时候， 我有一个程序级的 model 这个 model 有很多 activity 绑定在上面，那么我就不想某个 activity 去销毁它。我通过使用单例模式来实现这个程序级 model 的功能。虽然这不应该在这个系列中讨论，但是我觉得有必要提一下。一个使用单例的例子是有一个 RSS 程序，其中一个 activity 显示摘要，另一个 activity 显示详细内容，拿不想在两个 activity 之间重复加载数据，这个时候你就可以使用这个模式来保证模型在内存中。无论如何，你是否销毁模型不那么重要，它仍然是 MVC 。你只需要根据需求来修改就好了。

##总结

到这里，你应该很清楚的知道这三个部分如何在 android 程序中有机结合了。花些时间来检查一下代码吧。但你看到 TapController 的时候可能会觉得陌生，但是它所做的只是代理消息而已。我们会深入讨论一下状态模式的使用和为什么要使用它。
