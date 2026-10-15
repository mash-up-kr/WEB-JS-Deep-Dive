# 45장. 프로미스

Ajax를 콜백으로 처리하다 보면 "콜백 지옥"에 빠진다. 비동기 처리의 결과와 상태를 하나의 객체로 다루는 방식인**프로미스(Promise)**를 통해 해결할 수 있다.

=&gt; 핵심은 "비동기 작업을 **값처럼** 다룰 수 있게 됐다"는 것이다. 콜백은 "일이 끝나면 이 함수를 불러줘"라고 떠넘기는 방식이라 중첩·에러 처리에 취약했는데, 프로미스는 "결과를 담은 객체"를 **반환**받아 `.then`으로 체이닝하므로 평평하고 에러도 한곳에서 잡힌다. 



## 비동기 처리를 위한 콜백 패턴의 단점

### 콜백 지옥

- 비동기 함수는 "함수 내부에 비동기로 동작하는 코드를 포함한 함수"다. 문제는 **비동기 함수의 처리 결과를 외부로 반환하거나 상위 스코프 변수에 할당할 수 없다**는 것이다.

```js
let todos;
function get(url) {
  const xhr = new XMLHttpRequest();
  xhr.open('GET', url);
  xhr.send();
  xhr.onload = () => {
    if (xhr.status === 200) {
      todos = JSON.parse(xhr.response); // 여기서 할당해도...
    }
  };
}
get('/todos/1');
console.log(todos); // undefined! (아직 응답이 안 옴)
```

`console.log(todos)`가 `undefined`인 이유는 `get`은 요청만 넘기고 **즉시 끝나버리고**(비동기), 응답은 한참 뒤 콜백에서 도착하기 때문이다. 그래서 비동기 결과는 **콜백 안에서** 써야 한다.

조금 더 뜯어보면, `xhr.onload` 콜백은 **응답이 도착했을 때 태스크 큐에 들어갔다가, 콜 스택이 빈 뒤에야** 실행된다. 반면 `console.log(todos)`는 `get` 호출 직후의 **동기 코드**라 그보다 훨씬 먼저 실행된다. 그래서 그 시점엔 `todos`가 아직 `undefined`다. 이것이 **"비동기 함수의 처리 결과는 반환값이나 상위 스코프 변수로 전달할 수 없다"**는 말의 실체이고, 콜백 패턴이 등장한 이유다.

문제는 "A 결과로 B를 요청하고, B로 C를 요청하는" 식으로 비동기가 이어지면 콜백이 콜백을 부르며 **오른쪽으로 끝없이 깊어진다.** 이것이 **콜백 지옥(callback hell)**이다.

```js
get('/posts/1', ({ userId }) => {
  get(`/users/${userId}`, (userInfo) => {
    get(`/users/${userInfo.id}/friends`, (friends) => {
      // ... 계단처럼 계속 깊어진다 (파멸의 피라미드)
    });
  });
});
```

### 에러 처리의 한계

- 콜백 패턴의 더 치명적인 문제는 **에러 처리가 안 된다**는 점이다. `try...catch`로 비동기 콜백의 에러를 잡을 수 없다.

```js
try {
  setTimeout(() => {
    throw new Error('Error!'); // 콜백 안에서 에러 발생
  }, 1000);
} catch (e) {
  console.error('캐치한 에러', e); // 여기서 못 잡는다!
}
```

=&gt; `try` 블록은 `setTimeout`을 **호출만 하고 즉시 끝난다.** 1초 뒤 콜백이 실행될 때는 이미 `try...catch`를 빠져나간 뒤이고, 콜백은 전혀 다른 실행 컨텍스트(이벤트 루프가 콜 스택에 새로 올린)에서 돌아간다. 에러는 **호출자 방향으로 전파**되는데, 그 시점의 호출자는 `try` 블록이 아니다. 그래서 못 잡는다.

<details class="orca-details">
<summary>에러가 "호출자 방향으로 전파된다"는 게 콜 스택으로 보면 무슨 뜻일까?</summary>

에러는 **자신을 호출한 함수(콜 스택에서 자기 바로 아래)로 거슬러 올라가며** 전파된다. `try...catch`는 그 전파 경로에 있어야만 에러를 잡을 수 있다.

동기 코드라면 이게 잘 맞는다.

```js
function a() { throw new Error('!'); }
function b() { a(); } // a의 호출자 = b
try { b(); } catch (e) {} // b의 호출자 = try → 전파 경로에 있음 → 잡힘
// 콜 스택: try > b > a  (에러가 a→b→try로 거슬러 올라감)
```

하지만 비동기 콜백은 **콜 스택이 한 번 비워진 뒤, 이벤트 루프가 새로 올린다**. 그래서 콜백이 실행될 때 콜 스택은 `setTimeout 콜백` 하나뿐이고, 그 아래에 `try` 블록이 없다.

```
[try { setTimeout(...) } catch] 실행 → 콜 스택 비워짐
... 1초 뒤 ...
[setTimeout 콜백] 만 콜 스택에 올라감 → 에러가 거슬러 올라가봐야 try가 없음
```

=&gt; 즉 "콜백의 에러를 try로 못 잡는다"는 건 비동기의 본질적 한계다. 프로미스는 이 문제를 **에러를 객체의 상태(rejected)로 저장**해두고, `.catch`로 나중에 꺼내 처리하는 방식으로 푼다. 에러를 "전파"가 아니라 "상태"로 다루는 발상의 전환이다.

</details>



## 프로미스의 생성

- 프로미스는 `Promise` 생성자 함수에 **executor(실행자) 함수**를 넘겨 만든다. 이 함수는 `resolve`와 `reject` 두 함수를 인수로 받는다.
- 비동기 처리가 **성공하면** `resolve`를, **실패하면** `reject`를 호출한다. `resolve`에 넘긴 값은 나중에 `then`이, `reject`에 넘긴 값은 `catch`가 받는다.

```js
const promise = new Promise((resolve, reject) => {
  if (/* 성공 */ true) {
    resolve('성공 결과');   // 성공 → 결과값을 담아 호출
  } else {
    reject(new Error('실패')); // 실패 → 에러를 담아 호출
  }
});
```

여기서 중요한 포인트가 하나 있다. `new Promise(...)`에 넘긴 executor 함수는 **프로미스가 생성되는 즉시, 동기적으로 실행**된다. "비동기 객체"라는 이름 때문에 executor도 나중에 실행될 것 같지만, 그 안의 `resolve`/`reject` 호출이 비동기일 뿐 **executor 자체는 지금 당장 돌아간다.**

```js
console.log('1');
new Promise((resolve) => {
  console.log('2'); // executor는 '즉시' 실행된다
  resolve();
}).then(() => console.log('4')); // then 콜백만 나중(마이크로태스크)
console.log('3');
// 출력: 1 → 2 → 3 → 4
```

<details class="orca-details">
<summary>executor 함수는 왜 비동기가 아니라 "즉시" 실행될까?</summary>

프로미스의 역할이 **"비동기 작업을 시작시키고, 그 결과를 담는 그릇"**이기 때문이다. 작업을 **시작**하는 코드(요청 보내기, 타이머 걸기)는 지금 당장 실행돼야 하고, **결과를 받는** 부분(`then`)만 나중에 실행되면 된다.

```js
new Promise((resolve) => {
  // 이 안은 '지금' 실행 → 비동기 작업을 '시작'시키는 자리
  xhr.send();              // 요청을 지금 보냄
  xhr.onload = () => resolve(xhr.response); // 결과는 나중에 resolve
});
```

만약 executor까지 나중에 실행된다면, "프로미스를 만들었는데 정작 요청은 아직 안 보낸" 이상한 상태가 된다. 그래서 executor는 즉시 실행해 작업을 걸어두고, 그 완료 시점에 `resolve`가 불려 상태가 바뀌는 구조다.

=&gt; 정리하면 **"작업 시작(executor) = 동기, 결과 수신(then·catch) = 비동기(마이크로태스크)"**다. 이 구분을 알면 위 예제의 `1→2→3→4` 순서가 자연스럽게 이해된다. 면접에서 "프로미스 안 코드는 언제 실행되나요?"를 물으면 이걸 답하면 된다.

</details>

<details class="orca-details">
<summary>executor 안에서 throw를 하면 어떻게 될까? reject랑 같을까?</summary>

**executor 안에서 에러를 던지면 자동으로 reject된 것과 똑같이 처리**된다. 이게 프로미스가 콜백보다 안전한 이유 중 하나다.

```js
const p = new Promise((resolve, reject) => {
  throw new Error('터짐'); // reject(new Error('터짐'))와 동일
});
p.catch((e) => console.error(e.message)); // '터짐' — catch로 잡힘
```

앞에서 봤듯(에러 처리의 한계) 콜백 패턴에선 비동기 함수 안의 `throw`를 바깥 `try...catch`로 못 잡았다. 그런데 프로미스는 executor 안의 `throw`를 **엔진이 가로채 rejected 상태로 바꿔**주므로, 바깥에서 `.catch`로 안전하게 처리할 수 있다.

=&gt; 그래서 프로미스 안에서는 에러를 "던져도(throw)" "거부해도(reject)" 결국 같은 rejected 경로로 흐른다. "비동기 에러를 상태로 저장한다"는 프로미스의 철학이 여기서도 드러난다. 단, `resolve`가 이미 호출된 뒤의 `throw`는 무시된다(이미 settled라 불변이니까).

</details>

### 프로미스의 상태

프로미스는 **비동기 처리의 상태**를 가지며, 상태에 따라 결과를 관리한다.

| 상태 | 의미 | 언제 |
| --- | --- | --- |
| **pending** | 비동기 처리가 아직 진행 중 | 프로미스 생성 직후 (기본) |
| **fulfilled** | 비동기 처리 **성공** | `resolve` 호출됨 |
| **rejected** | 비동기 처리 **실패** | `reject` 호출됨 |

- `pending`에서 시작해, `resolve`/`reject`가 호출되면 `fulfilled`/`rejected`로 바뀐다.
- `fulfilled` 또는 `rejected`가 된 상태를 **settled(처리됨)**라 한다.
- **한번 settled가 되면 다른 상태로 다시 바뀌지 않는다.** (결과는 확정되고 불변)

XHR 요청을 프로미스로 감싸면 이렇게 된다.

```js
const promiseGet = (url) => {
  return new Promise((resolve, reject) => {
    const xhr = new XMLHttpRequest();
    xhr.open('GET', url);
    xhr.send();
    xhr.onload = () => {
      if (xhr.status === 200) {
        resolve(JSON.parse(xhr.response)); // 성공 → 결과를 resolve
      } else {
        reject(new Error(xhr.status));     // 실패 → 에러를 reject
      }
    };
  });
};
```

