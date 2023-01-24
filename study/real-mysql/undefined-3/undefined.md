# 트랜잭션

## MySQL에서의 트랜잭션

> InnoDB만 작성되었습니다.
>
> MyISAM이나 Memory 스토리지 엔진은 트랜잭션을 지원하지 않습니다.

트랜잭션은 꼭 여러 개의 변경 작업을 수행하는 쿼리가 조합됐을 때만 의미 있는 개념은 아니다.\
트랜잭션은 하나의 논리적인 작업 셋에 하나의 쿼리가 있든 두 개 이상의 쿼리가 있든 관계없이 논리적인 작업 셋 자체가 100% 적용되거나 아무것도 적용되지 않아야 함을 보장해 주는 것이다.

다음 예제를 한번 보자.

```sql
CREATE TABLE tab_innodb ( fdpk INT NOT NULL, PRIMARY KEY (fpdk) ) ENGINE=INNODB;
INSERT INTO tab_innodb (fdpk) VALUES (3);

INSERT INTO tab_innodb (fdpk) VALUES (1), (2), (3);
```

해당 쿼리를 실행시키면 PK 중복 오류로 쿼리가 실패했다.\
트랜잭션이 이루어지지 않는다면 1, 2의 값은 성공적으로 INSERT되고 3값만 에러를 뱉게 된다.\
하지만 트랜잭션이 활성화 되어 있다면 하나의 작업에서 실패한 것이 있기 때문에 모든 것이 롤백 되어 1, 2의 값도 작성되지 않는다.
