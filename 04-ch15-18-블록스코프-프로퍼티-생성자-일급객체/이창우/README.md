# 모던 자바스크립트 Deep Dive 15~18장 정리

15장부터 18장까지는 "선언과 객체 생성을 어디까지 통제할 수 있는가"를 다룬다. `let` / `const`로 스코프와 재할당을 통제하고, 프로퍼티 어트리뷰트로 객체의 변경을 통제하고, 생성자 함수로 객체 생성 과정을 통제한다.

## 15장. let, const 키워드와 블록 레벨 스코프

### var 키워드의 문제점

**1. 변수 중복 선언 가능**

```js
var x = 1;
var x = 100; // 에러 없이 재선언 + 재할당된다

var y = 1;
var y; // 초기화문이 없으면 무시된다

console.log(x); // 100
console.log(y); // 1
```

이미 선언된 변수를 다시 선언해도 에러가 나지 않는다. 코드 규모가 커지면 의도치 않게 앞의 값을 덮어쓰기 쉽다.

**2. 함수 레벨 스코프**

```js
var x = 1;

if (true) {
  var x = 10; // 별개의 변수가 아니라 위의 x를 덮어쓴 것
}

for (var i = 0; i < 3; i++) {}

console.log(x); // 10
console.log(i); // 3: 반복문이 끝나도 남아 있다
```

`var`는 **함수 코드 블록만** 지역 스코프로 인정한다. `if`, `for`, `while` 안에서 선언해도 함수 밖이라면 전역 변수가 된다. 스코프가 넓어질수록 관리가 어려워진다.

**3. 변수 호이스팅**

```js
console.log(foo); // undefined — 에러가 아니다
var foo = 123;
console.log(foo); // 123
```

선언이 스코프 선두로 끌어 올려진 것처럼 동작해서, 선언 이전에 참조해도 에러 없이 `undefined`가 나온다. 에러가 안 난다는 게 오히려 문제다. 잘못된 코드가 조용히 동작한다.

### let 키워드

`var`의 단점을 보완하기 위해 ES6에서 도입됐다.

**1. 변수 중복 선언 금지**

```js
let bar = 123;
// let bar = 456; // SyntaxError: Identifier 'bar' has already been declared
```

**2. 블록 레벨 스코프**

```js
let x = 1;

if (true) {
  let x = 10; // 블록 안에서만 유효한 별개의 변수
  console.log(x); // 10
}

console.log(x); // 1
```

함수뿐 아니라 `if`, `for`, `while`, `try/catch` 등 **모든 코드 블록**을 지역 스코프로 인정한다. 코드 블록 하나하나가 스코프 레이어가 된다.

```text
전역 스코프 { x = 1 }
   └─ 블록 스코프 { x = 10 }   ← 블록을 벗어나면 소멸
```

**3. 호이스팅 대응 — 선언 단계와 초기화 단계의 분리**

변수는 **선언 단계**에서 실행 컨텍스트의 렉시컬 환경에 식별자를 등록해 엔진에게 "이 변수가 존재한다"고 알린다. `var`와 `let`은 이 이후가 다르다.

- `var`: 선언 단계와 초기화 단계가 **한 번에** 진행된다 → 선언 전 참조 시 `undefined`
- `let`: 선언 단계와 초기화 단계가 **분리**된다 → 초기화 전 참조 시 `ReferenceError`

```js
// var
console.log(a); // undefined
var a = 1;

// let
// console.log(b); // ReferenceError: Cannot access 'b' before initialization
let b = 1;
```

선언은 됐지만 초기화되지 않아 접근할 수 없는 구간을 **일시적 사각지대(TDZ, Temporal Dead Zone)** 라고 한다.

```text
스코프 시작
  │   ← TDZ: 식별자는 등록됐지만 접근하면 ReferenceError
let b = 1;   ← 초기화, 여기부터 접근 가능
  │
스코프 끝
```

⇒ `let`이 호이스팅되지 않는 것이 아니다. 호이스팅은 되지만 초기화 전 접근을 **의도적으로 막는다.**

**4. 전역 객체의 프로퍼티가 아니다**

```js
var x = 1;
let y = 2;

console.log(window.x); // 1
console.log(window.y); // undefined
```

`var` 전역 변수는 전역 객체의 프로퍼티가 되지만 `let`은 아니다. 14장에서 본 전역 오염 문제를 한 겹 더 막아준다.

