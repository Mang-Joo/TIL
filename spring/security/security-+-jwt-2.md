# Security + Jwt를 활용한 로그인 (2)

여기선 1에서 했던 아키텍처를 기반으로 로그인 하는 것을 구현해보려고 한다.

일단 필자의 프로젝트는 자바17 + 스프링부트 3 + Security + Jpa로 구성하였다.\
DB는 간단하게 인메모리 h2 database를 사용하였다.

### Gradle

```gradle
implementation 'org.springframework.boot:spring-boot-starter-data-jpa'
implementation 'org.springframework.boot:spring-boot-starter-security'
implementation 'org.springframework.boot:spring-boot-starter-web'
implementation 'org.springframework.boot:spring-boot-starter-oauth2-resource-server'
implementation 'javax.xml.bind:jaxb-api:2.3.0-b170201.1204'
implementation group: 'io.jsonwebtoken', name: 'jjwt', version: '0.6.0'
compileOnly 'org.projectlombok:lombok'
runtimeOnly 'com.h2database:h2'
annotationProcessor 'org.projectlombok:lombok'
testImplementation 'org.springframework.boot:spring-boot-starter-test'
testImplementation 'org.springframework.security:spring-security-test'
```

일단 가장 먼저 Filter를 만들 것이다.

Filter는 로그인 요청이 들어왔을 때 `AuthenticationToken`을 생성하여 `AuthenticationManager`에 요청을 보낸다.

### Filter

다음은 Filter의 코드이다.

```java
@Slf4j
public class LoginFilter extends AbstractAuthenticationProcessingFilter {

    private final ObjectMapper objectMapper;

    private final SuccessHandler successHandler;

    private final FailureHandler failureHandler;

    public LoginFilter(String defaultFilterProcessesUrl, ObjectMapper objectMapper, SuccessHandler successHandler, FailureHandler failureHandler) {
        super(defaultFilterProcessesUrl);
        this.objectMapper = objectMapper;
        this.successHandler = successHandler;
        this.failureHandler = failureHandler;
    }

    @Override
    public Authentication attemptAuthentication(HttpServletRequest request, HttpServletResponse response) throws AuthenticationException {
        if (!request.getContentType().equals("application/json")) {
            throw new IllegalArgumentException("Content type must be application/json");
        }
        if (!request.getMethod().equals(HttpMethod.POST.name())) {
            throw new IllegalArgumentException("must be POST Method");
        }
        
        try {
            LoginRequest loginRequest = objectMapper.readValue(request.getReader(), LoginRequest.class);
            log.info("LoginRequest: {}", loginRequest);
            return getAuthenticationManager().authenticate(new UsernamePasswordAuthenticationToken(loginRequest.username(), loginRequest.password()));
        } catch (DatabindException e) {
            throw new IllegalArgumentException("json parse error");
        } catch (IOException e) {
            throw new RuntimeException(e);
        }
    }

    @Override
    protected void successfulAuthentication(HttpServletRequest request, HttpServletResponse response, FilterChain chain, Authentication authResult) throws IOException {
        //Success Handler
        successHandler.onAuthenticationSuccess(request, response, authResult);
    }

    @Override
    protected void unsuccessfulAuthentication(HttpServletRequest request, HttpServletResponse response, AuthenticationException failed) throws IOException {
        //FailureHandler Handler
        failureHandler.onAuthenticationFailure(request, response, failed);
    }
}
```

일단 필자는 `AbstractAuthenticationProcessingFilter` 클래스를 상속하여 사용했다.

일단 생성자 쪽부터 설명하자면 Filter는 Bean으로 등록하지 않고 사용한다. 프로퍼티들에 값을 집어넣어주는 부분은 `Config`에서 처리하게 된다.\
또한 `defaultFilterProcessUrl` 이란 해당 Url만 필터를 사용하는 변수이다.(생성자에 `defaultFilterProccessesUrl`대신 `RequestMatcher`를 사용하여 처리할 수도 있다.)\
`SuccessHandler`, `FailuerHandler`는 성공 했을 때와 실패했을 때를 처리하기 위한 객체이다.

요청을 받아 처리하고 위임을 하는 메서드는 `attempAuthentication` 메서드이다.