=&gt; 즉 프로미스는 **"비동기 처리 상태와 처리 결과를 관리하는 객체"**다. 콜백을 넘겨주는 대신, "결과를 담을 그릇(프로미스 객체)"을 반환받는 것이다.

<details class="orca-details">
<summary>pending 상태인 프로미스를 console.log로 찍으면 뭐가 보일까?</summary>

프로미스 객체 자체를 출력하면, **내부 상태와 결과값이 함께** 보인다. 개발자 도구에서 직접 확인할 수 있다.

```js
const p1 = Promise.resolve(42);
console.log(p1); // Promise { <fulfilled>: 42 }

const p2 = new Promise(() => {}); // resolve를 영영 안 부름
console.log(p2); // Promise { <pending> }

const p3 = Promise.reject(new Error('x'));
console.log(p3); // Promise { <rejected>: Error: x }
```

여기서 꼭 짚을 것. **프로미스의 결과값은 `promise.value` 같은 걸로 직접 꺼낼 수 없다.** 위의 `<fulfilled>: 42`는 디버거가 보여주는 내부 슬롯(`[[PromiseState]]`·`[[PromiseResult]]`)일 뿐, 코드로 접근하는 프로퍼티가 아니다. 결과를 꺼내려면 **반드시 `then`(또는 `await`)을 거쳐야** 한다.

```js
const p = Promise.resolve(42);
console.log(p.value); // undefined — 이렇게는 못 꺼낸다
p.then((v) => console.log(v)); // 42 — 이렇게 꺼낸다
```

=&gt; "왜 프로미스에서 값을 바로 못 빼내고 then을 거쳐야 하나요?"라는 질문의 답이 이것이다. 값이 **아직 없을 수도 있고(pending)**, 언제 생길지 모르니, "생기면 그때 알려줘"라고 `then`으로 예약하는 구조다. 내부 슬롯은 엔진 전용이라 코드로는 닫혀 있다.

</details>

<details class="orca-details">
<summary>resolve()에 또 다른 프로미스를 넘기면 어떻게 될까?</summary>

**resolve에 프로미스를 넘기면, 그 프로미스가 끝날 때까지 기다렸다가 그 결과를 따라간다.** 이걸 "프로미스가 다른 프로미스에 **고정(resolve)된다**"고 한다. 면접에서 은근히 자주 나오는 포인트다.

```js
const inner = new Promise((resolve) => setTimeout(() => resolve('안쪽'), 1000));

const outer = new Promise((resolve) => {
  resolve(inner); // 값이 아니라 '프로미스'를 resolve
});

outer.then((v) => console.log(v)); // '안쪽' (1초 뒤) — inner의 결과를 그대로 받음
```

`outer`는 `inner`라는 프로미스로 resolve됐지만, `then`이 받는 값은 `inner` 객체가 아니라 `inner`가 **최종적으로 내놓은 값**(`'안쪽'`)이다. 프로미스가 중첩되지 않고 **납작하게 펴지는(flattening)** 것이다.

=&gt; 이 성질 덕분에 체이닝에서 "`then` 콜백이 프로미스를 return하면 다음 then이 그걸 기다린다"가 가능하다(뒤의 체이닝 토글과 연결된다). 그래서 `Promise.resolve(Promise.resolve(1))`도 결국 `1`이 된다 — 프로미스는 프로미스를 감싸지 않고 늘 한 겹으로 펴진다.

</details>

<details class="orca-details">
<summary>프로미스가 "한번 settled되면 안 바뀐다"는 게 왜 중요할까?</summary>

**결과가 확정되어 불변이기 때문에, 비동기 결과를 "값처럼" 안전하게 다룰 수 있다.**

콜백은 "언제, 몇 번 불릴지" 보장이 없다. 실수로 콜백을 두 번 부르거나, 성공·실패 콜백이 둘 다 불리는 버그가 생길 수 있다. 반면 프로미스는 **딱 한 번만 settled**되고, 그 뒤 `resolve`/`reject`를 또 호출해도 **무시**된다.

```js
const p = new Promise((resolve, reject) => {
  resolve('첫 번째'); // 이것만 반영됨
  resolve('두 번째'); // 무시
  reject('에러');     // 무시
});
p.then(console.log); // '첫 번째'
```

그리고 **이미 settled된 프로미스에 나중에 `.then`을 붙여도** 그 결과를 받을 수 있다. 결과가 객체에 저장돼 있기 때문이다.

```js
const p = Promise.resolve('완료');
setTimeout(() => p.then(console.log), 1000); // 1초 뒤에 붙여도 '완료' 받음
```

=&gt; 이 "한 번 확정, 이후 불변" 성질 덕분에 프로미스는 콜백보다 예측 가능하고 안전하다. "비동기 작업의 결과를 담은, 변하지 않는 값"이라는 관점이 프로미스의 핵심이다.

</details>



## 프로미스의 후속 처리 메서드

- 프로미스의 상태가 바뀌면(settled), 그 결과를 받아 처리할 후속 메서드를 쓴다. `then`·`catch`·`finally` 셋이다.
- **이 메서드들은 모두 프로미스를 반환**한다. (그래서 체이닝이 가능하다)
- 그리고 후속 처리 메서드의 콜백 함수는 **비동기로 동작**한다. 프로미스가 이미 settled 상태여도, 콜백은 지금 당장 실행되지 않고 마이크로태스크 큐를 거쳐 실행된다. (뒤의 "마이크로태스크 큐" 절에서 자세히)

프로미스의 상태에 따라, 세 메서드는 다음과 같이 선택적으로 콜백을 호출한다.

| 메서드 | fulfilled일 때 | rejected일 때 |
| --- | --- | --- |
| **then** | 첫 번째 콜백 호출 | 두 번째 콜백 호출 |
| **catch** | (호출 안 됨) | 콜백 호출 |
| **finally** | 콜백 호출 | 콜백 호출 |

### then

- 두 개의 콜백을 받는다. **첫 번째는 성공(fulfilled) 시**, **두 번째는 실패(rejected) 시** 호출된다. 두 콜백 모두 **선택적**이라, 성공만 처리하고 싶으면 첫 번째만 넘겨도 된다.

```js
// 성공 → 첫 번째 콜백
new Promise((resolve) => resolve('성공'))
  .then((v) => console.log(v), (e) => console.error(e)); // '성공'

// 실패 → 두 번째 콜백
new Promise((_, reject) => reject(new Error('실패')))
  .then((v) => console.log(v), (e) => console.error(e)); // Error: 실패
```

### catch

- 한 개의 콜백을 받고, **rejected 상태일 때만** 호출된다. `then(undefined, onRejected)`와 같다.

```js
new Promise((_, reject) => reject(new Error('실패')))
  .catch((e) => console.error(e)); // Error: 실패
```

### finally

- 성공·실패와 **상관없이 무조건 한 번** 호출된다. 뒷정리(로딩 스피너 끄기 등)에 쓴다.

```js
promiseGet('/todos/1')
  .then((res) => console.log(res))
  .catch((err) => console.error(err))
  .finally(() => console.log('로딩 끝')); // 성공이든 실패든 실행
```

<details class="orca-details">
<summary>then을 여러 번 쓰는 거랑 then을 체이닝하는 건 뭐가 다를까?</summary>

같은 프로미스에 `then`을 **여러 번 따로 붙이는 것**과, `then`을 **줄줄이 이어 붙이는 것**은 동작이 완전히 다르다. 이걸 헷갈리면 순서 버그가 난다.

**① 같은 프로미스에 then을 여러 번 (분기)**

하나의 프로미스 결과를 여러 곳에서 각자 받는다. 서로 독립적이고, **같은 값**을 받는다.

```js
const p = Promise.resolve(10);
p.then((v) => console.log(v * 2)); // 20
p.then((v) => console.log(v * 3)); // 30 — 둘 다 원본 10을 받음
```

**② then을 체이닝 (순차 변환)**

앞 `then`의 **반환값이 다음 then으로** 흘러간다. 값이 단계마다 바뀐다.

```js
Promise.resolve(10)
  .then((v) => v * 2)  // 20을 반환
  .then((v) => v * 3)  // 앞에서 온 20을 받아 60
  .then((v) => console.log(v)); // 60
```

=&gt; "하나의 결과를 여러 군데 뿌리면 **여러 번 then(분기)**, 결과를 단계적으로 가공하면 **체이닝**"이다. 체이닝은 컨베이어 벨트처럼 값이 한 방향으로 흐르고, 여러 번 then은 같은 샘에서 물을 여러 번 긷는 것과 같다.

</details>



## 프로미스의 에러 처리

- 에러는 `then`의 두 번째 인수로도 받을 수 있지만, `catch`를 쓰는 게 **권장**된다.  
=&gt;`then`의 두 번째 콜백은 **첫 번째 콜백(then의 성공 처리)에서 발생한 에러를 잡지 못한다.** 반면 `catch`를 체인 맨 뒤에 두면 **앞의 모든 then에서 난 에러까지** 잡기 때문이다.

```js
// 나쁜 예: then의 성공 콜백에서 난 에러를 두 번째 콜백이 못 잡음
promiseGet('/todos/1').then(
  (res) => console.xxx(res), // 여기서 에러 나면
  (err) => console.error(err) // ← 이 콜백은 못 잡는다!
);

// 좋은 예: catch가 앞의 모든 에러를 잡음
promiseGet('/todos/1')
  .then((res) => console.log(res))
  .catch((err) => console.error(err)); // then에서 난 에러도 여기서 잡힘
```

=&gt; 그래서 **"에러 처리는 체인 맨 끝의 `catch`로"**가 기본 원칙이다. 콜백 패턴에선 불가능했던 "한곳에서 모든 에러 처리"가 프로미스에선 `catch` 하나로 된다.

<details class="orca-details">
<summary>catch가 "앞의 모든 then 에러를 잡는다"는 게 어떻게 가능할까?</summary>

**프로미스 체인에서 에러(rejected)가 발생하면, 그 뒤의 `then`들은 건너뛰고 가장 가까운 `catch`까지 "흘러내려가기" 때문**이다.

핵심은 이것이다. `then`은 **rejected 프로미스를 만나면 성공 콜백을 실행하지 않고, rejected 상태를 그대로 다음으로 전달**한다. 그래서 에러는 체인을 따라 아래로 떨어지다가 `catch`를 만나면 거기서 처리된다.

