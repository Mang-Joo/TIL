# 권한과 역할

## 권한

5.7 버전 까지는 글로벌(Global)권한과 객체 단위의 권한으로 구분됐다.\
데이터베이스나 테이블 이외의 객체에 적용되는 권한을 글로벌 권한이라고 하며, 데이터베이스나 테이블을 제어하는 데 필요한 권한을 객체 권한이라고 한다.\
객체 권한은 `GRANT` 명령으로 권한을 부여할 때 반드시 특정 객체를 명시해야 하며, 반대로 글로벌 권한은 `GRANT` 명령에서 특정 객체를 명시하지 말아야 한다.

### 사용자 권한 부여

사용자에게 권한을 부여할 때는 `GRANT` 명령어를 사용한다.

```sql
GRANT privilege_list ON db.table TO 'user'@'host';
```

#### 글로벌 권한

`GRANT OPTION` 권한은 다른 권한과 달리 `GRANT` 명령의 마지막에 `WITH GRANT OPTION` 을 명시해서 부여한다. `TO` 키워드 뒤에는 권한을 부여할 대상 사용자를 명시하고, `ON` 키워드 뒤에는 어떤 DB의 어떤 오브젝트에 권한을 부여할 지 결정할 수 있는데, 권한의 범위에 따라 사용하는 방법이 달라진다.

{% code title="글로벌 권한" lineNumbers="true" %}
```sql
GRANT SUPER ON *.* TO 'user'@'localhost';
```
{% endcode %}

글로벌 권한은 특정 DB나 테이블에 적용할 수 없기 때문에 부여할 때는 항상 \*.\*를 사용하게 된다.\
\*.\*은 모든 DB 의 모든 오브젝트를 포함해서 MySQL 서버 전체를 의미한다.\
CREATE USER나 CREATE ROLE과 같은 글로벌 권한은 DB단위나 오브젝트 단위로 부여할 수 있는 권한이 아니므로 항상 `*.*`로만 대상을 사용할 수 있다.

#### DB 권한

{% code title="DB 권한" lineNumbers="true" %}
```sql
GRANT EVENT ON *.* TO 'user'@'localhost';
GRANT EVENT ON employees.* TO 'user'@'localhost';
```
{% endcode %}

DB 권한은 특정 DB에 대해서만 권한을 부여하거나 서버에 존재하는 모든 DB에 대해 권한을 부여할 수 있기 때문에 위의 예제와 같이 ON 절에 `*.*` 이나 `employees.*` 모두 사용할 수 있다.

하지만 DB 권한만 부여하는 경우에는 `employees.department`와 같이 테이블까지 명시할 수 없다. DB 권한은 서버의 모든 DB에 적용할 수 있으므로 대상에 `*.*`를 사용할 수 있다.\
또한 특정 DB에 대해서만 권한을 부여하는 것도 가능하기에 `db.*`도 가능하다.

#### 테이블 권한

{% code title="테이블 권한" lineNumbers="true" %}
```sql
GRANT SELECT, INSERT, UPDATE, DELETE ON *.* TO 'user'@'localhost';
GRANT SELECT, INSERT, UPDATE, DELETE ON employees.* TO 'user'@'localhost';
GRANT SELECT, INSERT, UPDATE, DELETE ON employees.department TO 'user'@'localhost';
```
{% endcode %}

테이블 권한은 전체 권한을 주는 것도 가능하지만 예제 3번과 같이 특정 DB의 특정 TABLE에 권한을 주는 것도 가능하다.\
테이블의 특정 컬럼에 대해서만 권한을 부여하는 경우에는 GRANT 문법이 조금 달라진다.\
컬럼에 부여할 수 있는 권한은 DELETE를 제외한 나머지 SELECT, INSERT, UPDATE가 가능하며, 각 권한 뒤에 컬럼을 명시하는 형태로 부여한다.

