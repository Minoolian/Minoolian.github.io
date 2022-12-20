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


## 사용 예시
***

### 1. 단일 Step

```java
@Slf4j
@Configuration
@EnableBatchProcessing
public class ExampleMonoJobConfig {

    @Autowired
    public JobBuilderFactory jobBuilderFactory;

    @Autowired
    public StepBuilderFactory stepBuilderFactory;

    @Bean
    public Job exampleMonoJob() {

        Job exampleJob = jobBuilderFactory.get("exampleJob")
                .start(step())
                .build();

        return exampleJob;
    }

    @Bean
    public Step step() {
        return stepBuilderFactory.get("step")
                .tasklet((contribution, chunkContext) -> {
                    log.info("Step!");
                    return RepeatStatus.FINISHED;
                })
                .build();
    }
}
```

### 2. 다중 Step

```java
@Slf4j
@Configuration
@EnableBatchProcessing
public class ExampleMultiJobConfig {

    @Autowired
    public JobBuilderFactory jobBuilderFactory;

    @Autowired
    public StepBuilderFactory stepBuilderFactory;

    @Bean
    public Job exampleMultiJob() {

        Job exampleJob = jobBuilderFactory.get("exampleJob")
                .start(startStep())
                .next(nextStep())
                .next(lastStep())
                .build();

        return exampleJob;
    }

    public Step lastStep() {
        return stepBuilderFactory.get("lastStep")
                .tasklet((contribution, chunkContext) -> {
                    log.info("last step");
                    return RepeatStatus.FINISHED;
                })
                .build();
    }

    public Step nextStep() {
        return stepBuilderFactory.get("nextStep")
                .tasklet((contribution, chunkContext) -> {
                    log.info("next step");
                    return RepeatStatus.FINISHED;
                })
                .build();
    }

    public Step startStep() {
        return stepBuilderFactory.get("startStep")
                .tasklet((contribution, chunkContext) -> {
                    log.info("start step");
                    return RepeatStatus.FINISHED;
                })
                .build();
    }
}
```

### 3. Flow를 통한 Step

```java
@Slf4j
@Configuration
@EnableBatchProcessing
public class ExampleFlowJobConfig {

    @Autowired
    public JobBuilderFactory jobBuilderFactory;

    @Autowired
    public StepBuilderFactory stepBuilderFactory;

    @Bean
    public Job exampleFlowJob() {

        Job exampleJob = jobBuilderFactory.get("exampleFlowJob")
                .start(startStep())
                .on("FAILED")
                .to(failOverStep())
                .on("*")
                .to(writeStep())
                .on("*")
                .end()

                .from(startStep())
                .on("COMPLETED")
                .to(processStep())
                .on("*")
                .to(writeStep())
                .on("*")
                .end()

                .from(startStep())
                .on("*")
                .to(writeStep())
                .on("*")
                .end()

                .end()
                .build();

        return exampleJob;
    }

    @Bean
    public Step startStep() {
        return stepBuilderFactory.get("startStep")
                .tasklet((contribution, chunkContext) -> {
                    log.info("Start Step!");

                    String result = "COMPLETED";

                    //Flow에서 on은 RepeatStatus가 아닌 ExitStatus를 바라본다.
                    if(result.equals("COMPLETED"))
                        contribution.setExitStatus(ExitStatus.COMPLETED);
                    else if(result.equals("FAIL"))
                        contribution.setExitStatus(ExitStatus.FAILED);
                    else if(result.equals("UNKNOWN"))
                        contribution.setExitStatus(ExitStatus.UNKNOWN);

                    return RepeatStatus.FINISHED;
                })
                .build();
    }

    @Bean
    public Step failOverStep(){
        return stepBuilderFactory.get("failedOverStep")
                .tasklet((contribution, chunkContext) -> {
                    log.info("FailOver Step!");
                    return RepeatStatus.FINISHED;
                })
                .build();
    }

    @Bean
    public Step writeStep(){
        return stepBuilderFactory.get("writeStep")
                .tasklet((contribution, chunkContext) -> {
                    log.info("Write Step!");
                    return RepeatStatus.FINISHED;
                })
                .build();
    }

    public Step processStep(){
        return stepBuilderFactory.get("processStep")
                .tasklet((contribution, chunkContext) -> {
                    log.info("Process Step!");
                    return RepeatStatus.FINISHED;
                })
                .build();
    }
}
```

위의 flow 구성에서 `flow`와 `from`이 `if-else if`와 유사한 동작을 하는 것을 확인하였다.


## 결론
***

