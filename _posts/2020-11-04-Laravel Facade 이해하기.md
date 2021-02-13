---
layout: post
title:  "Laravel Facade 와 Proxy Pattern 의 이해"
date:   2020-10-20 00:40:12
---

Laravel 에서 사용하는 `Facade` 란 용어는 실제 Facade Pattern 과는 거리가 있다. 이는 실제로 Proxy, Locator, Surrogate 등으로 불리는 게 더 정확하다고 한다.

`Facade` 란 단어의 의미는 `실제와는 다른 표면, 허울` 이란 뜻을 가지고 있는데, 이를 이용 해 Facade Pattern 그리고 Laravel 의 Facade 에 대해 이해해보려 한다.

<br>

## Facade Pattern

---

그렇다면 Facade Pattern 이란 무엇인지 간단하게 살펴보자.

Facade Pattern 은 단순히 생각하면, 복잡한 코드를 실행시키는 간단하고 단순한 어댑터를 사용하는 방식이다.

A 라는 클래스 B, C, D, E 등 여러가지의 클래스를는 주입받고 있고 이 네개의 클래스들이 포함하는 여러개의 메서드들을 호출하고 있다고 생각해보자.

B, C, D, E 클래스의 여러 메서드들이 모여서 동작하는 코드를 A 클래스가 적합한 사용 방식에 따라 일련의 메서드에 각 여러 클래스들의 메서드를 담고 있다.

우리는 A 클래스 내부의 복잡성을 고려하고 싶지 않을 수도, 또는 복잡성을 숨기고 싶을 수 있다.

```php

$classA = new A('some data..');

function clientCode(A $classA)
{
    $classA->someMethod();
}

```

그와 같은 경우 우리는 위와 같은 간단한 예제와 같이 A 내부의 서브 클래스들에 의존적하지 않고, 내부 코드 변경 또한 `clientCode` 에 영향을 주지 않는 일종의 인터페이스를 만들어 사용을 할 수 있다.

이제 `Facade` 란 단어와 연결해 생각해보자. 파사드 패턴은 많은 내부구조를 실제와는 다른 표현을 만들어 감싸서 편리한 인터페이스를 제공해 준다고 이해하면 쉬울 것 같다.

<br>

## Proxy Pattern

---

그렇다면 프록시 패턴이란 무엇일까?

> 실제 기능을 수행하는 객체(Real Object) 대신 가상의 객체(Proxy Object)를 사용해 로직의 흐름을 제어하는 디자인 패턴.

보통 데이터를 캐싱할 때 많이 사용되는데, 간단하게 코드와 함께 프록시 패턴을 알아보도록 하자.

```php
interface GetData {
    public function get(int $id);
}

class GetSimpleData implements GetData {
    public function get(int $id)
    {
        // find some data ..
        return 'some data';
    }
}

class GetCacheData implements GetData {
    private GetSimpleData $getSimpleData;
    public function __construct(GetSimpleData $getSimpleData) 
    {
        $this->getSimpleData = $getSimpleData;
    }

    public function get(int $id)
    {
        // if has cached data return cache data
        return 'cached data';
    
        // if has not cached data   
        return $this->getSimpleData->get();
    }
}
```

[Refactoring guru](https://refactoring.guru/design-patterns/proxy/php/example) 에 있는 코드를 참고해 간단한 예제를 만들었다. 위 코드는 인터페이스를 사용해본 PHP 사용자라면 쉽게 이해할 수 있을 것이다.

위와 같이 프록시 패턴을 사용하면, 캐시된 데이터를 확인해 작업을 쉽게 분기처리 할 수 있다.

<br>

## Laravel 의 Facade

---

그렇다면 라라벨에서의 Facade 란 어떤 식으로 동작하는 걸까?

라라벨 파사드는 컨테이너 내부의 서비스를 Static 과 유사한 인터페이스를 제공하는 클래스다. 이런 파사드가 프록시와 같은 역할을 한다고 한다.

Cache Facade 를 통해 살펴보자.

```php
Cache::get('id');
```

위와 같은 코드는 `Illuminate\Support\Facades` 의 `getFacadeAccessor` 메서드를 호출한다.

```php
protected static function getFacadeAccessor()
{
    return 'cache';
}
```

이 메소드는 서비스 컨테이너의 바인딩 이름을 반환하는데, 이 때 컨테이너로 부터 cache 로 이름지어진 바인딩 객체를 찾아 메소드 호출을 요청한다.

이 경우는 파사드로 호출한 get 메서드가 될 것이다. 이 후 정말 `Container` 클래스의 `make` 메서드 내부를 통해 `Illuminate\Cache` 가 리졸빙 되고 get 메서드를 찾아가는 것을 확인할 수 있다.

<br><br>

_참고_

_https://www.brandonsavage.net/lets-talk-about-facades/_

_https://stitcher.io/blog/service-locator-anti-pattern_

proxy pattern ? 그것도 아니다.

Facade is similar to Proxy in that both buffer a complex entity and initialize it on its own. Unlike Facade, Proxy has the same interface as its service object, which makes them interchangeable.

프록시는 원본인 애를 상속하거나, 인터페이스를 통해
사용되서, 동일한 타입 시그니쳐로 사용이 가능한데

The purpose of the Proxy is to add behavior while The purpose of the Facade is to simplify, which may actually involve removing behavior.

프록시의 목적은 기능 추가고
파사드의 목적은 단순화다
오히려 프록시랑 데코레이터면 모를까

<br><br><br>
