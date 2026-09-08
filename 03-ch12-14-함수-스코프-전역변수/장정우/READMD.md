# 모던 자바스크립트 Deep Dive 13~14장 정리

## 13장. 스코프

### 스코프란?

> **네이밍 차이**  
> - 스코프(scope): 식별자(변수·함수 이름)에 접근할 수 있는 범위.
> - 컨텍스트(context): 코드가 실행되는 상황과 환경.

**같은 이름의 식별자가 여러 개 있을 때**, 자바스크립트 엔진은 현재 실행 중인 코드의 스코프부터 식별자를 검색한다.
(같은 이름의 식별자를 가급적 피하자. 파악하기 어렵다.)

```js
  // 난감한 사례
const arr1 = [
  ['a', 'b'], 
  ['c', 'd']
];

arr1.forEach((item, index) => {
  item.forEach((item, index) => {
    console.log(item, index);
  });
});
```

```js
function add(x, y) {
  // x와 y는 함수 내부에서만 참조할 수 있다.
  console.log(x + y);
}

add(2, 5); // 7
console.log(x); // ReferenceError

/**
 전역 스코프
 ├─ add
 └─ x 없음

 add 함수 스코프
 ├─ x
 └─ y
**/
```

식별자는 자신이 선언된 스코프와 그 하위 스코프에서 유효하다.
스코프가 없다면 전부 다 global 하게 이름이 관리 될테니, 전역에서 이름이 충돌하게 된다.

### 전역 스코프와 지역 스코프

- 전역 스코프: 코드의 가장 바깥 영역
- 지역 스코프: 함수 내부처럼 특정 코드 블록이나 함수에 의해 만들어지는 영역

```js
var x = 'global x';

function outer() {
  var x = 'outer local x';

  function inner() {
    var x = 'inner local x';
    console.log(x); // 'inner local x'
  }

  inner();
  console.log(x); // 'outer local x'
}

outer();
console.log(x); // 'global x'
```

같은 이름의 변수가 여러 스코프에 있으면 가장 가까운 스코프의 변수가 우선된다.
자바스크립트 엔진이 여러 스코프에서 같은 이름의 식별자 중 어떤 변수를 참조할지 결정하는 과정을 **식별자 결정(identifier resolution)**이라고 한다.

```js
var x = 'global';

function foo() {
  var x = 'local';
  console.log(x); // 'local': 현재 함수 스코프의 x를 참조
}

foo();
console.log(x); // 'global': 전역 스코프의 x를 참조
```

식별자 결정은 현재 스코프에서 시작해 상위 스코프로 이동하며 이루어진다.
이를 **스코프 체인을 통한 식별자 검색**이라고 한다.

### 스코프 체인 

> outerEnvironmentReference를 통한 참조 체인

모든 스코프는 계층적으로 연결되어 있다.
변수를 참조하면 자바스크립트 엔진은 현재 스코프에서 검색을 시작해 상위 스코프로 이동하며 식별자를 찾는다.

```js 
var x = 'global';

function outer() {
  var y = 'outer';

  function inner() {
    var z = 'inner';

    console.log(z); // 현재 스코프에서 검색
    console.log(y); // outer 스코프에서 검색
    console.log(x); // 전역 스코프에서 검색
  }

  inner();
}

outer();
```

반대로 하위 스코프에서 선언된 식별자를 상위 스코프가 참조할 수는 없다.

```js
function outer() {
  function inner() {
    const secret = 'inner';
  }

  console.log(secret); // ReferenceError
}
```

### 함수 레벨 스코프

`var` 키워드로 선언한 변수는 함수의 코드 블록만 지역 스코프로 인정한다.

따라서 `if`, `for`, `while` 같은 블록 안에서 선언해도 함수 바깥으로 유출될 수 있다.
(if / for / switch / while 같은 문들은 별개의 실행 컨텍스트를 생성하지 않기 때문. 이것들은 어떻게 변수가 선언되었냐가 더 중요.)

```js
var x = 1;

if (true) {
  var x = 10;
}

console.log(x); // 10
```

이러한 특성 때문에 `var`는 의도하지 않은 변수 재할당이나 스코프 오염을 만들 수 있다.
ES6 이후에는 블록 레벨 스코프를 지원하는 `let`과 `const`를 사용하는 것이 일반적이다.

### 렉시컬 스코프

자바스크립트는 함수를 **어디서 호출했는지**가 아니라 **어디서 정의했는지**에 따라 상위 스코프를 결정한다.
이를 렉시컬 스코프(또는 정적 스코프)라고 한다.

```js
var x = 1;

function foo() {
  var x = 10;
  bar();
}

function bar() {
  console.log(x);
}

foo(); // 1
```

`bar` 함수는 전역에서 정의되었기 때문에 전역 변수 `x`를 참조한다.
`foo`에서 호출되었다는 사실은 `bar`의 상위 스코프에 영향을 주지 않는다.

## 14장. 전역 변수의 문제점

