# 스레드 스케줄링(Thread Scheduling)

대부분 최신 운영체제에서는 스케줄 되는 대상은 프로세스가 아니라 커널 수준 스레드이다. 사용자 수준 스레드는 스레드 라이브러리에 의해 관리되고 커널은 그들의 존재를 알지 못한다. LWP를 통한 간접적인 방식일지라도 사용자 수준 스레드는 궁극적으로 영관된 커널 수즌 스레드에 사상되어야 한다.

> LWP(Light Weight Process)
>
> 경량 프로세스 프로세스에 있는 메모리 공간을 공유한다.

### 경쟁 범위

사용자 수준과 커널 수준 스레드의 차이 중 하나는 그들이 어떻게 스케줄 되느냐에 있다. N:1과 N:M 모델을 구현하는 시스템에서는 LWP상에서 스케줄 한다.

동일한 프로세스에 속한 스레드들 사이에서 CPU를 경쟁하기 때문에 프로세스-경쟁-범위(Process-Contention Scope, **PCS**)로 알려져 있다.

LWP의 커널 스레드를 물리적인 CPU 코어로 스케줄 하는 것을 필요로 한다. CPU상에 어느 커널 스레드를 스케줄 할 것인지 결정하기 위해서 커널은 시스템-경쟁-범위(System-Contention Scope, SCS)를 사용한다. CPU에 대한 경쟁은 시스템 상의 모든 스레드 사이에서 일어난다. 1:1 모델을 사용하는 시스템은 오직 SCS만을 사용하여 스케줄 한다.

PCS는 우선순위에 따라 행해진다.스케줄러는 가장 높은 우선순위를 가진 실행 가능한 프로세스를 선택한다. 사용자 수준 스레드의 우선순위는 프로그래머에 의해 지정되고 라이브러리에 의해 조정되지 않는다.



### 다중 처리기 스케줄링(Multiple-Processor Scheduling)

여러 개의 CPU가 사용 가능하다면, 여러 스레드가 병렬로 실행될 수 있으므로 **부하 공유(load sharing)** 가 가능해진다. 하지만 이렇게 사용하게 되면 스케줄링 문제는 더욱 더 복잡해진다. Multi-Processor라는 용어는 여러 개의 물리적 프로세서를 제공하는 시스템을 말한다.

**Multi-Processor**는 다음 시스템 아키텍처들에 사용할 수 있다.

* 다중 코어 CPU
* 다중 스레드 코어
* NUMA 시스템
* 이기종 다중 처리



### Multi-Processor 스케줄링에 대한 접근 방법

Multi-Processor 시스템에서 CPU 스케줄링에 관한 한 가지 해결 방법은 `마스터 서버(master server)` 라는 하나의 처리기가 모든 스케줄링 결정과 I/O처리 그리고 다른 시스템의 활동을 취급하게 하는 것이다. 다른 처리기들은 사용자 코드만을 수행한다. 이러한 `비대칭 다중 처리(asymmetric multiprocessing)`는 오직 하나의 코어만 시스템 자료구조에 접근하여 자료 공유의 필요성을 배제하기 때문에 간단하다. 하지만 이 방식의 단점은 마스터 서버가 전체 시스템 성능을 저하할 수 있는 병목이 된다는 것이다.

표준적인 접근 방식은 `대칭 다중 처리(symmetric multiprocessing, SMP)`이며 각 프로세서는 스스로 스케줄링 할 수 있다. 스케줄러가 준비 큐를 검사하고, 실행할 스레드를 선택하여 스케줄링이 진행된다. 스레드를 관리하기 위한 두 가지 전략을 제공한다.

1. 모든 스레드가 공통 준비 큐에 있을 수 있다.
2. 각 프로세서는 자신만의 스레드 큐를 가질 수 있다.

**첫 번째 옵션**을 선택하면 공유 준비 큐에 경쟁 조건이 생길 수 있으므로, 두 개의 다른 프로세서가 동일한 스레드를 스케줄 하지 않도록 그리고 큐에서 스레드가 없어지지 않도록 보장해야 한다. 그렇기 때문에 **경쟁 조건**으로부터 공통 준비 큐를 보호하기 위해 락킹 기법의 하나를 사용할 수 있다.

두 번째 옵션은 각 프로세서가 자신만의 실행 큐에서 스레드를 스케줄 할 수 있도록 허용하므로 공유 실행 큐와 관련되어 발생할 수 있는 **성능 문제를 겪지 않는다**. 따라서 SMP를 지원하는 시스템에서 가장 일반적인 접근 방식이다.
