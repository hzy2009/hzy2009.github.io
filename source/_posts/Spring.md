---
title: Spring提供的事件驱动模型(观察者模式)
date: 2019-09-25 10:13:24
tags: Spring 观察者模式 事件驱动模型
---
## 事件驱动模型
事件驱动模型也就是我们常说的观察者模式，或者发布-订阅模型，理解它的几个关键点：
* 首先是一种对象间的一对多关系，比如微信和微信公众号的关系，微信是观察者，微信公众号是被观擦者
* 当目标(微信公众号)发生改变，观察者(微信用户)可以接收到改变
* 观察者如何处理，目标无需干涉，所以就降低耦合度
   
## 用户注册的例子
游戏用户注册成功后，可能会发生这么多事
* 加积分
* 发确认邮件
* 获得游戏大礼包

***问题***
UserServiceImple类里需要引用创建积分的service，确认邮件的service等，耦合严重，增删功能比较麻烦，也有可能需要调用第三方接口，速度慢，代码维护性差，同时访问数据库多次，用户体验差。    

从以上例子可以看出应该使用一个观察者来解耦service里的依赖关系增加一个Lisyener来解耦UserService和其他功能，即注册成功后，只需要通知相关监听器，不需要关系它们如何处理，一个接口只完成一个功能，增删功能非常容易。  

首先看下Spring提供的事件驱动模型体系图
{% plantuml %}
    Application <|-- ApplicationContextEvent 
    ApplicationContextEvent <|-- ContextStartedEvent
    ApplicationContextEvent <|-- ContextStopedEvent
    ApplicationContextEvent <|-- ContextClosedEvent
    ApplicationContextEvent <|-- ContextRefreshedEvent
{% endplantuml %}

## 事件代表者：ApplicationEvent
系统默认提供了如下ApplicationEvent事件实现
只有一个ApplicationContextEvent，表示ApplicationContext容器事件，有如下实现
* ContextStantedEvent:ApplicationContext启动后触发的事件  
* ContextStoppedEvent:ApplicationContext停止后触发的事件
* ContextRefresheEvent:ApplicationContext初始化或者刷新完成后触发的事件
* ContextClosedEvent:ApplicationContext关闭后触发的事件

## 事件发布者：ApplicationEventPublisher
ApplicationContext接口继承了ApplicationEventPublisher，并在AbstractApplicationContext实现了具体代码，实际执行委托给了ApplicationEventMulicaster。源码如下：

```
protected void publishEvent(Object event, @Nullable ResolvableType eventType) {
	Assert.notNull(event, "Event must not be null");

	// Decorate event as an ApplicationEvent if necessary
	ApplicationEvent applicationEvent;
	if (event instanceof ApplicationEvent) {
		applicationEvent = (ApplicationEvent) event;
	} else {
		applicationEvent = new PayloadApplicationEvent<>(this, event);
		if (eventType == null) {
			eventType = ((PayloadApplicationEvent<?>) applicationEvent).getResolvableType();
		}
	}

	// Multicast right now if possible - or lazily once the multicaster is initialized
	if (this.earlyApplicationEvents != null) {
		this.earlyApplicationEvents.add(applicationEvent);
	} else {
		getApplicationEventMulticaster().multicastEvent(applicationEvent, eventType);
	}

	// Publish event via parent context as well...
	if (this.parent != null) {
		if (this.parent instanceof AbstractApplicationContext) {
			((AbstractApplicationContext) this.parent).publishEvent(event, eventType);
		}
		else {
			this.parent.publishEvent(event);
		}
	}
}
```

我们常用的ApplicationContext都继承自AbstractApplicationContext，如ClassPathXmlApplicationContext、XmlWebApplicationContext等。ApplicationContext自动到容器找一个名为ApplicationEventMulticaster实现，如果没有自己new一个SimpleApplicationEventMulticaster,其中SimpleApplicationEventMulticaster的源码可以看出，spring默认的事件处理方式是并行处理，根据事件的泛型类型，找到对应的Listener，并且使用线程池给给每个Listener分配线程处理。发布事件源码如下：

```
	public void multicastEvent(final ApplicationEvent event, @Nullable ResolvableType eventType) {
		ResolvableType type = (eventType != null ? eventType : resolveDefaultEventType(event));
		for (final ApplicationListener<?> listener : getApplicationListeners(event, type)) {
			Executor executor = getTaskExecutor();
			if (executor != null) {
				executor.execute(() -> invokeListener(listener, event));
			}
			else {
				invokeListener(listener, event);
			}
		}
	}
```


## 事件监听者(处理者):ApplicationListener
ApplicationListener 提供 onApplicationEvent方法，我们只需要实现该接口，并指定我们需要的处理的事件。并把改类放到spring容器管理。如下代码：
```
@Component 
public class EditReceiveOwnQuantityAfterBackOrder implements ApplicationListener<OrderBackEvent> {

	@Override
	public void onApplicationEvent(OrderBackEvent event) {
		// 业务代码
	}
}
```