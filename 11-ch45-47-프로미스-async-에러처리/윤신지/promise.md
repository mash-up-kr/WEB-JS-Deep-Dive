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

