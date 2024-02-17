---
layout: post
title:  "Maria DB(MySQL) Connector Troubleshooting"
date:   2024-02-17 00:40:12
---

## MariaDB Connector?

MariaDB connector 는 Type4 JDBC 드라이버입니다. 

MariaDB 와 MySQL 데이터베이스 서버와 함께 사용하기 위한 경량 JDBC 커넥터

---

## 겪었던 문제

아래가 문제의 코드인데, db 에 데이터를 반영하고 난 뒤 재 조회를 하게 되는 경우 업데이트 되지 않는채로 데이터가 조회 되는 일이 발생했다.

가끔씩은 반영 된 데이터가 조회되기도 한다.

```kotlin
function run () {
    // 조회
    var order = orderService.find()
    
    // 수정
    orderService.updateSomething()
    
    // 재조회
    var updatedOrder = orderService.find()
    
    // 업데이트 되지 않은 정보가 조회된다 ??????
}

class OrderService {
    @Transactional(readOnly = true)
    fun find() {
        // 주문 조회
    }
    
    fun updateSomething() {
        // 수정
    }
}
```

이 문제를 겪기 전과 후에 변경 된 점은 OrderService 에 있는 `@Transactional(readOnly = true)` 구문이다.

### 먼저 JDBC connection 을 확인해보자

우선 JDBC connection 을 property 로 정의하는 방식은 아래와 같다.

jdbc:(mysql|mariadb):[replication:|loadbalance:|sequential:|aurora:]//<hostDescription>[,<hostDescription>...]/[database][?<key1>=<value1>[&<key2>=<value2>]]

> 1.5.1 버전 이전에는 jdbc:mysql:aurora://클러스터엔드포인트:포트,리더엔드포인트:포트 이런식으로도 사용했으나, 그 뒤로는 클러스터엔드포인트만 명시하는 것을 권고
 
![ha mode](/assets/posts/mariadb/hamode.png)

이 중 replication, loadbalance, sequential, aurora 등의 옵션을 maria db connector 에선 HA mode (High availability) 라고 정의하고 있다.

HA Mode 에 따라 Failover 기능을 지원하는데, Aurora 는 Master 의 상태를 보고 Slave 중 하나를 Master 로 승격시키는 방식으로 동작한다.
(그 사이에 Scale 조정 및 tier 확인등을 통한 우선순위 산정 등의 과정이 포함되지만, 생략한다.)

![connection](/assets/posts/mariadb/connection1.png)

문제가 발생하던 시점엔 서비스에서 ha mode 를 aurora 로 준 상태로 실행되고 있었다.

<br>

![Aurora Listener](/assets/posts/mariadb/auroralistener.png)

이 경우 `AuroraListener` 에서 cluster endpoint 에 대한 연결 된 노드들의 endpoint 를 protocol query 로 실행해 가져오는 것을 볼 수 있다.

현재 문제가 재현되는 환경에선 master 한대와 Read replica 한대가 조회 되었다.

이는 실제 해당 데이터베이스에 쿼리를 직접 실행해도 확인할 수 있다.

![Protocol Query](/assets/posts/mariadb/sql.png)

<br>
<br>

### 그래서 ReadOnly 가 왜?

Spring 에서 Transactional 을 처리하는 과정에 Transactional Annotation 에 대한 definition 을 파헤치는 부분이 있다.

![DatasourceUtil](/assets/posts/mariadb/datasourceutil.png)

위 코드와 같이 `DataSourceUtil` 에서 readonly 의 여부에 따라 connection 을 readOnly 속성으로 설정해주는 곳이 있다.

![MasterSlavesListener](/assets/posts/mariadb/switch.png)

이 경우 실제로 `MasterSlavesListener` 에서 connection 을 아까 Aurora protocol query 를 통해 받아온 read replica (slave) 의 connection 으로 전환 하는 것을 볼 수 있다.

<br>

DB Aurora cluster 주소와, aurora HA mode 옵션만으로 readOnly 프로퍼티가 true 인 경우, read replica 로 쿼리가 발생하게 된다.

이 경우 update logic 을 통해 master 에 반영 된 데이터가 read replica 들로 동기화 되기 전에 데이터가 조회 될 수 있다는 뜻이다.


---

Failover 기능은 우리 서비스의 가용성을 위해 꼭 필요한 옵션이다. 

오로라DB 에서 제공하는 failover 기능을 사용하기 위해 db connection 을 `jdbc:mysql:aurora:{cluster endpoint}` 와 같은 형태로 반영한 적이 있었는데, application 상에서 read replica 로 직접적인 쿼리가 발생할 수 있다는 점을 놓쳤다.

무거운 쿼리를 의도적으로 read replica 로 발생시킬 수는 있겠지만, 위와 같이 의도하지 않은 방향으로 서비스가 동작 할 수 있으니, 확인이 꼭 필요하다.

또한, 상위 트랜잭션이 있다면, 하위 트랜잭션이 기본적으로 상위 트랜잭션에 포함되기 때문에 이 문제를 마주쳐보지 못했을 수도 있다. 

중첩 된 트랜잭션에서 하위 트랜잭션을 의도적으로 read replica 로 보내고 싶은 경우가 있다면 `REQUIRES_NEW` 와 같은 옵션으로 전파옵션을 명시하여야 한다. 


_참고 문서_
- [maria db connector](https://mariadb.com/kb/en/about-mariadb-connector-j/)
- [maria db connector github source](https://github.com/mariadb-corporation/mariadb-connector-j)
- [failover](https://mariadb.com/kb/en/failover-and-high-availability-with-mariadb-connector-j/?fbclid=IwAR2EnwLRBGc1T0bQLJTloP9WnisrjM0smV2h4bGa23UcT9Teq55gYVkwctI#primaryreplica-connection-selection)