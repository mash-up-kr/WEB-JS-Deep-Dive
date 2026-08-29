# 모던 자바스크립트 Deep Dive 15~18장 정리

## 15장. let, const 키워드와 블록 레벨 스코프

### `var` 키워드로 선언한 변수의 문제점

`var`는 블록 스코프를 만들지 않고 가장 가까운 함수 스코프에 바인딩된다.  
함수 바깥에서는 일반 스크립트의 경우 전역 스코프에, ES6 모듈의 경우 모듈 스코프에 바인딩된다.

### 함수 레벨 스코프

함수 레벨 스코프는 함수 단위로 변수의 유효 범위를 결정하는 방식이다.
`var`로 선언한 변수는 `if`, `for`, `while` 같은 블록을 스코프로 인정하지 않고 가장 가까운 함수에 속한다.

```js
function example() {
  if (true) {
    var value = 10;
  }

  console.log(value); // 10
}

example();
```

반면 `let`과 `const`는 블록 레벨 스코프를 사용하므로 블록 밖에서 접근할 수 없다.

```js
function example() {
  if (true) {
    let value = 10;
    const anotherValue = 20;
  }

  // console.log(value);        // ReferenceError
  // console.log(anotherValue);  // ReferenceError
}
```

```js
if (true) {
  var value = 10; // 사라지지 않는다
}

console.log(value); // 10
```

같은 스코프에서 `var`로 변수를 중복 선언해도 에러가 발생하지 않는다.
실수로 값을 재할당할 수 있어 `let`과 `const`가 도입되었다.

```js
var count = 1;
var count = 2;

console.log(count); // 2
```

### 변수 호이스팅

> 실행 컨텍스트는 최초 실행될때, environmentRecord에 현재 문맥의 식별자 정보를 수집한다.  
> 현재 컨텍스트가 식별자 정보를 수집해서 environmentRecord에 담는 과정을 **호이스팅** 이라 한다.

> const user = {};          // 변수 식별자  
> function add() {}         // 함수 식별자  
> class Person {}           // 클래스 식별자  
> function sum(x, y) {}     // 매개변수 식별자  

`var`로 선언한 변수는 선언과 초기화가 먼저 진행되는 것처럼 동작한다.
따라서 선언문 이전에 참조하면 `undefined`가 반환된다.

```js
console.log(foo); // undefined
var foo = 1;
```

실제로는 다음과 비슷하게 동작한다고 이해할 수 있다.

```js
var foo;
console.log(foo); // undefined
foo = 1;
```

### `let`과 일시적 사각지대(TDZ)

> TDZ(Temporal Dead Zone): 선언만 되고 아직 초기화 되지 않는 변수가 머무는 공간

`let`과 `const`도 선언 자체는 스코프의 시작 지점에서 인식되지만,
선언문에 도달하기 전까지는 초기화되지 않는다. 이 구간을 **일시적 사각지대(TDZ)**라고 한다.

```js
console.log(foo); // ReferenceError
let foo = 1;
```

`var`처럼 `undefined`가 출력되지 않는 이유는 선언 전에 변수에 접근했기 때문이다.

### `let`과 `const`의 블록 레벨 스코프

`let`과 `const`는 `if`, `for`, `while` 같은 문들을 블록 단위로 스코프를 만든다. (switch는 별도의 블록 레벨 스코프를 만든다고 함.)

```js
let result = 'global';

if (true) {
  let result = 'block';
  console.log(result); // 'block'
}

console.log(result); // 'global'
```

### 전역 객체와 `let`

전역 코드에서 `var`로 선언한 변수는 브라우저의 전역 객체 프로퍼티가 될 수 있지만,
`let`과 `const`로 선언한 전역 변수는 전역 객체의 프로퍼티가 아니다.

```js
var a = 1;
let b = 2;

console.log(window.a); // 1
console.log(window.b); // undefined
```

```markdown
- 일반 스크립트의 var → 전역 객체 프로퍼티
- 일반 스크립트의 let, const → 전역 스코프에는 있지만 전역 객체 프로퍼티는 아님
- ES6 모듈의 var, let, const → 모듈 스코프
```


### `const`와 상수

`const`로 선언한 변수는 반드시 선언과 동시에 초기화해야 하며 재할당할 수 없다.

```js
const taxRate = 0.1;
// taxRate = 0.2; // TypeError
```

단, `const`가 객체의 내부 변경까지 막는 것은 아니다.

```js
const user = { name: 'Lee' };

user.name = 'Kim'; // 가능
// user = {};      // 재할당은 불가능
```

실무에서는 기본적으로 `const`를 사용하고, 재할당이 필요한 경우에만 `let`을 사용하는 방식이 안전하다.

