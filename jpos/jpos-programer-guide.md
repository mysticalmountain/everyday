[TOC]

# 新建JPOS项目
## 生成JOS项目整体架构
1. 下载JOS 项目模板，地址：https://github.com/jpos/jPOS-template.git
2. 打开build.gradle文件，修改你需要使用的jpos版本
```groovy
dependencies {
    compile ('org.jpos:jpos:2.0.+') {
        exclude(module: 'junit')
        exclude(module: 'hamcrest-core')
    }
    testCompile 'junit:junit:4.8.2'
}
```
3. 生成IDE对象的项目配置文件
```groovy
gradle idea
gradle eclipse
```
:smile *jpos项目骨架已经生成完毕*
4. 验证项目是否正常启动
*启动日志*
```
<log realm="Q2.system" at="Tue Dec 15 17:33:28 CST 2015.86" lifespan="506ms">
  <info>
    Q2 started, deployDir=D:\Workspace\source\sample\jpos-sample\deploy
    
    jPOS 1.9.8 master/baae23b (2014-08-10 15:13:59 UYT) 

-----BEGIN PGP SIGNED MESSAGE-----
Hash: SHA1

jPOS Community Edition, licensed under GNU AGPL v3.0.
This software is probably not suitable for commercial use.
Please see http://jpos.org/license for details.

-----BEGIN PGP SIGNATURE-----
Version: GnuPG v1.4.9 (Darwin)

iQEcBAEBAgAGBQJMolHDAAoJEOQyeO71nYtFv74H/3OgehDGEy1VXp2U3/GcAobg
HH2eZjPUz53r38ARPiU3pzm9LwDa3WZgJJaa/b9VrJwKvbPwe9+0kY3gScDE1skT
ladHt+KHHmGQArEutkzHlpZa73RbroFEIa1qmN6MaDEHGoxZqDh0Sv2cpvOaVYGO
St8ZaddLBPC17bSjAPWo9sWbvL7FgPFOHhnPmbeux8SLtnfWxXWsgo5hLBanKmO1
1z+I/w/6DL6ZYZU6bAJUk+eyVVImJqw0x3IEElI07Nh9MC6BA4iJ77ejobj8HI2r
q9ulRPEqH9NR79619lNKVUkE206dVlXo7xHmJS1QZy5v/GT66xBxyDVfTduPFXk=
=oP+v
-----END PGP SIGNATURE-----

  </info>
</log>
```

## 写一个简单的QBean
1. 写QBean有两种方式，继承QBeanSupport类，或者实现QBean接口。如果选择实现QBean接口需要写个接口继承QBean接口，然后再写类实现上两个接口。我选择继承QBeanSupport类的方式
```java
package com.sample.qbean;

import org.jpos.core.Configuration;
import org.jpos.core.ConfigurationException;
import org.jpos.q2.Q2;
import org.jpos.q2.QBean;
import org.jpos.q2.QBeanSupport;
import org.jpos.util.Log;

/**
 * User: andongxu
 * Time: 15-12-14 下午4:46
 */
public class QTest extends QBeanSupport {

    private Log log;
    private volatile int state;

    public QTest() {
        state = -1;
        log = Log.getLog(Q2.LOGGER_NAME, "QTEST");
        log.info("constructor");
    }

    @Override
    public void init() {
        log.info("initializing");
        state = QBean.STARTING;
    }

    @Override
    public void start() {
        log.info("starting");
    }

    @Override
    public void stop() {
        log.info("stopping");
    }

    @Override
    public void destroy() {
        log.info("destroying");
    }

    @Override
    public int getState() {
        return state;
    }

    @Override
    public String getStateAsString() {
        return state >= 0 ? QBean.stateString[state] : "Unknown";
    }
}
```
> Q2 为QBean生命周期提供了四个简单的方法：init,start,stop,destroy
> int getState(),String getStateAsString()用在加务管理和基于JMX的监控系统上
> ResourceBundle QFactory.properties文件为很多我们熟知的QBeans提供了快捷称呼，定义入下： 
```java
shutdown=org.jpos.q2.qbean.Shutdown
script=org.jpos.q2.qbean.BSH
jython=org.jpos.q2.qbean.Jython
spacelet=org.jpos.q2.qbean.SpaceLet
sysmon=org.jpos.q2.qbean.SystemMonitor
txnmgr=org.jpos.transaction.TransactionManager
transaction-manager=org.jpos.transaction.TransactionManager
qmux=org.jpos.q2.iso.QMUX
channel-adaptor=org.jpos.q2.iso.ChannelAdaptor
```
像下面这样使用
```java
<script name="myscript">
...
...
</script>
``` 

