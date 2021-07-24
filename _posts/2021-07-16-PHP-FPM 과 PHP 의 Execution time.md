---
layout: post
title:  "PHP-FPM 과 PHP 의 Execution time"
date:   2021-07-16 00:40:12
---

## PHP-FPM 과 PHP 의 Execution time

---

### PHP-FPM (PHP FastCGI Process Manager)?

---

CGI(Common Gateway Interface)는 웹서버와 외부 프로그램을 연결해주는 표준화된 프로토콜이다.

서버로 요청이 들어왔을 때 그것이 웹서버가 처리 할 수 없는 정보일 때 그 정보를 처리 할 수 있는 외부 프로그램을 호출해서 그 결과를 웹서버가 받아서 브라우저로 전송하는 역할을 수행하는데, 
쉽게 말해서 `PHP 파일을 웹서버가 처리할 수 있는 HTML 파일로 반환하는 단계` 정도로 이해하면 될 것 같다.

CGI 는 한 Request 당 하나의 프로세스를 생성한다, 이 과정에서 프로세스를 생성하고 삭제하는 수많은 작업이 매번 이루어지는데, 당연히 시간적인 부분에서 낭비가 아닐 수 없다.

이를 개선하기 위해서 만들어진 게 FastCGI 란 녀석이다. 만들어진 프로세스를 재사용하며 Request를 처리한다. 이 부분에서 낭비되는 시간을 많이 절약할 수 있다. 이론상으론 3~30배의 성능 개선 효과가 있다고 얘기한다.

그리고 이 FastCGI 기능을 사용할 수 있게 만들어진 게 바로 FastCGI Manage, FPM 이다.

apache의 경우에는 apache용 php 모듈 (mod-php)이 존재하지만, Nginx 의 경우 이 모듈이 내장돼있지 않기 때문에 보통 Nginx + PHP-FPM 조합으로 PHP 환경을 구성한다.


<br>

### PHP의 실행시간 조절

---

Laravel 등으로 간단하게 Application 을 간단히 올린 후 테스트를 해보자.

_php -i | grep max 등과 같이 확인하면 max_execution_time 이 0 (제한없음) 으로 나오는 현상이 있는데, 이 부분은 cli sapi 와 php-fpm 설정이 달라서 나타나는 현상이다._
_phpinfo(); 를 통해 더 정확한 설정 정보를 알 수 있다는 점 참고하자._


### php.ini max_execution_time

---

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

<br>

### php-fpm.d/www.conf request_terminate_timeout

---

request_terminate_timeout 기본으로 주석처리 돼있으며, php.ini 의 `max_execution_time` 값을 취한다.

process management level 의 설정으로, http 요청 시간, sleep 도 count 되며 이를 설정한 값 보다 php execution time 이 늘어나면 502 Bad Gateway 가 떨어진다.

<br>

### nginx fastcgi_read_timeout

---

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

<br>

_추가 : mysql pdo driver 는 connection timeout attribute 를 지원하지 않는다.. sqlite 는 되는데 ㅠㅠ_

<br>

_참고_

- _https://stackoverflow.com/questions/53040971/handling-execution-time-set-by-php-fpm-and-php-file/53041994_
- _https://blog.lael.be/post/9251_
<br><br><br>
