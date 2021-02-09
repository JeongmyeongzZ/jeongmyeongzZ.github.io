---
layout: post
title:  "Laravel 에서 Eager load 활용하기"
date:   2021-02-03 00:40:12
---

ORM(Object Relational Mapping)은 DB 를 다루기 편한 방법을 제공한다. 데이터베이스 관계를 쉽게 정의하고 사용할 수 있는데, 이는 개발 중 여러 부분에서 개발자를 소홀하게 만든다.

DB 를 호출 해 데이터를 가져올 시 로딩 기법(fetch type)에는 Lazy load와 Lazy load가 있다.

기본 수준에서 ORM은 lazy load 관련 모델 데이터이며, Laravel도 마찬가지다.

Lazy Load 는 말 그대로, `게으른`방식이다. 주 테이블 외의 조인할 테이블의 데이터 등은 필요한 경우에만 가져온다.

사용자 마다 하나의 게시글을 가지고 있다고 가정해보자.

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

쿼리 로그를 살펴보면, 세 명의 유저일 때 총 4개의 쿼리가 실행됐다. 이와 같은 상황을 N+1 문제라고 한다. 이렇게 많은 양의 쿼리가 실행되면, 성능에 당연히 안좋은 영향을 끼친다.

이러한 문제가 발생하지 않게 한 번에 관계된 많은 데이터가 필요할 때는 `Eager load`를 이용할 수 있다.

Laravel 은 이 문제를 `with` 메서드로 해결 할 수 있다.

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

위와 같이 `with` 메서드를 사용하면 어떤 관계가 `eager load` 되어야 하는 지 지정할 수 있다.

더 복잡한 관계, 여러 관계, 컬럼을 지정하는 등 여러 방법을 제공하고 있으니, [문서](https://laravel.com/docs/8.x/eloquent-relationships#eager-loading) 를 같이 확인해보면 좋을 것 같다.

```php
$books = App\Book::with(['author', 'publisher'])->get();

$books = App\Book::with('author:id,name')->get();

$books = App\Models\Book::with('author.contacts')->get();
```

기본적으로 항상 일부 관계를 eager load 하도록 명시할 수 있는데, 이는 모델에 with field 에 명시해주면 된다.

```php
class Book extends Model
{
    /**
     * The relationships that should always be loaded.
     *
     * @var array
     */
    protected $with = ['author'];
```

일부 상황에서 해당 명시된 관계를 제외하고 로드하고 싶을 땐, `without` 메서드를 사용하면 된다.

주 모델이 이미 조회 된 후에는 `load` 를 이용할 수 있는데, 이도 아래와 같이 사용하면 된다.

```php
$books = App\Models\Book::all();

if ($someCondition) {
    $books->load('author', 'publisher');
}
```

<br><br>

_참고: https://laravel.com/docs/8.x/eloquent-relationships#eager-loading_

<br><br><br>
