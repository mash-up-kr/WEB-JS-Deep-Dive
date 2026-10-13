# 모던 자바스크립트 Deep Dive 26~27장 정리

## 26장. ES6 함수

### 함수의 구분

ES6 이전부터 사용하던 함수 선언문과 함수 표현식은 하나의 함수로 여러 역할을 할 수 있다.

```js
function foo() {
  return 1;
}

foo();      // 1. 일반 함수로 호출
new foo();  // 2. 생성자 함수로 호출

const obj = { foo };
obj.foo();  // 3. 객체의 메서드로 호출
```

호출할 수 있는 함수 객체를 `callable`, `new`로 인스턴스를 만들 수 있는 함수 객체를 `constructor`라고 한다. 위의 `foo`는 두 성질을 모두 가진다.

문제는 콜백이나 메서드처럼 객체를 만들 목적이 없는 함수도 생성자로 호출할 수 있다는 점이다.

```js
const double = function (number) {
  return number * 2;
};

[1, 2, 3].map(double); // 콜백으로 만든 함수
new double();           // 의도와 맞지 않지만 호출 가능
```

ES6는 기존 함수의 동작을 바꾸지 않고, 용도에 맞는 새로운 함수 문법을 추가했다.

```js
function normal() {}       // 일반 함수
const arrow = () => {};    // 화살표 함수

const obj = {
  method() {}              // ES6 메서드
};

new normal();     // 가능
new arrow();      // TypeError
new obj.method(); // TypeError
```

| 함수 종류 | 일반 호출 | `new` 호출 |
| --- | :---: | :---: |
| 함수 선언문·함수 표현식 | 가능 | 가능 |
| 메서드 축약 표현 | 가능 | 불가능 |
| 화살표 함수 | 가능 | 불가능 |

ES6가 기존 함수를 제거하거나 변경하지 않은 이유는 이미 작성된 코드와의 호환성을 지키기 위해서다. 지금도 함수 선언문과 함수 표현식은 예전처럼 생성자로 호출할 수 있다.

### 메서드

객체에 들어 있는 함수는 일상적으로 모두 메서드라고 부른다. 하지만 이 절에서 말하는 **ES6 메서드**는 메서드 축약 표현으로 정의한 함수다.

```js
const user = {
  name: 'Lee',

  sayHi: function () {
    return `안녕하세요, ${this.name}`;
  },

  sayBye() {
    return `안녕히 가세요, ${this.name}`;
  }
};

user.sayHi();  // 함수 표현식을 메서드 방식으로 호출
user.sayBye(); // ES6 메서드를 호출
```

두 함수 모두 `user.method()` 형태로 호출할 수 있지만 내부 성질은 다르다.

| 구분 | `sayHi: function () {}` | `sayBye() {}` |
| --- | :---: | :---: |
| 함수 종류 | 일반 함수 | ES6 메서드 |
| `new` 호출 | 가능 | 불가능 |
| `[[HomeObject]]` | 없음 | 있음 |
| `super` | 사용 불가 | 사용 가능 |

#### 메서드가 자신이 만들어진 객체를 기억하는 방법

메서드 축약 표현으로 만든 함수는 자신이 어느 객체에서 정의됐는지를 `[[HomeObject]]`라는 내부 정보로 기억한다. `super`는 이 정보를 이용해 해당 객체의 프로토타입에서 메서드를 찾는다.

```js
const person = {
  introduce() {
    return `제 이름은 ${this.name}입니다`;
  }
};

const student = {
  __proto__: person,
  name: '정우',

  introduce() {
    return `${super.introduce()}. 학생입니다`;
  }
};

student.introduce();
// 제 이름은 정우입니다. 학생입니다
```

실행 흐름은 다음과 같다.

1. `student.introduce`는 자신이 `student`에서 정의됐다는 것을 기억한다.
2. `super.introduce`는 `student`의 프로토타입인 `person`에서 `introduce`를 찾는다.
3. `person.introduce`를 실행하지만 `this`는 현재 호출 객체인 `student`를 유지한다.

```text
student.introduce
        ↓ [[HomeObject]]는 student
student의 프로토타입
        ↓
person.introduce
```

