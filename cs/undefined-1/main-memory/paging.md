# Paging(페이징)

## Basic

Physical Memory는 Frame이라고 불리는 같은 크기의 블록으로 잘게 쪼개진다. Logical Memory는 Page라고 불리는 같은 크기의 블록으로 잘게 쪼개진다.

이렇게 해서 프로세스가 실행 될 때 프로세스는 Page를 통해서 Frame에 할당한다.

이렇게 사용하면 하나의 프로세스는 메모리에 연속적이게 할당되지 않아도 모든 부분을 메모리에 할당시킬 수 있다.

CPU에서 나오는 주소는 Page Number(P)와 Page Offset(D)의 부분으로 나누어진다.

Page Number는 Page Table을 액세스 할 때 사용한다.

> Page Table
>
> Page Table은 각 프로세스마다 하나의 Page Table을 가지게 된다. 이 Page Table엔 하나의 프로세스가 사용하는 Page들을 다 가지고 있다. 그래서 Logical Memory는 Table 안에 있는 Page를 바라보게 되고 Table을 통해서 Physical Memory에 할당한다.

<figure><img src="../../../.gitbook/assets/Pasted image 20231218192312.png" alt=""><figcaption></figcaption></figure>

위 그림과 같이 CPU는 Page Number(P)를 통해서 Page Table에 접근한다. 그럼 Page Table에서 가지고 있는 Frame Number를 찾게 되고, Offset은 그대로 사용하며 F, D의 조합이 생성된다. 그럼 이 F, D 조합을 통해서 Physical Memory에 접근할 수 있게 된다.

## Hardware Support

Page Table은 프로세스 별 자료구조이므로 Table에 대한 포인터는 각 프로세스의 PCB에 저장된다.

CPU Scheduler가 실행할 프로세스를 선택하면 사용자 Register에 다시 적재하고, 저장 된 사용자 Page Table로부터 적절한 Hardware Page Table value를 다시 적재해야된다.

그래서 대 부분의 테이블을 Main Memory에 저장하고, `Page-Table Base Register(PTBR)` 로 하여금 Page Table을 가리키게 된다. 이렇게 사용하게 되면 Register만을 변화시키면 되고 Context Switching 시간을 감소시킬 수 있다.

하지만 이 방법을 사용하면 Context Switching 시간은 감소하지만 Memory Access Time이 증가하게 된다.

이유는 Memory Access를 2번 해야 하기 때문이다.

Page Number를 기준으로 Page Table에서 원하는 Frame Number를 가져온다. 이 때 1번 Access한다. 그렇다면 실제 데이터에 접근하기 위해 또 한번 Access 한다. 이렇게 총 2번의 메모리 접근이 발생하게 된다.

이 문제를 해결하기 위해 `Translation Look-aside Buffers(TLB)`라고 불리우는 `Hardware Cache`가 사용된다.

TLB는 `associative memory(연관 메모리)`로 구성된다.

먼저 CPU는 TLB에 Page를 찾아달라고 요청을 한다.

그럼 TLB는 해당 Page Number가 있으면 메모리 Page Table에 Access하지 않고 바로 메모리에 Access 할 수 있게 된다.

TLB에 있는 Page를 찾게 되면 `TLB Hit`라고 하고, TLB에 없으면 `TLB Miss` 라고한다

## Protection

그렇다면 Page는 어떻게 보호를 처리할까?

각 페이지에 붙어있는 Protection bits에 의해 구현된다. 이 bits는 Page Table에 속해 있다.

각 Bit는 `읽고 쓰기` 또는 `Read Only` 임을 각각 정의 할 수 있다.

Read Only에 쓰기를 요청하면 Memory protection violation이 발생한다.

각 entry에는 valid / invalid 라는 비트가 있다. valid로 설정되면 합법적인 페이지를 나타내며 invalid는 Logical Address space에 속하지 않는다는 것을 의미한다.

어떤 14비트의 Address space System에서 프로그램은 10비트를 사용할 수 있다. 이러한 상황에 11비트 이상에 접근하려고 하면 invalid page reference가 발생한다.

## Shared Pages (DLL)

Paging의 장점은 공통의 코드르 **Share** 할 수 있다는 점이다.

사용자 프로세스에서 표준 C 라이브러리인 libc가 필요하다고 하자.

각 프로세스가 자체 libc의 사본을 해당 주소 공간에 적재하는 방법이 있다.

하지만 이렇게 사용하게 된다면 libc가 2MB에 40개의 프로세스가 올라간다면 80MB를 사용하게 된다.(같은 라이브러리 코드로)

하지만 libc 같은 경우는 자체 수정을 할 수 없는 코드로서 실행 중에 절대 변경되지 않는 읽기 전용 코드이다.

그래서 해당 코드를 여러 프로세스에서 같은 곳을 바라보게 만들어 공통 코드를 한 공간에만 올려 재사용 할 수 있다.

<figure><img src="../../../.gitbook/assets/Pasted image 20231218222625.png" alt=""><figcaption></figcaption></figure>
