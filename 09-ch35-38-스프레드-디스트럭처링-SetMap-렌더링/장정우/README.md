# 모던 자바스크립트 Deep Dive 35~38장 정리

## 35장. 스프레드 문법

### 스프레드 문법이란?

스프레드 문법 `...`은 이터러블을 펼쳐 **개별 값의 목록**으로 만든다.   
배열, 문자열, Set, Map 등은 이터러블이므로 펼칠 수 있다.  

```js
console.log(...[1, 2, 3]); // 1 2 3
console.log(...'ABC');     // A B C
console.log(...new Set([1, 2, 2])); // 1 2
```

스프레드 문법의 결과는 독립적인 값이 아니다. 함수 호출의 인수 목록, 배열 리터럴의 요소 목록처럼 **여러 값을 받는 자리**에서 사용한다.

```js
const numbers = [10, 20, 30];

Math.max(...numbers); // Math.max(10, 20, 30)과 같다

// const values = ...numbers; // SyntaxError: 펼친 결과를 변수 하나에 담을 수 없음
```

8주차에서 다룬 Rest 파라미터도 `...`을 쓰지만 방향이 반대다.

```js
function sum(...numbers) { // Rest: 여러 인수를 배열 하나로 모음
  return numbers.reduce((total, number) => total + number, 0);
}

sum(...[1, 2, 3]); // Spread: 배열을 개별 인수로 펼침 → 6
```

| 문법 | 사용 위치 | 동작 |
| --- | --- | --- |
| Rest 파라미터 | 함수 정의 | 여러 인수를 배열로 모음 |
| 스프레드 문법 | 함수 호출, 배열·객체 리터럴 | 값을 펼침 |

### 배열에서의 활용

배열을 펼치면 `concat`이나 `splice`로 하던 일을 간결하게 표현할 수 있다.

```js
const front = [1, 2];
const back = [3, 4];

const combined = [...front, ...back]; // [1, 2, 3, 4]
const inserted = [0, ...front, 3];    // [0, 1, 2, 3]
```

이터러블이 아닌 유사 배열 객체는 배열 안에서 바로 펼칠 수 없다. 이때는 `Array.from`으로 배열을 만든다.

```js
const arrayLike = { 0: 'a', 1: 'b', length: 2 };

// [...arrayLike];       // TypeError: 이터러블이 아님
Array.from(arrayLike);   // ['a', 'b']
```

### 객체에서의 활용

객체 리터럴 안에서는 객체의 **자신이 가진 열거 가능한 프로퍼티**를 새 객체로 복사할 수 있다.  
일반 객체가 이터러블이 아니어도 객체 리터럴의 스프레드는 사용할 수 있다.

```js
const user = { name: 'Kim', role: 'user' };
const copy = { ...user };

console.log(copy);          // { name: 'Kim', role: 'user' }
console.log(copy === user); // false

// [...user]; // TypeError: 일반 객체는 이터러블이 아님
```

여러 객체를 펼칠 때 같은 프로퍼티가 있으면 **뒤에 나온 값이 앞의 값을 덮는다**.

```js
const base = { name: 'Kim', role: 'user' };

const admin = { ...base, role: 'admin' };
// { name: 'Kim', role: 'admin' }

const stillUser = { role: 'admin', ...base };
// { role: 'user', name: 'Kim' }
```

따라서 기본 설정을 만들고 사용자 설정으로 덮을 때는 순서가 중요하다.

```js
const defaults = { theme: 'light', pageSize: 20 };
const options = { pageSize: 50 };

const settings = { ...defaults, ...options };
// { theme: 'light', pageSize: 50 }
```

### 주의할 점

스프레드로 만든 배열과 객체는 **얕은 복사**다. 바깥 컨테이너는 새로 만들어지지만, 안에 들어 있던 객체는 같은 객체를 참조한다.

```js
const original = [{ name: 'Kim' }];
const copy = [...original];

copy[0].name = 'Lee';
console.log(original[0].name); // 'Lee'
console.log(copy === original);    // false
console.log(copy[0] === original[0]); // true
```

객체 스프레드도 중첩 객체까지 자동으로 복사하지 않는다.

```js
const user = { profile: { city: 'Seoul' } };
const copiedUser = { ...user };

copiedUser.profile.city = 'Busan';
console.log(user.profile.city); // 'Busan'
```

## 36장. 디스트럭처링 할당

