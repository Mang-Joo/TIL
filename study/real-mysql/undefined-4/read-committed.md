# READ COMMITTED

온라인 서비스에서 가장 많이 사용 되는 격리 수준이다.

이 레벨에서는 `더티 리드` 같은 현상은 발생하지 않는다.\
어떤 트랜잭션에서 데이터를 변경하더라도 **COMMIT이 완료된 데이터만 다른 트랜잭션에서 조회**할 수 있다.

<img src="../../../.gitbook/assets/file.excalidraw (3).svg" alt="" class="gitbook-drawing">

예제와 같이 변경하더라도 언두 로그에 백업 되어 있는 데이터를 가져오게 된다.\
여기서 최종적으로 `커밋`을 해야 변경된 데이터를 참조할 수 있게 된다.

`READ COMMITTED` 격리 수준에서도 `NON-REPETABLE READ` 라는 부정합의 문제가 있다.\
하나의 트랜잭션 내에서 똑같은 쿼리를 실행 했을 때 항상 같은 결과를 가져와야 한다는 `REPETABLE READ` 정합성에 어긋나게 된다.

해당 문제를 해결하기 위해서 `REPETABLE READ` 이상의 격리 수준을 사용해야 한다.
