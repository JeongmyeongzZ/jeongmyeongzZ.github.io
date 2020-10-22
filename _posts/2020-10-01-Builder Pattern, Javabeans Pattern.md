---
layout: post
title:  "Builder Pattern, 그리고 JavaBeans Pattern"
date:   2020-10-01 00:40:12
---

우리는 종종 객체를 생성할 때, 필수되는 인자와 필수가 아닌 인자를 동시에 받아야 하는 경우가 있다.

게시글을 작성한다고 생각해보자. 제목과 내용은 필수값이지만 부제목, 첨부파일, 출처 등의 여러가지 부가적인 값이 있다.
 
```php

class Article {
   
    private string $title;
    private string $content;
    private ?string $subTitle;
    private ?array $files;
    private ?string $source;
    ...

    public function __construct(
        string $title,
        string $content,
        string $subTitle = null,
        array $files = [],
        string $source = null,
        ...
    ) {
        $this->title = $title;
        $this->content = $content;
        $this->subTitle = $subTitle;
        $this->files = $files;
        $this->source = $source;
        ...
    }
}

```

`$article = new Article('title', 'content', 'subtitle', null, 'source', ...);`

위와 같이 `Article` 객체를 생성할 때, 우리는 수 많은 인자들을 순서에 맞게 넘겨줘야 한다. 중간에 비어있는 값이 들어올 수도, 모든 필수가 아닌 인자들을 넘겨주지 않을 수도 있다. 

인자가 많아질 수록 사용하는 입장에선 실수가 늘어나고, 헷갈릴 수 밖에 없는데, 이는 유지보수면에서 치명적일 수 있다.

<br><br>

## JavaBeans Pattern

---

이를 타개할 방법으로 먼저 자바빈(JavaBeans) 패턴이 있다.

이는 클래스 내에 Getter, Setter 들을 생성하여 필드 값들을 담는 방법인데, 생성자에 수 많은 인자들을 넘겨주지 않아도 된다.

```php

class Article {
    private string $title;
    private string $content;
    private ?string $subTitle;
    private ?array $files;
    private ?string $source;

    public function setTitle(string $title)
    {
        $this->title = $title;
    }

    public function getTitle()
    {
        return $this->title;
    }

    public function setContent(string $content)
    {
        $this->content = $content;
    }

    public function getContent()
    {
        return $this->content;
    }

    ...
}

```

```php

$article = new Article();

$article->setTitle('title');
$article->setContent('content');
...

```

doc, annotation 등을 사용한다면 필수 값을 보다 쉽게 구분 할 수 있고, 사용자는 보다 직관적이고 가독성이 좋은 코드를 마주할 수 있다.

하지만 이에도 문제점이 존재하는데, 코드의 양이 많아지고 `Article` 객체를 setter 함수를 통해 언제든 변경 가능 한 점이라는 부분이다.

_VO(Value Object)_ 라는 객체를 들어 얘기해보면, `VO`는 이름 그대로 값을 담은 객체라는 의미를 가짐과 동시에, read-only (Immutable) 한 객체라는 의미도 함께 지니는데, 위 코드는 이를 만족하지 못 한다.

객체 생성 이후에 setter 를 통해 변경 된 객체는 특히나 값 객체 변경에 대한 추적이 어려워진다.
  
<br><br>
  
## Builder Pattern

---

이를 보완한 패턴이 바로 빌더 패턴이다.

```php
class Article {
    private ArticleBuilder $builder;
    private string $title;
    private string $content;
    private ?string $subTitle;
    ...

    public function __construct(ArticleBuilder $builder)
    {
        $this->builder = $builder;
    
        $this->title = $builder->getTitle();
        $this->content = $builder->getContent();
        $this->subTitle = $builder->getSubTitle();
        ...
    }
}

class ArticleBuilder {
    private string $title;
    private string $content;
    private ?string $subTitle;
    ...

    public function __construct(string $title, string $content)
    {
        $this->title = $title;
        $this->content = $content;
    }

    public function getTitle()
    {
        return $this->title;
    }

    public function getContent()
    {
        return $this->content;
    }

    public function setSubTitle(string $subTitle)
    {
        $this->subTitle = $subTitle;
    }
    
    ...

    public function getSubTitle()
    {
        return $this->subTitle;
    }
    
    ...

    public function build()
    {
        return new Article($this);
    }
}

```

```php

$article = (new ArticleBuilder('title', 'content'))->setSubTitle('sub title ..')->build();

```

필수값은 생성자에 인자로 넘겨받고, 부가적인 값은 빌더의 setter 메서드를 통해 체이닝을 하는 형태이다.

빌더를 통해 `Article` 객체를 생성하면, 필수 값과 부가적인 인자들을 우선 눈에 띄게 구분 시킬 수 있다. `ArticleBuilder` 객체가 생성 될 때마다 `Article` 객체도 새로 생성 되며, `Article` 객체는 빌더의 `build` 메서드로만 넘겨 받을 수 있어, 객체 일관성을 유지할 수 있다.

PHP 상에선 Inner class 사용이 제한 돼 setter 의 사용이 캡슐화를 위해 불가피 하지만, 불변성이 보장받는 값 객체를 이 처럼 생성 할 수 있다.

<br><br><br>
