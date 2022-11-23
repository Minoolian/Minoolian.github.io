---
layout: post
title: "[Kafka] zookeeper is not a recognized option"
category: tech
tags: troubleshooting
---

Springboot에서 Kafka를 이용한 채팅을 테스트해보던 도중 `Kafka topic 생성시` 발생한 오류에 대해 포스팅하려 한다.

* unordered toc
{:toc .large-only}

# 👿 Problem
***

Kafka 토픽 생성을 위한 명령 `kafka-topic.bat --create --zookeeper localhost:2181 --replication-factor 1 --partition 1 --topic kafka-chat` 사용 시 **zookeeper is not recognized option** 오류가 발생한다.

![문제](https://user-images.githubusercontent.com/44282342/204129832-e0a3af32-5dc0-41a2-ae6f-1d0ecb7fa1f3.PNG)


# 👼 Solution
***

![해결](https://user-images.githubusercontent.com/44282342/204130337-623ec425-7bb6-474c-af05-cd3d6ea67b0b.PNG)

Error 메시지 자체에서 zookeeper 옵션 자체를 인식하지 못하는 것으로 보였다. 이 옵션이 deprecate 되었는지 확인해보니, kafka 0.10.0 부터는 ***offset 저장소를 zookeeper 에서 kafka 브로커로*** 옮겼기 때문에 브로커를 찾기위한 `bootstrap-server` 옵션을 사용한다.