```js
Promise.resolve()
  .then(() => { throw new Error('1단계 에러'); }) // 여기서 reject
  .then(() => console.log('실행 안 됨'))          // 건너뜀 (rejected라서)
  .then(() => console.log('이것도 안 됨'))         // 건너뜀
  .catch((e) => console.error('잡힘:', e.message)); // '잡힘: 1단계 에러'
```

이건 마치 동기 코드에서 `try` 블록 안 어디서 에러가 나든 하나의 `catch`로 모이는 것과 같다. 프로미스 체인이 "비동기판 try...catch"처럼 동작하는 셈이다.

=&gt; 주의할 점: `catch` 뒤에 또 `then`을 붙이면, catch가 에러를 처리한 뒤 **다시 fulfilled 상태로** 체인이 이어진다(catch가 에러를 "삼켰으니"). 그래서 "에러가 나도 계속 진행"하려면 catch를 중간에, "에러 나면 멈춤"이면 맨 끝에 둔다. 에러의 흐름을 설계할 수 있다는 게 프로미스의 강점이다.

</details>

<details class="orca-details">
<summary>catch를 안 붙이면 에러는 어디로 갈까? (unhandled rejection)</summary>

**아무도 안 잡은 reject는 "처리되지 않은 거부(unhandled rejection)"가 되어, 콘솔에 경고가 뜨거나 프로그램이 죽을 수 있다.** 실무에서 꽤 자주 겪는 문제다.

```js
Promise.reject(new Error('아무도 안 잡음'));
// 콘솔: Uncaught (in promise) Error: 아무도 안 잡음
```

조용히 넘어가는 게 아니라, 브라우저·Node 모두 "이 에러 처리 안 했어요!"라고 알려준다. 특히 **Node.js에서는 처리되지 않은 rejection이 프로세스 전체를 종료**시킬 수 있다(버전에 따라). 그래서 전역으로 잡아두는 안전망을 걸기도 한다.

```js
// 브라우저
window.addEventListener('unhandledrejection', (e) => {
  console.error('안 잡힌 에러:', e.reason);
  e.preventDefault(); // 기본 경고 막기
});

// Node.js
process.on('unhandledRejection', (reason) => {
  console.error('안 잡힌 에러:', reason);
});
```

=&gt; 그래서 **모든 프로미스 체인은 끝에 `catch`(또는 async/await의 try…catch)를 붙이는 게 원칙**이다. "에러가 안 나겠지"라고 catch를 생략하면, 실제 에러가 났을 때 조용히 사라지는 게 아니라 엉뚱한 곳에서 터진다. 면접에서 "프로미스 에러를 안 잡으면 어떻게 되나요?"의 답이 바로 이 unhandled rejection이다.

</details>



## 프로미스 체이닝

- `then`·`catch`·`finally`가 **항상 프로미스를 반환**하므로, 연속해서 이어 붙일 수 있다. 이것이 **프로미스 체이닝(chaining)**이다.
- 콜백 지옥의 "오른쪽으로 깊어지던" 중첩이, 체이닝에선 **아래로 평평하게** 펴진다.

```js
const url = 'https://jsonplaceholder.typicode.com';

// 콜백 지옥이던 것이...
promiseGet(`${url}/posts/1`)
  .then(({ userId }) => promiseGet(`${url}/users/${userId}`)) // 프로미스 반환
  .then((userInfo) => console.log(userInfo))
  .catch((err) => console.error(err)); // 에러는 한곳에서
```

핵심은 이것이다. `then`의 콜백에서 프로미스를 `return`하면, 다음 `then`은 그 프로미스가 settled될 때까지 **기다렸다가 결과를 받는다.** 그래서 "순차적 비동기"가 중첩 없이 표현된다.

=&gt; 다만 체이닝도 결국 콜백(`then`의 인수)을 쓴다. 이마저도 동기 코드처럼 보이게 하는 게 **async/await**다.

```js
// 같은 걸 async/await으로 
(async () => {
  const { userId } = await promiseGet(`${url}/posts/1`);
  const userInfo = await promiseGet(`${url}/users/${userId}`);
  console.log(userInfo);
})();
```

<details class="orca-details">
<summary>then 콜백에서 "값을 return"하는 것과 "프로미스를 return"하는 건 뭐가 다를까?</summary>

둘 다 다음 `then`으로 전달되지만, **프로미스를 반환하면 "그게 끝날 때까지 기다린다"**는 차이가 있다.

- **일반 값 반환**: 그 값이 즉시 다음 then으로 전달된다. (자동으로 `Promise.resolve`로 감싸짐)
- **프로미스 반환**: 그 프로미스가 settled될 때까지 **다음 then이 대기**하고, 그 결과값이 전달된다.

```js
Promise.resolve(1)
  .then((n) => n + 1)            // 일반 값 2 반환 → 즉시 전달
  .then((n) => Promise.resolve(n * 10)) // 프로미스 반환 → 기다렸다가 20 전달
  .then((n) => console.log(n));  // 20
```

이 "프로미스를 반환하면 기다린다"는 규칙 덕분에, 비동기 작업을 **순차적으로 줄 세울** 수 있다. 앞 요청의 응답이 와야 다음 요청을 보내는 체인이 자연스럽게 만들어진다.

=&gt; 흔한 실수: `then` 콜백에서 프로미스를 `return` 안 하고 그냥 호출만 하면, 다음 then이 기다리지 않고 바로 실행돼 순서가 꼬인다. "비동기를 이어 붙이려면 반드시 return"이 체이닝의 철칙이다.

</details>



## 프로미스의 정적 메서드

`Promise`는 생성자지만, 편의를 위한 **정적 메서드** 5가지를 함께 제공한다.

### Promise.resolve / Promise.reject

- **이미 존재하는 값**을 프로미스로 감쌀 때 쓴다. `resolve`는 fulfilled 프로미스를, `reject`는 rejected 프로미스를 만든다.

```js
// new Promise((resolve) => resolve([1,2,3])) 과 동일
Promise.resolve([1, 2, 3]).then(console.log); // [1, 2, 3]
Promise.reject(new Error('에러')).catch(console.error); // Error: 에러
```

### Promise.all — 병렬 처리, 모두 성공해야

- 여러 프로미스를 **병렬로 실행**하고, **전부 fulfilled되면** 결과를 **배열로** 반환한다. (순서 보장)
- **하나라도 rejected되면 즉시 전체가 rejected**된다(나머지를 기다리지 않음).

```js
Promise.all([
  promiseGet('/users/1'),
  promiseGet('/users/2'),
  promiseGet('/users/3'),
])
  .then((users) => console.log(users)) // [user1, user2, user3] (요청 순서대로)
  .catch(console.error); // 하나라도 실패하면 여기로
```

"독립적인 여러 요청을 동시에 보내고, 다 모이면 처리"할 때 쓴다. 순차로 하면 느린 걸 병렬로 묶어 빠르게 만든다.

몇 가지 세부 동작을 더 알아두자.

- **결과 순서가 보장된다.** 먼저 끝난 순서가 아니라, **전달한 프로미스의 순서대로** 결과 배열이 만들어진다. 그래서 `[users[0], users[1], users[2]]`가 요청 순서와 일치한다.
- **하나라도 rejected되면 즉시 전체가 rejected**된다. 나머지 프로미스가 끝나기를 기다리지 않고 바로 `catch`로 간다. 이를 **fail-fast**(빠른 실패)라 한다.
- 배열에 **프로미스가 아닌 값**을 넣으면, 자바스크립트가 `Promise.resolve`로 감싸서 처리한다. 그래서 프로미스와 일반 값을 섞어 넣어도 된다.

```js
Promise.all([1, 2, promiseGet('/users/1')]) // 1, 2는 Promise.resolve(1), (2)로 래핑됨
  .then(console.log); // [1, 2, user]
```

<details class="orca-details">
<summary>반복문(for) 안에서 await을 돌리면 왜 느려질까?</summary>

**`for` 안에서 `await`을 쓰면 요청이 "하나 끝나야 다음 하나"가 되어 순차 실행**된다. 독립적인 요청이라면 이건 큰 낭비다. 

```js
// 느림: 순차 (요청 10개 × 1초 = 총 10초)
const users = [];
for (const id of ids) {
  users.push(await promiseGet(`/users/${id}`)); // 하나씩 기다림
}

// 빠름: 병렬 (요청 10개를 동시에 = 총 1초)
const users = await Promise.all(
  ids.map((id) => promiseGet(`/users/${id}`)) // 전부 띄워놓고 한 번에 대기
);
```

`for...await`은 "이전 요청의 응답을 받고 나서야 다음 요청을 시작"한다. 그래서 요청들이 서로 독립적인데도 대기 시간이 **합(순차)**으로 쌓인다. `Promise.all`은 요청을 **전부 먼저 띄워놓고** 한꺼번에 기다리므로, 대기 시간이 **최댓값(병렬)**으로 줄어든다.

=&gt; "요청끼리 순서·의존성이 있으면 for await(순차), 독립적이면 `Promise.all`(병렬)"이 기준이다. 면접에서 "반복문 안 await의 문제"는 단골 질문인데, `map` + `Promise.all`로 **병렬화**하는 게 모범 답안이다. 단, 앞 요청 결과가 다음 요청에 필요한 경우(의존적)엔 순차가 맞다.

</details>

### Promise.race — 가장 빠른 하나

- 여러 프로미스 중 **가장 먼저 settled된 하나의 결과**를 반환한다. (경주, race)

```js
Promise.race([
  new Promise((resolve) => setTimeout(() => resolve(1), 3000)),
  new Promise((resolve) => setTimeout(() => resolve(2), 2000)),
  new Promise((resolve) => setTimeout(() => resolve(3), 1000)), // 가장 빠름
]).then(console.log); // 3
```

<details class="orca-details" open>
<summary>Promise.race로 "요청 타임아웃"을 만들 수 있다던데?</summary>

맞다. **"진짜 요청"과 "일정 시간 뒤 reject되는 타이머"를 race시키면**, 요청이 그 시간 안에 안 오면 타임아웃 에러가 나게 할 수 있다. `race`의 대표적인 실무 활용이다.

```js
function withTimeout(promise, ms) {
  const timeout = new Promise((_, reject) =>
    setTimeout(() => reject(new Error('시간 초과')), ms)
  );
  return Promise.race([promise, timeout]); // 둘 중 먼저 끝나는 쪽
}

withTimeout(promiseGet('/slow-api'), 3000)
  .then((data) => console.log(data))
  .catch((err) => console.error(err.message)); // 3초 넘으면 '시간 초과'
```

