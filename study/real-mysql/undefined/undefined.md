# 설치

### 컨테이너 생성

`docker-compose up -d`

{% code title="docker-compose.yml" lineNumbers="true" %}
```yaml
version: '3'
services:
  mangjoo-db:
    image: mysql:8.0
    container_name: mangjoo-container
    restart: always
    ports:
      - 3338:3306
    environment:
      MYSQL_ROOT_PASSWORD: 1234
      TZ: Asia/Seoul
    command:
      - --character-set-server=utf8mb4
      - --collation-server=utf8mb4_unicode_ci
    volumes:
      - ./init:/docker-entrypoint-initdb.d
    platform: linux/x86_64
```
{% endcode %}

{% code title="./init/default.sql" lineNumbers="true" %}
```yaml

# 기본 설정사항 작성
```
{% endcode %}

#### 설정 정보

```
host : IP:3338
USER : ROOT
PASSWORD : 1234
```

#### 설정 확인

```
mysql -uroot -p1234

> SELECT CURRENT_USER();
+----------------+
| CURRENT_USER() |
+----------------+
| root@localhost |
+----------------+
1 row in set (0.00 sec)
```

## 업그레이드

### 인플레이스 업그레이드(In-Pace Upgrade)

* MySQL 서버의 데이터 파일을 그대로 두고 업그레이드 하는 방법
* 여러가지 제약사항이 있지만, 업그레이드 시간을 크게 단축할 수 있다.

### 논리적 업그레이드(Logical Upgrade)

* `mysqldump` 도구 등을 이용해 MySQL 서버의 데이터를 SQL 문장이나 텍스트 파일로 덤프 한 후, 새로 업그레이드 된 버전에 덤프된 데이터를 적재하는 방법
* 제약 사항이 거의 없지만, 업그레이드 시간이 매우 많이 소요될 수 있다.

#### 인플레이스 업그레이드 제약 사항

1. 데이터 파일의 변경이 필요하기 때문에 반드시 직전 버전에서만 업그레이드가 허용 된다.\
   ex) MySQL 5.5버전에서 5.6버전으로는 가능하지만 5.7이나 그 이상으로는 업그레이드 불가능하다.\
   \
   5.1버전에서 8.0으로 업그레이드 해야 된다면 인플레이스 업그레이드보단 논리적 업그레이드를 고려하는게 더 좋은 방법일 수 있다.
2. 메이저 버전 업그레이드가 특정 마이너 버전에서만 가능한 경우도 있다.\
   ex) MySQL 5.7.8버전에서 8.0 버전으로 바로 업그레이드 할 수 없다.\
   이 경우는 5.7.8 버전이 `GA(General Availability)` 버전이 아니기 때문이다.\
   그렇기 때문에 최소 GA버전을 지나 15\~20번 이상의 마이너 버전을 선택하는 것이 좋다.
