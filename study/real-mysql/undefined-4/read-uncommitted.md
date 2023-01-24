# READ UNCOMMITTED

`READ UNCOMMITTED` 격리 수준에서는 각 트랜잭션에서의 변경 내용이  `COMMIT`이나 `ROLLBACK` 여부에 상관 없이 다른 트랜잭션에서 보인다.\
다른 트랜잭션이 사용자 B가 실행하는 SELECT 쿼리의 결과에 어떤 영향을 미치는지를 보여주는 예제다.

<img src="../../../.gitbook/assets/file.excalidraw (1).svg" alt="" class="gitbook-drawing">

커밋되지 않더라도 사용자 B는 `Lara`를 조회할 수 있다.\
문제는 사용자 A가 처리 도중 문제가 발생하여 롤백되더라도 사용자 B는 `Lara`가 정상적인 사원이라고 처리 할 것이라는 점이다.

이처럼 어떤 **트랜잭션에서 처리한 작업이 완료되지 않더라도** 다른 트랜잭션에서 볼 수 있는 현상을 `더티 리드`라고 하며, 더티 리드가 허용되는 격리 수준이 `READ UNCOMMITTED`이다.

**해당 격리 수준은 사용하지 않는 것을 권장한다.**
