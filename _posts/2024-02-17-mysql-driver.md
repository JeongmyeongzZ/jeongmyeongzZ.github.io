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

### ReadOnly 옵션에 따른 동작 방식을 파헤쳐보자

우선 JDBC connection 을 property 로 정의하는 방식은 아래와 같다.

jdbc:(mysql|mariadb):[replication:|loadbalance:|sequential:|aurora:]//<hostDescription>[,<hostDescription>...]/[database][?<key1>=<value1>[&<key2>=<value2>]]

> 1.5.1 버전 이전에는 jdbc:mysql:aurora://클러스터엔드포인트:포트,리더엔드포인트:포트 이런식으로도 사용했으나, 그 뒤로는 클러스터엔드포인트만 명시하는 것을 권고
 
이 중 replication, loadbalance, sequential, aurora 등의 옵션을 maria db connector 에선 HA mode (High availability) 라고 정의하고 있다.

![ha mode](/assets/posts/mariadb/hamode.png)h



_참고 문서_
- [maria db connector](https://mariadb.com/kb/en/about-mariadb-connector-j/)
- [maria db connector github source](https://github.com/mariadb-corporation/mariadb-connector-j)
- [failover](https://mariadb.com/kb/en/failover-and-high-availability-with-mariadb-connector-j/?fbclid=IwAR2EnwLRBGc1T0bQLJTloP9WnisrjM0smV2h4bGa23UcT9Teq55gYVkwctI#primaryreplica-connection-selection)