요청이 3초 안에 응답하면 그 결과가 이기고, 3초가 먼저 지나면 타이머의 reject가 이겨서 `catch`로 간다.

=&gt; 단, 주의할 점. `race`에서 타이머가 이겨도 **진짜 요청이 취소되는 건 아니다.** 요청은 백그라운드에서 계속 진행된다(그냥 결과를 무시할 뿐). 실제로 요청까지 끊으려면 `AbortController`를 함께 써야 한다. "race는 '누가 먼저냐'를 가릴 뿐, 진 쪽을 멈추진 않는다"는 게 포인트다.

</details>

### Promise.allSettled — 모두 끝까지 기다림(성공·실패 무관)

- 전달한 프로미스가 **모두 settled될 때까지 기다린** 뒤, 각각의 성공/실패를 **객체 배열로** 반환한다. `all`과 달리 **하나가 실패해도 중단하지 않는다.**

```js
Promise.allSettled([
  new Promise((resolve) => setTimeout(() => resolve(1), 2000)),
  new Promise((_, reject) => setTimeout(() => reject(new Error('실패')), 1000)),
]).then(console.log);
/*
[
  { status: 'fulfilled', value: 1 },
  { status: 'rejected', reason: Error: 실패 }
]
*/
```

<details class="orca-details">
<summary>Promise.all vs allSettled, 실무에서 언제 뭘 쓸까?</summary>

**"하나라도 실패하면 전부 실패로 볼 것이냐"**로 갈린다.

- `Promise.all` : 모든 요청이 **다 성공해야 의미 있을 때.** 하나라도 실패하면 즉시 중단하고 에러로 처리한다. 예: "결제 + 재고 차감 + 배송 등록"을 한 번에 — 하나라도 실패하면 전체를 롤백해야 하는 경우.

```js
// 하나라도 실패하면 전체 실패 → 빠른 실패(fail-fast)
Promise.all([pay(), reduceStock(), registerShip()])
  .then(() => '주문 완료')
  .catch(() => '주문 실패'); // 하나만 터져도 여기로
```

- `Promise.allSettled` : **각각 독립적이라 일부 실패해도 나머지는 살려야 할 때.** 예: "유저 10명의 프로필을 동시에 불러오는데, 몇 명 실패해도 성공한 사람 건 보여주기".

```js
// 실패한 것만 걸러내고 성공한 것은 쓴다
const results = await Promise.allSettled(ids.map(fetchUser));
const users = results
  .filter((r) => r.status === 'fulfilled')
  .map((r) => r.value);
```

=&gt; 한마디로 **"전부 성공이 조건이면 all(fail-fast), 부분 성공을 허용하면 allSettled"**다. 참고로 `Promise.any`(ES2021)라는 것도 있는데, "하나라도 **성공**하면 그걸 반환"(race의 성공 버전)이라 여러 미러 서버 중 가장 먼저 성공하는 응답을 쓸 때 유용하다.

</details>

<details class="orca-details">
<summary>Promise.all이 "병렬"이라는데, 자바스크립트는 싱글 스레드 아닌가?</summary>

맞다. 하지만 **"병렬"의 의미가 "요청을 동시에 띄워놓고 기다린다"는 뜻**이라 모순이 아니다.

네트워크 요청 같은 비동기 작업은 **브라우저(멀티 스레드)가 처리**한다. 자바스크립트 엔진은 요청을 "띄워놓기만" 하고 바로 다음 줄로 넘어간다. 그래서 `Promise.all`에 요청 3개를 넣으면, 엔진이 **셋을 거의 동시에 브라우저에 던져놓고**, 응답이 다 올 때까지 기다린다.

```js
// 순차: 앞 요청이 끝나야 다음 (총 3초)
const a = await req(); // 1초
const b = await req(); // 1초
const c = await req(); // 1초

// 병렬: 셋을 동시에 띄움 (총 1초 — 가장 느린 것 기준)
const [a, b, c] = await Promise.all([req(), req(), req()]);
```

=&gt; 그래서 "엔진은 싱글 스레드지만, 네트워크 I/O는 브라우저가 병렬로 해주니" `Promise.all`로 독립적인 요청을 묶으면 전체 대기 시간이 **합(순차)이 아니라 최댓값(병렬)**으로 줄어든다. 이게 실무에서 성능 최적화의 핵심 기법이다.

</details>



## 마이크로태스크 큐

- 프로미스 후속 처리 메서드(`then`·`catch`·`finally`)의 콜백은, 일반 태스크 큐가 아니라 **마이크로태스크 큐**에 저장된다.
- 마이크로태스크 큐는 **태스크 큐보다 우선순위가 높다.** 그래서 프로미스 콜백이 `setTimeout` 콜백보다 먼저 실행된다.

```js
setTimeout(() => console.log(1), 0); // 태스크 큐

Promise.resolve()
  .then(() => console.log(2))   // 마이크로태스크 큐
  .then(() => console.log(3));

// 출력: 2 → 3 → 1
```

=&gt; `setTimeout(…, 0)`이 먼저 쓰여 있는데도 `2 → 3 → 1` 순서가 나오는 이유다. 이벤트 루프는 콜 스택이 비면 **마이크로태스크 큐를 전부 비운 뒤** 태스크 큐를 처리한다. 



## fetch

- `fetch` : XMLHttpRequest보다 간단하고 **프로미스를 기반으로 하는** HTTP 요청 Web API

```js
fetch('https://jsonplaceholder.typicode.com/todos/1')
  .then((response) => response.json()) // 응답을 JSON으로 파싱 (이것도 프로미스)
  .then((json) => console.log(json))
  .catch((err) => console.error(err));
```

`fetch`는 **프로미스를 반환**하므로, 응답을 `then`으로 받는다. 이때 첫 번째 `then`이 받는 것은 데이터가 아니라 **Response 객체**다. 실제 데이터를 꺼내려면 `response.json()`(또는 `text()` 등)을 호출해야 하는데, **이 메서드도 프로미스를 반환**하므로 `then`을 한 번 더 거친다. 그래서 fetch는 보통 `then`이 두 번 이어진다.

fetch로 REST API 요청(GET·POST·PATCH·DELETE)을 한데 묶으면 이렇게 정리할 수 있다. XHR로 작성하던 것보다 훨씬 간결하다.

```js
const request = {
  get(url) {
    return fetch(url);
  },
  post(url, payload) {
    return fetch(url, {
      method: 'POST',
      headers: { 'content-Type': 'application/json' },
      body: JSON.stringify(payload),
    });
  },
  patch(url, payload) {
    return fetch(url, {
      method: 'PATCH',
      headers: { 'content-Type': 'application/json' },
      body: JSON.stringify(payload),
    });
  },
  delete(url) {
    return fetch(url, { method: 'DELETE' });
  },
};

request
  .get('https://jsonplaceholder.typicode.com/todos/1')
  .then((response) => response.json())
  .then((todos) => console.log(todos))
  .catch((err) => console.error(err));
```

- 주의할 점이 하나 있다. `fetch`는 **HTTP 에러(4xx, 5xx)를 reject하지 않는다.** 네트워크 자체가 끊기거나 CORS 에러일 때만 reject한다. 404·500 같은 응답도 "일단 응답은 받았으니" **fulfilled로 처리**하고, `response.ok`를 `false`로 줄 뿐이다.

```js
fetch('/없는주소')
  .then((response) => {
    if (!response.ok) throw new Error(response.status); // 직접 확인해야!
    return response.json();
  })
  .then((data) => console.log(data))
  .catch((err) => console.error(err));
```

<details class="orca-details">
<summary>fetch는 왜 404를 에러로 안 볼까? axios는 다르다던데</summary>

**fetch의 설계 철학이 "서버에서 응답이 오기만 하면 성공"**이기 때문이다. fetch 입장에서 "실패(reject)"는 **요청이 서버에 닿지도 못한 경우**(네트워크 단절, CORS 차단)만을 말한다. 404·500은 "서버가 '없어요/에러났어요'라고 **정상적으로 대답한 것**"이라, fetch는 이걸 통신 성공으로 본다.

그래서 fetch를 쓸 땐 `response.ok`(또는 `response.status`)를 **직접 확인**해야 한다. 안 그러면 404 응답이 성공 경로(`.then`)로 흘러가 버그가 된다.

```js
// fetch: HTTP 에러를 직접 걸러야 함
const res = await fetch('/api');
if (!res.ok) throw new Error(res.status); // 이걸 빼먹기 쉽다

// axios: 2xx 외 상태는 자동으로 reject → catch로 바로 감
try {
  const res = await axios.get('/api');
} catch (e) {
  console.log(e.response.status); // 404, 500 등 자동으로 여기로
}
```

=&gt; 그래서 실무에서 `axios`를 선호하는 이유 중 하나가 이것이다. axios는 **4xx·5xx를 자동으로 reject**해줘서 에러 처리가 직관적이고, JSON 변환·타임아웃·인터셉터 같은 편의 기능도 많다. "fetch는 내장이지만 ok 체크를 잊지 말 것, axios는 그걸 알아서 해준다"로 기억하면 된다.

</details>

# 46장. 제너레이터와 async/await

프로미스로 콜백 지옥은 풀었지만, `then`을 줄줄이 잇는 체이닝도 결국 콜백을 쓴다. async/await는 그 프로미스를 **마치 동기 코드처럼** 쓰게 해주는 문법이다. `then` 없이 `await` 한 줄로 결과를 받고, 에러는 익숙한 `try...catch`로 잡는다.

=&gt; 핵심은 "비동기 코드를 **위에서 아래로 읽히게** 만든다"는 것이다. 그 바탕에 제너레이터가 있는데, async/await는 사실 "제너레이터로 비동기를 다루던 패턴을 언어가 문법으로 감싼 것"이다. 그래서 제너레이터를 먼저 짧게 보고, 본론인 async/await으로 넘어간다.



## 제너레이터란?

- 제너레이터(generator)는 **코드 블록의 실행을 일시 중지했다가, 필요한 시점에 재개**할 수 있는 특수한 함수다.
- 일반 함수는 한번 호출되면 끝까지 쭉 실행되고, 호출자는 중간에 개입할 수 없다. 제너레이터는 **실행의 흐름을 호출자가 쥐고** 멈추고 재개할 수 있다.

제너레이터와 일반 함수의 차이는 크게 세 가지다.