가장 먼저 요청은 json과 HttpMethod Post 형식만 받는걸로 명시해두었다.\
보통 로그인은 form으로도 많이 하기 때문에 form을 원한다면 application/json를 수정하여 사용하면 된다.\
그 다음 objectMapper를 통하여 요청 받은 것을 `LoginRequest` 객체로 매핑한다.

#### LoginRequest

LoginRequest 클래스는 다음과 같이 간단하게 작성하였다.

```java
package com.mangjoo.io.securityexample.security.filter;

public record LoginRequest(
        String username,
        String password
) {
}
```

매핑 시킨 후 `UsernamePasswordAuthenticationToken` 객체를 생성하여 AuthenticationManager에 authenticate를 return한다.

첫번째 매개변수는 `Pricipal`(계정)를 넣어주고, 2번째 매개변수는 `Credentials`(암호화)를 넣어주게 된다.\
혹시 Role이 있다면, 3번째 매개변수에 `List<GrantedAuthority>`(권한)를 넣어주면 된다.

이렇게 하면 Manager는 Provider에게 인증을 위임할 것이고, 예외가 발생하지 않는다면 `successfulAuthentication`를 예외가 발생하면 `unsuccessfulAuthentication`를 실행시킨다.

성공하면 Provider에서 return하는 인증된 객체를 authResult로 확인할 수 있고, 실패하면 AuthenticationException을 통해 예외를 확인할 수 있다.\
그렇기 때문에 가장 중요한 것은 인증할 때 예외를 발생시킬 경우 <mark style="color:red;">**꼭!!**</mark> `AuthenticationException`을 상속하고 있는 예외를 발생시켜야 한다.

일단 Filter작성을 한 후 Manager는 구현하지 않고 Provider를 구현하도록 하겠다.

Manager를 구현하지 않는 이유는 Manager의 역할은 위임하는 용도인데 우리가 직접 구현하지 않아도 인터페이스로 되어 있기 때문에 기본을 사용하면 된다.(필요한 경우엔 구현하여 사용할 수 있다.)

### Provider

다음은 Provider의 구현이다.

```java
@Component
@RequiredArgsConstructor
public class LoginProvider implements AuthenticationProvider {

    private final CustomUserDetailsService customUserDetailsService;
    private final PasswordEncoder passwordEncoder;

    @Override
    public Authentication authenticate(Authentication authentication) throws AuthenticationException {
        String username = (String) authentication.getPrincipal();
        String password = (String) authentication.getCredentials();

        MemberEntity memberEntity = customUserDetailsService.loadUserByUsername(username);

        if (!passwordEncoder.matches(password, memberEntity.getPassword())) {
            throw new BadCredentialsException("password not match");
        }

        List<GrantedAuthority> authorities = List.of(
                new SimpleGrantedAuthority(memberEntity.getRole().name())
        );

        return new UsernamePasswordAuthenticationToken(memberEntity.getId(), null, authorities);
    }

    @Override
    public boolean supports(Class<?> authentication) {
        return UsernamePasswordAuthenticationToken.class.isAssignableFrom(authentication);
    }
}
```

여기서는 `AuthenticationProvider` 인터페이스를 구현하게 된다.

`CustomUserDetailService`는 User를 실제로 검증하기 위한 객체이고, `PasswordEncoder`는 인코딩을 하여 저장하기 때문에 인코딩 객체를 주입받게 된다.

Principal에 username, Credentials에 password를 넣어줬기 때문에 String으로 변환하여 값을 가져온다.\
Principal과 Credentials는 Object이기 때문에 Filter에서 주입한 객체로 변환하여 사용할 수도 있다.

`username`을 통해서 `Member`객체를 가져오게 되는데 해당 코드는 그냥 Service를 구현해도 되고, UserDetailsService 인터페이스를 구현해도 된다.\
단, 인터페이스를 구현할 경우 return Type(UserDetails)이 정해져있기 때문에 DB에서 가져온 데이터를 변환해야 하는 번거로움이 있다.

그 후 권한을 가져와 `GrantedAuthority`로 변환하여 `Token`을 생성하여 반환한다.

여기서 중요한 부분은 `supports` 메서드다.\
`UsernamePasswordAuthenticationToken` 객체를 반환하는 클래스만 사용하겠다고 명시를 해주는 부분이다.\
그렇기 때문에 여러 Provider를 생성하고 싶다면 Filter쪽에서 return하는 Token을 커스텀하여 사용해야 한다.

