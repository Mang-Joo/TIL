# 사용자

## 사용자 식별

MySQL은 사용자의 계정뿐만 아니라 접속 지점(도메인 또는 IP 주소)까지도 계정의 일부가 된다.

```
'svc_id'@'127.0.0.1'
```

해당 계정은 해당 DB에서만(LocalHost) 접속이 가능하다.

만약 모든 외부 컴퓨터에서 접속이 가능한 사용자 계정을 생성하고 싶다면 호스트 부분을 `%`문자로 대체하면 된다. 단, 동일한 계정이 있을 때 주의해야한다.

```
'svc_id'@'192.168.0.10' (비밀번호는 1234)
'svc_id'@'%' (비밀번호는 abc)
```

MySQL은 가장 작은 것을 먼저 선택한다.\
즉, `'svc_id'@'192.168.0.10'` 을 선택하게 된다.

## 사용자 계정 관리

### 시스템 계정과 일반 계정

시스템 계정은 MySQL 서버 내부적으로 실행되는 백그라운드 스레드와는 무관하며, 시스템 계정도 일반 계정과 같이 사용자를 위한 계정이다. 시스템 계정은 데이터베이스 서버 관리자(DBA)를 위한 계정이며, 일반 계정은 응용 프로그램이나 개발자를 위한 계정 정도로 생각하면 된다.

시스템 계정은 시스템 계정과 일반 계정을 관리(생성 삭제 및 변경) 할 수 있지만, 일반 계정은 시스템 계정을 관리할 수 없다.\
시스템 계정으로 하는 일은 다음과 같다.

* 계정 관리(계정 생성 및 삭제 및 권한 부여, 제거)
* 다른 세션 또는 그 세션에서 실행 중인 쿼리를 강제 종료
* 스토어드 프로그램 생성 시 DEFINER를 타 사용자로 설정

### 계정 생성

5.7 버전까지는 GRANT 명령으로 권한 부여와 생성을 동시에 했지만 8.0버전 부터는 생성(CREATE USER)과 권한(GRANT)을 나누었다.\
계정을 생성할 때는 다음과 같은 다양한 옵션을 설정할 수 있다.

* 계정의 인증 방식과 비밀번호
* 비밀번호 관련 옵션(비밀번호 유효 기간, 비밀번호 이력 개수, 비밀번호 재사용 불가 기간)
* 기본 역할(Role)
* SSL 옵션
* 계정 잠금 여부

일반적으로 많이 사용되는 옵션을 가진 CREATE USER 명령은 다음과 같다.

```sql
CREATE USER 'user'@'%'
        INDETIFIED WITH 'mysql_native_password' BY 'password'
        REQUIRE NONE
        PASSWORD EXPIRE INTERVAL 30 DAY
        ACCOUNT UNLOCK
        PASSWORD HISTORY DEFAULT
        PASSWORD REUSE INTERVAl DEFAULT
        PASSWORD REQUIRE CURRENT DEFAULT;
```

#### INDETIFIED WITH

사용자 인증 방식과 비밀번호를 설정한다. `IDENTIFIED WITH` 뒤에는 반드시 인증 방식(인증 플러그인의 이름)을 명시해야 한다.\
대표적으로 4가지 방식이 있다.

* Native Pluggable Authentication: 단순히 비밀번호에 대한 해시(SHA-1 알고리즘) 값을 저장해두고, 클라이언트가 보낸 값과 해시값이 일치하는지 비교하는 인증 방식
* Caching SHA-2 Pluggable Authentication: 8.0 버전에서는 조금 더 보안된 인증 방식으로, 암호화 해시값 생성을 위해 SHA-2(256비트) 알고리즘을 사용한다.\
  내부적으로는 `Salt` 키를 활용하며, 수천 번의 해시 계산을 수행해서 결과를 만들어 내기 때문에 동일한 키 값에 대해서도 결과가 달라진다.\
  또한 이 방식을 사용하려면 SSL/TLS 또는 RSA 키페어를 반드시 사용해야 하는데, 이를 위해 클라이언트에서 접속할 때 SSL 옵션을 활성화해야 한다.
* PAM Pluggable Authentication: 유닉스나 리눅스 패스워드 또는 LDAP(Lightweight Directory Access Protocol) 같은 외부 인증을 사용할 수 있게 해주는 인증 방식
* LDAP Pluggable Authentication: LDAP을 이용한 외부 인증을 사용할 수 있게 해주는 인증 방식

MySQL 8.0은 기본적으로 `Caching SHA-2 Pluggable Authentication` 을 사용한다.\
이 설정을 바꾸고 싶으면 `my.cnf` 설정 파일에 변경하면 된다.

```
SET GLOBAL default_authentication_plugin="mysql_native_password"
```

#### REQUIRE

SSL/TLS 채널은 사용할지 여부를 설정한다.\
만약 별도로 사용하지 않으면 비 암호화 채널이지만, `Caching SHA-2 Authentication` 인증 방식을 사용하면 암호화된 채널만으로 MySQL 서버에 접속할 수 있게 된다.

#### PASSWORD EXPIRE

비밀번호 유효 기간을 설정하는 옵션이다.\
별도로 명시하지 않으면 `default_password_lifetime` 시스템 변수에 저장된 기간으로 유효 기간이 설정된다.\
설정 가능한 옵션은 다음과 같다.

