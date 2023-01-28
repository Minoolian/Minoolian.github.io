---
layout: post
title: "[Spring] Springboot 3 with Java 11"
category: tech
tags: troubleshooting spring
---

Springboot 3버전 프로젝트를 Java 11로 구성할 때 발생한 오류에 대해 포스팅하려 한다.

* unordered toc
{:toc .large-only}

# 👿 Problem
***

Springboot 3.0.2 버전으로 프로젝트를 구성하려는데, 아래와 같은 에러가 발생하면서 Gradle이 import되지 않는 현상이 발생했다.

```shell

No matching variant of org.springframework.boot:spring-boot-gradle-plugin:3.0.2 was found. The consumer was configured to find a runtime of a library compatible with Java 11, packaged as a jar, and its dependencies declared externally, as well as attribute 'org.gradle.plugin.api-version' with value '7.6' but:
          - Variant 'apiElements' capability org.springframework.boot:spring-boot-gradle-plugin:3.0.2 declares a library, packaged as a jar, and its dependencies declared externally:
              - Incompatible because this component declares an API of a component compatible with Java 17 and the consumer needed a runtime of a component compatible with Java 11
              - Other compatible attribute:
                  - Doesn't say anything about org.gradle.plugin.api-version (required '7.6')
          - Variant 'javadocElements' capability org.springframework.boot:spring-boot-gradle-plugin:3.0.2 declares a runtime of a component, and its dependencies declared externally:
              - Incompatible because this component declares documentation and the consumer needed a library
              - Other compatible attributes:
                  - Doesn't say anything about its target Java version (required compatibility with Java 11)
                  - Doesn't say anything about its elements (required them packaged as a jar)
                  - Doesn't say anything about org.gradle.plugin.api-version (required '7.6')
          - Variant 'mavenOptionalApiElements' capability org.springframework.boot:spring-boot-gradle-plugin-maven-optional:3.0.2 declares a library, packaged as a jar, and its dependencies declared externally:
              - Incompatible because this component declares an API of a component compatible with Java 17 and the consumer needed a runtime of a component compatible with Java 11
              - Other compatible attribute:
                  - Doesn't say anything about org.gradle.plugin.api-version (required '7.6')
          - Variant 'mavenOptionalRuntimeElements' capability org.springframework.boot:spring-boot-gradle-plugin-maven-optional:3.0.2 declares a runtime of a library, packaged as a jar, and its dependencies declared externally:
              - Incompatible because this component declares a component compatible with Java 17 and the consumer needed a component compatible with Java 11
              - Other compatible attribute:
                  - Doesn't say anything about org.gradle.plugin.api-version (required '7.6')
          - Variant 'runtimeElements' capability org.springframework.boot:spring-boot-gradle-plugin:3.0.2 declares a runtime of a library, packaged as a jar, and its dependencies declared externally:
              - Incompatible because this component declares a component compatible with Java 17 and the consumer needed a component compatible with Java 11
              - Other compatible attribute:
                  - Doesn't say anything about org.gradle.plugin.api-version (required '7.6')
          - Variant 'sourcesElements' capability org.springframework.boot:spring-boot-gradle-plugin:3.0.2 declares a runtime of a component, and its dependencies declared externally:
              - Incompatible because this component declares documentation and the consumer needed a library
              - Other compatible attributes:
                  - Doesn't say anything about its target Java version (required compatibility with Java 11)
                  - Doesn't say anything about its elements (required them packaged as a jar)
                  - Doesn't say anything about org.gradle.plugin.api-version (required '7.6')

```

위 error 메시지를 보면 Java 11과 호환에 대한 문제가 있는 것으로 보이기에 해결방법을 찾아보았다.


# 👼 Solution
***

![image](https://user-images.githubusercontent.com/44282342/215272198-03420c71-5328-4fc5-8f60-6174ae1b90ef.png)

Springboot 3.0 이상부터는 `Java 17` 이상이 필요하다고 한다. 더해서 ***Springboot 2.x 버전에서도 Java 17***이 잘 동작한다고하니 IntelliJ 설정을 JDK11로 변경해보려 한다.

## Project Structure 변경

SDK를 JDK 17로 변경해준다. (*나는 Oracle 페이지에서 직접 JDK를 다운받아 변경하였다.*)

![image](https://user-images.githubusercontent.com/44282342/215272320-7e75dca5-5a77-4a3c-bcc4-9b2c689d01a4.png)

다른 블로그에서 Gradle JVM 변경에 대한 내용도 있었지만, Project Structure를 변경하니 같이 변경되어 추가로 진행하지는 않았다.

모든 설정을 마치고 **Gradle을 Refresh** 해주면 정상적으로 동작하는 것을 확인할 수 있다.

Spring Initializr 설정에도 최신인 3.x 버전을 권장하는 것으로 보이니 이제는 Java 17을 사용하도록 해야겠다.