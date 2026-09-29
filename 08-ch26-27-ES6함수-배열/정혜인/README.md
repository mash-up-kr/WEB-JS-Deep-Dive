# 26ch ES6 함수의 추가 기능

## 함수의 구분

**ES6 이전의 모든 함수는 일반 함수로서 호출할 수 있는 것은 물론 생성자 함수로서 호출할 수 있음**. 즉, ES6 이전의 모든 함수는 일반 함수로서 호출할 수 있는 것은 물론 생성자 함수로서 호출도 가능함( = callable이면서 constructor가 됨)

```javascript
var foo = function () {
  return 1;
};

// 일반적인 함수로서 호출
foo(); // -> 1

// 생성자 함수로서 호출
new foo(); // -> foo {}

// 메서드로서 호출
var obj = { foo: foo };
obj.foo(); // -> 1
```

- **객체에 바인딩된 함수**가 `constructor`= `prototype` 프로퍼티를 가지며 **프로토타입 객체**도 생성한다는 것 → 성능 면에서 **문제**가 있을 수 있음
- 콜백함수도 constructor이므로 불필요한 프로토타입 객체를 생성함
- 메서드와 화살표 함수는 `non-constructor`이므로 차이가 존재

## 메서드

- ES6 이전 사양에는 메서드에 대한 명확한 정의가 존재하지 않음
- 일반적으로 메서드는 객체에 바인딩된 함수를 일컫는 의미로 사용됨. 하지만 ES6 사양에서는 메서드는 `메서드 축약 표현으로 정의된 함수만`을 의미

```javascript
const obj = {
  x: 1,
  // foo는 메서드이다.
  foo() {
    return this.x;
  },
  // bar에 바인딩된 함수는 메서드가 아닌 일반 함수이다.
  bar: function () {
    return this.x;
  },
};

console.log(obj.foo()); // 1
console.log(obj.bar()); // 1
```

- ES6 사양에서 정의한 메서드는 인스턴스를 생성할 수 없는 `non-constructor` → 따라서 생성자 함수로 호출이 불가
- `prototype` 프로퍼티가 없고 프로토타입도 생성하지 않음
- ES6 메서드는 자신을 바인딩한 객체를 가리키는 내부 슬롯 `[[HomeObject]]`를 소유 → `super` 키워드를 사용가능

## 화살표 함수

화살표함수는 `function` 키워드 대신 `화살표(=>)`를 사용하여 기존의 함수 정의 방식보다 간략하게 함수를 정의가능

특히 콜백 함수 내부에서 this가 전역 객체를 가리키는 문제를 해결하기 위한 대안으로 유용함

### 화살표 함수 정의

- 함수 몸체 정의
  - 함수 몸체가 **하나의 문**으로 구성된다면 함수 몸체를 감싸는 중괄호 {}를 생략가능→ 이때 함수 몸체 내부의 문이 값으로 평가될 수 있는 **표현식**인 문이라면 암묵적으로 반환됨
- **객체 리터럴**을 반환하는 경우 소괄호로 감싸줌
- 화살표 함수도 **즉시 실행 함수**로 사용할 수 있음
- 화살표 함수도 **일급 객체**이므로 `Array.prototype.map`, `Array.prototype.filter` 같은 고차함수에 인수로 전달 가능

### 화살표 함수와 일반 함수의 차이

- 화살표 함수는 인스턴스를 생성할 수 없는 `non-constructor임` → prototype 프로퍼티가 없고 프로토타입도 생성하지 않음
- 중복된 매개변수 이름을 선언할 수 없음 → 중복된 매개변수 이름을 선언시 **일반 함수**의 경우 `strict mode`에서만 에러가 발생
- 화살표 함수는 함수 자체의 `this`, `arguments`, `super`, `new.target` 바인딩을 갖지 않음 → 화살표 함수 내부에서 이를 참조하면 스코프 체인을 통해 상위 스코프의 `this`, `arguments`, `super`, `new.target`들을 참조

### this

화살표 함수와 일반함수의 가장 큰 차이점은 `this`

화살표 함수는 다른 함수의 인수로 전달되어 콜백 함수로 사용되는 경우가 많음

```javascript
class Prefixer {
  constructor(prefix) {
    this.prefix = prefix;
  }

  add(arr) {
    // add 메서드는 인수로 전달된 배열 arr을 순회하며 배열의 모든 요소에 prefix를 추가한다.
    // 1️⃣
    return arr.map(function (item) {
      return this.prefix + item; // 2️⃣
      // -> TypeError: Cannot read property 'prefix' of undefined
    });
  }
}

const prefixer = new Prefixer("-webkit-");
console.log(prefixer.add(["transition", "user-select"]));
```

위의 예제는 일반함수로서 호출되는 콜백함수에서 일어나는 문제점 존재
→ 1️⃣에서 this는 메서드를 호출한 `prefixer` 객체를 가리킴
→ 하지만 2️⃣에서는 `undefined`를 가리킴

- ES6에서는 화살표 함수를 통해 `콜백 함수 내부의 this 문제`를 해결 가능

```javascript
class Prefixer {
  constructor(prefix) {
    this.prefix = prefix;
  }

  add(arr) {
    return arr.map((item) => this.prefix + item);
  }
}

const prefixer = new Prefixer("-webkit-");
console.log(prefixer.add(["transition", "user-select"]));
// ['-webkit-transition', '-webkit-user-select']
```

