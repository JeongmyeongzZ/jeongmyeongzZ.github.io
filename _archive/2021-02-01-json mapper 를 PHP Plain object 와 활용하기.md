---
layout: post
title:  "json mapper 를 PHP Plain object 와 활용하기"
date:   2021-02-01 00:40:12
---

## Json mapper

---

[json mapper](https://github.com/cweiske/jsonmapper) 는 json 데이터를 PHP Class 로 매핑해주는 편리한 패키지다.

string, integer, boolean, array 와 같은 simple types 뿐 아니라 object, 다차원 배열 유형도 지원하며 기본적으로 아래와 같이 사용한다.

```php 
$mapper = new JsonMapper();

$contactObject = $mapper->map($jsonContact, new Contact());
```

<br><br>

## 언제 사용할까

---

필자는 기본적인 Laravel 패키지의 MVC 구조에서, Service layer 를 추가해 비즈니스 로직을 담는데, 이 때 컨트롤러와 서비스 레이어 사이의 데이터 전달을 DTO를 통해 하려한다.

이 때 Plain object 의 가장 간단한 사용 사례가 필드를 캡슐화하는 DTO 또는 Entity 이다.

비즈니스 로직은 일반적으로 프로그램, 서비스의 업무에 관련된 모든 처리들을 하는 부분을 뜻하는데, 이 과정에 데이터베이스와의 연결 및 통신, 데이터 가공 등이 포함된다.

쉽게 말해 인터페이스와 데이터베이스 사이의 정보처리를 하는 로직들을 의미한다.

보통 인터페이스 계층에는 HTTP, CLI 등이 포함되는데, 서비스 레이어 (어플리케이션 영역) 에서는 상위의 컨트롤러 (인터페이스 영역)에 대해서는 알 수 없어야 한다.

라라벨 컨트롤러의 `Illuminate\Http\Request` 객체는 `HTTP` 에 의존적이기에 우리는 이 요청들을 담은 데이터 집합을 DTO 로 매핑해 서비스로 전달해주려 한다.

<br><br>

---

필자는 Http Request 객체에 trait 로 해당 코드를 심었다.

artisan 명령어로 생성되는 파일들은 라라벨의 stub 파일들을 기준으로 생성되는데, `php artisan stub:publish` 명령어를 통해 생성되는 stub 파일을 커스터마이징 할 수 있다.

`/stubs` path 에 모든 stub 파일들이 생성되는데, 이 중 request 만 사용할 예정이므로 이외의 파일은 모두 삭제해도 좋다.

```php
<?php

namespace {{ namespace }};

use Illuminate\Foundation\Http\FormRequest;

class {{ class }} extends FormRequest
{
    /**
     * Determine if the user is authorized to make this request.
     *
     * @return bool
     */
    public function authorize()
    {
        return false;
    }

    /**
     * Get the validation rules that apply to the request.
     *
     * @return array
     */
    public function rules()
    {
        return [
            //
        ];
    }
}
```

첫 stub 파일은 위와같은 파일들로 구성 돼 있다. 이제 위 request stub 파일을 아래와 같이 수정해보자.

```php
<?php

namespace {{ namespace }};

use App\Http\Requests\MapHttpRequest;
use Illuminate\Foundation\Http\FormRequest;
use ReflectionException;

class {{ class }} extends FormRequest
{
    use MapHttpRequest;

    /**
     * Determine if the user is authorized to make this request.
     *
     * @return bool
     */
    public function authorize()
    {
        return false;
    }

    /**
     * Get the validation rules that apply to the request.
     *
     * @return array
     */
    public function rules()
    {
        return [
            //
        ];
    }

    /**
     * Map with custom request object class.
     * Change class to request object class name
     *
     * @return mixed|object
     * @throws ReflectionException
     */
    public function toRequestObject()
    {
        return $this->mapHttpRequestToRequestObject($this->json(), );
    }
}
```

이제 `php artisan make:request` 명령어를 통해 생성된 request 객체는 위 stub 파일을 기반으로 생성 될 것 이다.

```php
<?php


namespace App\Http\Requests;


use Illuminate\Http\Request;
use JsonMapper;
use JsonMapper_Exception;
use ReflectionClass;
use ReflectionException;
use RuntimeException;
use Symfony\Component\HttpFoundation\ParameterBag;

trait MapHttpRequests
{
    /**
     * @param ParameterBag $request
     * @param string $className
     * @return mixed|object
     * @throws ReflectionException
     */
    public function mapHttpRequestToRequestObject(ParameterBag $request, string $className)
    {
        try {
            $mapper = new JsonMapper();
            $mapper->bIgnoreVisibility = true;

            return $mapper->map(
                $request,
                (new ReflectionClass($className))->newInstanceWithoutConstructor()
            );
        } catch (JsonMapper_Exception $exception) {
            throw new RuntimeException($exception->getMessage(), $exception->getCode(), $exception);
        }
    }
}
```

<br>

이제 작성한 코드를 기반으로 Http request 를 통해 들어온 데이터를 service layer 로 전달하는 코드를 구현해보자.

```php
<?php

namespace App\Http\Requests;

use App\Http\Requests\MapHttpRequest;
use App\Requests\SomeRequest as RequestObject;
use Illuminate\Foundation\Http\FormRequest;
use ReflectionException;

class SomeRequest extends FormRequest
{
    use MapHttpRequest;

    /**
     * Determine if the user is authorized to make this request.
     *
     * @return bool
     */
    public function authorize()
    {
        return false;
    }

    /**
     * Get the validation rules that apply to the request.
     *
     * @return array
     */
    public function rules()
    {
        return [
            'title' => 'required',
            'body' => 'required',
        ];
    }

    /**
     * Map with custom request object class.
     * Change class to request object class name
     *
     * @return mixed|object
     * @throws ReflectionException
     */
    public function toRequestObject()
    {
        return $this->mapHttpRequestToRequestObject($this->json(), RequestObject::class);
    }
}
```

```php
<?php

declare(strict_types=1);

namespace App\Requests;

class SomeRequest
{
    private string $title;

    private string $body;

    /**
     * @param string $title
     * @param string $body
     */
    public function __construct(string $title, string $body)
    {
        $this->title = $title;
        $this->body = $body;
    }

    /**
     * @return string
     */
    public function getTitle(): string
    {
        return $this->title;
    }

    /**
     * @return string
     */
    public function getBody(): string
    {
        return $this->body;
    }
}


```

```php
<?php

declare(strict_types=1);

namespace App\Http\Controllers;

use Illuminate\Http\Request;
use App\Http\Requests\SomeRequest;
use App\Http\Controllers\Controller;

class SomeController extends Controller
{
    /**
     * @var SomeService
     */
    private SomeService $service;

    /**
     * TrackController constructor.
     * @param SomeService $service
     */
    public function __construct(SomeService $service)
    {
        $this->service = $service;
    }

    /**
     * @param SomeRequest $request
     */
    public function store(SomeRequest $request)
    {
        $this->service->save($request->toRequestObject());
    }
}

```

이렇게 비즈니스 로직을 포함한 서비스 레이어에서 

- `request->all()` 과 같은 필수값 정의와 값의 모호함을 해결해 줄 수 없는 기본 array 형태의 파라미터

- 그리고 Http 영역에 의존적인 Laravel Request 객체

를 사용하는 것이 아닌 Dto 객체를 전달받아 사용하는 방법을 작성해봤다.

많은 양의 데이터가 아니거나, 필수 값 정의등이 필요하지 않을 경우엔 별도의 Dto 객체를 계속해서 생성해줘야 할 지는 프로젝트를 운영하며 정리가 필요할 것 같다.

이도 도메인별로 코드들이 잘 정리돼 있다면 충분히 운영가능할 사이즈가 아닐까 생각이 든다.

<br>

_참고_

- _https://ceobe.dev/laravel-popo-request-for-service-layer/_
- _https://laravel.kr/docs/8.x/artisan#stub-customization_
<br><br><br>
