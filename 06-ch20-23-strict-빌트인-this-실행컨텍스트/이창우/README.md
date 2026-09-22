# 모던 자바스크립트 Deep Dive 20 ~ 22장 정리

> 20장 strict mode · 21장 빌트인 객체 · 22장 this

## 20장 strict mode

### strict mode

- 자바스크립트 언어의 문법을 좀 더 엄격히 적용하여 오류를 발생시킬 가능성이 높거나, 최적화 작업의 문제가 될 수 있는 코드에 대한 명시적인 에러 발생
- 함수 선두에 `use strict` 선언을 통해 적용

```js
function foo() {
  'use strict'; // 반드시 함수 선두(또는 전역 선두)에 위치해야 한다

  x = 10; // ReferenceError: x is not defined
}
foo();
```

```js
function foo() {
  x = 10; // 선두가 아니면 무시된다 → 에러가 발생하지 않음
  'use strict';
}
foo();
```

- 왜 에러가 나는가? → `x`를 선언 없이 할당하면 자바스크립트 엔진이 암묵적으로 전역 객체의 프로퍼티로 만들어버린다(**암묵적 전역**). strict mode는 이걸 에러로 잡아준다.

```js
// non-strict
function foo() {
  x = 10; // 선언한 적 없는데
}
foo();
console.log(x); // 10 ← window.x 로 만들어져버림 (암묵적 전역)
```

- 전역에 `strict mode`를 적용시키는 건 좋지 않다. 라이브러리 중 `non-strict`할 수 있기에, 관리하기가 어렵기 때문. 이 경우 즉시실행함수를 통해 스코프를 구분해야 한다.

```js
// 전역 전체에 적용하지 말고, 즉시실행함수로 스코프를 나눠서 적용
(function () {
  'use strict';

  // 이 안에서만 strict mode가 적용된다
  // Do something...
})();
```

> 도입 시, 암묵적 전역, 변수·함수·매개변수의 삭제, 매개변수 이름 중복, with 구문 등 안티패턴 발견 시 에러를 준다

```js
(function () {
  'use strict';

  // 1. 암묵적 전역
  x = 1; // ReferenceError: x is not defined

  // 2. 변수, 함수, 매개변수의 삭제 (delete 연산자)
  var y = 1;
  delete y; // SyntaxError: Delete of an unqualified identifier in strict mode.

  function bar(a) {
    delete a; // SyntaxError
  }
  delete bar; // SyntaxError

  // 3. 매개변수 이름의 중복
  function baz(a, a) { // SyntaxError: Duplicate parameter name not allowed in this context
    return a + a;
  }

  // 4. with 문의 사용
  with ({ x: 1 }) { // SyntaxError: Strict mode code may not include a with statement
    console.log(x);
  }
})();
```

- strict mode에서는 일반 함수로서 호출하면 this에 undefined가 바인딩. 생성자 함수가 아닌 일반 함수 내부에서는 this를 사용할 필요가 없기 때문

```js
(function () {
  'use strict';

  function foo() {
    console.log(this); // undefined (non-strict 였다면 window)
  }
  foo();

  function Foo() {
    console.log(this); // Foo {} — 생성자 함수로 호출하면 인스턴스
  }
  new Foo();
})();
```

## 21장 빌트인 객체

### 빌트인 객체

- **표준 빌트인 객체**
  - ECMAScript 사양에 정의된 객체, 실행 환경과 관계 없이 언제나 사용 가능
  - `Object`, `String`, `Number`, `Boolean`, `Symbol`, `Date`, `Math`, `RegExp`, `Array`, `Map/Set`, `Promise`, `JSON`, `Error` 등
- **호스트 객체**
  - 자바스크립트 실행 환경에서 추가로 제공하는 객체
  - 브라우저: DOM, BOM, `fetch`, `XMLHttpRequest`, Web Storage 등 / Node.js: Node 고유 API
- **사용자 정의 객체**
  - 사용자가 직접 정의한 객체

<br />

- 생성자 함수 객체인 표준 빌트인 객체는, 프로토타입 메서드와 정적 메서드 제공
- 생성자 함수 객체가 아닌, 표준 빌트인 객체는 정적 메서드만 제공 (예: `Math`, `JSON`, `Reflect`)

