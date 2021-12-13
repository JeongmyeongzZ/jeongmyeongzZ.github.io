---
layout: post
title:  "Laravel 에서의 Singleton binding 그리고 Scoped singleton binding"
date:   2021-12-10 00:40:12
---

singleton 메소드로 클래스나 인터페이스를 바인딩 하면 컨테이너는 한 번만 해당 의존성을 해결한다. 

싱글톤 바인딩으로 의존성이 해결되면, 컨테이너의 다른 부분에서 호출될 때 동일한 객체 인스턴스가 반환된다.

singleton 이 아닌, bind 또는 별도의 바인딩 명시 없이 리플렉션을 통한 자동 의존성 주입은 각 다른 오브젝트 해시값을 리턴하게 된다.

여기까지가 Laravel 의 가장 기본이 되는, 핵심적인 `Service Container` 에 인스턴스를 바인딩 하는 기본적인 singleton 방식이다.
```php
// AppServiceProvider.php

$this->app->singleton(TestService::class, function () {
    return new TestService();
});
```

```php
// TestService.php

<?php

declare(strict_types=1);

namespace App\Services;

class TestService
{
    public function test()
    {
        echo spl_object_hash($this) . "\n";
    }
}
```

```php
$service = app(TestService::class);

$service->test(); // 000000003cc56d770000000007fa48c5

$service = app(TestService::class);

$service->test(); // 000000003cc56d770000000007fa48c5
```
---

이번 8.47 release 엔 이 Service Container 에 `Binding Scoped Singletons` 라는 기능이 추가됐다.

singleton 과 어느 부분이 다른지 [공식문서](https://laravel.com/docs/8.x/container#binding-scoped) 를 먼저 살펴보자.

> The scoped method binds a class or interface into the container that should only be resolved one time within a given Laravel request / job lifecycle. While this method is similar to the singleton method, instances registered using the scoped method will be flushed whenever the Laravel application starts a new "lifecycle".

> scoped 메서드 바인딩은 Laravel request / job 라이프 사이클에서 한 번만 해결해줄 수 있도록 한다. singleton 메서드와 유사하지만, scoped 메서드를 통해 바인딩 된 인스턴스들은 라라벨 애플리케이션의 새로운 라이프사이클이 시작될 때 flush 된다.

기존 Laravel 기반의 애플리케이션 동작에 익숙한 사람, PHP 기반 애플리케이션을 다루는 사람이라면 의아할 수 있다. Laravel 에선 lifecycle, 즉 request 마다 프로세스가 response 를 반환하기 때문이다.

공식문서 해당 부분 마지막 줄에 `such as when a Laravel Octane worker processes a new request or when a Laravel queue worker processes a new job` 부분을 살펴보자.

`laravel octane 의 워커가 새로운 request 를 처리하거나, 라라벨 큐 워커가 새로운 job 을 처리할 때.` 라고 나와있다.

Laravel Octane 은 Laracon Online 2021 에서 처음 선보였는데, Swoole와 RoadRunner (Reactive 가 한창 핫할 때 들어본 녀석들) 환경을 활용해, 쉽게 말해 자바 기반 애플리케이션 처럼 프로세스가 죽지않고 떠있는 방식으로 고성능을 자랑하는 First-party 패키지이다.

CPU 코어 수를 기반으로 여러 Worker 프로세스를 분기해 애플리케이션을 보다 효율적으로 실행할 수 있는 코루틴(coroutine)을 사용하는데, 위에서 말한 Laravel Octane 의 워커가 이 녀석이다.

워커는 요청과 요청 사이에 활성 상태를 유지하면서 새로운 요청을 대기하며 프레임워크의 startup 시간을 최소화하고, 무거운 HTTP 워크로드가 많은 어플리케이션에 많은 개선 효과를 가져다 준다.

Artisan 명령어로 실행하는 Command 들의 멀티 스레드 활용으로 더 뛰어난 성능을 발휘할 수도 있을 것 같다.

singleton 으로 바인딩 된 인스턴스는 컨테이너에 올라가있지만, Laravel application 의 request/response (one lifecycle) 마다 새로운 인스턴스를 활용해야 할 때가 있을 수 있는데, 그 때 scoped 라는 메서드를 활용하면 된다.

라라벨의 큐는 알다시피 Beanstalk, Amazone SQS, Redis, RDB 와 같은 시스템 간 연동 API 도 제공하는 워커를 포함하고 있다. Octane 과 같이 이 큐에 쌓인 Job 들을 해소할 때도 scoped binding 을 사용할 수 있다.

<br><br>

_참고_
- _https://laravel.com/docs/8.x/container_
- _https://laravel-news.com/laravel-8-47-0_
- _https://silnex.github.io/blog/laravel-octane/_

<br><br><br>