### 변수의 생명 주기

변수의 생명 주기는 선언된 스코프와 밀접한 관련이 있다.

- 지역 변수: 함수가 호출되면 생성되고, 함수가 종료되면 소멸한다.
- 전역 변수: 애플리케이션이 종료될 때까지 유지된다.

```js
function foo() {
  var x = 'local';
  console.log(x); // 'local'
}

foo();
// 함수가 종료된 뒤에는 지역 변수 x에 접근할 수 없다.
console.log(x); // ReferenceError
```

지역 변수는 함수가 종료되면 더 이상 사용되지 않을 때 가비지 컬렉터의 대상이 될 수 있다.
단, 다른 함수가 해당 변수를 참조하고 있다면 클로저 때문에 더 오래 유지될 수 있다.

전역 변수는 전역 객체의 프로퍼티가 되며, 프로그램이 종료될 때까지 유지된다.
따라서 전역 변수는 필요한 경우에만 신중하게 사용해야 한다.

### 전역 변수의 문제점

#### 1. 암묵적 결합

전역 변수를 선언하면 어디서든 참조하고 변경할 수 있다.
여러 함수가 같은 전역 변수에 의존하면 함수의 동작이 외부 상태에 쉽게 영향을 받는다.

```js
let count = 0;

function increase() {
  count++;
}

increase();
console.log(count); // 1
```

`increase` 함수는 매개변수나 반환값만으로 동작을 설명할 수 없고, 외부의 `count`에 의존한다.
이처럼 전역 변수와 함수가 강하게 결합되면 코드의 예측과 테스트가 어려워진다.

#### 2. 긴 생명 주기

전역 변수는 애플리케이션이 종료될 때까지 메모리에 남아 있을 수 있다.
사용이 끝난 값도 오래 유지되면 메모리 사용량이 늘고 상태 관리가 복잡해진다.

#### 3. 스코프 체인 검색 비용

전역 변수는 스코프 체인의 가장 상위에 있다.
지역 변수보다 검색 과정이 길어질 수 있고, 같은 이름의 지역 변수가 추가되면 어떤 값이 사용되는지 혼동하기 쉽다.

```js
const value = 'global';

function printValue() {
  console.log(value);
}

printValue(); // 'global'
```

#### 4. 네임스페이스 오염

여러 파일이 하나의 전역 스코프를 공유하면 서로 다른 파일에서 같은 이름을 사용할 때 충돌할 수 있다.

```js
// first.js
var app = 'first';

// second.js
var app = 'second';

console.log(app); // 'second'
```

### 전역 변수 사용을 줄이는 방법

#### 즉시 실행 함수(IIFE)

함수 정의와 동시에 호출해 변수를 함수 내부에 가둔다.

```js
(function () {
  var privateValue = 10;
  console.log(privateValue); // 10
})();

console.log(privateValue); // ReferenceError
```

라이브러리처럼 전역으로 노출할 값이 적고, 나머지 구현을 숨기고 싶을 때 사용할 수 있다.

#### 네임스페이스 객체

하나의 전역 객체 아래에 프로퍼티를 모아 전역 이름의 충돌을 줄인다.

```js
var MYAPP = {};

MYAPP.name = 'Lee';
MYAPP.user = {
  age: 20
};

console.log(MYAPP.name); // 'Lee'
```

다만 네임스페이스 객체 자체는 전역 변수이므로 근본적인 해결책이라기보다 충돌을 줄이는 방법이다.

#### 모듈 패턴

즉시 실행 함수로 공개할 값과 비공개 값을 구분하고, 공개할 프로퍼티만 반환한다.

```js
var Counter = (function () {
  var num = 0;

  return {
    increase() {
      return ++num;
    },
    decrease() {
      return --num;
    }
  };
})();

console.log(Counter.num);       // undefined
console.log(Counter.increase()); // 1
console.log(Counter.decrease()); // 0
```

`num`은 외부에 직접 노출되지 않고 반환된 메서드를 통해서만 접근할 수 있다.

#### ES6 모듈

ES6 모듈을 사용하면 모듈 파일은 자체적인 모듈 스코프를 갖는다.
모듈 내부의 변수는 기본적으로 전역 객체의 프로퍼티가 되지 않으며, `export`한 값만 외부에 공개된다.

```js
// counter.js
let count = 0;

export function increase() {
  return ++count;
}
```

```js
// app.js
import { increase } from './counter.js';

console.log(increase()); // 1
```

```js
  <script src="normal.js"></script>
  <script type="module" src="module.js"></script>

  // normal.js
  var a = 1;
  console.log(window.a); // 1

  // module.js
  var b = 2;
  console.log(window.b); // undefined
```

브라우저에서 직접 사용할 때는 `script` 태그에 `type="module"`을 지정한다.

```html
<script type="module" src="app.js"></script>
```

실무에서는 가능한 한 전역 변수 대신 모듈의 `import`와 `export`를 사용해 의존 관계를 명확하게 만드는 것이 좋다.