* PASSWORD EXPIRE: 계정 생성과 동시에 비밀번호의 만료 처리
* PASSWORD EXPIRE NEVER: 계정 비밀번호의 만료 기간 없음
* PASSWORD EXPIRE DEFAULT:default\_password\_lifetime 시스템 변수에 저장된 기간으로 비밀번호의 유효기간을 설정
* PASSWORD EXPIRE INTERVAL n DAY: 비밀번호의 유효 기간을 오늘부터 n일자로 설정

#### PASSWORD HISTORY

한 번 사용했던 비밀번호를 재사용하지 못하게 설정하는 옵션이다.

* PASSWORD HISTORY DEFAULT: password\_history 시스템 변수에 저장된 개수만큼 비밀번호 이력을 저장하여 이력에 남아있는 비밀번호는 재사용 할 수 없다.
* PASSWORD HISTORY n: 비밀번호 이력을 최근 n개 까지만 저장하여 이력에 남아있는 비밀번호는 재사용 할 수 없다.

MySQL 서버는 DB의 `password_history` 테이블을 사용하여 비밀번호 이력을 저장한다.

#### PASSWORD REUSE INTERVAL

한 번 사용했던 비밀번호의 재사용 금지 기간을 설정하는 옵션이다.

* PASSWORD REUSE INTERVAL DEFAULT: `password_reuse_interval` 변수에 저장된 기간으로 설정
* PASSWORD REUSE INTERVAL n DAY: n일자 이후에 비밀번호를 재사용 할 수 있게 설정

#### PASSWORD REQUIRE

기간이 만료되어 새로운 비밀번호를 설정할 때 현재 비밀번호를 필요로 할 지 말지를 결정하는 옵션

* PASSORD REQUIRE CURRENT: 비밀번호를 변경할 때 현재 비밀번호를 먼저 입력하도록 설정
* PASSWORD REQUIRE OPTIONAL: 현재 비밀번호를 입력하지 않아도 되도록 설정
* PASSWORD REQUIRE DEFAULT: `password_require_current` 시스템 변수의 값으로 설정

#### ACCOUNT LOCK / UNLOCK

계정 생성 시 또는 ALTER USER 명령어를 통해 계정 정보를 변경할 때 잠금 여부를 결정

* ACCOUNT LOCK: 사용하지 못하게 잠금
* ACCOUNT UNLOCK: 잠긴 계정을 다시 사용 가능 상태로  잠금 해제

## 비밀번호 관리

### 고수준 비밀번호

비밀번호는 유효기간이나 이력 관리를 통한 재사용 금지 기능 뿐만 아니라, 비밀번호를 쉽게 유추할 수 있는 단어들이 사용되지 않게 글자의 조합을 강제하거나 금칙어를 설정하는 기능도 있다.\
유효성 체크 규칙을 적용하려면 `validate_password` 컴포넌트를 이용하면 된다.

비밀번호 정책은 크게 세 가지를 선택 할 수 있다.

* LOW: 비밀번호 길이만 검증
* MEDIUM: 비밀번호의 길이를 검증하며, 숫자와 대소문자, 그리고 특수문자의 배합을 검증
* STRONG: MEDIUM 레벨의 검증을 모두 수행하며, 금칙어가 포함되었는지 여부까지 검증

비밀번호의 길이는 `validate_password.length` 시스템 변수에 설정된 길이 이상의 비밀번호가 사용됐는지를 검증한다.

숫자와 대소문자, 특수문자는 `validate_password.mixed_case_count` 와  `validate_password.number_count`, `validate_password.special_char_count` 시스템 변수에 설정된 글자 수 이상을 포함하고 있는지 검증한다.

금칙어는 `validate_password.dictionary_file` 시스템 변수에 설정된 단어를 포함하고 있는지 검증한다.

`validate_password.dictionary_file` 시스템 변수에 금칙어들이 저장된 사전 파일을 등록하면 된다.

### 이중 비밀번호

데이터베이스 계정의 비밀번호는 서비스가 실행 중인 상태에서 변경이 불가능했다.\
그래서 서비스에서 데이터 베이스 계정의 비밀번호는 몇 년 동안 그대로 사용되는 경우도 존재한다.

이 문제를 해결하기 위해 MySQL 8.0버전 부터는 비밀번호로 2개의 값을 동시에 사용할 수 있는 기능을 추가했다.

2개의 비밀번호는 프라이머리(Primary), 세컨더리(Secondary)로 구분된다. 최근에 설정한 비밀번호가 프라이머리이며, 이전 비밀번호는 세컨더리가 된다. \
이중 비밀번호를 사용하려면 변경 구문에 `RETAIN CURRENT PASSWORD` 옵션만 추가하면 된다.

{% code lineNumbers="true" %}
```sql
ALTER USER 'root'@'localhost' IDENTIFIED BY 'old_password'

ALTER USER 'root'@'localhost' IDENTIFIED BY 'new_password'
                                RETAIN CURRENT PASSWORD;
```
{% endcode %}

이렇게 설정할 경우 `'old_password'` 가 세컨더리 비밀번호가 되고 `'new_password'` 가 프라이머리 비밀번호가 된다.

계정은 두 비밀번호 중 아무거나  입력해도 로그인이 된다.  그 후 배포 및 재시작을 순차적으로 실행한다.

재시작이 완료되면 다음 명령으로 세컨더리 비밀번호를 삭제한다.\
꼭 삭제해야 하는 것은 아니지만, 하는 것을 권장한다.

```sql
ALTER USER 'root'@'localhost' DISCARD OLD PASSWORD;
```

이렇게 하면 기존 비밀번호는 로그인이 불가능 해지며 새로운 비밀번호만 로그인 할 수 있다.
