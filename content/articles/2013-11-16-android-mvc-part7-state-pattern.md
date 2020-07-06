---
title: Android MVC Part7 State Pattern
date: 2013-11-16
category: android
tags: mvc
Parts: mvc
---

这部分的内容是之前答应过之后再说的状态模式，这个模式在 `TapController` 用到。与 MVC 和 DAOs 不同的是，我不会把状态模式归为 android 程序架构必须实现的部分。然而，它的确非常有用。因为我在 TapController 中用它来处理消息，所以还是有必要说一下。
<!-- excerpt -->

##状态模式

在之前的系列中我没有提到它是怎样在 android 里工作的。但是在这部分，我们将会更深入一些来探讨它。[状态模式][1]是一个面向对象的模式，目的是可以动态切换对象的行为。还记得对象具有属性和行为吗？你会觉得这听起来像是策略模式？对，你说得没错。这里不同的地方就是模式的意图。状态模式是根据属性的不同而改变其行为的。客户端甚至不知道调用的对象已经发生转换。那么是怎么做到动态切换而接口不变的？我们需要把行为封装到对象内部，然后让他妈共享一套接口。接下来我们聊聊具体的例子，这个例子就是 Tap Counter 工程了。

[1]: http://en.wikipedia.org/wiki/State_pattern

我们要看的连个对象叫做 `UnlockedState` 和 `LockedState`。当 `CounterVo` 设置为被锁，这意味着我们不希望属性改变。例如，如果用户点击了增加按钮，如果状态是锁住得话，值将不会增加。但是如果不是锁住的状态，用户点击将会使值增加。这是基于 CounterVo.locked 的两种不同行为。

当 TapController.handleMessage 被触发，它就会把消息代理给其中一种状态。这里使用了 Controller.messageState 来确定用哪一种转台。让我们看看代码吧。

    :::java
    @Override
    public boolean handleMessage(int what) {
        return messageState.handleMessage(what);
    }

>这的确非常简单，但是 messageState 的状态是如何改变的呢？

两个状态之间是彼此知道对方的。当 UnLockedState 接到 MESSAGE_UPDATE_LOCK 这个消息时，它就会把 messageState 的引用改为 LockedState 。然后当 MESSAGE_UPDATE_LOCK 消息被 LockedState 处理时它会更细 messageState 的引用到 UnlockedState。我觉得我们看看代码会更好理解一些。

    :::java
    package com.musselwhizzle.tapcounter.controllers;
     
    public class UnlockedState extends TapState {
     
        // ... properties
     
        public UnlockedState(TapController controller) {
            super(controller);
        }
     
        @Override
        public boolean handleMessage(int what) {
            switch(what) {
                case TapController.MESSAGE_INCREMENT_COUNT:
                    moveCount(1);
                    return true;
                case TapController.MESSAGE_DECREMENT_COUNT:
                    moveCount(-1);
                    return true;
                case TapController.MESSAGE_RESET_COUNT:
                    model.setCount(0);
                    return true;
                default:
                    return super.handleMessage(what);
            }
     
        }
     
        @Override
        public boolean handleMessage(int what, Object data) {
            switch(what) {
                case TapController.MESSAGE_UPDATE_LOCK:
                    updateLock((Boolean)data);
                    return true;
                case TapController.MESSAGE_UPDATE_LABEL:
                    updateLabel((String)data);
                    return true;
                case TapController.MESSAGE_KEY_EVENT:
                    return handleKeyEvent((KeyEvent)data);
                default:
                    return super.handleMessage(what, data);
            }
        }
     
        private boolean handleKeyEvent(KeyEvent event) {
            // .. handles the key event
        }
     
        private void moveCount(int amount) {
            model.setCount(model.getCount()+amount);
        }
     
        private void updateLock(boolean lock) {
            model.setLocked(lock);
            controller.setMessageState(new LockedState(controller));
        }
     
        private void updateLabel(String label) {
            model.setLabel(label);
        }
    }


UnLockedState 通知控制器去改变状态为 LockedState 的代码如下。

    :::java
    controller.setMessageState(new LockedState(controller));

现在我们可以改变状态和知道代理发生了什么事了。让我们来看看两种不同状态是如何处理 TapController.MESSAGE_INCREMENT_COUNT 的。在 UnlockedState 看到的代码就是你想的那样

    :::java
    @Override
    public boolean handleMessage(int what) {
        switch(what) {
            case TapController.MESSAGE_INCREMENT_COUNT:
        model.setCount(model.getCount()+1);
        return true;
        }
    }

然后看看 LockedState

    :::java
    @Override
    public boolean handleMessage(int what) {
        switch(what) {
            case TapController.MESSAGE_INCREMENT_COUNT:
        return true;
        }
    }

什么也没做，只是返回了 true 表示处理了这个消息。

>那么为什么你要做这些工作？你不可以使用 if-else 语句来判断 CounterVo.locked 来完成这些工作吗？

我们可以这么做，但当你使用 if-else 来管理状态，一旦状态变得复杂或者增多，你的 if-else 将会变得又长又臭。你的代码会变得很难维护，如果想加入新特性很可能只有重构了。使用状态模式你可以很方便地为新状态增加新的行为，而且遵循 开放/封闭 原则。当然，在我们的案例中，它有可能有杀鸡用牛刀的感觉。但无论如何，我只想创建一个简单的使用真实解决方案和架构的真实工程。