디스트럭처링 할당은 배열이나 객체의 값을 꺼내 여러 변수에 나누어 담는 문법이다.  
값을 꺼내는 쪽의 구조에 맞춰 변수를 선언한다.  

### 배열 디스트럭처링

배열은 **위치**를 기준으로 할당한다. 왼쪽 변수의 수와 오른쪽 요소의 수가 같을 필요는 없다.

```js
const colors = ['red', 'green', 'blue'];
const [first, second] = colors;

console.log(first);  // 'red'
console.log(second); // 'green'

const [a, , c] = colors;
console.log(a, c);   // 'red' 'blue'
```

나머지 요소를 모으는 Rest 요소는 반드시 마지막에 둔다.

```js
const [head, ...tail] = [1, 2, 3, 4];

console.log(head); // 1
console.log(tail); // [2, 3, 4]
```

배열 디스트럭처링의 오른쪽 값은 배열에 한정되지 않고 **이터러블**이면 된다.

```js
const [firstChar, secondChar] = 'JS';
console.log(firstChar, secondChar); // 'J' 'S'

const [firstNumber] = new Set([3, 1, 2]);
console.log(firstNumber); // 3
```

변수 교환이나 함수의 여러 반환값을 받는 데도 사용할 수 있다.

```js
let left = 1;
let right = 2;
[left, right] = [right, left];
console.log(left, right); // 2 1

function getCoordinates() {
  return [127.0, 37.5];
}

const [longitude, latitude] = getCoordinates();
```

### 객체 디스트럭처링

객체는 배열과 달리 **프로퍼티 이름**을 기준으로 할당한다. 작성 순서는 중요하지 않다.

```js
const user = { name: 'Kim', age: 28 };
const { age, name } = user;

console.log(name, age); // 'Kim' 28
```

여기서 `name`이라는 변수가 선언되는 것이 아니다. `name`은 찾을 프로퍼티이고, 실제로 선언되는 변수는 `displayName`이다.

없는 프로퍼티의 값은 `undefined`가 된다. 기본값은 배열과 마찬가지로 값이 `undefined`일 때 적용된다.

```js
const { title = '제목 없음' } = { title: undefined };
console.log(title); // '제목 없음'

const { content = '내용 없음' } = { content: null };
console.log(content); // null
```

나머지 프로퍼티는 새 객체로 모을 수 있다.

```js
const { id, ...details } = { id: 1, name: 'Kim', age: 28 };

console.log(id);      // 1
console.log(details); // { name: 'Kim', age: 28 }
```

### 중첩 구조와 함수 매개변수

중첩된 객체에서는 어느 단계의 프로퍼티를 꺼내는지 구조를 그대로 적는다.

```js
const user = {
  name: 'Kim',
  address: { city: 'Seoul', zipCode: '03000' }
};

const {
  name,
  address: { city }
} = user;

console.log(name, city); // 'Kim' 'Seoul'
```

함수 매개변수에 적용하면 필요한 프로퍼티만 이름으로 받을 수 있다.

```js
function printUser({ name, role = 'guest' }) {
  console.log(`${name}: ${role}`);
}

printUser({ name: 'Kim', role: 'admin' }); // 'Kim: admin'
printUser({ name: 'Lee' });                // 'Lee: guest'
```

객체 자체를 전달하지 않거나 중첩 객체가 빠질 수 있다면 기본값을 적어야 한다.

```js
function getCity({ address: { city } = {} } = {}) {
  return city;
}

getCity({ address: { city: 'Seoul' } }); // 'Seoul'
getCity({});                             // undefined
getCity();                               // undefined
```

## 37장. Set과 Map

`Set`은 **값만** 저장하는 중복 없는 모음이고, `Map`은 **키와 값의 쌍**을 저장한다.

### 앞서 이해하면 좋은 내용

숫자나 문자열 같은 원시 값은 값으로 비교한다. 하지만 객체는 프로퍼티가 똑같아도 **서로 따로 만든 객체라면 다른 값**으로 취급한다.

```js
console.log(1 === 1); // true

const a = { id: 1 };
const b = { id: 1 };
const sameAsA = a;

console.log(a === b);       // false: 내용은 같아도 다른 객체
console.log(a === sameAsA); // true: 같은 객체를 가리킴
```

이를 메모리 관점에서 개념적으로 그리면 다음과 같다.

