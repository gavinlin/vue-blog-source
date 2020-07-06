---
title: Android MVC Part3 The View
date: 2013-11-13
category: android
tags: mvc
Parts: mvc
---

##什么是视图？

在 Android 中很多同学说 View 就是视图。但是 View 只有显示的能力，它不能处理响应和发送消息。先来看看视图的特点：
<!-- excerpt -->

1. 绑定模型
2. 发送信息给控制器
3. 处理控制器返回的信息

看来 activity 符合我们的要求，它可以绑定，可以处理信息，而且控制着 android 中所谓的 view 。但是 activity 又有一些其他的特性使它不像是 MVC 中的 V ，例如它管理着生命周期，而且它有一些像控制器的方法

+ boolean dispatchKeyEvent(KeyEvent event);
+ boolean onOptionsItemSelected(MenuItem item);
+ void onCreateContextMenu(….);
+ so on….

>要知道 dispatchEvent 如果返回 true 表示这个 activity 会处理该事件。但是实际上，处理事件的逻辑应该交给控制器做的。

我们通过一些方法来避免上述的问题，使 activities 更像是一个视图。其灵感来源于[这篇文章][1]

为了把 activity 变成 view ，我们做了下面的操作。

##1. 数据绑定

来看看 TapActivity 是如何绑定数据的

    :::java
    package com.lingavin.tapcounter.activities;

    import com.lingavin.tapcounter.R;
    import com.lingavin.tapcounter.R.layout;
    import com.lingavin.tapcounter.vos.CounterVo;
    import com.lingavin.tapcounter.vos.OnChangeListener;

    import android.app.Activity;
    import android.os.Bundle;
    import android.os.Handler;
    import android.os.Message;
    import android.widget.Button;
    import android.widget.CompoundButton;
    import android.widget.EditText;
    import android.widget.TextView;

    public class TapActivity extends Activity implements OnChangeListener {

        private static final String TAG = TapActivity.class.getSimpleName();
        private CounterVo mCounterVo;
        
        private EditText label;
        private TextView count;
        private Button minusBtn;
        private Button plusBtn;
        private CompoundButton lockedBtn;
        
        private static final int UPDATE_VIEW = 0;
        
        private Handler mHandler = new Handler(){

            @Override
            public void handleMessage(Message msg) {
                super.handleMessage(msg);
                int what = msg.what;
                switch(what){
                case UPDATE_VIEW:
                    updateView();
                    break;
                default:
                }
            }
            
        };
        
        @Override
        protected void onCreate(Bundle savedInstanceState) {
            super.onCreate(savedInstanceState);
            setContentView(R.layout.activity_main);
            
            mCounterVo = new CounterVo();
            mCounterVo.addListener(this);
            
            initViews();
        }

        private void initViews() {
            lockedBtn = (CompoundButton) findViewById(R.id.lockBtn);
            ......
        }

        @Override
        public void onChange(Object model) {
            mHandler.sendEmptyMessage(UPDATE_VIEW);
        }
        
        private void updateView(){
            if(!label.getText().toString().equals(mCounterVo.getLabel()))
                label.setText(mCounterVo.getLabel());
            label.setEnabled(!mCounterVo.isLocked());
            count.setText(Integer.toString(mCounterVo.getCount()));
            lockedBtn.setChecked(mCounterVo.isLocked());
        }

    }

在 `onCreate` 函数中我们实例化了 `CounterVo` 并把 `TabActivity` 作为观察者注册上 `mCounterVo` 。

    :::java
    mCounterVo = new CounterVo();
    mCounterVo.addListener(this);

这样每当模型有改变，`onChange` 函数会被调用，我们就是可以更新我们的子视图了。一定要记住 android 需要在 UI线程来修改 UI 的状态，所以加入了 Handler。就这样，我们的数据就绑定好了。

先休息一下，接下来实现如何发送消息给控制器。

##2. 发送消息

现在我们需要把 activity 的控制器特性去除，把消息交给真正的控制器来处理。我们通过委托来实现这个功能。

    :::java
    package com.lingavin.tapcounter.activities;

    public class TapActivity extends Activity implements OnChangeListener {
        
        public static final String EXTRA_TAP_ID = "tapId";

        private CounterVo counterVo;
        private TapController controller;

        @Override
        protected void onCreate(Bundle savedInstanceState) {
            super.onCreate(savedInstanceState);
            setContentView(R.layout.activity_main);
            
            counterVo = new CounterVo();
            counterVo.addListener(this);
            controller = new TapController(counterVo);
            
            initViews();
        }

        private void initViews() {
            label = (EditText) findViewById(R.id.label);
            ......
            
            plusBtn.setOnClickListener(new OnClickListener(){
                
                @Override
                public void onClick(View arg0) {
                    controller.handlMessage(TapController.MESSAGE_INCREMENT_COUNT);
                }
                
            });
            
            minusBtn.setOnClickListener(new OnClickListener(){

                @Override
                public void onClick(View v) {
                    controller.handlMessage(TapController.MESSAGE_DECREMENT_COUNT);
                }
                
            });
        }
        
        @Override
        public boolean dispatchKeyEvent(KeyEvent event) {
            boolean handled = controller.handleMessage(TapController.MESSAGE_KEY_EVENT, event);
            if(!handled)
                return super.dispatchKeyEvent(event);
            return handled;
        }
    }

好了现在 dispatchKeyEvent 的消息都通过代理传给了 controller 去处理了。而且按钮的点击事件也有 controller 来处理。

>等等，那么我们该如何更新视图呢？

不用着急，还记得数据绑定没有？我们把消息传给了控制器。控制器去更新模型，模型通过数据绑定来触发 activity 的 onChange 。这样 UI 界面就更新了。

##处理消息

这个和发送消息给 controller 是差不多的，这里就不详细说了。

[1]: http://mindtherobot.com/blog/675/android-architecture-message-based-mvc/
