---
layout: post
title:  "Pipeline 으로 DB transaction 구현하기"
date:   2020-10-20 00:40:12
---

반복되는 일련의 작업, Aspect.. etc.. 
 
```php
$this->dispatcher->pipeThrough([
    UseDatabaseTransaction::class,
    FirstService::class,
    SecondService::class,
])->dispatch(new SomeJob(['title' => '제목입니다', 'contents' => '내용입니다']));

$this->job->dispatch(['title' => '제목입니다', 'contents' => '내용입니다']);

$pipes = [
    UseDatabaseTransaction::class,
    FirstService::class,
    SecondService::class,
];

$post = app(Pipeline::class)
    ->send($this->something)
    ->through($pipes)
    ->then(function ($content) {
        return $content;
    });
```


<br><br><br>