2. 在deploy目录添加配置文件，将QBean注入到jpos
配置文件名称：90_qtest.xml
```xml
<qbean name="qtest" logger="Q2" realm="qtest" class="com.sample.qbean.QTest">

</qbean>
```
3. 启动程序，发现生命周期方法的日志已经打印
```xml
</log>
<log realm="QTEST" at="Tue Dec 15 17:49:06 CST 2015.214">
  <info>
    constructor
  </info>
</log>
<log realm="QTEST" at="Tue Dec 15 17:49:06 CST 2015.218">
  <info>
    initializing
  </info>
</log>
<log realm="Q2.system" at="Tue Dec 15 17:49:06 CST 2015.218" lifespan="209ms">
  <info>
    deploy: D:\Workspace\source\sample\jpos-sample\deploy\90_qtest.xml
  </info>
</log>
<log realm="QTEST" at="Tue Dec 15 17:49:06 CST 2015.218">
  <info>
    starting
  </info>
</log>
```

## 写一个服务接收ISO8583报文
1. 新建一个类ISOServer继承ISORequestListener
```java
package com.sample.server;

import org.jpos.iso.ISOMsg;
import org.jpos.iso.ISORequestListener;
import org.jpos.iso.ISOSource;
import org.jpos.q2.Q2;
import org.jpos.util.Log;
import org.jpos.util.NameRegistrar;

/**
 * ISOServer 接收ISO消息
 * User: andongxu
 * Time: 15-12-15 下午3:14
 */
public class ISOServer implements ISORequestListener {

    private Log log = Log.getLog(Q2.LOGGER_NAME, "server");

    @Override
    public boolean process(ISOSource isoSource, ISOMsg isoMsg) {
        try {
            log.info("receive message:" + new String(isoMsg.pack()));
            ISOMsg isoMessageResponse = (ISOMsg) isoMsg.clone();
            isoMessageResponse.set(39, "00");
            isoSource.send(isoMessageResponse);
            return true;
        } catch (Exception e) {
            return false;
        }
    }
}
```

