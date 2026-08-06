# 모던 자바스크립트 Deep Dive 4~7장 정리

변수, 표현식, 데이터 타입, 연산자 중심의 기본 문법 정리

## 4장. 변수

### 변수와 식별자

- **변수**: 값을 저장하기 위해 확보한 메모리 공간 또는 해당 공간의 이름
- **식별자**: 값 자체가 아닌 값의 위치를 찾기 위한 이름
- **할당**: 변수에 값 저장
- **참조**: 변수에 저장된 값 읽기

```js
const score = 80;
```

- `score`: 식별자
- `80`: 값

### 변수 선언

- **선언**: 변수 이름을 실행 컨텍스트에 등록
- **초기화**: 메모리 공간 확보와 초기값 할당

`var` 선언은 선언과 초기화가 함께 진행된다.

```js
var score;
console.log(score); // undefined
```

선언하지 않은 식별자에 접근하면 `ReferenceError`가 발생한다.

```js
console.log(userName); // ReferenceError
```

### 호이스팅

코드 실행 전 선언문을 먼저 처리하는 자바스크립트 엔진의 평가 과정이다. 선언문이 코드 상단으로 이동한 것처럼 보이는 현상을 호이스팅이라 한다.

```js
console.log(score); // undefined
var score;

score = 80;
console.log(score); // 80
```

- 변수 선언: 런타임 이전 처리
- 값 할당: 런타임 처리
- 실제 선언문이 이동한 것이 아니라 평가 과정의 결과

### 선언 키워드와 네이밍

```js
const MAX_RETRY_COUNT = 3;
let currentPage = 1;
```

- 변경 없는 값: `const`
- 재할당이 필요한 값: `let`
- `var`: 함수 레벨 스코프와 호이스팅 특성으로 새 코드에서는 사용을 지양
- 변수·함수: `camelCase`
- 클래스·생성자 함수: `PascalCase`

## 5장. 표현식과 문

### 값과 리터럴

- **값**: 표현식의 평가 결과
- **리터럴**: 값을 만들기 위한 표기법

```js
10;
'hello';
true;
null;
[];
{};
function () {};
```

### 표현식

값으로 평가 가능한 코드다.

```js
10 + 20;
score;
user.name;
getUser();
```

- 리터럴 표현식: `10`, `'hello'`
- 식별자 표현식: `score`, `user.name`
- 연산자 표현식: `10 + 20`, `score === 80`
- 함수 호출 표현식: `getUser()`

### 문

프로그램을 구성하는 최소 실행 단위다.

```js
let score; // 변수 선언문
score = 80; // 할당문

if (score >= 60) {
  console.log('pass');
}
```

문에는 선언문, 할당문, 조건문, 반복문 등이 있다.

### 표현식인 문과 표현식이 아닌 문

할당문은 값으로 평가 가능한 표현식인 문이다.

```js
let score;
const result = (score = 100);

console.log(result); // 100
```

변수 선언문과 조건문은 값으로 평가할 수 없다.

```js
let score;

// const result = let userName; // SyntaxError
// const result = if (score > 60) {}; // SyntaxError
```

### 세미콜론 자동 삽입

- 세미콜론: 문의 종료 표시
- ASI(Automatic Semicolon Insertion): 세미콜론 자동 삽입 기능
- 줄바꿈 위치에 따라 의도와 다른 해석이 발생할 수 있음
- 블록으로 끝나는 `if`, `for`, 함수 선언문 뒤에는 세미콜론을 사용하지 않음

> 표현식은 값이 필요한 자리이고, 문은 실행 흐름을 구성한다.

## 6장. 데이터 타입

### 타입 구분

| 구분 | 타입 |
| --- | --- |
| 원시 타입 | `number`, `string`, `boolean`, `undefined`, `null`, `symbol`, `bigint` |
| 객체 타입 | 객체, 배열, 함수 등 |

`bigint`는 책 초판 이후 추가된 원시 타입이다.

### 숫자 타입

정수와 실수를 구분하지 않는 배정밀도 64비트 부동소수점 방식이다.

특별한 값으로 `Infinity`, `-Infinity`, `NaN`이 있다.

```js
1 === 1.0; // true
3 / 2; // 1.5

10 / 0; // Infinity
10 / -0; // -Infinity
1 * 'hello'; // NaN
```

부동소수점 계산에는 오차가 발생할 수 있다.

```js
0.1 + 0.2; // 0.30000000000000004
```

