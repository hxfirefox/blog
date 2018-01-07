ODL buleprint 使用指引
====================

Blueprint是OSGi的依赖注入框架的规范，继承自Spring DM并与之非常相似，Karaf包含了Apache Aries blueprint实现。

使用blueprint时，bundle提供XML来描述：bundle需要的OSGi服务依赖，bundle业务逻辑需要实例化的Java对象，以及如何将Java对象与服务依赖进行联系。此外bundle还可以导出/通告自己的OSGi服务。

Blueprint XML文件中共有4个元素：bean，service，reference和reference-list：

- bean - 描述Java对象进行实例化时的class名和可选的构造参数及属性
- service - 通告bean具有的OSGi服务
- reference - 导入OSGi服务，该服务实现特定接口以及/或者满足特定的属性过滤条件
- reference-list - 导入多个OSGi服务

关于上述元素和blueprint的详细信息，可参考[OSGi纲要规范](https://osgi.org/download/r4v42/r4.enterprise.pdf)中的Blueprint章节，也可参考[Aries文档](http://aries.apache.org/modules/blueprint.html)。

The blueprint extender is the component that extracts and parses blueprint XML resources from bundles as they are activated and creates the blueprint containers. By default, the extender looks for XML resources under the standard OSGI-INF/blueprint path inside bundles. The parsing and container creation is done asynchronously so there's no implicit deterministic startup ordering as is the case with Opendaylight's config subsystem via feature ordering. Therefore, in order to preserve this functionality with blueprint, if needed, and to avoid intermittent timing issues on startup, Opendaylight has its own component that scans a custom path, org/opendaylight/blueprint. This allows for the creation of blueprint containers to be potentially ordered. So it is recommended to put your blueprint XML files under src/main/resources/org/opendaylight/blueprint in your bundle projects (as of this writing, ordering hasn't been implemented as there hasn't been a need for it).

下面将具体说明如何编辑ODL bundle的blueprint XML以及ODL对blueprint的扩展提供的便利方式使用MD-SAL核心服务。

# 序言： XML vs. Java annotations

本指引描述“纯XML”实现的Blueprint，在一些ODL项目中已经出现使用Java注解来替代XML中的<bean>的Blueprint实现方式，例如：Genius、NetVirt。本指引推荐首先阅读本文来获得基础知识，之后如对ODL中的实现方式感兴趣，则可参考[Best Practices DI Guidelines](https://wiki.opendaylight.org/view/BestPractices/DI_Guidelines)。

# Importing MD-SAL services

如下XML通过reference元素导入标准MD-SAL服务，如DataBroker、RpcProviderRegistry以及NotificationPublishService。

```
<?xml version="1.0" encoding="UTF-8"?>
<blueprint xmlns="http://www.osgi.org/xmlns/blueprint/v1.0.0"
                 xmlns:odl="http://opendaylight.org/xmlns/blueprint/v1.0.0"
    odl:use-default-for-reference-types="true">

  <reference id="dataBroker" interface="org.opendaylight.controller.md.sal.binding.api.DataBroker" odl:type="default"/>
  <reference id="rpcRegistry" interface="org.opendaylight.controller.sal.binding.api.RpcProviderRegistry"/>
  <reference id="notificationService" 
          interface="org.opendaylight.controller.md.sal.binding.api.NotificationPublishService"/>

</blueprint>
```

Internally, the reference element creates a proxy that implements the specified interface and wraps the actual service instance. This is done due to the dynamic nature of OSGi services as implementations may, theoretically, come and go if bundles are dynamically stopped or updated/restarted. In practice, though, this is difficult to achieve in large platforms like Opendaylight with hundreds of bundles (but that's another topic).

On startup, the container will first wait for all referenced service dependencies to be satisfied before continuing. The default timeout is 5 minutes. If the timeout expires, the container fails fast. In addition, if a service later becomes unavailable it will block method calls and wait the timeout period for a new service instance to become available. If the timeout expires, a runtime exception is thrown. This can be handled in application code if one really wants to support dynamic bundle updates but, as mentioned, in practice this is not realistically feasible with large applications.

There may be multiple service implementations advertised for the same interface. This is the case with the DataBroker - there is the default implementation and the specialized PingPongDataBroker. It is ambiguous as to which service is obtained unless an OSGi service property filter is specified to distinguish which implementation is desired.

MD-SAL services are advertised with a specific property named type. Opendaylight defines an extension attribute to the reference element named type that, internally, is added to the OSGi service filter. This is illustrated in the above example for the DataBroker reference element, where the odl:type attribute is set to default. If one wanted the PingPongDataBroker then set it to pingpong. In order to access the extension, the Opendaylight extension namespace, http://opendaylight.org/xmlns/blueprint/v1.0.0, must be specified in the root blueprint element and type must be prefixed appropriately.

As a convention, the default type is used for the default implementation of the service with other types being specialized implementations. This allows consumers to obtain whichever implementation the provider deems as the default without having to explicitly know it and allows the provider to switch to a new default implementation without requiring all consumers to change their referenced type.

To avoid having to specify the type attribute for every reference element, another Opendaylight extension, use-default-for-reference-types, can be specified in the root blueprint element. This attribute automatically adds a filter to all reference elements such that the type property is either not set or set to default. This ensures the default implementation is imported unless type is explicitly specified for a reference element, in which case it overrides the use-default-for-reference-types setting. It is recommended to always set use-default-for-reference-types to true as a fallback.

The odl:type attribute can also be specified for service elements as a shortcut to add the service property. 

# Instantiating business logic
## Using direct class instantiation
Now that we've seen how to import MD-SAL services, the next step is to instantiate the business logic.

```
<?xml version="1.0" encoding="UTF-8"?>
<blueprint xmlns="http://www.osgi.org/xmlns/blueprint/v1.0.0"
                 xmlns:odl="http://opendaylight.org/xmlns/blueprint/v1.0.0"
    odl:use-default-for-reference-types="true">

  <reference id="dataBroker" interface="org.opendaylight.controller.md.sal.binding.api.DataBroker"/>

  <bean id="foo" class="org.opendaylight.app.Foo" init-method="init" destroy-method="close">
    <property name="dataBroker" ref="dataBroker"/>
  </bean>

  <bean id="bar" class="org.opendaylight.app.Bar">
    <argument ref="foo"/>
  </bean>

</blueprint>
```

# MD-SAL blueprint 扩展
## Global RPCs

ODL提供2种blueprint的扩展，简化注册与使用global MD-SAL RPC服务。

注册RpcService实现时，使用rpc-implementation元素。下面的例子中org.opendaylight.app.FooRpcServiceImpl实现了org.opendaylight.app.FooRpcService接口。

```
<?xml version="1.0" encoding="UTF-8"?>
<blueprint xmlns="http://www.osgi.org/xmlns/blueprint/v1.0.0"
                 xmlns:odl="http://opendaylight.org/xmlns/blueprint/v1.0.0">

  <bean id="fooRpcService" class="org.opendaylight.app.FooRpcServiceImpl">
    <!-- constructor args -->
  </bean>

  <odl:rpc-implementation ref="fooRpcService"/>

</blueprint>
```

通过引用实现实例（如上所示），系统自动发现被实现的RpcService接口并向MD-SAL RpcProviderRegistry注册实现。如果实现了多个RpcService接口，则所有被实现的接口均会得到注册。通过interface属性还能指定接口。

当容器销毁时，注册自动注销。

使用rpc-service元素来消费org.opendaylight.app.FooRpcService（如下）。

```
<?xml version="1.0" encoding="UTF-8"?>
<blueprint xmlns="http://www.osgi.org/xmlns/blueprint/v1.0.0"
                 xmlns:odl="http://opendaylight.org/xmlns/blueprint/v1.0.0">

  <odl:rpc-service id="fooRpcService" interface="org.opendaylight.app.FooRpcService"/>

  <bean id="bar" class="org.opendaylight.app.Bar">
    <argument ref="fooRpcService"/>
  </bean>

</blueprint>
```

rpc-service元素获得了指定接口（该接口注册在MD-SAL RpcProviderRegistry中）的RpcService实现，并在创建实例（id=fooRpcService）后注入到org.opendaylight.app.Bar实例中。

## Routed RPCs

routed-rpc-implementation扩展注册routed RpcService实现。

```
<?xml version="1.0" encoding="UTF-8"?>
<blueprint xmlns="http://www.osgi.org/xmlns/blueprint/v1.0.0"
                 xmlns:odl="http://opendaylight.org/xmlns/blueprint/v1.0.0">

  <bean id="fooRoutedRpcService" class="org.opendaylight.app.FooRoutedRpcServiceImpl">
    <!-- constructor args -->
  </bean>

  <odl:routed-rpc-implementation id="fooRoutedRpcServiceReg" ref="fooRoutedRpcService"/>

  <bean id="bar" class="org.opendaylight.app.Bar">
    <argument ref="fooRoutedRpcServiceReg"/>
  </bean>

</blueprint>
```

routed-rpc-implementation元素发现引用的实例实现了RpcService接口，并向MD-SAL RpcProviderRegistry进行注册。它也会创建实例（id=fooRpcService）后注入到org.opendaylight.app.Bar实例中。当容器销毁时，RoutedRpcRegistration实例自动关闭。

如果实例实现了多个RpcService接口，则会在使用该blueprint扩展时发生失败。这种场景下必须通过interface属性指定需要的接口，并为每个接口创建一个routed-rpc-implementation元素。

## NotificationListener

notification-listener扩展元素向MD-SAL NotificationService注册NotificationListener实现以接受yang通知。

```
<?xml version="1.0" encoding="UTF-8"?>
<blueprint xmlns="http://www.osgi.org/xmlns/blueprint/v1.0.0"
                 xmlns:odl="http://opendaylight.org/xmlns/blueprint/v1.0.0">

  <bean id="fooListener" class="org.opendaylight.app.FooNotificationListener">
    <!-- constructor args -->
  </bean>

  <odl:notification-listener ref="fooListener"/>

</blueprint>
```