1. **제너레이터 함수는 함수 호출자에게 실행 제어권을 양도할 수 있다.** 일반 함수는 호출되면 제어권이 함수에 넘어가 끝까지 실행되지만, 제너레이터는 `yield`로 **실행을 호출자에게 다시 넘긴다.**
2. **제너레이터 함수는 함수 호출자와 상태를 주고받을 수 있다.** 일반 함수는 매개변수로 한 번 받고 끝이지만, 제너레이터는 실행 도중에도 호출자와 값을 **양방향으로** 교환할 수 있다.
3. **제너레이터 함수를 호출하면 제너레이터 객체를 반환한다.** 일반 함수는 호출하면 몸체를 실행하고 값을 반환하지만, 제너레이터는 몸체를 실행하지 않고 **제너레이터 객체(이터러블이면서 이터레이터)**를 반환한다.

=&gt; 이 "멈췄다 재개한다 + 호출자와 상태를 주고받는다"는 성질이 핵심이다. 비동기 작업을 `yield`로 멈춰두고 결과가 오면 재개하면 **비동기를 동기처럼** 쓸 수 있는데, 바로 이 패턴을 언어가 공식 문법으로 만든 것이 async/await다.



## 제너레이터 함수의 정의

- 제너레이터 함수는 `function*` 키워드로 선언한다. 애스터리스크(`*`)의 위치는 `function`과 함수 이름 사이라면 어디든 상관없지만, 일반적으로 `function` 바로 뒤에 붙인다.

```js
// 선언문
function* genDecFunc() {
  yield 1;
}

// 표현식
const genExpFunc = function* () {
  yield 1;
};

// 메서드
const obj = {
  * genObjMethod() {
    yield 1;
  },
};

// 클래스 메서드
class MyClass {
  * genClsMethod() {
    yield 1;
  }
}
```

제너레이터 함수에는 두 가지 제약이 있다.

- **화살표 함수로 정의할 수 없다.**

```js
const genArrowFunc = * () => {}; // SyntaxError
```

- `new` 연산자와 함께 **생성자 함수로 호출할 수 없다.** 제너레이터는 non-constructor다.

```js
function* genFunc() { yield 1; }
new genFunc(); // TypeError: genFunc is not a constructor
```



## 제너레이터 객체

- 제너레이터 함수를 호출하면 몸체가 실행되는 게 아니라 **제너레이터 객체**를 반환한다. 이 객체는 **이터러블(iterable)이면서 동시에 이터레이터(iterator)**다.
- 즉 `Symbol.iterator` 메서드를 가져 이터러블이고, `next` 메서드도 가져 이터레이터이기도 하다. 그래서 `for...of`로 순회하거나 스프레드로 펼칠 수 있다.

```js
function* genFunc() {
  yield 1;
  yield 2;
  yield 3;
}
const gen = genFunc(); // 제너레이터 객체 반환 (아직 몸체 실행 안 됨)

// 이터레이터: next로 하나씩
console.log(gen.next()); // { value: 1, done: false }

// 이터러블: for...of / 스프레드도 가능
console.log([...genFunc()]); // [1, 2, 3]
```

제너레이터 객체는 `next` 외에 `return`과 `throw` 메서드도 갖는다.

| 메서드 | 동작 | 반환 |
| --- | --- | --- |
| **next()** | 다음 `yield`까지 실행하고 멈춤 | `{ value: yield된 값, done: false }` |
| **return(v)** | 제너레이터를 **강제 종료** | `{ value: v, done: true }` |
| **throw(e)** | 제너레이터 안에 에러를 던지고 종료 | `{ value: undefined, done: true }` |

```js
const gen = genFunc();
console.log(gen.next());        // { value: 1, done: false }
console.log(gen.return('끝'));  // { value: '끝', done: true } — 즉시 종료
console.log(gen.next());        // { value: undefined, done: true } — 이미 끝남
```

=&gt; `done`이 `true`가 되면 제너레이터는 더 이상 진행하지 않는다. `done: false`는 "아직 남았다", `done: true`는 "끝났다"는 신호다.



## 제너레이터의 일시 중지와 재개

- 제너레이터는 `yield`로 **실행을 멈추고**, `next()` 호출로 **다음 `yield`까지 재개**한다. 이 과정에서 호출자와 값을 주고받는다.

```js
function* genFunc() {
  const x = yield 1; // ① yield 1로 멈춤 → next()로 재개되면 x에 값 할당
  const y = yield x + 10; // ② yield로 멈춤
  return x + y;
}

const gen = genFunc();
console.log(gen.next());   // { value: 1, done: false }  — ①의 yield 1까지 실행
console.log(gen.next(10)); // { value: 20, done: false } — 10이 x에 할당 → x+10=20
console.log(gen.next(20)); // { value: 30, done: true }  — 20이 y에 할당 → return 30
```

여기서 두 방향의 값 전달을 구분하는 게 핵심이다.

- **제너레이터 → 호출자**: `yield` 뒤의 값이 `next()`가 반환하는 객체의 `value`로 나간다.
- **호출자 → 제너레이터**: `next(인수)`로 넘긴 인수가, **멈춰 있던 `yield` 표현식의 평가 결과**로 들어간다.

주의할 점은 **처음 호출하는 `next()`에는 인수를 전달해도 무시**된다는 것이다. 첫 `next()`는 "첫 `yield`까지 실행"만 할 뿐, 아직 어떤 `yield`도 호출자의 값을 기다리고 있지 않기 때문이다.

=&gt; 즉 제너레이터는 `yield`로 값을 내보내고, `next(인수)`로 값을 받아들이는 **양방향 통신 창구**를 가진다. 이 양방향성이 "비동기 결과를 받아 재개한다"를 가능하게 하는 바탕이다.

<details class="orca-details">
<summary>next()에 넘긴 값이 "직전 yield의 결과"가 된다는 게 왜 헷갈릴까?</summary>

`yield`가 **값을 내보내는 동시에, 재개될 때 값을 받아들이는 "구멍"** 역할을 두 번에 나눠 하기 때문이다. 한 `yield`가 "출력"과 "입력"을 서로 다른 시점에 담당한다.

```js
function* gen() {
  const received = yield '나갑니다'; // 이 yield는...
  console.log('받은 값:', received);
}

const g = gen();
console.log(g.next());       // { value: '나갑니다', done: false }
//   → 여기서 yield는 '나갑니다'를 '내보내고' 멈춘다 (아직 received는 미정)
console.log(g.next('들어옴')); // '받은 값: 들어옴'
//   → next로 넘긴 '들어옴'이 '멈춰 있던 yield 표현식의 값'이 되어 received에 할당
```

같은 `yield '나갑니다'` 한 줄이, **첫 next에선 '나갑니다'를 출력**하고, **두 번째 next에선 '들어옴'을 입력**받는다. 출력과 입력의 시점이 어긋나 있어서 헷갈린다.

=&gt; "yield는 값을 내보내며 멈추고, 그 자리는 다음 next의 인수로 채워진다"로 기억하면 된다. 그래서 첫 next에는 채울 yield가 아직 없어 인수가 무시되는 것이다. 이 양방향 구조가 async/await에서 "await로 멈추고, 결과가 오면 그 자리에 값이 채워져 재개"되는 것과 똑같은 원리다.

</details>



## 제너레이터의 활용

### 이터러블 구현

- 제너레이터로 이터러블을 아주 간결하게 만들 수 있다. 특히 **무한 수열**처럼 미리 다 만들 수 없는 것도, 요청이 올 때마다 하나씩 계산(지연 평가)해 표현한다.

```js
// 제너레이터 없이 무한 피보나치 이터러블 (수동 구현 — 번거롭다)
const infiniteFiboOld = (function () {
  let [pre, cur] = [0, 1];
  return {
    [Symbol.iterator]() { return this; },
    next() {
      [pre, cur] = [cur, pre + cur];
      return { value: cur };
    },
  };
}());

// 제너레이터로 (훨씬 간결)
const infiniteFibo = (function* () {
  let [pre, cur] = [0, 1];
  while (true) {
    [pre, cur] = [cur, pre + cur];
    yield cur; // 요청(next)이 올 때마다 다음 값 하나만 계산
  }
}());

for (const num of infiniteFibo) {
  if (num > 100) break;
  console.log(num); // 1 2 3 5 8 13 21 34 55 89
}
```

일반 배열이라면 무한 개의 값을 미리 만들 수 없지만, 제너레이터는 **값을 미리 다 만들지 않고 `next`가 올 때마다 하나씩** 계산한다. 이를 **지연 평가(lazy evaluation)**라 한다.

### 비동기 처리

- 제너레이터의 "멈췄다 재개" 성질을 이용하면 **비동기를 동기처럼** 쓸 수 있다. `yield`로 비동기 작업을 멈춰두고, 결과가 오면 `next(결과)`로 재개하는 것이다.

```js
function* fetchTodo() {
  const url = 'https://jsonplaceholder.typicode.com/todos/1';
  // yield로 멈추고, 실행기가 fetch 결과를 next로 넣어주면 response에 할당된다
  const response = yield fetch(url);
  const todo = yield response.json();
  console.log(todo);
}

// 제너레이터를 자동으로 재개해주는 '실행기'가 필요하다
function run(genFunc) {
  const gen = genFunc();
  function onResolved(result) {
    const { value, done } = gen.next(result); // 결과를 넣어 재개
    if (done) return;
    value.then(onResolved); // 다음 비동기가 끝나면 또 재개
  }
  onResolved();
}
run(fetchTodo);
```

=&gt; 이 "제너레이터 + 실행기" 패턴이 바로 async/await의 전신이다. `yield`가 `await`로, `function*`가 `async`로, 그리고 저 번거로운 실행기가 **언어 내부로 들어가 자동화**된 것이 async/await다. 그래서 async/await를 이해하려면 제너레이터의 이 동작을 알아야 한다.

<details class="orca-details">
<summary>제너레이터를 요즘 실무에서 직접 쓸 일이 있을까?</summary>

비동기 처리 용도로는 **거의 안 쓴다.** 그 자리를 async/await가 완전히 대체했기 때문이다. 하지만 제너레이터 자체가 사라진 건 아니고, 지연 평가나 세밀한 흐름 제어가 필요한 특수한 경우에 빛을 발한다.

```js
// 고유 id를 무한히 뽑아내는 생성기
function* idMaker() {
  let id = 1;
  while (true) yield id++; // next() 할 때만 한 칸씩 진행
}
const gen = idMaker();
gen.next().value; // 1
gen.next().value; // 2
```

Redux-Saga 같은 라이브러리가 제너레이터로 비동기 흐름(취소·재시도·동시성 제어 등)을 세밀하게 다루고, 무한 수열이나 커스텀 이터레이터를 만들 때도 유용하다.

