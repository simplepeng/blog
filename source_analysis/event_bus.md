# EventBus

EventBus是[greenrobot](https://github.com/greenrobot)开源的一款在Android和Java上都能用的基于`订阅/发布`的事件总线框架。

[开源地址](https://github.com/greenrobot/EventBus)

想弄明白的问题：

* 如何注册/反注册的
* 注册类是如何收到消息的
* 粘性消息如何工作的
* 如何切换线程的

基于`v3.2.0`源码分析

### 如何注册/反注册的

```java
//EveutBus.java
public static EventBus getDefault() {
    EventBus instance = defaultInstance;
    if (instance == null) {
        synchronized (EventBus.class) {
            instance = EventBus.defaultInstance;
            if (instance == null) {
                instance = EventBus.defaultInstance = new EventBus();
            }
        }
    }
    return instance;
}
```

单例模式，很典型的double-checked单例实现了。

```java
//EveutBus.java
public void register(Object subscriber) {
    Class<?> subscriberClass = subscriber.getClass();
    List<SubscriberMethod> subscriberMethods = subscriberMethodFinder.findSubscriberMethods(subscriberClass);
    synchronized (this) {
        for (SubscriberMethod subscriberMethod : subscriberMethods) {
            subscribe(subscriber, subscriberMethod);
        }
    }
}
```

`findSubscriberMethods`方法从订阅类中找出所有被注解标记的方法，然后调用`subscribe`方法注册

```java
//EveutBus.java
public synchronized void unregister(Object subscriber) {
    List<Class<?>> subscribedTypes = typesBySubscriber.get(subscriber);
    if (subscribedTypes != null) {
        for (Class<?> eventType : subscribedTypes) {
            unsubscribeByEventType(subscriber, eventType);
        }
        typesBySubscriber.remove(subscriber);
    } else {
        logger.log(Level.WARNING, "Subscriber to unregister was not registered before: " + subscriber.getClass());
    }
}
```

`typesBySubscriber`是一个Map，找出event所有的注册类，从Map里面删除掉。

### 注册类是如何收到消息的

```java
//EveutBus.java
public void post(Object event) {
    PostingThreadState postingState = currentPostingThreadState.get();
    List<Object> eventQueue = postingState.eventQueue;
    eventQueue.add(event);

    if (!postingState.isPosting) {
        postingState.isMainThread = isMainThread();
        postingState.isPosting = true;
        if (postingState.canceled) {
            throw new EventBusException("Internal error. Abort state was not reset");
        }
        try {
            while (!eventQueue.isEmpty()) {
                postSingleEvent(eventQueue.remove(0), postingState);
            }
        } finally {
            postingState.isPosting = false;
            postingState.isMainThread = false;
        }
    }
}
```

调用`post(Object event)`方法后，只要`eventQueue`队列不为空，就一直调用`postSingleEvent`方法。

```java
//EveutBus.java
private void postSingleEvent(Object event, PostingThreadState postingState) throws Error {
    Class<?> eventClass = event.getClass();
    boolean subscriptionFound = false;
    if (eventInheritance) {
        List<Class<?>> eventTypes = lookupAllEventTypes(eventClass);
        int countTypes = eventTypes.size();
        for (int h = 0; h < countTypes; h++) {
            Class<?> clazz = eventTypes.get(h);
            subscriptionFound |= postSingleEventForEventType(event, postingState, clazz);
        }
    } else {
        subscriptionFound = postSingleEventForEventType(event, postingState, eventClass);
    }
    if (!subscriptionFound) {
        if (logNoSubscriberMessages) {
            logger.log(Level.FINE, "No subscribers registered for event " + eventClass);
        }
        if (sendNoSubscriberEvent && eventClass != NoSubscriberEvent.class &&
                eventClass != SubscriberExceptionEvent.class) {
            post(new NoSubscriberEvent(this, event));
        }
    }
}
```

postSingleEvent方法中，调用`lookupAllEventTypes`方法找出`event`对应的所有注册类。最终都会走到`invokeSubscriber`执行订阅方法。

```java
//EveutBus.java
 void invokeSubscriber(Subscription subscription, Object event) {
     try {
         subscription.subscriberMethod.method.invoke(subscription.subscriber, event);
     } catch (InvocationTargetException e) {
         handleSubscriberException(subscription, event, e.getCause());
     } catch (IllegalAccessException e) {
         throw new IllegalStateException("Unexpected exception", e);
     }
 }
```

### 粘性消息如何工作的

```java
//EveutBus.java
public void postSticky(Object event) {
    synchronized (stickyEvents) {
        stickyEvents.put(event.getClass(), event);
    }
    // Should be posted after it is putted, in case the subscriber wants to remove immediately
    post(event);
} 
```

粘性消息只是把`event`存到了`stickyEvents`Map中，然后在`subscribe`方法中判断订阅方法是否是`粘性订阅方法`，如果是就接受消息。

```java
//EveutBus.java
private void subscribe(Object subscriber, SubscriberMethod subscriberMethod) {
    //...
    if (subscriberMethod.sticky) {
        if (eventInheritance) {
            // Existing sticky events of all subclasses of eventType have to be considered.
            // Note: Iterating over all events may be inefficient with lots of sticky events,
            // thus data structure should be changed to allow a more efficient lookup
            // (e.g. an additional map storing sub classes of super classes: Class -> List<Class>).
            Set<Map.Entry<Class<?>, Object>> entries = stickyEvents.entrySet();
            for (Map.Entry<Class<?>, Object> entry : entries) {
                Class<?> candidateEventType = entry.getKey();
                if (eventType.isAssignableFrom(candidateEventType)) {
                    Object stickyEvent = entry.getValue();
                    checkPostStickyEventToSubscription(newSubscription, stickyEvent);
                }
            }
        } else {
            Object stickyEvent = stickyEvents.get(eventType);
            checkPostStickyEventToSubscription(newSubscription, stickyEvent);
        }
    }
}
```

### 如何切换线程的

`postToSubscription`方法中对所有线程做了对应分支，例如切换到主线程：

```java
//EveutBus.java
private void postToSubscription(Subscription subscription, Object event, boolean isMainThread) {
    switch (subscription.subscriberMethod.threadMode) {
        //...
        case MAIN:
            if (isMainThread) {
                invokeSubscriber(subscription, event);
            } else {
                mainThreadPoster.enqueue(subscription, event);
            }
            break;
        //...
    }
}
```

`mainThreadPoster`不过也只是个`Handler`的封装类罢了，`HandlerPoster`是Handler的子类，也是真正的切换线程类。

```java
public class HandlerPoster extends Handler implements Poster {

    private final PendingPostQueue queue;
    private final int maxMillisInsideHandleMessage;
    private final EventBus eventBus;
    private boolean handlerActive;
}
```

### 一些优化

[NiceKTX](https://github.com/simplepeng/NiceKTX)：基于`LifeCycle`自动反注册的KTX扩展函数。

