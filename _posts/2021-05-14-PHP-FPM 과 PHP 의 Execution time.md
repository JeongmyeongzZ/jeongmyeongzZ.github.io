---
layout: post
title:  "PHP-FPM 과 PHP 의 Execution time"
date:   2021-05-14 00:40:12
---

## 

---


php-fpm 최대 실행시간
php.ini 에서 설정된 최대실행시간


일단 이건 이 아래 max_input_time과 관련이 있을것으로 보이네요.
로컬에서 sleep(120)걸고 해보니까 60초에 timeou이 나요. 값 바꾸는거 실험 좀 해볼게요

<br>

_참고_

- _https://stackoverflow.com/questions/53040971/handling-execution-time-set-by-php-fpm-and-php-file/53041994_
<br><br><br>
