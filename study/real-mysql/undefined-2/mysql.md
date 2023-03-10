---
description: InnoDB를 기준으로 작성
---

# MySQL 엔진 아키텍처

MySQL의 전체적인 구조는 다음과 같다.

<img src="../../../.gitbook/assets/file.drawing.svg" alt="" class="gitbook-drawing">

## MySQL 엔진

접속 및 쿼리 요청과 쿼리 최적화 등을 처리하는 엔진

## 스토리지 엔진

실제 데이터를 디스크 스토리지에 저장하거나 디스크 스토리지로부터 데이터를 읽어오는 엔진

## 핸들러 API

쿼리 실행기에서 데이터를 쓰거나 읽어야 할 때는 각 스토리지 엔진에 쓰기 또는 읽기를 요청하는데, 이러한 요청을 핸들러 요청이라 하고, 여기서 사용되는 API를 핸들러 API라고 한다.

InnoDB 스토리지 엔진 또한 이 핸들러 API를 이용해 MySQL 엔진과 데이터를 주고받는다.
