# 모던 자바스크립트 Deep Dive 46~47장 정리

## 46장. 제너레이터와 async/await

이번 장은 문법보다 **코루틴, 제너레이터, async/await이 어떻게 이어지는지**를 이해하는 것이 핵심이었다.

```text
코루틴(개념)
  → 제너레이터 (자바스크립트가 코루틴을 구현한 것)
    → async/await (제너레이터 + 프로미스 + 자동 실행기를 문법으로 만든 것)
```

### 먼저 알아 둘 개념: 코루틴

**코루틴(coroutine)**은 실행 도중 스스로 멈추고, 그때의 상태를 기억했다가, 멈춘 지점부터 이어서 실행할 수 있는 함수다.

| 구분 | 서브루틴 (일반 함수) | 코루틴 |
| --- | --- | --- |
| 시작 지점 | 항상 함수의 처음 | 지난번에 멈춘 곳 |
| 실행 방식 | 한 번 들어가면 끝까지 실행 | 중간에 멈췄다가 이어서 실행 |
| 값 반환 | `return`으로 한 번 | 멈출 때마다 값을 내보낼 수 있음 |
| 지역 변수 | `return`하면 사라짐 | 멈춰 있는 동안에도 보존됨 |

#### 실행 위치 자체가 상태다

초록, 노랑, 빨강을 반복하는 신호등을 두 방식으로 만들어 보자.

```js
// 일반 함수: 어디까지 했는지를 변수로 따로 관리해야 한다
let step = 0;
function nextLight() {
  const colors = ['초록', '노랑', '빨강'];
  return colors[step++ % 3];
}

// 제너레이터: 몇 번째 yield에서 멈췄는지가 곧 상태다
function* trafficLight() {
  while (true) {
    yield '초록';
    yield '노랑';
    yield '빨강';
  }
}

const light = trafficLight();
light.next().value; // 초록
light.next().value; // 노랑
```

일반 함수는 호출이 끝나면 상태를 잊으므로 바깥 변수나 클로저에 저장해야 한다. 코루틴은 **멈춘 위치가 곧 상태**라서 단계가 여러 개인 흐름을 위에서 아래로 읽히는 순차 코드로 쓸 수 있다. async/await이 비동기 코드를 동기 코드처럼 보이게 하는 원리도 이것이다.

#### 자바스크립트 코루틴의 제약

자바스크립트의 제너레이터와 async 함수는 **스택리스(stackless)** 코루틴이다. 코루틴 본문에서만 멈출 수 있고, 안쪽에서 호출한 함수에서는 멈출 수 없다.