```text
a       ──────→ 객체 A { id: 1 }
sameAsA ──────↗

b       ──────→ 객체 B { id: 1 }
```

`a`와 `sameAsA`는 같은 객체를 가리키고, `b`는 별도로 만들어진 객체를 가리킨다.

객체를 복사하는 것과 같은 객체를 다른 변수에 담는 것도 구분해야 한다.

```js
const original = { id: 1 };
const alias = original;
const copy = { ...original };

console.log(original === alias); // true
console.log(original === copy);  // false
```

앞에서 배운 스프레드로 만든 `copy`는 내용이 같아도 **새 객체**다. 이 차이가 Set의 중복 판단과 Map의 키 검색에 그대로 적용된다.

### Set

#### Set은 언제 사용하는가?

Set은 같은 값을 한 번만 저장한다. 이미 처리한 사용자 ID처럼 **존재 여부만** 알아야 할 때 알맞다.

```js
const processedIds = new Set([101, 102, 101]);

console.log(processedIds.size);     // 2
console.log(processedIds.has(101)); // true
```

#### 값의 중복 판단

원시 값은 같은 값을 다시 넣으면 중복으로 처리한다. 객체는 **같은 객체를 다시 넣을 때만** 중복이다.

```js
const a = { id: 1 };
const b = { id: 1 };
const set = new Set([a, b, a]);

console.log(set.size);           // 2: a와 b는 서로 다른 객체
console.log(set.has(a));         // true
console.log(set.has({ id: 1 })); // false: 여기서 새로 만든 객체
```

Set은 값 비교에 `SameValueZero` 방식의 동일성 판단을 사용한다. 일반적인 `===`와 거의 같지만 `NaN`을 자기 자신과 같은 값으로 취급한다. `+0`과 `-0`도 하나로 취급한다.

```js
console.log(NaN === NaN); // false

const set = new Set([NaN, NaN, +0, -0]);
console.log(set.size); // 2: NaN 하나, 0 하나
```

#### 주요 메서드와 반환값

| 사용법 | 역할 | 반환값 |
| --- | --- | --- |
| `set.size` | 값의 개수 확인 | 숫자 |
| `set.add(value)` | 값 추가 | Set 자신 |
| `set.has(value)` | 값의 존재 확인 | 불리언 |
| `set.delete(value)` | 값 삭제 | 삭제 성공 여부를 나타내는 불리언 |
| `set.clear()` | 모든 값 삭제 | `undefined` |

```js
const set = new Set([1, 2]);

set.add(3).add(3);    // add는 Set을 반환하므로 연결 가능
set.delete(2);        // true
set.delete(9);        // false: 없는 값
console.log([...set]); // [1, 3]
```

`delete`는 Set을 반환하지 않으므로 `set.delete(1).delete(3)`처럼 연결할 수 없다.

#### 순회와 집합 연산

Set은 삽입한 순서대로 순회할 수 있고, 스프레드 문법으로 배열로 바꿀 수 있다.

```js
const set = new Set(['kim', 'lee', 'kim']);

for (const name of set) {
  console.log(name); // 'kim', 'lee' 순서
}

console.log([...set]); // ['kim', 'lee']
```

두 집합을 비교할 때는 `has`로 공통 요소를 확인하는 방식이 기본이다.

```js
const a = new Set([1, 2, 3]);
const b = new Set([2, 3, 4]);

const common = new Set([...a].filter(value => b.has(value)));
console.log([...common]); // [2, 3]
```

### Map

#### Map은 언제 사용하는가?

Map은 키마다 값을 연결해 저장한다. 사용자 ID별 마지막 처리 시간처럼 **키와 그에 대응하는 값**이 필요할 때 사용한다.

```js
const lastProcessedAt = new Map();

lastProcessedAt.set(101, '10:30');
lastProcessedAt.set(101, '11:00');

console.log(lastProcessedAt.size);     // 1
console.log(lastProcessedAt.get(101)); // '11:00'
```

일반 객체의 프로퍼티 키는 문자열이나 Symbol이다. Map은 숫자, 객체, 함수 등 **어떤 값이든 키**로 사용할 수 있다.

```js
const object = { 1: 'one' };
console.log(Object.keys(object)); // ['1']: 숫자 키가 문자열이 됨

const map = new Map([[1, 'one']]);
console.log(map.has(1));   // true
console.log(map.has('1')); // false: 숫자 1과 문자열 '1'은 다른 키
```

