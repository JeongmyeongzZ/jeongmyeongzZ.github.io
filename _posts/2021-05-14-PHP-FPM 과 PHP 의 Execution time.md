---
layout: post
title:  "PHP-FPM 과 PHP 의 Execution time"
date:   2021-05-14 00:40:12
---

## PHP 의 실행시간 조절

---
Laravel 로 간단하게 Application 을 간단히 올린 후 테스트를 해보자.

_php -i | grep max 등과 같이 확인하면 max_execution_time 이 0 (제한없음) 으로 나오는 현상이 있는데, 이 부분은 cli sapi 와 php-fpm 설정이 달라서 나타나는 현상이다._
_phpinfo(); 를 통해 더 정확한 설정 정보를 알 수 있다는 점 참고하자._


### php.ini max_execution_time 의 영향을 받는 것

```php
Route::get('/', function () {
    sleep(100);
    return view('welcome');
});
```

100초의 sleep 을 준 뒤 페이지를 리프레시 해도, 의도와는 다르게 실제로 100초를 기다린 뒤 view 화면을 띄우는 것을 볼 수 있다.

`sleep` function 은 실제 php script 실행시간을 중단시키는 작업이기에, 실제 execution time 에 영향을 주진 않는 것으로 보인다.



max_input_time

php
max_execution_time

php-fpm
request_terminate_timeout

nginx
fastcgi_read_timeout

502 Bad Gateway 떨어짐 sleep 박으면ㅂ


php-fpm 최대 실행시간
php.ini 에서 설정된 최대실행시간


일단 이건 이 아래 max_input_time과 관련이 있을것으로 보이네요.
로컬에서 sleep(120)걸고 해보니까 60초에 timeou이 나요. 값 바꾸는거 실험 좀 해볼게요
\\\
아래는 좀 더 쉽게 설명하기  위한 다른 예제..
```
<?php
while(true) {}

결과 : Fatal error: Maximum execution time of 30 seconds exceeded in /var/www/html/test.php on line 5

30초 뒤 응답
<?php
sleep(100);

결과 : HTTP 504 gateway timeout
60초 뒤 응답
```

이렇게 php 파일 각각 만들어서 웹으로 호출하면 둘다 타임아웃 나오지만 그 시간과 양상이 다릅니다.
php.ini 에 기본값으로 설정된 max_execution_time (30초) 는 I/O wait 은 안 쳐주는것 같아요.

60초에 죽은건 웹서버가 php-fpm 과의 fastcgi 연결을 끊은거에요. (기본값 60초임)
그래서 고도몰은 카프카에 이벤트 발행하고 응답 기다리는 poll() 메소드의 파라미터가 0이라 nonblocking 이라, CPU 를 열심히 쓰고  max_execution_time 을 다 채워서 오류를 내고
라라벨쪽 프로젝트들은 poll(1) 으로 파라미터를 넘겨 실행해 blocking모드고 I/O wait 가 1ms 이지만 꾸준히 있다보니 max_execution_time 보다 더 길게 실행될수 있었던 것 같아요.

@geongu.hwang  datadog상에서 라라벨 프로젝트들은 왜 5분까지 실행되었는가?
에 대한 답은 아마 php 의 기본 실행 제한시간이 I/O wait 에 의해 무시되는 것 같다는게 제 생각입니다. (다른 파라미터들에 의해 영향을 받아서 더 나중에 timeout이 됨)

그럼 php-fpm (WAS) 에서는 각 프로세스당 요청을 받아서 처리할때 따로 시간제한이 없는가 물으실거 같아서.. 이게 기본값이 무제한 (0) 인 것 맞더라고요.

그래서
<?php
for ($i = 0 ; $i < 10000 ; $++) {
file_put_contents('/tmp/log', "writing log per 1s\n", FILE_APPEND);
sleep(1);
}

이렇게 해보면, nginx가 php-fpm 과 fastcgi 연결을 끊어도 저 요청은 계속 돌아가면서 파일을 쓰고있어요.
이런식으로 요청이 몰려오는데 php-fpm 프로세스들이 계속 오랜시간 (무한일수도 있고..) 돌아가니까 프로세스가 꽉 차고, EC2 인스턴스들이 다 unhealthy 가 되었던게 아닐까 싶어요. (편집됨) 
www.conf.in


추가 : mysql pdo driver 는 connection timeout attribute 를 지원하지 않는다.. sqlite 는 되는데 ㅠㅠ 



<br>

_참고_

- _https://stackoverflow.com/questions/53040971/handling-execution-time-set-by-php-fpm-and-php-file/53041994_
- _https://blog.lael.be/post/9251_
<br><br><br>