반면 프로퍼티 값으로 할당한 일반 함수에는 `[[HomeObject]]`가 없다. 어느 객체의 프로토타입에서 탐색해야 하는지 알 수 없으므로 `super`를 사용할 수 없다.

```js
const student = {
  __proto__: person,

  introduce: function () {
    return super.introduce(); // SyntaxError
  }
};
```

따라서 메서드 축약 표현은 단순히 코드를 짧게 쓰는 문법 설탕이 아니다. 일반 함수와 달리 생성자로 호출할 수 없고, `[[HomeObject]]`를 가지므로 `super`를 사용할 수 있다.

### 화살표 함수

화살표 함수는 함수를 짧게 작성하기 위한 문법이면서 일반 함수와 다른 동작을 가진다.

```js
const double = number => number * 2;

[1, 2, 3].map(double); // [2, 4, 6]
```

화살표 함수는 생성자로 호출할 수 없으며 자체 `this`와 `arguments`도 갖지 않는다.

| 구분 | 일반 함수 | 화살표 함수 |
| --- | :---: | :---: |
| `new` 호출 | 가능 | 불가능 |
| 자체 `this` | 있음 | 없음 |
| 자체 `arguments` | 있음 | 없음 |

#### 콜백에서 화살표 함수가 유용한 이유

다음 코드는 배열의 각 숫자에 인스턴스의 `factor`를 곱하려는 코드다.

```js
class Calculator {
  constructor(factor) {
    this.factor = factor;
  }

  multiply(numbers) {
    return numbers.map(function (number) {
      return number * this.factor;
    });
  }
}

const calculator = new Calculator(2);
calculator.multiply([1, 2, 3]); // TypeError
```

`multiply`의 `this`는 `calculator`지만, `map`은 콜백을 별도의 일반 함수로 호출한다. `map`에 콜백의 `this`를 따로 전달하지 않았고 클래스 코드는 strict mode이므로, 콜백 안의 `this`는 `undefined`가 된다.

화살표 함수에는 자체 `this`가 없다. 따라서 상위 스코프인 `multiply` 메서드의 `this`를 사용한다.

```js
class Calculator {
  constructor(factor) {
    this.factor = factor;
  }

  multiply(numbers) {
    return numbers.map(number => number * this.factor);
  }
}

new Calculator(2).multiply([1, 2, 3]); // [2, 4, 6]
```

이를 **lexical this**라고 한다. 화살표 함수가 `this`를 새로 만드는 것이 아니라, 일반 변수처럼 상위 스코프에서 `this`를 찾는 것이다.

#### 일반 함수 콜백에서 `this`를 사용하는 방법

`map`에서는 두 번째 인수로 콜백에서 사용할 `this`를 전달할 수 있다.

```js
multiply(numbers) {
  return numbers.map(function (number) {
    return number * this.factor;
  }, this);
}
```

`bind`로 `this`가 고정된 새 함수를 만들어도 된다.

```js
multiply(numbers) {
  return numbers.map(function (number) {
    return number * this.factor;
  }.bind(this));
}
```

두 방식 모두 가능하다. 차이는 `map`이 `thisArg`를 지원하기 때문에 첫 번째 방식을 사용할 수 있다는 점이다. 모든 API가 `thisArg`를 지원하는 것은 아니므로 그런 경우에는 `bind`나 화살표 함수를 사용한다.

| 상황 | 적합한 방식 |
| --- | --- |
| 상위 함수의 `this`를 사용하는 콜백 | 화살표 함수 |
| 호출하는 API가 정해 주는 `this`가 필요한 콜백 | 일반 함수 |
| 객체나 클래스의 일반 메서드 | 메서드 축약 표현 |

객체 메서드를 화살표 함수로 만들면 `obj.method()`로 호출해도 `obj`가 `this`가 되지 않으므로 주의해야 한다.

### Rest 파라미터

함수를 정의할 때 적은 이름은 **매개변수(parameter)**, 호출할 때 전달한 값은 **인수(argument)**다.

```js
function createUser(name, role, ...permissions) {
  console.log(name);        // 'Lee'
  console.log(role);        // 'admin'
  console.log(permissions); // ['read', 'write']
}

createUser('Lee', 'admin', 'read', 'write');
```

`name`과 `role`은 앞의 두 인수를 받고, Rest 파라미터 `permissions`는 남은 인수를 실제 배열로 받는다.

