---
layout: posts
title: JPA CustomRepository 작성 시 유의사항
date: 2022-12-26 17:03:01
tags:
    - Spring JPA
    - Hibernate
---

## 서론

 패키지 구조를 수정 후, 기존에 정상 작동하던 custom repository에서 에러가 발생하여, 그 원인과 해결 방법에 대해 정리한다.

* * *

## 본론

 코드의 구조는 아래처럼 되어 있다.

### Service

```java
@RequiredArgsConstructor
@Service
public class ReviewFindService {

    private final ReviewRepository reviewRepository;

    public Review findById(long id) {
        return this.reviewRepository.findById(id)
            .orElseThrow(EntityNotFoundException::new);
    }

    public List<Review> findMyReviews() {
        return this.reviewRepository.myReviews();
    }
}
```

### Repository

```java
public interface ReviewRepository extends JpaRepository<Review, Long>, ReviewCustomRepository {

}
```

### Custom Repository

```java
public interface ReviewCustomRepository {

    List<Review> myReviews();
}
```

### Custom Repository Implementation 

```java
@Repository
public class ReviewCustomRepositoryImpl implements ReviewCustomRepository {

    @Override
    public List<Review> myReviews() {
        return new ArrayList<>();
    }
}
```

### 그림으로

{% asset_img "1_구조.png" %}

<br />
 Service에서 Repository를 선언하여 사용하고, Repository는 CustomRepository를 상속받고, CustomRepositroy를 CustomRepositoryImpl가 구현체로 작성된 상태이다.
이때, ReviewFindService의 findMyReviews()를 호출하면, 아래와 같은 exception이 발생하고, 코드가 정상적으로 작동하지 않았다.

```java
Caused by: java.lang.IllegalArgumentException: Failed to create query for method public abstract java.util.List com.jgji.sokdak.domain.review.domain.ReviewCustomRepository.myReviews()! No property 'myReviews' found for type 'Review'
	at org.springframework.data.jpa.repository.query.PartTreeJpaQuery.<init>(PartTreeJpaQuery.java:96)
	at org.springframework.data.jpa.repository.query.JpaQueryLookupStrategy$CreateQueryLookupStrategy.resolveQuery(JpaQueryLookupStrategy.java:119)
	at org.springframework.data.jpa.repository.query.JpaQueryLookupStrategy$CreateIfNotFoundQueryLookupStrategy.resolveQuery(JpaQueryLookupStrategy.java:259)
	at org.springframework.data.jpa.repository.query.JpaQueryLookupStrategy$AbstractQueryLookupStrategy.resolveQuery(JpaQueryLookupStrategy.java:93)
	at org.springframework.data.repository.core.support.QueryExecutorMethodInterceptor.lookupQuery(QueryExecutorMethodInterceptor.java:103)
	... 134 more
Caused by: org.springframework.data.mapping.PropertyReferenceException: No property 'myReviews' found for type 'Review'
	at org.springframework.data.mapping.PropertyPath.<init>(PropertyPath.java:91)
	at org.springframework.data.mapping.PropertyPath.create(PropertyPath.java:438)
	at org.springframework.data.mapping.PropertyPath.create(PropertyPath.java:414)
	at org.springframework.data.mapping.PropertyPath.lambda$from$0(PropertyPath.java:367)
	at java.base/java.util.concurrent.ConcurrentMap.computeIfAbsent(ConcurrentMap.java:330)
	at org.springframework.data.mapping.PropertyPath.from(PropertyPath.java:349)
	at org.springframework.data.mapping.PropertyPath.from(PropertyPath.java:332)
	at org.springframework.data.repository.query.parser.Part.<init>(Part.java:81)
	at org.springframework.data.repository.query.parser.PartTree$OrPart.lambda$new$0(PartTree.java:250)
	at java.base/java.util.stream.ReferencePipeline$3$1.accept(ReferencePipeline.java:195)
	at java.base/java.util.stream.ReferencePipeline$2$1.accept(ReferencePipeline.java:177)
	at java.base/java.util.Spliterators$ArraySpliterator.forEachRemaining(Spliterators.java:948)
	at java.base/java.util.stream.AbstractPipeline.copyInto(AbstractPipeline.java:484)
	at java.base/java.util.stream.AbstractPipeline.wrapAndCopyInto(AbstractPipeline.java:474)
	at java.base/java.util.stream.ReduceOps$ReduceOp.evaluateSequential(ReduceOps.java:913)
	at java.base/java.util.stream.AbstractPipeline.evaluate(AbstractPipeline.java:234)
	at java.base/java.util.stream.ReferencePipeline.collect(ReferencePipeline.java:578)
	at org.springframework.data.repository.query.parser.PartTree$OrPart.<init>(PartTree.java:251)
	at org.springframework.data.repository.query.parser.PartTree$Predicate.lambda$new$0(PartTree.java:384)
	at java.base/java.util.stream.ReferencePipeline$3$1.accept(ReferencePipeline.java:195)
	at java.base/java.util.stream.ReferencePipeline$2$1.accept(ReferencePipeline.java:177)
	at java.base/java.util.Spliterators$ArraySpliterator.forEachRemaining(Spliterators.java:948)
	at java.base/java.util.stream.AbstractPipeline.copyInto(AbstractPipeline.java:484)
	at java.base/java.util.stream.AbstractPipeline.wrapAndCopyInto(AbstractPipeline.java:474)
	at java.base/java.util.stream.ReduceOps$ReduceOp.evaluateSequential(ReduceOps.java:913)
	at java.base/java.util.stream.AbstractPipeline.evaluate(AbstractPipeline.java:234)
	at java.base/java.util.stream.ReferencePipeline.collect(ReferencePipeline.java:578)
	at org.springframework.data.repository.query.parser.PartTree$Predicate.<init>(PartTree.java:385)
	at org.springframework.data.repository.query.parser.PartTree.<init>(PartTree.java:93)
	at org.springframework.data.jpa.repository.query.PartTreeJpaQuery.<init>(PartTreeJpaQuery.java:89)
	... 138 more
```