- 화살표 함수는 함수 자체의 this 바인딩을 갖지 않음 → 화살표 함수 내부에서 this를 참조하면 상위 스코프의 this를 그대로 참조 =`lexical this`
- 만약 화살표 함수와 화살표 함수가 중첩되어 있다면 스코프 체인 상에서 가장 가까운 상위 함수 중 화살표 함수가 아닌 함수의 this를 참조
- 하지만 이런 특성 때문에 메서드를 화살표 함수로 정의하는 것은 피해야함
- 메서드를 정의할 때는 ES6 메서드 축약 표현으로 정의한 ES6 메서드를 사용하는 것이 좋음
- 프로퍼티를 동적 추가 할 때는 ES6메더스 정의를 사용할 수 없으므로 일반 함수를 할당함
- 일반 함수가 아닌 ES6 메서드를 동적 추가하고 싶다면 다음과 같이 객체 리터럴을 바인딩하고 프로토 타입의 constructor 프로퍼티와 생성자 함수 간의 연결을 재설정

```javascript
function Person(name) {
  this.name = name;
}

Person.prototype = {
  // constructor 프로퍼티와 생성자 함수 간의 연결을 재설정
  constructor: Person,
  sayHi() {
    console.log(`Hi ${this.name}`);
  },
};

const person = new Person("Lee");
person.sayHi(); // Hi Lee
```

### super

- 화살표 함수는 함수 자체의 super 바인딩을 갖지 않음 → 화살표 함수 내부에서 super를 참조하면 this와 마찬가지로 상위 스코프의 super를 참조

```javascript
class Base {
  constructor(name) {
    this.name = name;
  }

  sayHi() {
    return `Hi! ${this.name}`;
  }
}

class Derived extends Base {
  // 화살표 함수의 super는 상위 스코프인 constructor의 super를 가리킨다.
  sayHi = () => `${super.sayHi()} how are you doing?`;
}

const derived = new Derived("Lee");
console.log(derived.sayHi()); // Hi! Lee how are you doing?
```

- `super`는 내부슬롯 `[[HomeObject]]`를 갖는 ES6 메서드 내에서만 사용할 수 있는 키워드
- 화살표 함수가 super 바인딩을 갖지 않음 → 암묵적으로 생성된 constructor의 super 바인딩을 참조해서 에러가 발생하지 않음

### arguments

화살표 함수는 함수 자체의 arguments 바인딩을 갖지 않음 → 화살표 함수 내에서 arguments를 참조하면 상위 스코프의 arguments를 참조

```javascript
(function () {
  // 화살표 함수 foo의 arguments는 상위 스코프인 즉시 실행 함수의 arguments를 가리킨다.
  const foo = () => console.log(arguments); // [Arguments] { '0': 1, '1': 2 }
  foo(3, 4);
})(1, 2);

// 화살표 함수 foo의 arguments는 상위 스코프인 전역의 arguments를 가리킨다.
// 하지만 전역에는 arguments 객체가 존재하지 않는다. arguments 객체는 함수 내부에서만 유효하다.
const foo = () => console.log(arguments);
foo(1, 2); // ReferenceError: arguments is not defined
```

- 위의 예제처럼 화살표 함수로 상위 스코프의 `arguments`를 참조할 수는 있지만 자신에게 전달된 인수 목록을 확인할 수 없음
  → 화살표 함수로 가변 인자 함수를 구현해야 할 때는 `Rest` 파라미터를 사용하는 것이 좋음

## Rest 파라미터

### 기본 문법

**Rest 파라미터**는 매개변수 이름 앞에 세개의 점 `...`을 붙여서 정의한 `매개변수`로,함수에 전달된 인수들의 목록을 배열로 전달받는다.

```javascript
function foo(param, ...rest) {
  // 매개변수 rest는 인수들의 목록을 배열로 전달받는 Rest 파라미터다.
  console.log(param); // 1
  console.log(rest); // [ 2, 3, 4, 5 ]
}

foo(1, 2, 3, 4, 5);

function foo(param1, param2, ...rest) {
  console.log(param1); // 1
  console.log(param2); // 2
  console.log(rest); // [3, 4, 5]
}

bar(1, 2, 3, 4, 5);
```

- Rest 파라미터는 단 하나만 선언 가능
- Rest 파라미터는 함수 정의 시 선언한 매개변수 개수를 나타내는 함수 객체의 `length` 프로퍼티에 영항을 주지 않음

### Rest 파라미터와 arguments 객체

- ES5에서는 함수를 정의할 때 매개변수의 개수를 확정할 수 없는 `가변 인자 함수`의 경우 `argument` 객체를 활용하여 인수를 전달받음
  → `argument` 객체는 유사 배열 객체이므로 배열로 변환해야 배열 메서드를 사용할 수 있음

```javascript
function sum() {
  // 유사 배열 객체인 arguments 객체를 배열로 변환한다.
  var array = Array.prototype.slice.call(arguments);

  return array.reduce(function (pre, cur) {
    return pre + cur;
  }, 0);
}

console.log(sum(1, 2, 3, 4, 5)); // 15
```

- ES6에서는 `Rest`파라미터를 활용하여 `가변 인자 함수`의 인수 목록을 배열로 직접 전달받을 수 있음
- 화살표 함수는 **`arguments`** 객체를 갖지 않으므로 화살표 함수로 가변 인자 함수를 구현해야 할 때는 반드시 `Rest` 파라미터를 사용해야함

## 매개변수 기본값

자바스크립트 엔진이 매개변수의 개수와 인수의 개수를 체크하지 않기 때문에 인수가 전달되지 않은 매개변수의 값은 `undefined`가 되어비므로 이를 방치하면 의도치 않은 결과가 나오는 문제점 발생

