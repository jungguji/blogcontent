---
title: 동시성 이슈와 Redis( Redisson )를 이용한 해결방법
date: 2024-12-24 12:26:46
tags:
    - redisson
    - redis
    - 동시성
---

## 도입


동시성 제어는 여러 사용자가 동시에 시스템을 사용할 때 발생할 수 있는 데이터 무결성과 성능 문제를 해결하는 데 중요한 역할을 한다. 본 글은 Redis와 Redisson 라이브러리를 활용하여 분산 환경에서 발생하는 동시성 이슈를 해결하기 위한 방안을 탐구하고자 한다. 본 글에서는 사내에서 발생했던 동시성 문제와 비슷한 상황을 예시 코드로 재현하여 문제를 정의하고, 이를 해결하기 위한 방법을 설명한다.
* * *

## 이론적 배경

1. 동시성 제어와 분산 락의 중요성
   - 동시성 이슈는 분산 환경에서 동일한 데이터에 여러 프로세스가 동시에 접근할 때 발생한다. 이를 해결하기 위해 락(lock) 메커니즘이 널리 사용된다.
   - [일반적인 락(mutex lock, spin lock 등)](https://ko.wikipedia.org/wiki/%EB%9D%BD_(%EC%BB%B4%ED%93%A8%ED%84%B0_%EA%B3%BC%ED%95%99))은 단일 시스템 내에서만 작동하며, 분산 락은 여러 시스템 간의 경쟁 상태를 관리한다.
     - 단일 시스템의 예로는 하나의 WAS(Tomcat)에서 실행되는 애플리케이션이 있고, 이 환경에서는 스레드 간의 동기화가 주로 필요하다.
     - 여러 시스템의 예로는 여러 개의 WAS(Tomcat) 인스턴스가 로드 밸런서를 통해 분산 처리하는 환경이 있으며, 이러한 경우 서버 간 데이터 동기화를 위한 분산 락이 필요하다.


2. Redis와 Redisson
   - Redis는 인메모리 데이터베이스로 높은 처리 속도와 다양한 데이터 구조를 지원한다. 이를 활용한 분산 락 구현은 효율적인 동시성 제어를 제공한다.
   - Redisson은 Redis 클라이언트 라이브러리로, 분산 락을 포함한 다양한 동시성 제어 기능을 제공한다.

## 연구방법

**1. 문제 정의**
   - 챗봇 대화 서비스를 운영중이었고, 유저가 챗봇과 대화 시 대화에 필요한 서비스 내 재화가 소모 되는 방식이었다.
   - 서버 구조는 다음과 같았다:     
     - 메시지큐를 이용해 채팅 서버로 메시지를 전달하고, 답장 서버에서 답장을 메시지큐를 통해 다시 수신하여 처리하는 구조였다.
    {% asset_img 1.png %}
    - 답장 서버는 8대가 운영 중이었고, 유저는 동시에 여러 챗봇과 대화가 가능했다.
    - 메시지큐에 동일한 유저가 여러 챗봇과의 대화를 진행하며 생성된 메시지가 각각 저장되고, 여러 서버에서 이러한 메시지를 동시에 처리하면서 동시성 문제가 발생하여, 유저가 소비해야 하는 재화가 올바르게 차감되지 않는 현상이 나타났다.
  - 이를 재현하기 위해 아래와 같은 코드를 작성하여 실험을 설계하였다.

**2. 실험 설계**
   - 문제를 재현하기 위해 아래 코드를 작성하여 실험을 설계하였다.
   ```java
   public ResponseEntity<String> consumeResource(int userId) {
        Optional<User> userOpt = resourceRepository.findById(userId);
        try {
            Thread.sleep(100); // 로직 수행에 100ms가 걸린다고 가정하고, 100ms 지연
        } catch (InterruptedException e) {
            Thread.currentThread().interrupt();
        }

        User user = userOpt.get();
        user.consumeResource(1);

        resourceRepository.save(user);
        log.info("User {} 자원 1 사용, 남은 자원: {}", userId, user.getRemainingResources());
        return ResponseEntity.ok("자원 소비. 남은 자원: " + user.getRemainingResources());
    }
    ```
    - 자원 소비 중 동시성 문제가 발생하는 현상을 확인하기 위한 예시 코드로, 여러 스레드에서 동일한 유저의 데이터를 동시에 처리할 경우 재화가 예상보다 적게 차감되는 동작을 확인하였다.
    - 이 코드는 단일 시스템에서 동작하는 코드로, 분산 락이 아닌 일반적인 락으로도 해결할 수 있는 상황이다. 하지만 실제 문제와 동일하게 여러 스레드나 서버가 하나의 자원에 동시 접근하는 상황을 재현할 수 있으므로, 실험 결과의 신뢰성에는 문제가 없을 것으로 보인다.
  
**2.1. 실험 결과**
   - [wrk](https://github.com/wg/wrk)를 사용하여 동일 자원에 대해 다수의 요청을 보냈으며, 다음과 같은 결과가 확인되었다.
   - {% asset_img 2.png %}
   - <details>
   <summary>테스트에 사용된 lua script</summary>
      ```lua
         -- 요청 수 제한
         local max_requests = 10 -- 요청 수 제한
         local request_count = 0 -- 현재 요청 수

         request = function()
            if request_count >= max_requests then
               wrk.thread:stop() -- 요청 제한에 도달하면 쓰레드 중지
            end

            request_count = request_count + 1
            local user_id = 1
            local path = "/resources/" .. user_id .. "/consume"
            return wrk.format("POST", path)
         end
      ```
   </details>
   - **실험 명령어:**
      ```shell
      wrk -t5 -c20 -d1s -s test_consume.lua http://localhost:8080
      ```
   - **실험 결과 요약:**
      - 로그에서 동일 자원에 대해 여러 요청이 동시에 처리되며 자원의 남은 개수가 정확히 감소하지 않는 현상이 나타났다.
      - 이는 요청 간 자원 상태가 정확히 반영되지 않고 동일한 초기 상태로 처리된 결과이다.
      - 예를 들어, 로그에서 "User 1 자원 1 사용, 남은 자원: 9"가 반복 출력되었으며, 이후 일부 요청만 자원이 감소된 상태를 올바르게 반영하였다.
   - **문제 분석:**
      - 요청이 짧은 시간에 집중적으로 발생하며, 트랜잭션 커밋이 완료되기 전에 다른 요청이 동일 자원의 상태를 조회하고 처리하였다.
      - 이러한 동시성 이슈는 예시 코드에서 자원 접근 시 락을 사용하지 않았기 때문에 발생한 것으로 보인다.
      - 결과적으로, 여러 요청이 자원을 중복해서 처리하며 데이터의 무결성이 훼손되었다.
  
**3. 문제 해결**
   - **분산 락 설계:**
      - [Redisson](https://redisson.org/docs/data-and-services/locks-and-synchronizers/#lock)의 [tryLock](https://javadoc.io/doc/org.redisson/redisson/latest/org/redisson/api/RLock.html) 메서드를 활용하여 특정 리소스에 대한 락을 획득한다.
      - 락 획득에 실패한 요청은 대기하거나 재시도하도록 구현한다.
      ```java
      @PostMapping("{userId}/consume")
      public ResponseEntity<String> consumeResource(@PathVariable("userId") int userId) {
         String lockName = "user:" + userId + ":resource:lock";

         RLock lock = redissonClient.getLock(lockName);
         int remainingResources = 0;
         try {
               boolean isLocked = lock.tryLock(3, 10, TimeUnit.SECONDS);

               if (isLocked) {
                  remainingResources = resourceService.consumeResource(userId);
               }
         } catch (RuntimeException | InterruptedException e) {
               throw new RuntimeException("Lock 획득 실패", e);
         } finally {
               if (lock.isHeldByCurrentThread()) { // 락 소유 여부 확인
                  lock.unlock();
               }
         }

         return ResponseEntity.ok("자원 소비. 남은 자원: " + remainingResources);
      }
      ```
   - **락 해제 매커니즘:**
      Redisson 내부적으로 락을 관리하고 해제하는 메커니즘은 Redis Pub/Sub 시스템과 AsyncSemaphore를 활용하여 다음과 같은 방식으로 동작한다:

      <details>
         <summary>Redisson tryLock() Code</summary>
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
                        // 2. 타이머 갱신 및 대기
                        time -= System.currentTimeMillis() - current;
                        if (time <= 0L) {
                           this.acquireFailed(waitTime, unit, threadId);
                           return false;
                        } else {
                           current = System.currentTimeMillis();
                           // 3. Pub/Sub 채널 구독
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

                           try {
                              time -= System.currentTimeMillis() - current;
                              if (time <= 0L) {
                                    this.acquireFailed(waitTime, unit, threadId);
                                    boolean var25 = false;
                                    return var25;
                              } else {
                                    boolean var16;
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
                                       
                                       // 4. Latch 대기와 재시도
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
                                    return var16;
                              }
                           } finally {
                              // 5. 구독 해제
                              this.unsubscribe((RedissonLockEntry)this.commandExecutor.getNow(subscribeFuture), threadId);
                           }
                        }
                  }
               }
            ```
         </details>
         
      **1. 최초 락 획득 시도**
      - tryAcquire 메서드를 통해 락 획득 가능 여부를 확인하며, null을 반환하면 현재 락을 바로 획득할 수 있음을 의미한다.
      - 락이 이미 선점된 경우, ttl(잠금의 남은 유효 시간)을 반환하여 대기할 시간을 계산한다.
      **2. 타이머 갱신 및 대기**
      - 대기 중 경과된 시간을 waitTime에서 차감하며 남은 시간이 0 이하일 경우 대기를 종료하고 false를 반환한다.
      **3. Pub/Sub 채널 구독**
      - 락 해제 이벤트를 감지하기 위해 Redis Pub/Sub 채널에 구독을 요청한다.
      - subscribeFuture를 통해 락이 해제되었거나 구독에 실패한 경우 작업이 비동기적으로 처리된다.
      **4. latch 대기와 재시도**
      - Pub/Sub 채널 구독이 성공한 상태에서 tryAcquire를 반복 호출하며 락 획득을 재시도한다.
      - TTL 값에 따라 latch 대기 시간이 설정되며, 다른 스레드에서 락이 해제되면 latch가 해제되어 작업이 이어진다.
      **5. 구독 해제**
      - 락을 획득하든 못 하든 메서드를 종료하기 전에 Pub/Sub 구독 리소스를 해제하여 시스템 자원을 정리한다.

**3.1. Redisson 적용 후 실험 결과:**
   - 2.1. 실험과 동일한 script와 wrk 명령어를 이용해 실험을 진행했으며, 다음과 같은 결과가 확인되었다.
   - {% asset_img 4.png %}
   - **실험 결과 요약:**
      - Redisson을 사용하여 분산 락을 적용한 결과, 모든 요청이 순차적으로 처리되었으며 데이터의 무결성이 유지되었다.
      - 로그에서 확인할 수 있듯이 동일한 자원에 대해 락이 적용되어 요청이 병렬적으로 처리되지 않고 순차적으로 처리되었다.
      - 예를 들어, "User 1 자원 1 사용, 남은 자원: 9"에서 시작하여 요청이 처리될 때마다 자원이 정확히 감소하는 모습을 확인할 수 있었다.
      - 실험 중에는 잘못된 자원 감소나 동시성 문제가 발생하지 않았다.

   - **문제 해결 확인:**
    - Redisson의 분산 락을 적용함으로써 동시성 문제로 인한 데이터 무결성 훼손이 해결되었다.
    - 요청량이 많을 때에도 대기 상태로 처리되며 자원의 상태가 정확히 갱신되었다.

## 결과
- 테스트 결과
   - Redisson 기반 분산 락을 적용한 후, 중복 트랜잭션 문제가 해결되었다.
   - 부하 테스트(wrk)에서 동일한 유저의 데이터를 처리할 때, 데이터 무결성이 유지됨을 확인하였다.
   - 이로써 분산 서버 환경에서 발생하던 자원 동시성 문제를 해결할 수 있었다.
* * *

## 논의

Pub/Sub 기반 락 메커니즘은 성능과 자원 활용면에서 유리하며, 이를 사용하는 Redisson을 사용하여 분산 환경에서도 효율적으로 동시성 제어가 가능하지만,
Redis를 따로 구축하여야 한다는 점, Redis의 장애 시 락 메커니즘이 정상적으로 동작하지 않을 수 있다는 점과 락 해제 실패 또는 락 대기 시간 초과 시 추가적 오류 처리가 필요하다는 한계점 등이 존재한다.
해당 한계점들을 개선하기 위해 이후에 Redis Cluster를 활용해 고가용성을 확보하여야하고, Redis 외에 기타 다른 도구(Zookeeper, Etcd 등)와의 성능 비교가 필요하다.

## 결론

본 글에서는 Redis와 Redisson을 활용한 분산 락 메커니즘을 통해 동시성 이슈를 해결하는 방법을 제시하였다. 
실험 결과, 제안된 방법은 데이터 무결성을 효과적으로 유지하며, 부하 테스트에서도 우수한 성능을 보였다. 
본 글이 분산 환경에서의 동시성 문제를 해결하려는 개발자들에게 실질적인 가이드라인을 제공하길 기대하며, 본 글에서 사용된 `tryLock()`의 내부 작동원리는 [동시성 제어를 위한 Redisson tryLock 메서드의 작동 원리](https://jungguji.github.io/2025/01/09/%EB%8F%99%EC%8B%9C%EC%84%B1-%EC%A0%9C%EC%96%B4%EB%A5%BC-%EC%9C%84%ED%95%9C-Redisson-tryLock-%EB%A9%94%EC%84%9C%EB%93%9C%EC%9D%98-%EC%9E%91%EB%8F%99-%EC%9B%90%EB%A6%AC/)에서 확인 할 수 있다.