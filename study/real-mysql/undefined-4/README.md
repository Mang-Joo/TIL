# 격리

격리 수준이란 여러 트랜잭션이 동시에 처리될 때 특정 트랜잭션이 다른 트랜잭션에서 변경하거나 조회하는 데이터를 볼 수 있게 허용할지 말지를 결정하는 것이다.

격리 수준은 크게 4가지가 있다.

* READ UNCOMMITTED
* READ COMMITTED
* REPEATABLE READ
* SERIALIZABLE

DIRTY READ라고도 불리는 READ UNCOMITTED는 대부분 사용하지 않는다.

일반적으로 많이 사용되는 격리 수준은 READ COMMITTED나 REPETABLE READ 중 하나를 사용한다.