```text
'Lee'             → name
'admin'           → role
'read', 'write'   → permissions: ['read', 'write']
```

#### `arguments`와 Rest 파라미터

일반 함수 안에서는 전달된 모든 인수를 `arguments` 객체로 확인할 수 있다.

```js
function example(first, ...rest) {
  console.log(arguments); // [10, 20, 30] 전체
  console.log(rest);      // [20, 30] 나머지
}

example(10, 20, 30);
```

| 구분 | `arguments` | Rest 파라미터 |
| --- | --- | --- |
| 담는 값 | 전달된 모든 인수 | 앞의 매개변수가 받고 남은 인수 |
| 형태 | 유사 배열 객체 | 실제 배열 |
| 배열 메서드 | 직접 사용 불가 | 바로 사용 가능 |

`arguments`는 함수 객체에 영구적으로 붙은 프로퍼티가 아니다. 일반 함수가 호출될 때마다 해당 호출을 위해 새로 만들어지는 지역 바인딩이다.

```js
function foo() {
  return arguments;
}

const first = foo(1, 2);
const second = foo(3, 4);

first === second; // false
```

화살표 함수에는 자체 `arguments`가 없으므로 가변 인수를 받으려면 Rest 파라미터를 사용한다.

```js
const sum = (...numbers) =>
  numbers.reduce((total, number) => total + number, 0);

sum(1, 2, 3); // 6
```

Rest와 Spread는 같은 `...` 문법을 사용하지만 방향이 반대다.

```js
const numbers = [1, 2, 3];

function sum(...values) { // Rest: 여러 인수를 배열로 모음
  return values.reduce((a, b) => a + b, 0);
}

sum(...numbers); // Spread: 배열을 여러 인수로 펼침
```

Rest 파라미터는 반드시 마지막에 하나만 선언할 수 있으며 함수의 `length`에는 포함되지 않는다.

## 27장. 배열

### 배열이란?

배열은 여러 값을 순서대로 저장하고, 각 값에 번호를 붙여 관리하는 자료구조다.

```js
const fruits = ['apple', 'banana', 'orange'];

fruits[0];     // 'apple'
fruits[1];     // 'banana'
fruits.length; // 3
```

배열에 저장된 값을 **요소**, 요소의 위치를 나타내는 번호를 **인덱스**라고 한다. 인덱스는 0부터 시작한다.

자바스크립트 배열에는 숫자, 문자열, 객체, 함수 등 모든 종류의 값을 함께 넣을 수 있다.

```js
const values = [
  1,
  'hello',
  { name: 'Lee' },
  () => 'function'
];
```

배열의 핵심은 단순히 값을 여러 개 저장하는 것이 아니다. **값의 순서와 길이가 있으므로 처음부터 끝까지 순회하기 좋다**는 점이다.

### 자바스크립트 배열은 일반적인 배열과 다르다

자료구조에서 말하는 전통적인 배열은 같은 크기의 값이 연속된 메모리 공간에 저장되는 구조다.

```text
시작 주소가 1000이고 각 요소가 8바이트라면

인덱스 0 → 1000
인덱스 1 → 1008
인덱스 2 → 1016
```

각 요소의 주소를 계산할 수 있으므로 인덱스를 통한 접근이 매우 빠르다. 대신 중간에 값을 삽입하거나 삭제하면 뒤의 요소들을 이동해야 한다.

자바스크립트 배열은 언어의 관점에서 **숫자처럼 보이는 프로퍼티 키와 `length`를 가진 특수한 객체**다.

```js
const numbers = [10, 20, 30];

typeof numbers; // 'object'

Object.getOwnPropertyDescriptors(numbers);
// {
//   '0': { value: 10, ... },
//   '1': { value: 20, ... },
//   '2': { value: 30, ... },
//   length: { value: 3, ... }
// }
```

배열의 인덱스도 실제로는 객체의 프로퍼티 키처럼 동작한다.

```js
const numbers = [10, 20];

numbers[0];   // 10
numbers['0']; // 10
```

그래서 자바스크립트 배열에는 다음과 같은 동작이 가능하다.

```js
const arr = [];

arr[0] = '첫 번째 값';
arr[10] = '열한 번째 값';

arr.length; // 11
// 인덱스 1~9에는 요소가 없는 희소 배열이 된다.
```

