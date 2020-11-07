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

```php
some code block
```

<br> 



<br><br><br>
