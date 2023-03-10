# InnoDB 버퍼 풀(리두 로그)

InnoDB 스토리지 엔진에서 가장 핵심적인 부분이다.\
디스크의 데이터 파일이나 인덱스의 정보를 메모리에 캐시해 두는 공간이다.\
쓰기 작업을 지연시켜 일괄 작업으로 처리할 수 있게 해주는 버퍼 역할도 같이 한다.\
일반적인 애플리케이션에서 `INSERT`, `UPDATE`, `DELETE`처럼 데이터를 변경하는 쿼리는 데이터 파일의 이곳 저곳에 위치한 레코드를 변경하기 때문에 랜덤한 디스크 작업을 발생시킨다.\
하지만 버퍼 풀이 이러한 변경된 데이터를 모아서 처리하여 랜덤한 디스크 작업의 횟수를 줄이게 된다.

## 버퍼 풀의 구조

버퍼 풀이라는 메모리 공간을 페이지 크기의 조각으로 쪼개어 스토리지 엔진이 데이터를 필요로 할 때 해당 데이터 페이지를 읽어서 각 조각에 저장한다.\
버퍼 풀의 페이지 크기 조각을 관리하기 위해 스토리지 엔진은 크게 `LRU(Least Recently Used) 리스트`와 플러시 리스트, 그리고 프리 리스트라는 3개의 자료 구조를 관리한다.

* 프리 리스트: 버퍼 풀에서 실제 사용자 데이터로 채워지지 않은 비어 있는 페이지들의 목록이며, 사용자의 쿼리가 새롭게 디스크의 데이터 페이지를 읽어와야 하는 경우 사용된다.
* LRU 리스트: 엄밀하게 LRU와 MRU(Most Recently Used) 리스트가 결합된 형태라고 보면 된다.\
  &#x20;                  Old영역은 LRU에 해당하며, New영역은 MRU에 해당한다.

<img src="../../../.gitbook/assets/file.excalidraw (1) (2).svg" alt="LRU 리스트" class="gitbook-drawing">

해당 그림을 봤을 때 가장 먼저 떠오른건 GC이다.\
GC와 같이 많이 사용하는 건 조금씩 Old 영역으로 이동하고, 많이 사용하지 않는 것은 점점 밀려 결국엔 삭제되는 구조이다.

## 버퍼 풀과 리두 로그

버퍼 풀은 서버의 메모리가 허용하는 만큼 크게 설정하면 할수록 쿼리의 성능이 빨라진다.\
버퍼 풀은 데이터베이스 서버의 성능 향상을 위해 `데이터 캐시`와 `쓰기 버퍼링`이라는 두 가지 용도가 있는데, 메모리 공간을 늘리는 것은 데이터 캐시 기능만 향상 시키는 것이다.\
쓰기 버퍼링의 성능을 향상 시키려면 버퍼 풀과 리두 로그의 관계를 이해해야 한다.

버퍼 풀은 클린 페이지와 더티 페이지가 존재한다.

* 클린 페이지: 디스크에서 읽은 그 자체의 상태
* 더티 페이지: 디스크에서 읽은 파일에 `INSERT`, `UPDATE`, `DELETE`가 사용되어 데이터 변경된 상태

더티 페이지는 버퍼 풀에 무한정 머무를 수 있는 것은 아니다. 스토리지 엔진에서 리두 로그는 1개 이상의 고정 크기 파일을 연결해서 순환 고리처럼 사용한다.\
즉, 데이터 변경이 계속 발생하면 `리두 로그 파일`에 기록됐던 로그 엔트리는 어느 순간 다시 새로운 로그 엔트리로 덮어 쓰인다.

우리가 직접 Disk에 I/O 작업을 하게 되면 메모리에서 사용하는 것 보다 굉장히 느리다.\
그렇기 때문에 InnoDB에서는 바로 Disk를 사용하지 않고, `리두 로그`에 저장해둔다.\
`리두 로그`에 일정량이 다 차게 되면 그때 버퍼 풀이 Disk I/O를 한번에 하여 DB의 성능을 높여준다.

## Double Write Buffer

리두 로그는 리두 로그 공간의 낭비를 막기 위해 페이지의 변경된 내용만 기록한다.\
이로 인해 스토리지 엔진에서 더티 페이지를 디스크 파일로 플러시할 때 일부만 기록되는 문제가 발생하면 그 페이지의 내용은 복구할 수 없을 수도 있다.\
이렇게 페이지가 일부만 기록 되는 현상을 `파셜 페이지(Partial-page)` 라고 하는데, 이러한 현상은 하드웨어 문제나 시스템이 비정상 종료가 될 때 발생한다.

이러한 문제를 막기 위해 `Double-Write` 기법을 사용하여, 리두 로그를 양쪽에 저장한다.\
그 후 **DB 서버가 켜질 때** `Double Write Buffer`를 확인하여 문제가 생겼던 부분들을 처리한다.