```js
// 생성자 함수가 아닌 표준 빌트인 객체 → 정적 메서드만 제공
Math.max(1, 2, 3); // 3
JSON.stringify({ a: 1 }); // '{"a":1}'
// new Math(); // TypeError: Math is not a constructor
```

- 표준 빌트인 객체인 `Number`의 프로퍼티에 바인딩 된 객체, `Number.prototype`은 다양한 빌트인 프로토타입 메서드 제공 → 이는 상속받는 `Number` 인스턴스가 사용 가능. 표준 빌트인 객체 `Number`는 인스턴스 없이 정적 호출 가능한 정적 메서드 제공

```js
// 생성자 함수로 호출 → Number 인스턴스(객체) 생성
const numObj = new Number(1.5);
console.log(numObj); // Number {1.5} ← 원시값이 아니라 객체다

// 1) 프로토타입 메서드: 인스턴스가 상속받아 사용
console.log(numObj.toFixed(0)); // '2'
console.log(Object.getPrototypeOf(numObj) === Number.prototype); // true

// 2) 정적 메서드: 인스턴스 없이 Number 객체가 직접 호출
console.log(Number.isInteger(0.5)); // false
// console.log(numObj.isInteger(0.5)); // TypeError: numObj.isInteger is not a function
```

### 래퍼 객체

- 원시값으로 표현하더라도, 마치 프로퍼티와 메서드를 가진 객체로 동작한다. 원시값 ≠ 객체임에도
- 자바스크립트 엔진이 암묵적으로 대응되는 객체 값으로 변환시켜줌 ⇒ **래퍼 객체**

```js
const str = 'hello'; // 원시값

// 원시값에 마침표 표기법으로 접근하는 순간,
// 엔진이 암묵적으로 래퍼 객체 new String('hello') 를 생성해서 그 객체의 메서드를 실행한다
console.log(str.toUpperCase()); // 'HELLO'
console.log(str.length); // 5

// 처리가 끝나면 래퍼 객체는 버려지고, 다시 원시값으로 되돌아간다
console.log(typeof str); // 'string'
```

```js
const str = 'hi';

str.name = 'Lee'; // 이때 만들어진 래퍼 객체에 프로퍼티가 추가되지만,
// 그 래퍼 객체는 곧바로 가비지 컬렉션 대상이 된다

console.log(str.name); // undefined ← 새로 만들어진 래퍼 객체라서 값이 없다
```

### 전역 객체

- 코드가 실행되기 이전 단계에서 자바스크립트 엔진에 의해 어떤 객체보다 먼저 생성되는 특수한 객체이자 최상위 객체
- 브라우저 환경에서는 `window`, 노드 환경에서는 `global`, (ES11)에서는 `globalThis`가 도입. `window`와 `global`을 통일한 식별자
- 전역 객체는 개발자가 의도적으로 생성할 수 없다, 생성자 함수가 제공되지 않는다.
- (복습) `let`, `const` 키워드가 아닌 `var`로 선언하게 되는 경우 전역 객체의 프로퍼티로 할당이 가능하다

```js
// 브라우저 환경
var foo = 1;
console.log(window.foo); // 1 ← var 전역 변수는 전역 객체의 프로퍼티

let bar = 2;
console.log(window.bar); // undefined ← let/const 는 전역 객체의 프로퍼티가 아니다
// (전역 렉시컬 환경의 선언적 환경 레코드에 존재한다)

// 암묵적 전역도 전역 객체의 프로퍼티가 된다
baz = 3;
console.log(window.baz); // 3

// 표준 빌트인 객체도 전역 객체의 프로퍼티다 → 그래서 window 없이 쓸 수 있다
console.log(window.Math === Math); // true
```

### 빌트인 전역 프로퍼티, 함수

- 전역 객체의 프로퍼티: `Infinity`, `NaN`, `undefined`

```js
console.log(window.Infinity === Infinity); // true
console.log(3 / 0); // Infinity

console.log(typeof NaN); // 'number'
console.log(NaN === NaN); // false ← 자기 자신과도 같지 않다

console.log(window.undefined); // undefined
```

