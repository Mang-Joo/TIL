# 유한 버퍼 문제

해당 문제는 프로듀서(Producer)와 컨슈머(Consumer) 간의 협력에 관련된 동기화 문제이다. 다수의 프로듀서가 데이터를 생성하고, 다수의 컨슈머가 데이터를 소비하는 상황에 발생하게 된다.

* 프로듀서는 버퍼에 데이터를 생성하여 저장한다. 그러나 버퍼가 가득 차면 추가로 데이터를 생성할 수 없다.
* 컨슈머는 버퍼에서 데이터를 소비한다. 하지만 데이터가 비어있으면 데이터를 소비할 수 없다.
* 프로듀서와 컨슈머는 동시에 실행될 수 있으며, 동시에 여러 프로듀서와 여러 컨슈머가 존재할 수 있다.
* 데드락이나 스레드 경쟁과 같은 문제가 발생할 수 있다.

이 문제를 해결하기 위해서 다음과 같은 자료 구조를 사용한다.

```c
int n;
semaphore mutex = 1;
semaphore empty = n;
semaphore full = 0;
```

n개의 버퍼로 구성된 풀이 있으며, 각 버퍼는 한 항목을 저장할 수 있다. mutex 이진 세마포어는 버퍼 풀에 접근하기 위한 상포 배제 기능을 제공하며 1로 초기화 된다. empty와 full 세마포어들은 각각 비어 있는 버퍼의 수와 꽉 찬 버퍼의 수를 기록한다.

생산자가 소비자를 위해 꽉 찬 버퍼를 생산해내고, 소비자는 생산자를 위해 비어 있는 버퍼를 생산해내는 것으로 해석할 수 있다.

```c
while(true) {
	wait(empty);
	wait(mutex);
	... // add item
	signal(mutex);
	signal(full);
}

while(true) {
	wait(full);
	wait(mutex);
	... // remove an item
	signal(mutex);
	signal(empty);
}
```
