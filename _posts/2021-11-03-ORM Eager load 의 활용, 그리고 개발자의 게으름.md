---
layout: post
title:  "ORM Eager load 의 활용, 그리고 개발자 게으름"
date:   2021-11-03 00:40:12
---

ORM(Object Relational Mapping)은 DB 를 다루기 편한 방법을 제공한다. 가장 큰 강점 중 하나가 데이터베이스 관계를 쉽게 정의하고 사용할 수 있다는 점이다.

그런데, 아이러니하게도 이는 개발 중 여러 부분에 있어서 개발자를 소홀하고 게으르게 만든다.

DB 를 호출 해 데이터를 가져올 시 로딩 기법(fetch type)에는 Lazy load와 Lazy load가 있다. (기본 수준에서 ORM은 보통 lazy load 관련 모델 데이터)

Lazy Load 는 말 그대로, `게으른`방식이다. 주 테이블 외의 조인할 테이블의 데이터 등은 필요한 경우에만 가져온다.

사용자 마다 하나의 게시글을 가지고 있다고 가정해보자. (One to Many, Laravel 과 Spring JPA 에서 모두 LAZY 방식이 default)

### PHP, Laravel
```php
$users = User::all();

$users->map(function (User $user) {
    return $user->post;
});
```

```
[2020-02-02 06:21:58] local.INFO: select * from `users`  
[2020-02-02 06:22:06] local.INFO: select * from `posts` where `posts`.`user_id` = ? limit 1 [1] 
[2020-02-02 06:22:06] local.INFO: select * from `posts` where `posts`.`user_id` = ? limit 1 [1] 
[2020-02-02 06:22:06] local.INFO: select * from `posts` where `posts`.`user_id` = ? limit 1 [1]
```

### Java, Spring
```java
@Entity
class Post {
    @Id
    private Long id;
    
    @ManyToOne(name="user_id")
    private User user;
}

@Entity
class User {
    @Id
    private Long id;
    
    @OneToMany
    private HashSet<Post> posts;
}

orders.forEach{ order -> order.getPosts() }
```

```
Hibernate:
select
post1_.id as id1_2_1_,
from
post post0_
where
user0_.id=?
```

쿼리 로그를 살펴보면, 세 명의 유저일 때 총 4개의 쿼리가 실행됐다. 이와 같은 상황을 N+1 문제라고 한다. 조회 시에 게시글 정보를 로드하지 않기 때문에 이후 게시글에 접근 할 때마다 새로운 쿼리를 실행시키는 것이다.

이렇게 많은 양의 쿼리가 실행되면, 성능에 당연히 안좋은 영향을 끼친다.

이러한 문제가 발생하지 않게 한 번에 관계된 많은 데이터가 필요할 때는 `Eager load`를 이용할 수 있다.

Laravel Eloquent ORM 은 이 문제를 `with` 메서드로, Spring JPA 에선 `FetchType.EAGER` 명시를 통해 해결 할 수 있다.

```php
$users = User::with('post')->get();

$users->map(function (User $user) {
    return $user->post;
});
```

```
[2020-02-02 06:21:58] local.INFO: select * from `users`  
[2020-02-02 06:22:06] local.INFO: select * from `posts` where user_id in (1, 2, 3)
```

```java
@OneToMany(fetch = FetchType.EAGER)
private HashSet<Post> posts;

...

orders.forEach{ order -> order.getPosts() }
```

```
Hibernate:
select
user0_.id as id1_2_0_,
post1_.id as id1_1_1_,
post1_.user_id as user_id3_1_2_
from
user user0_
left outer join
post post1_
on user0_.id=post1_.user_id
where
user0_.id=?
```

Eager load 시엔 명시한 연관 엔터티에 있는 정보를 한 번에 가져오는 것을 볼 수 있다.

다만 두 ORM 에서의 차이는 있다. Eloquent ORM 에서는 관계간 조회를 위해 where in 쿼리를 활용하고, JPA ORM 에서는 흔히 사용하는 join 을 활용한다.

DBMS 가 발전함에 있어서 where in 과 같은 sub query clause 성능이 많이 개선 되고 있다고 하더라도, join 보다 성능이 떨어지는 게 현실이다. Eloquent ORM 을 사용하면서 무분별한 relation 활용은 eager load 를 사용하더라도 문제가 있어보인다.

ORM 의 편리함이 개발자를 소홀하게 만든다고 했다, Laravel 에선 이런 부분이 쌓이고 쌓이면 상용 환경에 큰 부하가 생길 수 있어 쿼리 튜닝 (join 대체 등)이 사실 필수적이다.

<br>

Spring 은 그럼 join 을 통해 동작하니 소홀해도 괜찮을까? ManyToOne, OneToOne 컬럼의 FetchType 을 LAZY 로 하였을 경우 N+1 문제가 발생하고, OneToMany 는 문제가 없을까?

그렇지 않다. OneToMany 관계를 가진 여러개의 릴레이션이 있다고 생각해보자. 유저가 게시글도, 코멘트도, 여러개의 프로필도 가지고 있을 수 있다. 이럴 땐 모두 각각 쿼리가 호출 되는 문제가 발생한다.

수려하게 join clause 를 이어 붙여 정돈 된, 예쁜 쿼리 하나로 해결하고 싶지 않을까?

```java
@LazyCollection(LazyCollectionOption.FALSE)
@OneToMany
private HashSet<Post> posts;

@LazyCollection(LazyCollectionOption.FALSE)
@OneToMany
private HashSet<Comment> comments;
...

@Query(
    "select user from User user " +
    "join fetch user.posts " +
    "join fetch user.comments " +
    "where user.userId = :userId"
)
User find(@Param(value = "userId") long userId);
```

와 같이 JPQL의 fetch join 을 사용해서 풀 수 있다. Eager load 설정 값을 지우고 LazyCollection, 그리고 Query 를 명시하거나 엔티티그래프, 배치 사이즈 등으로 N+1 문제를 해결할 수 있다.

웬만해서는 소홀해지고 게을러지지 않는 것이 좋겠다. Eager load 도 남발해서는 안된다.

Laravel Eloquent ORM 에선 이런 복잡한 문제를 해결하는 방식보다, 생산성 그리고 적은 코드를 모토로 삼고 있는 것과 같이 where in clause 로 해결한 것 같다.

<br><br>

_참고_: 

- _https://laravel.com/docs/8.x/eloquent-relationships#eager-loading_
- _https://ict-nroo.tistory.com/132_
- _http://jaynewho.com/post/39_

<br><br><br>
