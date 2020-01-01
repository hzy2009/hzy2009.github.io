---
title: springboot consul 启动失败调错
date: 2020-01-01 16:44:54
tags:
---

## 环境
* windows10
* jdk8

## 配置
### bootstrap.yml
 ```
 spring:
  application:
    name: devops-platform
  cloud:
    config:
      enabled: false
    consul:
      host: ${consul_host:**********}
      port: 8500 #${consul_port}
      enabled: true
      config:
        enabled: true #默认是true
        format: YAML    # 表示consul上面文件的格式 有四种 YAML PROPERTIES KEY-VALUE FILES
        data-key: devopsconfig #表示consul上面的KEY值(或者说文件的名字) 默认是data
        acl-token: ${consul_token:**********}
          #watch:
        #enabled: true #实时刷新配置
        #prefix: devops #设置配置值的基本文件夹
        #default-context: #设置所有应用程序使用的文件夹名称
        #profile-separator: #设置用于使用配置文件在属性源中分隔配置文件名称的分隔符的值
      discovery:
        enabled: true
        register: true
        serviceName: ${spring.application.name}
        healthCheckPath: /actuator/health
        healthCheckInterval: 15s
        prefer-ip-address: true
        tags: dev-/${spring.application.name}
        instanceId: ${spring.application.name}:${vcap.application.instance_id:${spring.application.instance_id:${spring.cloud.client.hostname}:${server.port}}}
        acl-token: ${consul_token:**********}
 ```


## 报错信息
```
D:\work\code\devops\essence-platform\target>java  -jar xxx.jar
2020-01-01 16:39:05.172 INFO  [main] o.s.c.a.AnnotationConfigApplicationContext - Refreshing org.springframework.context.annotation.AnnotationConfigApplicationContext@79698539: startup date [Wed Jan 01 16:39:05 CST 2020]; root of context hierarchy
2020-01-01 16:39:05.410 INFO  [main] o.s.b.f.a.AutowiredAnnotationBeanPostProcessor - JSR-330 'javax.inject.Inject' annotation found and supported for autowiring
2020-01-01 16:39:05.492 INFO  [main] o.s.c.s.PostProcessorRegistrationDelegate$BeanPostProcessorChecker - Bean 'org.springframework.retry.annotation.RetryConfiguration' of type [org.springframework.retry.annotation.RetryConfiguration$$EnhancerBySpringCGLIB$$1919688e] is not eligible for getting processed by all BeanPostProcessors (for example: not eligible for auto-proxying)
2020-01-01 16:39:05.505 INFO  [main] o.s.c.s.PostProcessorRegistrationDelegate$BeanPostProcessorChecker - Bean 'configurationPropertiesRebinderAutoConfiguration' of type [org.springframework.cloud.autoconfigure.ConfigurationPropertiesRebinderAutoConfiguration$$EnhancerBySpringCGLIB$$3f6fd366] is not eligible for getting processed by all BeanPostProcessors (for example: not eligible for auto-proxying)

  .   ____          _            __ _ _
 /\\ / ___'_ __ _ _(_)_ __  __ _ \ \ \ \
( ( )\___ | '_ | '_| | '_ \/ _` | \ \ \ \
 \\/  ___)| |_)| | | | | || (_| |  ) ) ) )
  '  |____| .__|_| |_|_| |_\__, | / / / /
 =========|_|==============|___/=/_/_/_/
 :: Spring Boot ::        (v2.0.1.RELEASE)

