> ⭐ 표시: 스터디에서 중점으로 짚고 넘어갈 부분

## 46장 제너레이터와 async/await

### 제너레이터란 ⭐

- ES6 도입, 코드 블록의 실행을 **일시 중지했다가 필요한 시점에 재개**할 수 있는 특수한 함수
- 일반 함수와의 차이
  1. **함수 호출자에게 실행 제어권을 양도**할 수 있음
     - 일반 함수는 호출되면 끝날 때까지 함수가 제어권을 가짐
     - 제너레이터는 호출자가 함수 실행을 제어할 수 있음
  2. **호출자와 상태를 양방향으로 주고받을 수 있음**
     - 일반 함수는 매개변수로 받고 반환값으로 돌려주는 게 전부
     - 제너레이터는 실행 중간에 호출자에게 값을 주고, 호출자에게서 값을 받을 수도 있음
  3. 호출하면 함수 코드를 실행하는 게 아니라 **제너레이터 객체를 반환**

### 제너레이터 함수의 정의

```jsx
function* genFunc() { yield 1; }          // 함수 선언문
const genExpFunc = function* () { yield 1; }; // 함수 표현식
const obj = { *genObjMethod() { yield 1; } }; // 메서드
class MyClass { *genClsMethod() { yield 1; } } // 클래스 메서드
```

- `function*` 키워드로 선언, 하나 이상의 `yield` 표현식 포함
- `*`는 function과 함수 이름 사이 어디든 가능하지만 **`function*`처럼 붙이는 걸 권장**
- **화살표 함수로 정의 불가**
- **`new`로 생성자 호출 불가**

### 제너레이터 객체

- 제너레이터 함수를 호출하면 코드 블록이 실행되지 않고 **제너레이터 객체를 생성해서 반환**
- 제너레이터 객체는 **이터러블이면서 동시에 이터레이터**
  - `Symbol.iterator` 메서드를 상속받는 이터러블
  - `next` 메서드를 가진 이터레이터
- 이터레이터에 없는 `return`, `throw` 메서드도 가짐

| 메서드 | 동작 | 반환하는 이터레이터 리절트 객체 |
| --- | --- | --- |
| `next()` | 다음 `yield`까지 실행 | `{ value: yield된 값, done: false }` |
| `return(x)` | 제너레이터 종료 | `{ value: x, done: true }` |
| `throw(err)` | 인수로 받은 에러 발생 (catch에서 받음) | `{ value: undefined, done: true }` |

### 제너레이터의 일시 중지와 재개 ⭐

```jsx
function* genFunc() {
  yield 1;
  yield 2;
  yield 3;
}

const generator = genFunc();

generator.next(); // { value: 1, done: false }
generator.next(); // { value: 2, done: false }
generator.next(); // { value: 3, done: false }
generator.next(); // { value: undefined, done: true }
```

- `next()`를 호출하면 **`yield` 표현식까지 실행하고 일시 중지** → 제어권이 호출자로 넘어감
- 다시 `next()`를 호출하면 멈췄던 곳부터 다음 `yield`까지 실행
- `yield` 키워드
  - 제너레이터 함수의 실행을 일시 중지시키고
  - `yield` 뒤 표현식의 평가 결과를 호출자에게 반환
- 끝까지 실행되면 `value`는 함수의 **반환값**, `done`은 `true`

#### next에 인수 전달

```jsx
function* genFunc() {
  const x = yield 1;     // 두 번째 next의 인수가 x에 할당
  const y = yield x + 10; // 세 번째 next의 인수가 y에 할당
  return x + y;
}

const generator = genFunc();

generator.next();   // { value: 1, done: false }
generator.next(10); // { value: 20, done: false }  → x = 10
generator.next(20); // { value: 30, done: true }   → y = 20
```

- `next`에 넘긴 인수는 **`yield` 표현식을 할당받는 변수**에 할당됨
- 주의: `yield` 표현식이 먼저 값을 내보내고 멈추기 때문에, 변수 할당은 **다음 `next` 호출 때** 일어남
- **첫 번째 `next`의 인수는 무시됨** (받아줄 yield가 아직 없음)
- 이 양방향 소통 덕분에 **비동기 처리를 동기 처리처럼** 구현할 수 있음 → async/await의 기반

### 제너레이터의 활용

#### 이터러블 구현

- 이터레이션 프로토콜을 직접 구현하는 것보다 훨씬 간단

```jsx
const infiniteFibonacci = (function* () {
  let [pre, cur] = [0, 1];
  while (true) {
    [pre, cur] = [cur, pre + cur];
    yield cur;
  }
}());

for (const num of infiniteFibonacci) {
  if (num > 10000) break;
  console.log(num); // 1 2 3 5 8 ... 6765
}
```

