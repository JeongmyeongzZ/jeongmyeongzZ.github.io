---
layout: post
title:  "SVG 아이콘 Webfont 로 사용하기"
date:   2020-09-20 00:40:12
---

## .editorconfig

---

코드베이스의 프로젝트를 작업하는 사람마다 코딩 스타일은 모두 다르다. 

Java 의 JSR, PHP 의 PSR 과 같은 표준 규격이 있더라도, 표준 규격에 벗어나는 세세한 코드 스타일은 모두 다르기 마련인데, 작업자가 많아질 수록 한 코드베이스에 여러 코드스타일이 혼용되기 쉽다.

IntelliJ 와 같은 IDE 에선 코드들을 포매팅하는 강력한 기능을 지원하고 있는데, 이와 같은 기능으로도 모든 사람마다 IDE 설정이 다르기에 이 문제를 해결하긴 쉽지 않다.

우리는 이러한 문제들을 `.editorconfig` 파일에 코드 스타일을 명시함으로써 해결할 수 있다.

<br><br>


## PHPSTORM (IntelliJ) 에서 .editorconfig 활용하기

---

IDE 환경설정 > Editor > Code Style > PHP > setting (cog icon) > Export > Editorconfig File 을 눌러보면, Editorconfig 파일로 현재 ide 에 적용된 php 코드 스타일을 그대로 가져오는 것을 볼 수 있다.

ij... 와 같이 IntelliJ IDE 에서만 지원하는 프로퍼티들이 있는데, 회사 또는 그룹원들이 모두 같은 툴을 사용하고 있다면 충분히 사용하는데 문제가 없어보인다.

팀원 및 그룹원들간에 소통을 통해 표준 권고를 따르며, 코드 스타일을 맞춘 뒤 코드레벨 `.editorconfig` 파일로 export 해 사용하면 프로젝트 내에 혼선을 줄일 수 있을 것 같다.

추가로 IntelliJ IDE 에선 커밋마다 코드를 리포맷팅 해주는 기능도 제공하고 있으니, 이를 적극 이용하는 것도 하나의 방법이다.

<br><br><br>
