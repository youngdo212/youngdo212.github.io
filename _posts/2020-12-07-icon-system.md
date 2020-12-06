---
layout: post
title: "아이콘 시스템 구축하기"
description: ""
date: 2020-12-07
tags: [svg, icon, sprites]
comments: true
---

# 목차

- 아이콘 시스템
- 목표 맞는 아이콘 시스템 고르기
- 아이콘 시스템 구축하기(feat. SVG Sprites)

# 아이콘 시스템

웹 비디오 플레이어 라이브러리 [BetterPlayer](https://github.com/youngdo212/better-player/)에 다양한 버튼 아이콘이 필요하게 되면서, 어떤 아이콘 시스템을 구축할 지 선택해야 했다. 찾아본 결과 총 3가지의 대표적인 아이콘 시스템이 존재했다.

1. CSS Sprites
2. Icon Fonts
3. SVG Icons

## CSS Sprites

Sprites란 여러 개의 이미지를 하나의 큰 이미지로 만들어, 여러 번의 네트워크 요청을 하나로 줄여 퍼포먼스적인 이점을 얻는 기술을 뜻한다. Sprites가 없었다면 아이콘의 개수 만큼 HTTP 요청을 하게 된다. 클라이언트에서 서버로 HTTP 요청을 보내는 것은 생각보다 오래 걸리는 작업이고, 브라우저는 동시에 일어날 수 있는 HTTP 요청의 수를 제한하고 있기 때문에, 작은 파일을 여러번 요청하는 시간이 하나의 큰 파일을 요청하는 시간보다 더 걸릴 수 있다. 따라서 아이콘을 Sprites로 만들고 CSS를 이용해 좌표와 크기를 설정해 아이콘 시스템을 구축하는 방법이 CSS Sprites다.

Sprites는 보통 직접 만들지 않고 Grunt나 Gulp 등을 이용해서 자동으로 생성시키는데, 이런 도구를 이용면 Sprites 이미지 파일과 CSS 파일을 제공한다. 예를 들어 spritesmith와 grunt를 이용해서 다음과 같은 Sprites와 CSS 파일을 생성할 수 있다.

![sprites](/images/sample-sprites.png)

```css
.icon-pause {
  background-image: url(sprites.png);
  background-position: -34px 0px;
  width: 32px;
  height: 32px;
}

.icon-play {
  background-image: url(sprites.png);
  background-position: 0px -32px;
  width: 32px;
  height: 32px;
}
```

CSS의 background-image 속성으로 sprites를 불러오고 background-position 속성으로 위치를 정한다. 이제 클래스 이름을 이용해 다음과 같이 사용할 수 있다(`<i>` 태그는 아이콘을 위한 태그는 아니지만 아이콘을 시맨틱하게 나타내기 위해 많이 쓰인다).

```html
<i class="icon-play"></i>
```

## Icon Fonts

Icon fonts란 문자나 숫자 대신 기호로 구성되어 있는 폰트를 뜻한다. 문자가 들어가 있지 않아도 기본적으로 폰트이기 때문에 확대해도 깨지지 않는다는 장점이 있다. 원하는 기호를 나타내기 위해서 각 기호에 대응되는 유니코드나 [합자(ligature)](https://alistapart.com/article/the-era-of-symbol-fonts/)를 이용한다.

Icon fonts를 사용하기 위해서는 1) 폰트를 불러온 뒤 2) 아이콘으로 사용할 엘리먼트의 font-famaily css 속성에 불러온 폰트를 적용시킨 뒤, 3) 아이콘에 대응되는 유니코드(또는 합자)를 적어주면 된다. Material Icons를 한 번 사용해 보자. 아래의 태그를 HTML에 삽입해 폰트를 불러온다.

```html
<link href="https://fonts.googleapis.com/icon?family=Material+Icons" rel="stylesheet">
```

위 태그를 삽입하면 Material Icons 폰트와 함께  `material-icons` class를 사용할 수 있다. `material-icons` class는 다음과 같이 구성되어 있다

```css
.material-icons {
  font-family: 'Material Icons';
  font-weight: normal;
  font-style: normal;
  font-size: 24px;  /* Preferred icon size */
  display: inline-block;
  line-height: 1;
  text-transform: none;
  letter-spacing: normal;
  word-wrap: normal;
  white-space: nowrap;
  direction: ltr;

  /* Support for all WebKit browsers. */
  -webkit-font-smoothing: antialiased;
  /* Support for Safari and Chrome. */
  text-rendering: optimizeLegibility;

  /* Support for Firefox. */
  -moz-osx-font-smoothing: grayscale;

  /* Support for IE. */
  font-feature-settings: 'liga';
}
```

