---
title: "Reflection"
date: 2020-10-21T22:25:06+09:00
draft: true
tags: ["programming", "java", "spring"]
---

## Reflection?

영어로 해석하자면 `반사` 라는 뜻을 가진다.

그 뜻에 걸맞게 java 에선 jvm 의 runtime 시 동작하는 object 의 class, method name, annotation 등을
알 수 있다.

runtime 시 접근이 가능해서 반사라는 의미를 부여해서 reflection 이라 명명했다고 한다. ~~카더라~~

## 왜 알아야 하나?

사실 사용하지 않는 것은 아니다. 기본적으로 spring framework 내에서 사용 시 reflection 을 통해 annotation 을 적용한다는지
spring bean factory 를 통해 application 최초 구동 시 bean 객체를 만들때도 사용한다.

이러한 특징을 살려 기본적인 java code 외 상황에 따라 reflection 을 이용해 custom annotation 이나 기타 object 를 만들어
손쉽게 로직을 풀어 나갈 수 있다.

## 구성

- getType() : 필드 타입 클래스

- getAnnotations() : 필드에 달린 어노테이션의 클래스들

- getDeclaringClass() : 해당 필드가 어떤 클래스에서 선언되었는지 (부모클래스일수도 있음)

- setAccessible(Boolean b) : 프라이빗 필드는 보통 접근이 불가. true 로 해줘야 가능

- get(Object obj) : 특정 object 에서 해당 필드의 value 를 가져오기

이와 같은 class 가 있다고 하자.

```java

@Getter
@Builder
@NoArgsConstructor
@AllArgsConstructor
@Entity
@Table(name = "TEST_OBJ")
public class TestObject {
    @Id
    @GeneratedValue(strategy = GenerationType.AUTO)
    private Long id;

    private String name;

    // lob 절 대 쓰 지 마
    // 뭔가 방법이 있을텐데, ResultSet extract 를 실패한다.
    @Lob
    @Column(columnDefinition = "text")
//    @Type(type = "text")
    private String content;

    @Temporal(TemporalType.TIMESTAMP)
    private Date createdDate;

    @Temporal(TemporalType.TIMESTAMP)
    private Date updatedDate;

}

```

```java
@SpringBootTest
public class ReflectionTest {

  @Test
  public void reflectionTest() {
    TestObject testObject = TestObject.builder()
            .id(1L)
            .name("name")
            .content("content")
            .createdDate(new Date())
            .updatedDate(null)
            .build();

    // 첫 field 는 Long type 의 id 가 지정된다.
    for (Field field : testObject.getClass().getDeclaredFields()) {
        field.getType(); // 1. img
        field.getAnnotatedType();
        field.getDeclaredAnnotations(); // 2. img
    }
  }
}
```

![field.getType()](https://jungqui.github.io/images/reflection/field_getType.png)
1번 이미지

![field.getDeclaredAnnotations()](https://jungqui.github.io/images/reflection/filed_getAnntation.png)
2번 이미지

이와 같이 만약, class 내의 정보를 모른다고 하더라도 이와 같이 class 내의 정보를 runtime 시 가져와서 사용이 가능하다.

사실 이렇게만 쓴다면 이걸 왜 써야 하는지 잘 모를 수 있다. 하지만 대표적인 reflection 사용의 예는 custom annotation 이다.

------
특정 변수를 선언 할 때마다 별도의 `이름` 을 지정하고 그 이름을 출력하는 로직을 사용한다고 가정을 하자.

만약 어노테이션과 리플렉션을 사용하지 않는다면 Object 내에 별도의 param 을 만들어 항상 생성 시 마다 지정을 해야 할 것이다.

이름 하나를 출력하기 위해 class 내에 param 을 생성하는 것은 너무 비효율적인 방법이다. 하지만 reflection 과 annotation 을 이용한다면 아래와 같이 구성이 가능하다.

```java
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
public @interface JKAnnotation {
  String name() default "JKLee";
}
```

- target : 해당 annotation 을 어떤 


```java
@Data
@NoArgsConstructor
@AllArgsConstructor
@JKAnnotation
public class AnnotationObject {
  private String name;
  private String content;
}

// -------------다른 class 라 가정한다----------------- //

@Data
@NoArgsConstructor
@AllArgsConstructor
@JKAnnotation(name = "not default")
public class AnnotationObjectNotDefault {
  private String name;
  private String content;
}
```

```java
@SpringBootTest
public class AnnotationTest {

  @Test
  public void annotationTest() {
    AnnotationObject annotationObject = new AnnotationObject(
        "name", "content"
    );
  
    AnnotationObjectNotDefault annotationObjectNotDefault = new AnnotationObjectNotDefault(
        "name not default",
		"content not default"
	);

    // 실제 로직이 이와 같다 가정하자.
    List<Annotation> annotations = new ArrayList<>(Arrays.asList(annotationObject.getClass().getAnnotations()));
    annotations.addAll(Arrays.asList(annotationObjectNotDefault.getClass().getAnnotations()));

    for(Annotation annotation : annotations){
      if(annotation instanceof JKAnnotation) {
        // annotation 을 이용해서 지정한 name을 출력한다.
        JKAnnotation myAnnotation = (JKAnnotation) annotation;
        System.out.println("name: " + myAnnotation.name());
      }
    }
  }
}
```

이외에도 여러가지가 있는데 본인이 진행하면서 익숙해질겸 파악하길 바란다.

~~사실 필자부터 익숙하지가 않다;~~

## 그래서?

앞으로 필자도 꾸준히 공부를 할 예정이다. 단순히 개념만 파악하는것 보다 실제 사용해가면서 필요한 부분에
요소요소 형식으로 프로젝트를 꾸며나가다 보면 정말 아름답고 간단하게 구현을 할 수 있을 것 같다.

코드는 [여기](https://github.com/JUNGQUI/spring) 에 있다.