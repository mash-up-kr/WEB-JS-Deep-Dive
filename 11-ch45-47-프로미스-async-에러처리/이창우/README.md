# 모던 자바스크립트 Deep Dive 45 ~ 47장 정리

---

## 45장 프로미스

- 자바스크립트는 비동기 처리를 위한 패턴으로 **콜백 함수**를 사용한다.
- 그런데 콜백 패턴은
  1. **콜백 헬**로 인해 가독성이 나쁘고
  2. 비동기 처리 중 발생한 **에러 처리가 곤란**하며
  3. 여러 개의 비동기 처리를 한 번에 처리하는 데 한계가 있다.
- ⇒ ES6에서 **프로미스(Promise)** 도입

### 비동기 처리를 위한 콜백 패턴의 단점

#### 비동기 함수는 결과를 밖으로 꺼낼 수 없다

- 비동기 함수를 호출하면, 내부의 비동기로 동작하는 코드가 완료되지 않았더라도 **기다리지 않고 즉시 종료**된다.
- 즉 **비동기 함수 내부의 비동기로 동작하는 코드는 비동기 함수가 종료된 이후에 완료**된다. (42장 이벤트 루프 그대로)
- ⇒ 그래서 내부 비동기 코드에서 처리 결과를 **외부로 반환하거나 상위 스코프 변수에 할당하면 기대한 대로 동작하지 않는다.**

```js
let g = 0;

setTimeout(() => {
  g = 100;
  console.log('A:', g);
}, 0);

console.log('B:', g);
// B: 0
// A: 100
```

- `setTimeout`의 콜백은 태스크 큐에서 대기하다가 **전역 코드가 다 끝난 뒤에** 실행된다.
- ⇒ `console.log('B:', g)`가 실행되는 시점엔 아직 `g = 100`이 실행되기 전

서버 응답도 똑같다.

```js
let todos;

// GET 요청을 위한 비동기 함수
const get = url => {
  const xhr = new XMLHttpRequest();
  xhr.open('GET', url);
  xhr.send();

  xhr.onload = () => {
    if (xhr.status === 200) {
      // ① 서버의 응답을 상위 스코프의 변수에 할당한다
      todos = JSON.parse(xhr.response);
    } else {
      console.error(`${xhr.status} ${xhr.statusText}`);
    }
  };
};

get('https://jsonplaceholder.typicode.com/posts/1');
console.log(todos); // ② undefined
```

- `xhr.onload` 핸들러는 `load` 이벤트가 발생하면 태스크 큐에 들어갔다가, **콜 스택이 비어야** 실행된다.
- ⇒ ②의 `console.log`가 먼저 실행되고, ①은 그 뒤. 그래서 `undefined`

⇒ **비동기 함수는 비동기 처리 결과를 외부에 반환할 수 없고, 상위 스코프의 변수에 할당할 수도 없다.**
⇒ 그래서 결과에 대한 후속 처리는 **비동기 함수 내부에서** 해야 하고, 그걸 위해 **콜백 함수를 전달**하는 게 일반적이었다.

#### 콜백 헬

```js
const get = (url, successCallback, failureCallback) => {
  const xhr = new XMLHttpRequest();
  xhr.open('GET', url);
  xhr.send();

  xhr.onload = () => {
    if (xhr.status === 200) {
      successCallback(JSON.parse(xhr.response)); // 결과를 콜백에 넘긴다
    } else {
      failureCallback(xhr.status);
    }
  };
};
```

- 문제는 **비동기 처리 결과로 또 비동기 함수를 호출해야 하는 경우**
- 비동기 함수 내부의 비동기는 부모 비동기 함수의 결과가 있어야 동작이 보장된다 → 콜백 안에 콜백, 그 안에 또 콜백...

```js
get('/step1', a => {
  get(`/step2/${a}`, b => {
    get(`/step3/${b}`, c => {
      get(`/step4/${c}`, d => {
        console.log(d);
      });
    });
  });
});
```

