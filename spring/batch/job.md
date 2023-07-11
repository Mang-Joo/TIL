# Job

## Job

배치라는 계층 구조 안에서 가장 상위에 있는 것을 의미한다.

즉, 우리가 업무를 지시 받을 때 "OOO 배치 하나 만들어줘"라고 말한다면 그게 하나의 Job이 된다.

단 Job은 무조건 하나 이상의 Step을 포함해야 한다.

### Job 구현체

* Simple Job
  * 순차적으로 Step을 실행시키는 Job
* Flow Job
  * 특정한 조건과 흐름에 따라 Step을 구성하여 실행하는 Job

### Job 구성

1. JobParameters
2. JobLauncher\
   실제로 Job을 실행하는 객체
3. Job\
   하나의 배치를 실행하는 객체
4. Step\
   Job 안에 여러가지 로직을 담는 객체

