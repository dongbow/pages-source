---
title: 基于Spring AOP的分布式锁简单实现
date: 2019-07-23 14:50:35
categories: 分布式锁
tags: 
	- 分布式锁
	- Spring AOP
---
防止数据的并发操作出现数据不一致，保证操作是幂等的，我们可以对对象加锁、方法加锁，例如synchronized、ReentrantLock等，但是上述Jdk提供的锁时基于内存的，在分布式环境下是不适用的，那么在分布式环境下该如何实现锁操作呢？  
分布式环境下需要保证集群中对同一数据、方法等操作状态，所以需要把状态单独存储，分布式锁的实现常用方式是基于缓存或者ZK，常用分布式缓存包括Redis和memcached,通过在缓存中设置同一个key和过期时间实现集群中拿到的状态是一致的，ZK是分布式锁实现是基于ZK的临时顺序节点，通过判断同一name下节点是否是最小节点确定是否获取到锁。  
本文不具体介绍如何实现缓存的写入和删除或者ZK的临时顺序节点创建等实现，主要针对怎么利用Spring AOP消除冗余的if...else,try{}catch(){}等代码，至于为什么，举个正常写一下锁调用的和利用AOP实现的简单对比。  
1. 正常写法  

```java
public String test() {
        
    final String lockKey = "lock_key";
    
    if (!DistributedLock.getInstance().lock(lockKey, 10, TimeUnit.SECONDS)) {
        // 没有获取到锁, 返回
    }
    try {
        // 获取到锁, 进行业务逻辑
        // ...
    } finally {
        // 释放锁
        DistributedLock.getInstance().unlock(lockKey);
    }

    return "ok";
}
```  

2. AOP实现   
 
```java
@Lock(prefix="lock_key")
public String test() {
    return "ok";
}
```  
看上面两个对比，是不是发现下面的写法很简洁，不用重复写相同的代码了，下面们一下具体实现，其实主要是`@Lock`注解的定义，和利用AOP对注解的拦截  

@Lock  

```java
package com.github.lock.annotation;

import java.lang.annotation.Documented;
import java.lang.annotation.ElementType;
import java.lang.annotation.Retention;
import java.lang.annotation.RetentionPolicy;
import java.lang.annotation.Target;

/**
 * @author wangdongbo
 * @since 2019/7/22.
 */
@Documented
@Target(ElementType.METHOD)
@Retention(RetentionPolicy.RUNTIME)
public @interface Lock {

    /**
     * 锁前缀, 为空时取'classname_method_'
     */
    String prefix() default "";

    /**
     * 方法参数名，支持方法中的入参，支持方法参数是对象时对象属性的字段
     * 不支持集合/数组参数
     */
    String[] parameter() default {};

    /**
     * 锁定时间
     */
    int lockSeconds() default 10;

    /**
     * 是否重试，默认重试
     */
    boolean retry() default true;

    /**
     * 重试次数，默认为1，retry为true时可用
     */
    int retryCount() default 1;

    /**
     * 没拿到锁时等待多久后重试，retry为true时可用
     */
    int tryLockSeconds() default 10;

    /**
     * 提示
     */
    String desc() default "数据正在操作中，请稍后再试";

}
```  

LockResolver.java  