=&gt; 정리하면, "비동기는 async/await에 자리를 내줬지만, 무한 수열·지연 평가·세밀한 흐름 제어가 필요하면 제너레이터가 여전히 쓸모 있다"는 정도로 알아두면 된다. 제너레이터를 배우는 진짜 이유는 async/await의 뿌리를 이해하기 위해서다.

</details>



## async/await

- async/await는 **프로미스를 기반으로 동작**한다. 프로미스의 `then`·`catch`·`finally` 없이, 마치 동기 코드처럼 비동기를 다루게 해준다.
- 기존 프로미스 체이닝과 비교하면 차이가 분명하다.

```js
// 프로미스 체이닝
function getUser() {
  return promiseGet('/posts/1')
    .then(({ userId }) => promiseGet(`/users/${userId}`))
    .then((userInfo) => console.log(userInfo))
    .catch((err) => console.error(err));
}

// async/await — 위에서 아래로 읽힌다
async function getUser() {
  try {
    const { userId } = await promiseGet('/posts/1');
    const userInfo = await promiseGet(`/users/${userId}`);
    console.log(userInfo);
  } catch (err) {
    console.error(err);
  }
}
```

=&gt; 체이닝에선 콜백 안으로 들어가야 결과를 쓸 수 있었는데, async/await에선 `await`가 **결과를 바로 변수에 담아준다.** 동기 코드와 모양이 거의 같아져서 읽기 쉽다.



## async 함수

- `async` 키워드를 함수 앞에 붙이면 **async 함수**가 된다. 선언문·표현식·화살표 함수·메서드 어디에나 붙일 수 있다.
- async 함수의 가장 중요한 성질은 **언제나 프로미스를 반환한다**는 것이다. 명시적으로 프로미스를 반환하지 않아도, 반환값을 `resolve`하는 프로미스가 암묵적으로 반환된다.

```js
// 여러 형태
async function foo(n) { return n; }         // 선언문
const bar = async function (n) { return n; }; // 표현식
const baz = async (n) => n;                 // 화살표 함수
const obj = { async qux(n) { return n; } }; // 메서드
class MyClass { async quux(n) { return n; } } // 클래스 메서드

// return n만 했는데도 프로미스가 반환된다
foo(1).then((v) => console.log(v)); // 1
```

`foo(1)`은 숫자 `1`이 아니라 `1`로 resolve된 **프로미스**를 반환한다. 그래서 `.then`으로 결과를 꺼낼 수 있다.

=&gt; 즉 async를 붙이는 순간, 그 함수의 반환값은 **무조건 프로미스로 감싸진다.** 이게 async 함수와 일반 함수의 결정적 차이다.

<details class="orca-details">
<summary>async 함수에서 return한 값은 왜 프로미스로 감싸질까?</summary>

async 함수의 목적이 "**이 함수는 비동기다, 결과를 기다려서 써라**"를 호출자에게 알리는 것이기 때문이다. 결과를 프로미스로 감싸야, 호출자가 `.then`이나 `await`으로 "준비되면 받겠다"고 예약할 수 있다.

```js
async function getNumber() {
  return 42; // Promise.resolve(42)로 감싸진다
}
console.log(getNumber());         // Promise { 42 } — 값이 아니라 프로미스
getNumber().then((v) => console.log(v)); // 42 — 이렇게 꺼낸다
```

재밌는 점은, async 함수 안에서 **프로미스를 return하면 이중으로 감싸지지 않는다.** 이미 프로미스면 그대로 그 프로미스를 따라간다.

```js
async function getData() {
  return Promise.resolve('데이터'); // 이중 포장 안 됨
}
getData().then((v) => console.log(v)); // '데이터'
```

=&gt; 그래서 async 함수는 "return한 게 값이면 프로미스로 감싸고, 이미 프로미스면 그대로 둔다". 결과적으로 호출자는 **항상 프로미스 하나**를 받는다고 믿고 `await`하면 된다. 반환 타입이 일관되니 다루기 쉽다.

</details>

<details class="orca-details">
<summary>클래스의 constructor는 왜 async로 만들 수 없을까?</summary>

**constructor는 반드시 "인스턴스(객체)"를 반환해야 하는데, async를 붙이면 "프로미스"를 반환하게 되어 규칙이 깨지기 때문**이다.

```js
class User {
  async constructor() {} // SyntaxError!
}
```

`new User()`는 User 인스턴스를 돌려줘야 하는데, async constructor는 프로미스를 돌려주려 하니 모순이다. 그래서 문법 자체가 금지돼 있다.

그럼 "생성할 때 비동기로 데이터를 받아와야 한다면" 어떻게 할까? 보통 **정적 팩토리 메서드** 패턴을 쓴다.

```js
class User {
  constructor(data) { this.data = data; } // 동기 생성자
  static async create(id) {
    const data = await promiseGet(`/users/${id}`); // 여기서 비동기
    return new User(data); // 완성된 데이터로 인스턴스 생성
  }
}
const user = await User.create(1); // 이렇게 사용
```

=&gt; "생성자는 동기로 두고, 비동기 초기화가 필요하면 `static async` 메서드로 감싸서 인스턴스를 만들어 반환"하는 게 정석이다. 생성자에서 비동기를 억지로 하려 하지 말자.

</details>



## await 키워드

- `await`는 **프로미스가 settled될 때까지 기다렸다가, 그 결과값을 반환**한다. 프로미스가 fulfilled면 결과값을, rejected면 에러를 던진다.
- `await`는 **반드시 async 함수 내부에서만** 쓸 수 있다.

```js
async function fetchData() {
  const response = await fetch('/data'); // 응답이 올 때까지 여기서 '대기'
  const data = await response.json();    // 파싱될 때까지 또 '대기'
  return data;
}
```

핵심은 `await`가 **그 줄에서 실행을 잠시 멈추고, 프로미스가 끝나면 다시 이어간다**는 것이다. 멈춰 있는 동안 자바스크립트 엔진은 다른 일을 하다가, 결과가 오면 그 지점부터 재개한다. 그래서 코드는 동기처럼 보이지만 **실제로는 블로킹 없이** 돌아간다.

=&gt; `await 프로미스`는 사실상 "그 프로미스의 `.then` 콜백 안으로 들어간 것"과 같다. 다만 들여쓰기 없이, 결과를 변수에 바로 담는 모양으로 쓰는 것뿐이다.

<details class="orca-details">
<summary>await가 "기다린다"는데, 그 동안 다른 코드도 멈추는 걸까?</summary>

**아니다. 멈추는 건 그 async 함수 "내부"의 다음 줄일 뿐, 바깥 코드나 전체 프로그램은 멈추지 않는다.** 이게 async/await가 동기 코드와 결정적으로 다른 점이다.

```js
async function run() {
  console.log('A');
  await new Promise((r) => setTimeout(r, 1000)); // 여기서 1초 '대기'
  console.log('B'); // 1초 뒤에 실행
}

run();
console.log('C'); // run이 await에서 멈춰도 '즉시' 실행됨
// 출력: A → C → (1초 뒤) B
```

`run()`이 `await`에서 멈춰도, 바깥의 `console.log('C')`는 기다리지 않고 바로 실행된다. `await`를 만나면 async 함수는 **"나머지는 결과 오면 할게"하고 제어권을 바깥에 돌려주기** 때문이다. 멈춘 나머지(`console.log('B')`)는 프로미스가 끝난 뒤 마이크로태스크로 재개된다.

=&gt; 그래서 "async 함수 안에서는 동기처럼 순차 실행, 바깥에서 보면 여전히 논블로킹 비동기"다. `await`를 "스레드를 멈추는 sleep"으로 오해하면 안 된다. 멈추는 것처럼 보이는 건 그 함수 내부의 흐름뿐이고, 엔진은 그 시간에 다른 일을 한다.

</details>



## 순차 처리와 병렬 처리

- `await`를 줄줄이 쓰면 **앞 작업이 끝나야 다음 작업을 시작**하는 순차 처리가 된다. 작업끼리 의존 관계가 있으면 맞는 방식이다.
- 하지만 **서로 독립적인 작업**을 순차로 `await`하면, 기다릴 필요 없는데 기다리느라 전체가 느려진다. 이땐 `Promise.all`로 묶어 병렬 처리한다.

```js
// 순차 처리 — 앞 결과가 다음에 필요할 때 (총 3초)
async function sequential() {
  const a = await requestA(); // 1초
  const b = await requestB(); // 1초 (a가 끝나야 시작)
  const c = await requestC(); // 1초
  return [a, b, c];
}

// 병렬 처리 — 독립적인 작업일 때 (총 1초)
async function parallel() {
  const [a, b, c] = await Promise.all([requestA(), requestB(), requestC()]);
  return [a, b, c];
}
```

=&gt; `Promise.all`은 요청 셋을 **동시에 띄워놓고** 한꺼번에 기다리므로, 대기 시간이 합이 아니라 가장 느린 것 하나로 줄어든다. "앞 결과가 다음 요청에 필요하면 순차, 독립적이면 병렬"이 기준이다.

<details class="orca-details">
<summary>await를 순차로 쓰면 왜 느려지는지 "실행이 멈추는 지점"으로 설명하면?</summary>

`await`는 그 줄에서 **프로미스가 끝날 때까지 다음 줄로 안 넘어간다.** 그래서 순차 코드는 "요청을 보내고 → 응답을 기다리고 → 그다음 요청을 보내고 → 또 기다리고"를 반복한다. 요청들이 서로 상관없어도 줄을 서는 것이다.

```js
// 순차: 요청을 '하나 끝나야 다음 하나' 보냄
const a = await requestA(); // ① A 보내고 1초 기다림
const b = await requestB(); // ② A가 끝난 뒤에야 B 보내고 1초 기다림
// → A 기다리는 1초 동안 B는 시작조차 안 됨 = 낭비
```

병렬은 순서가 다르다. `Promise.all`에 넣는 순간 **요청 셋이 거의 동시에 출발**하고, `await`는 "셋 다 올 때까지"만 한 번 기다린다.

```js
// 병렬: 셋을 '먼저 다 띄우고' 한꺼번에 기다림
const [a, b, c] = await Promise.all([requestA(), requestB(), requestC()]);
// → A, B, C가 동시에 진행되므로 총 대기는 가장 느린 것 하나
```

=&gt; 포인트는 "`await`는 그 자리에서 멈추므로, 멈추는 지점이 많을수록 대기가 쌓인다"는 것이다. 독립적인 요청은 **먼저 다 출발시킨 뒤(프로미스를 만든 뒤) 나중에 한 번만 await**해야 병렬이 된다. `Promise.all`이 바로 그 "먼저 다 출발시키기"를 해준다.