### const 키워드

`let`의 특징을 그대로 가지면서 두 가지가 추가된다.

**1. 선언과 초기화를 반드시 동시에 해야 한다**

```js
const x = 1;
// const y; // SyntaxError: Missing initializer in const declaration
```

**2. 재할당이 금지된다**

```js
const x = 1;
// x = 2; // TypeError: Assignment to constant variable.
```

다만 **재할당이 금지될 뿐, 불변을 보장하지는 않는다.** `const`가 막는 것은 "변수가 가리키는 값을 바꾸는 것"이지 "그 값의 내용을 바꾸는 것"이 아니다.

```js
const person = { name: 'Lee' };

person.name = 'Kim'; // 가능 — 참조 값은 그대로다
console.log(person); // { name: 'Kim' }

// person = {}; // TypeError — 참조 값 자체를 바꾸는 건 불가
```

객체까지 얼리고 싶다면 16장의 `Object.freeze`가 필요하다.

⇒ 정리하면 **기본은 `const`, 재할당이 필요할 때만 `let`, `var`는 쓰지 않는다.** 재할당이 필요한지 아닌지가 코드에 그대로 드러나는 것이 장점이다.

### 15장 핵심 흐름

```text
var  → 중복 선언 O · 함수 레벨 스코프 · 선언+초기화 동시 (undefined) · 전역 객체 프로퍼티
let  → 중복 선언 X · 블록 레벨 스코프 · 선언/초기화 분리 (TDZ)     · 전역 객체 프로퍼티 X
const→ let의 특징 + 선언 시 초기화 필수 + 재할당 금지 (내용 변경은 가능)
```

## 16장. 프로퍼티 어트리뷰트

### 내부 슬롯과 내부 메서드

내부 슬롯과 내부 메서드는 자바스크립트 엔진의 구현 알고리즘을 설명하기 위해 명세에서 정의한 **의사 프로퍼티 / 의사 메서드**다. `[[...]]` 형태로 표기한다.

원칙적으로 직접 접근할 수 없지만, 일부는 간접적인 접근 수단을 제공한다.

```js
const o = {};

// o.[[Prototype]]     // 문법적으로 접근 불가
console.log(o.__proto__); // Object.prototype — 간접 접근은 가능
```

### 프로퍼티 어트리뷰트와 프로퍼티 디스크립터

엔진은 프로퍼티를 생성할 때 그 상태를 나타내는 **프로퍼티 어트리뷰트**를 자동으로 정의한다. 어트리뷰트도 내부 슬롯이다.

`Object.getOwnPropertyDescriptor` / `Object.getOwnPropertyDescriptors`로 간접 확인할 수 있다.

```js
const person = { name: 'Lee' };

console.log(Object.getOwnPropertyDescriptor(person, 'name'));
// { value: 'Lee', writable: true, enumerable: true, configurable: true }

person.age = 20;

console.log(Object.getOwnPropertyDescriptors(person));
// {
//   name: { value: 'Lee', writable: true, enumerable: true, configurable: true },
//   age:  { value: 20,    writable: true, enumerable: true, configurable: true }
// }
```

### 데이터 프로퍼티와 접근자 프로퍼티

프로퍼티는 두 종류로 나뉜다.

**데이터 프로퍼티** — 키와 값으로 구성된 일반적인 프로퍼티

| 어트리뷰트 | 디스크립터 | 설명 |
| --- | --- | --- |
| `[[Value]]` | `value` | 프로퍼티 키로 접근했을 때 반환되는 값 |
| `[[Writable]]` | `writable` | 값 변경 가능 여부 |
| `[[Enumerable]]` | `enumerable` | 열거 가능 여부 (`for...in`, `Object.keys`) |
| `[[Configurable]]` | `configurable` | 재정의 / 삭제 가능 여부 |

**접근자 프로퍼티** — 자체 값을 갖지 않고, 값을 읽거나 저장할 때 호출되는 **접근자 함수**로 구성된다.

| 어트리뷰트 | 디스크립터 | 설명 |
| --- | --- | --- |
| `[[Get]]` | `get` | 값을 읽을 때 호출되는 함수 |
| `[[Set]]` | `set` | 값을 저장할 때 호출되는 함수 |
| `[[Enumerable]]` | `enumerable` | 열거 가능 여부 |
| `[[Configurable]]` | `configurable` | 재정의 / 삭제 가능 여부 |