## 16장. 프로퍼티 어트리뷰트

> [이전의 프로토타입 정리](https://github.com/jeongwoo903/study-log/blob/main/2025/02/%5BJS%5D%20%EB%8B%A4%EC%8B%9C%20%ED%95%9C%20%EB%B2%88%20JS%3A%20%ED%94%84%EB%A1%9C%ED%86%A0%ED%83%80%EC%9E%85.md)

### 내부 슬롯과 내부 메서드

자바스크립트 엔진은 객체의 동작을 구현하기 위해 내부 슬롯과 내부 메서드를 사용한다.
`[[Value]]`, `[[Writable]]`처럼 대괄호로 표시되는 내부 정보는 일반 코드에서 직접 접근할 수 없다.

```js
const object = {};

// object.[[Prototype]] // 직접 접근할 수 없는 내부 슬롯 표기
object.__proto__;        // 프로토타입에 접근하는 비표준 레거시 접근 방식
```
내부 슬롯은 일반 코드에서 직접 접근할 수 없다.
> 다만 `Object.getPrototypeOf`처럼 특정 내부 슬롯의 상태를 확인할 수 있도록 제공된 표준 메서드가 있다.  
> `Object.getOwnPropertyDescriptor`는 내부 슬롯 전체가 아니라 프로퍼티의 디스크립터 정보를 확인하는 메서드다.

### 데이터 프로퍼티

값을 직접 저장하는 일반적인 프로퍼티다. 데이터 프로퍼티는 다음 네 가지 어트리뷰트를 가진다.

| 내부 어트리뷰트 | 디스크립터 키 | 의미 |
| --- | --- | --- |
| `[[Value]]` | `value` | 프로퍼티가 저장하는 값 |
| `[[Writable]]` | `writable` | `true`이면 값을 변경할 수 있음 |
| `[[Enumerable]]` | `enumerable` | `true`이면 `Object.keys`나 `for...in`으로 열거할 수 있음 |
| `[[Configurable]]` | `configurable` | `true`이면 프로퍼티 삭제 및 어트리뷰트 변경이 가능함 |

```js
const person = { name: 'Lee' };

console.log(Object.getOwnPropertyDescriptor(person, 'name'));
// { value: 'Lee', writable: true, enumerable: true, configurable: true }
```

### 접근자 프로퍼티

접근자 프로퍼티는 값을 직접 저장하지 않고 `get`과 `set` 함수로 값을 읽거나 저장한다.

| 내부 어트리뷰트 | 디스크립터 키 | 의미 |
| --- | --- | --- |
| `[[Get]]` | `get` | 프로퍼티를 읽을 때 호출되는 함수 |
| `[[Set]]` | `set` | 프로퍼티에 값을 할당할 때 호출되는 함수 |
| `[[Enumerable]]` | `enumerable` | `true`이면 열거할 수 있음 |
| `[[Configurable]]` | `configurable` | `true`이면 프로퍼티 삭제 및 어트리뷰트 변경이 가능함 |

접근자 프로퍼티에는 `[[Value]]`와 `[[Writable]]`이 없다. 따라서 값을 저장하려면 별도의 데이터 프로퍼티나 클로저 등을 사용해야 한다.

```js
const person = {
  firstName: 'Ungmo',
  lastName: 'Lee',

  get fullName() {
    return `${this.firstName} ${this.lastName}`;
  },

  set fullName(name) {
    [this.firstName, this.lastName] = name.split(' ');
  }
};

console.log(person.fullName); // 'Ungmo Lee'
person.fullName = 'Heegun Lee';
console.log(person.firstName); // 'Heegun'

console.log(Object.getOwnPropertyDescriptor(person, 'fullName'));
// { get: [Function: get fullName], set: [Function: set fullName],
//   enumerable: true, configurable: true }
```

### 프로퍼티 정의

`Object.defineProperty`로 프로퍼티 어트리뷰트를 직접 설정할 수 있다.
생략한 어트리뷰트의 기본값은 `false` 또는 `undefined`이므로 주의해야 한다.

```js
const person = {};

Object.defineProperty(person, 'name', {
  value: 'Lee',
  writable: true,
  enumerable: true,
  configurable: true
});

console.log(person.name); // 'Lee'
```

### 객체 변경 방지

객체의 변경을 제한하는 메서드는 제한 범위가 서로 다르다.

```js
const extensible = { name: 'Lee' };
Object.preventExtensions(extensible); // 프로퍼티 추가 금지

const sealed = { name: 'Lee' };
Object.seal(sealed); // 추가·삭제 금지, 값 변경은 가능

const frozen = { name: 'Lee' };
Object.freeze(frozen); // 추가·삭제·값 변경 모두 금지
```

상태 확인에는 다음 메서드를 사용할 수 있다.

```js
Object.isExtensible(extensible); // false
Object.isSealed(sealed);         // true
Object.isFrozen(frozen);         // true
```

`seal`과 `freeze`는 얕게 동작하므로 중첩 객체까지 자동으로 보호하지 않는다.

> [추가 정리](https://github.com/mash-up-kr/WEB-JS-Deep-Dive/blob/jeongwoo903/02-ch08-11-%EC%A0%9C%EC%96%B4%EB%AC%B8-%ED%83%80%EC%9E%85%EB%B3%80%ED%99%98-%EA%B0%9D%EC%B2%B4%EB%A6%AC%ED%84%B0%EB%9F%B4-%EB%B9%84%EA%B5%90/%EC%9E%A5%EC%A0%95%EC%9A%B0/READMD.md#objectfreeze%EC%99%80-objectseal)

## 17장. 생성자 함수에 의한 객체 생성

### 생성자 함수

생성자 함수는 `new` 연산자와 함께 호출해 여러 개의 인스턴스를 만들 수 있는 함수다.

```js
function Circle(radius) {
  this.radius = radius;
  this.getDiameter = function () {
    return 2 * this.radius;
  };
}

const circle = new Circle(5);
console.log(circle.getDiameter()); // 10
```

일반 함수 호출과 생성자 함수 호출은 `new` 사용 여부에 따라 동작이 달라진다.

```js
function User(name) {
  this.name = name;
}

const user = new User('Lee'); // 인스턴스 생성
const result = User('Kim');   // 일반 함수 호출

console.log(user.name); // 'Lee'
console.log(result);    // undefined
```

### `new` 연산자의 동작

`new User('Lee')`는 개념적으로 다음 과정을 거친다.

구체적인 단계와 `this` 바인딩은 아래에서 예제로 확인할 수 있다.

```js
function Person(name) {
  this.name = name;
}

const person = new Person('Lee');

console.log(person instanceof Person); // true
console.log(person.name);              // 'Lee'
```

### 인스턴스 생성과 `this` 바인딩

생성자 함수에 `new` 연산자를 사용하면 자바스크립트 엔진이 다음 과정을 암묵적으로 처리한다.

```text
빈 객체 생성
  ↓
[[Prototype]]을 User.prototype으로 연결
  ↓
생성자 함수의 this를 새 객체에 바인딩
  ↓
생성자 함수 본문 실행 및 프로퍼티 초기화
  ↓
완성된 인스턴스 반환
```

```js
function User(name) {
  // 이때 this는 새로 생성된 인스턴스를 가리킨다.
  this.name = name;
}

const user = new User('Lee');

console.log(user); // User { name: 'Lee' }
console.log(Object.getPrototypeOf(user) === User.prototype); // true
```

### 생성자 함수의 반환값

생성자 함수가 명시적으로 객체를 반환하면 그 객체가 인스턴스 대신 반환된다.
반면 원시값을 반환하면 무시되고, `this`로 초기화한 인스턴스가 반환된다.
(..??)

```js
function ReturnObject() {
  this.value = 1;
  return { value: 2 };
}

function ReturnPrimitive() {
  this.value = 1;
  return 2; // 원시값은 무시된다.
}

console.log(new ReturnObject().value);    // 2
console.log(new ReturnPrimitive().value); // 1
```

### 내부 메서드 `[[Call]]`과 `[[Construct]]`

함수 객체는 일반 객체와 달리 호출을 위한 `[[Call]]` 내부 메서드를 가진다.  
생성자 함수로 사용할 수 있는 함수 객체는 여기에 `[[Construct]]` 내부 메서드도 가진다.

| 구분 | 호출 방법 | 의미 |
| --- | --- | --- |
| `[[Call]]` | `fn()` | 일반 함수로 호출 |
| `[[Construct]]` | `new fn()` | 생성자 함수로 호출해 인스턴스 생성 |

```js
function normal() {}

normal();     // [[Call]] 호출
new normal(); // [[Construct]] 호출
```

`new` 연산자는 함수에 `[[Construct]]`가 없으면 오류를 발생시킨다.

### `callable`과 `constructor`

함수 객체는 `[[Call]]`과 `[[Construct]]`의 보유 여부에 따라 구분할 수 있다.

| 함수 종류 | `fn()` | `new fn()` | 설명 |
| --- | ---: | ---: | --- |
| 일반 함수 | 가능 | 가능 | `[[Call]]`, `[[Construct]]`를 모두 가짐 |
| 화살표 함수 | 가능 | 불가능 | `[[Call]]`만 가짐 |
| 객체 메서드 | 가능 | 불가능 | `[[Call]]`만 가짐 |
| 클래스 | 불가능 | 가능 | `new` 없이 호출하면 오류 |

```js
function normal() {}
const arrow = () => {};
const object = { method() {} };
class Person {}

normal();
new normal();

arrow();
// new arrow();       // TypeError

object.method();
// new object.method(); // TypeError

// Person();           // TypeError
new Person();
```

즉, 모든 함수 객체가 생성자 함수인 것은 아니다. `callable`은 호출할 수 있다는 뜻이고,
`constructor`는 `new`와 함께 호출해 인스턴스를 생성할 수 있다는 뜻이다.

### `new.target`

`new.target`은 함수가 `new`와 함께 호출되었는지 확인할 때 사용한다.

```js
function Person(name) {
  if (!new.target) {
    return new Person(name);
  }

  this.name = name;
}

const person = Person('Lee');
console.log(person.name); // 'Lee'
```

일반 함수로 호출했을 때도 생성자 함수처럼 동작하게 만들 수 있지만, 호출 방식을 혼용하기보다는 사용 규칙을 명확히 하는 편이 좋다.

### 생성자 함수와 `prototype`

생성자 함수로 생성된 인스턴스는 생성자 함수의 `prototype` 객체를 상속받는다.
공통 메서드를 `prototype`에 두면 인스턴스마다 메서드를 새로 만들지 않아 메모리를 절약할 수 있다.

```js
function Circle(radius) {
  this.radius = radius;
}

Circle.prototype.getDiameter = function () {
  return 2 * this.radius;
};

const circle = new Circle(5);
console.log(circle.getDiameter()); // 10
```

## 18장. 함수와 일급 객체

### 함수는 일급 객체다

> 차후 함수형 프로그래밍을 공부할때도 쓰이는 개념

일급 객체는 다음 조건을 만족하는 값이다.

- 무명의 리터럴로 생성할 수 있다.
- 변수나 자료구조에 저장할 수 있다.
- 함수의 매개변수로 전달할 수 있다.
- 함수의 반환값으로 사용할 수 있다.

자바스크립트의 함수는 이 조건을 모두 만족한다. 

> 보통 함수가 일급객체인건 함수형 언어의 특징이다.  
> Java, C, C++처럼 객체·포인터 중심의 언어에서는 람다, 함수 포인터, 함수형 인터페이스 등을 통해 제한적으로 지원한다.

```js
const add = (x, y) => x + y;

function operate(operator, x, y) {
  return operator(x, y);
}

console.log(operate(add, 2, 3)); // 5
```

### 함수 객체의 프로퍼티

함수도 객체이므로 프로퍼티를 가질 수 있다.

```js
function multiply(x, y) {
  return x * y;
}

console.log(Object.getOwnPropertyDescriptors(multiply));
```

대표적인 함수 객체 프로퍼티는 다음과 같다.

- `arguments`: 함수 호출 시 전달된 인수를 담은 유사 배열 객체
- `caller`: 현재 함수가 호출된 경로를 가리키는 비표준·비권장 프로퍼티
- `length`: 함수가 선언한 매개변수의 개수
- `name`: 함수 이름
- `prototype`: 생성자 함수가 인스턴스에 연결할 프로토타입 객체

### `arguments` 객체

`arguments`는 함수에 전달된 모든 인수를 확인할 때 사용할 수 있다.
가변 인자를 다룰 때는 현대적인 문법인 rest parameter를 우선 고려한다.

```js
function sum() {
  let result = 0;

  for (let i = 0; i < arguments.length; i++) {
    result += arguments[i];
  }

  return result;
}

console.log(sum(1, 2, 3)); // 6
```

```js
function sumWithRest(...numbers) {
  return numbers.reduce((total, number) => total + number, 0);
}

console.log(sumWithRest(1, 2, 3)); // 6
```

### `length`, `name`, `prototype`

```js
function foo(x, y) {}

console.log(foo.length);  // 2
console.log(foo.name);    // 'foo'
console.log(foo.prototype); // { constructor: foo }
```

`length`는 실제 호출한 인수 개수가 아니라 함수 선언부의 매개변수 개수를 의미한다.
`prototype`은 일반 함수 객체에는 존재하지만, 화살표 함수에는 존재하지 않는다.

```js
function normal() {}
const arrow = () => {};

console.log(normal.prototype); // 객체
console.log(arrow.prototype);  // undefined
```

`callable`과 `constructor`의 차이 및 함수 종류별 `new` 가능 여부는
[17장의 내부 메서드 설명](#내부-메서드-call과-construct)에서 다룬다.