</details>



## async/await의 에러 처리

- async/await의 에러는 익숙한 `try...catch`로 **잡는다.** 프로미스의 `.catch`를 안 써도 된다.
- 콜백 패턴에선 비동기 에러를 `try...catch`로 못 잡았는데, async/await에선 **된다.** `await`가 프로미스의 rejected를 에러로 **던져주기** 때문이다.

```js
async function foo() {
  try {
    const response = await fetch('https://wrong.url'); // 네트워크 에러 가능
    const data = await response.json();
    console.log(data);
  } catch (e) {
    console.error(e); // 네트워크 에러도, 파싱 에러도, 일반 에러도 전부 여기로
  }
}
```

`try` 블록 안에서 발생한 **모든 에러**(네트워크 에러, 파싱 에러, 일반 코드 에러)를 하나의 `catch`로 잡을 수 있다. 비동기와 동기 에러를 구분 없이 한곳에서 처리하는 것이다.

만약 async 함수 안에서 에러를 `try...catch`로 잡지 않으면, **그 에러를 `reject`하는 프로미스가 반환**된다. 그래서 호출하는 쪽에서 `.catch`로 잡을 수도 있다.

```js
async function foo() {
  const response = await fetch('https://wrong.url'); // 에러 발생, 안 잡음
  return await response.json();
}
foo().catch((e) => console.error(e)); // 호출자가 .catch로 잡음
```

=&gt; 그래서 "비동기 에러는 try...catch로 못 잡는다"는 콜백 시절의 한계가 async/await에선 사라진다. 동기 코드와 똑같이 `try...catch`로 감싸면 된다. 이게 async/await가 주는 가장 큰 실용적 이점 중 하나다.

<details class="orca-details">
<summary>Promise.all을 await할 때 하나가 실패하면 try catch로 어떻게 잡힐까?</summary>

`Promise.all`은 **하나라도 rejected되면 즉시 전체가 rejected**되므로, 그 거부가 `await` 지점에서 에러로 던져져 `catch` 하나로 잡힌다.

```js
async function run() {
  try {
    const [a, b, c] = await Promise.all([ok(), fail(), ok()]); // fail()이 터지면
    console.log(a, b, c); // 실행 안 됨
  } catch (e) {
    console.error('하나라도 실패:', e); // 여기로
  }
}
```

문제는 이것이다. `Promise.all`은 fail-fast라 **하나가 실패하면 나머지 성공한 결과도 받지 못한다.** "일부 실패해도 성공한 건 쓰고 싶다"면 `Promise.allSettled`를 써야 한다.

```js
const results = await Promise.allSettled([ok(), fail(), ok()]);
// 각각 { status, value } 또는 { status, reason } — 전부 받을 수 있음
const succeeded = results.filter((r) => r.status === 'fulfilled');
```

=&gt; "전부 성공해야 하면 `Promise.all` + try/catch, 부분 성공을 허용하면 `Promise.allSettled`로 하나하나 걸러내기". await와 에러 처리를 조합할 때 이 둘의 차이를 기억해두면 좋다.

</details>

<details class="orca-details">
<summary>await를 try 밖에서 쓰거나, 에러를 아예 안 잡으면 어떻게 될까?</summary>

async 함수 안에서 발생한 에러를 **아무 데서도 안 잡으면**, 그 함수가 반환한 프로미스가 rejected 상태가 되어 **처리되지 않은 거부(unhandled rejection)**가 된다. 콘솔에 경고가 뜨거나, 환경에 따라 프로그램이 죽을 수 있다.

```js
async function foo() {
  throw new Error('안 잡음');
}
foo(); // Uncaught (in promise) Error: 안 잡음 — 아무도 안 잡으면 경고
```

그래서 async 함수를 호출할 땐 둘 중 하나는 해야 한다.

```js
// ① 함수 안에서 try...catch
async function foo() {
  try { /* ... */ } catch (e) { /* 처리 */ }
}

// ② 호출하는 쪽에서 .catch (또는 상위 async에서 try...catch)
foo().catch((e) => console.error(e));
```

=&gt; "async 함수의 에러는 결국 reject된 프로미스가 된다"는 걸 기억하면 된다. 그래서 프로미스처럼 어딘가에서 반드시 잡아줘야 하고, 안 잡으면 조용히 사라지는 게 아니라 unhandled rejection으로 드러난다.

</details>

# 47장. 에러 처리

## 에러 처리의 필요성

- 에러가 발생하면 **프로그램이 그 지점에서 강제 종료**된다. 그 뒤 코드는 실행되지 않는다.

```js
console.log('[Start]');
foo(); // ReferenceError: foo is not defined — 여기서 멈춤
console.log('[End]'); // 실행되지 않는다!
```

- `try...catch`로 감싸면, 에러가 나도 **프로그램이 죽지 않고** 흐름을 이어갈 수 있다.

```js
console.log('[Start]');
try {
  foo();
} catch (error) {
  console.error('[에러 발생]', error); // 에러를 잡아 처리
}
console.log('[End]'); // 정상 실행된다!
```

에러는 꼭 `throw`로만 나는 게 아니다. 예를 들어 없는 DOM 요소를 조작하려다 나는 `TypeError`가 흔하다.

```js
// querySelector는 요소가 없으면 null을 반환한다
const $button = document.querySelector('button'); // null일 수 있음
$button.classList.add('disabled'); // TypeError: $button이 null이면 터짐
```

이런 경우 옵셔널 체이닝으로 가볍게 막을 수 있다.

```js
const $button = document.querySelector('button');
$button?.classList.add('disabled'); // $button이 null이면 아무 일도 안 함
```

=&gt; 즉 에러 처리는 "혹시 모를 상황에 대비해 프로그램이 통째로 죽는 걸 막는" 안전장치다. 사용자에게 "앱이 멈췄다"가 아니라 "문제가 생겼지만 복구했다"를 보여주는 것이 목표다.

<details class="orca-details">
<summary>에러를 try catch로 &quot;안 잡고&quot; 그냥 두면 어떤 문제가 생길까?</summary>

**에러 하나가 그 아래 모든 코드를 멈추고, 심하면 앱 전체가 하얗게 죽는다.** 특히 프런트엔드에선 사용자 경험에 직격탄이다.

```js
function render() {
  const data = JSON.parse(badJson); // 여기서 터지면
  draw(data);                       // 이 아래는 아예 실행 안 됨
  bindEvents();                     // 이것도 안 됨 → 화면이 멈춤
}
```

그래서 실무에선 몇 가지 방어선을 둔다.

- **지역 방어**: 터질 수 있는 지점을 `try...catch`로 감싸, 거기만 복구하고 나머지는 살린다.
- **전역 방어**: 미처 못 잡은 에러를 위해 전역 핸들러를 둔다.

```js
window.addEventListener('error', (e) => report(e.error));          // 동기 에러
window.addEventListener('unhandledrejection', (e) => report(e.reason)); // 프로미스 에러
```

- **React 같은 프레임워크**: Error Boundary로 컴포넌트 트리의 에러를 잡아, 전체 대신 **그 부분만** 대체 UI로 바꾼다.

=&gt; 정리하면, "에러를 안 잡으면 한 곳의 작은 실패가 전체 실패로 번진다". 그래서 터질 만한 곳은 지역적으로 잡아 복구하고, 그래도 샌 에러는 전역·Error Boundary로 받아 로깅하고 사용자에겐 대체 화면을 보여주는 게 실무 패턴이다.

</details>



## try...catch...finally 문

- 에러 처리의 기본 문법이다. `try`에 실행할 코드, `catch`에 에러 발생 시 실행할 코드, `finally`에 **에러 여부와 상관없이 항상** 실행할 코드를 쓴다.

```js
try {
  foo(); // ① 실행할 코드 (에러가 날 수 있음)
} catch (err) {
  console.error(err); // ② try에서 에러가 나면 실행 (err에 에러 객체가 담김)
} finally {
  console.log('끝'); // ③ 에러가 나든 안 나든 무조건 실행
}
```

- **try**: 실행하려는 코드. 여기서 에러가 나면 즉시 catch로 점프한다.
- **catch**: try에서 에러가 났을 때만 실행된다. 매개변수로 에러 객체를 받는다.
- **finally**: 에러 발생 여부와 관계없이 **반드시 한 번** 실행된다. 주로 뒷정리(파일 닫기, 로딩 끄기 등)에 쓴다.

=&gt; `finally`가 중요한 이유는, try에서 에러가 나서 catch로 빠지든, 아니면 try 안에서 `return`을 하든 **무조건 실행이 보장**되기 때문이다. "성공해도 실패해도 꼭 해야 하는 뒷정리"는 finally에 둔다.

<details class="orca-details">
<summary>finally는 try 안에서 return을 해도 실행될까?</summary>

**실행된다.** 이게 finally의 핵심 가치다. try나 catch에서 `return`으로 함수를 빠져나가려 해도, finally는 **그 return이 실제로 반환되기 직전에** 반드시 실행된다.

```js
function test() {
  try {
    return 'try의 반환'; // 반환하려는 순간
  } finally {
    console.log('finally는 그래도 실행됨'); // 이게 먼저 실행된 뒤
  }
}
console.log(test());
// 출력: finally는 그래도 실행됨 → try의 반환
```

그래서 "리소스를 썼으면 성공이든 실패든 반드시 닫아야 하는" 상황에 완벽하다.

```js
function readData() {
  const resource = openResource();
  try {
    return resource.read(); // 성공해서 return해도
  } catch (e) {
    return null;            // 실패해서 return해도
  } finally {
    resource.close();       // 무조건 닫힌다
  }
}
```

=&gt; 주의할 점: finally 안에서 `return`을 하면 **try/catch의 return을 덮어쓴다.** 의도치 않은 동작의 원인이 되니, finally에선 보통 return을 쓰지 않고 뒷정리만 한다. "finally는 무슨 일이 있어도 실행된다"를 리소스 정리에 활용하는 게 정석이다.

</details>



## Error 객체

- `Error` 생성자로 에러 객체를 만든다. 인수로 에러 메시지를 넘긴다.

```js
const error = new Error('뭔가 잘못됨');
console.log(error.message); // '뭔가 잘못됨'
console.log(error.stack);   // 에러 발생 위치와 호출 스택 정보
```

