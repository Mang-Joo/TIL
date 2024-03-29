# Readers-Writers 문제

하나의 데이터베이스가 다수의 병행 프로세스 간에 공유된다고 하면, 일부는 데이터를 읽기만 할 수 있고, 일부는 데이터의 갱신 하기를 원할 수 있다.

이러한 경우 전자(읽기만 하는 경우)를 `Reader`라고 하고, 후자(갱신을 하는 경우) `writers`라고 한다.

하나의 `writer`와 다른 `Thread`가 동시에 접근하면, 혼란이 발생할 수 있다.

#### 첫번째 문제 해결안

`writer`가 공유 객체를 사용할 수 있는 허가를 아직 얻지 못했다면, 어느 `reader`도 기다리게 해서는 안된다.

단순히 `writer`가 기다리고 있기 때문에 다른 `reader`들이 끝날 때까지 기다리는 `reader`가 있어서는 안된다.

즉, `writer`때문에 `reader`가 지연되는 일이 발생하면 안된다.

#### 두번째 문제 해결안

`writer`가 준비되면 가능한 한 빨리 쓰기를 수행할 것을 요구한다. 즉, `writer`가 객체에 접근하려고 기다리고 있다면, 새로운 `reader`들은 읽기를 시작하지 못한다.

***

이러한 문제들은 기아 문제를 낳을 수 있다.

**첫 번째** 경우에는 `writer`가 기아상태에 빠질 수 있으며, **두 번째** 경우에는 `reader`가 기아 상태에 빠질 수 있다.

첫 번째 문제에 대한 해결안은 `mutex semaphore`를 추가하는 것이다. `reader` 프로세스는 다음과 같은 자료구조를 사용한다.

```C
semaphore rw_mutex = 1;
semaphore mutex = 1;
int read_count = 0;
```

`ru_mutex 세마포어`는 `reader`와 `writer`가 모두 공유한다. `mutex 세마포어`는 `read_count`를 갱신할 때 **상호 배제**를 보장하기 위해 사용한다.

`read_count`는 현재 몇 개의 프로세스들이 객체를 읽고 있는지 알려준다.

`rw_mutex 세마포어`는 `writer`들을 위한 상호 배제 세마포이다.

또한 임계구역으로 진입하는 `첫 번째 reader`와, 빠져나오는 `마지막 reader`에 의해서도 사용된다. 그러나 다른 `reader`들이 임계구역 안에 있는 동안 임계구역을 드나드는 `reader`들은 이것을 사용하지 않는다.

`writer`가 임계구역에 있고, n개의 `reader`가 기다리고 있으면 한 개의 `reader`만이 `rw_mutex`와 관련된 큐에 삽입되고, 나머지 n-1개의 `reeader`들은 `mutex`와 관련된 큐에 삽입된다. 또한 `writer`가 `signal(rw_mutex)`를 수행하면 대기 중인 `reader` 혹은 대기 중인 한 개의 `writer`의 수행이 재개된다. 어느쪽을 수행할 지는 `CPU 스케줄러`가 결정 할 것이다.

`Reader-Writer 락`은 대충화 되어 몇몇 일반 시스템에서 사용된다.(ex. db)

다음과 같은 상황에서 `Reader-Writer 락`을 사용하기 유용하다.

* 공유 데이터를 읽기만 하는 프로세스와 쓰기만 하는 스레드를 식별하기 쉬운 응용
* `writer`보다 `reader`의 개수가 많은 응용.
  * reader-writer 락을 설정하는 것이 세마포어나 상호 배제 락을 사용하는 것보다 오버헤드가 더 크다. 이 오버헤드는 여러 `reader`가 동시에 병행 처리를 함으로써 상쇄시킬 수 있다.