```js
const person = {
  firstName: 'Ungmo', // 데이터 프로퍼티
  lastName: 'Lee',    // 데이터 프로퍼티

  // fullName은 접근자 프로퍼티
  get fullName() {
    return `${this.firstName} ${this.lastName}`;
  },
  set fullName(name) {
    [this.firstName, this.lastName] = name.split(' ');
  },
};

console.log(person.fullName); // Ungmo Lee — [[Get]] 호출
person.fullName = 'Heegun Lee'; // [[Set]] 호출
console.log(person.firstName); // Heegun

console.log(Object.getOwnPropertyDescriptor(person, 'fullName'));
// { get: f, set: f, enumerable: true, configurable: true }
```

### 프로퍼티 정의

새 프로퍼티를 추가하면서 어트리뷰트를 명시적으로 정의하거나, 기존 프로퍼티의 어트리뷰트를 재정의하는 것.

```js
const person = {};

Object.defineProperty(person, 'firstName', {
  value: 'Ungmo',
  writable: true,
  enumerable: true,
  configurable: true,
});

// 생략한 어트리뷰트는 기본값(false / undefined)이 된다
Object.defineProperty(person, 'lastName', {
  value: 'Lee',
});

console.log(Object.getOwnPropertyDescriptor(person, 'lastName'));
// { value: 'Lee', writable: false, enumerable: false, configurable: false }

person.lastName = 'Kim'; // 무시된다 (strict mode에서는 TypeError)
console.log(Object.keys(person)); // ['firstName'] — lastName은 열거되지 않는다
delete person.lastName; // 무시된다
```

⇒ 객체 리터럴로 만든 프로퍼티는 어트리뷰트가 전부 `true`지만, `defineProperty`로 만들면 생략한 값이 전부 `false`가 된다.

### 객체 변경 방지

객체는 변경 가능한 값이라 프로퍼티 추가·삭제·값 변경·어트리뷰트 재정의가 모두 가능하다. 이를 단계별로 막는 메서드가 있다.

| 메서드 | 추가 | 삭제 | 값 읽기 | 값 쓰기 | 어트리뷰트 재정의 |
| --- | --- | --- | --- | --- | --- |
| `Object.preventExtensions` | ✕ | ○ | ○ | ○ | ○ |
| `Object.seal` | ✕ | ✕ | ○ | ○ | ✕ |
| `Object.freeze` | ✕ | ✕ | ○ | ✕ | ✕ |

```js
const person = { name: 'Lee', address: { city: 'Seoul' } };

Object.freeze(person);
console.log(Object.isFrozen(person)); // true

person.age = 20;      // 무시
person.name = 'Kim';  // 무시
delete person.name;   // 무시
console.log(person); // { name: 'Lee', address: { city: 'Seoul' } }

// 단, 얕은 변경 방지다 — 중첩 객체는 얼지 않는다
person.address.city = 'Busan';
console.log(person.address.city); // Busan
```

중첩 객체까지 얼리려면 재귀적으로 `freeze`를 호출해야 한다.

```js
function deepFreeze(target) {
  if (target && typeof target === 'object' && !Object.isFrozen(target)) {
    Object.freeze(target);
    Object.keys(target).forEach(key => deepFreeze(target[key]));
  }
  return target;
}
```

### 16장 핵심 흐름

```text
프로퍼티
  ├─ 데이터 프로퍼티   [[Value]] [[Writable]] [[Enumerable]] [[Configurable]]
  └─ 접근자 프로퍼티   [[Get]] [[Set]] [[Enumerable]] [[Configurable]]
       ↓ 확인
Object.getOwnPropertyDescriptor(s)
       ↓ 정의
Object.defineProperty / defineProperties
       ↓ 변경 방지 (강도 순)
preventExtensions → seal → freeze   ※ 모두 얕은 변경 방지
```

## 17장. 생성자 함수에 의한 객체 생성

### 언제 생성자 함수를 쓰는가

객체를 하나 만들 때는 객체 리터럴이 훨씬 간편하다. 특별한 이유가 없다면 생성자 함수를 쓰지 않는다. 그럼 어떤 경우에 쓰는 게 나을까?

**1. 같은 형태의 객체가 여럿 필요한데 상태를 구분해야 할 때**

