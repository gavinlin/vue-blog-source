---
title: DLNA/UPNP相关理论
date: 2013-08-29 
category: Android
---

之前炒得很热的多屏互动，其中就用到了**upnp**技术。

简单地说，upnp就是通过一些列网络协议，实现了网络内各种设备的通讯和数据传输。其中用到的网络协议有
<!-- excerpt -->

基于HTTP的**SSDP（Simple Service Discovery Protocol）**作用是网络中的设备发现。

基于UDP的**GENA（General Event Notification Architecture）**事件通知框架，看到UDP就知道其是非可靠传输，一般用作控制设备或者回应时的消息协议。

基于TCP的**SOAP（Simple Object Access Protocol）**简单对象访问协议。

![upnp architecture](/images/upnp/upnp.jpg)

UPnP定义了通用的设备与控制点之间的交互协议，包括寻址、发现、描述、控制、事件、展示等部分，如下图：

![upnp stream](/images/upnp/upnp_stream.jpg)

* 寻址：设备通过DHCP或Auto-IP自动获取一个IP地址，并测试及定时检查地址可用性。

* 发现：通过SSDP协议，当设备加入网络时向控制点宣告其可用服务。同样，当控制点加入网络时允许其搜索到感兴趣的设备。

* 描述：使用XML形式，让控制点了解设备及其提供的功能。描述包括设备描述、服务描述、非标准厂商扩展等。

* 控制：采用SOAP协议（Web Service协议），控制点向服务发出动作，设备服务执行后返回结果或错误码。或者控制点查询服务的状态变量值。

* 事件：当服务状态变量发生改变时，发布者（服务）向订阅者发送事件消息，提醒订阅者状态变量改变。控制点采用GENA协议订阅服务所提供的事件。

* 展示：控制点采用浏览器获得设备展示页面。

<br/>

##Upnp影音架构

除了定义了上面的基本的通信行为，upnp还定义了一个AV架构。

UPnP定义的基本影音设备包括MediaServer（媒体服务器）、MediaRenderer（媒体渲染器）。其中MediaServer服务包括：AVTransport(AVT)、ConnectionManager(CM)、ContentDirectory(CDS)等，MediaRenderer服务包括：AVTransport(AVT)、ConnectionManager(CM)、RenderingControl(RCS)等。

![av architecture](/images/upnp/upnp_av.jpg)

基本上mediaserver是内容的提供者，controlpoint通过控制mediaserver，mediarenderer从mediaserver里取得数据并播放，那么mediarenderer就是内容的呈现者。

![upnp control](/images/upnp/upnp_control.jpg)

1. Control Point调用MediaServer的CDS::Browser/Search动作，获取MediaServer内容目录对象

2. Control Point调用MediaRenderer的CM::GetProtocolInfo动作，获取MediaRenderer支持的数据格式

3. Control Point对支持的格式进行内容选择

4. Control Point分别调用MediaServer及MediaRenderer的CM::PrepareForConnection建立连接

5. Control Point调用MediaRenderer的AVT::SetAVTransportURI动作，设置内容URI路径

6. Control Point调用MediaRenderer的AVT::Play动作，启动播放

7. MediaServer与MediaRenderer之间通过HTTP GET进行内容传输

8. Control Point调用MediaRenderer的RCS::SetVolumn动作，设置音量

9. 内容传输完成，根据需要设置下一首音乐的URI

10. Control Point分别调用MediaServer及MediaRenderer的CM::ConnectionComplete终止连接

<br/>

##DLNA

至于DLNA，就是在upnp的基础上，定了更多的标准，来实现家庭影音互动，其核心基础就是upnp。