```javascript
function sum(x, y) {
  return x + y;
}

console.log(sum(1)); // NaN
```

- ES6에서 도입된 `매개변수 기본값`을 사용하면 인수 체크 및 초기화를 간단히 할 수 있음

```javascript
function sum(x = 0, y = 0) {
  return x + y;
}

console.log(sum(1, 2)); // 3
console.log(sum(1)); // 1
```

- 매개변수 기본값은 매개변수에 인수를 전달하지 않은 경우와 `undefined`를 전달한 경우에만 유효

```javascript
function logName(name = "Lee") {
  console.log(name);
}

logName(); // Lee
logName(undefined); // Lee
logName(null); // null
```

- `Rest` 파라미터에는 기본값을 지정이 불가능함

```javascript
function foo(...rest = []) {
  console.log(rest);
}

// SyntaxError: Rest parameter may not have a default initializer
```

- 매개변수 기본값은 함수 객체의 length 프로퍼티와 arguments 객체에 아무런 영항을 주지 않음

```javascript
function sum(x, y = 0) {
  console.log(arguments);
}

console.log(sum.length); // 1

sum(1); // Arguments { '0': 1 }
sum(1, 2); // Arguments { '0': 1, '1': 2 }
```

<br/>

# 27ch 배열

## 배열이란?

여러 개의 값을 순차적으로 나열한 자료구조를 의미

- 요소: 배열이 가지고 있는 값
  - 배열의 요소는 배열에서 자신의 위치를 나타내는 0 이상의 정수인 `인덱스(index)`를 갖고있음
  - 인덱스를 통해 배열의 요소에 접근이 가능함

```javascript
const arr = ['apple', 'mango', 'orange'];

arr[0] // -> 'apple'
arr[1] // -> 'mango'
arr[2] // -> 'orange'
```

- 배열을 요소의 개수, 즉 배열의 길이를 나타내는 `length` 프로퍼티 존재

```javascript
arr.length // -> 3
```

- 배열과 객체의 차이점

| 구분 | 객체 | 배열 |
| --- | --- | --- |
| 구조 | 프로퍼티 키와 프로퍼티 값 | 인덱스와 요소 |
| 값의 참조 | 프로퍼티 키 | 인덱스 |
| 값의 순서 | X | O |
| length 프로퍼티 | X | O |

- 배열은 일반 객체와 다르게 값의 순서와 length 프로퍼티 존재 → 반복문으로 값에 접근할 때 유용함

## 자바스크립트 배열은 배열이 아니다

`밀집 배열`: 배열의 요소는 하나의 데이터 타입으로 통일되어 있으며 서로 연속적으로 인접해 있는 것

- 일반적인 의미의 배열은 각 요소가 동일한 데이터 크기를 갖고 연속적으로 이어져 있어 인덱스를 통해 효율적으로 요소에 접근가능
- 정렬되지 않은 배열에서 특정 요소를 검색하는 경우 → 모든 요소를 특정 요소를 찾을 때까지 검색이 필요 + 베열에 요소를 삽입, 삭제시 배열의 요소를 연속적으로 유지하기 위해 요소를 일일이 이동시켜야 한다는 단점이 존재
- 배열의 요소를 위한 각각의 메모리 공간은 동일한 크기를 갖지 않아도 되며, 연속적으로 이어져 있지 않을 수 있음
- `희소 배열`: 배열의 요소가 연속적으로 이어져 있지 않는 배열
- 자바스크립트의 배열은 일반적인 배열의 동작을 흉내낸 특수한 객체임

```javascript
// "16.2. 프로퍼티 어트리뷰트와 프로퍼티 디스크립터 객체" 참고
console.log(Object.getOwnPropertyDescriptors([1, 2, 3]));
/*
{
  '0': {value: 1, writable: true, enumerable: true, configurable: true}
  '1': {value: 2, writable: true, enumerable: true, configurable: true}
  '2': {value: 3, writable: true, enumerable: true, configurable: true}
  length: {value: 3, writable: true, enumerable: false, configurable: false}
}
*/
```

- 일반적인 배열과 자바스크립트 배열의 장단점

| 구분 | 일반적인 배열 | 자바스크립트 배열 |
| --- | --- | --- |
| 장점 | 인덱스로 요소에 빠르게 접근 가능 | 요소 삽입 및 삭제할 때 효율적이지 않음 |
| 단점 | 해시 테이블로 구현된 객체이므로 요소에 접근할 때 상대적으로 느림 | 요소 삽입 및 삭제할 때 빠른 성능을 가짐 |

## length 프로퍼티와 희소 배열

- `length 프로퍼티` : 요소의 개수, 즉 배열의 길이를 나타내는 0 이상의 정수를 값으로 가짐
- length 프로퍼티의 값은 배열에 요소를 추가하거나 삭제하면 자동 갱신됨
- length 프로퍼티 값보다 작은 값을 할당하면 배열의 길이가 줄어듦
- length 프로퍼티 값보다 큰 값을 할당하면 배열의 길이는 늘어나지 않음
- `희소 배열` : 배열의 요소가 연속적으로 위치하지 않고 일부가 비어 있는 배열
  - 희소 배열은 length와 배열 요소의 개수가 일치하지 않음
  - 희소 배열의 length는 희소 배열의 실제 요소 개수보다 언제나 큼

