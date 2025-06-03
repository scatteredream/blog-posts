---
name: circle-ref-spring
title: spring 循环依赖
date: 2025-05-25
tags:
- spring
- bean
- 动态代理
- 源码
categories: spring
---



# Spring 循环依赖

## 获取单例对象

1. **优先查询一级缓存（`singletonObjects`）**
   - 一级缓存也叫单例池，存储的是**完全初始化**的单例 Bean（例如已注入所有依赖且完成代理增强的对象）。
   - **作用**：直接获取可用 Bean，避免重复创建。
   - 优先级最高：如果找到直接返回，不触发后续缓存查询。
2. **未找到则查询二级缓存（`earlySingletonObjects`）**
   - 二级缓存存储的是**已实例化但未完成初始化**的 Bean（半成品）。
   - **作用**：在循环依赖中临时暴露早期引用（例如 A 依赖 B 时，B 可能正在创建中，需引用 A 的半成品）。
   - 注意：若二级缓存中存在目标 Bean，则直接返回，但此时 Bean 可能尚未完成属性注入或代理。
3. **最后查询三级缓存（`singletonFactories`）**
   - 三级缓存是 beanName 到 对象工厂（`ObjectFactory`）的映射，对象工厂是个函数式接口，这个接口用于动态生成 Bean 的早期引用或代理对象。
   - 触发条件：仅当一、二级缓存均未找到时，调用工厂生成 Bean 实例，之后将其提升至二级缓存。
   - **关键作用**：支持 AOP 代理的延迟生成（例如解决代理对象的循环依赖）。



## 如果只有二级缓存，就可以解决问题

- `getBean(a)`，实例化对象 A 以后放入二级缓存（裸对象），然后 A 开始属性注入
- 遇到一个属性 B，先从一级缓存里面拿发现没有，瞄一眼二级缓存里面也没有，于是开始 `getBean(b)`：
- 实例化对象B以后将其放入二级缓存（裸对象），B 开始属性注入，发现 A 不在一级缓存，但是从二级缓存里面拿到了 A 的裸对象注入 B，此时 B 算初始化完成，把 B 从二级缓存里面删掉，放到一级缓存里面，至此 B 创建完成。
- 最后 A 用于注入的方法就能返回一个从缓存里面拿到的 B 对象，A 的注入也就完成了。

### 二级缓存的不足

但是二级缓存的问题是，A是代理，有B，B有需要注入A，首先A创建出实例，随后A就走到了populateBean这一步

然后去拿B，B创建，又想来获取A了，此时A那边属于是一个刚创建实例的状态，并未走到生成代理对象那一步，因此直接注入就会出问题，所以应该注册一个回调函数，把A的实例注册进去，函数的返回值是A的对象（实例/代理实例），所以就产生了三级缓存。三级缓存的 `ObjectFactory` 主要是用于提供一个钩子，这个接口的方法返回的就是bean对象，不同之处在于可以在返回裸对象前，给其套上一层代理再返回。如果只有二级缓存，就没有机会返回代理对象。

## 源码流程

> `DefaultSingletonBeanRegistry#getSingleton(name,true)` DCL双重校验锁。

```java
@Nullable
protected Object getSingleton(String beanName, boolean allowEarlyReference) {
    // 快速从一二级缓存检查已有的单例
    Object singletonObject = this.singletonObjects.get(beanName);
    if (singletonObject == null && isSingletonCurrentlyInCreation(beanName)) {
        singletonObject = this.earlySingletonObjects.get(beanName);
        if (singletonObject == null && allowEarlyReference) {
            synchronized (this.singletonObjects) {
                // 加锁，从三级缓存创建单例对象
                singletonObject = this.singletonObjects.get(beanName);
                if (singletonObject == null) {
                    singletonObject = this.earlySingletonObjects.get(beanName);
                    if (singletonObject == null) {
                        ObjectFactory<?> singletonFactory = this.singletonFactories.get(beanName);
                        if (singletonFactory != null) {
                            singletonObject = singletonFactory.getObject();
                            this.earlySingletonObjects.put(beanName, singletonObject);
                            this.singletonFactories.remove(beanName);
                        }
                    }
                }
            }
        }
    }
    return singletonObject;
}
```