return값에 Principal은 식별자인 Id를 넣어주었고 인증은 끝났기 때문에 비밀번호가 필요없기 때문에 `null`로 넣어주었다. 마지막엔 권한을 넣어서 `return` 하였다.

#### PasswordEncoder

PasswordEncoder는 BCrypt를 사용하였다.

```java
package com.mangjoo.io.securityexample.security.config;

import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.security.crypto.bcrypt.BCryptPasswordEncoder;
import org.springframework.security.crypto.password.PasswordEncoder;

@Configuration
public class PasswordConfig {
    @Bean
    public PasswordEncoder passwordEncoder() {
        return new BCryptPasswordEncoder();
    }
}
```

#### CustomUserDetailsService

다음은 CustomUserDetailsService이다.

```java
package com.mangjoo.io.securityexample.security.service;

import com.mangjoo.io.securityexample.application.persistence.MemberEntity;
import com.mangjoo.io.securityexample.application.persistence.MemberJpaRepository;
import lombok.RequiredArgsConstructor;
import org.springframework.security.core.userdetails.UsernameNotFoundException;
import org.springframework.stereotype.Service;

@RequiredArgsConstructor
@Service
public class CustomUserDetailsService {

    private final MemberJpaRepository memberJpaRepository;

    public MemberEntity loadUserByUsername(String username) {
        return memberJpaRepository.findByUsername(username)
                .orElseThrow(() -> new UsernameNotFoundException("user not found"));
    }
}

```

#### MemberJpaRepository

```java
package com.mangjoo.io.securityexample.application.persistence;

import org.springframework.data.jpa.repository.JpaRepository;

import java.util.Optional;

public interface MemberJpaRepository extends JpaRepository<MemberEntity, Long> {

    Optional<MemberEntity> findByUsername(String username);
}
```

#### MemberEntity

```java
package com.mangjoo.io.securityexample.application.persistence;

import jakarta.persistence.*;
import lombok.*;

import java.util.Objects;

@Entity
@Getter
@AllArgsConstructor
@Table(name = "member")
@NoArgsConstructor(access = AccessLevel.PROTECTED)
@Builder
public class MemberEntity {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    private String username;
    private String password;

    @Enumerated(EnumType.STRING)
    private Role role;

    public boolean matchPassword(String password) {
        return this.password.equals(password);
    }

    @Override
    public boolean equals(Object o) {
        if (this == o) return true;
        if (o == null || getClass() != o.getClass()) return false;
        MemberEntity that = (MemberEntity) o;
        return Objects.equals(id, that.id);
    }

    @Override
    public int hashCode() {
        return Objects.hash(id);
    }
}
```

Service쪽 로직은 간단한 조회 로직이기 때문에 따로 설명하지 않고 넘어가도록 하겠다.

### SuccessHandler

성공 시 Filter에 작성한 `successfulAuthentication` 메서드를 통하여 성공에 관한 처리를 진행하면 된다.

```java
@Override
    protected void successfulAuthentication(HttpServletRequest request, HttpServletResponse response, FilterChain chain, Authentication authResult) throws IOException {
        //Success Handler
        successHandler.onAuthenticationSuccess(request, response, authResult);
    }
```

아까 말했던 `authResult`가 return된 `UsernamePasswordAuthenticationToken`을 가져오게 된다.

#### successHandler

```java
package com.mangjoo.io.securityexample.security.handler;

import com.fasterxml.jackson.databind.ObjectMapper;
import com.mangjoo.io.securityexample.application.persistence.Role;
import com.mangjoo.io.securityexample.security.jwt.JwtService;
import jakarta.servlet.FilterChain;
import jakarta.servlet.ServletException;
import jakarta.servlet.http.HttpServletRequest;
import jakarta.servlet.http.HttpServletResponse;
import jakarta.servlet.http.HttpSession;
import lombok.RequiredArgsConstructor;
import org.springframework.http.MediaType;
import org.springframework.security.core.Authentication;
import org.springframework.security.web.WebAttributes;
import org.springframework.security.web.authentication.AuthenticationSuccessHandler;
import org.springframework.stereotype.Component;

import java.io.IOException;


@Component
@RequiredArgsConstructor
public class SuccessHandler implements AuthenticationSuccessHandler {

    private final ObjectMapper objectMapper;
    private final JwtService jwtService;

    @Override
    public void onAuthenticationSuccess(HttpServletRequest request, HttpServletResponse response, FilterChain chain, Authentication authentication) throws IOException, ServletException {
        AuthenticationSuccessHandler.super.onAuthenticationSuccess(request, response, chain, authentication);
    }

    @Override
    public void onAuthenticationSuccess(HttpServletRequest request, HttpServletResponse response, Authentication authentication) throws IOException {
        Long id = (Long) authentication.getPrincipal();
        Role role = authentication.getAuthorities()
                .stream()
                .map(grantedAuthority -> Role.valueOf(grantedAuthority.getAuthority()))
                .findFirst()
                .orElseThrow(() -> new IllegalArgumentException("no role"));

        String token = jwtService.generateJwt(id, role);

        response.setStatus(HttpServletResponse.SC_OK);
        response.setContentType(MediaType.APPLICATION_JSON_VALUE);

        objectMapper.writeValue(response.getWriter(), token);

        HttpSession session = request.getSession(false);
        if (session != null) {
            session.removeAttribute(WebAttributes.AUTHENTICATION_EXCEPTION);
        }
    }
}
```