```javascript
// 희소 배열
const sparse = [, 2, , 4];

// 희소 배열의 length 프로퍼티 값은 요소의 개수와 일치하지 않는다.
console.log(sparse.length); // 4
console.log(sparse); // [empty, 2, empty, 4]

// 배열 sparse에는 인덱스가 0, 2인 요소가 존재하지 않는다.
console.log(Object.getOwnPropertyDescriptors(sparse));
/*
{
  '1': { value: 2, writable: true, enumerable: true, configurable: true },
  '3': { value: 4, writable: true, enumerable: true, configurable: true },
  length: { value: 4, writable: true, enumerable: false, configurable: false }
}
*/
```

- 희소배열 사용은 비추
  - 배열의 기본적인 개념과도 다름
  - 성능에 좋지 않은 영향을 주기 때문
- 배열에는 같은 타입의 요소를 연속적으로 위치시키는 것이 좋음

## 배열 생성

### 배열 리터럴

- `배열 리터럴`은 0개 이상의 요소를 쉼표로 구분하여 대괄호([ ])로 묶음
- 객체 리터럴과 달리 프로퍼티 키가 없고 값만 존재함
- 배열 리터럴 요소를 하나도 추가하지 않으면 배열의 길이, 즉 length 프로퍼티 값이 0인 빈 배열이 됨
- 배열 리터럴에 요소를 생략하면 희소 배열이 됨

### Array 생성자 함수

Array 생성자 함수를 통해 배열을 생성할 수 있음

- 전달된 인수가 1개이고 숫자인 경우 length 프로퍼티 값이 인수인 배열을 생성한다.
- 전달된 인수가 없는 경우 빈 배열을 생성하며, 배열 리터럴 `[]`과 같음

```javascript
new Array(); // -> []
```

- 전달된 인수가 2개 이상이거나 숫자가 아닌 경우 인수를 요소로 갖는 배열을 생성

```javascript
// 전달된 인수가 2개 이상이면 인수를 요소로 갖는 배열을 생성한다.
new Array(1, 2, 3); // -> [1, 2, 3]

// 전달된 인수가 1개지만 숫자가 아니면 인수를 요소로 갖는 배열을 생성한다.
new Array({}); // -> [{}]
```

- `Array 생성자 함수`는 new 연산자와 함께 호출되지 않더라도, 배열을 생성하는 생성자 함수로 동작

```javascript
Array(1, 2, 3); // -> [1, 2, 3]
```

### Array.of

`Array.of 메서드`는 전달된 인수를 요소로 갖는 배열을 생성

```javascript
// 전달된 인수가 1개이고 숫자이더라도 인수를 요소로 갖는 배열을 생성한다.
Array.of(1); // -> [1]

Array.of(1, 2, 3); // -> [1, 2, 3]

Array.of('string'); // -> ['string']
```

### Array.from

`Array.from 메서드`는 **유사 배열 객체 또는 이터러블 객체**를 인수로 전달 받아 배열로 변환하여 반환

```javascript
// 유사 배열 객체를 변환하여 배열을 생성한다.
Array.from({ length: 2, 0: 'a', 1: 'b' }); // -> ['a', 'b']

// 이터러블을 변환하여 배열을 생성한다. 문자열은 이터러블이다.
Array.from('Hello'); // -> ['H', 'e', 'l', 'l', 'o']
```

## 배열 요소의 참조

대괄호([ ]) 표기법을 사용해 배열의 요소를 참조하며, 대괄호 안에는 인덱스가 와야함

## 배열 요소의 추가와 갱신

- 존재하지 않는 인덱스를 사용해 값을 할당하면 배열에 요소를 동적으로 추가 가능

```javascript
const arr = [0];

// 배열 요소의 추가
arr[1] = 1;

console.log(arr); // [0, 1]
console.log(arr.length); // 2
```

- 현재 배열의 length 프로퍼티 값보다 큰 인덱스로 새로운 요소를 추가하면 희소 배열이 됨

```javascript
arr[100] = 100;

console.log(arr); // [0, 1, empty × 98, 100]
console.log(arr.length); // 101
```

- 정수가 아닌 값을 인덱스로 사용하면 요소가 아닌 프로퍼티가 생성 → length 프로퍼티 값에 영향을 주지는 않음

```javascript
const arr = [];

// 배열 요소의 추가
arr[0] = 1;
arr['1'] = 2;

// 프로퍼티 추가
arr['foo'] = 3;
arr.bar = 4;
arr[1.1] = 5;
arr[-1] = 6;

console.log(arr); // [1, 2, foo: 3, bar: 4, '1.1': 5, '-1': 6]

// 프로퍼티는 length에 영향을 주지 않는다.
console.log(arr.length); // 2
```

## 배열 요소의 삭제

배열은 사실 객체 → `delete 연산자`를 사용하여 특정 요소를 삭제가 가능함

- delete 연산자는 객체의 프로퍼티를 삭제 → 희소 배열이 되고 length 프로퍼티는 그대로 됨

```javascript
const arr = [1, 2, 3];

// 배열 요소의 삭제
delete arr[1];
console.log(arr); // [1, empty, 3]

// length 프로퍼티에 영향을 주지 않는다. 즉, 희소 배열이 된다.
console.log(arr.length); // 3
```

- 희소 배열을 만드는 것은 좋지 않으므로 배열의 특정 요소를 완전히 삭제하는 `Array.prototype.splice 메서드`를 사용

```javascript
const arr = [1, 2, 3];

// Array.prototype.splice(삭제를 시작할 인덱스, 삭제할 요소 수)
// arr[1]부터 1개의 요소를 제거
arr.splice(1, 1);
console.log(arr); // [1, 3]

// length 프로퍼티가 자동 갱신된다.
console.log(arr.length); // 2
```

