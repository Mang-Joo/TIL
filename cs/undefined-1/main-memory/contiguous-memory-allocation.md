# Contiguous Memory Allocation(연속된 메모리 할당)

일반적으로 여러 사용자 프로세스가 동시에 메모리에 상주하기를 원한다. 그래서 Contiguous Memory Allocation에서 각 프로세스는 다음 프로세스가 적재된 영역과 인접한 하나의 메모리 영역에 적재된다. 이 메모리 할당 기법을 사용하기 위해선 Memory Protection이 매우 중요하다.

## Memory Protection

프로세스가 자신이 소유하지 않은 메모리를 접근할 수 없게 강제할 수 있다. Relocation Register와 Limit Register를 사용하면 소유하지 않은 메모리를 접근할 수 없게 할 수 있다.

## Memory Allocation

Memory Allocation에서 가장 간단한 방법은 프로세스를 메모리의 가변 크기 파티션에 할당하는 것이다. **각 파티션엔 하나의 프로세스만** 적재될 수 있다. 해당 기법은 사용 가능한 메모리 부분과 사용 중인 부분을 나타내는 테이블을 유지한다.

처음에는 모든 메모리가 사용 가능하고, 하나의 사용 가능한 메모리 블록은 `hole`이라고 부른다.&#x20;

<figure><img src="../../../.gitbook/assets/Pasted image 20231218190335.png" alt=""><figcaption></figcaption></figure>

> 해당 그림을 보면 처음엔 프로세스들이 모든 메모리를 사용중이다.
>
> 그 후 중간에 process 8이 빠져나오며 hole이 생긴다.
>
> 그 hole에 빈 공간에서 process 9를 할당하고 마지막엔 process 5가 메모리를 반납하며 hole 이 2개가 생기게 된다.

이러한 기법은 Dynamic Storage Allocation Problem의 한 예이다. 가용 List로부터 필요한 메모리를 어떻게 할당 시켜줄 것인지에 대한 문제다.

이러한 알고리즘은 3가지 기법이 존재한다.

* First-Fit(최초 적합)
  * 사용 가능한 hole을 찾을 때 가장 먼저 만나는 hole을 만나면 적재하는 방법
* Best-Fit(최적 적합)
  * 사용 가능한 hole 중 가장 작은 것에 적재하는 방법
* Worst-Fit(최악 적합)
  * 사용 가능한 hole 중 가장 큰 것에 적재하는 방법

Worst-Fit을 제외한 방법을 추구하고, 어느 것이 더 좋다 라고 얘기할 수 없지만 속도적인 측면에선 First-Fit이 가장 빠르다.

## Fragmentation

위에서 얘기한 First-Fit, Best-Fit 모두 external fragmentation이 발생한다.

프로세스가 메모리에 적재되고 제거되는 일이 반복되다 보면 가용 공간이 너무 작은 조각들이 되게 된다.

이러한 공간들이 합치면 프로세스를 적재할 만큼 충분하지만 분산되어 있어서 사용하지 못하는 상황이 발생한다. 이러한 현상을 `External Fragmentation(외부 단편화)` 라고 한다.

Fragmentation은 external뿐만 아니라 internal에서도 발생한다.

메모리 가용 공간이 1,000 B지만 프로세스가 990 B의 가용공간이 필요하여 사용한다면 10 B 만큼의 hole이 생기게 된다.

이러한 현상을 `Internal Fragmentation(내부 단편화)`이라고 부른다.

그럼 External Fragmentation은 어떻게 해결해야 할까? 가장 간단한 방법으론 Compaction이 있다.

모든 내용을 한곳으로 몰고 가용 공간을 반대 방향으로 몰아서 큰 블록을 다시 만드는 것이다.

하지만 이러한 방법은 현실적으로 불가능하다.

프로세스 전체를 모두 새로운 메모리에 다시 적재해야 하고 그러한 시간동안에 생기는 지연 시간은 큰 문제가 될 수 있다.

그래서 메모리를 관리하기 위해서 `Paging` 이라는 기법을 사용한다.
