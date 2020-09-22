---
layout: post
title:  "SVG 아이콘 Webfont 로 사용하기"
date:   2020-09-20 00:40:12
---

## 웹페이지에서의 아이콘

아이콘들이 존재하지 않는 페이지들은 이제 찾아보기 힘들다. 특히 모바일 및 태블릿과 같이 디바이스 스크린이 한정적인 환경에서는 더더욱 그렇다.

이는 아이콘이 표현하려는 콘텍스트를 텍스트 대신 해줄 뿐 아니라 사용자 액션까지 묘사해줄 수 있는 부분 때문인데, 사이트의 디자인 적 세련감을 줌과 동시에 사용자에게 사이트에 대한 인식 및 사용자의 콘텐츠에 대한 관심을 높이는 등의 효과도 있다.

<br><br>

## SVG

페이지가 포함한 이미지가 많아질 수록 http 요청이 많아지게 되는데, SVG는 크기가 작을 뿐만 아니라 SVG코드를 HTML의 인라인에 포함하여 HTTP 요청을 제거할 수 있어 유연성이 빨라진다. 

또한 이와 같은 이유로 SVG를 적용하면 기존의 아이콘 폰트가 다운로드되고 캐싱되고 렌더링 되기 전 상태의 깜박거림을 더 이상 보지 않는 효과도 볼 수 있다.

그 외 SVG 의 특징, 장점을 정리하자면 대략 이렇다.

- 화질에 영향을 받지 않는 벡터 이미지

- XML 기반의 문서이기에 크기 및 스타일 등의 수정이 쉽다.

- 애니메이션 효과를 CSS3 등을 통해 적용할 수 있다. 

![vector-raster](/assets/posts/svg-webfont/vector-raster.png)

_출처 https://junojunho.com/front-end/svg-icon_

<br><br>

## SVG Web Icon

페이지를 제작하다보면 동일한 아이콘이 여러 컴포넌트등에서 공통적으로 사용하게 되는데, SVG 파일을 보다 효율적으로 색, 크기등을 조정하고 싶을 수 있다.

그리고 이를 효율 적으로 사용할 수 있게 도와준 대표적인 라이브러리가 [폰트어썸](https://fontawesome.com/)이다. 
 
폰트어썸과 같은 아이콘 라이브러리들의 경우 내부를 살펴보면 공통적으로 아래와 같은 모양을 하고 있는데, 이역시 SVG 기반의 이미지를 사용해 폰트화 시켰다는 것을 알 수 있다.

```css
@font-face { 
    font-family: 'FontAwesome'; 
    src: url('../fonts/fontawesome-webfont.eot?v=4.4.0'); 
    src: url('../fonts/fontawesome-webfont.eot?#iefix&v=4.4.0') format('embedded-opentype'), url('../fonts/fontawesome-webfont.woff2?v=4.4.0') format('woff2'), url('../fonts/fontawesome-webfont.woff?v=4.4.0') format('woff'), url('../fonts/fontawesome-webfont.ttf?v=4.4.0') format('truetype'), url('../fonts/fontawesome-webfont.svg?v=4.4.0#fontawesomeregular') format('svg'); 
    font-weight: normal; 
    font-style: normal; 
}
```

이와 같이 아이콘 폰트를 쉽게 만들어주는 여러 플랫폼들이 있는데, 대표적으로 [Icommon](https://icomoon.io/app/#/select), [glyphter](https://glyphter.com/) 등이 있다.

필자는 매번 새로 수정되고, 추가되는 아이콘들을 매번 어떤 플랫폼을 통해 변환하고, 해당 폰트들을 다시 옮기는 번거로운 작업이 아닌, 프로젝트 내에 일련의 경로에 있는 아이콘들을 빌드마다 변환을 자동화 시켜보고 싶었다.
 
<br><br>
 
## Webfont generator

먼저 프로젝트에 해당 패키지를 설치한다.

`npm install --save-dev webfonts-generator`

vue, react, angular 등의 환경의 경우 `/build/webfont-loader.js` 파일을 생성한다. _필자는 라라벨 환경 프로젝트에서 작업했으며, webpack.mix.js 파일에 해당 스크립트를 넣었다._

![bed](/assets/posts/svg-webfont/bed.svg) 해당 파일을 사용해보자.

```js
const webfontsGenerator = require('webfonts-generator');

webfontsGenerator({
    files: [
        'public/images/icons/bed.svg',
    ],
    dest: 'public/fonts/my-icons/', // 만들어질 폰트 경로
    fontName: 'my-icons', // 만들어질 파일명
    html: true, // html 프리뷰 생성
    templateOptions: {
        baseSelector: '.my-icon', // 기본 클래스
        classPrefix: 'my-icon-' // 기본 클래스와 함께 쓰여질 클래스명
    }
}, function(error) {
    // something with error
})
```
이제 생성 된 `my-icons.css` 파일을 살펴보면 아래와 같은 파일이 생성되었을 것이다. 우리는 이제 이를 활용하면 된다.

해당 CSS 파일을 import 해 사용해보도록 하자.

```css
@font-face {
	font-family: "my-icons";
	src: url("my-icons.eot?56e598c3eae167236641b7cbca5cdcc1?#iefix") format("embedded-opentype"),
    url("my-icons.woff2?56e598c3eae167236641b7cbca5cdcc1") format("woff2"),
    url("my-icons.woff?56e598c3eae167236641b7cbca5cdcc1") format("woff");
}

.my-icon {
	line-height: 1;
}

.my-icon:before {
	font-family: my-icons !important;
	font-style: normal;
	font-weight: normal !important;
	vertical-align: top;
}

.my-icon-a_icon:before {
	content: "\f101";
}
```

페이지에 정상적으로 노출되는 것을 확인할 수 있다 !

![bed](/assets/posts/svg-webfont/bed-result.png) 

<br><br><br>
