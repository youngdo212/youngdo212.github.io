---
layout: post
title: "웹팩과 함께하는 라이브러리 제작"
description: ""
date: 2020-11-30
tags: [webpack]
comments: true
---

# 웹팩과 함께하는 라이브러리 제작

웹용 비디오 플레이어 라이브러리([BetterPlayer](https://github.com/youngdo212/better-player))를 만들면서 번들러로 웹팩을 사용했다. 웹팩으로 웹 페이지 번들링은 해봤지만 라이브러리를 번들링하는 것은 이번이 처음이었고, 추가 환경 설정이 필요했었다. 시행착오를 거치면서 환경 설정하며 알게 된 것들을 남기고자 한다. 두 가지에 대해서 얘기를 나눠보고자 한다.

1. 라이브러리 노출시키기
2. 외부 라이브러리를 제외하여 번들 사이즈 줄이기

Webpack v4.44.2 기준으로 작성하였다.

### 들어가기 앞서...

우리가 모듈 시스템에서 흔히 사용하는 `require()` 함수가 어떻게 구현되어 있는지 잠깐 알아보자. 자바스크립트 파일을 모듈로 구현하고자 할 때 `module.exports` 나 `require()` 함수를 당연하게 쓰고 있지만 이는 사실 Node.js와 같은 자바스크립트 런타임 환경에 사전 정의되어 있는 객체와 함수들이다. 런타임 환경에 따라 구현 방식은 다르겠지만 간략하게 살펴보면 아래와 같은 구조를 가진다.

```javascript
require.cache = Object.create(null);

function require(name) {
  if (!(name in require.cache)) {
    let code = readFile(name);
    let module = {exports: {}};
    require.cache[name] = module;
    let wrapper = Function("require, exports, module", code); // wrapper 정의
    wrapper(require, module.exports, module); // require, module.exports를 사용할 수 있는 이유
  }
  return require.cache[name].exports;
}
```

- readFile 함수는 파일을 불러와 문자열로 반환하는 함수라고 가정하자. 브라우저나 Node.js는 자신만의 파일 접근 함수를 제공한다.

`wrapper` 함수를 정의하는 부분을 보자. require, exports, module의 세 개의 인자를 받는 함수를 정의하고 이를 wrapper라는 이름의 변수에 할당했다. 이제 wrapper 함수를 인자와 함께 호출하면 내가 만든 자바스크립트 파일에서 `require` 함수와 `module` 객체를 별도 정의할 필요 없이 사용할 수 있는 것이다.

어쨋든, 파일을 모듈로 만들어 사용하기 위해서는 `module.exports` 가 필요하다는 사실만 기억하고 이제 본문으로 넘어가자!

## calc 라이브러리를 만들어보자

사칙연산 기능을 제공하는 `calc.js` 라이브러리를 만들어보면서 라이브러리를 위한 웹팩 설정을 하나 하나 알아보자. 시작단계의 웹팩 환경 설정 파일은 다음과 같다

```jsx
// webpack.config.js
const path = require('path');

module.exports = {
  entry: './src/index.js',
  output: {
    filename: 'calc.js',
    path: path.resolve(__dirname, 'dist'),
  },
};
```

## 1) 라이브러리 노출시키기

일반적으로 라이브러리는 다른 어플리케이션에 의해 모듈로 사용된다. 따라서 라이브러리의 index.js는 다음과 같이 export문으로 구성될 것이다.

```javascript
// src/index.js
export function add(a, b) {
  return a + b;
}

export function multiply(a, b) {
  return a * b;
}
```

이제 webpack을 실행해서 번들링을 수행하면 번들 파일이 생성된다. 쉽게 이해하기 위해 번들 파일을 간략하게 만들면 아래와 같다.

```jsx
// dist/calc.js

(function anonymous(modules) {
  var installedModules = {}; // 모듈 캐싱용 객체

  function webpackRequire(moduleId) {

    // 모듈이 캐싱되어 있나 확인한다
    if(installedModules[moduleId]) {
      return installedModules[moduleId].exports;
    }
    var module = installedModules[moduleId] = {
      exports: {}
    };

    modules[moduleId].call(module.exports, module, module.exports, webpackRequire);

    return module.exports; // 우리가 모듈로 만든 add, multiply가 속성으로 들어가있는 객체를 반환
  }

  webpackRequire.define = function(exports, name, getter) {
    exports[name] = getter;
  };
   
  return webpackRequire("./src/index.js");
})({
  './src/index.js': function(module, webpackExports, webpackRequire) {

    webpackRequire.define(webpackExports, "add", function() { return add; });
    webpackRequire.define(webpackExports, "multiply", function() { return multiply; });

    function add(a, b) {
      return a + b;
    }
  
    function multiply(a, b) {
      return a * b;
    }
  }
}) // 즉시 실행 패턴으로 anonymous 함수를 호출한다
```

`anonymous` 함수는 add, multiply 함수를 속성으로 갖는 객체를 반환한다. 하지만 첫 번째 줄을 보면 알 수 있듯이 anonymous가 반환하는 객체를 어느 변수에서도 할당받고 있지 않다. 이 상태의 번들 파일을 그대로 사용하면 어떻게 해도 add, multiply 함수를 사용할 수 없을 것이다.

따라서 웹팩에서는 환경 설정 객체에 output.library 속성을 제공한다. 이 속성은 전역 환경에 노출할 변수 이름을 타낸다.

```jsx
// webpack.config.js
module.exports = {
  ...
  output: {
    ...
    library: 'calc',
  },
};
```

이렇게 환경 설정을 변경하고 다시 번들링 하면 다음의 코드가 추가된다.

```jsx
// dist/calc.js

var calc = (function anonymous(modules) {
...
```

이제, `calc.js` 파일을 브라우저에서 로드하면 전역 환경에서 add, multiply 속성을 가지는 calc라는 이름의 객체를 사용할 수 있다!

```html
<body>
  <script src="dist/calc.js"></script>
  <script>
    console.log(calc.add(1,2)); // 3
  </script>
</body>
```

근데 이 파일은 아직 모듈처럼 사용할 수는 없다. 앞에서 설명했듯이 자바스크립트 파일을 모듈처럼 사용하려면 `module.exports` 키워드가 필요하기 때문이다. `module.exports` 를 사용해야 `require()` 함수를 통해 불러올 수 있다.

이를 위해 웹팩에서는 환경 설정 객체에 output.libraryTarget 속성을 제공한다. 이 속성에 'umd'을 할당하면 전역 환경에서 뿐만 아니라 `require()` 함수를 이용한 모듈로도 사용할 수 있게 된다(libraryTarget의 다른 옵션을 보고 싶다면 [다음](https://webpack.js.org/configuration/output/#outputlibrarytarget)을 참고하자). 이제 웹팩 환경 설정을 수정해보자

```jsx
// webpack.config.js
module.exports = {
  ...
  output: {
    ...
    library: 'calc',
    libraryTarget: 'umd',
  },
};
```

환경 설정을 수정하고 이제 웹팩을 실행하면 다음과 같은 코드가 상단에 추가된다.

```jsx
// dist/calc.js

(function webpackUniversalModuleDefinition(root, factory) {
  if(typeof exports === 'object' && typeof module === 'object')
    module.exports = factory();   // commonjs2 환경
  else if(typeof define === 'function' && define.amd)
    define([], factory);          // amd 환경
  else if(typeof exports === 'object')
    exports["calc"] = factory();  // commonjs 환경
  else
    root["calc"] = factory();     // 전역 변수로 사용하는 경우
})(window, function factory() {
  return /* 여기부터 동일 */(function anonymous(modules) {
    var installedModules = {};
...
```

`webpackUniversalModuleDefinition` 함수가 `calc.js` 파일이 실행되는 환경에 맞춰 add, mulitply 함수를 노출해준다. 이제, 다음과 같이 Node.js 환경에서도 사용할 수 있다(전역 환경에서도 여전히 사용 가능하다).

```jsx
const calc = require("calc.js");

console.log(calc.add(1, 2));
```

### default를 사용하고 있다면?

개별 함수를 export하지 않고 default를 통해 한 번에 제공하고 있다면 아래와 같이 번들링이 된다.

```jsx
// src/index.js
export default {
  add(a, b) {
    return a + b;
  },
  multiply(a, b) {
    return a * b;
  }
}
```

```jsx
// dist/calc.js
const modules = {
  './src/index.js': function(module, webpackExports, webpackRequire) {
    webpackExports["default"] = {
      add(a, b) {
        return a + b;
      },
      multiply(a, b) {
        return a * b;
      },
    };	
  }
}

...
```

add와 multiply 함수가 `module.exports`가 아닌 `module.exports.default`의 속성으로 들어간다. 따라서 사용할 때 `default` 속성을 함께 사용하는 것이 필요하다.

```jsx
const calc = require("calc.js");

console.log(calc.default.add(1, 2));
```

```html
<body>
  <script src="dist/calc.js"></script>
  <script>
    console.log(calc.default.add(1,2)); // 3
  </script>
</body>
```

여간 불편한게 아닐 수 없다. 이전처럼 사용하고 싶다면 웹팩 설정에서 output.libraryExport를 'default'로 설정해주자!

```jsx
// webpack.config.js
module.exports = {
  ...
  output: {
    ...
    library: 'calc',
    libraryTarget: 'umd',
    libraryExport: 'default'
  },
};
```

이 웹팩 환경 설정과 함께 웹팩을 실행하면 다음과 같은 번들 파일이 생성된다.

```jsx
// dist/calc.js

(function webpackUniversalModuleDefinition(root, factory) {
  // ...
})(window, function factory() {
  return (function anonymous(modules) {
    // ...
  })({ "./src/index": function() { /* ... */ } })['default']; // default 속성을 반환한다
});
```

## 2) 의존하는 외부 라이브러리 번들에서 제외하기(feat. externals)

내가 만드는 라이브러리가 또 다른 라이브러리를 의존할 수도 있다. calc 라이브러리에서 lodash를 쓰게 됐다고 가정해보자.

```jsx
// src/index.js
import _ from 'lodash';

export function add(arr) {
  return _.reduce(arr, (acc, el) => acc + el, 0);
}

export function multiply(arr) {
  return _.reduce(arr, (acc, el) => acc * el, 0);
}
```

웹팩은 의존하는 모든 모듈을 전부 번들링한다. 따라서 이 상태에서 웹팩을 통해 번들링을 하게 되면 번들 파일에 lodash가 들어가 파일 크기가 커지게 된다. 번들 크기를 줄이는 방법 중 하나는 lodash를 번들 파일에서 제외하는 것이다. 다만, lodash가 없으면 calc 라이브러리는 작동하지 않기 때문에 calc를 사용하기 위해선 lodash가 이미 설치되어 있어야한다.

- calc를 패키지 형태로 npm을 통해 로컬 환경에서(Node.js) 설치해 사용하는 경우, calc 설치 시 package.json의 의존성에 명시된 lodash가 자동으로 설치된다.
- calc 번들 파일을 그대로 브라우저에서 실행시키는 경우, 브라우저 전역 환경에서 lodash가 있어야만 calc가 제대로 동작한다. 따라서 CDN 등을 통해 사전에 lodash를 브라우저 전역 변수에 할당시켜 놔야한다.

lodash를 번들에서 제외하기 위해선 웹팩의 `externals` 속성을 이용하자. 

```jsx
// webpack.config.js

module.exports = {
  ...
  externals: {
    lodash: '_',
  },
};
```

externals 객체의 속성이름 `lodash` 는 `import _ from 'lodash'` 문의 `lodash` 모듈을 번들링에 제외하라는 뜻이 된다. 제외된 lodash 모듈을 대체하기 위해서 속성값 `'_'` 가 모듈을 찾는데 사용될 것이다. 웹팩으로 번들링 된 아래 번들 파일을 보자.

```jsx
// dist/calc.js

(function webpackUniversalModuleDefinition(root, factory) {
  if(typeof exports === 'object' && typeof module === 'object')
    module.exports = factory(require("_"));  // commonjs2 모듈 환경에서 '_' 모듈을 찾는다
  else if(typeof define === 'function' && define.amd)
    define(["_"], factory);                  // amd 모듈 환경에서 '_' 모듈을 찾는다
  else if(typeof exports === 'object')     
    exports["calc"] = factory(require("_")); // commonjs 모듈 환경에서 '_' 모듈을 찾는다
  else
    root["calc"] = factory(root["_"]);       // window 객체에서 '_' 모듈을 찾는다
})(window, function factory(__WEBPACK_EXTERNAL_MODULE_lodash__) {
  return (function anonymous(modules) {
    // ...
  })({
    "./src/index.js": function() { /* ... */ }
    "lodash": function(module, exports) {
      module.exports = __WEBPACK_EXTERNAL_MODULE_lodash__;
    },
  });
});
```

각 환경에 따라서 `'_'` 이름의 모듈을 찾고 `factory` 함수에 찾은 모듈을 인자로 넣어준다. `__WEBPACK_EXTERNAL_MODULE_lodash__` 가 찾은 모듈이며 lodash의 접근은 전부 이를 통해 이루어 질 것이다. 

만약 모듈 환경마다 다른 이름을 적용하고 싶다면 다음과 같이 사용하자.

```jsx
// webpack.config.js
module.exports = {
  ...
  externals: {
    lodash: {
      commonjs: 'lodash',
      commonjs2: 'lodash',
      amd: 'lodash',
      root: '_',
    },
  },
};
```

다음과 같이 번들링 될 것이다.

```jsx
// dist/calc.js
(function webpackUniversalModuleDefinition(root, factory) {
  if(typeof exports === 'object' && typeof module === 'object')
    module.exports = factory(require("lodash")); // commonjs2 환경에서는 'lodash' 사용
  else if(typeof define === 'function' && define.amd)
    define(["lodash"], factory);                 // amd 환경에서는 'lodash'사용
  else if(typeof exports === 'object')
    exports["calc"] = factory(require("lodash"));// commonjs 환경에서 'lodash' 사용
  else
    root["calc"] = factory(root["_"]);           // 전역 환경에서 '_' 사용
...
```

### Regex를 이용해서 외부 라이브러리 제외하기

라이브러리의 일부 파일만 로드해서 사용하는 경우도 있다. 다음을 보자

```jsx
import reduce from 'lodash/reduce';
import map from 'lodash/map';

export function add(arr) {
  return reduce(arr, (acc, el) => acc + el, 0);
}

export function multiply(arr) {
  return map(arr, (acc, el) => acc * el, 0);
}
```

lodash 모듈을 전부 불러오는 대신 사용하는 하위 모듈만 불러오는 경우이다. 한두개 정도는 다음과 같이 직접 문자열로 제외할 수 있다.

```jsx
// webpack.config.js
module.exports = {
  //...
  externals: [
    'lodash/reduce',
    'lodash/map',
  ],
};
```

하지만 셀 수 없을 정도로 많이 사용하거나 수시로 바뀌는 경우 일일이 추가하는 것이 번거로울 수 있다. 그럴때 정규표현식을 이용해서 작성이 가능하다. 다음을 보자.

```jsx
// webpack.config.js
module.exports = {
  //...
  externals: /^lodash\/.+$/,
};
```

이렇게 정규표현식을 작성하면 `lodash/...` 형태의 모듈 모듈을 전부 제외할 수 있다. 이를 이용해서 번들링 하면 다음과 같은 결과를 얻는다.

```jsx
// dist/calc.js
(function webpackUniversalModuleDefinition(root, factory) {
  if(typeof exports === 'object' && typeof module === 'object')
    module.exports = factory(require("lodash/map"), require("lodash/reduce"));
  else if(typeof define === 'function' && define.amd)
    define(["lodash/map", "lodash/reduce"], factory);
  else if(typeof exports === 'object')
    exports["calc"] = factory(require("lodash/map"), require("lodash/reduce"));
  else
    root["calc"] = factory(root["lodash/map"], root["lodash/reduce"]);
})(window, function(__WEBPACK_EXTERNAL_MODULE_lodash_map__, __WEBPACK_EXTERNAL_MODULE_lodash_reduce__) {
...
```

## 결론

지금까지 라이브러리를 다양한 환경에 노출하는 방법과, 외부 라이브러리 제외를 통해 번들링 사이즈를 줄일 수 있는 방법을 알아보았다. 이 외에도 추가적으로 필요한 부분이 있다면 계속 업데이트 해야겠다.

## 참고자료

[https://webpack.js.org/guides/author-libraries/](https://webpack.js.org/guides/author-libraries/)

[https://webpack.js.org/configuration/externals/](https://webpack.js.org/configuration/externals/)

[https://eloquentjavascript.net/10_modules.html](https://eloquentjavascript.net/10_modules.html)