```js
// 객체 리터럴 — 프로퍼티 구조가 같은데도 매번 반복해서 써야 한다
const circle1 = {
  radius: 5,
  getDiameter() {
    return 2 * this.radius;
  },
};

const circle2 = {
  radius: 10,
  getDiameter() {
    return 2 * this.radius;
  },
};

// 생성자 함수 — 구조는 템플릿으로 두고 상태만 다르게 만든다
function Circle(radius) {
  this.radius = radius;
  this.getDiameter = function () {
    return 2 * this.radius;
  };
}

const circle3 = new Circle(5);
const circle4 = new Circle(10);

console.log(circle3.getDiameter()); // 10
console.log(circle4.getDiameter()); // 20
```

**2. this 참조를 의도적으로 결정할 때**

`this`는 함수를 **어떻게 호출했는지**에 따라 달라진다.

| 호출 방식 | this가 가리키는 값 |
| --- | --- |
| 일반 함수로서 호출 | 전역 객체 |
| 메서드로서 호출 | 메서드를 호출한 객체 |
| 생성자 함수로서 호출 | 생성자 함수가 생성할 인스턴스 |

```js
function foo() {
  console.log(this);
}

foo(); // window (전역 객체)

const obj = { foo };
obj.foo(); // obj

const inst = new foo(); // inst (새로 생성된 인스턴스)
```

`new` 키워드가 있으면 생성된 인스턴스가 반환값으로 쓰이고, 없으면 일반 함수처럼 동작한다.

### 생성자 함수의 인스턴스 생성 과정

생성자 함수의 역할은 프로퍼티 구조가 동일한 인스턴스를 만드는 **템플릿**으로 동작하는 것이다. 목적은 ① 인스턴스 생성과 ② 인스턴스 초기화다. **생성은 항상 하지만, 초기화는 옵션이다.**

```js
function Circle(radius) {
  // 1. 암묵적으로 빈 객체가 생성되고 this에 바인딩된다

  // 2. this에 바인딩된 인스턴스를 초기화한다
  this.radius = radius;
  this.getDiameter = function () {
    return 2 * this.radius;
  };

  // 3. 완성된 인스턴스가 바인딩된 this가 암묵적으로 반환된다
}

const circle = new Circle(5);
console.log(circle); // Circle { radius: 5, getDiameter: f }
```

**1. 인스턴스 생성과 this 바인딩**

암묵적으로 빈 객체가 생성되고, 이 객체가 인스턴스가 된다. 그리고 인스턴스에 `this`가 바인딩된다. 이 처리는 **함수 몸체 코드가 실행되기 이전, 런타임 이전**에 일어난다. 생성자 함수 내부의 `this`가 인스턴스를 가리키는 이유다.

**2. 인스턴스 초기화**

함수 몸체의 코드가 한 줄씩 실행되며 `this`에 바인딩된 인스턴스에 프로퍼티와 메서드를 추가하고, 인수로 받은 초기값을 할당한다.

**3. 인스턴스 반환**

모든 처리가 끝나면 `this`가 암묵적으로 반환된다.

```js
function Circle(radius) {
  this.radius = radius;
  return {}; // 명시적으로 객체를 반환하면 this 반환이 무시된다
}
console.log(new Circle(5)); // {}

function Square(size) {
  this.size = size;
  return 100; // 원시 값 반환은 무시되고 this가 반환된다
}
console.log(new Square(5)); // Square { size: 5 }
```

⇒ 왜 원시 값 반환만 의도적으로 무시할까? `new` 연산자의 목적이 인스턴스를 반환하는 것이기 때문이다. 더 정확히는 내부 메서드 `[[Construct]]`의 **반환 타입 자체가 객체로 정의**되어 있기 때문. 그래서 생성자 함수 안에서는 `return` 문을 아예 쓰지 않는 것이 관례다.

### 내부 메서드 [[Call]]과 [[Construct]]

함수는 일반 함수로 호출할 수도 있고 생성자 함수로 호출할 수도 있다. 함수도 객체이므로 일반 객체와 동일하게 내부 슬롯과 내부 메서드를 갖는다. 다만 **일반 객체는 호출할 수 없고 함수는 호출할 수 있는데**, `[[Call]]`, `[[Construct]]` 같은 내부 메서드를 추가로 가지고 있기 때문이다.