| 전통적인 배열 | 자바스크립트 배열 |
| --- | --- |
| 같은 크기의 요소가 연속해서 저장됨 | 서로 다른 타입의 값을 저장할 수 있음 |
| 요소 사이에 빈 공간이 없음 | 중간 인덱스가 비어 있는 희소 배열 가능 |
| 메모리 주소 계산으로 요소에 접근 | 객체의 인덱스 프로퍼티처럼 요소를 다룸 |

따라서 “자바스크립트 배열은 배열이 아니다”라는 말은 **C 언어의 배열처럼 고정된 연속 메모리만을 의미하지 않는다**는 뜻이다. 실제 자바스크립트 엔진은 성능을 위해 값이 빽빽한 배열을 연속된 메모리 형태로 최적화하기도 한다. 모든 자바스크립트 배열이 항상 해시 테이블로만 동작한다고 이해할 필요는 없다.

실무에서는 가능하면 인덱스를 건너뛰지 않고 같은 성격의 데이터를 연속해서 저장하는 편이 엔진 최적화와 코드 이해에 유리하다.

### 유사 배열 객체와 이터러블

#### 유사 배열 객체

유사 배열 객체는 인덱스처럼 생긴 프로퍼티와 `length`를 가져 배열처럼 보이는 객체다.

```js
const arrayLike = {
  0: 'apple',
  1: 'banana',
  length: 2
};

arrayLike[0];     // 'apple'
arrayLike.length; // 2
```

하지만 실제 배열은 아니므로 배열 메서드를 바로 사용할 수 없다.

```js
Array.isArray(arrayLike); // false
// arrayLike.map(...)     // TypeError
```

일반 함수의 `arguments` 객체와 일부 DOM 컬렉션이 대표적인 유사 배열 객체다.

#### 이터러블

이터러블은 `Symbol.iterator` 메서드를 가지고 있어 값을 하나씩 꺼낼 수 있는 객체다. 배열, 문자열, `Set`, `Map` 등이 이터러블이다.

```js
const set = new Set(['apple', 'banana']);

for (const fruit of set) {
  console.log(fruit);
}

const fruits = [...set]; // ['apple', 'banana']
```

두 개념의 기준은 서로 다르다.

| 구분 | 판단 기준 | 할 수 있는 일 |
| --- | --- | --- |
| 유사 배열 객체 | 인덱스 프로퍼티와 `length` | 인덱스로 값에 접근 |
| 이터러블 | `Symbol.iterator` 보유 | `for...of`, Spread 사용 |

배열은 인덱스와 `length`를 가지면서 `Symbol.iterator`도 제공하므로 유사 배열의 특징과 이터러블의 특징을 모두 갖는다. 유사 배열 객체나 이터러블을 실제 배열로 바꿀 때는 `Array.from`을 사용할 수 있다.

```js
Array.from(arrayLike); // ['apple', 'banana']
Array.from(set);       // ['apple', 'banana']

Array.from({ length: 5 }, (_, index) => index + 1);
// [1, 2, 3, 4, 5]
```

### 기본 배열 메서드

기본 메서드는 사용법을 모두 외우기보다 **원본 배열을 변경하는지**를 함께 확인하는 것이 중요하다.

| 목적 | 메서드 | 원본 변경 | 핵심 동작 |
| --- | --- | :---: | --- |
| 끝에 추가 | `push` | O | 추가 후 배열 길이 반환 |
| 끝에서 제거 | `pop` | O | 제거한 요소 반환 |
| 앞에 추가 | `unshift` | O | 추가 후 배열 길이 반환 |
| 앞에서 제거 | `shift` | O | 제거한 요소 반환 |
| 중간 추가·삭제 | `splice` | O | 제거한 요소들을 배열로 반환 |
| 일부 복사 | `slice` | X | 선택한 범위의 새 배열 반환 |
| 배열 결합 | `concat` | X | 결합한 새 배열 반환 |
| 포함 여부 | `includes` | X | 불리언 반환 |
| 순서 반전 | `reverse` | O | 원본 순서를 뒤집음 |
| 정렬 | `sort` | O | 기본값은 문자열 기준 정렬 |