- `yield`는 `function*` 안에서만 쓸 수 있다.
- `await`은 `async` 함수 안에서만 쓸 수 있다.
- `forEach` 콜백 안의 `await`은 콜백만 멈추고 바깥 함수를 멈추지 못한다. (아래 [반복문 안의 await](#반복문-안의-await) 참고)

### 제너레이터란?

제너레이터는 코드 블록의 실행을 일시 중지했다가 필요한 시점에 재개할 수 있는 함수다. `function*`로 정의하고, 화살표 함수로는 정의할 수 없으며 `new`로 호출할 수 없다.

```js
function* genFunc() {
  console.log('본문 실행');
  yield 1;
}

const generator = genFunc(); // 아무것도 출력되지 않는다
```

제너레이터 함수를 호출하면 본문을 실행하지 않고 **제너레이터 객체**를 반환한다. 제너레이터 객체는 이터러블이면서 동시에 이터레이터다.

| 구분 | 일반 함수 | 제너레이터 함수 |
| --- | --- | --- |
| 호출하면 | 본문을 끝까지 실행 | 본문을 실행하지 않고 제너레이터 객체 반환 |
| 실행 제어권 | 끝날 때까지 함수가 가짐 | `yield`로 호출자에게 넘기고 `next`로 돌려받음 |
| 값 주고받기 | 인수로 받고 `return`으로 한 번 반환 | `yield`로 여러 번 내보내고 `next(값)`으로 받음 |

### 제너레이터의 일시 중지와 재개

```js
function* genFunc() {
  const x = yield 1;
  const y = yield (x + 10);
  return x + y;
}

const generator = genFunc(0);

let res = generator.next();
console.log(res); // { value: 1, done: false }

res = generator.next(10);
console.log(res); // { value: 20, done: false }

res = generator.next(20);
console.log(res); // { value: 30, done: true }
```

`genFunc(0)`의 0은 받을 매개변수가 없으므로 어디에도 쓰이지 않는다.

#### 한 줄이 두 번의 next에 나뉘어 실행된다

이 예제가 헷갈리는 이유는 `const x = yield 1;` 한 줄이 **두 번의 `next` 호출에 걸쳐** 실행되기 때문이다.

1. `yield` 오른쪽의 `1`을 평가한다.
2. 그 값을 내보내고 **멈춘다.** ← 멈추는 곳
3. 다음 `next(10)`이 오면 `yield 1` 표현식 전체가 `10`으로 바뀐다.
4. `x`에 대입한다.

"yield 앞에서 멈춘다"고 기억하면 오른쪽도 아직 평가되지 않은 것처럼 착각하기 쉽다. **값을 내보낸 직후, 결과를 받기 직전**에 멈춘다.

#### 첫 next의 인수는 항상 무시된다

첫 `next`의 인수는 코드 모양과 상관없이 **항상** 버려진다. 값을 받는 것은 멈춰 있는 `yield`인데, 첫 `next` 시점에는 아직 멈춘 적이 없기 때문이다. 첫 구간은 이전 `yield`가 아니라 함수 시작에서 출발한다.

- 인수 표현식 자체는 평가된다. `it.next(side())`라고 쓰면 `side()`는 호출되고 결과만 버려진다.
- 제너레이터 안의 `arguments`로도 꺼낼 수 없다. `arguments`는 `genFunc(...)` 호출의 인수다.
- 시작값이 필요하면 제너레이터 함수의 매개변수로 전달한다.

```js
function* counter(init) {
  const v = yield init;
}

const it = counter(100);
it.next();  // { value: 100, done: false }
it.next(5); // v = 5
```

#### yield name은 name을 기다리지 않는다

처음에는 아래 코드에서 첫 `next('이름')`의 값이 `name`에 들어간다고 생각했다.

```js
function* a() {
  let name;
  const v = yield name;
  console.log('v =', v, '/ name =', name);
}

const it = a();
it.next('이름'); // { value: undefined, done: false }
it.next('값');   // v = 값 / name = undefined
```

`yield name`은 `name`의 현재 값(`undefined`)을 **내보내기만** 한다. 들어오는 값은 항상 `yield` 표현식 쪽, 즉 `v`로 간다. `name`에 값을 받으려면 `yield`를 대입문의 오른쪽에 둬야 한다.

```js
function* a() {
  const name = yield; // 아무것도 내보내지 않고 멈춤
  console.log(name);
}

const it = a();
it.next();       // yield에서 멈추게 하는 준비 호출
it.next('이름'); // 이름
```

### next, return, throw

세 메서드는 모두 **멈춰 있는 `yield` 자리에서 제너레이터를 재개**한다. 차이는 그 자리가 무엇으로 바뀌느냐다.

| 메서드 | 멈춘 `yield` 자리가 이렇게 바뀜 |
| --- | --- |
| `next(v)` | 값 `v` |
| `throw(e)` | `throw e` (그 자리에서 에러 발생) |
| `return(v)` | `return v` (그 자리에서 함수가 반환한 것처럼) |

#### throw

```js
function* g1() {
  try {
    const v = yield 1;
    console.log('v =', v); // 실행되지 않음
  } catch (e) {
    console.log('catch:', e);
  }
  yield 2;
}

const it1 = g1();
it1.next();         // { value: 1, done: false }
it1.throw('에러!'); // catch: 에러! → { value: 2, done: false }
```

제너레이터 안에서 에러를 잡으면 실행이 이어지므로, `throw()`의 반환값은 **다음 `yield`의 결과**다.

```js
function* g2() {
  yield 1;
  yield 2;
}

const it2 = g2();
it2.next();
try {
  it2.throw('에러!');
} catch (e) {
  console.log('바깥에서 catch:', e); // 바깥에서 catch: 에러!
}
it2.next(); // { value: undefined, done: true }
```

제너레이터 안에 `catch`가 없으면 일반 함수처럼 **호출자 방향으로 전파**된다. 제너레이터는 `completed`가 된다.

```text
[g2 실행 컨텍스트]   ← yield 1 자리에서 에러 발생, catch 없음 → g2 종료
[throw 내장 함수]    ← 그대로 통과
[전역 실행 컨텍스트] ← 여기까지 전파, try...catch가 없으면 Uncaught 에러
```

#### return

```js
function* g3() {
  try {
    yield 1;
    yield 2;
  } finally {
    console.log('정리 작업');
  }
}

const it3 = g3();
it3.next();         // { value: 1, done: false }
it3.return('끝냄'); // 정리 작업 → { value: '끝냄', done: true }
it3.next();         // { value: undefined, done: true }
```

`yield 1` 자리에서 `return`한 것과 같으므로 일반 함수처럼 `finally`가 실행된다.

`for...of`를 `break`로 중간에 끝내면 엔진이 `return()`을 자동으로 호출한다. 그래서 제너레이터의 `finally`에서 정리 작업을 할 수 있다.

```js
for (const n of g3()) {
  if (n === 1) break; // return() 자동 호출 → 정리 작업
}
```

### 실행 컨텍스트로 본 제너레이터

일반 함수는 `return`하면 실행 컨텍스트가 콜 스택에서 제거되고 버려진다. 제너레이터는 멈출 때 실행 컨텍스트가 콜 스택에서 제거되지만 **버려지지 않고 제너레이터 객체에 보관**된다.

명세에서 제너레이터 객체는 다음 내부 슬롯을 가진다.

| 내부 슬롯 | 값 |
| --- | --- |
| `[[GeneratorState]]` | `suspended-start`, `suspended-yield`, `executing`, `completed` 중 하나 |
| `[[GeneratorContext]]` | 멈춘 실행 컨텍스트 |

```text
suspended-start ──next()──▶ executing ──return/끝──▶ completed
                              ▲    │
                        next()│    │yield
                              │    ▼
                         suspended-yield
```

예제 46-09에서 첫 `yield 1`을 만났을 때의 흐름은 다음과 같다.

```text
next() 호출 중                         yield 1 이후

[genFunc 실행 컨텍스트] ← 실행 중       (스택에서 제거)
[next 실행 컨텍스트]                    [next 실행 컨텍스트] ← 결과를 받아 반환
[전역 실행 컨텍스트]                    [전역 실행 컨텍스트]

                                        generator.[[GeneratorContext]]
                                          → genFunc 실행 컨텍스트 (2행에서 멈춘 상태로 보관)
                                        generator.[[GeneratorState]] = suspended-yield
```

다음 `next`가 호출되면 상태를 `executing`으로 바꾸고, `[[GeneratorContext]]`의 컨텍스트를 다시 콜 스택에 올려 멈춘 위치부터 실행한다.

V8에서 실제로 무엇을 보관하는지 확인해 봤다.

```js
// node --allow-natives-syntax debug.js
function* gen() { let count = 100; yield 1; count++; yield count; }
const it = gen();
it.next();
%DebugPrint(it);
```

```text
[JSGeneratorObject]
 - function: <JSFunction gen>     ← 어떤 함수인지
 - context: ...                   ← 스코프 체인
 - receiver: <JSGlobalProxy>      ← this
 - continuation: 1 (suspended)    ← 몇 번째 yield에서 멈췄는지
 - register file: <FixedArray[4]> ← 지역 변수(count 등) 값
```

24장의 클로저와 비교하면 차이가 분명하다.

| 구분 | 살아남는 것 |
| --- | --- |
| 클로저 | 렉시컬 환경(변수) |
| 제너레이터, async 함수 | 렉시컬 환경 + 실행 위치 + 지역 변수 값 |

클로저는 **변수**만 살려 두고, 코루틴은 **함수가 실행되던 그 순간 전체**를 살려 둔다.

async 함수는 실행 컨텍스트를 담은 객체가 코드에 드러나지 않는다. 명세의 `Await` 동작은 현재 실행 컨텍스트를 캡처한 재개용 함수를 `await` 대상 프로미스에 `then`처럼 등록한다. 따라서 **프로미스 → 재개용 콜백 → 실행 컨텍스트**로 이어지는 참조가 멈춘 함수를 붙잡고 있다.

### 제너레이터의 활용

#### 이터러블 구현

제너레이터를 사용하면 이터레이션 프로토콜을 직접 구현하지 않아도 이터러블을 만들 수 있다. 값을 필요할 때 하나씩 만들므로 무한 수열도 표현할 수 있다.

```js
const infiniteFibonacci = (function* () {
  let [pre, cur] = [0, 1];

  while (true) {
    [pre, cur] = [cur, pre + cur];
    yield cur;
  }
}());

for (const num of infiniteFibonacci) {
  if (num > 10000) break;
  console.log(num); // 1 2 3 5 8 … 4181 6765
}
```

#### 비동기 처리

`yield`로 프로미스를 내보내고, 바깥에서 프로미스가 settled(성공 또는 실패로 처리 완료)되면 결과를 넣어 재개하면 비동기 코드를 동기 코드처럼 쓸 수 있다.

```js
function* task() {
  try {
    const data = yield fetchUser(); // 1. 프로미스를 내보내고 멈춤
    console.log('성공:', data);
  } catch (e) {
    console.log('catch:', e);        // 3. 실패면 여기서 잡힘
  }
}

const it = task();
const { value: promise } = it.next();
promise.then(
  data => it.next(data), // 2. 성공: yield 자리에 data를 넣어 재개
  err => it.throw(err),  // 2. 실패: yield 자리에서 err를 던져 재개
);
```

제너레이터 안의 `try...catch`는 자기 안에서 발생한 에러만 잡을 수 있다. 프로미스는 나중에 제너레이터 바깥에서 실패하므로, 그 에러를 **멈춘 `yield` 자리로 전달하는 통로**가 `it.throw()`다.

이 과정을 반복하는 실행기를 만들면 다음과 같다.

```js
function run(genFn) {
  return new Promise((resolve, reject) => {
    const gen = genFn();

    function step(method, arg) {
      let result;
      try {
        result = gen[method](arg); // gen.next(v) 또는 gen.throw(e)
      } catch (e) {
        return reject(e);          // 제너레이터가 에러를 잡지 않으면 전체 실패
      }

      const { value, done } = result;
      if (done) return resolve(value);

      Promise.resolve(value).then(
        v => step('next', v),
        e => step('throw', e),
      );
    }

    step('next');
  });
}
```

### async/await

`async` 함수는 **항상 프로미스를 반환**한다. `await`은 프로미스가 settled될 때까지 **async 함수만** 멈추고, 결과를 반환한다.

```js
async function task() {
  try {
    const data = await fetchUser();
    console.log('성공:', data);
  } catch (e) {
    console.log('catch:', e);
  }
}
```

위의 제너레이터 버전과 모양이 같다. `promise.then(next, throw)`로 직접 구동하던 부분을 엔진이 맡는다.

#### 제너레이터 실행기와 비교

| 제너레이터 + 실행기 | async/await |
| --- | --- |
| `function*` | `async function` |
| `yield promise` | `await promise` |
| 직접 만든 `run()` | 엔진이 대신 처리 |
| 실패하면 `gen.throw(err)` | `await` 자리에서 에러가 발생해 `try...catch`로 잡힘 |
| 이터레이터를 반환하고 실행기가 프로미스로 감쌈 | 항상 프로미스를 반환 |

`run` 함수의 각 부분을 `async`와 `await`이 나눠서 대신한다.

```js
// 역할만 보이도록 줄인 코드
function run(genFn) {
  return new Promise((resolve, reject) => { // ┐ async가 대신함
    // ...                                  // │ 프로미스 반환,
    catch (e) { return reject(e); }         // │ 못 잡은 에러는 reject,
    if (done) return resolve(value);        // ┘ return 값은 resolve

    Promise.resolve(value).then(            // ┐ await이 대신함
      v => step('next', v),                 // │ 값을 프로미스로 감싸고
      e => step('throw', e),                // │ 결과에 따라 next 또는 throw로 재개
    );                                      // ┘
  });
}
```

#### 반복문 안의 await

```js
const ids = [1, 2, 3];

ids.forEach(async id => {
  await delay(100);
  console.log(id);
});
console.log('forEach 끝'); // 가장 먼저 출력된다

for (const id of ids) {
  await delay(100);
  console.log(id); // 1, 2, 3을 차례로 기다린다
}
console.log('for...of 끝'); // 3 다음에 출력된다
```

`forEach`의 콜백은 각각 별도의 async 함수라서 `await`이 콜백만 멈춘다. `forEach`는 콜백이 반환한 프로미스를 기다리지 않는다. 순서대로 기다려야 하면 `for...of`를 쓴다.

#### 에러 처리

`await`한 프로미스가 실패하면 `await` 자리에서 에러가 발생하므로 `try...catch`로 잡을 수 있다. 잡지 않으면 async 함수가 반환한 프로미스가 `rejected`가 된다.

```js
async function load() {
  await Promise.reject(new Error('실패'));
}

load().catch(e => console.log(e.message)); // 실패
```

## 47장. 에러 처리

### 에러 처리가 필요한 이유

에러가 발생했는데 처리하지 않으면 프로그램이 강제 종료된다. 에러가 발생하지 않더라도 예외적인 상황에 대응해야 할 때가 있다.

```js
// 브라우저 환경
const $button = document.querySelector('button'); // 요소가 없으면 null
$button.classList.add('disabled');                 // TypeError
$button?.classList.add('disabled');                // 에러 없이 넘어감
```

`querySelector`는 요소를 찾지 못해도 에러를 던지지 않고 `null`을 반환한다. 이런 경우는 옵셔널 체이닝이나 조건문으로 대응한다.

### try...catch...finally

```js
try {
  foo(); // ReferenceError
} catch (err) {
  console.log(err.name, '/', err.message); // ReferenceError / foo is not defined
} finally {
  console.log('항상 실행');
}
```

`try`에서 에러가 발생하면 `catch`로 이동하고, `finally`는 에러 발생 여부와 상관없이 실행된다. 에러를 처리하면 프로그램은 강제 종료되지 않고 계속 실행된다.

### Error 객체

`Error` 생성자 함수로 에러 객체를 만들 수 있다. 에러 객체는 `message`와 호출 스택 정보인 `stack` 프로퍼티를 갖는다.

| 생성자 함수 | 발생하는 상황 | 예시 |
| --- | --- | --- |
| `Error` | 일반적인 에러 | 직접 만든 에러 |
| `SyntaxError` | 문법에 맞지 않는 코드 | `JSON.parse('{x')` |
| `ReferenceError` | 참조할 수 없는 식별자 | `undefinedVar` |
| `TypeError` | 값의 타입이 맞지 않음 | `null.x` |
| `RangeError` | 허용 범위를 벗어난 값 | `new Array(-1)` |
| `URIError` | 잘못된 URI 처리 | `decodeURIComponent('%')` |
| `EvalError` | `eval` 관련 에러 | 현재는 거의 발생하지 않음 |

모두 `Error`를 상속하므로 `instanceof Error`는 `true`다.

### throw 문

```js
try {
  new Error('만들기만 함');          // 아무 일도 일어나지 않는다
  throw new Error('던져야 에러가 발생'); // 여기서 catch로 이동
} catch (err) {
  console.log(err.message); // 던져야 에러가 발생
}
```

에러 객체를 **만드는 것과 던지는 것은 다르다.** `throw`는 에러 객체가 아닌 값도 던질 수 있지만, `stack` 정보가 남는 `Error` 객체를 던지는 것이 디버깅에 유리하다.

### 에러의 전파

```js
const foo = () => { throw Error('foo에서 발생한 에러'); };
const bar = () => { foo(); };
const baz = () => { bar(); };

try {
  baz();
} catch (err) {
  console.log(err.message); // foo에서 발생한 에러
}
```

에러는 **호출자 방향으로 전파**된다. 콜 스택의 아래 방향으로 내려가다가 처음 만나는 `try...catch`에서 잡힌다.

```text
[foo] ← 에러 발생
[bar] ← 통과
[baz] ← 통과
[전역] ← try...catch에서 잡힘
```

#### 비동기 코드에서의 전파

`setTimeout` 콜백에서 발생한 에러는 바깥 `try...catch`로 잡을 수 없다.

```js
try {
  setTimeout(() => {
    throw new Error('Error!');
  }, 1000);
} catch (e) {
  console.error('캐치한 에러', e); // 실행되지 않는다
}
```

콜백이 실행될 때 `try` 블록은 이미 끝났고, 이벤트 루프가 비어 있는 콜 스택에 콜백을 올려 실행한다. 콜백을 호출한 것이 `try` 블록이 아니므로 에러가 바깥 `catch`까지 전파되지 않는다.

```text
[현재 실행]  try { setTimeout 호출 } 종료 → 콜 스택이 빔
                    ⋮ 1초 후
[새로운 실행] 콜백 실행 → throw → 받아 줄 try...catch가 없음
```

```js
async function load() {
  try {
    await new Promise((_, reject) =>
      setTimeout(() => reject(new Error('나중에 발생한 에러')), 100));
  } catch (err) {
    console.log('잡힘:', err.message); // 잡힘: 나중에 발생한 에러
  }
}
```

async/await을 쓰면 나중에 발생한 에러도 `await` 자리로 전달되므로 동기 코드처럼 처리할 수 있다.

| 방식 | 에러를 잡는 방법 |
| --- | --- |
| 콜백 | 콜백 안에서 직접 처리 (바깥 `try...catch` 불가) |
| 프로미스 | `catch` 메서드 |
| 제너레이터 + 실행기 | `gen.throw()`로 멈춘 `yield` 자리에 전달 |
| async/await | `try...catch` |

### 원인 에러를 담아 다시 던지기

ES2022부터 `Error` 생성자의 두 번째 인수로 `cause`를 전달할 수 있다. 낮은 수준의 에러를 의미 있는 에러로 감싸면서 원래 원인을 잃지 않는다.

```js
try {
  JSON.parse('{잘못된 JSON');
} catch (err) {
  throw new Error('설정 파일을 읽지 못했습니다', { cause: err });
}
// 바깥에서 err.cause.name → 'SyntaxError'
```