---
title: 동시성 제어를 위한 Redisson tryLock 메서드의 작동 원리
date: 2025-01-09 12:42:28
description: Redisson의 tryLock 메서드와 Redis를 활용한 분산 락 구현 및 동시성 문제 해결 방법에 대해 다룹니다.
slug: solving-concurrency-issues-with-redis-and-redisson
keywords:
  - Redisson tryLock 사용법
  - Redis를 활용한 동시성 제어
  - Java 분산 락 구현
  - 분산 시스템 동시성 문제 해결 방법
tags:
  - Redisson
  - Redis
  - 동시성 제어
  - 분산 락
  - Java
  - Spring Boot
  - tryLock
  - Java 동시성 제어
---

## 도입

동시성 제어는 여러 사용자가 동시에 시스템을 사용할 때 데이터 무결성을 유지하고 성능 저하를 방지하는 데 중요한 역할을 한다. 특히 분산 환경에서는 이러한 문제가 더욱 두드러지며, 이를 해결하기 위한 방법으로 Redis와 Redisson 라이브러리가 자주 사용된다. 본 포스팅에서는 동시성 제어를 위해 사용되는 Redisson의 tryLock 메서드가 어떻게 작동하는지 소개한다.
본 포스팅은 [Redisson 3.41.0 버전](https://javadoc.io/doc/org.redisson/redisson/latest/index.html)을 기준으로 작성되었다.

* * *

## 이론적 배경

**1. 동시성 제어의 필요성**
   - 동시성 이슈는 여러 프로세스가 동시에 동일한 자원에 접근할 때 발생한다. 이를 해결하기 위해 락(lock) 메커니즘이 사용된다. 
   - 단일 시스템에서는 일반적인 락으로 해결할 수 있지만, 분산 시스템에서는 분산 락이 필요하다.

**2. Redis와 Redisson**
   - Redis는 인메모리 데이터베이스로, 높은 성능과 다양한 데이터 구조를 제공한다. 
   - Redisson은 Redis 기반의 클라이언트 라이브러리로, 분산 락 및 다양한 동시성 제어 기능을 지원한다.
   - Redisson을 이용한 실제적인 분산락 적용 방법은 이전 포스팅 [동시성 이슈와 Redis( Redisson )를 이용한 해결방법](https://jungguji.github.io/2024/12/24/%EB%8F%99%EC%8B%9C%EC%84%B1-%EC%9D%B4%EC%8A%88%EC%99%80-Redis-Redisson-%EB%A5%BC-%EC%9D%B4%EC%9A%A9%ED%95%9C-%ED%95%B4%EA%B2%B0%EB%B0%A9%EB%B2%95/)에서 확인할 수 있다.

## tryLock 메서드의 작동 원리
Redisson의 tryLock 메서드는 분산 락을 구현하는 데 핵심 역할을 한다. 아래는 주요 작동 원리와 코드에 대한 설명이며, 아래 코드는 Redisson 3.41.0 기준으로 동작한다. 특정 버전에서는 내부 구현이 달라질 수 있으니 공식 문서를  참고하기 바란다.

```java
public boolean tryLock(long waitTime, long leaseTime, TimeUnit unit) throws InterruptedException {
    long time = unit.toMillis(waitTime);
    long current = System.currentTimeMillis();
    long threadId = Thread.currentThread().getId();
    // 1. 최초 락 획득 시도
    Long ttl = this.tryAcquire(waitTime, leaseTime, unit, threadId);
    if (ttl == null) {
        return true;
    } else {
        // 2. 타이머 갱신
        time -= System.currentTimeMillis() - current;
        if (time <= 0L) {
            this.acquireFailed(waitTime, unit, threadId);
            return false;
        } else {
            // 3. 락에 대한 pub/sub 채널 구독
            current = System.currentTimeMillis();
            CompletableFuture<RedissonLockEntry> subscribeFuture = this.subscribe(threadId);

            try {
                subscribeFuture.get(time, TimeUnit.MILLISECONDS);
            } catch (TimeoutException var21) {
                if (!subscribeFuture.completeExceptionally(new RedisTimeoutException("Unable to acquire subscription lock after " + time + "ms. Try to increase 'subscriptionsPerConnection' and/or 'subscriptionConnectionPoolSize' parameters."))) {
                    subscribeFuture.whenComplete((res, ex) -> {
                        if (ex == null) {
                            this.unsubscribe(res, threadId);
                        }

                    });
                }

                this.acquireFailed(waitTime, unit, threadId);
                return false;
            } catch (ExecutionException var22) {
                ExecutionException e = var22;
                LOGGER.error(e.getMessage(), e);
                this.acquireFailed(waitTime, unit, threadId);
                return false;
            }

            boolean var16;
            try {
                time -= System.currentTimeMillis() - current;
                if (time <= 0L) {
                    this.acquireFailed(waitTime, unit, threadId);
                    boolean var25 = false;
                    return var25;
                }

                do {
                    long currentTime = System.currentTimeMillis();
                    ttl = this.tryAcquire(waitTime, leaseTime, unit, threadId);
                    if (ttl == null) {
                        var16 = true;
                        return var16;
                    }

                    time -= System.currentTimeMillis() - currentTime;
                    if (time <= 0L) {
                        this.acquireFailed(waitTime, unit, threadId);
                        var16 = false;
                        return var16;
                    }

                    currentTime = System.currentTimeMillis();
                    if (ttl >= 0L && ttl < time) {
                        ((RedissonLockEntry)this.commandExecutor.getNow(subscribeFuture)).getLatch().tryAcquire(ttl, TimeUnit.MILLISECONDS);
                    } else {
                        ((RedissonLockEntry)this.commandExecutor.getNow(subscribeFuture)).getLatch().tryAcquire(time, TimeUnit.MILLISECONDS);
                    }

                    time -= System.currentTimeMillis() - currentTime;
                } while(time > 0L);

                this.acquireFailed(waitTime, unit, threadId);
                var16 = false;
            } finally {
                this.unsubscribe((RedissonLockEntry)this.commandExecutor.getNow(subscribeFuture), threadId);
            }

            return var16;
        }
    }
}
```

### 1. 최초 락 획득 시도
```java
Long ttl = this.tryAcquire(waitTime, leaseTime, unit, threadId);
if (ttl == null) {
    return true;
} else {
    // ttl != null이면 락이 선점되어 있음을 의미
    ...
}
```
- `this.tryAcquire()`는  락 획득 가능 여부를 확인한다.
  - 반환값이 `null`인 경우, 현재 락을 바로 획득할 수 있음을 의미한다.
  - 반환값이 `null`이 아닌 경우, 이미 락이 선점되었거나 대기가 필요함을 나타낸다.

### 2. 타이머 갱신
```java
time -= System.currentTimeMillis() - current;
if (time <= 0L) {
	this.acquireFailed(waitTime, unit, threadId);
	return false;
} else {
	...
}
```

- 남은 대기 시간을 갱신하며, 시간이 0 이하인 경우 락 획득을 포기한다.

### 3. 락에 대한 pub/sub 채널 구독
```java
current = System.currentTimeMillis();
CompletableFuture<RedissonLockEntry> subscribeFuture = this.subscribe(threadId);

try {
    subscribeFuture.get(time, TimeUnit.MILLISECONDS);
} catch (TimeoutException var21) { 
	... 
} catch (ExecutionException var22) { 
	... 
}
```

- `subscribe()` 는 Pub/Sub를 통해 락 이벤트를 감지할 준비를 한다.
- `subscribeFuture.get()`을 호출해 설정된 시간 동안 구독이 성공하기를 기다린다.
	- 시간이 초과되면(`TimeoutException`) 구독을 종료하고 `false`를 반환한다.
	- `ExecutionException`이 발생하면 에러를 로깅하고 `false`를 반환한다.

```java
protected CompletableFuture<RedissonLockEntry> subscribe(long threadId) {
	return this.pubSub.subscribe(this.getEntryName(), this.getChannelName());
}
```

- `threadId`는 구독 채널 생성에 직접 사용되지 않고, 락 이름(lock name)과 채널 ID를 기반으로 Pub/Sub 채널이 생성된다.
- 이 과정은 락 해제 이벤트를 감지하기 위해 필수적이다.

```java
public CompletableFuture<E> subscribe(String entryName, String channelName) {
    // a. Semaphore 생성 및 acquire 호출
    AsyncSemaphore semaphore = this.service.getSemaphore(new ChannelName(channelName));
    CompletableFuture<E> newPromise = new CompletableFuture();
    semaphore.acquire().thenAccept((c) -> {
        if (newPromise.isDone()) {
            semaphore.release();
        } else {
            // b. Pub/Sub 채널 확인 및 생성
            E entry = (PubSubEntry)this.entries.get(entryName);
            if (entry != null) {
                // (1) 기존 엔트리가 있는 경우
                entry.acquire();
                semaphore.release();
                entry.getPromise().whenComplete((r, e) -> {
                    if (e != null) {
                        newPromise.completeExceptionally(e);
                    } else {
                        newPromise.complete(r);
                    }
                });
            } else {
                // (2) 새로운 엔트리를 생성
                E value = this.createEntry(newPromise);
                value.acquire();
                E oldValue = (PubSubEntry)this.entries.putIfAbsent(entryName, value);
                if (oldValue != null) {
                    oldValue.acquire();
                    semaphore.release();
                    oldValue.getPromise().whenComplete((r, e) -> {
                        if (e != null) {
                            newPromise.completeExceptionally(e);
                        } else {
                            newPromise.complete(r);
                        }
                    });
                } else {
                    // (3) Redis Pub/Sub 연결 로직 수행
                    RedisPubSubListener<Object> listener = this.createListener(channelName, value);
                    CompletableFuture<PubSubConnectionEntry> s = this.service.subscribeNoTimeout(LongCodec.INSTANCE, channelName, semaphore, new RedisPubSubListener[]{listener});
                    newPromise.whenComplete((r, e) -> {
                        if (e != null) {
                            s.completeExceptionally(e);
                        }

                    });
                    // (4) Pub/Sub 구독 상태 관리 및 실패 처리
                    s.whenComplete((r, e) -> {
                        if (e != null) {
                            this.entries.remove(entryName);
                            value.getPromise().completeExceptionally(e);
                        } else {
                            if (!value.getPromise().complete(value) && value.getPromise().isCompletedExceptionally()) {
                                this.entries.remove(entryName);
                            }

                        }
                    });
                }
            }
        }
    });
    return newPromise;
}
```

**a. Semaphore 생성 및 acquire 호출**

```java
AsyncSemaphore semaphore = this.service.getSemaphore(new ChannelName(channelName));
CompletableFuture<E> newPromise = new CompletableFuture();
semaphore.acquire().thenAccept((c) -> {
    if (newPromise.isDone()) {
        semaphore.release();
    } else {
        ...
    }
    ...
});
```

- `getSemaphore()`를 통해 해당 채널 이름에 대한 비동기 세마포어를 가져온다.
	- 여러 스레드나 인스턴스에서 동일한 채널에 구독 요청이 들어올 수 있으므로, 세마포어를 사용해 동시성을 제어한다.
- `acquire()`를 호출하여 락 획득을 시도한다. 
	- 이 작업은 비동기로 수행되며, 락 획득이 완료되면 `thenAccept` 블록이 실행된다.
	- 이 때 획득 시도하는 락은 **채널 구독을 위한 락**이다.
	- 구독 요청은 `AsyncSemaphore` 내부의 큐([FastRemovalQueue](https://www.javadoc.io/static/org.redisson/redisson/3.41.0/org/redisson/misc/FastRemovalQueue.html))를 통해 순서대로 처리된다.
- 작업 완료 상태 확인 후 락 해제:
	- 만약 `newPromise.isDone()` 상태라면, 작업이 이미 완료된 것이므로 락을 해제한다.
    - 이는 동일한 채널을 구독 중인 다른 스레드가 작업을 이미 완료했음을 감지하기 위함이다.

**b. Pub/Sub 채널 확인 및 생성**
```java
E entry = (PubSubEntry)this.entries.get(entryName);
if (entry != null) {
    // (1) 기존 엔트리가 있는 경우
    entry.acquire();
    semaphore.release();
    entry.getPromise().whenComplete((r, e) -> {
        if (e != null) {
            newPromise.completeExceptionally(e);
        } else {
            newPromise.complete(r);
        }
    });
} else {
    // (2) 새로운 엔트리를 생성
    E value = this.createEntry(newPromise);
    value.acquire();
    E oldValue = (PubSubEntry)this.entries.putIfAbsent(entryName, value);
    if (oldValue != null) {
        oldValue.acquire();
        semaphore.release();
        oldValue.getPromise().whenComplete((r, e) -> {
            if (e != null) {
                newPromise.completeExceptionally(e);
            } else {
                newPromise.complete(r);
            }
        });
    } else {
        // (3) Redis Pub/Sub 연결 로직 수행
        RedisPubSubListener<Object> listener = this.createListener(channelName, value);
        CompletableFuture<PubSubConnectionEntry> s = this.service.subscribeNoTimeout(LongCodec.INSTANCE, channelName, semaphore, new RedisPubSubListener[]{listener});
        newPromise.whenComplete((r, e) -> {
            if (e != null) {
                s.completeExceptionally(e);
            }

        });
        // (4) Pub/Sub 구독 상태 관리 및 실패 처리
        s.whenComplete((r, e) -> {
            if (e != null) {
                this.entries.remove(entryName);
                value.getPromise().completeExceptionally(e);
            } else {
                if (!value.getPromise().complete(value) && value.getPromise().isCompletedExceptionally()) {
                    this.entries.remove(entryName);
                }

            }
        });
    }
}
```

- `lock key(entryName)`로 Pub/Sub 채널을 확인하거나 생성한다.
  - `entries` 맵에서 `entryName`에 해당하는 채널을 가져오고, 이미 존재하면 재사용한다.
  - 채널이 없을 경우 새로 생성해 등록한다.
  

**(1) 기존 채널이 있는 경우:**

```java
entry.acquire();
semaphore.release();
entry.getPromise().whenComplete((r, e) -> {
    if (e != null) {
        newPromise.completeExceptionally(e);
    } else {
        newPromise.complete(r);
    }
});
```

- entry.acquire()를 호출해 채널을 재사용한다.
- 이후 semaphore.release()로 채널 구독을 위한 락을 해제한다.
- 작업 완료 시 entry.getPromise()로 결과를 처리한다.


**(2) 새 채널을 생성하는 경우:**

```java
E value = this.createEntry(newPromise);
value.acquire();
E oldValue = (PubSubEntry)this.entries.putIfAbsent(entryName, value);
if (oldValue != null) {
    oldValue.acquire();
    semaphore.release();
    oldValue.getPromise().whenComplete((r, e) -> {
        if (e != null) {
            newPromise.completeExceptionally(e);
        } else {
            newPromise.complete(r);
        }
    });
```

- `createEntry(newPromise)`로 새로운 엔트리를 생성한다.
- [putIfAbsent(entryName, value)](https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/concurrent/ConcurrentHashMap.html#putIfAbsent(K,V))를 통해 이미 생성된 엔트리가 있는지 확인한다.
- 기존 엔트리가 있으면 이를 사용하고, 없으면 새로 생성한 엔트리를 등록한다.


**(3) Redis Pub/Sub 채널 연결:**

```java
RedisPubSubListener<Object> listener = this.createListener(channelName, value);
CompletableFuture<PubSubConnectionEntry> s = this.service.subscribeNoTimeout(LongCodec.INSTANCE, channelName, semaphore, new RedisPubSubListener[]{listener});
newPromise.whenComplete((r, e) -> {
    if (e != null) {
        s.completeExceptionally(e);
    }

});
```

- `createListener(channelName, value)`를 통해 실제 Redis Pub/Sub 채널을 구독한다.
- `subscribeNoTimeout()`로 비동기적으로 연결을 시도한다.

**(4) Pub/Sub 구독 상태 관리 및 실패 처리:**


```java
s.whenComplete((r, e) -> {
    if (e != null) {
        this.entries.remove(entryName);
        value.getPromise().completeExceptionally(e);
    } else {
        if (!value.getPromise().complete(value) && value.getPromise().isCompletedExceptionally()) {
            this.entries.remove(entryName);
        }

    }
});
```

- **구독 실패 처리:**
  - `e != null` 조건은 Pub/Sub 구독 중 예외가 발생했음을 의미한다.
  - 실패한 경우, 다음 작업을 수행한다:
    - `this.entries.remove(entryName)`: 실패한 구독에 해당하는 `entryName`을 `entries` 맵에서 제거한다.
    - `value.getPromise().completeExceptionally(e)`: 예외를 호출자에게 전달하여 구독 실패 원인을 알린다.
- **구독 성공 상태 관리:**
  - 구독이 성공적으로 완료되었더라도, 추가적인 상태 확인을 수행한다:
  - `value.getPromise().complete(value)`: `value`를 `Promise`로 완료하고 성공 여부를 반환한다.
  - `Promise`가 이미 완료되었거나(`isCompletedExceptionally`로 확인) 정상적으로 완료되지 않은 경우:
    - `this.entries.remove(entryName)`: 일관성을 유지하기 위해 해당 엔트리를 제거한다.
- **상태 확인의 목적:**
  - Redis Pub/Sub 구독 실패 및 이상 상태를 감지하고, 관련 자원을 정리하여 시스템 일관성을 보장한다.
  - 실패나 이상 상태 시 적절한 정리 작업이 이루어지도록 설계되어 있다.

### c. 락에 대한 Pub/Sub 채널 구독 정리

**1. Semaphore 생성 및 acquire 호출**
   - `AsyncSemaphore`는 채널 단위의 subscribe 작업에 대한 동시성을 제어한다.
   - `acquire()` 호출 시, 내부적으로 대기 큐가 생성되어 락이 해제될 때 대기 중인 작업이 순차적으로 처리된다.
   - 이를 통해 여러 스레드가 동일한 채널에 대해 동시에 subscribe 요청을 보내더라도 안전하게 작업이 처리된다.
   - 작업 완료 상태(`newPromise.isDone()`) 확인 후, 이미 완료된 작업에 대한 락을 해제한다.

**2. Pub/Sub 채널 확인 및 생성**
  - entries 맵에서 entryName에 해당하는 채널을 가져오고, 이미 존재하면 이를 재사용한다.
  - 존재하지 않을 경우:
    - `createEntry(newPromise)`로 새로운 엔트리를 생성하여 등록한다.
    - `putIfAbsent(entryName, value)`를 통해 중복 등록을 방지하고, 기존 엔트리가 있으면 이를 재사용한다.
    - 기존 엔트리와 새 엔트리는 동일한 처리 로직으로 이어지며, 작업이 완료되면 결과를 호출자에게 전달한다.

**3. Redis Pub/Sub 채널 연결**
  - 새로운 엔트리에 대해 `createListener(channelName, value)`를 통해 Redis Pub/Sub 채널을 구독한다.
  - `subscribeNoTimeout()` 를 사용하여 비동기적으로 구독을 시도한다.
  - 구독 중 예외 발생 시, 구독 결과를 처리하는 `CompletableFuture`에 예외 상태를 전달한다.
  
**4. Pub/Sub 구독 상태 관리 및 실패 처리**
  - Pub/Sub 구독이 성공했는지 여부를 확인하고, 실패 또는 이상 상태가 발생하면 관련 자원을 정리한다:
    - `e != null(구독 실패)`일 경우:
      - entries 맵에서 해당 entryName을 제거한다.
      - `value.getPromise().completeExceptionally(e)`로 구독 실패 원인을 호출자에게 전달한다.
    - 구독이 성공했더라도:
      - `value.getPromise().complete(value)` 호출로 성공 여부를 확인한다.
      - 성공하지 못했거나(`isCompletedExceptionally`) 예외 상태라면, entries에서 해당 엔트리를 제거한다.
  - 이를 통해 시스템의 데이터 일관성을 유지하고, 자원 누수를 방지한다.
  
결과적으로
- `Semaphore`를 사용해 subscribe 요청의 동시성을 관리한다.
- Pub/Sub 채널의 상태를 확인하거나 생성하고, Redis 채널과 비동기적으로 연결한다.
- 구독 성공 및 실패 상태를 처리하고, 필요시 자원을 정리하여 시스템의 안정성을 보장한다.

여기까지가 락 이벤트를 감지하기 위한 Pub/Sub 채널에 구독하기 위한 로직이다. 

### 4. 락 획득 재시도 및 Latch 대기 로직
```java
do {
    long currentTime = System.currentTimeMillis();
    ttl = this.tryAcquire(waitTime, leaseTime, unit, threadId);
    if (ttl == null) {
        var16 = true;
        return var16;
    }

    time -= System.currentTimeMillis() - currentTime;
    if (time <= 0L) {
        this.acquireFailed(waitTime, unit, threadId);
        var16 = false;
        return var16;
    }

    currentTime = System.currentTimeMillis();
    if (ttl >= 0L && ttl < time) {
        ((RedissonLockEntry)this.commandExecutor.getNow(subscribeFuture)).getLatch().tryAcquire(ttl, TimeUnit.MILLISECONDS);
    } else {
        ((RedissonLockEntry)this.commandExecutor.getNow(subscribeFuture)).getLatch().tryAcquire(time, TimeUnit.MILLISECONDS);
    }

    time -= System.currentTimeMillis() - currentTime;
} while(time > 0L);

this.acquireFailed(waitTime, unit, threadId);
var16 = false;
```

**a. 락 획득 재시도**
  - Pub/Sub 채널 구독이 성공한 상태에서 다시 tryAcquire를 반복 호출해 락 획득을 재시도한다.
  - `ttl == null`일 경우, 즉시 락을 획득할 수 있으므로 true를 반환하며 종료한다.
  
**b. 남은 시간 갱신**
  - 락 획득을 재시도하기 전에 현재 시간을 기준으로 남은 대기 시간을 갱신한다(`time -= ...`).
  - 남은 대기 시간이 없으면(`time <= 0`), `acquireFailed()` 를 호출해 실패 처리를 진행한 후 `false`를 반환한다.

**c. Latch 대기**
  - `ttl` 값에 따라 `Latch`를 사용해 일정 시간 대기한다.
    - `ttl >= 0`**인 경우**: 락 해제까지의 남은 시간 동안 대기한다.
    - `ttl < 0` **또는 현재 남은 시간 이하인 경우**: 설정된 대기 시간까지 대기한다.
	- `Latch`는 `다른 스레드/프로세스가 락을 해제하거나 갱신하는 이벤트`가 발생하면 해제되는 동기화 장치로 사용되며, Redisson에서는 `Semaphore`를 Latch를 사용한다.

**d. 반복 조건**
  - `do-while` 루프를 통해 남은 대기 시간이 소진될 때까지 락 획득을 반복 시도한다.
  - 대기 시간이 소진되면 `acquireFailed()`를 호출하고 `false`를 반환하며 종료한다.

### 5. 구독 해제

```java
} finally {
    this.unsubscribe((RedissonLockEntry)this.commandExecutor.getNow(subscribeFuture), threadId);
}
```

- Pub/Sub 채널을 구독했으므로, 메서드를 종료하기 전에 반드시 unsubscribe 메서드를 호출해 리소스를 해제한다.
- 이는 락 획득 여부와 관계없이 실행되며, Pub/Sub 구독을 통해 사용된 자원을 정리하는 역할을 한다.
* * *

## tryLock()의 장점 및 한계

### 장점
1. 효율적인 이벤트 기반 처리
   - Pub/Sub 메커니즘을 활용하여 락 해제 이벤트를 감지함으로써, 불필요한 대기 시간을 최소화한다.
   - 락이 해제될 가능성이 있는 경우 반복적으로 대기하거나 재시도하지 않고 이벤트 기반으로 처리하므로 자원을 효율적으로 사용한다.

2. 정교한 타이머 관리
   - 남은 대기 시간을 지속적으로 갱신하며, 설정된 제한 시간 내에 락 획득 여부를 결정한다.
   - 대기 시간이 초과될 경우 즉시 실패 처리를 진행하므로, 클라이언트 응답 지연을 방지한다.

3. 안정적인 자원 정리
   - 락 획득 여부와 관계없이 finally 블록을 통해 모든 구독 리소스를 해제하여 시스템 자원의 안정성을 유지한다.

4. Latch를 활용한 동기화
   - 락 획득 시도가 실패하더라도, Latch를 통해 TTL 동안 대기하며 락 해제 가능성을 최대한 활용한다.
  
### 한계

1. 락 획득 시도와 Pub/Sub의 비동기 처리 간의 간격
   - 락 획득 재시도와 Pub/Sub 이벤트 처리 사이의 간격에서 `레이스 컨디션`이 발생할 가능성이 있다. 락이 해제된 후 새로 락이 설정되는 순간에 간격이 생길 수 있으므로, 타이밍에 민감한 경우 신중한 설계가 필요하다.
   <details>
   <summary>레이스 컨디션 발생 원인</summary>
   ```java
    do {
        long currentTime = System.currentTimeMillis();
        ttl = this.tryAcquire(waitTime, leaseTime, unit, threadId);
        if (ttl == null) {
            var16 = true;
            return var16;
        }

        time -= System.currentTimeMillis() - currentTime;
        if (time <= 0L) {
            this.acquireFailed(waitTime, unit, threadId);
            var16 = false;
            return var16;
        }

        currentTime = System.currentTimeMillis();
        if (ttl >= 0L && ttl < time) {
            ((RedissonLockEntry)this.commandExecutor.getNow(subscribeFuture)).getLatch().tryAcquire(ttl, TimeUnit.MILLISECONDS);
        } else {
            ((RedissonLockEntry)this.commandExecutor.getNow(subscribeFuture)).getLatch().tryAcquire(time, TimeUnit.MILLISECONDS);
        }

        time -= System.currentTimeMillis() - currentTime;
    } while(time > 0L);
   ```

    **a. 첫 번째 tryAcquire 호출**

    ```java
        ttl = this.tryAcquire(waitTime, leaseTime, unit, threadId);
     ```

       - 현재 스레드에서 락을 획득하려 시도하지만, 락이 이미 다른 스레드에 의해 점유된 상태라 TTL(남은 유효 시간)을 반환받는다.
       - 이 시점에서 현재 스레드는 락 해제 이벤트를 기다릴 준비한다.
  
    **b. 락 반납 발생**
       - 락을 점유하고 있던 스레드(또는 프로세스)가 락을 반납한다.
       - 이 시점에서 락은 잠깐 동안 해제된 상태가 된다.
  
    **c. 다른 스레드가 락을 채감**
       - 락 해제 이벤트가 Pub/Sub를 통해 현재 스레드에 전달되기 전에, 다른 스레드가 `tryAcquire()`를 호출하여 락을 빠르게 획득한다.
       - 이로 인해 현재 스레드는 락 해제 이벤트를 감지하지 못하게 된다.
  
    **d. 현재 스레드의 Pub/Sub 대기**
    
        ```java
        ((RedissonLockEntry) this.commandExecutor.getNow(subscribeFuture))
        .getLatch().tryAcquire(ttl, TimeUnit.MILLISECONDS);
        ```

        - 현재 스레드는 여전히 락 해제 이벤트를 기다리지만, 락은 이미 다른 스레드에 의해 점유된 상태이다.
        - 이로 인해 현재 스레드는 대기 시간을 낭비하거나, 최종적으로 time이 초과되어 락 획득에 실패할 수 있다.
    </detail>


2. Pub/Sub 메시지 손실 가능성
   - Pub/Sub 시스템 특성상 구독자와 퍼블리셔 간 네트워크 이슈가 발생하면 락 해제 메시지를 놓칠 가능성이 있다. 이 경우 락을 획득하지 못하고 대기 시간을 낭비할 위험이 있다.

3. 복잡한 로직으로 인한 디버깅 난이도
   - tryLock()의 로직이 복잡하여, 장애 발생 시 원인을 파악하거나 디버깅하는 데 시간이 소요될 수 있다.
  
## 결론

Redisson의 `tryLock()`은 분산 환경에서 동시성 제어를 효과적으로 구현하기 위해 설계된 복잡한 로직을 제공한다.

`tryLock()`은 동시성 제어에서 효율성과 안정성을 동시에 추구하는 설계로, 분산 환경에서의 락 처리 문제를 효과적으로 해결한다. 그러나 복잡한 로직 구조와 Pub/Sub 시스템의 특성을 고려하여 사용 환경에 적합한 설정을 적용하고, 잠재적인 한계를 관리할 수 있는 보완책을 함께 설계해야 한다.

본 글이 `tryLock()` 내부 로직에 대한 깊은 이해를 돕고, 동시성 제어가 필요한 시스템 설계 시 참고 자료가 되길 바라며, 실제적으로 Redisson을 활용하여 분산락을 적용하고자 하는 사용자는 이전 글 [동시성 이슈와 Redis( Redisson )를 이용한 해결방법](https://jungguji.github.io/2024/12/24/%EB%8F%99%EC%8B%9C%EC%84%B1-%EC%9D%B4%EC%8A%88%EC%99%80-Redis-Redisson-%EB%A5%BC-%EC%9D%B4%EC%9A%A9%ED%95%9C-%ED%95%B4%EA%B2%B0%EB%B0%A9%EB%B2%95/)을 참고하길 바란다.