```js
const numbers = [1, 2, 3];

numbers.push(4); // [1, 2, 3, 4]
numbers.pop();   // 4
numbers.shift(); // 1
```

숫자를 정렬할 때는 비교 함수를 전달해야 한다.

```js
[10, 2, 1].sort();              // [1, 10, 2]
[10, 2, 1].sort((a, b) => a-b); // [1, 2, 10]
```

`delete arr[index]`는 요소를 제거해도 빈자리를 남기므로 배열 삭제에는 보통 `splice`나 `filter`를 사용한다.

### 실무에서 자주 쓰는 배열 고차 함수

다음 주문 데이터를 여러 방식으로 처리해 보자.

```js
const orders = [
  { id: 1, userId: 10, price: 12000, status: 'paid' },
  { id: 2, userId: 20, price: 8000, status: 'pending' },
  { id: 3, userId: 10, price: 15000, status: 'paid' }
];
```

| 필요한 결과 | 적합한 메서드 | 반환값 | 새 배열 생성 |
| --- | --- | --- | :---: |
| 각 요소로 부수 효과 실행 | `forEach` | `undefined` | X |
| 모든 요소를 다른 값으로 변환 | `map` | 새 배열 | O |
| 조건에 맞는 요소들만 선택 | `filter` | 새 배열 | O |
| 여러 요소를 하나의 결과로 누적 | `reduce` | 누적한 값 | 작성 방식에 따라 다름 |
| 조건을 만족하는 요소가 하나라도 있는지 확인 | `some` | 불리언 | X |
| 모든 요소가 조건을 만족하는지 확인 | `every` | 불리언 | X |
| 조건을 만족하는 첫 번째 요소 찾기 | `find` | 요소 또는 `undefined` | X |
| 조건을 만족하는 첫 번째 위치 찾기 | `findIndex` | 인덱스 또는 `-1` | X |

모든 배열 메서드가 새 배열을 만드는 것은 아니다. 위 메서드 중 자동으로 새 배열을 만드는 것은 `map`과 `filter`다. `reduce`는 숫자, 문자열, 객체, 배열 등 콜백이 누적한 값을 반환하므로 초기값과 콜백 작성 방식에 따라 결과가 달라진다.

`map`과 `filter`가 새 배열을 반환하더라도 내부의 객체까지 자동으로 복사하는 것은 아니다.

```js
const original = [{ id: 1, name: '하늘' }];
const filtered = original.filter(artist => artist.id === 1);

original !== filtered;       // true: 배열 자체는 새로 생성
original[0] === filtered[0]; // true: 요소 객체는 같은 참조
```

객체 요소까지 새로 만들고 싶다면 `map`에서 새 객체를 반환한다.

```js
const copied = original.map(artist => ({ ...artist }));

original !== copied;       // true
original[0] !== copied[0]; // true
```

#### `forEach`: 반환값이 필요 없는 작업

```js
orders.forEach(order => {
  console.log(`주문 ${order.id}: ${order.status}`);
});
```

로그 출력, DOM 갱신처럼 각 요소로 부수 효과를 실행할 때 사용한다. 새 배열이 필요하면 `forEach`에서 외부 배열에 `push`하기보다 `map`이나 `filter`를 먼저 고려한다.

`forEach`는 `break`로 중단할 수 없고 비동기 콜백의 완료도 기다려 주지 않는다. 중단이나 순차적인 `await`가 필요하면 `for...of`가 더 적합하다.

#### `map`: 같은 개수의 값을 변환

```js
const summaries = orders.map(order => ({
  id: order.id,
  priceText: `${order.price.toLocaleString()}원`
}));
```

원본 요소마다 새로운 값을 하나씩 만들기 때문에 반환 배열의 길이는 원본과 같다. API 응답을 화면에 필요한 형태로 바꿀 때 자주 사용한다.

실무에서는 서버에서 받은 객체 전체를 그대로 전달하기보다, 사용하는 곳에 필요한 필드만 골라 새로운 데이터 형태로 바꿀 때 `map`을 자주 사용한다.

