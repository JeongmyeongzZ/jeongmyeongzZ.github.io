---
layout: post
title:  "Laravel Facade 와 Proxy Pattern 의 이해"
date:   2020-10-20 00:40:12
---

Laravel 에서 사용하는 `Facade` 란 용어는 실제 Facade Pattern 과는 거리가 있다. 이는 실제로 Proxy, Locator, Surrogate 등으로 불리는 게 더 정확하다고 한다.

`Facade` 란 단어의 의미는 `실제와는 다른 표면, 허울` 이란 뜻을 가지고 있는데, 라라벨은 실제로 이 의미를 차용한 것 뿐이지 이를 Facade Pattern 이라고 명하고 있진 않다.

## Proxy Pattern

> 실제 기능을 수행하는 객체(Real Object) 대신 가상의 객체(Proxy Object)를 사용해 로직의 흐름을 제어하는 디자인 패턴.

---

Init

```php
some code block
```

<br> 



<br><br><br>
