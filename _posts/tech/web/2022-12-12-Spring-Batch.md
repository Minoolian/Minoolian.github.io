---
layout: post
title: "Spring Batch"
category: tech
tags: web
image:
  path:   https://user-images.githubusercontent.com/44282342/205482196-c8fbae08-98da-44ce-822e-7b7d99b4c315.png
---

* unordered toc
{:toc .large-only}

이번 포스팅은 **Spring Batch**에 대해 알아보려한다.

## Spring Batch
***

로깅/추적, 트랜잭션 관리, 작업 처리 통계, 작업 재시작, 리소스 관리 등 대용량 레코드 처리에 필수적인 기능을 제공한다. 또한 최적화 및 파티셔닝 기술을 통해 대용량 및 고성능 배치 작업을 가능하게 하는 고급 기술 서비스 및 기능을 제공한다.

Spring의 특성을 갖기 때문에 DI, AOP, 추상화 등 Spring Framework의 3대 요소를 모두 사용할 수 있다.

* 대용량의 비즈니스 데이터를 복잡한 작업으로 처리
* 특정 시점에 스케쥴러를 통한 자동화 작업 (푸시알림, 리포트 산출)
* 대용량 데이터 포맷을 변경, 유효성 검사 등 트랜잭션 안에서 처리 후 기록

> Spring Batch? Scheduler?
> Batch는 Job을 관리하지만 구동, 실행 기능이 없고, Scheduler는 이 반대로 Batch Job을 실행시키기 위해 Quartz와 같은 Scheduler를 사용한다.

## 용어
***

### Job

배치 과정을 하나의 단위로 만든 객체. 또한 배치 처리 과정에서 최상위 계층에 위치한다.

여러 Step 인스턴스를 포함하는 컨테이너이다.

### JobInstance

Job의 실행 단위. Job이 실행되면 하나의 Instance가 생성되며, 각 Instance는 `JobParameter 객체`에 따라 구분된다.

### JobExecution

JobInstance 실행 시도에 대한 객체. JobInstance 실행에 관한 상태, 시작시간, 종료시간, 생성시간 등의 정보를 갖는다.

### Step

Job 배치처리 정의 및 순차적인 단계 캡슐화. Job은 최소한 1개 이상의 Step을 가지며, 일괄 처리를 제어하는 모든 정보를 담고 있다.

Tasklet 또는 Chunk 기반의 처리방식 2개로 나뉜다.

### StepExecution

Step 실행 시도에 대한 객체. Job이 여러 Step으로 구성될 경우 하나의 Step이 실패하면 StepExecution은 생성되지 않는다. JobExecution에 저장된 정보 외 read, write, commit, skip의 수 등 정보를 저장한다.

### ExecutionContext

Job에서 데이터를 공유할 수 있는 저장소. JobExecutionContext는 Commit 시점에 저장되며, StepExecutionContext는 실행 사이에 저장된다. 이를 통해 Step간의 데이터공유 및 Job 실패시 마지막 실행 값을 재구성할 수 있다.

### JobRepository

위의 모든 배치처리 정보를 담는 매커니즘.

### JobLauncher

Job과 JobParameter를 사용하여 Job을 실행하는 객체. Job의 재실행 가능 여부 검증, 실행 방법, Parameter 유효성 검증 등을 수행한다.

스프링부트 환경에선 Job을 시작하는 기능을 제공하므로 다룰 필요가 없다.

### ItemReader

Step에서 Item을 읽어오는 인터페이스. 다양한 방법으로 Item을 읽을 수 있다.

### ItemWriter

처리된 데이터를 쓸때 사용. 처리 결과에 따라 insert, update, queue에 send 등의 행위를 한다. Item을 Chunk로 묶어 처리한다.

### ItemProcessor

Reader에서 읽은 Item 데이터를 처리. 필수 요소는 아님.

![image](https://user-images.githubusercontent.com/44282342/207274595-4848492d-b177-49fd-b688-80ee9e94aa62.png)


## hashCode()
***



## 결론
***

이 방법을 통해서 PS 풀이를 할 때 두 좌표객체의 동일성을 if문과 함께 검증할 수 있었다. 또한 추후 JPA Entity 객체에서 맞닥뜨렸던 문제를 다시 한번 접하게 된다면 지금 알게된 새로운 방법으로 접근하여 해결할 수 있을 것이다.

여러 자료를 검색하다보니 ***Hash를 이용한 자료구조를 사용하지 않으면*** `hashCode()` 메서드는 재정의하지 않아도 무방하지 않은가? 라는 의문에 대한 글을 많이 접한것 같다. 필자들마다 조금은 의견이 상이하지만 결국 Hash 자료구조를 사용할 일말의 가능성이 있기에 `equals()` 메서드를 재정의한다면 동시에 `hashCode()` 메서드도 **함께 재정의하는 것이 맞다**고 본다.