## 배열 메서드

- Array 생성자 함수는 **정적 메서드**를 제공
- 배열 객체의 프로토타입인 Array.prototype은 **프로토타입 메서드**를 제공
- 배열메서드가 결과물을 반환하는 패턴
  - 배열에는 원본 배열(배열 메서드를 호출한 배열, 즉 배열 메서드의 구현체 내부에서 this가 가리티는 객체)을 직접 변경하는 메서드
  - 원본 배열을 직접 변경하지 않고 새로운 배열을 생성하여 반환하는 메서드

```javascript
const arr = [1];

// push 메서드는 원본 배열(arr)을 직접 변경한다.
arr.push(2);
console.log(arr); // [1, 2]

// concat 메서드는 원본 배열(arr)을 직접 변경하지 않고 새로운 배열을 생성하여 반환한다.
const result = arr.concat(3);
console.log(arr);    // [1, 2]
console.log(result); // [1, 2, 3]
```

### Array.isArray

`Array.isArray`는 Array 생성자 함수의 정적 메서드로, `Array.isArray` 메서드는 전달된 인수가 배열이면 true, 배열이 아니면 false를 반환

### Array.prototype.indexOf

- `indexOf 메서드`는 원본 배열에서 인수로 전달된 요소를 검색하여 인덱스를 반환
- index 메서드는 **배열에 특정 요소가 존재하는지 확인**할 때 유용
- 원본 배열에 인수로 전달한 요소와 중복되는 요소가 여러 개 있다면 첫 번째로 검색된 요소의 인덱스를 반환
- 원본 배열에 인수로 전달한 요소가 존재하지 않으면 -1을 반환

```javascript
const arr = [1, 2, 2, 3];

// 배열 arr에서 요소 2를 검색하여 첫 번째로 검색된 요소의 인덱스를 반환한다.
arr.indexOf(2);    // -> 1
// 배열 arr에 요소 4가 없으므로 -1을 반환한다.
arr.indexOf(4);    // -> -1
// 두 번째 인수는 검색을 시작할 인덱스다. 두 번째 인수를 생략하면 처음부터 검색한다.
arr.indexOf(2, 2); // -> 2
```

### Array.prototype.push

`push 메서드`는 인수로 전달받은 모든 값을 원본 배열의 마지막 요소로 추가하고 변경된 length 프로퍼티 값을 반환하며, 원본 배열을 직접 변경

```javascript
const arr = [1, 2];

// 인수로 전달받은 모든 값을 원본 배열 arr의 마지막 요소로 추가하고 변경된 length 값을 반환한다.
let result = arr.push(3, 4);
console.log(result); // 4

// push 메서드는 원본 배열을 직접 변경한다.
console.log(arr); // [1, 2, 3, 4]
```

- 스프레드 문법을 이용하여 원본 배열 직접 변경 없이 요소 추가 가능

```javascript
const arr = [1, 2];

// ES6 스프레드 문법
const newArr = [...arr, 3];
console.log(newArr); // [1, 2, 3]
```

### Array.prototype.pop

`pop 메서드`는 원본 배열에서 마지막 요소를 제거하고 제거한 요소를 반환하며, 원본 배열을 직접 변경함

```javascript
const arr = [1, 2];

// 원본 배열에서 마지막 요소를 제거하고 제거한 요소를 반환한다.
let result = arr.pop();
console.log(result); // 2

// pop 메서드는 원본 배열을 직접 변경한다.
console.log(arr); // [1]
```

- pop과 push를 통해 스택을 만들 수 있다.
- `스택`은 **후입선출(LIFO - Last In First Out)** 방식의 자료구조다.
  - `푸시(push)`: 스택에 데이터를 밀어 넣는 것
  - `팝(pop)`: 스택에서 데이터를 꺼내는 것

- 스택을 사용한 생성자 함수 구현

```javascript
const Stack = (function () {
  function Stack(array = []) {
    if (!Array.isArray(array)) {
      // "47. 에러 처리" 참고
      throw new TypeError(`${array} is not an array.`);
    }
    this.array = array;
  }

  Stack.prototype = {
    // "19.10.1. 생성자 함수에 의한 프로토타입의 교체" 참고
    constructor: Stack,
    // 스택의 가장 마지막에 데이터를 밀어 넣는다.
    push(value) {
      return this.array.push(value);
    },
    // 스택의 가장 마지막 데이터, 즉 가장 나중에 밀어 넣은 최신 데이터를 꺼낸다.
    pop() {
      return this.array.pop();
    },
    // 스택의 복사본 배열을 반환한다.
    entries() {
      return [...this.array];
    }
  };

  return Stack;
}());

const stack = new Stack([1, 2]);
console.log(stack.entries()); // [1, 2]

stack.push(3);
console.log(stack.entries()); // [1, 2, 3]

stack.pop();
console.log(stack.entries()); // [1, 2]
```

- 클래스를 사용한 생성자 함수 구현

```javascript
class Stack {
  #array; // private class member

  constructor(array = []) {
    if (!Array.isArray(array)) {
      throw new TypeError(`${array} is not an array.`);
    }
    this.#array = array;
  }

  // 스택의 가장 마지막에 데이터를 밀어 넣는다.
  push(value) {
    return this.#array.push(value);
  }

  // 스택의 가장 마지막 데이터, 즉 가장 나중에 밀어 넣은 최신 데이터를 꺼낸다.
  pop() {
    return this.#array.pop();
  }

  // 스택의 복사본 배열을 반환한다.
  entries() {
    return [...this.#array];
  }
}

const stack = new Stack([1, 2]);
console.log(stack.entries()); // [1, 2]

stack.push(3);
console.log(stack.entries()); // [1, 2, 3]

stack.pop();
console.log(stack.entries()); // [1, 2]
```

