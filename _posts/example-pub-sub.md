---
name: pub-sub
title: 发布-订阅
date: 2025-05-16
categories: 设计模式
---

```java
// 事件接口
interface Event {}

// 示例事件类型
class MessageEvent implements Event {
    private final String message;

    public MessageEvent(String message) {
        this.message = message;
    }

    public String getMessage() {
        return message;
    }
}

// 订阅者接口
interface Subscriber<T extends Event> {
    void onEvent(T event);
}

// 事件总线（中心）
class EventBus {
    private final Map<Class<? extends Event>, List<Subscriber<? extends Event>>> subscribers = new HashMap<>();

    public <T extends Event> void subscribe(Class<T> eventType, Subscriber<T> subscriber) {
        subscribers.computeIfAbsent(eventType, k -> new ArrayList<>()).add(subscriber);
    }

    public <T extends Event> void publish(T event) {
        List<Subscriber<? extends Event>> eventSubscribers = subscribers.get(event.getClass());
        if (eventSubscribers != null) {
            for (Subscriber<? extends Event> sub : eventSubscribers) {
                @SuppressWarnings("unchecked")
                Subscriber<T> typedSub = (Subscriber<T>) sub;
                typedSub.onEvent(event);
            }
        }
    }
}
```

