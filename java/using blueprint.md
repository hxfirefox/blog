ODL buleprint 使用指引
====================

Blueprint是OSGi的依赖注入框架的规范，继承自Spring DM并与之非常相似，Karaf包含了Apache Aries blueprint实现。

使用blueprint时，bundle提供XML来描述：bundle需要的OSGi服务依赖，bundle业务逻辑需要实例化的Java对象，以及如何将Java对象与服务依赖进行联系。此外bundle还可以导出/通告自己的OSGi服务。

Blueprint XML文件中共有4个元素：bean，service，reference和reference-list：

- bean - an element that describes a Java object to be instantiated given a class name and optional constructor args and properties.
- service - advertises a bean as an OSGi service
- reference - imports a singleton OSGi service that implements a specified interface and/or satisfies a specified property filter
- reference-list - imports multiple OSGi services that implement a specified interface and/or satisfy a specified property filter


For detailed documentation of these elements and blueprint design, refer to the Blueprint chapter of the OSGi compendium spec. Also refer to the Aries documentation

The blueprint extender is the component that extracts and parses blueprint XML resources from bundles as they are activated and creates the blueprint containers. By default, the extender looks for XML resources under the standard OSGI-INF/blueprint path inside bundles. The parsing and container creation is done asynchronously so there's no implicit deterministic startup ordering as is the case with Opendaylight's config subsystem via feature ordering. Therefore, in order to preserve this functionality with blueprint, if needed, and to avoid intermittent timing issues on startup, Opendaylight has its own component that scans a custom path, org/opendaylight/blueprint. This allows for the creation of blueprint containers to be potentially ordered. So it is recommended to put your blueprint XML files under src/main/resources/org/opendaylight/blueprint in your bundle projects (as of this writing, ordering hasn't been implemented as there hasn't been a need for it).

The following sections illustrate examples for writing blueprint XML for Opendaylight bundles along with some best practices and illustrate the Opendaylight's blueprint extensions that provide additional functionality and convenient shortcuts for using MD-SAL core services. 

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