### Array.prototype.unshift

`unshift 메서드`는 인수로 전달받은 모든 값을 원본 배열의 선두에 요소로 추가하고 변경된 length 프로퍼티 값을 반환하며, 원본 배열을 직접 변경함

```javascript
const arr = [1, 2];

// 인수로 전달받은 모든 값을 원본 배열의 선두에 요소로 추가하고 변경된 length 값을 반환한다.
let result = arr.unshift(3, 4);
console.log(result); // 4

// unshift 메서드는 원본 배열을 직접 변경한다.
console.log(arr); // [3, 4, 1, 2]
```

### Array.prototype.shift

`shift 메서드`는 원본 배열에서 첫 번째 요소를 제거하고 제거한 요소를 반환하며, 빈 배열인 경우 `undefined`를 반환하고 원본 배열을 직접 변경함

```javascript
const arr = [1, 2];

// 원본 배열에서 첫 번째 요소를 제거하고 제거한 요소를 반환한다.
let result = arr.shift();
console.log(result); // 1

// shift 메서드는 원본 배열을 직접 변경한다.
console.log(arr); // [2]
```

shift와 push를 통해 큐를 만들 수 있음

- `큐`는 **선입선출(FIFO - First In First Out)** 방식의 자료구조

- 생성자 함수로 큐 구현

```javascript
const Queue = (function () {
  function Queue(array = []) {
    if (!Array.isArray(array)) {
      // "47. 에러 처리" 참고
      throw new TypeError(`${array} is not an array.`);
    }
    this.array = array;
  }

  Queue.prototype = {
    // "19.10.1. 생성자 함수에 의한 프로토타입의 교체" 참고
    constructor: Queue,
    // 큐의 가장 마지막에 데이터를 밀어 넣는다.
    enqueue(value) {
      return this.array.push(value);
    },
    // 큐의 가장 처음 데이터, 즉 가장 먼저 밀어 넣은 데이터를 꺼낸다.
    dequeue() {
      return this.array.shift();
    },
    // 큐의 복사본 배열을 반환한다.
    entries() {
      return [...this.array];
    }
  };

  return Queue;
}());

const queue = new Queue([1, 2]);
console.log(queue.entries()); // [1, 2]

queue.enqueue(3);
console.log(queue.entries()); // [1, 2, 3]

queue.dequeue();
console.log(queue.entries()); // [2, 3]
```

- 클래스로 큐 구현

```javascript
class Queue {
  #array; // private class member

  constructor(array = []) {
    if (!Array.isArray(array)) {
      throw new TypeError(`${array} is not an array.`);
    }
    this.#array = array;
  }

  // 큐의 가장 마지막에 데이터를 밀어 넣는다.
  enqueue(value) {
    return this.#array.push(value);
  }

  // 큐의 가장 처음 데이터, 즉 가장 먼저 밀어 넣은 데이터를 꺼낸다.
  dequeue() {
    return this.#array.shift();
  }

  // 큐의 복사본 배열을 반환한다.
  entries() {
    return [...this.#array];
  }
}

const queue = new Queue([1, 2]);
console.log(queue.entries()); // [1, 2]

queue.enqueue(3);
console.log(queue.entries()); // [1, 2, 3]

queue.dequeue();
console.log(queue.entries()); // [2, 3]
```

### Array.prototype.concat

`concat 메서드`는 인수로 전달된 값들(배열 또는 원시값)을 원본 배열의 마지막 요소로 추가한 새로운 배열을 반환하며, 원본 배열은 변경되지 않음

```javascript
const arr1 = [1, 2];
const arr2 = [3, 4];

// 배열 arr2를 원본 배열 arr1의 마지막 요소로 추가한 새로운 배열을 반환한다.
// 인수로 전달한 값이 배열인 경우 배열을 해체하여 새로운 배열의 요소로 추가한다.
let result = arr1.concat(arr2);
console.log(result); // [1, 2, 3, 4]

// 숫자를 원본 배열 arr1의 마지막 요소로 추가한 새로운 배열을 반환한다.
result = arr1.concat(3);
console.log(result); // [1, 2, 3]

// 배열 arr2와 숫자를 원본 배열 arr1의 마지막 요소로 추가한 새로운 배열을 반환한다.
result = arr1.concat(arr2, 5);
console.log(result); // [1, 2, 3, 4, 5]

// 원본 배열은 변경되지 않는다.
console.log(arr1); // [1, 2]
```

- push와 unshift 메서드는 `원본 배열을 직접 변경`하지만 concat 메서드는 원본 배열을 변경하지 않고 `새로운 배열을 반환`함

```javascript
const arr1 = [3, 4];

// unshift 메서드는 원본 배열을 직접 변경한다.
// 따라서 원본 배열을 변수에 저장해 두지 않으면 변경된 배열을 사용할 수 없다.
arr1.unshift(1, 2);
// unshift 메서드를 사용할 경우 원본 배열을 반드시 변수에 저장해 두어야 결과를 확인할 수 있다.
console.log(arr1); // [1, 2, 3, 4]

// push 메서드는 원본 배열을 직접 변경한다.
// 따라서 원본 배열을 변수에 저장해 두지 않으면 변경된 배열을 사용할 수 없다.
arr1.push(5, 6);
// push 메서드를 사용할 경우 원본 배열을 반드시 변수에 저장해 두어야 결과를 확인할 수 있다.
console.log(arr1); // [1, 2, 3, 4, 5, 6]

// unshift와 push 메서드는 concat 메서드로 대체할 수 있다.
const arr2 = [3, 4];

// concat 메서드는 원본 배열을 변경하지 않고 새로운 배열을 반환한다.
// arr1.unshift(1, 2)를 다음과 같이 대체할 수 있다.
let result = [1, 2].concat(arr2);
console.log(result); // [1, 2, 3, 4]

// arr1.push(5, 6)를 다음과 같이 대체할 수 있다.
result = result.concat(5, 6);
console.log(result); // [1, 2, 3, 4, 5, 6]
```