#### 키의 동일성

객체를 키로 쓸 때도 Set과 같은 규칙이 적용된다. **같은 객체로 `set`하면 값을 덮고, 새 객체로 `set`하면 새 항목이 생긴다.**

```js
const key = { id: 1 };
const map = new Map();

map.set(key, 'A');
map.set(key, 'B');        // 같은 객체이므로 'A'를 덮음
map.set({ id: 1 }, 'C');  // 모양만 같은 새 객체이므로 별도 항목

console.log(map.size);           // 2
console.log(map.get(key));       // 'B'
console.log(map.get({ id: 1 })); // undefined: 다시 새 객체를 만듦
```

객체의 프로퍼티를 바꿔도 그 객체 자체는 그대로다. 따라서 원래 객체를 변수로 갖고 있으면 키를 계속 찾을 수 있다.

```js
const user = { id: 1 };
const map = new Map([[user, '방문함']]);

user.id = 2;

console.log(map.get(user));      // '방문함': 여전히 같은 객체
console.log(map.get({ id: 2 })); // undefined: 모양만 같은 새 객체
```

객체를 키로 사용하려면 **나중에도 같은 객체를 참조할 수 있는지** 생각해야 한다. 내용만 가지고 다시 조회해야 한다면 `user.id` 같은 안정적인 원시 값을 키로 쓰는 편이 간단하다.

#### 주요 메서드와 반환값

| 사용법 | 역할 | 반환값 |
| --- | --- | --- |
| `map.size` | 항목 개수 확인 | 숫자 |
| `map.set(key, value)` | 추가 또는 기존 키의 값 변경 | Map 자신 |
| `map.get(key)` | 키에 대응하는 값 조회 | 저장된 값, 없으면 `undefined` |
| `map.has(key)` | 키의 존재 확인 | 불리언 |
| `map.delete(key)` | 키와 값 삭제 | 삭제 성공 여부를 나타내는 불리언 |
| `map.clear()` | 모든 항목 삭제 | `undefined` |

`get` 결과가 `undefined`여도 키가 없는 것은 아닐 수 있다. 값으로 `undefined`를 저장했을 수도 있으므로 **존재 여부는 `has`로 확인한다.**

```js
const map = new Map([['a', undefined]]);

console.log(map.get('a')); // undefined: 키는 있음
console.log(map.get('b')); // undefined: 키가 없음
console.log(map.has('a')); // true
console.log(map.has('b')); // false
```

#### 순회

Map을 순회하면 삽입 순서대로 `[키, 값]` 쌍을 얻는다. 기존 키의 값을 바꿔도 그 키의 순서는 바뀌지 않는다.

```js
const map = new Map([['a', 1], ['b', 2]]);
map.set('a', 3);

console.log([...map]); // [['a', 3], ['b', 2]]

for (const [key, value] of map) {
  console.log(key, value); // 'a' 3, 'b' 2
}
```

키나 값만 필요하면 `map.keys()` 또는 `map.values()`를 사용한다. `map.entries()`는 `[키, 값]` 쌍을 반환하며 Map 자체를 순회할 때와 같은 형태다.

### Set과 Map 선택 기준

| 필요한 정보 | 선택 | 예시 |
| --- | --- | --- |
| 값이 있는지 여부 | Set | 처리한 사용자 ID 목록 |
| 키에 대응하는 값 | Map | 사용자 ID별 마지막 처리 시간 |

둘 다 객체를 저장하거나 키로 쓸 수 있다.  
이때 가장 중요한 질문은 **내용이 같은 새 객체인가, 이전과 동일한 객체인가?**이다.

## 38장. 브라우저의 렌더링 과정

### 왜 렌더링 과정을 알아야 하는가?

브라우저 렌더링 과정은 HTML·CSS·JavaScript를 받아 **화면의 픽셀**로 만드는 과정이다.  
이 흐름을 알면 어떤 변경이 레이아웃까지 다시 계산하게 하는지, 어떤 변경이 색상만 다시 그리게 하는지 구분할 수 있다.

![HTML·CSS·JavaScript에서 DOM·CSSOM, 렌더 트리, 레이아웃, 페인트로 이어지는 중요 렌더링 경로](assets/38-browser-rendering/01-critical-rendering-path.svg)

