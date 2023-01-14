# 설정

## 접속

`'localhost'` 와 `'127.0.0.1'` 은 같아 보일 수 있지만 다르다.\
localhost로 로그인 할 경우 sock파일을 사용하여 `Unix domain socket` 을 이용한다.\
127.0.0.1 을 사용할 경우, 자기 서버를 가리키는 루프백 IP이기는 하지만 `TCP/IP` 방식을 사용한다.

## 서버 설정

MySQL은 단 하나의 설정 파일을 사용하는데, 리눅스를 포함한 유닉스 계열은 `my.cnf` 윈도우 계열은 `my.ini` 파일을 사용한다.

### 설정 파일의 구성

설정 파일은 하나의 파일에 여러 개의 설정 그룹을 담을 수 있으며, 대체로 실행 프로그램 이름을 그룹명으로 사용한다. 예를 들어, mysqldump 프로그램은 \[mysqldump] 설정 그룹을 참조한다.

{% code title="my.cnf" lineNumbers="true" %}
```
[mysqld_safe]
malloc-lib = /opt/lib/libtcmalloc_minimal.so

[mysqld]
socket = /usr/local/mysql/tmp/mysql.sock
port = 3306
```
{% endcode %}

### 시스템 변수의 특징

MySQL 서버는 기동하면서 설정 파일의 내요을 읽어 메모리나 작동 방식을 초기화하고, 접속된 사용자를 제어하기 위해 이러한 값을 별도로 저장해준다. 서버에서는 이렇게 저장된 값을 `시스템 변수(System Variables)`라고 한다.\
각 시스템 변수는 `SHOW VARIABLES` 또는 `SHOW GLOBAL VARIABLES` 라는 명령어로 확인할 수 있다.

### 글로벌 변수와 세션 변수

시스템 변수는 글로벌 변수와 세션 변수로 나누어져 있다.\
일반적으로 세션별로 적용되는 시스템 변수의 경우 글로벌 변수뿐만 아니라, 세션 변수에도 동시에 존재한다.\
이러한 경우 `Var Scope`에는 `Both` 라고 표시된다.

#### 글로벌 변수

하나의 MySQL 서버 인스턴스에서 전체적으로 영향을 미치는 시스템 변수를 의미한다.\
주로 MySQL 서버 자체에 관련된 설정일 경우가 많다.

* InnoDB 버퍼 풀 크기(innodb\_buffer\_pool\_size)
* MyISAM의 키 캐시 크기(key\_buffer\_size)

#### 세션 변수

MySQL 클라이언트가 서버에 접속할 때 기본으로 부여하는 옵션의 기본값을 제어하는 데 사용된다.\
클라이언트의 필요에 따라 개별 커넥션 단위로 다른 값으로 변경할 수 있는 것이 세션 변수이다.

여기서 기본값은 글로벌 시스템 변수이며, 각 클라이언트가 가지는 값이 세션 변수이다.

각 클라이언트에서 쿼리 단위로 자동 커밋을 수행할지 여부를 결정하는 `autocommit` 변수가 대표적인 예라고 볼 수 있다.

### 정적 변수와 동적 변수

#### 정적 변수

서버가 기동 중인 상태에서 변경해도 적용되지 않는다.\
my.cnf파일을 변경하더라도 서버가 재시작 하기전에는 적용되지 않는다.\
하지만 SHOW 명령을 통해서 변수값을 확인하거나 SET 명령을 통해 바꿀 수 있다.

#### 동적 변수

서버가 기동 중인 상태에서 SET 명령어를 통해 변수값을 변경할 수 있다.

```
SET GLOBAL max_connections=500;
```

하지만 SET 명령어를 통해 변경되는 변수값은 my.cnf에  적용되는 것은 아니기 때문에 영구히 적용하려면 my.cnf파일도 반드시 변경해야 한다.\
하지만 매번 설정 후 파일 변경을 하는 것이 쉬운일은 아니다.

MySQL 8.0 버전부턴 SET PERSIST 명령을 통해 실행 중인 시스템 변수의 변경과 동시에 설정 파일로도 기록할 수 있다.

#### SET PERSIST

```
SET PERSIST max_connection=5000;
```

또한 현재 바로 적용하지 않고 다음 재시작일 때부터 적용시키려면 SET PERSIST\_ONLY 명령어를 사용하면 된다.

```
SET PERSIST_ONLY max_connection=3000;
```

해당 내용은 mysqld-auto.cnf 파일에 내용이 저장되는데 수정을 했지만 그 수정을 제거하고 싶을 경우도 있다. 해당 파일을 직접 건드려서 삭제하는 방법도 있지만, 실수를 할 수 있기 때문에 추천하지 않는다.

대신 MySQL에서 `RESET PERSIST`를 제공한다.

```
RESET PERSIST max_connections;

## mysqld-auto.cnf 파일 모두 삭제
REST PERSIST
```
