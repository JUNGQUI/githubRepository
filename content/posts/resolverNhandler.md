---
title: "ResolverNhandler"
date: 2020-10-20T21:28:13+09:00
draft: true
tags: ["programming", "java"]
---

## Resolver & Handler?

뜻으로만 보면 resolve 역할을 수행하는 어떠한 것 (er) 이라고 볼 수 있다. Handler 또한 마찬가지로 helper 처럼 handling 을 해주는 것 (er) 이라고 볼 수 있다.

보통 가장 많이 보는 것이 spring mvc Dispatcher Servlet 이 handlerMapping 을 통해 결과를 수행하고 화면에 전달할 때 쓰는 view resolver 일 것이다.

> - view resolver?
>
>spring mvc 에서 data flow 를 보면 화면을 보여주는 과정이
>
>client -> Dispatcher Servlet -> HandlerMapping -> Controller -> Dispatcher Servlet -> view resolver -> client
>
>와 같이 진행 되는데, D.S 에서 HandlerMapping 을 통해 server 내 url 에 mapping 을 해주고 이와 같은 과정을 통해
>controller 와 연결이 된다.
>
>이후 controller 내에서 비즈니스 로직을 수행하고 화면을 보여줘야 할 경우 view resolver 를 통해 client 에게 화면을 제공해준다.

## 왜 알아야 하나?

그냥 내가 알고 싶어서(...) 진행한다. 사실 개발하면서 resolver 를 순수 구축해서 사용하는 등의 경우가 왕왕 있었는데, 정확하게 resolver 나 handler, helper 등에 대해
정의가 머릿속에 없어서 정리할 겸 진행하기로 했다.

## 구성

사실 마땅찮은 구성은 없다. 구현하기에 따라 이름 붙이기에 따라 resolver, handler, helper 등을 사용하는 것 같은데, 가장 예시를 들기 좋은게 spring mvc 의 resolver 라
spring mvc 를 기준으로 작성한다.

spring boot 에서 thymeleaf template 사용 시 ThymeleafViewResolver 가 기본적으로 view resolver 역할을 한다. ~~아마도~~

```java
...
// html return 시 string return 으로 redirect: 를 지정하듯이 resolver 가 지정해준다.
public static final String REDIRECT_URL_PREFIX = "redirect:";
    
public static final String FORWARD_URL_PREFIX = "forward:";

private boolean redirectContextRelative = true;
private boolean redirectHttp10Compatible = true;

private boolean alwaysProcessRedirectAndForward = true;

private boolean producePartialOutputWhileProcessing = AbstractThymeleafView.DEFAULT_PRODUCE_PARTIAL_OUTPUT_WHILE_PROCESSING;

// viewClass 로 ThymeleafView.class 를 지정해준다. 이를 통해 Thymeleaf template 에 맞는 화면들을 render 할 수 있다.
private Class<? extends AbstractThymeleafView> viewClass = ThymeleafView.class;   
private String[] viewNames = null;
private String[] excludedViewNames = null;
private int order = Integer.MAX_VALUE;


private final Map<String, Object> staticVariables = new LinkedHashMap<String, Object>(10);
private String contentType = null;
private boolean forceContentType = false;
private String characterEncoding = null;

private ISpringTemplateEngine templateEngine;
...
```

아주 잠깐 살펴보고 바로 이해할 수 있는 부분들은 resolver 는 개발자들이 실제로 신경써서 개발해야 하는 static 한 설정들에 대해
자동으로 설정을 해주고 해결해준다. 이와 같은 부분들이 resolver 가 하는 역할이라 볼 수 있다.

Handler 또한 resolver 와 비슷한 개념인데, 가장 대표적인게 Android 에서의 UI 작업 중 worker thread 의 결과 값을 main thread 에 전달하는 handler 가 있다.

main thread 와 worker thread 가 동시에 화면에 접근하여 처리 할 경우 critical section error 가 발생 할 수 있기에 
worker thread 가 별도의 작업 후 setting 시 main thread 에 render 를 요청하게 된다.

이 때 work thread 의 결과 값을 main thread 에 전달하는 작업을 handler 가 대신 진행해준다.

## 그래서?

이와 같이 반복적인 static 한 작업에 대해 대신 설정 및 수행을 해주는 역할이 resolver, handler, helper 등으로 명시 할 수 있다.

최근 들어 MSA 가 활발히 성행(?) 하고 있기에 비슷한 개념으로 DAO / DTO 분리 후 mapping 하는 작업에 대해서 빈번하게 개발하고 있는데
상호간 mapper 를 만들어서 stream + lambda 식으로 진행하고 있다.

이러한 부분들이 resolver 라고 볼 수 있는데 spring mvc resolver 를 참조해서 최대한 이쁘게(?) 만들어야겠다.