```js
const artists = [
  {
    id: 1,
    name: '하늘',
    age: 24,
    agency: 'Blue Sound',
    albums: ['First Light', 'Summer Night'],
    privateMemo: '내부 관리용 정보'
  },
  {
    id: 2,
    name: '민서',
    age: 27,
    agency: 'Star Music',
    albums: ['Beginning', 'Home'],
    privateMemo: '내부 관리용 정보'
  }
];

const artistCards = artists.map(({ id, name, age }) => ({
  id,
  name,
  age
}));

// [
//   { id: 1, name: '하늘', age: 24 },
//   { id: 2, name: '민서', age: 27 }
// ]
```

원본 배열의 요소 개수는 유지하면서 각 요소의 형태만 바뀐다. 이런 변환은 다음 상황에서 유용하다.

- API 응답을 화면 컴포넌트가 사용할 형태로 변환할 때
- 도메인 객체에서 외부에 전달할 필드만 선택할 때
- 서버 요청에 필요한 값만 골라 요청 본문을 만들 때
- 단위, 날짜, 금액 등을 표시용 문자열로 바꿀 때

조건에 맞는 일부 요소만 고른 뒤 형태도 바꿔야 한다면 `filter`와 `map`을 연결한다.

```js
const blueSoundArtistCards = artists
  .filter(artist => artist.agency === 'Blue Sound')
  .map(({ id, name }) => ({ id, name }));
```

`filter`는 **어떤 요소를 남길지**, `map`은 **남은 요소를 어떤 형태로 바꿀지** 담당한다.

#### `filter`: 조건에 맞는 여러 요소 선택

```js
const paidOrders = orders.filter(order => order.status === 'paid');
```

검색 조건, 상태 탭, 삭제된 항목 제외처럼 조건을 만족하는 여러 요소가 필요할 때 사용한다. 결과는 항상 배열이며, 조건을 만족하는 요소가 없으면 빈 배열을 반환한다.

#### `find`: 조건에 맞는 하나의 요소 찾기

```js
const order = orders.find(order => order.id === 2);
// { id: 2, userId: 20, price: 8000, status: 'pending' }
```

고유한 ID로 하나를 찾을 때는 `filter(...)[0]`보다 의도가 명확하다. 첫 번째 요소를 찾으면 순회를 멈추며, 찾지 못하면 `undefined`를 반환한다.

#### `some`: 하나라도 조건을 만족하는지 확인

```js
const hasPendingOrder = orders.some(
  order => order.status === 'pending'
);

// true
```

`some`은 조건에 맞는 요소들을 가져오는 메서드가 아니다. 콜백이 한 번이라도 `true`를 반환하면 즉시 순회를 멈추고 `true`를 반환한다.

실무에서는 다음과 같은 질문에 사용한다.

- 처리되지 않은 주문이 하나라도 있는가?
- 같은 이메일을 가진 사용자가 이미 있는가?
- 현재 사용자가 필요한 권한 중 하나라도 가지고 있는가?
- 입력값 중 오류가 하나라도 있는가?

빈 배열에서 `some`을 호출하면 확인할 요소가 없으므로 `false`다.

#### `every`: 모든 요소가 조건을 만족하는지 확인

```js
const allPricesAreValid = orders.every(order => order.price > 0);
// true
```

콜백이 한 번이라도 `false`를 반환하면 즉시 순회를 멈춘다. 모든 입력값의 검증 통과 여부나 모든 작업의 완료 여부를 확인할 때 적합하다.

빈 배열에서 `every`를 호출하면 반례가 없으므로 `true`를 반환한다는 점에 주의한다.

#### `reduce`: 배열을 하나의 결과로 누적

```js
const paidTotal = orders
  .filter(order => order.status === 'paid')
  .reduce((total, order) => total + order.price, 0);

// 27000
```

합계, 평균, 개수 집계, 객체 그룹화처럼 여러 요소를 하나의 값으로 만들 때 사용한다. 초기값을 생략하면 빈 배열에서 오류가 발생하고 누적값의 타입도 파악하기 어려워지므로 항상 초기값을 전달하는 편이 안전하다.

메서드는 하나만 고집하지 않고 질문의 형태에 맞춰 선택한다.

```text
값을 바꿀 것인가?        → map
여러 요소를 고를 것인가? → filter
한 요소를 찾을 것인가?   → find
하나라도 존재하는가?     → some
모두 만족하는가?         → every
하나의 결과로 합칠 것인가? → reduce
반환값 없이 작업할 것인가? → forEach
```