Principal에 담겨있는 id와 권한을 꺼내어 jwt를 생성한 후 response에 응답을 셋팅하여 전송해준다.\
또한 JWT를 사용하기 때문에 session이 있다면 session을 삭제하는 코드를 작성하였다.

#### JwtService

JWT를 생성하기 위한 코드이다.

```java
package com.mangjoo.io.securityexample.security.jwt;

import com.mangjoo.io.securityexample.application.persistence.Role;
import com.mangjoo.io.securityexample.security.exception.ExpiredTokenException;
import com.mangjoo.io.securityexample.security.exception.JwtAllException;
import com.mangjoo.io.securityexample.security.exception.JwtSignatureException;
import io.jsonwebtoken.*;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.stereotype.Component;

import java.sql.Timestamp;
import java.time.LocalDateTime;

@Component
public class JwtService {

    @Value("${jwt.token.secret}")
    private String secretKey;

    @Value("${jwt.token.time}")
    private long tokenValidTime;

    public String generateJwt(Long id, Role role) {

        LocalDateTime now = LocalDateTime.now();
        LocalDateTime authTime = now.plusSeconds(tokenValidTime);

        return Jwts.builder()
                .setHeaderParam(Header.TYPE, Header.JWT_TYPE)
                .setIssuedAt(Timestamp.valueOf(now))
                .setExpiration(Timestamp.valueOf(authTime))
                .setId(id.toString())
//                .claim("id", id)
                .claim("role", "ROLE_" + role)
                .signWith(SignatureAlgorithm.HS256, secretKey)
                .compact();
    }
}
```

secretKey와 time은 yml을 통해 셋팅하였다.

### Config

이제 마지막으로 Config를 작성해보겠다.

```java
package com.mangjoo.io.securityexample.security.config;

import com.fasterxml.jackson.databind.ObjectMapper;
import com.mangjoo.io.securityexample.security.filter.JwtAuthenticationFilter;
import com.mangjoo.io.securityexample.security.filter.LoginFilter;
import com.mangjoo.io.securityexample.security.handler.FailureHandler;
import com.mangjoo.io.securityexample.security.handler.SuccessHandler;
import com.mangjoo.io.securityexample.security.provider.JwtAuthenticationProvider;
import com.mangjoo.io.securityexample.security.provider.LoginProvider;
import lombok.RequiredArgsConstructor;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.security.authentication.AuthenticationManager;
import org.springframework.security.authentication.ProviderManager;
import org.springframework.security.config.annotation.web.builders.HttpSecurity;
import org.springframework.security.config.annotation.web.configuration.EnableWebSecurity;
import org.springframework.security.web.SecurityFilterChain;
import org.springframework.security.web.authentication.UsernamePasswordAuthenticationFilter;

import java.util.List;

import static org.springframework.security.config.http.SessionCreationPolicy.STATELESS;

@Configuration
@EnableWebSecurity
@RequiredArgsConstructor
public class SecurityConfig {

    private final ObjectMapper objectMapper;
    private final SuccessHandler successHandler;
    private final FailureHandler failureHandler;
    private final LoginProvider loginProvider;
    private final JwtAuthenticationProvider jwtAuthenticationProvider;

    @Bean
    public SecurityFilterChain configure(
            HttpSecurity httpSecurity,
            AuthenticationManager authenticationManager
    ) throws Exception {
        httpSecurity.csrf().disable()
                .sessionManagement()
                .sessionCreationPolicy(STATELESS)
                .and()
                .authorizeHttpRequests()
                .requestMatchers("/api/v1/login").permitAll()
                .requestMatchers("/api/v1/user/**").hasRole(USER.name())
                .anyRequest()
                .authenticated()
                .and()
                .addFilterBefore(loginFilter(authenticationManager), UsernamePasswordAuthenticationFilter.class);

        httpSecurity
                .authenticationProvider(loginProvider);

        return httpSecurity.build();
    }

    private LoginFilter loginFilter(AuthenticationManager authenticationManager) {
        LoginFilter loginFilter = new LoginFilter("/api/v1/login", objectMapper, successHandler, failureHandler);
        loginFilter.setAuthenticationManager(authenticationManager);
        return loginFilter;
    }

    @Bean
    public AuthenticationManager authenticationManager() {
        return new ProviderManager(List.of(loginProvider));
    }
}

```