{% code title="특정 컬럼 권한 부여" lineNumbers="true" %}
```sql
GRANT SELECT, INSERT, UPDATE(dept_name) ON employees.department TO 'user'@'localhost';
```
{% endcode %}

다음과 같이 권한을 부여하게 되면 `dept_name` 만 update가 가능하며 나머지에 대해선 SELECT, INSERT만 가능하다.\
하지만 이렇게 사용하는 것은 권장되지 않는다.\
하나라도 이렇게 설정된다면 테이블의 모든 컬럼에 대해서도 권한체크를 하기 때문에 전체적인 성능에 영향을 미칠 수 있다. 컬럼 단위의 접근 권한이 꼭 필요하다면 GRANT 명령으로 해결하기 보다는 테이블에서 권한을 허용하고자 하는 컬럼만으로 별도의 VIEW를 만들어 사용하는 방법도 생각해볼 수 있다.

## 역할

8.0 버전 부터는 권한을 묶어서 역할(ROLE)을 사용할 수 있다.\
실제 내부적으로는 역할과 계정은 똑같은 모습을 하고 있다.

우선 CREATE ROLE 명령어를 실행하여 역할을 생성하자.

{% code title="역할 생성" lineNumbers="true" %}
```sql
CREATE ROLE
    role_emp_read,
    role_emp_write;
```
{% endcode %}

기본적으로 역할을 그 자체로 사용될 수 없고 계정에 부여해야 하므로 유저를 생성한 후 적용해보자.

{% code title="유저 생성" lineNumbers="true" %}
```sql
CREATE USER reader@'127.0.0.1' IDENTIFIED BY 'qwerty';
CREATE USER writer@'127.0.0.1' IDENTIFIED BY 'qwerty';
```
{% endcode %}

이 계정에 역할을 부여하자.

<pre class="language-sql" data-title="역할 적용" data-line-numbers><code class="lang-sql">GRANT role_emp_read TO reader@'127.0.0.1';
<strong>GRANT role_emp_read, role_emp_write TO writer@'127.0.0.1';
</strong></code></pre>

실제로 역할은 부여되었지만, 계정의 활성화 된 역할을 조회해 보면 role\_emp\_read 역할이 없음이 나타날 것이다.

역할을 사용할 수 있게 하려면 `SET ROLE` 명령어를 통해 실행하여 해당 역할을 활성화해야한다.

{% code title="역할 활성화" %}
```sql
SET ROLE 'role_emp_read';
```
{% endcode %}

이렇게 매 역할을 생성할 때 마다 역할을 설정해주기엔 귀찮은 일이다.\
MySQL은 기본 설정을 비활성화로 해두었기 때문이다. `activate_all_roles_on_login` 시스템 변수로 설정 할 수 있다.

{% code title="역할 생성 시 자동 활성화" %}
```sql
SET GLOBAL activate_all_roles_on_login=ON;
```
{% endcode %}

### 역할과 사용자 계정

기본적으로 역할과 사용자 계정은 같은걸로 나타난다.

역할 호스트 부분에 아무것도 입력하지 않으면 다음과 같이 `'%" 가 자동으로 입력된다.`

```sql
CREATE ROLE
    role_emp_read@'%',
    role_emp_write@'%';
```

`호스트 부분을 작성하게 될 경우를 한번 보자`

```sql
CREATE ROLE role_emp_local_read@localhost;
CREATE USER reader@'127.0.0.1' IDENTIFIED BY 'qwerty';
GRANT SELECT ON employees.* TO role_emp_local_read@'localhost';
GRANT rome_emp_local_read@'localhost' TO reader@'127.0.0.1';
```

역할을 다른 계정에 부여하지 않고, 직접 로그인하는 용도로 사용한다면 그때는 역할의 호스트 부분이 중요해진다.\
즉, 역할을 사용자 계정처럼 사용했을 때 중요해진다.

역할은 기본적으로 생성하면 `account_locked` 값이 `'Y'` 로 설정되어 있다.&#x20;
