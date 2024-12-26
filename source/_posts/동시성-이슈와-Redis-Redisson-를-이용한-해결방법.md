---
title: 동시성 이슈와 Redis( Redisson )를 이용한 해결방법
date: 2024-12-24 12:26:46
tags:
    - redisson
    - redis
    - 동시성
---

# 서론

## 도입


동시성 제어는 여러 사용자가 동시에 시스템을 사용할 때 발생할 수 있는 데이터 무결성과 성능 문제를 해결하는 데 중요한 역할을 한다. 본 글은 Redis와 Redisson 라이브러리를 활용하여 분산 환경에서 발생하는 동시성 이슈를 해결하기 위한 방안을 탐구하고자 한다. 본 글에서는 사내에서 발생했던 동시성 문제와 비슷한 상황을 예시 코드로 재현하여 문제를 정의하고, 이를 해결하기 위한 방법을 설명한다.
* * *

# 본론

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

3. 문제 해결
   - 분산 락 설계
     - Redisson의 tryLock 메서드를 활용하여 특정 리소스에 대한 락을 획득.
     - 락 획득에 실패한 요청은 대기하거나 재시도하도록 구현.
   - 락 해제 메커니즘
     - 락 해제 시 Pub/Sub 방식을 사용하여 대기 중인 프로세스에 알림 전달.