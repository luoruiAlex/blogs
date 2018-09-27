### 概念
- Java Management Extension
- MBean：Managed Bean，代表一个被管理的资源实例。
  - 通过MBean中暴露的方法和属性，外界可以获取被管理的资源的状态和操纵MBean的行为
  - 本质上就是Java Object，通过自省和反射来操作，只是MBean更复杂
  - 一共有四种类型的MBean: Standard MBean, Dynamic MBean, Open MBean, Model MBean
- MBean Server：MBean生存在一个MBeanServer中。
  - MBeanServer管理这些MBean，并且代理外界对它们的访问。
  - MBeanServer提供了一种**注册机制**，是的外界可以通过名字来得到相应的MBean实例
- JMX Agent：Agent只是一个Java进程，它包括这个MBeanServer和一系列附加的MbeanService
  - 附加的service也通过MBean来发布
- Protocol Adapters and Connectors：使MBean服务器与管理应用程序能进行通信,一个代理要被管理，它必须提供至少一个Protocol Adapter或者Connector。面临多种管理应用时，代理可以包含各种不同的Protocol Adapters和Connectors。
  - RMI Connector, SNMP Adapter, IIOP Adapter, HTML Adapter, HTTP Connector.
  
### JMX三层
- Instrumentation 层
  - 主要包括了一系列的接口定义和描述如何开发MBean的规范
- Agent层
  - Agent 用来管理相应的资源，并且为远端用户提供访问的接口
- Distributed 层 
  - Distributed层关心Agent如何被远端用户访问的细节。它定义了一系列用来访问Agent的接口和组件，包括Adapter和Connector的描述。

### Stardard MBean
- 每一个MBean定义一个接口，而且这个接口的名字必须是其被管理的资源的对象类的名称后面加上”MBean”