2. 将服务注册到jpos中
在deploy目录新建配置文件"50_server.xml"
```xml
<server name="server" class="org.jpos.q2.iso.QServer" logger="Q2">
    <attr name="port" type="java.lang.Integer">10000</attr>
    <channel realm="server-channel" logger="Q2" class="org.jpos.iso.channel.XMLChannel" packager="org.jpos.iso.packager.XMLPackager">
    </channel>
    <request-listener class="com.sample.server.ISOServer" logger="Q2" realm="request-listener">
    </request-listener>
</server>
```
3. 写测试程序发送ISO8583报文
```java
package com.sample.message

import org.jpos.iso.BaseChannel
import org.jpos.iso.ISOException
import org.jpos.iso.ISOMsg
import org.jpos.iso.ISOPackager
import org.jpos.iso.channel.ASCIIChannel
import org.jpos.iso.channel.XMLChannel
import org.jpos.iso.packager.ISO87APackager
import org.jpos.iso.packager.XMLPackager
import org.jpos.util.SimpleLogListener

/**
 * User: andongxu
 * Time: 15-12-15 下午3:26 
 */
class SendMessage {

    public static void main(String[] args) throws Exception {
        String ip = "localhost";
        Integer port = 10000;
        ISOMsg msg = ISOMsgHelper.createEchoRequest();
        ISOPackager packager = new XMLPackager();
        XMLChannel channel = new XMLChannel(ip, port, packager);
        org.jpos.util.Logger jposLogger = new org.jpos.util.Logger();
        SimpleLogListener log4JListener = new SimpleLogListener();
        jposLogger.addListener(log4JListener);
        channel.setLogger(jposLogger, "client-channel");
        kirimConnectionless(msg, channel);
    }

    private static void kirimConnectionless(ISOMsg msg, BaseChannel channel) throws IOException, ISOException {
        channel.connect();
        channel.setTimeout(30000);
        channel.send(msg);
        ISOMsg reply = channel.receive();
        System.out.println("Reply : [" + new String(reply.pack()) + "]");
        channel.disconnect();
    }
}

package com.sample.message;

import org.jpos.iso.ISOException;
import org.jpos.iso.ISOMsg;

import java.text.SimpleDateFormat;
import java.util.Date;

public class ISOMsgHelper {
    static SimpleDateFormat formatterBit7 = new SimpleDateFormat("MMddHHmmss");
    static SimpleDateFormat formatterBit12 = new SimpleDateFormat("HHmmss");
    
    public static ISOMsg createEchoRequest() throws ISOException {
        ISOMsg netmanRequest = new ISOMsg();
        netmanRequest.setMTI("0800");
        netmanRequest.set(7, formatterBit7.format(new Date()));
        netmanRequest.set(12, formatterBit12.format(new Date()));
        netmanRequest.set(70, "301");
        return netmanRequest;
    }
}

```
4. 测试并查看日志
SendMessage.java日志
```
<log realm="client-channel/127.0.0.1:10000" at="Tue Dec 15 17:59:29 CST 2015.995" lifespan="1ms">
  <send>
    <isomsg>
      <!-- org.jpos.iso.packager.XMLPackager -->
      <field id="0" value="0800"/>
      <field id="7" value="1215175929"/>
      <field id="12" value="175929"/>
      <field id="70" value="301"/>
    </isomsg>
  </send>
</log>
<log realm="client-channel/127.0.0.1:10000" at="Tue Dec 15 17:59:30 CST 2015.9" lifespan="13ms">
  <receive>
    <isomsg direction="incoming">
      <!-- org.jpos.iso.packager.XMLPackager -->
      <field id="0" value="0800"/>
      <field id="7" value="1215175929"/>
      <field id="12" value="175929"/>
      <field id="39" value="00"/>
      <field id="70" value="301"/>
    </isomsg>
  </receive>
</log>
Reply : [<isomsg>
  <!-- org.jpos.iso.packager.XMLPackager -->
  <field id="0" value="0800"/>
  <field id="7" value="1215175929"/>
  <field id="12" value="175929"/>
  <field id="39" value="00"/>
  <field id="70" value="301"/>
</isomsg>
]
```
服务日志
```
<log realm="server-channel/127.0.0.1:62145" at="Tue Dec 15 17:59:29 CST 2015.997" lifespan="512ms">
  <receive>
    <isomsg direction="incoming">
      <!-- org.jpos.iso.packager.XMLPackager -->
      <field id="0" value="0800"/>
      <field id="7" value="1215175929"/>
      <field id="12" value="175929"/>
      <field id="70" value="301"/>
    </isomsg>
  </receive>
</log>
<log realm="server" at="Tue Dec 15 17:59:29 CST 2015.998">
  <info>
    receive message:<isomsg>
  <!-- org.jpos.iso.packager.XMLPackager -->
  <field id="0" value="0800"/>
  <field id="7" value="1215175929"/>
  <field id="12" value="175929"/>
  <field id="70" value="301"/>
</isomsg>

  </info>
</log>
<log realm="server-channel/127.0.0.1:62145" at="Tue Dec 15 17:59:29 CST 2015.999">
  <send>
    <isomsg>
      <!-- org.jpos.iso.packager.XMLPackager -->
      <field id="0" value="0800"/>
      <field id="7" value="1215175929"/>
      <field id="12" value="175929"/>
      <field id="39" value="00"/>
      <field id="70" value="301"/>
    </isomsg>
  </send>
</log>
<log realm="server-channel/127.0.0.1:62145" at="Tue Dec 15 17:59:30 CST 2015.31" lifespan="31ms">
  <receive>
    <peer-disconnect/>
  </receive>
</log>
<log realm="server.server.session/127.0.0.1:62145" at="Tue Dec 15 17:59:30 CST 2015.32">
  <session-end/>
</log>
```

