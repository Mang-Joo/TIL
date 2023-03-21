# Security + Jwt를 활용한 로그인 (1)

이번에 회사에서 시큐리티쪽 인증을 구현하는 업무를 맡았는데 시큐리티쪽 공부가 하나도 되어있지 않아 친구와 함께 시큐리티를 공부하게 되었다.

친구와 함께 공부하며 가장 중요하다고 생각이 든 부분은 일단 시큐리티 아키텍처를 이해하는게 가장 중요하다고 생각했다. 시큐리티 자체가 무엇인지조차 이해하지 못하다가 아키텍처를 이해하고 나니 그렇게 어렵게 느껴지지도 않았고, 정말 잘 만든 프레임워크라는 생각이 들었다.

나처럼 처음 시큐리티를 접한 사람들이 쉽게 이해하고 간단한 것 정도는 수정할 수 있었으면 하여 글을 작성하게 되었다.

### Spring Security란?

Spring Security는 자바 기반의 웹 응용 프로그램 보안 프레임워크이다. Spring Security는 인증, 권한 부여 및 보호 기능을 제공하여 웹 응용 프로그램의 보안을 관리하게 편하게 해준다.

Spring Security는 기본적으로 스프링 프레임워크와 통합되어 있으며, 스프링 기반의 웹 응용 프로그램에서 쉽게 사용할 수 있다.

### 아키텍처

<img src="../../.gitbook/assets/file.excalidraw.svg" alt="로그인 아키텍처" class="gitbook-drawing">



차근차근 설명해보면, 가장 처음에 사용자가 로그인 요청을 보낸다. 로그인 요청이 들어오면, `Filter`를 통해서 `Authentication`을 생성한다. `Filter`는 HTTP 요청과 응답을 처리하는 과정 중에 위치하여, 요청이 들어오면 먼저 `Filter`를 거치게 됩니다. 이때, `Filter`에서 `Authentication` 객체를 생성하여 `return`한다.

처음 해당 그림을 보았을 때 2번으로 갔다가 3번으로 가는 화살표가 이상했는데, 해당 그림은 코드를 보면 이해가 됩니다. 결국은 `Filter`에서 `Authentication` 객체를 생성하여 `AuthenticationManager`에게 전달한다. `AuthenticationManager`는 실질적인 인증은 `AuthenticationProvider`에게 위임한다.

`AuthenticationProvider`에서 요청 받은 `id`와 `password`를 담은 `Authentication`을 인자로 받아, id와 pw를 DB를 통해 검증하게 된다. 이 과정에서, `AuthenticationProvider`에서 사용하는 `UserDetailsService`를 `implements`하여 구현하거나 그냥 클래스를 생성하여 작성해도 된다. 단, UserDetailsService 인터페이스를 구현하게 되면 `UserDetails`를 무조건 리턴해야 하기 때문에 해당 부분은 잘 고민해서 작성하면 된다. 그림으로는 `UserDetails`를 리턴하게 적어뒀지만 꼭 그럴 필요는 없다는 뜻이다. (작성자는 UserDetailsService와 UserDetails를 사용하지 않았다.)

인증이 성공하면, Spring Security는 요청을 계속 진행하며, 인증에 대한 정보를 저장한다. 그 이후로는 스프링 아키텍처와 같이 `return`한 결과값을 가지고 에러없이 성공 시 `SuccessHandler`를 통하여 `JWT`를 발급하면 되고, 예외가 발생할 경우 `FailureHandler`에서 예외에 대한 처리를 해주면 된다.

아직 정확한 감이 안잡힐 수 있지만, 다음 로그인 처리에 대한 내용을 보게 되면 어느정도 감이 잡히게 될 것이다.

여기서 이해하지 못했다고 하루종일 이론을 보기보단 코드를 보고 한번 더 아키텍처를 보면 이해가 더 잘 될 것이다.