- font-family에 Material Icons 폰트가 적용된 것을 볼 수 있다

이제 이 클래스와 합자(또는 유니코드)를 이용해서 아이콘을 손 쉽게 불러올 수 있다.

```html
<i class="material-icons">face</i> // 합자 이용
<i class="material-icons">&#xE87C;</i> // 유니코드 이용
```

## SVG Icons

SVG Icons란 `<svg>` 태그를 이용한 아이콘을 말한다. SVG(Scalable Vector Graphics)의 이름 그대로, 벡터 그래픽을 사용하고 있기 때문에 icon fonts와 마찬가지로 확대해도 깨지지 않는다.

다양한 사이트에서 무료로 SVG 아이콘을 제공하고 있기 때문에 쉽게 다운받아 사용할 수 있다. [ICOMOON](https://icomoon.io/app/#/select)에서 제공하는 SVG 아이콘을 하나 다운받아 열어 보면 다음과 같이 작성되어 있다.

```html
<svg version="1.1" xmlns="http://www.w3.org/2000/svg" width="32" height="32" viewBox="0 0 32 32">
<title>home</title>
<path d="M32 18.451l-16-12.42-16 12.42v-5.064l16-12.42 16 12.42zM28 18v12h-8v-8h-8v8h-8v-12l12-9z"></path>
</svg>
```

이제 이걸 HTML에 붙여 넣기만 하면 된다.

# 목표에 맞는 아이콘 시스템 고르기

[BetterPlayer](https://github.com/youngdo212/better-player/) 라이브러리는 다양한 웹 사이트에서 사용될 목적으로 개발하고 있다. 웹 사이트마다 비디오 플레이어에 기대하는 기능과 디자인이 다르므로, 여기에 대응하기 위해 커스터마이징이 쉽게 가능해야 한다. 이를 기준으로 각 아이콘 시스템을 다시 살펴보았다.

## CSS Sprites

CSS Sprites는 일단 가장 널리 알려지고 오래 사용된 아이콘 시스템이다. 그만큼 안정성이 보장된다. 단점이 있다면 raster 그래픽 기반이라 확대할 경우 이미지가 깨질 수 있다는 점인데, 아이콘은 확대할 일이 거의 없는 이미지이기 때문에 이 부분은 크게 문제가 되진 않았다. 실제 문제가 되는 부분은 수정 사항이 발생했을 때 이미지를 새로 만들어야 한다는 점이다. 

css로 스타일 변경이 가능한 Icon fonts나 SVG fonts와는 다르게, CSS Sprites는 수정 사항이 발생할 경우 다시 이미지를 만들어야 한다. 심각한 문제는 아니라고 생각하지만 다른 두 아이콘 시스템이 CSS Sprites 만큼의 안정성과 편의성을 제공해 준다면 CSS Sprites를 사용할 이유는 없을 것이라고 생각해서 일단 보류하기로 했다.

## Icon fonts

Icon fonts를 벡터 기반이기 때문에 일단 깨질 염려는 없다. 다만 Icon fonts는 텍스트로 인식되기 때문에 브라우저가 안티 앨리어싱을 적용하게 되는 경우 뚜렷하지 않고 뭉개져서 렌더링이 될 가능성이 있다.

Icon fonts는 css를 이용한 스타일링이 가능하기 때문에 CSS Sprites 보다 좀 더 높은 유연성을 제공하지만, 아이콘의 요소별로 스타일을 적용할 수는 없다. 항상 아이콘 전체의 스타일을 변경해야 한다. 아이콘의 요소마다 각기 다른 스타일을 적용할 수 있는 SVG icons 보다는 유연성이 낮다고 볼 수 있다.

Blocker를 사용하는 사용자의 경우 custom fonts를 블록하는 기능을 사용할 수 있다. 이 경우 Icon fonts를 로드할 수 없어 아이콘이 보이지 않는 문제가 발생한다.

가장 큰 문제는 아이콘을 만들어서 사용하기 쉽지 않다는 점이다. CSS Sprites와 SVG Icons는 이미지만 생성하면 되지만 Icon fonts를 사용해서 커스텀 아이콘을 만들고 싶다면 폰트를 만들어야 한다. 새로운 폰트를 만드는 것은 그리 쉬운 작업이 아니다. 사전에 만들어진 폰트(fontawesome 과 같이)를 사용할 수는 있지만 일부 아이콘을 위해 사용하지 않는 아이콘까지 불러오는 일은 그리 반갑지만은 않을 것이다. 이런 여러가지 단점 때문에 Icon fonts는 고려하지 않도록 하였다.

## SVG Icons

SVG Icons는 상대적으로 최근에 사용되기 시작한 아이콘 시스템이다. SVG Icons는 css를 통해 아이콘의 부분마다 스타일링이 가능하기 때문에 CSS Sprites와 Font icons보다 유연성이 높다. 또한 벡터 기반이라 확대 시 깨질 염려도 없고, 텍스트로 인식되지 않기 때문에 안티 앨리어싱으로 인한 뭉개짐이 발생할 일이 없다. Blocker를 이용해서 svg를 블록하는 일도 일어나지 않는다. 커스텀 아이콘은 이미지로 만들면 되기 때문에 아이콘을 만들어서 사용하기도 쉽다.

다만, ie9 이상부터 지원하기 때문에 브라우저 지원 범위가 셋 중에 가장 떨어지는 편이지만, BetterPlayer의 최소 지원 브라우저는 ie 10 부터이므로 문제될 것은 없었다. 

따라서 SVG Icons를 사용해서 아이콘 시스템을 구축하기로 결정했다.

# 아이콘 시스템 구축하기(feat. SVG Sprites)

비디오 플레이어의 재생 아이콘, 일시 정지 아이콘 등 각 아이콘의 SVG 파일을 준비하고, HTML에 넣어주기만 하면 SVG를 이용한 아이콘은 완성이 된다.

```html
<div class="play-button">
  <svg
    class="icon"
    xmlns="http://www.w3.org/2000/svg"
    viewBox="0 0 32 32"
  >
    <path d="M6 4l20 12-20 12z"></path>
  </svg>
</div>
```

하지만 BetterPlayer에는 커스텀 아이콘도 적용을 할 수 있어야 하기 때문에 그대로 HTML에 넣어 버리면 사용자가 커스텀 아이콘을 적용 할 수가 없다.

이를 해결하기 위해서 `<use>` 태그와 `<symbol>` 태그를 이용하기로 했다. 이 두 태그를 이용하면 다음과 같이 작성이 가능하다.

```html
<svg xmlns="http://www.w3.org/2000/svg">
  <symbol viewBox="0 0 32 32" id="play">
    <path d="M6 4l20 12-20 12z"></path>
  </symbol>
</svg>

// ...

<div class="play-button">
  <svg class="icon">
    <use href="#play"></use>
  </svg>
</div>
```

`<symbol>` 태그는 그래픽 템플릿을 정의할 때 사용하는 엘리먼트다. svg document안에 `<symbol>` 태그와 적절한 id 속성을 이용해 아이콘을 생성하면, `<use>` 태그와 href 속성을 이용해서 해당 아이콘을 복제해올 수 있다. `<use>` 태그와 `<symbol>` 태그가 서로 다른 svg document에 존재해도 잘 작동한다.

엘리먼트에 `<use>` 태그를 이용해 참조할 아이콘의 id만 정의해 놓는다면, 아이콘 구현체를 소스 코드와 분리하는 것이 가능해 진다. 이렇게 되면 커스텀 아이콘을 symbol로 정의해 별도로 불러와 문서 어딘가에 저장만 해놓기만 하면 된다.

## SVG Sprites

아이콘을 개별로 관리하지 말고 CSS Sprites 처럼 하나의 파일로 관리해 보도록 하자. symbol은 다음처럼 한 svg document에 여러 개가 존재할 수 있다.

```html
// icon-sprites.svg
<svg xmlns="http://www.w3.org/2000/svg">
  <symbol viewBox="0 0 32 32" id="play">
    <path d="M6 4l20 12-20 12z"></path>
  </symbol>
  <symbol viewBox="0 0 32 32" id="pause">...</symbol>
  <symbol viewBox="0 0 32 32" id="fullscreen">...</symbol>
  <symbol viewBox="0 0 32 32" id="mute">...</symbol>
</svg>
```

`icon-sprites.svg` 파일을 위처럼 직접 작성하는 것도 가능하지만, 너무 비효율 적이므로 자동으로 svg sprites를 만들어 주는 도구를 이용하자. 다양한 도구가 존재하지만 BetterPlayer에서는 grunt와 grunt-svgstore를 이용했다. `Gruntfile.js` 파일은 다음과 같이 작성했다.

```jsx
module.exports = function (grunt) {
  grunt.initConfig({
    pkg: grunt.file.readJSON('package.json'),
    svgstore: {
      options: {
        prefix: 'better-player-',
      },
      default: {
        files: {
          'dist/better-player.svg': ['src/icons/*.svg'],
        },
      },
    },
  });

  grunt.loadNpmTasks('grunt-svgstore');

  grunt.registerTask('default', ['svgstore']);
};
```

BetterPlayer는 번들러로 webpack을 사용하고 있다. webpack 빌드 시 grunt도 실행될 수 있도록 npm script를 다음과 같이 수정했다.

```jsx
// package.json
{
  // ...
  "scripts": {
    "build": "webpack && grunt",
  },
}
```

이제 `npm run build` 를 통해 빌드를 하면 번들 파일을 생성하고나서 grunt가 `src/icons` 에 있는 모든 svg 파일을 하나의 sprites로 만들어 준다.

## 비동기로 SVG Sprites 불러오기

만약 사용자가 커스텀 아이콘을 사용하고 싶었다면 자신만의 방식을 이용해서 SVG Sprites를 준비해 뒀을 것이다. 준비된 sprites를 사용자 앱의 HTML에 추가해도 되지만, 좀 더 명시적으로 사용하기 위해서 BetterPlayer가 sprites의 삽입을 담당하도록 할 것이다. 사용자는 sprites url을 준비해 다음과 같이 BetterPlayer에 제공할 수 있다.

```jsx
const player = new BetterPlayer({
  iconUrl: 'path/to/sprites.svg',
  // ...
}
```

BetterPlayer는 url을 받아 비동기로 파일을 불러와 문서에 삽입할 것이다. `loadSprite()` 라는 함수를 만들어서 BetterPlayer가 호출할 수 있도록 하자. loadSprites 함수는 다음과 같이 작성한다.

```jsx
function loadSprite(url) {
  const body = document.body;
  const spriteEl = getElementByClassName(
    body,
    'better-player__svg-sprite-wrapper'
  );

  if (!spriteEl) {
    request({ url }).then(response => {
      const wrapper = document.createElement('div');
      wrapper.innerHTML = response;
      wrapper.style.display = 'none';
      wrapper.setAttribute('hidden', '');
      wrapper.className = 'better-player__svg-sprite-wrapper';
      body.insertAdjacentElement('afterbegin', wrapper);
    });
  }
}
```

loadSprite를 호출하면 인자로 넘겨받은 url을 이용해 sprites를 불러오고, 불러온 svg document는 wrapper 엘리먼트로 감사 안보이게 한 뒤 document의 가장 첫 번째 자식으로 추가할 것이다.

물론 사용자가 커스텀 아이콘 url을 지정하지 않는 경우 BetterPlayer의 CDN에서 기본 icon sprites를 불러올 수 있도록 한다. 자, 이렇게 지금까지 커스텀 아이콘을 추가할 수도 있고, css를 이용해 간단하게 아이콘의 스타일도 변경할 수 있는 아이콘 시스템을 만들어 보았다.

# 참고 자료

- [https://css-tricks.com/css-sprites/](https://css-tricks.com/css-sprites/)
- [https://stackoverflow.com/questions/1191961/what-are-the-advantages-of-using-css-sprites-in-web-applications](https://stackoverflow.com/questions/1191961/what-are-the-advantages-of-using-css-sprites-in-web-applications)
- [https://css-tricks.com/examples/IconFont/](https://css-tricks.com/examples/IconFont/)
- [https://alistapart.com/article/the-era-of-symbol-fonts/](https://alistapart.com/article/the-era-of-symbol-fonts/)
- [https://google.github.io/material-design-icons/#icon-font-for-the-web](https://google.github.io/material-design-icons/#icon-font-for-the-web)
- [https://css-tricks.com/icon-fonts-vs-svg/](https://css-tricks.com/icon-fonts-vs-svg/)
- [https://icomoon.io/app/#/select](https://icomoon.io/app/#/select)
- [https://css-tricks.com/svg-sprites-use-better-icon-fonts/](https://css-tricks.com/svg-sprites-use-better-icon-fonts/)