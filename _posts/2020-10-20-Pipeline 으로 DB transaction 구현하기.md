---
layout: post
title:  "Pipeline 으로 DB transaction 구현하기"
date:   2020-10-20 00:40:12
---

## Laravel Pipeline 이해하기

---

라라벨을 사용하면서 파이프라인을 이해하기 가장 좋은 예제는 미들웨어다.

`Illuminate\Foundation\Http\Kernel` 클래스를 보면 아래와 같은 코드를 볼 수 있다.

```php
return (new Pipeline($this->app))
    ->send($request)
    ->through($this->app->shouldSkipMiddleware() ? [] : $this->middleware)
    ->then($this->dispatchToRouter());
```

Http Request 가 미들웨어들을 통과한 다음 라우터에 디스패치 되는 모습이다. `$request` 가 파이프라인 배열안에 담긴 클래스를 차례로 통과하며
Http Request 에 대한 검증 및 데이터 변환을 시켜주는 방식으로 동작한다.
 
<br> 

## Pipeline 을 비즈니스 로직에 담기
 
---

제목, 부제목 등 게시글에 대한 내용이 저장되고, 게시글이 노출 될 여러 카테고리를 지정하고, 그 이후 게시글에 담긴 이미지들이 저장 되는 로직이 있다고 생각해보자.
 
```php

try {
    DB::beginTransaction();
    
    $this->saveEntireArticle->run($data); // save article & categories & images

    DB::commit();
} catch (Exception $e) {
    DB::rollback();
}
```

와 같이 단순하게 트랜잭션을 사용할 수 있을 것 이다. 

일련의 게시글을 저장하는 클래스가 여러 도메인의 비즈니스 로직을 품은 서비스를 호출하고 있다고 생각할 때 우린 파이프라인을 사용해 처리할 수 있다.

먼저 파이프라인으로 사용 될 Pipe interface 를 생성해보자.

```php
interface Pipe
{
    public function handle($data, Closure $next);
}
```

그리고, 일련의 작업들을 처리할 클래스, 트랜잭션을 담당할 클래스를 생성해보자.

```php
class UseDatabaseTransaction implements Pipe
{

    public function handle($data, Closure $next)
    {
        DB::transaction(function () use ($data, $next) {
            return $next($data);
        });
    }
}

class SaveArticleImages implements Pipe
{

    public function handle($data, Closure $next)
    {
        Image::create(..)

        return  $next($content);
    }
}

...
```


```php
class SaveEntireArticle {
    public function run(array $data)
    {
        $pipes = [
            UseDatabaseTransaction::class,
            SaveArticle::class,
            SaveArticleCategories::class,
            SaveArticleImages::class,
        ];
        
        $article = app(Pipeline::class)
            ->send($data)
            ->through($pipes)
            ->then(function ($article) {
                return $article;
            });
    }
}
```

미들웨어 예제를 바탕으로 코드를 이해해보면, 파이프라인에 담겨 실행될 pipe 들에 게시글내용 저장 클래스, 카테고리 저장 클래스, 이미지 저장 클래스들이 담긴다.

그리고, 이를 트랜잭션 처리할 `UseDatabaseTransaction` 클래스를 가장 파이프들의 앞에 위치 시켰다.

물론 `SaveEntireArticle` 를 호출 할 때 pipe 에 트랜잭션이 들어가고, `then` function 내에서 `SaveEntireArticle` 를 호출 하는 것도 동일하게 동작한다. 

<br>

## Job 에서 Pipeline 사용하기

---

위와 같은 클래스를 Job 으로 구현하면 `Dispatcher` 객체가 제공하는 `pipeThrough` 를 이용해 더 쉽게 동작을 구현할 수 있다.

`SaveEntireArticle` 라는 Job 을 dispatch 할 경우도 아래와 같이 처리하거나, 위 코드블럭 예제와 동일하게 Job 내부에 Pipeline 을 구현할 수 있다.

```php
$this->dispatcher->pipeThrough([
    UseDatabaseTransaction::class,
])->dispatch(new SaveEntireArticle($data));
```

<br><br><br>