- 무한 반복이어도 `next`를 호출할 때만 값을 만드니까 문제없음 → **지연 평가**

#### 비동기 처리

- 제너레이터로 프로미스를 `yield`하고, 결과가 오면 `next`로 다시 넘겨주는 **제너레이터 실행기**를 만들면 비동기를 동기처럼 작성 가능
- 직접 구현하기보다 `co` 라이브러리를 썼었음
- 지금은 **async/await**가 이 역할을 대신함 → 제너레이터 실행기는 원리만 이해하면 됨

### async/await ⭐

- ES8(ES2017) 도입
- 제너레이터보다 간단하고 가독성 좋게 비동기 처리를 **동기 처리처럼** 구현
- **프로미스 기반**으로 동작 → `then/catch/finally` 후속 처리 메서드에 콜백을 넘길 필요 없음

#### async 함수

- `async` 키워드로 정의, **언제나 프로미스를 반환**
  - 명시적으로 프로미스를 반환하지 않아도 반환값을 resolve하는 프로미스를 암묵적으로 반환

```jsx
async function foo(n) { return n; }
foo(1).then(v => console.log(v)); // 1
```

- 함수 선언문, 함수 표현식, 화살표 함수, 메서드 모두 가능
- **클래스의 constructor는 async 불가** → constructor는 인스턴스를 반환해야 하는데 async 함수는 프로미스를 반환하기 때문

#### await 키워드

- **프로미스가 settled 상태가 될 때까지 대기**하다가, settled 되면 프로미스가 resolve한 처리 결과를 반환
- **반드시 프로미스 앞**에 사용
- **async 함수 내부**에서만 사용 가능
  - 참고 (책 범위 밖): ES2022부터 모듈 최상위에서는 async 없이 `await` 사용 가능 (top-level await)

```jsx
const fetchUser = async id => {
  const res = await fetch(`/users/${id}`); // 응답 올 때까지 대기
  const user = await res.json();
  return user;
};
```

- 주의: 서로 **연관 없는 비동기 작업을 하나씩 await하면 불필요하게 느려짐**

```jsx
// bad: 3초 (1초 + 1초 + 1초)
const a = await getA();
const b = await getB();
const c = await getC();

// good: 1초 (동시에 출발)
const [a, b, c] = await Promise.all([getA(), getB(), getC()]);
```

- 앞의 결과가 다음 요청에 필요하면 그때만 순차적으로 await
- 활용: 페이지 진입 시 상품 정보, 리뷰, 추천 목록처럼 **서로 독립적인 API**를 동시에 불러올 때 `Promise.all`

#### 에러 처리

- 콜백 패턴은 비동기 함수의 콜백이 호출자가 명확하지 않아서 `try...catch`로 에러를 못 잡음 (47장 에러 전파와 연결)
- async/await에서는 **`try...catch`로 에러 처리 가능**
  - 프로미스를 반환하는 비동기 함수는 명시적으로 호출하니까 **호출자가 명확**하기 때문

```jsx
const getUser = async id => {
  try {
    const res = await fetch(`/users/${id}`);
    if (!res.ok) throw new Error(res.statusText);
    return await res.json();
  } catch (err) {
    console.error(err);
  }
};
```

- async 함수 안에서 `catch`로 처리하지 않으면, **reject된 프로미스를 반환** → 호출하는 쪽에서 `.catch`로 처리 가능
- 활용: `fetch`는 **404, 500 응답에도 reject하지 않음** (네트워크 장애일 때만 reject) → `res.ok` 확인하고 직접 `throw` 해야 catch로 넘어감

---

## 47장 에러 처리

### 에러 처리의 필요성 ⭐

- 에러가 발생하면 **프로그램이 강제 종료**됨
- `try...catch`로 에러에 적절히 대응하면 강제 종료되지 않고 **계속 실행** 가능

```jsx
console.log('[Start]');

try {
  foo(); // ReferenceError
} catch (error) {
  console.error('[에러 발생]', error);
}

console.log('[End]'); // 실행됨
```

- 에러가 직접 발생하지 않는 **예외적인 상황**도 대응이 필요
  - ex) `querySelector`는 요소를 못 찾으면 에러가 아니라 `null` 반환 → 이후 `null.classList`에서 TypeError
  - 대응: `if` 문, 단축 평가, **옵셔널 체이닝** `$elem?.classList`
- 에러나 예외 상황에 대응하지 않으면 프로그램이 강제 종료됨 → **언제나 에러나 예외 상황이 발생할 수 있다는 걸 전제**로 코드 작성

### try...catch...finally 문

