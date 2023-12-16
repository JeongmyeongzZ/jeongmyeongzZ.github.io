---
layout: post
title:  "Jvm Options"
date:   2023-12-16 00:40:12
---


주문접수 jvm option이 날것 그대로군요?!
java -javaagent:/usr/local/bin/dd-java-agent.jar -Ddd.trace.anlytics.enabled=true -Ddd.jdbc.analytics.enabled=true -Ddd.logs.injection=true -Ddd.service.name=order-receipt-api -jar -Dspring.profiles.active=prod -Djava.awt.headless=true -Duser.timezone=Asia/Seoul -XX:InitialRAMPercentage=50 -XX:MaxRAMPercentage=50 /usr/local/bin/cloud-application.jar
MaxGCPauseMillis 나 UseStringDeduplication 같은 것들을 보통 넣기는 하는데, 적용할만한것들 조사가 필요하네요.


$RUN_JVM_PARAM \
-XX:InitialRAMPercentage=50 \
-XX:MaxRAMPercentage=50 \
-XX:MaxGCPauseMillis=50 \
-XX:+UseStringDeduplication \
-XX:+FlightRecorder \


--- 
-XX:InitialRAMPercentage=50

이 옵션은 JVM이 시작될 때 초기 힙 메모리 크기의 백분율을 설정합니다.
예를 들어, 전체 시스템 메모리가 4GB이고 이 옵션이 50으로 설정되어 있다면 초기 힙 크기는 2GB입니다.

-XX:InitialRAMPercentage는 Java Virtual Machine (JVM)이 시작될 때 할당되는 초기 힙 메모리 크기의 백분율을 설정하는 옵션입니다. 이 옵션은 Java 애플리케이션의 초기 힙 크기를 제어하여 성능과 메모리 사용을 조절하는 데 도움을 줍니다.

기본적으로 Java의 HotSpot JVM은 초기 힙 크기를 전체 힙 크기의 1/64로 설정합니다. 그러나 이 값을 조절하여 애플리케이션의 요구 사항에 맞게 최적화할 수 있습니다.

예를 들어, -XX:InitialRAMPercentage=50로 설정한다면 JVM은 전체 힙 메모리의 50%를 초기 힙 크기로 할당합니다. 이렇게 함으로써 애플리케이션이 시작될 때 더 많은 메모리를 사용할 수 있게 됩니다. 이는 초기에 메모리를 더 많이 할당하여 애플리케이션의 초기 부하를 줄일 수 있습니다. 하지만 이로 인해 시스템 전체에서 사용 가능한 메모리가 부족할 수 있으므로 조심해서 사용해야 합니다.

---

JVM이 사용할 수 있는 최대 힙 메모리 크기의 백분율을 설정합니다.
예를 들어, 전체 시스템 메모리가 4GB이고 이 옵션이 50으로 설정되어 있다면 최대 힙 크기는 2GB입니다.

-XX:MaxRAMPercentage는 Java Virtual Machine (JVM)이 사용할 수 있는 최대 힙 메모리 크기의 백분율을 설정하는 옵션입니다. 이 옵션을 사용하면 JVM이 시스템의 전체 메모리 중에서 얼마나 큰 부분을 사용할 수 있는지를 조절할 수 있습니다.

기본적으로 HotSpot JVM은 최대 힙 크기를 전체 힙 크기의 1/4로 설정합니다. 그러나 이 값을 조절하여 애플리케이션의 요구 사항에 맞게 최적화할 수 있습니다

예를 들어, -XX:MaxRAMPercentage=50로 설정한다면 JVM은 전체 시스템 메모리의 50%까지만 최대 힙 크기로 사용할 수 있습니다. 이렇게 함으로써 다른 프로세스나 시스템 서비스에 충분한 메모리를 확보할 수 있습니다. 또한 이 옵션을 통해 JVM이 사용할 수 있는 메모리 양을 제한하여 시스템 리소스를 효율적으로 관리할 수 있습니다.

---

가비지 컬렉션 (Garbage Collection)이 발생할 때 허용되는 최대 일시 중지 시간을 설정합니다.
이 값을 50으로 설정하면 가비지 컬렉션 중지 시간이 50밀리초를 넘지 않도록 노력합니다.

기본값: 이 옵션에는 명시적인 기본값이 없으며, 기본적으로 최대 일시 중지 시간에 대한 목표를 설정하지 않습니다.
설정 및 의미: 이 옵션은 가비지 컬렉션 (GC)이 발생할 때 허용되는 최대 일시 중지 시간을 설정합니다. 값은 밀리초 단위로 지정됩니다. 작은 값으로 설정하면 GC가 더 빨리 완료되도록 하지만, 너무 작게 설정하면 GC가 빈번하게 발생할 수 있습니다. 이 값을 조절하여 애플리케이션의 응답 시간과 GC 성능 사이의 균형을 찾을 수 있습니다.

---

-XX:+UseStringDeduplication

문자열 중복 제거를 활성화합니다.
이 옵션을 사용하면 JVM은 메모리에서 동일한 문자열을 공유하여 중복을 피하려고 시도합니다.
기본값: 일반적으로 이 옵션은 비활성화되어 있습니다.
설정 및 의미: 이 옵션을 활성화하면 JVM은 문자열 중복을 감지하고 중복된 문자열을 공유하여 메모리를 절약합니다. 이는 메모리 사용을 최적화하고 문자열 중복을 효과적으로 제거하여 애플리케이션의 메모리 효율성을 향상시킬 수 있습니다.

---

-XX:+FlightRecorder

Java Flight Recorder (JFR)를 활성화합니다.
JFR는 Java 애플리케이션의 실행 중에 발생하는 이벤트 및 트랜잭션 정보를 수집하고 분석하는 도구입니다. 애플리케이션 성능 프로파일링 및 디버깅에 유용합니다.

기본값: 기본적으로 이 옵션은 비활성화되어 있습니다.
설정 및 의미: 이 옵션을 활성화하면 Java Flight Recorder (JFR)가 활성화됩니다. JFR는 애플리케이션의 실행 중에 발생하는 이벤트 및 트랜잭션 정보를 수집하고 분석하는 도구입니다. 성능 프로파일링, 디버깅, 및 실시간 모니터링을 위해 사용됩니다. 애플리케이션의 성능을 모니터링하고 프로파일링하는 데 도움이 됩니다. 그러나 이 옵션을 활성화하면 약간의 오버헤드가 발생할 수 있으므로 프로덕션 환경에서 사용할 때 주의가 필요합니다.

---

