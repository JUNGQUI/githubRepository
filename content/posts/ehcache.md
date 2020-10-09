---
title: "Ehcache"
date: 2020-10-09T20:05:43+09:00
draft: true
tags: ["programming", "java"]
---

## Ehcache?

cache 다. [위키피디아](https://ko.wikipedia.org/wiki/캐시) 에 다음과 같이 정의되어 있다.

>캐시(cache, 문화어: 캐쉬, 고속완충기, 고속완충기억기)는 컴퓨터 과학에서 데이터나 값을 미리 복사해 놓는 임시 장소를 가리킨다. 
>
>캐시는 캐시의 접근 시간에 비해 원래 데이터를 접근하는 시간이 오래 걸리는 경우나 값을 다시 계산하는 시간을 절약하고 싶은 경우에 사용한다. 캐시에 데이터를 미리 복사해 놓으면 계산이나 접근 시간 없이 더 빠른 속도로 데이터에 접근할 수 있다.
>
>캐시는 시스템의 효율성을 위해 여러 분야에서 두루 쓰이고 있다.

보통 web 내에서 정적인 데이터나 변하지 않은 데이터를 빠른 시간 내에 로딩하게 위해 web cache 를 가장 많이 접해 봤을 것이다.

일반적으로 WAS - DB 간 통신의 비용이 몹시 비싸기 때문에 (큰 쿼리를 날려서 결과를 받아오는 과정) 
동일한 요청에 대해 해당하는 response 를 WAS 내 memory 나 disk 저장하고 해당 정보를 통해 비용을 줄이고 빠른 대응을 위해 
cache 를 사용한다.

spring 은 3.1 부터 EHcache 를 이용하여 cache 를 활용할 수 있게 만들었다.

## 왜 알아야 하나?

사실 cache 에 대해 잘 활용하는 케이스는 못 봤다. (아직까지) 개인적으로 생각하기엔 cache 를 잘 쓰려면 최대한 변화가 없는
정적인 data 에 대한 controller 라던가 하는 식으로 사용이 가능할 것이라 생각되는데 사실 서비스를 개발하다 보면 그러한 형식의
controller 가 생각보다 많이 없고, 있다 하더라도 그러한 controller 는 대부분 cache 를 쓰지 않아도 performance 가
걱정되는 수준은 아니기 때문에 사용하기에 무리가 있다고 생각한다.

하지만 늘 그렇듯이 잘 사용하면 도움이 될 것은 명확하기에 공부를 해야 한다.

## 구성

우선 gradle 에 cache 관련 library 를 설정해야 한다.

```java
...
compile('org.springframework.boot:spring-boot-starter-cache')
// https://mvnrepository.com/artifact/org.ehcache/ehcache
// net.sf.ehcache 가 2018년 10월부터 org.ehcache 로 변경되었다.
compile group: 'org.ehcache', name: 'ehcache', version: '3.9.0'
...
```

`spring-boot-starter-cache` 를 통해 annotation 을 이용한 cache enable 및 설정 등이 가능해진다.
또한 cache 는 ehcache 를 이용하기 위해 ehcache library 를 사용한다.

가장 중요한 cache 를 생성해야 하는데 `ehcache.xml` 라는 이름으로 아래와 같이 생성하였다.

([여기](https://jojoldu.tistory.com/57) 에서 참고하였다.)

```xml
<ehcache xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:noNamespaceSchemaLocation="http://ehcache.org/ehcache.xsd"
  updateCheck="false">
  <diskStore path="java.io.tmpdir" />

  <!--
  sampleCache1 캐시. 최대 10000개의 객체를 저장할 수 있으며,
  5분 이상 사용되지 않거나 또는 10분 이상 캐시에 저장되어 있을 경우
  캐시에서 제거된다. 저장되는 객체가 10000개를 넘길 경우,
  디스크 캐시에 저장한다.
  -->
  <cache name="sampleCache1"
    maxElementsInMemory="10000"
    maxElementsOnDisk="1000"
    eternal="false"
    overflowToDisk="true"
    timeToIdleSeconds="300"
    timeToLiveSeconds="600"
    memoryStoreEvictionPolicy="LFU"
  />

  <!--
  sampleCache2 캐시. 최대 1000개의 객체를 저장한다.
  오버플로우 된 객체를 디스크에 저장하지 않기 때문에
  캐시에 최대 개수는 1000개이다. eternal이 true 이므로,
  timeToLiveSeconds와 timeToIdleSeconds 값은 무시된다.
  -->
  <cache name="sampleCache2"
    maxElementsInMemory="1000"
    eternal="true"
    overflowToDisk="false"
    memoryStoreEvictionPolicy="FIFO"
  />

  <!--
  sampleCache3 캐시. 오버플로우 되는 객체를 디스크에 저장한다.
  디스크에 저장된 객체는 VM이 재가동할 때 다시 캐시로 로딩된다.
  디스크 유효성 검사 쓰레드는 10분 간격으로 수행된다.
  -->
  <cache name="sampleCache3"
    maxElementsInMemory="500"
    eternal="false"
    overflowToDisk="true"
    timeToIdleSeconds="300"
    timeToLiveSeconds="600"
    diskPersistent="true"
    diskExpiryThreadIntervalSeconds="600"
    memoryStoreEvictionPolicy="LFU"
  />

</ehcache>
```

이렇게 cache 를 상황에 따라 다르게 생성함으로 그때 그때 사용이 가능하다.

이제 실제로 controller 에 cache 를 적용한 부분을 보자면

#### controller
```java
@EnableCaching // 해당 class 에 cache 를 적용하겠다는 annotation
@RestController
@RequiredArgsConstructor
public class JKEhcache {

	private final TestObjectRepository testObjectRepository;

	@GetMapping(value = "/cache/{name}")
	public void ehcache(@PathVariable(value = "name") String name) {
		long startTime = System.currentTimeMillis();
		testObjectRepository.findByNameCache(name);
		long endTime = System.currentTimeMillis();
		System.out.println(name + " printed, " + (endTime - startTime));
	}

	@GetMapping(value = "/nocache/{name}")
	public void noEhcache(@PathVariable(value = "name") String name) {
		long startTime = System.currentTimeMillis();
		testObjectRepository.findByNameNoCache(name);
		long endTime = System.currentTimeMillis();
		System.out.println(name + " printed, " + (endTime - startTime));
	}

	@GetMapping(value = "/cache/refresh/{name}")
	public void refresh(@PathVariable(value = "name") String name) {
		testObjectRepository.refresh(name);
	}
}
```
#### service
```java
@Override
public TestObject findByNameNoCache(String name) {
    slowQuery();
    return TestObject.builder()
            .name(name)
            .createdDate(new Date())
            .build();
}

@Override
@Cacheable(value = "sampleCache1", key="#name") // sampleCache1 이란 이름의 cache 를 사용 할 것이며, name 을 기준으로 cache 를 적용한다.
public TestObject findByNameCache(String name) {
    slowQuery();
    return TestObject.builder()
            .name(name)
            .createdDate(new Date()) // createdDate 가 최초 cache 로 인해 생성된 값이 return 된다.
            .build();
}

@Override
@CacheEvict(value = "sampleCache1", key="#name") // 위와 동일하나 sampleCache1 을 초기화 한다.
public void refresh(String name) {
    System.out.println("cache clear");
}
```

이와 같이 간단한 annotation 몇 가지를 이용해서 cache 를 활성화 할 수 있다.

## 그래서?

앞서 이야기 했듯이 사실 정적인 data 가 있을 때만 사용 할 수 있는데 위와 같은 logic 에서 본인은 동일한
이름의 data 를 select 했지만, 실제 data 자체가 변형이 되어 (다른 controller 를 통해) 다른 값을
내놓게 된다면 log 를 봐도 틀렸는지 아닌지 확인하기가 굉장히 어렵다.

이렇기 때문에 잘 알고 사용해야 하고, business logic 자체도 빠삭(?)하게 알아야 한다.

다만 잘만 사용한다면 정말 performance 적으로는 더할 나위 없이 큰 개선을 기대할 수 있다.

실제로 예시로 사용해보니 딜레이 없이 바로 호출이 되었고, data 자체도 이전에 생성한 data 가 호출되었다. 

[source code](https://github.com/JUNGQUI/spring)

[참고 사이트](https://jojoldu.tistory.com/57)