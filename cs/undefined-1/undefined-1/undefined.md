# 세마포어

세마포어(Semaphore)는 동기화와 상호배타적 접근을 관리하기 위한 동기화 도구 중 하나이다. 주로 다중 프로세스나 스레드 간의 공유 자원에 대한 접근을 제어하고 조절하는 데 사용된다.

세마포어는 두 가지 주요 연산을 제공한다:

1. P(Produce) 연산: 이 연산은 세마포어 값을 1 감소시키고, 값이 0보다 작아지면 호출하는 프로세스나 스레드는 대기 상태로 전환된다. 이 연산은 세마포어 값을 감소시키는 작업을 나타낸다.
2. V(Verhogen) 연산: 이 연산은 세마포어 값을 1 증가시키고, 대기 중인 다른 프로세스나 스레드가 있다면 하나를 깨운다. 이 연산은 세마포어 값을 증가시키는 작업을 나타낸다.

Semaphore를 사용하면 공유 자원에 대한 접근을 조절할 수 있으며, 경쟁 조건을 방지하고 동기화 문제를 해결한다.

Semaphore는 값이 1 이상일 때 공유 자원에 접근 할 수 있다.

소유 개념이 없으므로 내가 직접 해지를 하지 않아도 된다.

**예제 코드(C)**

```c
#include <stdio.h>  
#include <pthread.h>  
  
int semaphore = 1;  
pthread_mutex_t mutex = PTHREAD_MUTEX_INITIALIZER;  
pthread_cond_t condition = PTHREAD_COND_INITIALIZER;  
  
void P() {  
    pthread_mutex_lock(&mutex);  
    while (semaphore <= 0) {
        pthread_cond_wait(&condition, &mutex);  
    }  
    semaphore--;  
    pthread_mutex_unlock(&mutex);  
}  
  
void V() {  
    pthread_mutex_lock(&mutex);  
    semaphore++;  
    pthread_cond_signal(&condition); // 대기 중인 스레드 깨움  
    pthread_mutex_unlock(&mutex);  
}  
  
void *worker(void *arg) {  
    // 공유 자원에 대한 작업 수행  
    P(); // 공유 자원에 대한 잠금  
    printf("Thread %ld is inside the critical section.\n", (long) arg);  
    // 일부 작업 수행  
    V(); // 공유 자원 잠금 해제  
    return NULL;  
}  
  
int main() {  
    pthread_t threads[3];  
    long i;  
    for (i = 0; i < 3; i++) {  
        pthread_create(&threads[i], NULL, worker, (void *) i);  
    }  
  
    for (i = 0; i < 3; i++) {  
        pthread_join(threads[i], NULL);  
    }  
  
    return 0;  
}
```
