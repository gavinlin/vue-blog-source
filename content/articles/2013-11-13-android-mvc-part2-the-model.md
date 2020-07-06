---
title: Android MVC Part2 The Model
date: 2013-11-13
category: Android
tags: mvc
Parts: mvc
---

##MVC 基本概念

MVC 是一些设计模式组合在一起的框架模式。其中的模型(The Model)代表着程序的状态。要知道，对象(Object)由状态（属性）和行为（方法）组成。所以程序也要状态（The Model）。本质上， models 是一个键值对象，当里面的状态改变的时候发送事件。视图（The View）是用户看到并与之交互的对象。视图作为观察者绑定到模型中，当模型的状态改变，视图会得到通知并作出相应的更新。当用户与视图交互，视图会发送事件给控制器（The Controller）。控制器的作用就是处理输入逻辑，它解释用户的手势，更新模型，而且向视图反馈信息。下面是它们的关系图。
<!-- excerpt -->

![mvc relationship]({filename}/images/mvc/part2/mvc_diagram.png)

##本项目的Model

在这个项目中，最主要的模型是 `CounterVo`

代码如下

    :::java
    package com.lingavin.tapcounter.vos;

    public class CounterVo extends SimpleObservable<CounterVo>{
        private int id =-1;
        public int getId(){
            return id;
        }
        public void setId(int id){
            this.id = id;
            notifyObservers(this);
        }
        
        private int count = 0;
        public int getCount(){
            return count;
        }
        public void setCount(int count){
            this.count = count;
            notifyObservers(this);
        }
        
        private String label="";
        public String getLabel(){
            return label;
        }
        public void setLabel(String label){
            this.label = label;
            notifyObservers(this);
        }
        
        private boolean locked = false;
        public boolean isLocked(){
            return locked;
        }
        public void setLocked(boolean locked){
            this.locked = locked;
            notifyObservers(this);
        }
        
        @Override
        synchronized public CounterVo clone(){
            CounterVo vo = new CounterVo();
            vo.setId(id);
            vo.setLabel(label);
            vo.setCount(count);
            vo.setLocked(locked);
            return vo;
        }
        
        synchronized public void consume(CounterVo vo){
            this.id = vo.getId();
            this.label = vo.getLabel();
            this.count = vo.getCount();
            this.locked = vo.isLocked();
            notifyObservers(this);
        }
    }

SimpleObservable:

    :::java
    package com.lingavin.tapcounter.vos;

    import java.util.ArrayList;

    public class SimpleObservable<T> implements EasyObservable<T> {

        private final ArrayList<OnChangeListener<T>> listeners = new ArrayList<OnChangeListener<T>>();
        
        @Override
        public void addListener(OnChangeListener<T> listener) {
            synchronized (listeners) {
                listeners.add(listener);
            }
        }

        @Override
        public void removeListener(OnChangeListener<T> listener) {
            synchronized (listeners) {
                listeners.remove(listener);
            }
        }
        
        protected void notifyObservers(final T model){
            synchronized (listeners) {
                for(OnChangeListener<T> listener:listeners){
                    listener.onChange(model);
                }
            }
        }

    }

请留意我们的模型只做两件事：

+ 存放状态，例如 id,label,count,locked 这些都是对象的属性。
+ 当属性改变的时候发放通知。实际上做这些工作是通过 `SimpleObservable` 完成。

如果对象想获得模型改变的通知，它可以通过实现 `Onchangelistener` 接口 ，然后把它自己作为观察者注册（下一章会讲）。当我们模型中任何一个属性有变化，模型会调用 `notifyObservers()` 方法来通知所有注册上的观察者。

##总结
上面这些就是 android 中的模型，下一章我们会讨论 视图，还有如何把视图绑定到模型中。

##问题

Q: 为什么把模型叫做 CounterVo?

A: Vo 的意思是 Value Object. Value Objects 是模型的一种. 你可以把它改成你想的名字，通常我把模型改名为 SomethingVo 通常意味着它是列表项的渲染器或者作为传递对象。