⇒ 이렇게 중첩이 쌓이고 쌓이면 **콜백 헬** 발생! 가독성이 나쁘고 실수하기 쉽다.

#### 에러 처리의 한계

- 더 큰 문제는 **에러 처리가 어렵다**는 것
- `try...catch...finally`: `try` 블록 실행 중 에러가 나면 `catch`의 `err` 변수에 전달, `finally`는 무조건 실행

```js
try {
  setTimeout(() => {
    throw new Error('Error!');
  }, 1000);
} catch (e) {
  // 에러를 캐치하지 못한다
  console.error('캐치한 에러', e);
}
```

- `setTimeout`은 콜백을 호출 스케줄링하고 **바로 종료** → 콜 스택에서 팝
- 1초 뒤 콜백이 실행되는 시점엔 이미 `try...catch` 평가가 **끝나버렸다**
- 에러는 **호출자(caller) 방향**으로 전파되는데, 콜백을 호출한 건 `setTimeout`이 아니다 (이벤트 루프가 태스크 큐에서 꺼내서 실행)

⇒ 콜백 헬 + 에러 처리 곤란 → **프로미스 등장**

### 프로미스의 생성

- `Promise`는 ECMAScript 사양에 정의된 **표준 빌트인 객체** (호스트 객체 X)
- `Promise` 생성자 함수는 비동기 처리를 수행할 콜백 함수를 인수로 받는다. 이 콜백은 `resolve`와 `reject` 함수를 인수로 전달받는다.
  - 콜백 내부에서 비동기 처리를 수행하고, 성공하면 `resolve`, 실패하면 `reject`를 호출

```js
const promiseGet = url => {
  return new Promise((resolve, reject) => {
    const xhr = new XMLHttpRequest();
    xhr.open('GET', url);
    xhr.send();

    xhr.onload = () => {
      if (xhr.status === 200) {
        resolve(JSON.parse(xhr.response)); // 성공 → resolve
      } else {
        reject(new Error(xhr.status)); // 실패 → reject
      }
    };
  });
};

promiseGet('https://jsonplaceholder.typicode.com/posts/1');
```

⇒ 결과를 밖으로 못 꺼내던 문제를, **결과를 담은 프로미스 객체를 반환**하는 걸로 해결

#### 프로미스의 상태

| 상태 | 의미 | 상태 변경 조건 |
| --- | --- | --- |
| `pending` | 비동기 처리가 아직 수행되지 않은 상태 | 프로미스가 생성된 직후 기본 상태 |
| `fulfilled` | 비동기 처리가 수행된 상태 (**성공**) | `resolve` 함수 호출 |
| `rejected` | 비동기 처리가 수행된 상태 (**실패**) | `reject` 함수 호출 |

- `fulfilled` 또는 `rejected` 상태를 **settled** 상태라고 한다. 한번 settled 되면 **다른 상태로 변화할 수 없다.**
- 프로미스는 **비동기 처리 상태**와 **처리 결과**를 관리하는 객체

```js
const fulfilled = new Promise(resolve => resolve(1));
// Promise {<fulfilled>: 1}

const rejected = new Promise((_, reject) => reject(new Error('error occurred')));
// Promise {<rejected>: Error: error occurred}
```

### 프로미스의 후속 처리 메서드

- 프로미스의 상태가 변하면 그에 따른 후속 처리를 해야 한다 → `then`, `catch`, `finally`
- 후속 처리 메서드에 인수로 전달한 콜백 함수가 **선택적으로 호출**된다.
- 모든 후속 처리 메서드는 **언제나 프로미스를 반환**한다.

```js
// then — 두 번째 인수로 rejected 처리도 가능하지만 보통 catch를 쓴다
new Promise(resolve => resolve('fulfilled'))
  .then(v => console.log(v), e => console.error(e)); // fulfilled

// catch — rejected 상태일 때만 호출
new Promise((_, reject) => reject(new Error('rejected')))
  .catch(e => console.log(e)); // Error: rejected

// finally — 성공/실패 상관없이 한 번 호출
new Promise(resolve => resolve())
  .finally(() => console.log('finally'));
```

