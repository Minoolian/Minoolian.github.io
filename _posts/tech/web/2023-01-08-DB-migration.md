---
layout: post
title: "DB migration"
category: tech
tags: web
image:
  path:   https://user-images.githubusercontent.com/44282342/215490324-8f5a8015-3523-4abe-9cb7-b491e2527325.png
---

* unordered toc
{:toc .large-only}

이번 포스팅은 **DB Migration**에 대해 알아보려한다.

## 필요성
***

개발하면서 요구사항에따라 테이블의 컬럼(필드)을 변경하곤 하는데, 문득 운영DB에서 기존 테이블의 변경이 있으면 어떻게 처리할까 궁금해졌다. 구글링을 통해 가장 많이 검색되는 DB 형상관리가 가능한 `Flyway`를 한번 사용해보려한다.

## Flyway
***

> 공식 페이지  
> Flyway is an open-source database migration tool. It strongly favors simplicity and convention over configuration.  
> If you are on the JVM, we recommend using the Java API for migrating the database on application startup. Alternatively, you can also use the Maven plugin or Gradle plugin.

공식페이지에 따르면 Java 진영에서 많이 사용되는 DB Migration 툴인것 같다.


### Dependency

```java
//file: "build.gradle"
dependencies {
    implementation 'org.flywaydb:flyway-core'
}
```

### application.yml

```yml
#file: "application.yml"
spring:
  h2:
    console:
      enabled: true
      path: /h2
  datasource:
    url: jdbc:h2:mem:test

  jpa:
    hibernate:
      ddl-auto: validate
    properties:
      hibernate:
        format_sql: true
```

DDL이 자동으로 수행되지 않도록 ddl-auto를 **validate**로 사용한다.

### Table 작성

```java
//file: "Person.java"
@Entity
public class Person {

    @Id
    @GeneratedValue(strategy = GenerationType.AUTO)
    private Long personId;

    private String name;

    private int age;
    
}
```

테스트용 Person Table을 작성하여 이를 테스트 해보려한다. 현재 `ddl-auto=validate` 상태이고, 휘발성 DB인 H2를 사용하고 있으므로 유효성 검증이 실패할 것이다.

### 버전관리

Flyway를 통해 버전관리를 함으로써 변경이력들을 추적할 수 있고, SQL 쿼리를 직접 작성하여 무분별한 테이블 DDL을 막을 수 있다.

* 기본 경로는 의존성을 추가하면서 자동으로 생성된 `resources.db.migration`이며 외부 설정파일(application.yml)을 통해 변경 가능하다.
* 별도의 Naming 규칙을 반드시 지켜야한다.

![image](https://user-images.githubusercontent.com/44282342/215480396-1305cefa-ffa1-4b85-ba6d-7c21e1f7c692.png)

```sql
-- file: "V1__init_db.sql"
create sequence person_seq start with 1 increment by 50;

create table person(
    person_id  bigint  not null,
    name        varchar(255),
    age         int,
    primary key (person_id)
)
```

Person 테이블 생성에 대한 V1 DDL이다.

![image](https://user-images.githubusercontent.com/44282342/215486465-d77f4573-5746-44b4-a3e7-7cb52fd80c0a.png)

버전관리 테이블이 생성된다.

이후 테이블의 DDL이 필요하다면 새 버전을 생성하여 관리한다.

```sql
-- file: "V2__add_person_address.sql"
alter table person add column address varchar(255);
```

Person 테이블의 address 컬럼을 추가하여 테스트한 결과 정상적으로 버전이 기록되는 것을 확인할 수 있다. 만약 버전을 작성하지 않는다면 유효성 검증 과정에서 `missing column` 오류가 발생한다.

![image](https://user-images.githubusercontent.com/44282342/215487210-fe680123-e1da-46a2-b889-22bfa21ec5f3.png)

## 결론
***

운영DB에서 `ddl-auto`옵션의 `create` `create-drop` `update`을 사용하면 안된다는 것을 알고 있었지만, 그 후에는 직접 DB에 접속하여 작성해야하는지 아니면 어떻게 관리하면 되는지 궁금했었는데 말끔히 해결되었다. 또한 DML 쿼리들보단 DDL 쿼리 사용이 적었는데 **Flyway 사용**으로 조금 더 익숙해질 수 있는 계기가 될 것으로 보인다.