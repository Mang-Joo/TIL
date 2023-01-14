---
description: Open Close Principle
---

# OCP(개방-폐쇄 원칙)

### 개방-폐쇄 원칙

개방-폐쇄 원칙이란 **소프트웨어 개체는 확장에는 열려있어야 하고, 변경에는 닫혀있어야 한다.** \
****즉, 개체의 행위를 확장할 때 개체를 변경해서는 안된다는 의미이다.

예를 들어 하나의 서비스에서 Excel을 변환하여 List로 반환하는 로직을 진행한다고 해보자. 이 때 Excel -> CSV 파일로 변경되었다. 이 것을 클래스로 구현 하였을 경우 if문으로 분기처리를 하게 되는 로직이 생기며 코드가 변경되었다.

이러한 경우를 해결하기 위해서 interface를 활용하여 구현체를 따로 만든 후 interface를 사용하면 Excel일때든 CSV일때든 상관없이 구현할 수 있다.

Interactor(Service)는 OCP를 가장 잘 준수할 수 있는 곳에 위치한다. DB, Controller, View 에서 발생한 어떤 변경도 Intreactor(Service)에 영향을 주지 않는다.

Interactor는 애플리케이션에서 가장 높은 수준의 정책을 가지고 있다. Interactor 이외의 컴포넌트는 모두 주변적인 문제를 처리한다.

**가장 중요한 문제는 Interactor가 담당한다.**

### 방향성 제어

`Repository`의 인터페이스는 추상 메서드를 가지며, 그걸 구현하는 구현체는 `JpaRepository`가 될 수 있고, Mybatis가 될 수도 있다. 만약 인터페이스를 두지 않는다면 `Service` -> `JpaRepository`로 바로 향하게 된다.\
이러한 것은 코드를 확장하게 된다면, 그걸 의존하고 있는 `Service Class`도 변경되어야 한다.\
이러한 문제를 막기 위해 `interface`를 사용한다.

### 정보 은닉

Service의 인터페이스는 방향성 제어와는 다른 목적을 가진다.

<img src="broken-reference" alt="1.1" class="gitbook-drawing">

이런식으로 구현을 하게 되면 `Controller`는 `interactore`를 `interactor`는 `respository`에 의존성을 가지며 결국 `Controller`가 `repository`에 대한 의존성을 가지게 된다. 이러한 것을 '추이 종속성' 이라고 한다.

이러한 현상을 없애기 위해 다음과 같은 그림을 그리게 된다.

<img src="broken-reference" alt="1.2" class="gitbook-drawing">

추이 종속성을 가지게 되면, '**자신이 직접 사용하지 않는 요소에는 절대로 의존해선 안 된다'** 라는 원칙을 위반하게 된다.

이러한 그림은 `Controller`에서는 `Requester` 인터페이스를 바라보게 되고, `interactor`가 어떤식으로 변경되어도 `Controller`는 안전하게 된다.\
또한 반대로 `Controller`가 수정되어도 `interactor`는 영향을 받지 않게 된다.

이를 위해서 `interactor`의 내부를 은닉한다.

### 결론

<mark style="color:yellow;">**OCP는 아키텍처를 떠받치는 원동력 중 하나이다.**</mark>

하지만 그렇다고 미리 추상화를 하여 구현하는것은 오버엔지니어링이다.\ <mark style="color:yellow;">****</mark>처음 구현할 때 데이터 요청 -> 데이터 가공 -> 데이터 응답 처리만 깔끔하게 정리해두면 정리가 나중에 추상화 할 때 좀 더 수월하게 할 수 있을 것이다.
