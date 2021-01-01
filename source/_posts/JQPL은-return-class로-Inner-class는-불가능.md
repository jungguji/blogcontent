---
layout: posts
title: JQPL은 return class로 Inner class는 불가능
date: 2020-11-10 18:39:23
tags:
    - JPA
    - JPQL
---

## 결론

__JPQL의 return 값으로 Inner class를 지정하면 class를 찾을 수 없다는 에러가 발생 하므로 Inner class 말고 따로 class를 생성하고 그 class를 return class로 지정해줘야 한다.__

* * *

## 발견

아래 처럼 DTO를 생성하고 view별로 inner class로 DTO를 만들고 jpql의 return class로 지정해주었는데 클래스를 찾을 수 없다는 에러가 발생하였다.

### DTO

```java
public class ChannelDTO {

    @NoArgsConstructor
    @Getter
    public static class SubChannelList {
        private Integer subChannel;
        private KillTime killTime;

        @Builder
        public SubChannelList(Integer subChannel, KillTime killTime) {
            this.subChannel = subChannel;
            this.killTime = killTime;
        }
    }

    ...
}
```

### JPQL 호출 메서드

```java
public List<SubChannelList> findSubChannelList(Integer dungeonId, Integer mainChannel) {
    return em.createQuery(
            "select new com.jungguji.windbossgentimer.web.dto.channel.ChannelDTO.SubChannelList" +
                    "(c.subChannel, k)" +
                    " FROM Channel c left join c.killTimes k" +
                    " WHERE c.dungeon.id = :dungeonId AND c.mainChannel = :mainChannel" +
                    "", SubChannelList.class)
            .setParameter("dungeonId", dungeonId)
            .setParameter("mainChannel", mainChannel)
            .getResultList();
}
```

### 발생한 에러

```java
Caused by: org.hibernate.hql.internal.ast.QuerySyntaxException: Unable to locate class [com.jungguji.windbossgentimer.web.dto.channel.ChannelDTO.SubChannelList] [select new com.jungguji.windbossgentimer.web.dto.channel.ChannelDTO.SubChannelList(c.subChannel, k) FROM com.jungguji.windbossgentimer.domain.channel.Channel c left join c.killTimes k WHERE c.dungeon.id = :dungeonId AND c.mainChannel = :mainChannel]
    at org.hibernate.hql.internal.ast.QuerySyntaxException.convert(QuerySyntaxException.java:74)
    at org.hibernate.hql.internal.ast.ErrorTracker.throwQueryException(ErrorTracker.java:93)
    at org.hibernate.hql.internal.ast.QueryTranslatorImpl.analyze(QueryTranslatorImpl.java:282)
    at org.hibernate.hql.internal.ast.QueryTranslatorImpl.doCompile(QueryTranslatorImpl.java:192)
    at org.hibernate.hql.internal.ast.QueryTranslatorImpl.compile(QueryTranslatorImpl.java:144)
    at org.hibernate.engine.query.spi.HQLQueryPlan.<init>(HQLQueryPlan.java:113)
    at org.hibernate.engine.query.spi.HQLQueryPlan.<init>(HQLQueryPlan.java:73)
    at org.hibernate.engine.query.spi.QueryPlanCache.getHQLQueryPlan(QueryPlanCache.java:162)
    at org.hibernate.internal.AbstractSharedSessionContract.getQueryPlan(AbstractSharedSessionContract.java:604)
    at org.hibernate.internal.AbstractSharedSessionContract.createQuery(AbstractSharedSessionContract.java:716)
    ... 73 more
```

* * *

## 해결

Inner class로 되어있던 DTO를 class로 생성한다.

### 따로 생성한 DTO

```java
@NoArgsConstructor
@Getter
public class SubChannelListResponseDTO {
    private Integer subChannel;
    private KillTime killTime;

    @Builder
    public SubChannelListResponseDTO(Integer subChannel, KillTime killTime) {
        this.subChannel = subChannel;
        this.killTime = killTime;
    }
}
```

### 결과

![Junit Test 통과한 모습](source\_posts\JQPL은-return-class로-Inner-class는-불가능\결과.PNG)

* * *

## P.S

* 일반적으로 class명과 table명을 헷갈려 대소문자 때문에 발생하는 에러 이므로 대소문자도 잘 확인 하여야 한다.