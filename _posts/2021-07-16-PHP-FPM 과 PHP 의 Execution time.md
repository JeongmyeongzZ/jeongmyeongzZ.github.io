---
layout: post
title:  "PHP-FPM 과 PHP 의 Execution time"
date:   2021-07-16 00:40:12
---

## PHP 의 실행시간 조절

---
Laravel 등으로 간단하게 Application 을 간단히 올린 후 테스트를 해보자.

_php -i | grep max 등과 같이 확인하면 max_execution_time 이 0 (제한없음) 으로 나오는 현상이 있는데, 이 부분은 cli sapi 와 php-fpm 설정이 달라서 나타나는 현상이다._
_phpinfo(); 를 통해 더 정확한 설정 정보를 알 수 있다는 점 참고하자._


### php.ini max_execution_time

```php
Route::get('/', function () {
    sleep(100);
    return view('welcome');
});
```

100초의 sleep 을 준 뒤 페이지를 리프레시 해도, 의도와는 다르게 실제로 100초를 기다린 뒤 view 화면을 띄우는 것을 볼 수 있다.

`sleep` function 은 실제 php script 실행시간을 중단시키는 작업이기에, 실제 execution time 에 영향을 주진 않는 것으로 보인다.

http 요청 이후 응답 대기시간도 이와 같은 맥락으로 count 되지 않는다. (I/O wait time)

```php
while (true) {}
```
와 같은 PHP script 실행 시간이 초과될 경우 `Fatal error: Maximum execution time of 30 seconds exceeded in /var/www/html/test.php on line 5` 오류를 반환한다.

---

### php-fpm.d/www.conf request_terminate_timeout

request_terminate_timeout 기본으로 주석처리 돼있으며, php.ini 의 `max_execution_time` 값을 취한다.

process management level 의 설정으로, http 요청 시간, sleep 도 count 되며 이를 설정한 값 보다 php execution time 이 늘어나면 502 Bad Gateway 가 떨어진다.

---

### nginx fastcgi_read_timeout

이는 php-fpm 설정의 `request_terminate_timeout` 와는 조금 다르게 504 Gateway Time-out 이 떨어진다.

해당 값이 최우선시 되며, 내부 스크립트, I/O time 이 길어져도 서버단에서 끊어진다. 

```php
for ($i = 0 ; $i < 10000 ; $i++) {
    file_put_contents('/tmp/log', "writing log per 1s\n", FILE_APPEND);
    sleep(1);
}
```

그러나 이런 식의 PHP script 가 실행되고 있을 때, nginx가 php-fpm 과 fastcgi 연결을 끊어도 저 요청은 계속 돌아가면서 파일을 쓰고 있는 현상을 겪을 수 있으니, Nginx 의 설정만으로 요청을 핸들링 하는 것은 위험할 수 있다.

```
location ~ \.php$ {

// 하위에 설정 돼 있을 경우, php 파일에 한해서만 동작한다는 의미이니, php script 가 아닌 그 앞단(라우팅 등)에서는 정상적으로 동작하지 않는다.
```

---

_추가 : mysql pdo driver 는 connection timeout attribute 를 지원하지 않는다.. sqlite 는 되는데 ㅠㅠ_

<br>

_참고_

- _https://stackoverflow.com/questions/53040971/handling-execution-time-set-by-php-fpm-and-php-file/53041994_
- _https://blog.lael.be/post/9251_
<br><br><br>