### Array.prototype.splice

원본 배열의 중간에 요소를 추가하거나 중간에 있는 요소를 제거할 때 사용함

- splice는 3개의 매개변수가 있으며 원본 배열을 직접 변경함
  - `start`: 원본 배열의 요소를 제거하기 시작할 인덱스
  - `deleteCount`: start부터 제거할 요소의 개수
  - `items`: 제거한 위치에 삽입할 요소들의 목록

```javascript
const arr = [1, 2, 3, 4];

// 원본 배열의 인덱스 1부터 2개의 요소를 제거하고 그 자리에 새로운 요소 20, 30을 삽입한다.
const result = arr.splice(1, 2, 20, 30);

// 제거한 요소가 배열로 반환된다.
console.log(result); // [2, 3]
// splice 메서드는 원본 배열을 직접 변경한다.
console.log(arr); // [1, 20, 30, 4]
```

### Array.prototype.slice

`slice 메서드`는 인수로 전달된 범위의 요소들을 복사하여 배열로 반환하며, 원본 배열은 변경되지 않음

- slice 메서드는 2개의 매개변수 존재
  - `start`: 복사를 시작할 인덱스
  - `end`: 복사를 종료할 인덱스
- 두 번째 인수(end)를 생략할 시, 첫 번째 인수(start)로 전달받은 인덱스부터 모든 요소를 복사하여 배열로 반환
- 첫 번째 인수가 음수인 경우 배열의 끝에서부터 요소를 복사하여 배열로 반환
- slice 메서드의 인수를 모두 생략하면 원본 배열의 복사본을 생성하여 반환 → 복사본은 `얕은 복사`를 통해 생성됨

### Array.prototype.join

`join 메서드`는 원본배열의 모든 요소를 문자열로 반환 후, 인수로 전달받은 문자열, 즉 구분자로 연결한 문자열을 반환

- 구분자는 생략 가능하며 기본 구분자는 `콤마(,)`가 됨

### Array.prototype.reverse

`reverse 메서드`는 원본 배열의 순서를 반대로 뒤집으며, 원본 배열이 변경됨

### Array.prototype.fill

`fill 메서드`는 인수로 전달받은 값을 배열의 처음부터 끝까지 요소로 채우며, 원본 배열이 변경됨

- 두 번째 인수로 요소 채우기를 시작할 인덱스를 전달함
- 세 번째 인수로 요소 채우기를 멈출 인덱스를 전달함

### Array.prototype.includes

`includes 메서드`는 배열 내에 특정 요소가 포함되어 있는지 확인하여 true 또는 false를 반환

### Array.prototype.flat

`flat 메서드`는 인수로 전달한 깊이만큼 재귀적으로 배열을 평탄화함

```javascript
[1, [2, 3, 4, 5]].flat(); // -> [1, 2, 3, 4, 5]
```

## 배열 고차 함수

`고차 함수`는 함수를 인수로 전달 받거나 함수를 반환하는 함수를 의미

- 고차함수는 불변성을 지향하는 `함수형 프로그래밍`에 기반을 둠
  - `함수형 프로그래밍`: 조건문과 반복문을 제거하여 복잡성을 해결하고 변수의 사용을 억제하여 상태 변경을 피하려는 프로그래밍 패러다임으로, 순수 함수를 통해 부수효과를 최대한 억제하여 오류를 피하고 프로그램의 안정성을 높일 수 있음

### Array.prototype.sort

`sort 메서드`는 배열의 요소를 정렬하며, 원본 배열을 직접 변경하며 정렬된 배열을 반환함

- 기본적으로 `오름차순`으로 요소를 정렬
- sort 메서드는 `유니코드 코드 포인트의 순서`를 따름 → 숫자 요소 정렬시 메서드에 정렬 순서를 정의하는 비교 함수를 인수로 전달해줘야함

```javascript
const points = [40, 100, 1, 5, 2, 25, 10];

// 숫자 배열의 오름차순 정렬. 비교 함수의 반환값이 0보다 작으면 a를 우선하여 정렬한다.
points.sort((a, b) => a - b);
console.log(points); // [1, 2, 5, 10, 25, 40, 100]

// 숫자 배열에서 최소/최대값 취득
console.log(points[0], points[points.length-1]); // 1 100

// 숫자 배열의 내림차순 정렬. 비교 함수의 반환값이 0보다 작으면 b를 우선하여 정렬한다.
points.sort((a, b) => b - a);
console.log(points); // [100, 40, 25, 10, 5, 2, 1]

// 숫자 배열에서 최소/최대값 취득
console.log(points[points.length - 1], points[0]); // 1 100
```

### 🪄 27.9.2 Array.prototype.forEach

`forEach 메서드`는 for문을 대체할 수 있는 고차 함수다.

forEach 메서드는 반복문을 추상화한 고차 함수로서, 자신의 내부에서 반복문을 실행한다.