- `[[Call]]`을 갖는 함수 객체 = **callable**: 호출할 수 있는 객체
- `[[Construct]]`를 갖는 함수 객체 = **constructor**: 생성자 함수로서 호출할 수 있는 객체

**모든 함수 객체는 callable이지만, 모든 함수 객체가 constructor인 것은 아니다.**

```js
function foo() {}
foo();     // 일반 함수로서 호출 → [[Call]]
new foo(); // 생성자 함수로서 호출 → [[Construct]]
```

constructor / non-constructor는 함수가 **어디에 할당되어 있는지가 아니라 어떻게 정의됐는지**로 구분된다.

```js
// constructor: 함수 선언문, 함수 표현식, 클래스
function normal() {}
const expression = function () {};

// non-constructor: 메서드 축약 표현, 화살표 함수
const obj = {
  method() {}, // ES6 메서드 축약 표현
};
const arrow = () => {};

new normal();     // OK
new expression(); // OK
// new obj.method(); // TypeError: obj.method is not a constructor
// new arrow();      // TypeError: arrow is not a constructor

// 반면 프로퍼티 값으로 할당된 일반 함수 표현식은 여전히 constructor다
const obj2 = { method: function () {} };
new obj2.method(); // OK
```

⇒ 자바스크립트에서 "메서드"란 **ES6 메서드 축약 표현만** 의미한다.

### new 없이 생성자 함수를 호출하면

```js
function Person(name) {
  this.name = name;
}

const p = Person('Kim'); // new 없이 호출 → 일반 함수 호출

console.log(p);        // undefined — 반환값이 없다
console.log(window.name); // Kim — this가 전역 객체를 가리켜 전역이 오염됐다
```

일반 함수와 생성자 함수는 형식적인 차이가 없다. 그래서 **생성자 함수는 파스칼 케이스로 짓는 것이 관례**지만, 관례는 강제가 아니다.

### new.target

앞의 실수를 코드 레벨에서 막기 위해 ES6에 도입된 문법이다. 함수 내부에서 `new.target`을 사용하면 생성자 함수로서 호출됐는지 확인할 수 있다.

- 생성자 함수로서 호출 → 함수 자신
- 일반 함수로서 호출 → `undefined`

```js
function Circle(radius) {
  // new 없이 호출되면 new 붙여서 재귀 호출
  if (!new.target) {
    return new Circle(radius);
  }
  this.radius = radius;
}

const circle = Circle(5); // new 없이 호출해도
console.log(circle.radius); // 5 — 정상 동작한다
```

⇒ 그러면 `new.target`은 무엇을 보고 판단하는 걸까? 엔진이 `[[Construct]]`를 호출할 때 NewTarget 값을 함께 넘기고, 그 값을 그대로 읽는 것이다.

```text
new Person()
     ↓
① Person에 [[Construct]] 있나?
     ↓ yes
② [[Construct]] 호출
     ↓
③ NewTarget = Person 전달
     ↓
④ 함수 내부에서 new.target → Person

/*
  new 있음 : Person.[[Construct]]([], Person)  ← 세 번째 인자가 NewTarget
  new 없음 : Person.[[Call]](thisArg, [])      ← NewTarget 자체가 없다 → undefined
*/
```

`new.target`을 못 쓰는 환경이라면 스코프 세이프 생성자 패턴으로 대체한다.

```js
function Circle(radius) {
  if (!(this instanceof Circle)) {
    return new Circle(radius);
  }
  this.radius = radius;
}
```

빌트인 생성자 함수도 이 검사를 내부적으로 하고 있다.

```js
// Object, Function: new 없이 호출해도 동일하게 동작
console.log(Object() instanceof Object);   // true
console.log(new Object() instanceof Object); // true

// String, Number, Boolean: new 유무에 따라 결과가 다르다
console.log(typeof Number('123'));     // number — 타입 변환
console.log(typeof new Number('123')); // object — 래퍼 객체
```

### 17장 핵심 흐름

```text
new Person("Kim")
       ↓
Person.[[Construct]](["Kim"], Person)
       ↓
빈 객체 생성
       ↓
새 객체의 프로토타입 연결   // 19장에서 이어짐
       ↓
this ← 새 객체 (런타임 이전)
       ↓
함수 몸체 실행 → 인스턴스 초기화
       ↓
return 결과 검사 (객체면 그것을, 원시 값이면 무시하고 this)
       ↓
인스턴스 반환
```