우선, 구현체인 ReviewCustomRepositoryImpl에 Break point 걸어보니, 아예 ReviewCustomRepositoryImpl의 myReviews()가 호출조차 되지 않는 것을 확인했다.
그 후 이것저것 코드도 수정해 보고 하였는데, ReviewCustomRepository를 따로 선언하여 사용하니 정상적으로 호출되었다.

### Service에서 CustomRepository를 직접 참조

```java
@RequiredArgsConstructor
@Service
public class ReviewFindService {

    private final ReviewRepository reviewRepository;
    private final ReviewCustomRepository reviewCustomRepository;

    public Review findById(long id) {
        return this.reviewRepository.findById(id)
            .orElseThrow(EntityNotFoundException::new);
    }

    public List<Review> findMyReviews() {
        return this.reviewCustomRepository.myReviews();
    }
}

```

기존에 계속 사용하던 구조이기에, CustomRepository를 선언하여 사용하는 방식으로는 문제 해결이 안 된다고 판단하여, 서론에서 말했던 것처럼 최근에 작업한 패키지 구조 수정 중 무언가 문제가 발생하였다고 판단되어 패키지 구조를 열어보니...

<br />

{% asset_img "2_패키지구조.PNG" %}

<br />

위 이미지처럼, Custom Repository interface와 구현체인 Custom Repository Impl 클래스가 서로 다른 패키지에 존재하고 있었고, 

<br />

{% asset_img "3_성공_패키지구조.PNG" %}

<br />

위처럼 같은 패키지 아래 존재 하도록 위치를 조정해주니 정상적으로 동작하는 것을 확인할 수 있었다.
interface는 기본적으로 접근제어자가 public으로 선언되는데, 패키지 위치가 다른 것이 어째서 문제가 되는지 찾아보니.. Spring 공식 문서에서 아래와 같은 가이드를 찾을 수 있었다.

{% blockquote Spring Data JPA - Reference Documentation https://docs.spring.io/spring-data/jpa/docs/current/reference/html/#repositories.configuration Configuration %}
    The repository infrastructure tries to autodetect custom implementation fragments by scanning for classes below the package in which it found a repository. These classes need to follow the naming convention of appending a postfix defaulting to Impl.
{% endblockquote %}
* * *

## 결론

 결론적으로, Custom Repository를 작성할 때는 구현체 클래스에 postfix로 Impl가 붙게 만들어야 하고, interface와 같은 위치, 혹은 하위 패키지에 존재하도록 하여야 한다는 것을 알았다.


* * *

## 참고 사이트

- [https://docs.spring.io/spring-data/jpa/docs/current/reference/html/#repositories.configuration](https://docs.spring.io/spring-data/jpa/docs/current/reference/html/#repositories.configuration)
- [https://stackoverflow.com/questions/47377989/spring-jpa-custom-repository-different-packages](https://stackoverflow.com/questions/47377989/spring-jpa-custom-repository-different-packages)