```text
서버 응답
  ↓
HTML 파싱 → DOM ──────────┐
CSS 파싱  → CSSOM ─────────┼→ 렌더 트리 → 레이아웃 → 페인트 → 합성 → 화면
JavaScript 실행 → DOM/CSS 변경 ┘
```

위 그림은 개념을 설명하기 위한 순서다. 실제 브라우저는 HTML을 받는 동안 외부 자원을 요청하고 파싱·다운로드를 일부 병렬로 진행한다.

### 1. HTML을 받아 DOM을 만든다

브라우저는 서버에 문서를 요청하고 HTML 응답을 받는다. HTML의 바이트를 문자로 해석한 뒤 토큰으로 나누고, 태그의 부모·자식 관계에 맞춰 DOM 노드를 만든다. DOM은 HTML 문자열 자체가 아니라 **문서를 다루기 위한 객체 트리**다.

```text
응답 바이트 → 문자 → HTML 토큰 → DOM 노드 → DOM 트리
```

|![문서 노드와 html·head·title 요소 및 텍스트 노드로 구성된 DOM 트리 예시](assets/38-browser-rendering/02-dom-tree.png)|
|:---:|
|        문서 노드와 html·head·title 요소 및 텍스트 노드로 구성된 DOM 트리 예시                                                                                         |

HTML 파싱은 문서 전체를 받은 뒤 한 번에 시작하는 것이 아니라, 도착한 내용을 따라 점진적으로 진행된다. `<link>`, `<script>`, `<img>`를 만나면 브라우저는 필요한 외부 자원도 요청한다.

### 2. CSS와 JavaScript를 불러온다

외부 자원은 모두 같은 방식으로 HTML 파싱을 멈추지 않는다. **HTML 파싱을 막는지**와 **화면 표시를 막는지**를 구분해야 한다.

| 자원 | HTML 파싱 | 화면 표시와의 관계 |
| --- | --- | --- |
| 일반적인 스타일시트 `<link rel="stylesheet">` | 다운로드 중에도 대체로 계속 진행 | CSSOM이 준비될 때까지 해당 스타일이 필요한 렌더링을 지연시킴 |
| 일반 `<script src="...">` | 다운로드와 실행이 끝날 때까지 중단 | 스크립트가 DOM·스타일을 바꿀 수 있으므로 이후 렌더링에도 영향 |
| `<img>` | 요청 후 계속 진행 | 이미지가 도착한 뒤 표시되며 크기 정보가 없으면 레이아웃이 바뀔 수 있음 |

CSS 파일을 발견했다고 해서 DOM 생성이 무조건 중단되는 것은 아니다. CSS는 주로 **렌더링을 차단**한다. 다만 앞서 발견한 스타일시트가 아직 로드되지 않았다면, 뒤에 오는 일반 스크립트의 실행이 그 CSS를 기다릴 수 있고, 그동안 HTML 파싱도 멈춘다.

스크립트에 `async` 또는 `defer`를 붙이면 다운로드 중에는 HTML 파싱을 계속할 수 있다. 실행 시점은 서로 다르다.

```html
<script src="normal.js"></script>
<script async src="independent.js"></script>
<script defer src="app.js"></script>
```

| 방식 | 실행 시점 | 순서·DOM에 관한 주의점 |
| --- | --- | --- |
| 일반 스크립트 | 파서가 만난 위치에서 다운로드·실행 | 실행이 끝날 때까지 HTML 파싱 중단 |
| `async` | 다운로드가 끝나는 대로 실행 | DOM 완성 전일 수도 있고, 다른 `async` 스크립트와 실행 순서를 보장하지 않음 |
| `defer` | HTML 파싱이 끝난 뒤 실행 | 문서에 적힌 순서대로 실행하며 `DOMContentLoaded` 전에 완료 |

따라서 `async`와 `defer`를 모두 “DOM이 완성된 뒤 실행”이라고 묶으면 안 된다.  
DOM이 준비돼야 실행할 수 있는 코드에는 보통 `defer`가 더 잘 맞는다.

### 3. CSSOM과 렌더 트리를 만든다

브라우저는 CSS를 해석해 선택자와 선언, 상속·계단식 규칙을 계산하는 데 쓰이는 CSSOM을 만든다.  
DOM은 **무엇이 있는지**, CSSOM은 **어떻게 보여야 하는지**에 관한 정보다.