⇒ 내부적으로 일반 함수와 다르게 엔진이 돌고 있다. "빈 객체가 생성되고 this에 바인딩된다"는 말이 이 과정이다.

## 18장. 함수와 일급 객체

### 일급 객체의 조건

1. 무명의 리터럴로 생성할 수 있다 (런타임 생성이 가능하다)
2. 변수나 자료구조(객체, 배열)에 저장할 수 있다
3. 함수의 매개변수에 전달할 수 있다
4. 함수의 반환값으로 사용할 수 있다

자바스크립트 함수는 위 조건을 모두 만족하는 **일급 객체**다.

```js
// 1. 무명의 리터럴로 생성 + 2. 변수에 저장
const increase = function (num) {
  return ++num;
};
const decrease = function (num) {
  return --num;
};

// 2. 객체에 저장
const predicates = { increase, decrease };

// 3. 매개변수로 전달 + 4. 반환값으로 사용
function makeCounter(predicate) {
  let num = 0;
  return function () {
    num = predicate(num);
    return num;
  };
}

const increaser = makeCounter(predicates.increase);
console.log(increaser()); // 1
console.log(increaser()); // 2
```

함수는 값을 사용할 수 있는 곳(변수 할당문, 객체의 프로퍼티 값, 배열 요소, 함수 호출의 인수, 함수 반환문) 어디서든 쓸 수 있고 런타임에 함수 객체로 평가된다. 자바스크립트에서 함수형 프로그래밍이 가능한 이유다.

다만 함수 객체는 일반 객체에 없는 고유 프로퍼티와 내부 메서드(`[[Call]]`, `[[Construct]]`)를 갖는다.

### 함수 객체의 프로퍼티

```js
function square(number) {
  return number * number;
}

console.log(Object.getOwnPropertyDescriptors(square));
// length:    { value: 1,        writable: false, enumerable: false, configurable: true }
// name:      { value: 'square', writable: false, enumerable: false, configurable: true }
// arguments: { value: null,     ... }
// caller:    { value: null,     ... }
// prototype: { value: {...},    writable: true,  enumerable: false, configurable: false }

// __proto__는 square의 프로퍼티가 아니라 Object.prototype에서 상속받은 것이다
console.log(Object.getOwnPropertyDescriptor(square, '__proto__')); // undefined
console.log(Object.getOwnPropertyDescriptor(Object.prototype, '__proto__')); // { get: f, set: f, ... }
```

`length`, `name`, `arguments`, `caller`, `prototype`은 함수 객체의 고유 프로퍼티다. (`arguments`, `caller`는 표준에서 폐지된 프로퍼티라 사용하지 않는다.)

**length와 name**

```js
function foo(x, y) {}
console.log(foo.length); // 2 — 매개변수 개수

function bar() {}
console.log(bar.name); // bar

const anonymous = function () {};
console.log(anonymous.name); // anonymous — ES6부터는 변수 이름이 들어간다 (ES5에서는 '')

console.log((function () {}).name); // '' — 즉시 실행 함수는 빈 문자열
```

`length`는 **매개변수** 개수이고, `arguments.length`는 **인수** 개수다. 두 값이 다를 수 있다.

### arguments 프로퍼티

`arguments` 객체는 함수 호출 시 전달된 인수들의 정보를 담은 **순회 가능한(iterable) 유사 배열 객체**로, 함수 내부에서 지역 변수처럼 사용된다.

```js
function multiply(x, y) {
  console.log(arguments);
  return x * y;
}

console.log(multiply());        // NaN — [Arguments] {}
console.log(multiply(1));       // NaN — [Arguments] { '0': 1 }
console.log(multiply(1, 2));    // 2   — [Arguments] { '0': 1, '1': 2 }
console.log(multiply(1, 2, 3)); // 2   — [Arguments] { '0': 1, '1': 2, '2': 3 }
```

매개변수보다 인수를 적게 전달하면 나머지 매개변수는 `undefined`로 유지되고, 많이 전달하면 무시된다. ⇒ 무시된다고 버려지는 게 아니라 `arguments` 객체의 프로퍼티로 그대로 보관된다.

**유사 배열 객체이면서 동시에 이터러블**

`arguments`는 `length` 프로퍼티가 있어 `for` 문으로 순회할 수 있는 유사 배열 객체다. 여기에 ES6부터 `Symbol.iterator` 메서드를 가져 이터러블이 되었다(34장).