```js
promiseGet('https://jsonplaceholder.typicode.com/todos/1')
  .then(res => console.log(res))
  .catch(err => console.error(err))
  .finally(() => console.log('Bye!'));
```

⇒ `try...catch`로 못 잡던 비동기 에러를 **`catch`로 한 곳에서** 잡을 수 있다.

### 프로미스 체이닝

- `then`, `catch`, `finally`가 **언제나 프로미스를 반환**하니까 연속으로 호출할 수 있다 → **프로미스 체이닝**
- 콜백이 프로미스가 아닌 값을 반환해도 그 값을 **암묵적으로 resolve 해서** 프로미스로 감싸 반환

```js
const url = 'https://jsonplaceholder.typicode.com';

// 콜백 헬이었던 코드를 체이닝으로
promiseGet(`${url}/posts/1`)
  .then(({ userId }) => promiseGet(`${url}/users/${userId}`))
  .then(userInfo => console.log(userInfo))
  .catch(err => console.error(err));
```

⇒ 중첩 대신 **아래로 쭉** 이어진다. 그래도 콜백은 콜백이라 → 46장 `async/await`로 이어짐

### 마이크로태스크 큐

```js
setTimeout(() => console.log(1), 0);

Promise.resolve()
  .then(() => console.log(2))
  .then(() => console.log(3));
// 2 → 3 → 1
```

- 프로미스 후속 처리 메서드의 콜백은 태스크 큐가 아니라 **마이크로태스크 큐**에 저장된다.
- **마이크로태스크 큐가 태스크 큐보다 우선순위가 높다.**
  - 이벤트 루프는 콜 스택이 비면 **마이크로태스크 큐를 먼저 다 비우고**, 그다음 태스크 큐에서 하나를 꺼낸다.

```text
콜 스택이 비었다
   ↓
마이크로태스크 큐 (Promise.then / catch / finally, await 이후 코드) → 전부 비울 때까지
   ↓
태스크 큐 (setTimeout, setInterval, 이벤트 핸들러) → 하나 꺼내서 실행
   ↓
다시 마이크로태스크 큐 확인 ...
```

### 프로미스 문제 — 출력 순서는?

> 난이도: ① executor는 동기 → ② 체이닝 + 에러 처리 → ③ async/await

**① 문제 1 — Promise 생성자의 콜백은 언제 실행될까**

```js
console.log('A');

const promise = new Promise((resolve) => {
  console.log('B');
  resolve('C');
});

promise.then((value) => {
  console.log(value);
});

console.log('D');
```

**② 문제 2 — 체이닝 중간에 에러가 나면**

```js
console.log('start');

Promise.resolve(1)
  .then((value) => {
    console.log('A', value);
    return value + 1;
  })
  .then((value) => {
    console.log('B', value);
    throw new Error('oops');
  })
  .catch((error) => {
    console.log('C', error.message);
    return 10;
  })
  .then((value) => {
    console.log('D', value);
  });

Promise.resolve().then(() => {
  console.log('E');
});

console.log('end');
```

**③ 문제 3 — await 다음 줄은 언제 실행될까**

```js
console.log('A');

async function work() {
  console.log('B');

  await Promise.resolve();

  console.log('C');
}

work();

console.log('D');
```

## 핵심 흐름

```text
콜백 패턴의 한계  → 결과를 밖으로 못 꺼냄 → 콜백 중첩 → 콜백 헬 + try...catch로 에러 못 잡음
Promise         → 비동기 처리 상태와 결과를 관리하는 표준 빌트인 객체. 성공 resolve / 실패 reject
상태            → pending → fulfilled / rejected (settled 되면 끝)
후속 처리        → then / catch / finally. 언제나 프로미스를 반환 → 체이닝
마이크로태스크 큐 → 프로미스 콜백이 들어감. 태스크 큐보다 우선
```