- 전역 함수 `eval`(런타임에 평가하여 값을 생성), `isNaN`, `isFinite`, `parseFloat`, `parseInt` 등

```js
// eval: 전달받은 문자열을 런타임에 평가/실행한다
// → 보안에 취약하고 최적화가 되지 않아 느리다. 사용하지 말 것
eval('1 + 2;'); // 3

// isNaN: 인수를 숫자로 암묵적 타입 변환한 뒤 NaN인지 검사
isNaN('blabla'); // true  ('blabla' → NaN)
isNaN(''); // false ('' → 0)
isNaN(null); // false (null → 0)

// isFinite: 정상적인 유한수인지 검사
isFinite(0); // true
isFinite(Infinity); // false
isFinite(null); // true (null → 0)

// parseFloat / parseInt: 문자열을 숫자로
parseFloat('3.14가나다'); // 3.14 ← 앞에서부터 숫자로 해석되는 부분까지만
parseInt('10', 2); // 2 ← 2진수 '10'을 10진수로 해석
parseInt('He'); // NaN
```

## 22장 this

### this 키워드

- 객체는 프로퍼티와 메서드로 구성
- 메서드는 프로퍼티를 참조하고 구성할 수 있어야 한다 ⇒ 자신이 속한 객체를 가리키는 식별자를 참조할 수 있어야 한다.
- **this는 자신이 속한 객체 또는 자신이 생성할 인스턴스를 가리키는 자기 참조 변수.** this를 통해 자신이 속한 객체나 자신이 생성할 인스턴스의 프로퍼티, 메서드를 참조할 수 있다.
- this는 자바스크립트에 의해 암묵적으로 생성, 어디서든 참조가 가능하다. 함수를 호출하면 암묵적으로 `arguments`, `this`가 함수 내부에 전달. this가 가리키는 값, **this 바인딩은 함수 호출 방식에 의해 동적으로 결정**된다.
- **this binding**: this와 this가 가리키는 객체를 연결하는 것

```js
// 객체 리터럴 — 메서드 안에서 자기 객체를 어떻게 참조할까?
const circle = {
  radius: 5,
  getDiameter() {
    // this는 메서드를 호출한 객체(circle)를 가리킨다
    return 2 * this.radius;
  },
};
console.log(circle.getDiameter()); // 10
```

```js
// 생성자 함수 — 인스턴스는 아직 만들어지지도 않았다
function Circle(radius) {
  // this는 앞으로 생성할 인스턴스를 가리킨다
  this.radius = radius;
}
Circle.prototype.getDiameter = function () {
  return 2 * this.radius;
};

const circle = new Circle(5);
console.log(circle.getDiameter()); // 10
```

1. 객체 리터럴의 메서드 내부에서의 this는 메서드를 호출한 객체를 가리킨다.
2. 생성자 함수 내부의 this는 생성자 함수가 생성할 인스턴스를 가리킨다.
3. 전역에서 this는 전역 객체 `window`
4. 일반 함수 내부에서의 this는 전역 객체 `window` (strict mode에서는 `undefined`)

```js
// 1 ~ 4 한눈에 보기 (브라우저 환경)
console.log(this); // window ← 3. 전역

function square(number) {
  console.log(this); // window ← 4. 일반 함수 호출 (strict mode면 undefined)
  return number ** 2;
}
square(2);

const person = {
  name: 'Lee',
  getName() {
    console.log(this); // {name: 'Lee', getName: f} ← 1. 메서드 호출
    return this.name;
  },
};
person.getName();

function Person(name) {
  this.name = name;
  console.log(this); // Person {name: 'Lee'} ← 2. 생성자 함수 호출
}
new Person('Lee');
```

### 함수 호출과 this 바인딩

- this 바인딩은 함수가 **어떻게 호출되었는지**에 따라 동적으로 결정된다.
- 렉시컬 스코프와 this 바인딩은 결정 시기가 다름
  ⇒ 상위 스코프를 결정하는 렉시컬 스코프는, 함수 객체가 **생성되는 시점**에 상위 스코프를 결정. 하지만 this는 함수 **호출 시점**에 결정

