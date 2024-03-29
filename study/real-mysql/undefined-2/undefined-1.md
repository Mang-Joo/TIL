# 쿼리 실행 구조

<img src="../../../.gitbook/assets/file.excalidraw (2) (1) (1).svg" alt="쿼리 실행 구조" class="gitbook-drawing">

## 쿼리 파서

요청 들어온 쿼리 문장을 토큰으로 분리해 트리 형태의 구조로 만들어낸다.\
문법 오류는 이 과정에서 발견되고 에러 메세지를 전달한다.

## 전처리기

파서 과저에서 만들어진 트리를 기반으로 구조적인 문제점이 있는지 확인한다.\
각 토큰을 테이블 이름이나 컬럼 이름, 또는 내장 함수와 같은 개체를 매핑해 해당 객체의 존재 여부와 객체의 접근 권한 등을 확인한다.

## 옵티마이저

사용자의 요청으로 들어온 쿼리를 저렴한 비용으로 가장 빠르게 처리할지 결정한다.\
즉 나름의 최적화를 MySQL이 한번 해준다.

## 실행 엔진

실행 엔진은 만들어진 계획대로 각 핸들러에게 요청해서 받은 결과를 또 다른 핸들러 요청의 입력으로 연결하는 역할 수행한다.

즉 실행 엔진이 하는 역할은 스토리지 엔진과의 커넥션 역할을 한다고 할 수 있다.

## 핸들러(스토리지 엔진)

핸들러는 MySQL 서버의 가장 아래에서 실행 엔진의 요청에 따라 데이터를 디스크로 저장하고 읽어오는 역할을 한다.