```jsx
try {
  // 실행할 코드 (에러가 발생할 가능성이 있는)
} catch (err) {
  // try에서 에러 발생 시 실행, err에 에러 객체 전달
} finally {
  // 에러 발생 여부와 상관없이 반드시 한 번 실행
}
```

- `try` 코드 블록 먼저 실행 → 에러가 나면 `catch`의 `err` 변수에 에러 객체가 전달되고 `catch` 블록 실행
- `catch`의 `err`는 `catch` 블록 안에서만 유효
- `finally`는 생략 가능, catch도 생략 가능하지만 의미 없음
- 활용: 로딩 상태 해제처럼 **성공/실패와 관계없이 꼭 해야 하는 처리**는 `finally`에

```jsx
setLoading(true);
try {
  await submitOrder();
} catch (err) {
  showToast('주문에 실패했습니다');
} finally {
  setLoading(false); // 성공이든 실패든 로딩 해제
}
```

### Error 객체

- `new Error(message)`: 에러 객체 생성, 인수로 에러 상세 설명 메시지 전달
- 주요 프로퍼티
  - `message`: 생성자에 넘긴 에러 메시지
  - `stack`: 에러를 발생시킨 **콜 스택의 호출 정보**를 나타내는 문자열 → 디버깅용
- 7가지 에러 생성자 함수 (모두 `Error.prototype`을 상속)

| 생성자 | 인스턴스 |
| --- | --- |
| `Error` | 일반적 에러 |
| `SyntaxError` | 문법에 맞지 않는 문을 해석할 때 |
| `ReferenceError` | 참조할 수 없는 식별자를 참조할 때 |
| `TypeError` | 피연산자나 인수의 데이터 타입이 유효하지 않을 때 |
| `RangeError` | 숫자값의 허용 범위를 벗어났을 때 |
| `URIError` | `encodeURI`, `decodeURI`에 부적절한 인수를 전달할 때 |
| `EvalError` | `eval` 함수에서 에러가 발생할 때 |

### throw 문 ⭐

- **에러 객체를 생성하는 것과 에러를 발생시키는 건 다른 의미**

```jsx
try {
  new Error('something wrong'); // 생성만 함 → catch로 안 감
} catch (err) {
  console.log(err);
}
```

- 에러를 발생시키려면 **`throw` 문으로 에러 객체를 던져야** 함
- `throw` 하면 `catch` 블록이 실행되고, `catch`의 `err`에 던진 에러 객체가 전달됨

```jsx
const repeat = (n, f) => {
  if (typeof f !== 'function') throw new TypeError('f must be a function');
  for (let i = 0; i < n; i++) f(i);
};

try {
  repeat(2, 1); // TypeError
} catch (err) {
  console.error(err);
}
```

- `throw` 뒤에는 아무 값이나 올 수 있지만 **일반적으로 에러 객체**를 던짐 (stack 정보가 있어서)

### 에러의 전파 ⭐

- 에러는 **호출자(caller) 방향으로 전파**됨
  - = 콜 스택의 아래 방향 (실행 중인 실행 컨텍스트가 푸시되기 직전에 푸시된 실행 컨텍스트 방향)

```jsx
const foo = () => { throw Error('foo에서 발생한 에러'); };
const bar = () => { foo(); };
const baz = () => { bar(); };

try {
  baz();
} catch (err) {
  console.error(err); // foo → bar → baz → 전역으로 전파되어 여기서 캐치
}
```

- 에러를 캐치해서 적절히 대응하면 프로그램을 강제 종료하지 않고 코드 실행 흐름을 복구 가능
- 어디서도 캐치하지 않으면 **강제 종료**
- 주의: **비동기 함수의 콜백은 호출자가 없음**
  - `setTimeout`이나 프로미스 후속 처리 메서드의 콜백은 태스크 큐에 있다가 **콜 스택이 비었을 때** 이벤트 루프에 의해 푸시됨
  - 그래서 콜 스택의 가장 아래에 존재 → **에러를 전파할 호출자가 없음**

```jsx
try {
  setTimeout(() => { throw new Error('Error!'); });
} catch (err) {
  console.error(err); // 캐치 못 함
}
```

- 그래서 비동기 에러는 콜백 안에서 직접 처리하거나, 프로미스 `.catch` / async-await `try...catch`로 처리

---

## 48장 모듈

### 모듈의 일반적 의미

- 애플리케이션을 구성하는 개별적 요소로서 **재사용 가능한 코드 조각**
- 일반적으로 **기능을 기준으로 파일 단위로 분리**
- 모듈이 성립하려면 **자신만의 파일 스코프(모듈 스코프)** 를 가져야 함
  - 모듈의 자산(변수, 함수, 객체 등)은 기본적으로 **캡슐화**되어 다른 모듈에서 접근 불가