`NaN`은 자기 자신과도 일치하지 않는다.

```js
NaN === NaN; // false
Number.isNaN(NaN); // true
```

전역 `isNaN`은 인자를 숫자로 변환한 뒤 검사할 수 있으므로, `NaN` 검사에는 `Number.isNaN`을 사용한다.

### 문자열과 템플릿 리터럴

문자열은 변경 불가능한 원시 값이다. 템플릿 리터럴은 백틱으로 표현한다.

```js
const name = '창우';
const message = `안녕하세요, ${name}님`;
```

템플릿 리터럴은 멀티라인 문자열과 `${}`를 통한 표현식 삽입을 지원하며, 문자열 연결이 많을 때 `+`보다 가독성이 좋다.

### `undefined`와 `null`

- `undefined`: 값이 미할당된 상태
- `null`: 의도적인 값의 부재

```js
let selectedUser; // undefined
const deletedUser = null;
```

`null`을 할당한다고 즉시 메모리가 해제되는 것은 아니다. 참조가 끊긴 값이 가비지 컬렉션 대상이 되며, 실제 정리 시점은 엔진이 결정한다.

### `symbol`과 `bigint`

`symbol`은 중복되지 않는 유일한 값으로, 객체의 고유 프로퍼티 키에 사용한다.

```js
const key = Symbol('key');
const user = {
  [key]: 'value',
};
```

`bigint`는 안전한 정수 범위를 넘는 큰 정수에 사용한다.

```js
const max = Number.MAX_SAFE_INTEGER;
const largeNumber = 9_007_199_254_740_993n;
```

`bigint`와 `number`는 산술 연산을 혼합할 수 없으며, 큰 정수 계산이 필요한 경우에만 사용한다.

### 확인할 내용

- 값 할당 시 타입 결정 방식
- `undefined`와 `null`의 역할 구분
- 부동소수점 오차와 `NaN`
- 암묵적 타입 변환 발생 지점

## 7장. 연산자

### 산술 연산자

```js
10 + 3; // 13
10 - 3; // 7
10 * 3; // 30
10 / 3; // 3.333...
10 % 3; // 1
```

`+`는 문자열 연결에도 사용된다.

```js
'1' + 2; // '12'
1 + true; // 2
1 + null; // 1
1 + undefined; // NaN
```

- 암묵적 타입 변환은 의도하지 않은 결과를 만들 수 있음
- 타입 변환이 필요할 때는 `Number()`, `String()`, `Boolean()` 활용

```js
const quantity = Number(inputValue);
```

### 할당 연산자와 부수 효과

```js
let score = 10;

score += 5;
score -= 3;
```

부수 효과는 코드 실행 전후의 상태 변화다. 할당 연산자, 증감 연산자, `delete` 연산자가 대표적이다.

`++`, `--`는 반환 시점과 값 변경 시점을 혼동할 수 있으므로 주의한다.

```js
let index = 0;
index += 1;
```

### 비교 연산자

```js
5 === 5; // true
5 === '5'; // false
5 == '5'; // true
```

- `==`, `!=`: 비교 전 암묵적 타입 변환
- `===`, `!==`: 값과 타입을 동시에 비교
- 기본 비교에는 `===`, `!==` 사용

```js
0 == ''; // true
0 === ''; // false

Object.is(NaN, NaN); // true
Object.is(0, -0); // false
```

`Object.is`는 `NaN`, `-0` 같은 특수 값의 비교가 필요한 상황에서 사용한다.

### 논리 연산자와 단축 평가

```js
true && false; // false
true || false; // true
!true; // false

'cat' && 'dog'; // 'dog'
'cat' || 'dog'; // 'cat'
```

`&&`, `||`는 불리언 외 피연산자의 값을 반환할 수 있다. 단축 평가는 결과가 확정되면 이후 표현식을 평가하지 않는다.

### `||`와 `??`

```js
const count = 0;

count || 10; // 10
count ?? 10; // 0
```

- `||`: falsy 값(`0`, `false`, `''`, `null`, `undefined`)이면 기본값 사용
- `??`: `null`, `undefined`일 때만 기본값 사용
- `0`, 빈 문자열, `false`를 유지해야 하면 `??` 사용

### 우선순위

우선순위가 한눈에 들어오지 않는 식에는 괄호를 사용한다.

```js
10 * 2 + 3; // 23
10 * (2 + 3); // 50
```
