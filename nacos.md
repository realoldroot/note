根据 nacos 官网文档配置 springboot 项目
使用依赖

```
implementation 'com.alibaba.boot:nacos-config-spring-boot-starter:0.2.7'
implementation 'com.alibaba.boot:nacos-discovery-spring-boot-starter:0.2.7'

```

测试成功可以自动更新配置属性。

--- 

但是项目中已经使用了 Dubbo+Zookeeper，然后想换掉注册中心使用 nacos

遂增加依赖

```
mplementation 'org.apache.dubbo:dubbo-spring-boot-starter:2.7.10'
implementation('org.apache.dubbo:dubbo-registry-nacos:2.7.10') {
        exclude group: 'org.apache.dubbo', module: 'dubbo-common'
        exclude group: 'org.apache.dubbo', module: 'dubbo-remoting-api'
    }
```

运行报错

```
client error: invalid param. For input string: "${nacos.config.maxRetry${nacos.maxRetry:}}"
```

查找解决办法 https://github.com/alibaba/nacos/issues/3787
在**nacos1.4.0**中修复问题，需要升级 `com.alibaba.nacos:nacos-api:1.2.0`->`1.4.0`，
所以升级`nacos-spring-context`

```gradle
implementation 'com.alibaba.nacos:nacos-spring-context:1.1.0'
```
引用依赖
```
implementation 'com.alibaba.spring:spring-context-support:1.0.11'
```
运行抛出异常

```
Caused by: java.lang.ClassCastException: class java.lang.String cannot be cast to class java.lang.Class (java.lang.String and java.lang.Class are in module java.base of loader 'bootstrap')
	at org.apache.dubbo.config.spring.util.DubboAnnotationUtils.resolveServiceInterfaceClass(DubboAnnotationUtils.java:95) ~[dubbo-2.7.10.jar:2.7.10]
	at org.apache.dubbo.config.spring.util.DubboAnnotationUtils.resolveInterfaceName(DubboAnnotationUtils.java:78) ~[dubbo-2.7.10.jar:2.7.10]
	at org.apache.dubbo.config.spring.beans.factory.annotation.ServiceBeanNameBuilder.<init>(ServiceBeanNameBuilder.java:65) ~[dubbo-2.7.10.jar:2.7.10]
	at org.apache.dubbo.config.spring.beans.factory.annotation.ServiceBeanNameBuilder.create(ServiceBeanNameBuilder.java:78) ~[dubbo-2.7.10.jar:2.7.10]
	at org.apache.dubbo.config.spring.beans.factory.annotation.ReferenceAnnotationBeanPostProcessor.buildReferencedBeanName(ReferenceAnnotationBeanPostProcessor.java:361) ~[dubbo-2.7.10.jar:2.7.10]
	at org.apache.dubbo.config.spring.beans.factory.annotation.ReferenceAnnotationBeanPostProcessor.buildInjectedObjectCacheKey(ReferenceAnnotationBeanPostProcessor.java:350) ~[dubbo-2.7.10.jar:2.7.10]
	at com.alibaba.spring.beans.factory.annotation.AbstractAnnotationBeanPostProcessor.getInjectedObject(AbstractAnnotationBeanPostProcessor.java:404) ~[spring-context-support-1.0.11.jar:na]
	at com.alibaba.spring.beans.factory.annotation.AbstractAnnotationBeanPostProcessor$AnnotatedFieldElement.inject(AbstractAnnotationBeanPostProcessor.java:626) ~[spring-context-support-1.0.11.jar:na]
	at org.springframework.beans.factory.annotation.InjectionMetadata.inject(InjectionMetadata.java:119) ~[spring-beans-5.2.14.RELEASE.jar:5.2.14.RELEASE]
	at com.alibaba.spring.beans.factory.annotation.AbstractAnnotationBeanPostProcessor.postProcessPropertyValues(AbstractAnnotationBeanPostProcessor.java:179) ~[spring-context-support-1.0.11.jar:na]
	... 32 common frames omitted

```
翻了一下代码是因为`(Class<?>) "Void"`


查找解决办法 https://github.com/apache/dubbo/issues/7115

根据提示降低`spring-context-support`版本到**1.0.11**，运行再次报错

```
Caused by: java.lang.ClassCastException: class com.sun.proxy.$Proxy15 cannot be cast to class java.util.Map (com.sun.proxy.$Proxy15 is in unnamed module of loader 'app'; java.util.Map is in module java.base of loader 'bootstrap')
	at com.alibaba.nacos.spring.beans.factory.annotation.AnnotationNacosInjectedBeanPostProcessor.getNacosProperties(AnnotationNacosInjectedBeanPostProcessor.java:145) ~[nacos-spring-context-1.1.0.jar:na]
	at com.alibaba.nacos.spring.beans.factory.annotation.AnnotationNacosInjectedBeanPostProcessor.buildInjectedObjectCacheKey(AnnotationNacosInjectedBeanPostProcessor.java:135) ~[nacos-spring-context-1.1.0.jar:na]
	at com.alibaba.spring.beans.factory.annotation.AbstractAnnotationBeanPostProcessor.getInjectedObject(AbstractAnnotationBeanPostProcessor.java:357) ~[spring-context-support-1.0.10.jar:na]
	at com.alibaba.spring.beans.factory.annotation.AbstractAnnotationBeanPostProcessor$AnnotatedFieldElement.inject(AbstractAnnotationBeanPostProcessor.java:542) ~[spring-context-support-1.0.10.jar:na]
	at org.springframework.beans.factory.annotation.InjectionMetadata.inject(InjectionMetadata.java:119) ~[spring-beans-5.2.14.RELEASE.jar:5.2.14.RELEASE]
	at com.alibaba.spring.beans.factory.annotation.AbstractAnnotationBeanPostProcessor.postProcessPropertyValues(AbstractAnnotationBeanPostProcessor.java:145) ~[spring-context-support-1.0.10.jar:na]
	... 18 common frames omitted

```

`AnnotationNacosInjectedBeanPostProcessor`报错代码
```
private Map<String, Object> getNacosProperties(AnnotationAttributes attributes) {
		return (Map<String, Object>) attributes.get("properties");
	}

```