- Error 객체는 두 가지 주요 프로퍼티를 갖는다.
  - **message**: 생성할 때 넘긴 에러 메시지.
  - **stack**: 에러가 발생한 위치와, 거기까지 어떤 함수들을 거쳐 왔는지(호출 스택) 정보. 디버깅의 핵심이다.

자바스크립트는 상황에 맞는 **빌트인 에러 타입**을 여러 개 제공한다. 전부 `Error`를 상속한다.

```js
1 @ 1;                    // SyntaxError: 문법이 잘못됨
foo();                    // ReferenceError: 선언되지 않은 식별자 참조
null.foo;                 // TypeError: 타입에 맞지 않는 동작
new Array(-1);            // RangeError: 허용 범위를 벗어난 값
decodeURIComponent('%');  // URIError: 잘못된 URI 처리
```


| 에러 타입              | 언제                             |
| ------------------ | ------------------------------ |
| **SyntaxError**    | 문법에 맞지 않는 코드                   |
| **ReferenceError** | 선언되지 않은 식별자를 참조                |
| **TypeError**      | 타입이 맞지 않는 동작 (null의 프로퍼티 접근 등) |
| **RangeError**     | 허용 범위를 벗어난 값                   |
| **URIError**       | encodeURI 등에 잘못된 인수            |


=&gt; 에러 타입을 알면 메시지만 보고도 원인을 빠르게 좁힐 수 있다. "`TypeError: Cannot read properties of null`"이면 "null인 뭔가의 프로퍼티를 읽었구나"를 바로 안다.

<details class="orca-details">
<summary>나만의 에러 타입(커스텀 에러)을 만들 수 있을까?</summary>

`**Error`를 상속한 클래스를 만들면 된다.** 도메인에 맞는 에러 타입을 정의하면, `catch`에서 어떤 종류의 에러인지 구분해 다르게 대응할 수 있다.

```js
class ValidationError extends Error {
  constructor(message, field) {
    super(message);        // Error의 message 설정
    this.name = 'ValidationError'; // 이름 지정
    this.field = field;    // 추가 정보도 담을 수 있다
  }
}

try {
  throw new ValidationError('이메일 형식이 아님', 'email');
} catch (e) {
  if (e instanceof ValidationError) {
    console.log(`검증 실패: ${e.field}`); // 타입으로 분기
  } else {
    throw e; // 내가 모르는 에러는 다시 던진다
  }
}
```

이렇게 하면 `instanceof`로 "이건 검증 에러", "이건 네트워크 에러"를 구분해 각각 다른 처리를 할 수 있다. 서버 API에서도 `NotFoundError`, `UnauthorizedError`처럼 상태를 타입으로 나누면 코드가 명확해진다.

=&gt; 포인트는 **"내가 처리할 수 있는 에러만 잡고, 모르는 에러는 다시 throw"**하는 것이다. 모든 에러를 뭉뚱그려 잡으면(`catch (e) {}`로 삼키면) 진짜 심각한 버그까지 조용히 묻혀버린다. 커스텀 에러 + instanceof 분기는 그걸 막는 좋은 습관이다.

</details>



## throw 문

- 에러를 **발생시키려면** `throw`로 에러 객체를 던져야 한다. `new Error(...)`로 **만들기만** 해서는 에러가 나지 않는다.

```js
// 만들기만 하면 아무 일도 안 일어난다
try {
  new Error('만들기만 함'); // 그냥 객체 하나 생성한 것뿐
} catch (error) {
  console.log(error); // 실행되지 않음!
}

// throw로 던져야 에러가 발생한다
try {
  throw new Error('던짐'); // 여기서 에러 발생
} catch (error) {
  console.log(error); // 실행됨: Error: 던짐
}
```

- `throw`를 만나면 그 즉시 실행이 멈추고, 던져진 에러는 가장 가까운 `catch`로 전달된다. 이걸 이용해 **"잘못된 상황이면 에러를 던져" 흐름을 끊는다.**

```js
const repeat = (n, f) => {
  // f가 함수가 아니면 더 진행할 이유가 없으니 에러를 던진다
  if (typeof f !== 'function') throw new TypeError('f는 함수여야 함');
  for (let i = 0; i < n; i++) f(i);
};

try {
  repeat(2, 1); // 1은 함수가 아님 → throw
} catch (err) {
  console.error(err); // TypeError: f는 함수여야 함
}
```

=&gt; 즉 `throw`는 "이 상황은 정상이 아니다"를 **명시적으로 선언**하는 것이다. 잘못된 인수를 받고도 그냥 진행하면 엉뚱한 곳에서 더 이상한 에러가 나는데, 입구에서 던져버리면 원인이 분명해진다. 이를 "빠른 실패(fail-fast)"라 한다.

<details class="orca-details">
<summary>그냥 에러를 던지는 것과, 에러 없이 false나 null을 반환하는 건 뭐가 다를까?</summary>

**"이게 정상적인 결과냐, 비정상적인 상황이냐"로 갈린다.** 둘 다 "잘 안 됐음"을 알리지만, 호출하는 쪽의 대응이 달라진다.

- **반환값(false·null·빈 배열)**: "정상 범위 안에서 '없음·실패'를 표현"할 때. 호출자는 `if`로 가볍게 분기한다.

```js
function findUser(id) {
  const user = users.find((u) => u.id === id);
  return user ?? null; // 못 찾은 건 '비정상'이 아니라 자연스러운 결과
}
const user = findUser(999);
if (!user) console.log('없는 유저'); // 조용히 분기
```

- **throw**: "여기서 멈춰야 할 만큼 비정상적인 상황"일 때. 호출자는 try/catch로 잡거나, 안 잡으면 프로그램이 멈춘다.

```js
function withdraw(amount) {
  if (amount > balance) throw new Error('잔액 부족'); // 진행하면 안 되는 상황
  balance -= amount;
}
```

=&gt; 기준은 "**이 실패를 호출자가 반드시 의식해서 다뤄야 하는가**"다. "검색 결과 없음"처럼 흔하고 예상 가능한 건 반환값으로, "잔액 부족"이나 "잘못된 인수"처럼 진행하면 안 되는 비정상은 throw로. 모든 걸 throw로 하면 try/catch가 남발되고, 모든 걸 반환값으로 하면 심각한 문제를 놓치기 쉽다. 둘의 균형이 좋은 에러 설계다.

</details>



## 에러의 전파

- 에러는 **호출 스택을 따라 위로(호출자 방향으로) 전파**된다. 던져진 에러는 거슬러 올라가다가 **처음 만나는** `catch`에서 잡힌다.

```js
const foo = () => {
  throw Error('foo에서 발생'); // ④ 여기서 던져진 에러가
};
const bar = () => {
  foo(); // ③ bar는 그냥 통과시키고
};
const baz = () => {
  bar(); // ② baz도 통과시키고
};

try {
  baz(); // ① 결국 여기 try의 catch에서 잡힌다
} catch (err) {
  console.error(err); // Error: foo에서 발생
}
```

- `foo`에서 던진 에러를 `foo`·`bar`·`baz` 어디서도 안 잡으면, 에러는 호출한 순서의 **역방향**으로 거슬러 올라간다. `foo → bar → baz → try`. 그러다 `try...catch`를 만나 거기서 처리된다.

=&gt; 그래서 에러는 **발생한 곳에서 꼭 잡을 필요가 없다.** 적절한 상위 지점에 `try...catch`를 하나 두면, 그 아래 여러 함수에서 난 에러를 한곳에서 받을 수 있다. 어디서 잡을지를 설계하는 게 에러 처리의 핵심이다.

<details class="orca-details">
<summary>비동기 코드에서는 에러 전파가 왜 다르게 동작할까?</summary>

**비동기 콜백은 호출 스택이 한 번 비워진 뒤 새로 실행되므로, 바깥의** 

<details class="orca-details">
<summary></summary>

`try...catch`**가 그 전파 경로에 없기 때문**이다. 동기 코드의 전파 규칙이 비동기에선 그대로 적용되지 않는다.

```js
try {
  setTimeout(() => {
    throw new Error('콜백 에러'); // 1초 뒤 실행될 땐 이미 try를 빠져나간 뒤
  }, 1000);
} catch (e) {
  console.error(e); // 못 잡는다!
}
```

`setTimeout`은 콜백을 등록만 하고 즉시 끝난다. 1초 뒤 콜백이 실행될 때 호출 스택에는 그 콜백 하나뿐이고, 그 아래에 `try`가 없다. 그래서 에러가 거슬러 올라가 봐야 잡아줄 `catch`가 없다.

해결은 두 가지다. **콜백 안에서 직접 try/catch**를 하거나, **프로미스·async/await**를 쓰는 것이다.

```js
// 프로미스: .catch로
fetch('/data').then((r) => r.json()).catch((e) => console.error(e));

// async/await: try...catch가 다시 통한다
async function run() {
  try {
    const data = await fetch('/data').then((r) => r.json());
  } catch (e) {
    console.error(e); // 비동기 에러도 여기서 잡힘
  }
}
```

=&gt; 그래서 async/await의 큰 장점이 "**비동기 에러를 다시** `try...catch`**로 잡을 수 있게** 해준다"는 것이다. await가 프로미스의 rejected를 그 자리에서 에러로 던져주므로, 동기 코드와 똑같은 전파 규칙이 복원된다. "동기 에러는 try/catch, 콜백 비동기 에러는 콜백 안에서, 프로미스 비동기 에러는 .catch 또는 async의 try/catch"로 정리하면 된다.

</details>

<details class="orca-details">
<summary></summary>

**"에러가 나면 프로그램이 죽으니, 터질 만한 곳을** `try...catch`**로 감싸 잡고(또는** `throw`**로 비정상을 명시적으로 던지고), 그 에러를 적절한 상위 지점에서 받아 복구하는 것."**

</details>

- 왜? → 작은 실패 하나가 전체를 멈추는 걸 막고, 사용자에게 "복구됨"을 보여주려고
- 어떻게? → try/catch/finally로 잡고, throw로 비정상을 던지고, Error 객체로 원인을 담는다
- 어디서 잡나? → 에러는 호출 스택을 거슬러 올라가니, 꼭 발생 지점이 아니라 "처리하기 좋은 상위 지점"에서 잡는다
- 비동기는? → 콜백은 전파가 끊기므로 프로미스 .catch나 async/await의 try/catch를 쓴다

=&gt; 에러를 "예외적인 사고"가 아니라 "코드가 당연히 다뤄야 할 흐름의 일부"로 보는 관점이 중요하다. 던지고, 전파되고, 잡히는 이 흐름을 통제할 줄 알면 안정적인 프로그램을 만들 수 있다.

</details>

