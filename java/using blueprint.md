ODL buleprint 使用指引
====================

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
