---
layout: post
title:  "Pipeline 으로 DB transaction 구현하기"
date:   2020-10-20 00:40:12
---

반복되는 일련의 작업, Aspect.. etc.. 
 
```php
$pipes = [
            UseDatabaseTransaction::class,
            FirstService::class,
            SecondService::class,
        ];

        $post = app(Pipeline::class)
            ->send($something)
            ->through($pipes)
            ->then(function ($content) {
                return $content;
            });
```


<br><br><br>