2020-01-01 16:39:06.595 ERROR [main] o.s.c.consul.config.ConsulPropertySourceLocator - Fail fast is set and there was an error reading configuration from consul.
2020-01-01 16:39:07.608 ERROR [main] o.s.c.consul.config.ConsulPropertySourceLocator - Fail fast is set and there was an error reading configuration from consul.
2020-01-01 16:39:08.714 ERROR [main] o.s.c.consul.config.ConsulPropertySourceLocator - Fail fast is set and there was an error reading configuration from consul.
2020-01-01 16:39:09.929 ERROR [main] o.s.c.consul.config.ConsulPropertySourceLocator - Fail fast is set and there was an error reading configuration from consul.
2020-01-01 16:39:11.265 ERROR [main] o.s.c.consul.config.ConsulPropertySourceLocator - Fail fast is set and there was an error reading configuration from consul.
2020-01-01 16:39:12.736 ERROR [main] o.s.c.consul.config.ConsulPropertySourceLocator - Fail fast is set and there was an error reading configuration from consul.
2020-01-01 16:39:12.740 ERROR [main] org.springframework.boot.SpringApplication - Application run failed
org.yaml.snakeyaml.error.YAMLException: java.nio.charset.MalformedInputException: Input length = 1
        at org.yaml.snakeyaml.reader.StreamReader.update(StreamReader.java:254)
        at org.yaml.snakeyaml.reader.StreamReader.<init>(StreamReader.java:58)
        at org.yaml.snakeyaml.Yaml.loadAll(Yaml.java:537)
        at org.springframework.beans.factory.config.YamlProcessor.process(YamlProcessor.java:160)
        at org.springframework.beans.factory.config.YamlProcessor.process(YamlProcessor.java:138)
        at org.springframework.beans.factory.config.YamlPropertiesFactoryBean.createProperties(YamlPropertiesFactoryBean.java:133)
        at org.springframework.beans.factory.config.YamlPropertiesFactoryBean.getObject(YamlPropertiesFactoryBean.java:113)
        at org.springframework.cloud.consul.config.ConsulPropertySource.generateProperties(ConsulPropertySource.java:163)
        at org.springframework.cloud.consul.config.ConsulPropertySource.parseValue(ConsulPropertySource.java:134)
        at org.springframework.cloud.consul.config.ConsulPropertySource.parsePropertiesWithNonKeyValueFormat(ConsulPropertySource.java:123)
        at org.springframework.cloud.consul.config.ConsulPropertySource.init(ConsulPropertySource.java:79)
        at org.springframework.cloud.consul.config.ConsulPropertySourceLocator.create(ConsulPropertySourceLocator.java:166)
        at org.springframework.cloud.consul.config.ConsulPropertySourceLocator.locate(ConsulPropertySourceLocator.java:132)
        at org.springframework.cloud.consul.config.ConsulPropertySourceLocator$$FastClassBySpringCGLIB$$b35ebf8.invoke(<generated>)
        at org.springframework.cglib.proxy.MethodProxy.invoke(MethodProxy.java:204)
        at org.springframework.aop.framework.CglibAopProxy$CglibMethodInvocation.invokeJoinpoint(CglibAopProxy.java:747)
        at org.springframework.aop.framework.ReflectiveMethodInvocation.proceed(ReflectiveMethodInvocation.java:163)
        at org.springframework.retry.interceptor.RetryOperationsInterceptor$1.doWithRetry(RetryOperationsInterceptor.java:91)
        at org.springframework.retry.support.RetryTemplate.doExecute(RetryTemplate.java:287)
        at org.springframework.retry.support.RetryTemplate.execute(RetryTemplate.java:164)
        at org.springframework.retry.interceptor.RetryOperationsInterceptor.invoke(RetryOperationsInterceptor.java:118)
        at org.springframework.retry.annotation.AnnotationAwareRetryOperationsInterceptor.invoke(AnnotationAwareRetryOperationsInterceptor.java:153)
        at org.springframework.aop.framework.ReflectiveMethodInvocation.proceed(ReflectiveMethodInvocation.java:185)
        at org.springframework.aop.framework.CglibAopProxy$DynamicAdvisedInterceptor.intercept(CglibAopProxy.java:689)
        at org.springframework.cloud.consul.config.ConsulPropertySourceLocator$$EnhancerBySpringCGLIB$$66dc4cb6.locate(<generated>)
        at org.springframework.cloud.bootstrap.config.PropertySourceBootstrapConfiguration.initialize(PropertySourceBootstrapConfiguration.java:94)
        at org.springframework.boot.SpringApplication.applyInitializers(SpringApplication.java:633)
        at org.springframework.boot.SpringApplication.prepareContext(SpringApplication.java:373)
        at org.springframework.boot.SpringApplication.run(SpringApplication.java:325)
        at org.springframework.boot.SpringApplication.run(SpringApplication.java:1255)
        at org.springframework.boot.SpringApplication.run(SpringApplication.java:1243)
        at com.orientsec.devops.essence.EssencePlatformApplication.main(EssencePlatformApplication.java:27)
        at sun.reflect.NativeMethodAccessorImpl.invoke0(Native Method)
        at sun.reflect.NativeMethodAccessorImpl.invoke(Unknown Source)
        at sun.reflect.DelegatingMethodAccessorImpl.invoke(Unknown Source)
        at java.lang.reflect.Method.invoke(Unknown Source)
        at org.springframework.boot.loader.MainMethodRunner.run(MainMethodRunner.java:48)
        at org.springframework.boot.loader.Launcher.launch(Launcher.java:87)
        at org.springframework.boot.loader.Launcher.launch(Launcher.java:50)
        at org.springframework.boot.loader.JarLauncher.main(JarLauncher.java:51)
Caused by: java.nio.charset.MalformedInputException: Input length = 1
        at java.nio.charset.CoderResult.throwException(Unknown Source)
        at sun.nio.cs.StreamDecoder.implRead(Unknown Source)
        at sun.nio.cs.StreamDecoder.read(Unknown Source)
        at java.io.InputStreamReader.read(Unknown Source)
        at org.yaml.snakeyaml.reader.UnicodeReader.read(UnicodeReader.java:125)
        at org.yaml.snakeyaml.reader.StreamReader.update(StreamReader.java:223)
        ... 39 common frames omitted
```


## 解决方案
1. 去掉配置文件上面的中文

2. 修改启动参数，制定执行编码
```
D:\work\code\devops\essence-platform\target>java -Dfile.encoding=utf-8 -jar xxx.jar
```
