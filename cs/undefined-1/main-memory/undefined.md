# 배경

CPU 이용률과 사용자에 제공하는 컴퓨터 응답속도를 모두 향상하는 것을 실현하기 위해선 많은 프로세스를 메모리에 유지해야 한다. 즉, 메모리를 공유해야 한다.

### 기본 하드웨어

메인 메모리는 CPU가 직접 접근할 수 있는 유일한 범용 저장장치이다. 명령어들은 메모리 주소만을 인수로 취하고, 디스크의 주소를 인수로 취하지 않는다. -> 그렇기 때문에 모든 명령어와 데이터들은 메인 메모리와 레지스터에 있어야한다.

각각의 프로세스는 독립된 메모리 공간을 가지도록 보장해야 한다. 서로를 보호하고 병행 실행을 위해 여러 프로세스가 메모리에 적재되게 하는 것이 필수적이다. 메모리 공간을 분리하기 위해 legal memory address를 설정하고 프로세스가 합법적인 영역만을 접근하도록 하는 것이 필요하다.

가장 기본적인 방법은 base와 limit이라고 불리는 두 개의 레지스터들을 사용하여 보호 기법을 제공한다. base register는 가장 하단의 메모리 주소를 저장하고, limit register는 주어진 영역의 크기를 저장한다.

예를 들어 base register가 300,400이고, limit register가 10,000 이라면 프로그램은 300,400부터 310,400까지 사용할 수 있는 것이다. 만약 다른 프로세스가 해당 메모리 공간에 접근하게 되면 오류로 간주하고 trap을 발생시킨다.

### Address Binding(주소 할당)

프로그램은 이진 실행 파일 형태로 디스크에 저장되어 있다. 이 프로그램을 실행하려면 메모리로 가져와서 프로세스 문맥 내에 배치되어야 한다.

프로세스가 실행되면 메모리에서 모든 것을 액세스하고, 프로세스가 종료 되면 메모리를 회수한다.

보통 프로그램은 물리 메모리에 직접 할당하는 것이 아니라, int count 와 같이 symbol 형태로 표현된다. 컴파일러가 오브젝트 파일을 만들게 되면 Linker를 통해서 Logical Address가 등록되고, 실행 파일로 실행되면서 Loader를 통해서 Logical Address가 Physical Address로 적재된다.

### Logical vs Physical Space (논리 대 물리 주소 공간)

모든 논리 주소 집합을 Logical Address Space 라고 부르며, 논리 주소와 일치하는 모든 물리 주소 집합을 Physical Address Space라고 부른다.

그러면 `Logical Address`를 `Physical Address`로 변경해야 하는데 이 변환 작업은 하드웨어의 지원을 받아 진행되게 된다.

이 장치가 바로 **`Memory Management Unit(MMU)`** 이다. 모든 메모리는 `MMU`를 통해 `Logical to Physical`을 하게 된다. 뒤에서 나올 페이징 기법도 결국 MMU를 통해서 변환 작업을 가지게 된다.

사용자 프로그램은 절대 **Physical Memory Address** 에 접근하지 않는다. 사용자 프로그램은 `int count`를 `123` 이라는 포인터를 생성해서 그것에 대한 일을 처리한다.

그럼 MMU는 Relocation Register를 가지고 있는데 Relocation Register의 값이 `14000` 이라고 가정하면 MMU를 통해서 123이라는 Logical Address가 14123이라는 Physical Address로 접근시켜준다.



### Dynamic Loading (동적 적재)

Process가 실행되기 위해 미리 메모리에 올라와 있어야 한다. 하지만 이런 경우 프로세스의 크기가 메모리의 크기보다 커서는 안 된다. 그래서 메모리 공간을 더 효율적으로 이용하기 위해서 dynamic loading을 한다.

메모리에 프로그램이 적재되지만 루틴이 실제로 호출되기 전까진 디스크에 머물러 있는다. 그러고 루틴을 호출하게 되면 루틴이 이미 메모리에 적재됐는지를 조사한다. 적재되어 있지 않다면 relocatable linking loader가 불려 루틴을 메모리로 가져오고 테이블에 기록한다.

이 방법의 장점은 루틴이 필요한 경우에만 적재 된다는 것이다.

### Dynamic Linking & Shared Libraries

Dynamic Linking Library(DLL)은 프로그램이 실행될 때 사용자 프로그램에 연결되는 시스템 라이브러리이다.

연결에는 Static Linking과 Dynamic Linking이 있다.

Static Linking

* 라이브러리가 이진 프로그램에 끼어 들어가게 된다.
* 즉 컴파일 시점에 해당 라이브러리는 이미 포함되어 있다. Dynamic Linking
* 이것은 Dynamic Loading과 비슷하게 Linking이 실행 시점까지 미뤄지는 것이다.
* 예를 들면 libc 같은 것들이 있다.