이 둘을 바탕으로 화면에 표시할 노드와 계산된 스타일을 담은 렌더 트리를 만든다.   
`<head>`나 `display: none` 요소처럼 화면에 나타나지 않는 노드는 렌더 트리에 포함되지 않는다.   
반면 `visibility: hidden` 요소는 보이지 않아도 공간을 차지하므로 레이아웃 대상이 된다.

|![DOM과 CSSOM을 결합해 화면에 표시할 노드의 렌더 트리를 만드는 과정](assets/38-browser-rendering/03-render-tree.png)|
|:---:|

*원본 글에 실은 교재 『모던 자바스크립트 Deep Dive』의 렌더 트리 그림 38-8.*

### 4. 레이아웃으로 크기와 위치를 계산한다

렌더 트리가 만들어지면 브라우저는 각 요소의 크기와 위치를 계산한다. 이 단계가 **레이아웃**이다.   
DOM 노드를 추가·삭제하거나 요소의 `width`, `height`처럼 공간 배치에 영향을 주는 값을 바꾸면 영향을 받는 영역의 레이아웃을 다시 계산할 수 있다.   
다시 계산하는 일을 흔히 **리플로우(Reflow)** 라고 부른다.

```css
.card {
  width: 300px;
}

/* width를 바꾸면 배치가 달라질 수 있어 레이아웃 재계산이 필요하다. */
.card.expanded {
  width: 500px;
}
```

레이아웃 계산 뒤에는 영향을 받는 부분의 페인트와 합성이 이어질 수 있다. 매번 문서 전체를 다시 계산한다는 뜻은 아니며, 실제 작업 범위는 변경 내용과 브라우저의 최적화에 따라 달라진다.

### 5. 페인트하고 합성한다

**페인트**는 계산된 위치에 색상, 글자, 배경, 그림자 같은 시각 요소를 그리는 단계다. 위치와 크기를 바꾸지 않고 `color`나 `background-color`를 바꾼 경우에는 레이아웃 없이 필요한 부분만 다시 그릴 수 있다.  
이를 흔히 **리페인트(Repaint)**라고 부른다.

```css
.card.active {
  background-color: lightblue; /* 보통 레이아웃 없이 페인트 갱신 */
}
```

그린 내용을 여러 레이어에서 최종 화면으로 합치는 단계는 **합성(Composite)** 이다.  
`transform`이나 `opacity` 같은 속성은 상황에 따라 레이아웃과 페인트를 건너뛰고 합성 단계에서 갱신될 수 있다.   
어떤 단계가 실제로 실행됐는지는 측정으로 확인해야 한다.

| 변경 예시 | 일반적으로 필요한 작업 |
| --- | --- |
| 요소 추가, 크기·위치 변경 | 스타일 계산 → 레이아웃 → 페인트 → 합성 |
| 배경색·글자색 변경 | 스타일 계산 → 페인트 → 합성 |
| 합성 레이어의 `transform`·`opacity` 변경 | 합성 중심의 갱신이 가능 |

레이아웃이나 페인트가 자주 오래 걸리면 화면 갱신이 늦어져 버벅임(jank)이 보일 수 있다.  
목표 시간은 디스플레이의 주사율과 실행 환경에 따라 달라지므로, “항상 60fps”로 가정하기보다 실제 기기에서 측정한다.

### 개발자 도구에서 확인하기

원본 글에서는 간단한 페이지를 실행한 뒤 Chrome DevTools의 **Performance → Event Log**에서 요청, HTML 파싱, 스타일 계산, 레이아웃, 페인트, 스크립트 실행을 확인했다.

|![원본 글의 Chrome DevTools Performance Event Log: HTML·CSS·스크립트 요청과 파싱, 레이아웃, 페인트, 스크립트 평가가 표시된 화면](assets/38-browser-rendering/04-performance-event-log.png)|
|:---:|

이 기록에서는 스크립트 요청이 일찍 나타났지만 스크립트 평가는 첫 페인트 뒤에 보인다.

**요청 시점과 실행 시점은 다르다.**   
그렇다고 모든 스크립트가 첫 페인트 뒤에 실행된다는 규칙으로 일반화할 수는 없다.   
`<script>`의 위치와 `async`·`defer` 여부, 네트워크·메인 스레드 작업에 따라 순서가 달라진다.

|![본문 끝에 script를 둔 경우 스크립트 실행 시점을 설명한 원본 글의 GPT 답변 캡처](assets/38-browser-rendering/05-original-gpt-answer.png)|
|:---:|