------

> 流程正式开始
>
> `DefaultListableBeanFactory#preInstantiateSingletons()` 用于对解析到的` beanNames` 一一进行 `getBean(beanName)`
>
> `AbstractBeanFactory#doGetBean(name···)` 

1. `getSingleton(name,true)` 获取不到再往下走
2. 类似双亲委派，找 parent，parent 找不到再自行寻找
3. 自行寻找：(dependsOn) 然后 `getSingleton(name, singletonFactory)`
4. `getSingleton(name, singletonFactory)`: 先一级缓存找，找不到就真正开始<mark>创建工作</mark>：
   - 首先将其加到 CreationSet 表明其正在创建。
   - `singletonFactory`实际上就是一个`ObjectFactory`，这个函数式接口实现方法通过`createBean(name···)`获取对象。
   - 创建完成将其从 CreationSet 中移除，保证其只存在于一级缓存单例池中。最后返回创建好的 bean

> `AbstractAutowireCapableBeanFactory#doCreateBean(name,mbd,args)` 

1. `createBeanInstance()`：创建出裸对象，此时未注入依赖。（实际上返回的是 BeanWrapper，有更完善的功能，本质还是对象实例）

   - 当且仅当<u>单例+允许循环依赖+这个bean在 CreationSet 中</u>，才加三级缓存`addSingletonFactory`，一定要保证一级和二级缓存里面没有，然后把ObjectFactory加到三级缓存里面。所以单例 bean 加入了三级缓存：lambda：`getEarlyBeanReference()`返回创建的裸/代理对象 `singletonObject`

2. `populateBean()`：进行字段、方法注入。做一些` postProcessAfterInstantiation` 实例化之后初始化之前的工作。然后就是 `autowireByName/Name`，本质上就是通过 `getBean(name···)`获取实例。

   - 依赖注入就是这里遇到的问题，如果代理对象出现循环依赖，那么其生成应该是初始化之后，所以此阶段断然不能提供出代理对象，因此加入三级缓存提前暴露出一个引用，。

3. `initializeBean()`：

   - 调用 `BeanPostProcessor#postProcessBeforeInitialization`。

   - 调用初始化方法。(PostConstruct-initMethod-afterPropertiesSet)

   - 调用 `BeanPostProcessor#postProcessAfterInitialization`（**一般到这里才生成代理对象**）。

4. 完成上述工作之后，如果当前是存在于三级缓存，则调用下方的 `getSingleton(name,true)` ： true 代表允许早期引用（主要解决循环依赖）



> 代理：`AbstractAutoProxyCreator`

```java
// 这个方法是用于三级缓存生成对象的时候将bean放到earlyBeanReferences里面，
public Object getEarlyBeanReference(Object bean, String beanName) {
    Object cacheKey = this.getCacheKey(bean.getClass(), beanName);
    this.earlyBeanReferences.put(cacheKey, bean);
    return this.wrapIfNecessary(bean, beanName, cacheKey);
}
public Object postProcessAfterInitialization(@Nullable Object bean, String beanName) {
    if (bean != null) {
        Object cacheKey = this.getCacheKey(bean.getClass(), beanName);
        if (this.earlyBeanReferences.remove(cacheKey) != bean) {
            // wrap 包装成代理的核心方法
            return this.wrapIfNecessary(bean, beanName, cacheKey);
        }
    }

    return bean;
}
```

## 无法解决的循环引用

如果是 AB互相依赖，A只有含参构造，那么注入B完成之前就无法创建一个A实例出来，自然也没法加到缓存里面，A尝试注入B，B那边尝试注入A彻底卡死。

但是 从B开始又可以了，然而对于普通的bean来说，注册顺序并不是一个可控的状态，所以尽量避免含参构造的bean之间互相依赖