```js
var value = 1;

const obj = {
  value: 100,
  foo() {
    console.log("foo's this", this); // {value: 100, foo: f}
    console.log("foo's this.value", this.value); // 100

    function bar() {
      console.log("bar's this: ", this); // window
      console.log("bar's this.value: ", this.value); // 1
    }

    bar();
  },
};

obj.foo();
```

- `foo`는 객체 리터럴의 메서드로서 사용 ⇒ 메서드를 호출한 객체를 가리킨다.
- `bar`는 일반 함수로서 사용 ⇒ 전역 객체를 가리킨다.
- 콜백 함수나 중첩 함수도 마찬가지, 어떠한 함수라도 일반 함수라면 this는 전역 객체를 바인딩

```js
var value = 1;

const obj = {
  value: 100,
  foo() {
    console.log("foo's this: ", this); // {value: 100, foo: f}

    // 콜백 함수도 일반 함수로 호출되면 this는 전역 객체
    setTimeout(function () {
      console.log("callback's this: ", this); // window
      console.log("callback's this.value: ", this.value); // 1
    }, 100);
  },
};

obj.foo();
```

⇒ 보통 외부 함수인 메서드와 중첩 함수나 콜백 함수의 this가 일치하지 않는 건 예측하기 어렵게 만듦

- 콜백 함수의 this 바인딩을 메서드의 this 바인딩과 일치시키기 위한 방법

**① 임시 참조 변수 할당 (`that` / `self`)**

```js
var value = 1;

const obj = {
  value: 100,
  foo() {
    // this 바인딩(obj)을 일반 변수에 미리 담아둔다
    const that = this;

    setTimeout(function () {
      // 콜백 안에서는 this 대신 that을 쓴다 (스코프 체인을 통해 참조)
      console.log(that.value); // 100
    }, 100);
  },
};

obj.foo();
```

**② `bind`, `apply`, `call` 같은 `Function.prototype` 메서드 사용**

```js
var value = 1;

const obj = {
  value: 100,
  foo() {
    // 콜백 함수에 명시적으로 this를 바인딩한 새 함수를 넘긴다
    setTimeout(
      function () {
        console.log(this.value); // 100
      }.bind(this),
      100,
    );
  },
};

obj.foo();
```

**③ 화살표 함수를 사용**

```js
var value = 1;

const obj = {
  value: 100,
  foo() {
    // 화살표 함수는 자신만의 this 바인딩을 갖지 않는다
    // → 상위 스코프(foo)의 this를 그대로 참조한다
    setTimeout(() => console.log(this.value), 100); // 100
  },
};

obj.foo();
```

### 메서드 호출

- 메서드 내부의 this에는 **메서드를 호출한 객체**(마침표 앞의 객체)가 바인딩된다.

```js
const person = {
  name: 'Lee',
  getName() {
    return this.name;
  },
};

// 메서드를 호출한 객체는 person
console.log(person.getName()); // 'Lee'
```

- 중요한 건 **메서드를 소유한 객체가 아니라, 메서드를 호출한 객체**라는 점. 메서드는 객체에 포함된 것이 아니라 독립적인 객체이고, 프로퍼티가 그 함수 객체를 가리키고 있을 뿐이다.

```js
const anotherPerson = {
  name: 'Kim',
};

// getName 메서드를 다른 객체의 프로퍼티에 할당
anotherPerson.getName = person.getName;
console.log(anotherPerson.getName()); // 'Kim' ← 호출한 객체가 anotherPerson

// 일반 변수에 할당해서 일반 함수로 호출하면
const getName = person.getName;
console.log(getName()); // '' ← this가 window, window.name은 빈 문자열
```

- 프로토타입 메서드도 동일하다. 결국 **누가 호출했는가**만 본다.

```js
function Person(name) {
  this.name = name;
}

Person.prototype.getName = function () {
  return this.name;
};

const me = new Person('Lee');
console.log(me.getName()); // 'Lee' ← 호출한 객체는 me

Person.prototype.name = 'Kim';
console.log(Person.prototype.getName()); // 'Kim' ← 호출한 객체는 Person.prototype
```