```js
function sum() {
  let res = 0;
  for (let i = 0; i < arguments.length; i++) {
    res += arguments[i]; // 유사 배열 객체로서 순회
  }
  return res;
}
console.log(sum(1, 2, 3)); // 6

function sum2() {
  return [...arguments].reduce((a, b) => a + b, 0); // 이터러블이라 스프레드 가능
}
console.log(sum2(1, 2, 3)); // 6
```

유사 배열 객체는 배열이 아니므로 배열 메서드를 그대로 쓸 수 없다. 그래서 `Function.prototype.call` / `apply`로 우회해야 했다.

```js
function sum3() {
  const array = Array.prototype.slice.call(arguments); // 배열로 변환
  return array.reduce((a, b) => a + b, 0);
}
console.log(sum3(1, 2, 3)); // 6
```

이 번거로움을 해소하려고 나온 것이 ES6의 **Rest 파라미터**다.

```js
function sum4(...args) {
  return args.reduce((a, b) => a + b, 0); // 처음부터 진짜 배열
}
console.log(sum4(1, 2, 3)); // 6
```

참고로 유사 배열 객체는 이렇게 직접 만들 수도 있다.

```js
const obj = {
  0: 'a',
  1: 'b',
  length: 2,

  [Symbol.iterator]() {
    let index = 0;
    const self = this;

    return {
      next() {
        return index < self.length
          ? { value: self[index++], done: false }
          : { value: undefined, done: true };
      },
    };
  },
};

console.log([...obj]); // ['a', 'b'] — 유사 배열 객체이면서 이터러블
```

### __proto__ 접근자 프로퍼티

모든 객체는 `[[Prototype]]`이라는 내부 슬롯을 갖는다. 객체지향 프로그래밍의 상속을 구현하는 프로토타입 객체를 가리킨다.

`__proto__`는 이 내부 슬롯이 가리키는 프로토타입 객체에 접근하기 위한 **접근자 프로퍼티**다. 내부 슬롯에는 직접 접근할 수 없어 우회 수단으로 쓰인다.

```js
const obj = {};

console.log(obj.__proto__ === Object.prototype); // true

// __proto__는 obj의 프로퍼티가 아니라 Object.prototype이 소유한 접근자 프로퍼티다
console.log(obj.hasOwnProperty('__proto__')); // false
```

### prototype 프로퍼티

`prototype`은 **constructor만 소유하는** 프로퍼티다. 즉 내부 메서드 `[[Construct]]`를 갖는 함수 객체만 가진다.

```js
// constructor
console.log((function () {}).hasOwnProperty('prototype')); // true

// non-constructor
console.log((() => {}).hasOwnProperty('prototype'));       // false
console.log(({ method() {} }).method.hasOwnProperty('prototype')); // false

// 일반 객체도 당연히 없다
console.log({}.hasOwnProperty('prototype')); // false
```

함수가 생성자 함수로 호출될 때, `prototype` 프로퍼티는 **생성자 함수가 생성할 인스턴스의 프로토타입 객체**를 가리킨다.

```js
function Person(name) {
  this.name = name;
}

const me = new Person('Lee');

console.log(Person.prototype === Object.getPrototypeOf(me)); // true
console.log(me.__proto__ === Person.prototype);              // true
```

⇒ `__proto__`는 **모든 객체**가 (상속받아) 쓰는 "내 프로토타입은 누구인가", `prototype`은 **생성자 함수만** 갖는 "내가 만들 인스턴스의 프로토타입은 누구인가". 자세한 내용은 19장에서 이어진다.

### 18장 핵심 흐름

```text
함수 = 일급 객체
  ├─ 값처럼 다룰 수 있다 (리터럴 생성 · 저장 · 인수 전달 · 반환)
  └─ 일반 객체에 없는 것
       ├─ 내부 메서드 [[Call]] / [[Construct]]
       └─ 고유 프로퍼티
            ├─ length     : 매개변수 개수 (≠ arguments.length)
            ├─ name       : 함수 이름 (ES6부터 익명 함수도 채워짐)
            ├─ arguments  : 유사 배열 객체 + 이터러블 → Rest 파라미터로 대체
            └─ prototype  : constructor만 소유, 인스턴스의 프로토타입 → 19장
```