```java
package com.github.lock.resolver;

import com.github.lock.annotation.Lock;
import com.github.lock.exception.LockException;
import com.github.lock.util.UUIDUtil;
import com.github.lock.way.DistributedLock;
import com.google.common.base.Joiner;
import com.google.common.collect.Lists;
import lombok.extern.slf4j.Slf4j;
import org.apache.commons.lang.ArrayUtils;
import org.aspectj.lang.ProceedingJoinPoint;
import org.aspectj.lang.annotation.Around;
import org.aspectj.lang.annotation.Aspect;
import org.aspectj.lang.annotation.Pointcut;
import org.aspectj.lang.reflect.MethodSignature;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.core.LocalVariableTableParameterNameDiscoverer;
import org.springframework.core.annotation.Order;
import org.springframework.stereotype.Component;
import org.springframework.util.StringUtils;

import java.lang.reflect.Field;
import java.lang.reflect.Method;
import java.util.List;
import java.util.stream.Stream;

/**
 * @author wangdongbo
 * @since 2019/7/22.
 */
@Slf4j
@Aspect
@Order(1)
@Component
public class LockResolver {

    private static final String DOT = ".";
    private static final String DOT_SPLIT = "\\.";

    @Autowired
    private DistributedLock distributedLock;

    private static LocalVariableTableParameterNameDiscoverer parameterNameDiscoverer = new LocalVariableTableParameterNameDiscoverer();

    @Pointcut("@annotation(com.github.lock.annotation.Lock)")
    public void lockAspectMethod() { }

    @Around("lockAspectMethod()")
    public Object doAround(ProceedingJoinPoint joinPoint) throws Throwable {
        MethodSignature methodSignature = (MethodSignature) joinPoint.getSignature();
        Method method = methodSignature.getMethod();
        if (method == null) {
            throw new LockException("method is empty");
        }
        Lock lock = method.getAnnotation(Lock.class);
        String prefix;
        if (!StringUtils.isEmpty(lock.prefix())) {
            prefix = lock.prefix();
        } else {
            prefix = joinPoint.getTarget().getClass().getSimpleName() + "_" + method.getName() + "_";
        }
        String lockKey = prefix + getArgsConcat(method, lock.parameter(), joinPoint.getArgs());
        String requestId = UUIDUtil.getID();
        try {
            if (lock.retry()) {
                int count = lock.retryCount();
                for (int i = 0; i < count; i++) {
                    if (distributedLock.tryLock(lockKey, requestId, lock.lockSeconds(), lock.tryLockSeconds())) {
                        return joinPoint.proceed();
                    }
                }
            } else if (distributedLock.lock(lockKey, requestId, lock.lockSeconds())) {
                return joinPoint.proceed();
            }
            throw new LockException(lock.desc());
        } catch (Throwable e) {
            log.warn("add lock error:{}", e.getMessage(), e);
            throw new LockException(e.getMessage(), e);
        } finally {
            distributedLock.unlock(lockKey, requestId);
        }
    }

    private String getArgsConcat(Method method, String[] parameter, Object[] args) {
        String[] params = parameterNameDiscoverer.getParameterNames(method);
        if (ArrayUtils.isEmpty(parameter)) {
            return "";
        }
        List<Object> list = Lists.newArrayListWithCapacity(parameter.length);
        Stream.of(parameter).forEach(s -> {
            if (s.contains(DOT)) {
                list.add(parseObj(s, params, args));
            } else {
                int idx = ArrayUtils.indexOf(params, s);
                list.add(args[idx]);
            }
        });
        return Joiner.on("_").join(list);
    }

    private Object parseObj(String parameter, String[] params, Object[] args) {
        String firstField = parameter.split(DOT_SPLIT)[0];
        int idx = ArrayUtils.indexOf(params, firstField);
        Object value = args[idx];
        Class valueClass = value.getClass();
        Object[] fields = ArrayUtils.remove(parameter.split(DOT_SPLIT), 0);
        for (Object field : fields) {
            try {
                Field nextField = valueClass.getDeclaredField(String.valueOf(field));
                nextField.setAccessible(true);
                value = nextField.get(value);
                valueClass = value.getClass();
            } catch (NoSuchFieldException | IllegalAccessException e) {
                log.error(e.getMessage(), e);
            }
        }
        return value;
    }

}
```  

提供一个锁接口，具体可以根据自己的代码实现具体的功能。  

DistributedLock.java  
 
```java
package com.github.lock.way;

/**
 * @author wangdongbo
 * @since 2019/7/22.
 */
public interface DistributedLock {

    /**
     * 加锁
     *
     * @param lockKey    key
     * @param requestId  唯一请求ID
     * @param seconds    锁定时间[秒]
     * @return 是否加锁成功
     */
    boolean lock(String lockKey, String requestId, int seconds);

    /**
     * 尝试加锁
     *
     * @param lockKey     key
     * @param requestId   唯一请求ID
     * @param seconds     锁定时间[秒]
     * @param trySeconds  拿不到锁,等待多久之后再次尝试加锁[秒]
     * @return  是否加锁成功
     */
    boolean tryLock(String lockKey, String requestId, int seconds, int trySeconds);

    /**
     * 释放锁
     *
     * @param lockKey    key
     * @param requestId  唯一请求ID
     * @return  是否成功释放锁
     */
    boolean unlock(String lockKey, String requestId);

}
```  

以上代码就是主要逻辑代码，具体代码实现可以参考工程: [传送门](https://github.com/dongbow/lock)