### 생성자 함수에서의 호출

- 생성자 함수 내부의 this에는 생성자 함수가 (미래에) 생성할 **인스턴스**가 바인딩된다.

```js
function Circle(radius) {
  // this는 생성자 함수가 생성할 인스턴스를 가리킨다
  this.radius = radius;
  this.getDiameter = function () {
    return 2 * this.radius;
  };
}

const circle1 = new Circle(5);
const circle2 = new Circle(10);

console.log(circle1.getDiameter()); // 10
console.log(circle2.getDiameter()); // 20
```

- `new` 없이 호출하면 그냥 **일반 함수 호출**이 된다 ⇒ this는 전역 객체.

```js
// new를 빠뜨리면 일반 함수로서 호출된다
const circle3 = Circle(15);

console.log(circle3); // undefined ← 반환문이 없으므로 undefined 반환
console.log(radius); // 15 ← this가 window라서 window.radius 가 되어버렸다
```

### call, apply, bind 메서드에 의한 호출

- `Function.prototype`의 메서드이므로 모든 함수가 상속받아 사용할 수 있다.
- 세 메서드 모두 **this로 사용할 객체를 첫 번째 인수로 명시적으로 전달**한다.

**`call` / `apply`** — 함수를 즉시 호출한다. 인수 전달 방식만 다르다.

```js
function getThisBinding() {
  console.log(arguments);
  return this;
}

const thisArg = { a: 1 };

// this 바인딩 없이 호출 → window
console.log(getThisBinding()); // window

// call: 인수를 쉼표로 나열해서 전달
console.log(getThisBinding.call(thisArg, 1, 2, 3));
// Arguments(3) [1, 2, 3]
// {a: 1}

// apply: 인수를 배열로 묶어서 전달
console.log(getThisBinding.apply(thisArg, [1, 2, 3]));
// Arguments(3) [1, 2, 3]
// {a: 1}
```

- 대표적인 용도: **유사 배열 객체에 배열 메서드를 빌려 쓰는 것**

```js
function convertArgsToArray() {
  // arguments는 유사 배열 객체라서 Array.prototype.slice가 없다
  // → slice를 arguments에 바인딩해서 호출 = 배열로 복사
  const arr = Array.prototype.slice.call(arguments);
  return arr;
}

console.log(convertArgsToArray(1, 2, 3)); // [1, 2, 3]
```

**`bind`** — 함수를 호출하지 않고, **this가 고정된 새로운 함수를 반환**한다.

```js
function getThisBinding() {
  return this;
}

const thisArg = { a: 1 };

console.log(getThisBinding.bind(thisArg)); // getThisBinding 함수 자체(호출되지 않음)
console.log(getThisBinding.bind(thisArg)()); // {a: 1} ← 명시적으로 호출해야 한다
```

- 그래서 앞서 본 **콜백 함수의 this 불일치 문제**를 푸는 데 쓰인다.

```js
const person = {
  name: 'Lee',
  foo(callback) {
    // ①
    setTimeout(callback, 100); // ②
  },
};

person.foo(function () {
  // ③ 일반 함수로 호출되므로 this는 window
  console.log(`Hi! my name is ${this.name}.`); // Hi! my name is .
});
```

```js
const person = {
  name: 'Lee',
  foo(callback) {
    // 콜백 함수에 person을 미리 바인딩해서 넘긴다
    setTimeout(callback.bind(this), 100);
  },
};

person.foo(function () {
  console.log(`Hi! my name is ${this.name}.`); // Hi! my name is Lee.
});
```

### 정리 — 함수 호출 방식과 this 바인딩

| 함수 호출 방식 | this 바인딩 |
| --- | --- |
| 일반 함수 호출 | 전역 객체 (strict mode에서는 `undefined`) |
| 메서드 호출 | 메서드를 호출한 객체 (마침표 앞의 객체) |
| 생성자 함수 호출 | 생성자 함수가 (미래에) 생성할 인스턴스 |
| `call` / `apply` / `bind` 에 의한 간접 호출 | 첫 번째 인수로 전달한 객체 |