상속하는 방식은 디프리케이트 되었기 때문에 Bean으로 하는 방식을 선택하였다.

session은 사용하지 않을 것이기 때문에 STATELESS로 선언하였다.

login은 인증 없이 해야하기 때문에 permitAll()을 해주었고, user로 시작하는 모든 url은 USER권한을 가진 사용자만 사용할 수 있게 설정하였다.\
다른 요청에 대한 건들은 인증이 되어있어야 성공하게 하도록 authenticated()를 사용하였다.\
addFilteBefore에 loginFilter를 넣어준다.\
LoginFilter는 setAuthenticationManager를 통하여 manager를 주입해주고, return하여 filter생성 한 Filter를 주입하였다.

AuthenticationManager는 Bean타입일 경우 기본 Manager를 선언해주어야 하기 때문에 ProviderManager에 사용할 Provider들을 리스트 형식으로 넣어준 후 생성한다.

> Provider가 여러 개 들어갈 경우 순서를 잘 정해서 넣어주어야한다. Provider를 찾기 위해 순회를 하기 때문에 가장 많이 사용 할 Provider를 가장 앞에 넣어주어야 한다.

이렇게 하면 로그인을 위한 준비는 끝났다.

나는 테스트를 위해 다음과 같은 간단한 코드를 작성하였다.

```java
package com.mangjoo.io.securityexample;

import com.mangjoo.io.securityexample.application.persistence.MemberEntity;
import com.mangjoo.io.securityexample.application.persistence.MemberJpaRepository;
import com.mangjoo.io.securityexample.application.persistence.Role;
import org.springframework.boot.CommandLineRunner;
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.context.annotation.Bean;
import org.springframework.security.crypto.password.PasswordEncoder;

@SpringBootApplication
public class SecurityExampleApplication {

    public static void main(String[] args) {
        SpringApplication.run(SecurityExampleApplication.class, args);
    }

    @Bean
    CommandLineRunner commandLineRunner(PasswordEncoder passwordEncoder, MemberJpaRepository memberRepository) {
        return args -> {
            MemberEntity memberEntity = MemberEntity.builder()
                    .username("user")
                    .password(passwordEncoder.encode("1234"))
                    .role(Role.USER)
                    .build();
            memberRepository.save(memberEntity);
        };
    }
}
```

#### 요청

```http
###
POST http://localhost:8080/api/v1/login
Content-Type: application/json

{
  "username": "user",
  "password": "1234"
}

```

#### 응답

```http
POST http://localhost:8080/api/v1/login

HTTP/1.1 200 
X-Content-Type-Options: nosniff
X-XSS-Protection: 0
Cache-Control: no-cache, no-store, max-age=0, must-revalidate
Pragma: no-cache
Expires: 0
X-Frame-Options: DENY
Content-Type: application/json;charset=ISO-8859-1
Content-Length: 169
Date: Tue, 28 Mar 2023 12:36:50 GMT
Keep-Alive: timeout=60
Connection: keep-alive

"eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJpYXQiOjE2ODAwMDcwMTAsImV4cCI6MTc2NjQwNzAxMCwianRpIjoiMSIsInJvbGUiOiJST0xFX1VTRVIifQ.FPogk0gqIKGX889S3JMGHGwNG7elP4l5zfk0UCUtg7o"

Response code: 200; Time: 100ms (100 ms); Content length: 169 bytes (169 B)

```
