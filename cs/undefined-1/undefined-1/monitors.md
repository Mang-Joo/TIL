# Monitors

프로그래머가 세마포를 잘못 사용하게 되면 다야안 유형의 오류가 너무나도 쉽게 발생할 수 있다.

이러한 오류를 처리하기 위해 동기화 도구를 통합하여 고급 언어 구조물을 제공하여 해결하는 것이다.

그 중 하나가 `montiros(모니터)` 이다.

또한 모니터를 구현할 때는 세마포어를 사용하여 상호배제를 구현하는 경우가 많다. 모니터에 진입하는 동안에는 세마포어를 이용하여 다른 스레드가 모니터에 접근할 수 없게 만드는 것이다.

**사용법**

`abstract data type(ADT)` (추상화 된 데이터 형)은 데이터와 데이터를 조작하는 함수들의 집합을 하나의 단위로 묶어 보호한다.

`모니터 형`은 모니터 내부에서 상호 배제가 보장되는 프로그래머가 정의한 일련의 연산자 집합을 포함하는 ADT이다.

또한 이 변수들을 조작하기 위해 프로시저 or 함수들도 같이 포함하고 있다.

**monitor**의 대표적인 예는 `java`에서 `synchronized` 키워드를 사용하는 방식이다.

```java
public class SynchronizedExample {
    private int counter = 0;

    // synchronized 메서드
    public synchronized void increment() {
        counter++;
    }

    // synchronized 블록
    public void performTask() {
        synchronized (this) {
            counter++;
        }
    }

    public static void main(String[] args) {
        SynchronizedExample example = new SynchronizedExample();

        Thread thread1 = new Thread(() -> {
            for (int i = 0; i < 1000; i++) {
                example.increment();
            }
        });

        Thread thread2 = new Thread(() -> {
            for (int i = 0; i < 1000; i++) {
                example.increment();
            }
        });

        thread1.start();
        thread2.start();

        try {
            thread1.join();
            thread2.join();
        } catch (InterruptedException e) {
            e.printStackTrace();
        }

        System.out.println("Counter: " + example.counter);
    }
}

```