```javascript
const numbers = [1, 2, 3];
let pows = [];

// forEach 메서드는 numbers 배열의 모든 요소를 순회하면서 콜백 함수를 반복 호출한다.
numbers.forEach(item => pows.push(item ** 2));
console.log(pows); // [1, 4, 9]
```

forEach 메서드는 break, continue 문을 사용할 수 없다.

배열의 모든 요소를 빠짐없이 순회하며 중간에 순회를 중단하지 않는다.

### Array.prototype.map

`map 메서드`는 자신을 호출한 배열의 모든 요소를 순회하면서 인수로 전달받은 콜백 함수를 반복 호출

- 콜백 함수의 반환값들로 구성된 새로운 배열을 반환하며, 원본 배열은 변경되지 않음
- forEach 메서드는 단순이 반복문을 대체하기 위한 고차 함수라 하면, map 메서드는 요소값을 다른 값으로 매핑한 새로운 배열을 생성하기 위한 고차 함수라 할 수 있음
- map 메서드가 생성하여 반환하는 새로운 배열의 length 프로퍼티 값은 map 메서드를 호출한 length 프로퍼티 값과 반드시 일치함

### Array.prototype.filter

- `filter 메서드`는 자신을 호출한 배열의 모든 요소를 순회하면서 인수로 전달받은 콜백 함수를 반복 호출
- 콜백 함수의 반환 값이 true인 요소로만 구성된 새로운 배열을 반환하며, 원본 배열은 변경되지 않음

### Array.prototype.reduce

`reduce 메서드`는 콜백 함수의 반환값을 다음 순회 시에 콜백 함수의 첫 번째 인수로 전달하면서 콜백 함수를 호출하여 하나의 결과값을 만들어 반환하며, 원본 배열을 변경되지 않음

- 하나의 결과값을 반환

### Array.prototype.some

`some 메서드`는 자신을 호출한 배열의 요소를 순회하면서 인수로 전달된 콜백 함수를 호출

- 콜백 함수의 반환값이 단 한 번이라도 참이면 true, 모두 거짓이면 false를 반환
- 요소가 1개 이상 존재하는지 확인할 때 사용
- 빈 배열일 경우 언제나 `false`를 반환힘

```javascript
// 배열의 요소 중에 10보다 큰 요소가 1개 이상 존재하는지 확인
[5, 10, 15].some(item => item > 10); // -> true

// 배열의 요소 중에 0보다 작은 요소가 1개 이상 존재하는지 확인
[5, 10, 15].some(item => item < 0); // -> false

// 배열의 요소 중에 'banana'가 1개 이상 존재하는지 확인
['apple', 'banana', 'mango'].some(item => item === 'banana'); // -> true

// some 메서드를 호출한 배열이 빈 배열인 경우 언제나 false를 반환한다.
[].some(item => item > 3); // -> false
```

### Array.prototype.every

`every 메서드`는 자신을 호출한 배열의 요소를 순회하면서 인수로 전달된 콜백 함수를 호출

- 콜백 함수의 반환값이 모두 참이면 true, 단 한 번이라도 거짓이면 false를 반환
- 정의한 조건을 모두 만족하는지 확인할 때 사용
- 빈 배열일 경우 언제나 `true`를 반환

```javascript
// 배열의 모든 요소가 3보다 큰지 확인
[5, 10, 15].every(item => item > 3); // -> true

// 배열의 모든 요소가 10보다 큰지 확인
[5, 10, 15].every(item => item > 10); // -> false

// every 메서드를 호출한 배열이 빈 배열인 경우 언제나 true를 반환한다.
[].every(item => item > 3); // -> true
```

### Array.prototype.find

`find 메서드`는 자신을 호출한 배열의 요소를 순회하면서 인수로 전달된 콜백 함수를 호출

- 반환값이 ture인 첫 번째 요소를 반환
- 콜백 함수의 반환값이 true인 요소가 존재하지 않는다면 `undefined`를 반환
- 콜백 함수의 반환값이 true인 첫 번째 요소를 반환하므로 find의 결과값은 배열이 아닌 해당 요소값임

```javascript
// Array#filter는 배열을 반환한다.
[1, 2, 2, 3].filter(item => item === 2); // -> [2, 2]

// Array#find는 요소를 반환한다.
[1, 2, 2, 3].find(item => item === 2); // -> 2
```

### Array.prototype.findIndex

`findIndex 메서드`는 자신을 호출한 배열의 요소를 순회하면서 인수로 전달된 콜백 함수를 호출

- 반환값이 ture인 첫 번째 요소의 인덱스를 반환
- 콜백 함수의 반환값이 ture인 요소가 존재하지 않으면 `-1`을 반환

### Array.prototype.flatMap

`findMap 메서드`는 map 메서드를 통해 생성된 새로운 배열을 평탄화함

- map 메서드와 flat 메서드를 순차적으로 실행하는 효과가 있음

```javascript
const arr = ['hello', 'world'];

// map과 flat을 순차적으로 실행
arr.map(x => x.split('')).flat();
// -> ['h', 'e', 'l', 'l', 'o', 'w', 'o', 'r', 'l', 'd']

// flatMap은 map을 통해 생성된 새로운 배열을 평탄화한다.
arr.flatMap(x => x.split(''));
// -> ['h', 'e', 'l', 'l', 'o', 'w', 'o', 'r', 'l', 'd']
```

- 평탄화 깊이를 지정할 수는 없고 1단계만 평탄화함
- 깊이를 지정해야 할 땐 map 메서드와 flat 메서드를 각각 호출
