---
title: Android MVC Part4 The Controller
date: 2013-11-14
category: android
tags: mvc
Parts: mvc
---

在 MVC 的三个部分中，控制器是最繁忙的。回顾一下，模型只是一个比较优雅的键值对象。一个好的视图是一个呆子，它只是当模型改变的时候更新自己和控制器告诉他什么就做什么。而控制器主要做三件事：
<!-- excerpt -->

1. 更新模型
2. 处理视图传过来的信息
3. 发信息给试图

这里我们主要关注后面两件事情。

##处理消息

在所有部分当中，控制器获取消息的实现是最随意的，主要决定权在你手中，它不需要多态。你不需要跟着教程来做。而下面只是我的实现方式。但要记住 MVC 只是一个框架，不需要太注重细节，你应该关注你想要实现什么。下面是 TapListController 的示例代码。

>为什么选取 TapListController 而不是 TapController ？坑爹吗？

其实主要是因为 `TapController` 实现了 状态模式来代理消息。所以选择 `TapListController` 这个更简单的类来说，以避免越说越乱。

    :::java
    public class TapListController extends Controller{
        
        public static final int MESSAGE_GET_COUNTERS = 1;
        public static final int MESSAGE_MODEL_UPDATED = 2;
        public static final int MESSAGE_DELETE_COUNTER = 3;
        public static final int MESSAGE_INCREMENT_COUNTER = 4;
        public static final int MESSAGE_DECREMENT_COUNTER = 5;
        
        private ArrayList<CounterVo> model;
        public ArrayList<CounterVo> getModel(){
            return model;
        }
        
        public TapListController(ArrayList<CounterVo> model){
            this.model = model;
        }

        @Override
        boolean handleMessage(int what, Object data) {
            switch(what){
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
    }

记得视图是通过 `handleMessage` 来传递消息给控制器的。这个是我们消息系统的入口。很简单是吗？这个函数的第一个参数是动作，是一个常量，第二个参数是数据，用来传递额外的信息给控制器的。当控制器接受了信息后将返回 true 。

>等等，这种实现模式不就是 Handler 吗？那为什么还要重复做轮子？

这位聪明的同学说得没错，这类似于 Handler 。但是要知道 Handler 是异步的，不可能返回是否接受了消息。但是在实际处理消息的时候控制器可能会使用异步，具体的实现方式的细节可以隐藏在控制器当中。但不管怎么说，它需要有返回值告诉别的类它接受到消息了。

视图想发送消息给控制器可以使用下面的方法。

    :::java
    controller.handleMessage(TapListController.MESSAGE_INCREMENT_COUNTER, contextCounter);

##发送消息

发送消息和接收差不多，想看代码，下面就是了。

    :::java
    protected final void notifyOutboxHandlers(int what, int arg1, int arg2, Object obj) {
        if (!outboxHandlers.isEmpty()) {
            for (Handler handler : outboxHandlers) {
                Message msg = Message.obtain(handler, what, arg1, arg2, obj);
                msg.sendToTarget();
            }
        }
    }

当控制器传消息给视图，控制器不用理会视图在想什么和做什么。这里不需要有返回值所以使用 android 的消息系统就可以了，事实上这是 android 中比较好的消息传递方式，因为它是异步的。

##在一起

我们已经讲述了控制器的发送和接收信息了，下一节将会从宏观的角度简述 MVC 架构。