- 그래서 필요한 자산만 공개(**export**)하고, 사용하는 쪽은 필요한 것만 선택적으로 가져와서(**import**) 사용
- 장점
  - 코드의 단위를 명확히 분리
  - **재사용성** 향상
  - 개발 효율성과 **유지보수성** 향상

### 자바스크립트와 모듈

- 자바스크립트는 원래 웹페이지의 단순한 보조 기능 용도라 **모듈 시스템이 없었음**
- `script` 태그로 파일을 나눠 로드해도 모듈로 동작하지 않음
  - 분리된 파일들이 **하나의 전역을 공유** → 전역 변수 중복 등 문제
- 그래서 등장한 비공식 모듈 시스템
  - **CommonJS**: Node.js가 채택, 사실상 표준 (`require`, `module.exports`)
  - **AMD**: 브라우저용 비동기 모듈 정의
- Node.js는 모듈 시스템 사양을 따르기 때문에 파일별로 독립적인 모듈 스코프를 가짐

### ES6 모듈 (ESM) ⭐

- ES6에서 **클라이언트 사이드 자바스크립트에서도 동작하는 모듈 기능** 추가
- `script` 태그에 **`type="module"`** 을 추가하면 로드된 파일은 모듈로 동작

```html
<script type="module" src="app.mjs"></script>
```

- 일반 JS 파일과 구분하기 위해 확장자는 **`.mjs`** 권장
- ESM에는 **기본적으로 strict mode 적용**
- 참고 (책 범위 밖): `type="module"` 스크립트는 기본적으로 **defer처럼** 동작 (HTML 파싱 후 실행)

#### 모듈 스코프

- 일반 스크립트
  - 파일 최상위에서 `var`로 선언한 변수는 **전역 변수 → 전역 객체(window)의 프로퍼티**
  - 다른 script 파일과 전역을 공유해서 변수가 덮어씌워질 수 있음
- ESM
  - **파일 자체의 독자적인 모듈 스코프**를 가짐
  - 모듈 최상위에서 `var`로 선언해도 **전역 변수가 아니고 window의 프로퍼티도 아님**
  - 모듈 내에서 선언한 식별자는 **모듈 외부에서 참조 불가**

```jsx
// foo.mjs
var x = 'foo';
console.log(window.x); // undefined

// bar.mjs
console.log(x); // ReferenceError
```

#### export 키워드

- 모듈 내부의 식별자를 외부에 공개해서 다른 모듈이 재사용할 수 있게 함
- 선언문 앞에 `export`

```jsx
// lib.mjs
export const pi = Math.PI;
export function square(x) { return x * x; }
export class Person { constructor(name) { this.name = name; } }
```

- 하나씩 붙이기 번거로우면 **객체로 묶어서 한 번에**

```jsx
const pi = Math.PI;
function square(x) { return x * x; }
class Person { /* ... */ }

export { pi, square, Person };
```

#### import 키워드

```jsx
// app.mjs
import { pi, square, Person } from './lib.mjs';

console.log(square(10)); // 100
```

- 다른 모듈이 export한 식별자 이름으로 가져옴
- 브라우저 환경에서는 **파일 확장자 생략 불가**
- 진입점인 `app.mjs`만 script 태그로 로드하면 됨 → `app.mjs`가 import하는 `lib.mjs`는 script 태그 불필요
- 하나의 이름으로 한 번에 가져오기: `import * as lib from './lib.mjs';` → `lib.pi`, `lib.square`
- 이름 바꿔서 가져오기: `import { pi as PI, square as sq } from './lib.mjs';`
- export할 때도 `as`로 이름 변경 가능: `export { pi as PI };`

#### default export

- 모듈에서 **하나의 값만 export**할 때 `default` 키워드 사용
- 이름 없이도 export 가능, **모듈당 하나만**

```jsx
// lib.mjs
export default x => x * x;

// app.mjs
import square from './lib.mjs'; // {} 없이, 이름은 임의로
```

- `var`, `let`, `const` 키워드와 같이 사용 불가

```jsx
export default const foo = () => {}; // SyntaxError
```

- 활용: React 컴포넌트 파일은 컴포넌트 하나를 `export default`, 유틸 함수 모음 파일은 이름 있는 export를 쓰는 식으로 많이 나눔
  - 이름 있는 export는 import할 때 이름이 고정돼서 **자동 import, 리팩터링(이름 변경) 추적에 유리**

#### 실무에서는

- 브라우저 ESM을 그대로 쓰는 경우는 많지 않음
- 구형 브라우저 호환, 트랜스파일링, 많은 파일을 하나로 묶는 번들링 등의 이유로 **Webpack 같은 모듈 번들러**(요즘은 Vite 등)를 사용하는 게 일반적