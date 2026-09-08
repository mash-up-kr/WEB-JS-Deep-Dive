
![모던 자바스크립트 딥다이브](https://cdn.inflearn.com/public/courses/327974/cover/3b014384-8b3e-4f66-a4de-a94ffff11f58/Modern%20Javascript%20Deep%20Dive.png?w=736)

   [*모던 자바스크립트 Deep Dive](https://www.yes24.com/Product/Goods/92742567)을 토대로 공부한 것을 정리한 내용으로, 모든 인용문은 모던 자바스크립트 Deep Dive의 문구를 인용한 것입니다.*

# 20장. strict mode

## strict mode란?

```js
function foo() {
  x = 10;
}
foo();

console.log(x); // ??
```

`foo` 함수 내부에는 x 변수의 선언이 없다. 따라서 `x = 10`이 실행되면 자바스크립트 엔진은 x 변수가 어디서 선언됐는지 스코프 체인을 따라 검색하기 시작한다.  
먼저 foo 함수의 스코프에서 x 변수의 선언을 검색한다. 하지만 foo 함수의 스코프에는 x 변수의 선언이 없으므로 검색에 실패할 것이고, 그다음 상위 스코프(예제에서는 전역 스코프)에서 x 변수의 선언을 검색한다. 전역 스코프에도 x 변수의 선언은 존재하지 않는다.

여기서 자바스크립트 엔진이 `ReferenceError`를 발생시킬 것 같지만, 놀랍게도 자바스크립트 엔진은 암묵적으로 전역 객체에 x 프로퍼티를 동적 생성한다. ㅋㅋ 이때 전역 객체의 x 프로퍼티는 마치 전역 변수처럼 사용할 수 있는데, 이러한 현상을 **암묵적 전역**이라 한다.

=&gt; 즉, 개발자의 의도와는 상관없이 발생한 암묵적 전역은 오류를 발생시키는 원인이 될 가능성이 크다. 그래서 반드시 `var`, `let`, `const` 키워드를 사용하여 변수를 선언한 다음 사용해야 한다.

실수를 줄여 안정적인 코드를 생산하기 위해 ES5부터 **strict mode(엄격 모드)**가 추가되었다.

> `strict mode` 자바스크립트 언어의 문법을 좀 더 엄격히 적용하여 오류를 발생시킬 가능성이 높거나 자바스크립트 엔진의 최적화 작업에 문제를 일으킬 수 있는 코드에 대해 명시적인 에러를 발생시킨다.

ESLint 같은 린트 도구를 사용해도 strict mode와 유사한 효과를 얻을 수 있다. 린트 도구는 정적 분석 기능을 통해 소스코드를 실행하기 전에 소스코드를 스캔하여 문법적 오류만이 아니라 잠재적 오류까지 찾아내고 오류의 원인을 리포팅해 주는 유용한 도구다.

린트 도구는 강제성 있는 코딩 컨벤션까지 정의하고 강제할 수 있어서, 실무에서는 strict mode보다 린트 도구를 사용하는 편이 더 선호되기도 한다.

<details class="orca-details">
<summary>strict mode랑 ESLint는 결국 비슷한 목적인데, 실무에서는 뭘 쓰는 게 맞을까?</summary>

둘은 목적이 겹치지만 **동작하는 시점과 방식이 다르다.** strict mode는 자바스크립트 엔진이 **런타임**에 강제하는 언어 차원의 규칙이고, ESLint는 코드를 실행하기 전 **정적 분석** 단계에서 잡아주는 별도의 도구다.

실무에서는 대체로 **둘 다** 켜 있는 상태에 가깝다. 최신 프로젝트는 대부분 ES 모듈(`import`/`export`)이나 `class`를 쓰는데, 이 둘은 뒤에서 설명하듯 **자동으로 strict mode가 적용**되기 때문에 개발자가 신경 쓰지 않아도 이미 strict mode 위에서 돌아간다. 그 위에 ESLint를 얹어서 `no-undef`, `no-unused-vars` 같은 규칙으로 strict mode가 잡지 못하는 스타일/잠재 버그까지 커버하는 식이다.

=&gt; 즉 "strict mode냐 ESLint냐"의 양자택일이 아니라, **엔진이 강제하는 최소한의 안전벨트(strict mode) + 팀 규칙까지 강제하는 정적 분석기(ESLint)**를 함께 쓴다고 이해하면 된다.

</details>



## strict mode의 적용

strict mode를 적용하려면 전역의 선두 또는 함수 몸체의 선두에 `'use strict';`를 추가한다.

**전역에 적용한 strict mode**는 스크립트 전체에 적용된다.

```js
'use strict';

function foo() {
  x = 10; // ReferenceError: x is not defined
}
foo();
```

**함수 몸체의 선두에 추가**하면 해당 함수와 중첩 함수에 strict mode가 적용된다.

```js
function foo() {
  'use strict';

  x = 10; // ReferenceError: x is not defined
}
foo();
```

주의할 점은, `'use strict';`는 반드시 **코드의 선두**에 위치시켜야 한다는 것이다. 함수 몸체의 선두가 아니라 중간에 위치시키면 제대로 동작하지 않는다.

```js
function foo() {
  x = 10; // 에러를 발생시키지 않는다
  'use strict';
}
foo();
```

코드의 선두에 위치시키지 않은 `'use strict';`는 단순한 문자열 취급을 받고 무시되기 때문에 아무런 효력이 없다.



## 전역에 strict mode를 적용하는 것은 피하자

전역에 적용한 strict mode는 스크립트 단위로 적용된다.

```html
<!DOCTYPE html>
<html>
<body>
  <script>
    'use strict';
  </script>
  <script>
    x = 1; // 에러가 발생하지 않는다
    console.log(x); // 1
  </script>
  <script>
    'use strict';
    y = 1; // ReferenceError: y is not defined
    console.log(y);
  </script>
</body>
</html>
```

위 예제와 같이 strict mode 스크립트와 non-strict mode 스크립트를 혼용하는 것은 오류를 발생시킬 수 있다. 특히 외부 서드파티 라이브러리를 사용하는 경우 라이브러리가 non-strict mode인 경우도 있기 때문에, 전역에 strict mode를 적용하는 것은 바람직하지 않다.

이러한 경우 **즉시 실행 함수로 스크립트 전체를 감싸서 스코프를 구분**하고, 즉시 실행 함수의 선두에 strict mode를 적용한다.

```js
// 즉시 실행 함수의 선두에 strict mode 적용
(function () {
  'use strict';

  // Do something...
}());
```



## 함수 단위로 strict mode를 적용하는 것도 피하자

앞서 함수 단위로도 strict mode를 적용할 수 있지만 함수 단위로 strict mode를 적용하는 것도 피해야 한다.

어떤 함수는 strict mode를 적용하고 어떤 함수는 strict mode를 적용하지 않는 것은 바람직하지 않으며, 모든 함수에 일일이 strict mode를 적용하는 것은 번거롭다. 그리고 strict mode가 적용된 함수가 참조할 함수 외부의 컨텍스트에 strict mode를 적용하지 않는다면 문제가 발생할 수 있다.

```js
(function () {
  // non-strict mode
  var let = 10; // 에러가 발생하지 않는다

  function foo() {
    'use strict';

    let = 20; // SyntaxError: Unexpected strict mode reserved word
  }
  foo();
}());
```

=&gt; 따라서 strict mode는 즉시 실행 함수로 감싼 스크립트 단위로 적용하는 것이 바람직하다.



## strict mode가 발생시키는 에러

### 암묵적 전역

선언하지 않은 변수를 참조하면 `ReferenceError`가 발생한다.

```js
(function () {
  'use strict';

  x = 1;
  console.log(x); // ReferenceError: x is not defined
}());
```

### 변수, 함수, 매개변수의 삭제

`delete` 연산자로 변수, 함수, 매개변수를 삭제하면 `SyntaxError`가 발생한다.

```js
(function () {
  'use strict';

  var x = 1;
  delete x;
  // SyntaxError: Delete of an unqualified identifier in strict mode.

  function foo(a) {
    delete a;
    // SyntaxError: Delete of an unqualified identifier in strict mode.
  }
  delete foo;
  // SyntaxError: Delete of an unqualified identifier in strict mode.
}());
```

### 매개변수 이름의 중복

중복된 매개변수 이름을 사용하면 `SyntaxError`가 발생한다.

```js
(function () {
  'use strict';

  // SyntaxError: Duplicate parameter name not allowed in this context
  function foo(x, x) {
    return x + x;
  }
  console.log(foo(1, 2));
}());
```

### with 문의 사용

`with` 문을 사용하면 `SyntaxError`가 발생한다.

`with` 문은 전달된 객체를 스코프 체인에 추가한다. `with` 문은 동일한 객체의 프로퍼티를 반복해서 사용할 때 객체 이름을 생략할 수 있어서 코드가 간단해지는 효과가 있지만, 성능과 가독성이 나빠지는 문제가 있다. 따라서 `with` 문은 사용하지 않는 것이 좋다.

```js
(function () {
  'use strict';

  // SyntaxError: Strict mode code may not include a with statement
  with({ x: 1 }) {
    console.log(x);
  }
}());
```



## strict mode 적용에 의한 변화

### 일반 함수의 this

strict mode에서 함수를 **일반 함수로서 호출**하면 `this`에 `undefined`가 바인딩된다. 생성자 함수가 아닌 일반 함수 내부에서는 this를 사용할 필요가 없기 때문이다. 이때 에러는 발생하지 않는다.

```js
(function () {
  'use strict';

  function foo() {
    console.log(this); // undefined
  }
  foo();

  function Foo() {
    console.log(this); // Foo
  }
  new Foo();
}());
```

non-strict mode에서는 일반 함수로서 호출한 함수 내부의 this에 전역 객체(브라우저 환경에서는 window)가 바인딩된다.

<details class="orca-details">
<summary>일반 함수의 this가 undefined가 되는 게 왜 개선이라고 하는 걸까?</summary>

non-strict mode에서는 일반 함수를 그냥 호출했을 때 this가 전역 객체(`window`)를 가리킨다. 문제는 이게 **버그를 조용히 숨긴다**는 점이다.

예를 들어 생성자 함수를 `new` 없이 실수로 그냥 호출하면, non-strict mode에서는 this가 window가 되고, 함수 안에서 `this.name = ...` 같은 코드가 실행되면서 **전역 객체를 오염시킨다.** 에러도 안 나기 때문에 개발자는 뭔가 잘못됐다는 사실조차 모른 채 넘어간다.

```js
function User(name) {
  this.name = name; // new 없이 호출하면 window.name = name 이 됨
}
const u = User('kim'); // 앗, new를 빠뜨림
// non-strict: 에러 없음, window.name이 오염됨, u는 undefined
// strict:     this가 undefined라서 TypeError가 즉시 발생 → 실수를 바로 알아챔
```

=&gt; 즉 strict mode에서 this가 `undefined`가 되는 건 "불편하게 만든" 게 아니라, **위험한 실수를 조용히 넘기지 않고 즉시 에러로 드러나게** 해주는 안전장치인 셈이다.

</details>

### arguments 객체

strict mode에서는 매개변수에 전달된 인수를 재할당하여 변경해도 `arguments` 객체에 반영되지 않는다.

```js
(function (a) {
  'use strict';
  // 매개변수에 전달된 인수를 재할당하여 변경
  a = 2;

  // 변경된 인수가 arguments 객체에 반영되지 않는다
  console.log(arguments); // { 0: 1, length: 1 }
}(1));
```

non-strict mode에서는 매개변수를 재할당하면 arguments 객체에도 그 값이 반영되기 때문에, 매개변수와 arguments 객체가 서로 연동된다. strict mode는 이 연동을 끊어서 arguments 객체가 **함수 호출 시점의 원본 인수**를 그대로 유지하도록 만든다.




<details class="orca-details">
<summary>요즘도 'use strict'를 직접 써야 할까?</summary>

지금까지 strict mode를 켜는 법과 그로 인한 변화를 살펴봤지만, 사실 현대 자바스크립트를 작성한다면 `'use strict';`를 직접 타이핑할 일은 거의 없다.

- **ES6 모듈(ESM)** — `import`/`export`를 사용하는 모듈이나 `type="module"`로 로드되는 스크립트 코드는 **언제나 자동으로 strict mode**로 실행된다.
- **class** — 클래스 몸체 내부의 모든 코드도 **암묵적으로 strict mode**가 적용된다.

=&gt; 즉 번들러(Webpack, Vite 등)와 모듈 시스템을 사용하는 요즘 프로젝트는 별도로 선언하지 않아도 이미 대부분의 코드가 strict mode 위에서 돌아가고 있는 것이다.

</details>

<details class="orca-details">
<summary>그럼 strict mode를 몰라도 되는 건가? 왜 굳이 공부하는 걸까?</summary>

**자동으로 켜져 있기 때문에** 오히려 그 규칙을 알아야 한다.

모듈이나 class 안에서 코드를 짜다가 `this`가 갑자기 `undefined`로 나오거나, `delete`가 `SyntaxError`를 뱉거나, 암묵적 전역이 `ReferenceError`로 막히는 경험을 하게 되는데, 이 동작들이 전부 strict mode의 규칙에서 나온다. **왜 이런 에러가 나는지**를 이해하려면 결국 20장에서 배운 내용을 알아야 하는 것이다.

또 하나, 레거시 코드나 오래된 서드파티 스크립트를 다룰 때는 여전히 non-strict mode를 마주하게 된다. 이때 두 모드의 차이(암묵적 전역, this 바인딩, arguments 연동 등)를 알고 있어야 "이 코드가 왜 여기서만 다르게 동작하지?"를 설명할 수 있다.

=&gt; 결론적으로 strict mode는 **직접 켜는 스위치라기보다, 이미 켜져 있는 환경의 규칙집**에 가깝다. 그래서 오히려 더 알아둘 가치가 있다.

</details>

<details class="orca-details">
<summary>브라우저 호환성은 걱정 안 해도 될까?</summary>

strict mode는 ES5부터 도입된 표준이라 **IE10 이상을 포함한 현대 브라우저는 모두 지원**한다. 다만 아주 오래된 **IE9 이하**는 strict mode를 지원하지 않는데, 이 경우 `'use strict';`를 그냥 무시하고 문자열로 취급한다.

=&gt; 즉 지원하지 않는 브라우저라고 해서 에러가 나는 게 아니라, strict mode가 "적용되지 않은 채로 실행"될 뿐이다. 요즘은 IE 자체가 지원 종료됐기 때문에 실무에서 호환성을 걱정할 일은 사실상 없다고 봐도 된다.

</details>


</br>

# 21장. 빌트인 객체

## 자바스크립트 객체의 분류

자바스크립트의 객체는 크게 3개로 분류할 수 있다.

- `표준 빌트인 객체(standard built-in objects)` ECMAScript 사양에 정의된 객체. 애플리케이션 전역의 공통 기능을 제공한다. 자바스크립트 실행 환경(브라우저 또는 Node.js)과 관계없이 **언제나 사용**할 수 있다.
- `호스트 객체(host objects)` ECMAScript 사양에는 정의되어 있지 않지만, 자바스크립트 실행 환경(브라우저 또는 Node.js)에서 **추가로 제공**하는 객체.
- `사용자 정의 객체(user-defined objects)` 표준 빌트인 객체와 호스트 객체처럼 기본 제공되는 객체가 아닌, 사용자가 직접 정의한 객체.

브라우저 환경에서는 DOM, BOM, Canvas, XMLHttpRequest, fetch, requestAnimationFrame, SVG, Web Storage, Web Component, Web Worker 같은 클라이언트 사이드 Web API가 호스트 객체로 제공되고, Node.js 환경에서는 Node.js 고유의 API가 호스트 객체로 제공된다.



## 표준 빌트인 객체

자바스크립트는 Object, String, Number, Boolean, Symbol, Date, Math, RegExp, Array, Map/Set, WeakMap/WeakSet, Function, Promise, Reflect, Proxy, JSON, Error 등 40여 개의 표준 빌트인 객체를 제공한다.

이 중 Math, Reflect, JSON을 **제외**한 표준 빌트인 객체는 모두 인스턴스를 생성할 수 있는 **생성자 함수 객체**다.

=&gt; 즉, 생성자 함수 객체인 표준 빌트인 객체는 **프로토타입 메서드와 정적 메서드**를 제공하고, 생성자 함수 객체가 아닌 표준 빌트인 객체(Math, Reflect, JSON)는 **정적 메서드만** 제공한다.

```js
// String 생성자 함수에 의한 String 객체 생성
const strObj = new String('Lee'); // String {"Lee"}
console.log(typeof strObj); // object

// Number 생성자 함수에 의한 Number 객체 생성
const numObj = new Number(1.5); // Number {1.5}
console.log(typeof numObj); // object
```

표준 빌트인 객체인 Number의 프로토타입 메서드는 Number 객체의 인스턴스가, 정적 메서드는 Number 객체가 직접 호출한다.

```js
const numObj = new Number(1.5); // Number {1.5}

// toFixed는 Number.prototype의 프로토타입 메서드
console.log(numObj.toFixed()); // 2

// isInteger는 Number의 정적 메서드
console.log(Number.isInteger(0.5)); // false
```

생성자 함수인 표준 빌트인 객체가 생성한 인스턴스의 프로토타입은, 생성자 함수의 `prototype` 프로퍼티에 바인딩된 객체다.



## 원시값과 래퍼 객체

문자열이나 숫자, 불리언 등의 원시값이 있는데도 문자열, 숫자, 불리언 객체를 생성하는 String, Number, Boolean 등의 표준 빌트인 생성자 함수가 존재하는 이유는 무엇일까?

```js
const str = 'hello';

// 원시 타입인 문자열이 프로퍼티와 메서드를 갖고 있는 객체처럼 동작한다.
console.log(str.length); // 5
console.log(str.toUpperCase()); // HELLO
```

원시값은 객체가 아니므로 프로퍼티나 메서드를 가질 수 없는데도, 원시값인 문자열이 마치 객체처럼 동작한다.

이는 원시값인 문자열, 숫자, 불리언 값의 경우, 이들 원시값에 대해 마치 객체처럼 마침표 표기법(또는 대괄호 표기법)으로 접근하면 자바스크립트 엔진이 **일시적으로 원시값을 연관된 객체로 변환**해 주기 때문이다.

=&gt; 이처럼 문자열, 숫자, 불리언 값에 대해 객체처럼 접근하면 생성되는 임시 객체를 **래퍼 객체**라 한다.



1. 문자열에 대해 마침표 표기법으로 접근하면 그 순간 래퍼 객체인 String 생성자 함수의 인스턴스가 생성되고, 문자열은 래퍼 객체의 `[[StringData]]` 내부 슬롯에 할당된다.
2. 이때 문자열 래퍼 객체인 String 인스턴스는 String.prototype의 메서드를 상속받아 사용할 수 있다.
3. 그 후, 래퍼 객체의 처리가 종료되면 래퍼 객체의 `[[StringData]]` 내부 슬롯에 할당된 원시값으로 원래의 상태를 되돌리고, 래퍼 객체는 **가비지 컬렉션의 대상**이 된다.



문자열, 숫자, 불리언, 심벌은 이렇게 래퍼 객체를 통해 객체처럼 사용할 수 있으므로, 굳이 `new String('...')` 처럼 명시적으로 객체를 생성할 필요가 없다.

한편 `null`과 `undefined`는 래퍼 객체를 생성하지 않는다.

=&gt; 따라서 null과 undefined 값을 객체처럼 사용하면 에러가 발생한다.

<details class="orca-details">
<summary>그럼 new String('Lee')처럼 명시적으로 래퍼 객체를 만드는 건 언제 쓰나?</summary>

결론부터 말하면 **거의 쓸 일이 없고, 오히려 쓰지 않는 게 권장된다.**

`new String('Lee')`, `new Number(1)`, `new Boolean(true)` 처럼 명시적으로 래퍼 객체를 만들면 원시값이 아니라 **객체**가 되기 때문에 예상치 못한 결과를 만든다. 대표적으로 `typeof`가 `'object'`가 되고, `new Boolean(false)`는 객체이므로 조건문에서 **truthy**로 평가된다.

```js
const b = new Boolean(false);
if (b) {
  console.log('실행됨!'); // 객체는 언제나 truthy라서 실행된다
}
```

=&gt; 즉 원시값이 필요하면 그냥 리터럴(`'Lee'`, `1`, `true`)을 쓰면 되고, `String()`, `Number()`처럼 `new` 없이 함수로 호출해서 타입 변환에만 쓰는 게 일반적이다. `new`를 붙여 래퍼 객체를 직접 만드는 건 사실상 안티패턴에 가깝다.

</details>



## 전역 객체

전역 객체(global object)는 코드가 실행되기 이전 단계에, 자바스크립트 엔진에 의해 **어떤 객체보다도 먼저 생성되는 특수한 객체**이며, 어떤 객체에도 속하지 않은 **최상위 객체**다.

전역 객체는 자바스크립트 환경에 따라 지칭하는 이름이 제각각이다.

- 브라우저 환경: `window`(또는 `self`, `this`, `frames`)
- Node.js 환경: `global`

이렇게 환경마다 전역 객체를 가리키는 식별자가 달라 혼란스러웠는데, ES11(ECMAScript 2020)에서 도입된 `globalThis`는 브라우저 환경과 Node.js 환경에서 전역 객체를 가리키던 다양한 식별자를 **통일한 식별자**다.

```js
// 브라우저 환경
globalThis === this; // true
globalThis === window; // true

// Node.js 환경 (12.0.0 이상)
globalThis === global; // true
```

**전역 객체의 특징**을 정리하면 다음과 같다.

- 전역 객체는 개발자가 의도적으로 생성할 수 없다. 즉, 전역 객체를 생성할 수 있는 생성자 함수가 제공되지 않는다.
- 전역 객체의 프로퍼티를 참조할 때 `window`(또는 `global`)를 생략할 수 있다.
- 전역 객체는 표준 빌트인 객체(Object, String, Number, Function, Array 등)를 프로퍼티로 가지고 있다.
- 자바스크립트 실행 환경에 따라 추가적으로 프로퍼티와 메서드를 갖는다. (브라우저의 Web API, Node.js의 호스트 API 등)
- var 키워드로 선언한 전역 변수와 선언하지 않은 변수에 값을 할당한 암묵적 전역, 그리고 전역 함수는 전역 객체의 프로퍼티가 된다.
- 하지만 `let`이나 `const` 키워드로 선언한 전역 변수는 전역 객체의 프로퍼티가 아니다. 즉, `window.foo`와 같이 접근할 수 없다. (let, const로 선언한 전역 변수는 보이지 않는 개념적인 블록, 즉 전역 렉시컬 환경의 선언적 환경 레코드 내에 존재하게 된다.)



## 빌트인 전역 프로퍼티(built-in global property)

빌트인 전역 프로퍼티는 전역 객체의 프로퍼티를 의미한다. 주로 애플리케이션 전역에서 사용하는 값을 제공한다.

**Infinity**  
Infinity 프로퍼티는 무한대를 나타내는 숫자값 Infinity를 갖는다.

```js
console.log(3 / 0); // Infinity
console.log(-3 / 0); // -Infinity
console.log(typeof Infinity); // number
```

**NaN**  
NaN 프로퍼티는 숫자가 아님(Not-a-Number)을 나타내는 숫자값 NaN을 갖는다. NaN 프로퍼티는 `Number.NaN` 프로퍼티와 같다.

```js
console.log(Number('xyz')); // NaN
console.log(1 * 'string'); // NaN
console.log(typeof NaN); // number
```

**undefined**  
undefined 프로퍼티는 원시 타입 undefined를 값으로 갖는다.

```js
var foo;
console.log(foo); // undefined
console.log(undefined); // undefined
```



## 빌트인 전역 함수(built-in global function)

빌트인 전역 함수는 애플리케이션 전역에서 호출할 수 있는 빌트인 함수로서 전역 객체의 메서드다.

**eval**  
eval 함수는 자바스크립트 코드를 나타내는 문자열을 인수로 전달받는다. 전달받은 문자열 코드가 표현식이라면 런타임에 **평가하여 값을 생성**하고, 표현식이 아닌 문이라면 런타임에 **실행**한다.

```js
// 표현식인 문
eval('1 + 2;'); // 3
// 표현식이 아닌 문
eval('var x = 5;'); // undefined

// eval 함수에 의해 런타임에 변수 선언문이 실행되어 x 변수가 선언되었다.
console.log(x); // 5
```

eval 함수는 자신이 호출된 위치에 해당하는 기존의 스코프를 런타임에 동적으로 수정한다. 하지만 eval 함수는 **보안에 매우 취약**하고, 자바스크립트 엔진에 의해 최적화가 수행되지 않아 처리 속도가 느리다.

=&gt; 그러므로 eval 함수의 사용은 **금지**해야 한다.

**isFinite**  
전달받은 인수가 정상적인 유한수인지 검사하여, 유한수이면 true를 반환하고, 무한수이면 false를 반환한다. 인수의 타입이 숫자가 아닌 경우 숫자로 타입을 변환한 후 검사를 수행한다. 이때 인수가 NaN으로 평가되면 false를 반환한다.

```js
isFinite(0); // true
isFinite(2e64); // true
isFinite('10'); // true (‘10’ → 10)
isFinite(null); // true (null → 0)

isFinite(Infinity); // false
isFinite(NaN); // false
isFinite('Hello'); // false
```

`isFinite(null)`이 true인 이유는, null을 숫자로 변환하면 0이 되기 때문이다.

**isNaN**  
전달받은 인수가 NaN인지 검사하여 그 결과를 불리언 타입으로 반환한다. 인수의 타입이 숫자가 아닌 경우 숫자로 타입을 변환한 후 검사를 수행한다.

```js
isNaN(NaN); // true
isNaN(10); // false

isNaN('blabla'); // true (‘blabla’ → NaN)
isNaN('10'); // false (‘10’ → 10)
isNaN(''); // false (‘’ → 0)

isNaN(true); // false (true → 1)
isNaN(null); // false (null → 0)
isNaN(undefined); // true (undefined → NaN)
```

**parseFloat**  
전달받은 문자열 인수를 부동 소수점 숫자(floating point number), 즉 실수로 해석하여 반환한다.

```js
parseFloat('3.14'); // 3.14
parseFloat('10.00'); // 10

// 공백으로 구분된 문자열은 첫 번째 문자열만 변환한다.
parseFloat('34 45 66'); // 34
parseFloat('40 years'); // 40

// 첫 번째 문자열을 숫자로 변환할 수 없다면 NaN을 반환한다.
parseFloat('He was 40'); // NaN
```

**parseInt**  
전달받은 문자열 인수를 정수(integer)로 해석하여 반환한다. 두 번째 인수로 **진법을 나타내는 기수(2~36)**를 전달할 수 있다.

```js
parseInt('10'); // 10
parseInt('10.123'); // 10

// 두 번째 인수로 기수(2진수)를 전달 → 2진수로 해석하여 10진수 정수로 반환
parseInt('10', 2); // 2
parseInt('10', 16); // 16

// 기수를 지정한 숫자를 해당 기수의 문자열로 변환하려면 Number.prototype.toString 사용
const x = 15;
x.toString(2); // '1111'
parseInt(x.toString(2), 2); // 15
```

**encodeURI / decodeURI**  
encodeURI 함수는 완전한 URI 문자열로 전달받아 이스케이프 처리를 위해 인코딩한다. decodeURI 함수는 인코딩된 URI를 인수로 전달받아 이스케이프 처리 이전으로 디코딩한다.

> `이스케이프 처리` 네트워크를 통해 정보를 공유할 때, 어떤 시스템에서도 읽을 수 있는 아스키 문자 셋으로 변환하는 것

```js
const uri = 'http://example.com?name=이웅모&job=programmer&teacher';

const enc = encodeURI(uri);
console.log(enc);
// http://example.com?name=%EC%9D%B4%EC%9B%85%EB%AA%A8&job=programmer&teacher

const dec = decodeURI(enc);
console.log(dec);
// http://example.com?name=이웅모&job=programmer&teacher
```

**encodeURIComponent / decodeURIComponent**  
encodeURIComponent 함수는 URI 구성 요소(component)를 인수로 전달받아 인코딩한다. encodeURI 함수와의 차이는, `**=`, `?`, `&` 등의 문자까지 인코딩**한다는 점이다.

- `encodeURIComponent`: 인수로 전달된 문자열을 **URI의 구성 요소인 쿼리 스트링의 일부**로 간주 =&gt; `=`, `?`, `&`까지 인코딩
- `encodeURI`: 인수로 전달된 문자열을 **완전한 URI 전체**라고 간주 =&gt; `=`, `?`, `&`는 인코딩하지 않음

```js
const uriComp = 'name=이웅모&job=programmer&teacher';

// encodeURIComponent: 쿼리 스트링 구분자까지 인코딩
let enc = encodeURIComponent(uriComp);
console.log(enc);
// name%3D%EC%9D%B4%EC%9B%85%EB%AA%A8%26job%3Dprogrammer%26teacher

// encodeURI: 쿼리 스트링 구분자는 인코딩하지 않음
enc = encodeURI(uriComp);
console.log(enc);
// name=%EC%9D%B4%EC%9B%85%EB%AA%A8&job=programmer&teacher
```

<details class="orca-details">
<summary>encodeURI랑 encodeURIComponent 기준이 뭘까?</summary>

**URI 전체를 인코딩하느냐, URI의 한 조각(구성 요소)만 인코딩하느냐**

- `encodeURI`는 "이미 완성된 URI 전체"를 넘길 때 쓴다. 그래서 `?`, `&`, `=`, `/`, `#` 같은 **URI의 구분자 역할을 하는 문자는 인코딩하지 않고 그대로 둔다.** 이걸 인코딩해버리면 URI 구조 자체가 깨지기 때문이다.
- `encodeURIComponent`는 "쿼리 스트링의 값 하나" 같은 **조각**을 넘길 때 쓴다. 그래서 `?`, `&`, `=`까지 전부 인코딩한다. 값 안에 `&`나 `=`가 들어가도 파라미터 구분이 꼬이지 않게 하기 위해서다.

실무 팁으로는, **쿼리 파라미터의 값을 조립할 때는 거의 항상 `encodeURIComponent`를 쓴다.**

```js
const name = '이웅모&teacher'; // 값 안에 &가 들어있음
const url = `https://example.com?name=${encodeURIComponent(name)}`;
// name=%EC%9D%B4%EC%9B%85%EB%AA%A8%26teacher  → 파라미터가 안 깨짐
```

=&gt; 정리하면, **URL을 통째로 넘기면 `encodeURI`, 파라미터 값 하나를 끼워 넣으면 `encodeURIComponent`.** 다만 요즘은 `URLSearchParams`나 `URL` 객체가 이런 인코딩을 알아서 처리해주기 때문에, 직접 문자열을 조립할 때가 아니라면 그쪽을 쓰는 게 더 안전하다.

</details>



## 암묵적 전역(implicit global)

```js
var x = 10; // 전역 변수

function foo() {
  // 선언하지 않은 식별자에 값을 할당
  y = 20; // window.y = 20;
}
foo();

// 선언하지 않은 식별자 y를 전역에서 참조할 수 있다.
console.log(x + y); // 30
```

foo 함수 내의 `y = 20`이 실행되면, 자바스크립트 엔진은 y 변수에 값을 할당하기 위해 먼저 스코프 체인을 통해 선언된 변수인지 확인한다. 이때 어디에서도 y 변수의 선언을 찾을 수 없으므로 참조 에러가 발생해야 하지만, 자바스크립트 엔진은 `y = 20`을 `window.y = 20`으로 해석하여 전역 객체에 프로퍼티를 **동적 생성**한다. 결국 y는 전역 객체의 프로퍼티가 되어 마치 전역 변수처럼 동작한다.

=&gt; 이러한 현상을 **암묵적 전역**이라 한다.

하지만 y는 변수 선언 없이 단지 전역 객체의 프로퍼티로 추가되었을 뿐, **변수가 아니다.** 따라서 y는 변수가 아니므로 **변수 호이스팅이 발생하지 않는다.**

```js
// 전역 변수 x는 호이스팅이 발생한다.
console.log(x); // undefined
// 전역 객체의 프로퍼티인 y는 호이스팅이 발생하지 않는다.
console.log(y); // ReferenceError: y is not defined

var x = 10; // 전역 변수

function foo() {
  y = 20; // window.y = 20;
}
foo();
```

변수가 아니라 단지 프로퍼티인 y는 `delete` 연산자로 삭제할 수 있다. 반면 전역 변수는 프로퍼티이지만 delete 연산자로 삭제할 수 없다.

```js
var x = 10; // 전역 변수

function foo() {
  y = 20; // 암묵적 전역 (window.y = 20)
  console.log(x + y); // 30
}
foo();

console.log(window.x); // 10
console.log(window.y); // 20

delete x; // 전역 변수는 삭제되지 않는다.
delete y; // 프로퍼티는 삭제된다.

console.log(window.x); // 10
console.log(window.y); // undefined
```

=&gt; 이처럼 암묵적 전역은 의도치 않은 버그의 온상이 되므로, 앞서 20장에서 배운 것처럼 **strict mode를 적용**하여 방지하는 것이 좋다.


</br>

# 22장. this

## this 키워드

객체는 상태를 나타내는 **프로퍼티**와, 동작을 나타내는 **메서드**를 하나의 논리적인 단위로 묶은 복합적인 자료구조다. 동작을 나타내는 메서드는 자신이 속한 객체의 상태, 즉 프로퍼티를 참조하고 변경할 수 있어야 한다.

이때 메서드가 자신이 속한 객체의 프로퍼티를 참조하려면, 먼저 자신이 속한 객체를 가리키는 식별자를 참조할 수 있어야 한다.

```js
const circle = {
  radius: 5, // 프로퍼티: 객체 고유의 상태 데이터
  getDiameter() { // 메서드: 상태 데이터를 참조하고 조작하는 동작
    // 자신이 속한 객체인 circle을 참조할 수 있어야 한다.
    return 2 * circle.radius;
  },
};

console.log(circle.getDiameter()); // 10
```

위처럼 객체 리터럴 방식으로 생성한 객체의 경우, 메서드 내부에서 자신이 속한 객체를 가리키는 식별자 `circle`을 재귀적으로 참조할 수 있다. 하지만 이것은 좋지 않다. 객체 리터럴은 자신을 할당하기 이전이라 식별자를 참조할 수 없기 때문에 우연히 동작할 뿐이다.



생성자 함수 방식으로 인스턴스를 생성하는 경우를 보면 더 명확해진다.

```js
function Circle(radius) {
  // 이 시점에는 생성자 함수 자신이 생성할 인스턴스를 가리키는 식별자를 알 수 없다.
  ????.radius = radius;
}

Circle.prototype.getDiameter = function () {
  return 2 * ????.radius;
};

// 생성자 함수로 인스턴스를 생성하려면 먼저 생성자 함수를 정의해야 한다.
const circle = new Circle(5);
```

생성자 함수를 정의하는 시점에는 아직 인스턴스를 생성하기 이전이므로, 생성자 함수가 생성할 인스턴스를 가리키는 식별자를 알 수 없다.

=&gt; 따라서 자신이 속한 객체 또는 자신이 생성할 인스턴스를 가리키는 특수한 식별자가 필요한데, 이를 위해 자바스크립트는 `this`라는 특수한 식별자를 제공한다.

> `this` 자신이 속한 객체 또는 자신이 생성할 인스턴스를 가리키는 자기 참조 변수(self-referencing variable). this를 통해 자신이 속한 객체 또는 자신이 생성할 인스턴스의 프로퍼티나 메서드를 참조할 수 있다.

this는 자바스크립트 엔진에 의해 암묵적으로 생성되며, 코드 어디서든 참조할 수 있다.

=&gt; **this가 가리키는 값, 즉 this 바인딩은 함수 호출 방식에 의해 동적으로 결정된다.**

```js
// 객체 리터럴
const circle = {
  radius: 5,
  getDiameter() {
    // this는 메서드를 호출한 객체를 가리킨다.
    return 2 * this.radius;
  },
};
console.log(circle.getDiameter()); // 10
```

```js
// 생성자 함수
function Circle(radius) {
  // this는 생성자 함수가 생성할 인스턴스를 가리킨다.
  this.radius = radius;
}
Circle.prototype.getDiameter = function () {
  return 2 * this.radius;
};
const circle = new Circle(5);
console.log(circle.getDiameter()); // 10
```

`this`는 상황에 따라 가리키는 대상이 다르다.

- 전역에서의 this =&gt; 전역 객체 window
- 일반 함수 내부의 this =&gt; 전역 객체 window (strict mode에서는 undefined)
- 메서드 내부의 this =&gt; 메서드를 호출한 객체
- 생성자 함수 내부의 this =&gt; 생성자 함수가 생성할 인스턴스

<details class="orca-details">
<summary>다른 언어의 this랑 자바스크립트의 this는 뭐가 다른 걸까?</summary>

Java나 C++ 같은 클래스 기반 언어에서 `this`는 **언제나 클래스가 생성하는 인스턴스 자기 자신**을 가리킨다. 코드를 어디서 어떻게 호출하든 this가 무엇을 가리킬지는 컴파일 시점에 정해져 있어서 헷갈릴 일이 거의 없다.

반면 자바스크립트의 this는 **함수가 어떻게 호출되었는지(호출 방식)에 따라 런타임에 동적으로 결정된다.** 똑같은 함수라도 일반 함수로 부르면 전역 객체를, 메서드로 부르면 그 객체를, `new`와 함께 부르면 새 인스턴스를 가리킨다.

=&gt; 그래서 자바스크립트에서 this를 이해하는 핵심은 "이 함수가 **어디에 정의**되었나"가 아니라 "이 함수가 **어떻게 호출**되었나"를 보는 것이다. 이 관점만 잡으면 뒤에 나오는 4가지 바인딩 규칙이 훨씬 쉽게 읽힌다.

</details>



## 함수 호출 방식과 this 바인딩

### 일반 함수 호출

기본적으로 this에는 전역 객체(global object)가 바인딩된다.

```js
function foo() {
  console.log("foo's this: ", this); // window
  function bar() {
    console.log("bar's this: ", this); // window
  }
  bar();
}
foo();
```

이처럼 **일반 함수로 호출하면 함수 내부의 this에는 전역 객체가 바인딩된다.** 다만 this는 객체의 프로퍼티나 메서드를 참조하기 위한 자기 참조 변수이므로, 객체를 생성하지 않는 일반 함수에서 this는 사실 의미가 없다. 그래서 strict mode가 적용된 일반 함수 내부의 this에는 undefined가 바인딩된다.

문제는 **중첩 함수나 콜백 함수도 일반 함수로 호출되면 그 내부의 this에는 전역 객체가 바인딩된다**는 점이다. 이것은 메서드 내부에서도 마찬가지다.

```js
var value = 1;

const obj = {
  value: 100,
  foo() {
    console.log("foo's this: ", this); // { value: 100, foo: ƒ }
    console.log("foo's this.value: ", this.value); // 100

    // 메서드 내에서 정의한 중첩 함수
    function bar() {
      console.log("bar's this: ", this); // window
      console.log("bar's this.value: ", this.value); // 1
    }
    // 중첩 함수 bar를 일반 함수로 호출하면 내부의 this에는 전역 객체가 바인딩된다.
    bar();
  },
};

obj.foo();
```

콜백 함수의 경우도 동일하다.

```js
var value = 1;

const obj = {
  value: 100,
  foo() {
    console.log("foo's this: ", this); // { value: 100, foo: ƒ }
    // 콜백 함수 내부의 this에는 전역 객체가 바인딩된다.
    setTimeout(function () {
      console.log("callback's this: ", this); // window
      console.log("callback's this.value: ", this.value); // 1
    }, 100);
  },
};

obj.foo();
```

이처럼 일반 함수로 호출된 모든 함수(중첩 함수, 콜백 함수 포함) 내부의 this에는 전역 객체가 바인딩된다.

=&gt; 그런데 외부 함수인 메서드와 그 내부의 중첩 함수(또는 콜백 함수)의 this가 서로 다른 값을 가리키는 것은, 헬퍼 함수로 동작하기 어렵게 만드는 문제가 된다.

메서드 내부의 중첩 함수나 콜백 함수의 this 바인딩을 메서드의 this 바인딩과 일치시키기 위한 방법은 다음과 같다.

**① this를 변수(that)에 할당**

```js
var value = 1;

const obj = {
  value: 100,
  foo() {
    // this 바인딩(obj)을 변수 that에 할당한다.
    const that = this;

    setTimeout(function () {
      console.log(that.value); // 100
    }, 100);
  },
};
obj.foo();
```

**② Function.prototype.bind 메서드 사용**

```js
var value = 1;

const obj = {
  value: 100,
  foo() {
    // 콜백 함수에 명시적으로 this를 바인딩한다.
    setTimeout(function () {
      console.log(this.value); // 100
    }.bind(this), 100);
  },
};
obj.foo();
```

**③ 화살표 함수 사용 (권장)**

```js
var value = 1;

const obj = {
  value: 100,
  foo() {
    // 화살표 함수 내부의 this는 상위 스코프의 this를 가리킨다.
    setTimeout(() => console.log(this.value), 100); // 100
  },
};
obj.foo();
```

화살표 함수는 자체적인 this 바인딩을 갖지 않고, 상위 스코프의 this를 그대로 참조한다.



### 메서드 호출

메서드 내부의 this에는 메서드를 소유한 객체가 아니라, **메서드를 호출한 객체**, 즉 메서드를 호출할 때 메서드 이름 앞의 마침표(`.`) 연산자 앞에 기술한 객체가 바인딩된다.

```js
const person = {
  name: 'Lee',
  getName() {
    // this는 메서드를 호출한 객체에 바인딩된다.
    return this.name;
  },
};

// 메서드 getName을 호출한 객체는 person이다.
console.log(person.getName()); // Lee
```

주의할 점은 getName 메서드는 person 객체의 메서드로 **정의**된 것이 아니라, 독립적으로 존재하는 별도의 함수 객체이고 person 객체의 getName 프로퍼티가 이 함수 객체를 **가리키고 있을 뿐**이라는 것이다.

=&gt; 따라서 getName 프로퍼티가 가리키는 함수 객체는 다른 객체의 메서드가 될 수도 있고, 일반 변수에 할당하여 일반 함수로 호출될 수도 있다.

```js
const anotherPerson = {
  name: 'Kim',
};
// getName 메서드를 anotherPerson 객체의 메서드로 할당
anotherPerson.getName = person.getName;

// getName 메서드를 호출한 객체는 anotherPerson이다.
console.log(anotherPerson.getName()); // Kim

// getName 메서드를 변수에 할당
const getName = person.getName;

// getName 메서드를 일반 함수로 호출 (this는 전역 객체 window)
// 브라우저에서 window.name은 창의 이름을 나타내는 빌트인 프로퍼티로 기본값이 ''이다.
console.log(getName()); // ''
```

이처럼 메서드 내부의 this는 메서드를 소유한 객체와는 관계없이, **메서드를 호출한 객체에 바인딩**된다. 프로토타입 메서드 내부에서 사용된 this도 일반 메서드와 마찬가지로 해당 메서드를 호출한 객체에 바인딩된다.

```js
function Person(name) {
  this.name = name;
}
Person.prototype.getName = function () {
  return this.name;
};

const me = new Person('Lee');
// getName 메서드를 호출한 객체는 me이다.
console.log(me.getName()); // ① Lee

Person.prototype.name = 'Kim';
// getName 메서드를 호출한 객체는 Person.prototype이다.
console.log(Person.prototype.getName()); // ② Kim
```



### 생성자 함수 호출

생성자 함수 내부의 this에는 생성자 함수가 (미래에) 생성할 인스턴스가 바인딩된다.

```js
// 생성자 함수
function Circle(radius) {
  // this는 생성자 함수가 생성할 인스턴스를 가리킨다.
  this.radius = radius;
  this.getDiameter = function () {
    return 2 * this.radius;
  };
}

// 반지름이 5인 Circle 객체를 생성
const circle1 = new Circle(5);
// 반지름이 10인 Circle 객체를 생성
const circle2 = new Circle(10);

console.log(circle1.getDiameter()); // 10
console.log(circle2.getDiameter()); // 20
```

생성자 함수는 이름 그대로 객체(인스턴스)를 생성하는 함수다. 다만 일반 함수와 동일한 방법으로 정의하고, `new` 연산자와 함께 호출하면 해당 함수는 생성자 함수로 동작한다. 만약 new 연산자와 함께 호출하지 않으면 생성자 함수가 아니라 일반 함수로 동작한다.

```js
// new 연산자와 함께 호출하지 않으면 일반 함수로 동작한다.
const circle3 = Circle(15);

// 일반 함수로 호출된 Circle에는 반환문이 없으므로 undefined를 반환한다.
console.log(circle3); // undefined

// 일반 함수로 호출된 Circle 내부의 this는 전역 객체를 가리킨다.
console.log(radius); // 15
```



### Function.prototype.apply/call/bind 메서드에 의한 간접 호출

apply, call, bind 메서드는 Function.prototype의 메서드다.   
=&gt; 이들 메서드는 모든 함수가 상속받아 사용할 수 있다.

`Function.prototype.apply`와 `Function.prototype.call` 메서드는 this로 사용할 객체와 인수 리스트를 인수로 전달받아 함수를 호출한다.

```js
function getThisBinding() {
  return this;
}

// this로 사용할 객체
const thisArg = { a: 1 };

console.log(getThisBinding()); // window

// apply와 call 메서드는 함수를 호출하면서 첫 번째 인수로 전달한 객체를 this에 바인딩한다.
console.log(getThisBinding.apply(thisArg)); // { a: 1 }
console.log(getThisBinding.call(thisArg)); // { a: 1 }
```

apply와 call 메서드의 본질적인 기능은 함수를 호출하는 것이며, 함수를 호출하면서 첫 번째 인수로 전달한 특정 객체를 호출한 함수의 this에 바인딩한다. 이 둘은 호출할 함수에 인수를 전달하는 방식만 다를 뿐, 동일하게 동작한다.

- `apply`: 호출할 함수의 인수를 **배열**로 묶어 전달
- `call`: 호출할 함수의 인수를 **쉼표로 구분한 리스트** 형식으로 전달

```js
function getThisBinding() {
  console.log(arguments);
  return this;
}

const thisArg = { a: 1 };

// apply: 인수를 배열로 전달
console.log(getThisBinding.apply(thisArg, [1, 2, 3]));
// Arguments(3) [1, 2, 3]
// { a: 1 }

// call: 인수를 쉼표로 구분한 리스트로 전달
console.log(getThisBinding.call(thisArg, 1, 2, 3));
// Arguments(3) [1, 2, 3]
// { a: 1 }
```

apply와 call 메서드의 대표적인 용도는, arguments 객체와 같은 유사 배열 객체에 배열 메서드를 사용하는 경우다. arguments 객체는 배열이 아니기 때문에 Array.prototype의 메서드를 사용할 수 없으나, apply와 call 메서드를 이용하면 가능하다.

```js
function convertArgsToArray() {
  console.log(arguments);

  // arguments 객체를 배열로 변환
  // Array.prototype.slice를 인수 없이 호출하면 배열의 복사본을 생성한다.
  const arr = Array.prototype.slice.call(arguments);
  console.log(arr);

  return arr;
}

convertArgsToArray(1, 2, 3); // [1, 2, 3]
```

`Function.prototype.bind` 메서드는 apply, call 메서드와 달리 **함수를 호출하지 않는다.** 다만 첫 번째 인수로 전달한 값으로 this 바인딩이 교체된 함수를 새롭게 생성해 반환한다.

```js
function getThisBinding() {
  return this;
}

const thisArg = { a: 1 };

// bind 메서드는 함수를 호출하지 않고, this로 사용할 객체만 전달한다.
console.log(getThisBinding.bind(thisArg)); // getThisBinding

// bind 메서드는 함수를 호출하지 않으므로 명시적으로 호출해야 한다.
console.log(getThisBinding.bind(thisArg)()); // { a: 1 }
```

bind 메서드는 메서드의 this와 메서드 내부의 중첩 함수 또는 콜백 함수의 this가 불일치하는 문제를 해결하기 위해 유용하게 사용된다.

```js
const person = {
  name: 'Lee',
  foo(callback) {
    // ① 이 시점의 this는 person
    setTimeout(callback, 100);
  },
};

person.foo(function () {
  // ② 콜백 함수 내부의 this는 전역 객체 window를 가리킨다.
  // this.name은 window.name과 같으며 기본값은 ''이다.
  console.log(`Hi! my name is ${this.name}.`); // Hi! my name is .
});
```

위 예제는 콜백 함수 내부의 this(②)가 person 객체를 가리키지 않는 문제가 있다. 이때 bind 메서드로 콜백 함수의 this를 외부 함수 내부의 this와 일치시킬 수 있다.

```js
const person = {
  name: 'Lee',
  foo(callback) {
    // bind 메서드로 callback 함수 내부의 this 바인딩을 전달
    setTimeout(callback.bind(this), 100);
  },
};

person.foo(function () {
  console.log(`Hi! my name is ${this.name}.`); // Hi! my name is Lee.
});
```




| 함수 호출 방식                  | this 바인딩                         |
| ------------------------- | -------------------------------- |
| 일반 함수 호출                  | 전역 객체 (strict mode에서는 undefined) |
| 메서드 호출                    | 메서드를 호출한 객체                      |
| 생성자 함수 호출                 | 생성자 함수가 (미래에) 생성할 인스턴스           |
| apply/call/bind에 의한 간접 호출 | 첫 번째 인수로 전달한 객체                  |


<details class="orca-details">
<summary>apply, call, bind... 셋 다 this를 바꾸는 건데 각각 언제 사용하면 좋을까?</summary>

세 메서드의 목적은 같지만(this를 명시적으로 지정) 쓰임새는 갈린다.

- `**call` / `apply`**: **지금 당장 함수를 실행**하면서 this만 바꾸고 싶을 때. 인수를 낱개로 넘기면 `call`, 배열로 넘기면 `apply`다. 예전에는 `Math.max.apply(null, arr)`처럼 배열을 펼쳐 넣는 용도로 `apply`를 많이 썼는데, 요즘은 **스프레드 문법(`Math.max(...arr)`)**이 이 자리를 거의 대체했다.
- `**bind**`: **당장 실행하지 않고, this가 고정된 새 함수를 만들어두고 나중에 실행**할 때. 그래서 콜백이나 이벤트 핸들러처럼 "이 함수를 넘겨줄 건데 this는 유지되면 좋겠다" 하는 상황에 잘 맞는다.

그리고 사실 요즘 React나 모던 자바스크립트에서는 **화살표 함수**가 이 셋의 상당 부분을 대신한다. 화살표 함수는 상위 스코프의 this를 그대로 물려받으니 `bind(this)`를 따로 쓸 필요가 없기 때문이다.

=&gt; 정리하면, "즉시 실행 + this 교체"는 `call`(가끔 `apply`), "나중 실행 + this 고정"은 `bind`, 그리고 콜백의 this 유지는 대부분 **화살표 함수**로 해결한다고 보면 실무 감각에 맞다.

</details>


</br>

# 23장. 실행 컨텍스트(execution context)

## 소스코드의 타입

ECMAScript 사양은 소스코드(실행 가능한 코드)를 4가지 타입으로 구분한다. 이 4가지 타입의 소스코드는 실행 컨텍스트를 생성한다.


| 소스코드의 타입             | 설명                                                       |
| -------------------- | -------------------------------------------------------- |
| 전역 코드(global code)   | 전역에 존재하는 소스코드. 전역에 정의된 함수, 클래스 등의 내부 코드는 포함되지 않는다.       |
| 함수 코드(function code) | 함수 내부에 존재하는 소스코드. 함수 내부에 중첩된 함수, 클래스 등의 내부 코드는 포함되지 않는다. |
| eval 코드(eval code)   | 빌트인 전역 함수인 eval 함수에 인수로 전달되어 실행되는 소스코드.                  |
| 모듈 코드(module code)   | 모듈 내부에 존재하는 소스코드. 모듈 내부의 함수, 클래스 등의 내부 코드는 포함되지 않는다.     |


소스코드를 4가지 타입으로 구분하는 이유는, 소스코드의 타입에 따라 **실행 컨텍스트를 생성하는 과정과 관리 내용이 다르기** 때문이다.

- **전역 코드**: 전역 변수를 관리하기 위해 최상위 스코프인 전역 스코프를 생성해야 하고, var 키워드로 선언된 전역 변수와 함수 선언문으로 정의된 전역 함수를 전역 객체의 프로퍼티와 메서드로 바인딩하고 참조하기 위해 전역 객체와 연결되어야 한다.
- **함수 코드**: 지역 스코프를 생성하고, 지역 변수·매개변수·arguments 객체를 관리해야 한다. 그리고 생성한 지역 스코프를 전역 스코프에서 시작하는 스코프 체인의 일원으로 연결해야 한다.
- **eval 코드**: strict mode에서 독자적인 스코프를 생성한다.
- **모듈 코드**: 모듈별로 독립적인 모듈 스코프를 생성한다.



## 소스코드의 평가와 실행

모든 소스코드는 실행에 앞서 평가 과정을 거치며 코드를 실행하기 위한 준비를 한다. 자바스크립트 엔진은 소스코드를 **2개의 과정**, 즉 "소스코드의 평가"와 "소스코드의 실행"으로 나누어 처리한다.

**소스코드의 평가 과정**에서는 실행 컨텍스트를 생성하고, 변수·함수 등의 선언문만 먼저 실행하여 생성된 변수나 함수 식별자를 **키(key)로 실행 컨텍스트가 관리하는 스코프(렉시컬 환경의 환경 레코드)에 등록**한다.

소스코드 평가 과정이 끝나면 비로소 선언문을 제외한 소스코드가 순차적으로 실행되기 시작한다. 즉, **소스코드의 실행(런타임)**이 시작된다. 이때 소스코드 실행에 필요한 정보, 즉 변수나 함수의 참조를 실행 컨텍스트가 관리하는 스코프에서 검색해서 취득한다. 그리고 변수 값의 변경 등 소스코드의 실행 결과는 다시 실행 컨텍스트가 관리하는 스코프에 등록된다.

```js
var x;
x = 1;
```

위 예제를 사렾보면

1. **평가 과정**: 자바스크립트 엔진은 먼저 소스코드를 평가하여, 변수 선언문 `var x;`를 먼저 실행한다. 이때 생성된 변수 식별자 x는 실행 컨텍스트가 관리하는 스코프에 등록되고 undefined로 초기화된다.
2. **실행 과정**: 소스코드 평가가 끝나면, 비로소 변수 할당문 `x = 1;`만 실행된다. 이때 x 변수에 값을 할당하려면 먼저 x 변수가 선언된 변수인지 확인해야 하는데, 이를 위해 실행 컨텍스트가 관리하는 스코프에 x 변수가 등록되어 있는지 확인한다. 만약 x 변수가 등록되어 있다면 x 변수에 값을 할당하고, 할당 결과를 실행 컨텍스트에 등록하여 관리한다.

## 실행 컨텍스트의 역할

```js
// 전역 변수 선언
const x = 1;
const y = 2;

// 함수 정의
function foo(a) {
  // 지역 변수 선언
  const x = 10;
  const y = 20;

  // 메서드 호출
  console.log(a + x + y); // 130
}

// 함수 호출
foo(100);

// 메서드 호출
console.log(x + y); // 3
```

**① 전역 코드 평가**  
전역 코드를 실행하기에 앞서 전역 코드를 평가한다. 이때 선언문(변수, 함수 선언문)을 먼저 실행하고, 그 결과로 생성된 전역 변수와 전역 함수가 실행 컨텍스트가 관리하는 전역 스코프에 등록된다. 이때 var 키워드로 선언된 전역 변수와 함수 선언문으로 정의된 전역 함수는 전역 객체의 프로퍼티와 메서드가 된다.

**② 전역 코드 실행**  
전역 코드 평가 과정이 끝나면 런타임이 시작되어 전역 코드가 순차적으로 실행되기 시작한다. 이때 전역 변수에 값이 할당되고 함수가 호출된다. 함수가 호출되면 순차적으로 실행되던 전역 코드의 실행을 일시 중단하고, 코드 실행 순서를 변경하여 함수 내부로 진입한다.

**③ 함수 코드 평가**  
함수 호출에 의해 함수 내부로 진입하면, 함수 내부의 문들을 실행하기에 앞서 함수 코드 평가 과정을 거치며 함수 코드를 실행하기 위한 준비를 한다. 이때 매개변수와 지역 변수 선언문이 먼저 실행되고, 그 결과 생성된 매개변수와 지역 변수가 실행 컨텍스트가 관리하는 지역 스코프에 등록된다. 또한 함수 내부에서 지역적으로 사용될 arguments 객체도 생성되어 지역 스코프에 등록되고, this 바인딩도 결정된다.

**④ 함수 코드 실행**  
함수 코드 평가 과정이 끝나면 런타임이 시작되어 함수 코드가 순차적으로 실행되기 시작한다. 이때 매개변수와 지역 변수에 값이 할당되고, `console.log` 메서드가 호출된다.



여기서 console.log 메서드를 호출하기 위해 먼저 식별자인 console을 스코프 체인을 통해 검색한다. 이를 위해 함수 코드의 지역 스코프는 상위 스코프인 전역 스코프와 연결되어야 한다. 스코프 체인을 통해 console 식별자가 전역에 없으면, 자바스크립트 엔진은 전역 객체의 프로퍼티에서도 console을 검색한다.



이처럼 코드가 실행되려면 아래와 같은 것들을 관리할 수 있어야 한다.

- 선언에 의해 생성된 모든 식별자(변수, 함수, 클래스 등)를 스코프를 구분하여 등록하고, 상태 변화를 지속적으로 관리
- 스코프는 중첩 관계에 의해 스코프 체인을 형성해야 하며, 이를 통해 상위 스코프로 이동하며 식별자를 검색
- 현재 실행 중인 코드의 실행 순서를 변경(예: 함수 호출에 의한 실행 순서 변경)할 수 있어야 하며, 다시 되돌아갈 수도 있어야 함

=&gt; 이 모든 것을 관리하는 것이 바로 **실행 컨텍스트**다. 실행 컨텍스트는 소스코드를 실행하는 데 필요한 환경을 제공하고, 코드의 실행 결과를 실제로 관리하는 영역이다. 즉, 실행 컨텍스트는 **식별자를 등록하고 관리하는 스코프**와 **코드 실행 순서 관리**를 구현한 내부 메커니즘으로, 모든 코드는 실행 컨텍스트를 통해 실행되고 관리된다.



## 실행 컨텍스트 스택(execution context stack)

앞선 예제의 소스코드가 실행되면, 다음과 같이 실행 컨텍스트가 생성된다.

```js
const x = 1;

function foo() {
  const y = 2;

  function bar() {
    const z = 3;
    console.log(x + y + z);
  }
  bar();
}

foo(); // 6
```

위 예제는 전역 코드와 함수 코드로 이루어져 있다. 이때 생성된 실행 컨텍스트는 스택 자료구조로 관리된다. 이를 **실행 컨텍스트 스택**이라 부른다.

> 실행 컨텍스트 스택은 코드의 실행 순서를 관리한다.

스택은 후입선출(LIFO) 구조를 따른다. 위 예제의 실행 순서를 따라가 보면 이렇다.

**① 전역 코드의 평가와 실행**  
자바스크립트 엔진은 먼저 전역 코드를 평가하여 전역 실행 컨텍스트를 생성하고 실행 컨텍스트 스택에 푸시한다. 이때 전역 변수 x와 전역 함수 foo는 전역 실행 컨텍스트에 등록된다. 이후 전역 코드가 실행되기 시작하여 전역 변수 x에 값이 할당되고, 전역 함수 foo가 호출된다.

**② foo 함수 코드의 평가와 실행**  
전역 함수 foo가 호출되면 전역 코드의 실행은 일시 중단되고, 코드의 제어권이 foo 함수 내부로 이동한다. 자바스크립트 엔진은 foo 함수 코드를 평가하여 foo 함수 실행 컨텍스트를 생성하고 실행 컨텍스트 스택에 푸시한다. 이후 foo 함수 코드가 실행되기 시작하여 지역 변수 y에 값이 할당되고, 중첩 함수 bar가 호출된다.

**③ bar 함수 코드의 평가와 실행**  
중첩 함수 bar가 호출되면 foo 함수 코드의 실행은 일시 중단되고, 코드의 제어권이 bar 함수 내부로 이동한다. 자바스크립트 엔진은 bar 함수 코드를 평가하여 bar 함수 실행 컨텍스트를 생성하고 실행 컨텍스트 스택에 푸시한다. 이후 bar 함수 코드가 실행되기 시작하여 지역 변수 z에 값이 할당되고, `console.log` 메서드를 호출한 후 bar 함수는 종료된다.

**④ foo 함수 코드로 복귀**  
bar 함수가 종료되면 bar 함수 실행 컨텍스트가 실행 컨텍스트 스택에서 팝되어 제거되고, 코드의 제어권은 다시 foo 함수로 이동한다. 이때 자바스크립트 엔진은 foo 함수 실행 컨텍스트를 실행 컨텍스트 스택의 최상위, 즉 실행 중인 실행 컨텍스트로 관리한다. 이후 foo 함수도 더 이상 실행할 코드가 없으므로 종료된다.

**⑤ 전역 코드로 복귀**  
foo 함수가 종료되면 foo 함수 실행 컨텍스트가 실행 컨텍스트 스택에서 팝되어 제거되고, 코드의 제어권은 다시 전역 코드로 이동한다. 이후 실행할 전역 코드가 없으므로 전역 실행 컨텍스트도 실행 컨텍스트 스택에서 팝되어, 실행 컨텍스트 스택에는 아무것도 남아있지 않게 된다.

=&gt; 이처럼 실행 컨텍스트 스택은 코드의 실행 순서를 관리한다. 소스코드가 평가되면 실행 컨텍스트가 생성되고 스택에 푸시되며, 언제나 **실행 컨텍스트 스택의 최상위에 존재하는 실행 컨텍스트가 현재 실행 중인 코드의 실행 컨텍스트**다.



## 렉시컬 환경(Lexical Environment)

렉시컬 환경은 **식별자와 식별자에 바인딩된 값, 그리고 상위 스코프에 대한 참조를 기록**하는 자료구조로, 실행 컨텍스트를 구성하는 컴포넌트다.

렉시컬 환경은 키와 값을 갖는 객체 형태의 스코프(전역, 함수, 블록 스코프)를 생성하여 식별자를 키로 등록하고, 식별자에 바인딩된 값을 관리한다. 즉, 렉시컬 환경은 스코프를 구분하여 식별자를 등록하고 관리하는 저장소 역할을 하는 렉시컬 스코프의 실체다.

실행 컨텍스트는 `LexicalEnvironment` 컴포넌트와 `VariableEnvironment` 컴포넌트로 구성된다. 생성 초기에 이 둘은 하나의 동일한 렉시컬 환경을 참조한다. (이후 몇몇 상황을 만나면 서로 다른 것을 가리키게 되지만, 대부분의 경우 동일하다고 봐도 무방하다.)



렉시컬 환경은 **2개의 컴포넌트**로 구성된다.

**① 환경 레코드(Environment Record)**  
스코프에 포함된 식별자를 등록하고, 등록된 식별자에 바인딩된 값을 관리하는 저장소다. 환경 레코드는 소스코드의 타입에 따라 관리하는 내용에 차이가 있다.

**② 외부 렉시컬 환경에 대한 참조(Outer Lexical Environment Reference)**  
외부 렉시컬 환경에 대한 참조는 상위 스코프를 가리킨다. 이때 상위 스코프란, 외부 렉시컬 환경, 즉 해당 실행 컨텍스트를 생성한 소스코드를 포함하는 상위 코드의 렉시컬 환경을 말한다. 외부 렉시컬 환경에 대한 참조는 **스코프 체인을 구현**하는 장치다.

<details class="orca-details">
<summary>렉시컬 환경이랑 실행 컨텍스트는 뭐가 다른 거지? </summary>

둘의 관계는 "실행 컨텍스트가 렉시컬 환경을 **가지고 있다**"로 이해하면 깔끔하다.

- **실행 컨텍스트**는 코드를 실행하기 위한 전체 환경이자 관리 주체다. 스택에 푸시/팝되면서 **코드의 실행 순서**를 관리하는 역할이 크다.
- **렉시컬 환경**은 그 실행 컨텍스트 안에서 **식별자(변수/함수)와 그 값, 그리고 상위 스코프 참조를 실제로 저장**하는 자료구조다. 스코프의 실체가 바로 이 렉시컬 환경이다.

비유하자면, 실행 컨텍스트가 "지금 어떤 코드를 실행 중인지 관리하는 작업 지시서"라면, 렉시컬 환경은 "그 작업에 필요한 변수들이 담긴 서랍장"인 셈이다. 그리고 서랍장(렉시컬 환경)에는 **상위 서랍장으로 가는 통로(외부 렉시컬 환경 참조)**가 붙어 있어서, 지금 서랍에 없는 변수는 위층 서랍을 뒤지러 올라간다. 이 통로들이 이어진 것이 바로 **스코프 체인**이다.

=&gt; 즉 "실행 순서 관리 = 실행 컨텍스트 스택", "식별자와 스코프 관리 = 렉시컬 환경"으로 역할을 나눠서 기억하면 헷갈리지 않는다.

</details>



## 실행 컨텍스트의 생성과 식별자 검색 과정

```js
var x = 1;
const y = 2;

function foo(a) {
  var x = 3;
  const y = 4;

  function bar(b) {
    const z = 5;
    console.log(a + b + x + y + z);
  }
  bar(10);
}

foo(20); // 42
```

### ① 전역 객체 생성

전역 객체는 전역 코드가 평가되기 이전에 생성된다. 이때 전역 객체에는 빌트인 전역 프로퍼티와 빌트인 전역 함수, 그리고 표준 빌트인 객체가 추가되며, 동작 환경(클라이언트 사이드 또는 서버 사이드)에 따라 호스트 객체를 포함한다. 전역 객체도 Object.prototype을 상속받는다.

### ② 전역 코드 평가

전역 코드가 평가되면 다음 순서로 전역 실행 컨텍스트가 생성된다.

1. 전역 실행 컨텍스트 생성 → 실행 컨텍스트 스택에 푸시
2. 전역 렉시컬 환경 생성
  - 전역 환경 레코드 생성
    - **객체 환경 레코드(Object Environment Record)** 생성: var 키워드로 선언한 전역 변수와 함수 선언문으로 정의된 전역 함수, 빌트인 전역 프로퍼티/함수, 표준 빌트인 객체를 관리한다. (전역 객체와 연결)
    - **선언적 환경 레코드(Declarative Environment Record)** 생성: let, const 키워드로 선언한 전역 변수를 관리한다. (전역 객체의 프로퍼티가 아님. 그래서 let, const로 선언한 전역 변수는 개념적인 블록 내에 존재하게 된다.)
  - this 바인딩
  - 외부 렉시컬 환경에 대한 참조 결정 → 전역은 최상위 스코프이므로 `null`

여기서 var로 선언한 x는 객체 환경 레코드에 등록되어 undefined로 초기화되지만, const로 선언한 y는 선언적 환경 레코드에 등록되되 **초기화되지 않은 상태**가 된다.

=&gt; 이것이 바로 let, const로 선언한 변수의 **TDZ**가 발생하는 이유다. 선언은 되었지만 값이 초기화되기 이전이라, 그 사이에 변수를 참조하면 참조 에러가 발생한다.

### ③ 전역 코드 실행

이제 전역 코드가 순차적으로 실행되기 시작한다. 변수 할당문이 실행되어 전역 변수 x와 y에 값이 할당되고, 함수 foo가 호출된다.

이때 변수 할당문 또는 함수 호출문을 실행하려면 먼저 식별자를 검색해야 한다. 식별자는 스코프 체인을 통해 검색한다.

=&gt; 식별자를 검색할 때는, 실행 중인 실행 컨텍스트에서 식별자를 검색하기 시작한다. 만약 실행 중인 실행 컨텍스트의 렉시컬 환경에서 식별자를 검색할 수 없으면, 외부 렉시컬 환경에 대한 참조가 가리키는 렉시컬 환경, 즉 상위 스코프로 이동하여 식별자를 검색한다.

이것이 바로 **스코프 체인의 동작 원리**다. 만약 전역 렉시컬 환경까지 이동하여 식별자를 검색할 수 없으면 참조 에러(ReferenceError)를 발생시킨다.

### ④ foo 함수 코드 평가

foo 함수가 호출되면 전역 코드의 실행을 중단하고 foo 함수 내부로 진입한다. 이제 함수 코드를 평가한다.

1. 함수 실행 컨텍스트 생성 → 실행 컨텍스트 스택에 푸시
2. 함수 렉시컬 환경 생성
  - **함수 환경 레코드(Function Environment Record)** 생성: 매개변수, arguments 객체, 함수 내부의 지역 변수와 중첩 함수를 등록하고 관리한다.
  - this 바인딩
  - 외부 렉시컬 환경에 대한 참조 결정 → **foo 함수 정의 위치**에 따라 상위 스코프가 결정된다. (렉시컬 스코프)

여기서 중요한 점은, 외부 렉시컬 환경에 대한 참조가 **함수가 호출된 위치가 아니라 함수가 정의된 위치**를 기준으로 결정된다는 것이다. foo 함수는 전역에서 정의되었으므로, foo 함수 렉시컬 환경의 외부 렉시컬 환경 참조는 전역 렉시컬 환경을 가리킨다.

> `함수 객체의 내부 슬롯 [[Environment]]` 함수는 정의된 환경(위치)에 의해 상위 스코프를 기억한다. 즉, 함수 객체는 자신이 정의된 렉시컬 환경을 [[Environment]] 내부 슬롯에 저장하고, 함수가 호출되면 이 값을 외부 렉시컬 환경에 대한 참조에 할당한다. 이것이 렉시컬 스코프의 실체이며, 클로저가 동작하는 원리의 기반이 된다.

### ⑤ foo 함수 코드 실행 → ⑥ bar 함수 코드 평가 → ⑦ bar 함수 코드 실행

이후 과정은 앞과 동일하다. foo 함수 코드가 실행되며 매개변수 a에 값이 할당되고 지역 변수 x, y에 값이 할당된 후 bar 함수가 호출된다. bar 함수도 마찬가지로 평가·실행 과정을 거친다.

bar 함수 내부의 `console.log(a + b + x + y + z)`가 실행될 때, a, b, x, y, z 각 식별자는 스코프 체인을 따라 검색된다.

- z, b =&gt; bar 함수 렉시컬 환경에서 검색
- x, y, a =&gt; bar에 없으므로 상위 스코프인 foo 함수 렉시컬 환경에서 검색
- console =&gt; foo, 전역 어디에도 지역 식별자로 없으므로 최종적으로 전역 객체의 프로퍼티에서 검색

여기서 **식별자 검색과 프로퍼티 검색은 다르다**는 점을 짚고 넘어가자.

- **식별자**(예: a, x, y): 스코프 체인을 통해 검색한다.
- **프로퍼티**(예: console 뒤의 log): 프로토타입 체인을 통해 검색한다.

### ⑧ bar / foo 함수 코드 실행 종료

bar 함수가 종료되면 bar 함수 실행 컨텍스트가 스택에서 팝된다. 이어서 foo 함수도 종료되면 foo 함수 실행 컨텍스트가 스택에서 팝된다.

=&gt; (주의할 점!!) 실행 컨텍스트 스택에서 실행 컨텍스트가 팝되어 제거되었다고 해서, 그 실행 컨텍스트가 관리하던 **렉시컬 환경까지 즉시 소멸하는 것은 아니다.** 렉시컬 환경은 실행 컨텍스트에 의해 참조되기는 하지만, 독립적인 객체이므로 누군가 그 렉시컬 환경을 참조하고 있다면(예: 클로저) 가비지 컬렉션의 대상이 되지 않는다. 이것이 바로 다음 24장에서 배울 **클로저**의 원리와 연결된다.



## 실행 컨텍스트와 블록 레벨 스코프

마지막으로 블록 레벨 스코프를 실행 컨텍스트 관점에서 살펴보자.

var 키워드로 선언한 변수는 오로지 함수의 코드 블록만을 지역 스코프로 인정하는 **함수 레벨 스코프**를 따른다. 하지만 let, const 키워드로 선언한 변수는 모든 코드 블록(if, for, while, try/catch 등)을 지역 스코프로 인정하는 **블록 레벨 스코프**를 따른다.

```js
let x = 1;

if (true) {
  // if 문의 코드 블록 내에서는 새로운 렉시컬 환경을 생성한다.
  let x = 10;
  console.log(x); // 10
}

console.log(x); // 1
```

if 문의 코드 블록이 실행되면, if 문의 코드 블록을 위한 **블록 레벨 스코프를 생성**해야 한다. 이를 위해 선언적 환경 레코드를 갖는 렉시컬 환경을 새롭게 생성하여 기존의 전역 렉시컬 환경을 교체한다. 이때 새롭게 생성된 if 문의 코드 블록을 위한 렉시컬 환경의 외부 렉시컬 환경에 대한 참조는, if 문이 실행되기 이전의 렉시컬 환경(전역 렉시컬 환경)을 가리킨다. if 문의 코드 블록의 실행이 종료되면, 이전의 렉시컬 환경으로 되돌린다.

이러한 블록 레벨 스코프는 if 문뿐만 아니라 함수, for 문, while 문, try/catch 문 등 모든 블록에서 생성된다. 특히 for 문의 경우 매 반복마다 새로운 렉시컬 환경을 생성하여 반복 시점의 상태를 마치 스냅숏을 찍는 것처럼 저장하는데, 이것이 for 문에서 let과 var가 다르게 동작하는 이유다.

<details class="orca-details">
<summary>for 반복문에서 let이랑 var가 왜 다르게 동작하는지 실행 컨텍스트로 설명하면?</summary>

```js
// var
for (var i = 0; i < 3; i++) {
  setTimeout(() => console.log(i), 0);
}
// 3, 3, 3 출력

// let
for (let i = 0; i < 3; i++) {
  setTimeout(() => console.log(i), 0);
}
// 0, 1, 2 출력
```

`**var`는 함수 레벨 스코프**라서, 반복문이 아무리 돌아도 i는 **하나의 렉시컬 환경(전역 또는 함수)에 단 하나만** 존재한다. setTimeout의 콜백들은 전부 그 **똑같은 i 하나**를 참조하고, 실제로 콜백이 실행되는 시점(반복이 다 끝난 후)에 i는 이미 3이 되어 있다. 그래서 3이 세 번 찍힌다.

`**let`은 블록 레벨 스코프**라서, for 문이 매 반복마다 **새로운 렉시컬 환경을 생성**하고 그 시점의 i 값을 복사해 넣는다. 즉 반복마다 i가 담긴 서랍이 새로 만들어지는 셈이다. 각 setTimeout 콜백은 자기 반복 시점의 **서로 다른 렉시컬 환경**을 클로저로 붙잡고 있으므로, 0, 1, 2가 순서대로 찍힌다.

=&gt; 정리하면, "var는 렉시컬 환경 하나를 공유, let은 반복마다 렉시컬 환경을 새로 스냅숏"이라는 차이다. 이 동작이 바로 다음 장에서 배울 **클로저**와 곧장 이어진다.

</details>