## 写一个服务接收RESTful报文
我这里使用jersey框架
web server类
```java
package com.sample.server;

import com.sun.jersey.spi.container.servlet.ServletContainer;
import org.eclipse.jetty.servlet.ServletContextHandler;
import org.eclipse.jetty.servlet.ServletHolder;
import org.jpos.core.Configuration;
import org.jpos.q2.QBeanSupport;
import org.eclipse.jetty.server.Server;

import javax.naming.ConfigurationException;

/**
 * User: andongxu
 * Time: 15-12-15 下午3:56
 */
public class RestServer extends QBeanSupport {
    private Server server;
    private int port;

    @Override
    protected void initService() throws Exception {
        Configuration cfg = getConfiguration();
        port = cfg.getInt("port", 8080);
        server = new Server(port);

        String contextPath = cfg.get("contextPath", "/");
        log.info("context path = " + contextPath);
        ServletContextHandler servletContextHandler = new ServletContextHandler(server, contextPath);

        String[] packagesToScan = cfg.getAll("packagesToScan");
        String[] pathSpec = cfg.getAll("pathSpec");

        if (packagesToScan.length != pathSpec.length)
            throw new ConfigurationException("number of packagesToScan must equal to number of pathSpec");

        for (int i = 0; i < packagesToScan.length; i++) {
            ServletHolder servletHolder = new ServletHolder(ServletContainer.class);
            servletHolder.setInitParameter("com.sun.jersey.config.property.packages", packagesToScan[i]);
            servletContextHandler.addServlet(servletHolder, pathSpec[i]);
            log.info(packagesToScan[i] + " ==> " + pathSpec[i]);
        }
    }

    @Override
    protected void startService() throws Exception {
        if (!server.isRunning()) {
            server.start();
            log.info("server start on port " + port);
        } else {
            log.info("server already running on port " + port);
        }
    }

    @Override
    protected void stopService() throws Exception {
        if (server.isRunning()) {
            server.stop();
            log.info("server stop on port " + port);
        }
    }
}
```

controller

```java
package com.sample.controller;

import javax.servlet.http.HttpServletRequest;
import javax.ws.rs.GET;
import javax.ws.rs.Path;
import javax.ws.rs.Produces;
import javax.ws.rs.core.Context;
import javax.ws.rs.core.MediaType;
import javax.ws.rs.core.Response;
import java.io.FileWriter;

/**
 * User: andongxu
 * Time: 15-12-15 下午4:26
 */
@Path("/test")
public class TestController {

    @GET
    @Path("/get")
    @Produces(MediaType.APPLICATION_JSON)
    public Response create(@Context HttpServletRequest request) {
        try {
            return Response.status(Response.Status.OK).build();
        } catch (Exception e) {
            e.printStackTrace();
            return Response.status(Response.Status.BAD_REQUEST).build();
        }
    }
}

```

将server注入jpos,在deploy目录新建配配置文件“91_rest.xml”
```
<?xml version="1.0" encoding="UTF-8" standalone="no"?>
<dragon-rest class="com.sample.server.RestServer" logger="Q2" name="server-rest">
    <property name="port" value="8080"/>
    <property name="contextPath" value="/rest"/>

    <property name="packagesToScan" value="com.sample.controller"/>
    <property name="pathSpec" value="/*"/>
</dragon-rest>

```