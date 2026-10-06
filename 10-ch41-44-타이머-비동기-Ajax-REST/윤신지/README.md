# 41장. 타이머

타이머는 "지금 당장"이 아니라 "일정 시간 뒤에" 혹은 "일정 간격마다" 함수를 실행하게 해주는 기능이다. 

=&gt; 특히 디바운스와 스로틀은 검색창·스크롤·리사이즈 같은 "폭풍처럼 쏟아지는 이벤트"를 길들이는 기술이라, 프런트엔드 면접 단골이자 실무에서 성능을 좌우한다.



## 호출 스케줄링

- 함수를 지금 호출하지 않고, **일정 시간이 지난 뒤에 호출되도록 예약**하는 것을 호출 스케줄링(scheduling a call)이라 한다.
- 이를 위해 **타이머 함수** `setTimeout`·`setInterval`(생성)과 `clearTimeout`·`clearInterval`(제거)을 쓴다.



1. 타이머 함수는 ECMAScript 표준 빌트인 함수가 **아니다.** 브라우저·Node.js 같은 실행 환경이 제공하는 **호스트 객체**다.
2. 자바스크립트 엔진은 **싱글 스레드**(한 번에 하나만 처리)로 동작한다. 그래서 타이머 함수는 **비동기(asynchronous) 방식**으로 동작한다.

<details class="orca-details" open>
<summary>싱글 스레드인데 타이머가 어떻게 &quot;동시에&quot; 돌아갈까? (이벤트 루프)</summary>

**실제로 타이머를 세는 건 자바스크립트 엔진이 아니라, 실행 환경(브라우저/Node)의 다른 부분**이기 때문이다.

`setTimeout(cb, 1000)`을 호출하면 이런 일이 벌어진다.

1. 자바스크립트 엔진은 `setTimeout`을 **브라우저(Web API)에 넘기고 바로 다음 코드로** 넘어간다. (여기서 안 기다림 → 비동기)
2. 브라우저가 1초를 센다. (엔진과 **별개로** 동작)
3. 1초가 지나면 브라우저가 콜백 `cb`를 **태스크 큐(task queue)**에 넣는다.
4. **이벤트 루프**가 "콜 스택이 비었나?"를 계속 확인하다가, 비면 태스크 큐의 `cb`를 꺼내 실행한다.

```js
console.log('1');
setTimeout(() => console.log('2'), 0); // 0ms여도!
console.log('3');
// 출력: 1 → 3 → 2
```

`0ms`를 줬는데도 `2`가 맨 나중에 찍힌다. 콜백은 아무리 빨라도 **지금 실행 중인 코드(1, 3)가 다 끝나고 콜 스택이 빈 뒤**에야 큐에서 실행되기 때문이다.

=&gt; 그래서 타이머의 지연 시간은 "정확히 그 시간 뒤"가 아니라 "**최소한 그 시간은 지난 뒤, 콜 스택이 비면**"이라는 뜻이다. 싱글 스레드가 멈추지 않고 돌아가는 비결이 이 이벤트 루프 구조다. (자세한 건 42장 비동기 프로그래밍에서.)

</details>



## 타이머 함수

### setTimeout / clearTimeout

- `setTimeout(콜백, 지연시간, 인수...)`: 지연시간(ms) 뒤에 콜백을 **단 한 번** 호출한다. 세 번째 이후 인수는 콜백에 전달된다.
- 반환값은 타이머를 식별하는 **타이머 id**다. 이 id를 `clearTimeout`에 넘기면 아직 실행 전인 타이머를 **취소**할 수 있다.

```js
// 1초 후 한 번 실행
const timerId = setTimeout(() => console.log('Hi!'), 1000);

// 콜백에 인수 전달 (세 번째부터)
setTimeout((name) => console.log(`Hi! ${name}`), 1000, 'Lee');

// 아직 실행 안 된 타이머 취소
clearTimeout(timerId); // 'Hi!'가 출력되지 않음
```

지연시간을 생략하면 기본값은 `0`이다. 단, 앞의 토글에서 봤듯 `0`이어도 즉시 실행이 아니라 "콜 스택이 빈 다음"에 실행된다.

<details class="orca-details">
<summary>setTimeout(fn, 10)인데 10ms보다 더 늦게 실행될 때가 있다? (최소 지연·클램핑)</summary>

브라우저가 지연시간을 **"정확히 그 시간"이 아니라 "최소한 그 시간"**으로만 보장하기 때문이다. 게다가 HTML 명세에는 지연시간을 강제로 끌어올리는 **클램핑(clamping)** 규칙이 있다.

대표적인 세 가지 상황에서 우리가 준 값보다 실제 지연이 늘어난다.

1. **중첩 타이머 4ms 클램핑**: `setTimeout`이 5단계 이상 중첩되면, 그 아래부터는 지연시간이 `0`이어도 **최소 4ms로 강제**된다. 명세에 박혀 있는 규칙이다.
2. **비활성 탭 throttling**: 백그라운드 탭(다른 탭을 보고 있을 때)에서는 타이머가 **최소 1000ms(1초)로 느려진다.** 배터리·CPU 절약을 위해서다.
3. **콜 스택 지연**: 42장에서 봤듯, 타이머가 만료돼 콜백이 태스크 큐에 들어가도 **콜 스택이 비어야** 실행된다. 앞에 무거운 동기 코드가 돌고 있으면 그만큼 더 밀린다.

```js
let last = Date.now();
function tick(count) {
  const now = Date.now();
  console.log(count, now - last, 'ms'); // 5번째부터 ~4ms로 고정됨
  last = now;
  if (count < 7) setTimeout(() => tick(count + 1), 0);
}
setTimeout(() => tick(1), 0);
```

=&gt; 그래서 타이머는 **정밀한 시계가 아니다.** "정확히 N초 뒤"가 필요한 애니메이션은 `requestAnimationFrame`(프레임 동기)이나, 경과 시간을 매번 `Date.now()`로 직접 계산하는 방식을 써야 한다. 타이머의 지연시간은 "하한선"이라고 기억하자.

</details>

<details class="orca-details">
<summary>clearTimeout에 넘기는 그 값(타이머 id)의 정체는 뭘까?</summary>

타이머를 생성하면 돌려주는 id는, 브라우저가 내부적으로 관리하는 **"활성 타이머 목록"에서 이 타이머를 가리키는 식별자**다. 환경에 따라 정체가 다르다.

- **브라우저**: 0보다 큰 **정수**. 브라우저가 타이머마다 번호를 매겨 돌려준다. `clearTimeout(id)`는 "그 번호의 타이머를 목록에서 지워라"는 뜻이다.
- **Node.js**: 숫자가 아니라 `Timeout` 객체를 돌려준다. 그래서 Node에서 id를 `console.log`하면 객체가 찍힌다.

```js
const id = setTimeout(() => {}, 1000);
console.log(id);            // 브라우저: 숫자(예: 3) / Node: Timeout { ... }
clearTimeout(id);          // 목록에서 제거 → 콜백이 태스크 큐에 못 들어감
```

중요한 건, `clearTimeout`은 **아직 콜백이 태스크 큐에 들어가기 전(또는 대기 중)**이어야 취소가 된다는 점이다. 이미 콜백이 콜 스택에서 실행되기 시작했다면 취소할 수 없다.

=&gt; 그리고 `setTimeout`과 `setInterval`의 id는 **같은 풀(pool)에서 관리**돼서, `clearInterval`에 `setTimeout` id를 넘겨도 동작한다(권장하진 않는다). "id는 그저 타이머 목록의 열쇠"라고 이해하면 된다.

</details>

### setInterval / clearInterval

- `setInterval(콜백, 간격시간, 인수...)`: 간격시간(ms)마다 콜백을 **반복** 호출한다. 그 외는 `setTimeout`과 같다.
- `clearInterval(타이머id)`로 멈추지 않으면 **영원히 반복**하니 주의.

```js
let count = 1;
const timerId = setInterval(() => {
  console.log(count); // 1 2 3 4 5
  if (count++ === 5) clearInterval(timerId); // 5가 되면 중지
}, 1000);
```

<details class="orca-details">
<summary>setInterval 대신 setTimeout을 재귀로 쓰는 게 더 안전하다던데?</summary>

**정확한 간격과 실행 누적 문제** 때문에, 상황에 따라 `setTimeout` 재귀가 더 낫다.

`setInterval`은 "콜백 실행 시간과 무관하게" 정해진 간격마다 큐에 콜백을 넣는다. 그런데 콜백이 간격보다 오래 걸리거나 콜 스택이 밀리면, 콜백들이 **큐에 쌓였다가 한꺼번에 몰려 실행**될 수 있다.

`setTimeout`을 재귀로 쓰면 "**콜백이 끝난 뒤**에 다음 타이머를 건다"는 보장이 생긴다.

```js
// setInterval: 간격이 보장 안 될 수 있음 (콜백 누적 위험)
setInterval(task, 1000);

// setTimeout 재귀: 이전 작업이 끝난 뒤 다음 예약 (간격 안정적)
function run() {
  task();
  setTimeout(run, 1000); // task가 끝나고 나서 1초 뒤 재예약
}
setTimeout(run, 1000);
```

=&gt; "단순 반복은 `setInterval`, 콜백이 무겁거나 정확한 간격·순서가 중요하면 `setTimeout` 재귀"가 기준이다. 애니메이션이라면 아예 `requestAnimationFrame`을 쓰는 게 더 적합하다(아래 토글).

</details>

<details class="orca-details">
<summary>애니메이션엔 왜 setInterval 말고 requestAnimationFrame을 쓸까?</summary>

**브라우저의 화면 갱신 주기(보통 60fps)에 딱 맞춰 실행**되기 때문이다.

`setInterval(fn, 16)`으로 애니메이션을 만들면 몇 가지 문제가 있다.

- 화면 주사율과 어긋나 **프레임이 끊기거나(jank)** 어색해진다.
- **백그라운드 탭**에서도 계속 돌아 배터리·CPU를 낭비한다.

`requestAnimationFrame(fn)`은 브라우저가 "다음 화면을 그리기 직전"에 콜백을 호출한다. 그래서 주사율에 맞춰 부드럽고, 탭이 비활성화되면 **자동으로 멈춘다.**

```js
function animate() {
  // 위치 갱신 등...
  requestAnimationFrame(animate); // 다음 프레임에 재예약
}
requestAnimationFrame(animate);
```

=&gt; "시간 기반 로직(몇 초 뒤, 몇 초마다)은 타이머, 화면을 매 프레임 그리는 애니메이션은 `requestAnimationFrame`"으로 나누면 된다. 책 범위를 넘지만 실무에선 꼭 알아야 할 짝이다.

</details>



## 디바운스와 스로틀

- `scroll`, `resize`, `input`, `mousemove` 같은 이벤트는 **짧은 시간에 아주 많이, 연속으로** 발생한다. 그때마다 핸들러를 실행하면 성능이 급격히 나빠진다.
- **디바운스(debounce)**와 **스로틀(throttle)**은 이렇게 쏟아지는 이벤트를 **타이머로 묶어** 핸들러 호출 횟수를 확 줄이는 기법이다. 목적은 같지만 "언제 실행하느냐"가 다르다.

=&gt; **디바운스는 "다 끝나면 한 번", 스로틀은 "일정 간격으로 한 번씩".**

### 디바운스

- 연속해서 이벤트가 발생하면 **매번 이전 타이머를 취소하고 새 타이머를 건다.** 그래서 이벤트가 **멈추고 일정 시간이 지나야** 비로소 콜백이 **딱 한 번** 실행된다.
- "마지막 이벤트 기준"으로 동작한다고 생각하면 쉽다.

```js
function debounce(callback, delay) {
  let timerId;
  return (...args) => {
    // 이벤트가 또 오면, 직전에 걸어둔 타이머를 취소하고
    if (timerId) clearTimeout(timerId);
    // 새로 타이머를 건다 → 결국 '마지막' 타이머만 살아남아 실행됨
    timerId = setTimeout(() => callback(...args), delay);
  };
}

// 검색창: 입력이 멈춘 뒤 0.3초 후에만 요청
const onSearch = debounce((e) => {
  console.log('API 요청:', e.target.value);
}, 300);

input.addEventListener('input', onSearch);
```

**디바운스가 어울리는 경우**

- 검색창 자동완성 (타이핑이 멈춘 뒤 요청)
- `resize` 이벤트 처리 (창 조절이 끝난 뒤 한 번 계산)
- 폼 유효성 검사, 버튼 중복 클릭(연타) 방지

### 스로틀

- 이벤트가 아무리 쏟아져도 **일정 시간 간격마다 최대 한 번씩만** 콜백을 실행한다. 타이머가 돌고 있는 동안 들어온 이벤트는 무시한다.
- "일정 주기로 솎아낸다"고 생각하면 쉽다.

```js
function throttle(callback, delay) {
  let timerId;
  return (...args) => {
    // 타이머가 이미 돌고 있으면 무시 (이 구간의 이벤트는 버림)
    if (timerId) return;
    timerId = setTimeout(() => {
      callback(...args);
      timerId = null; // delay 뒤에 비워서 다음 실행을 허용
    }, delay);
  };
}

// 스크롤: 0.3초에 한 번씩만 위치 체크
const onScroll = throttle(() => {
  console.log('scroll 위치 체크');
}, 300);

window.addEventListener('scroll', onScroll);
```

**스로틀이 어울리는 경우**

- 스크롤 위치에 따른 처리(무한 스크롤, 스크롤 진행률 표시)
- `mousemove`로 마우스 추적
- 짧은 간격으로 계속 갱신돼야 하는 작업

<details class="orca-details">
<summary>스로틀을 &quot;타이머&quot; 말고 &quot;타임스탬프&quot;로 구현하면 뭐가 다를까?</summary>

스로틀 구현에는 크게 두 방식이 있고, **"맨 앞(leading)에서 즉시 실행되느냐"**가 갈린다.

**① 타임스탬프 방식 — leading(앞) 실행**

마지막 실행 시각을 기억해두고, "지금 시각 − 마지막 실행 시각"이 `delay`를 넘으면 바로 실행한다.

```js
function throttle(callback, delay) {
  let last = 0;
  return (...args) => {
    const now = Date.now();
    if (now - last >= delay) { // 간격이 지났으면
      last = now;
      callback(...args);       // 즉시 실행 (맨 처음에도 바로 실행됨)
    }
  };
}
```

→ 첫 이벤트에 **즉시 한 번** 반응한다. 대신 마지막 이벤트는 실행이 보장되지 않는다(trailing 없음).

**② 타이머 방식 — trailing(뒤) 실행**

앞 절의 구현이 이 방식이다. 타이머가 없을 때만 `setTimeout`을 걸고, `delay` 뒤에 실행한다.

```js
// 타이머가 돌고 있으면 무시, delay 뒤에 실행 → 첫 반응이 delay만큼 늦음
if (timerId) return;
timerId = setTimeout(() => { callback(...args); timerId = null; }, delay);
```

→ 첫 반응이 `delay`만큼 **늦지만**, 간격의 끝에서 실행이 보장된다.

=&gt; Lodash의 `throttle`은 이 둘을 합쳐 **leading과 trailing을 모두 기본 실행**(옵션으로 끌 수 있음)한다. 타임스탬프로 "맨 앞"을 잡고, 타이머로 "맨 뒤"를 잡는 식이다. "첫 반응이 즉시 필요하면 타임스탬프, 마지막 상태가 중요하면 타이머, 둘 다면 Lodash"로 이해하면 된다.

</details>

<details class="orca-details">
<summary>디바운스랑 스로틀, 결국 뭐가 다른 걸까?</summary>

**"마지막 한 번(디바운스)" vs "주기적으로 꾸준히(스로틀)"**가 핵심 차이다.

엘리베이터로 비유하면 —

- **디바운스**: 사람이 탈 때마다 문이 다시 닫히길 기다린다. **아무도 안 탈 때까지 기다렸다가** 출발. (마지막 이벤트 기준 한 번)
- **스로틀**: **일정 시간마다** 무조건 출발. 그 사이 몇 명이 타든 상관없이 주기적으로 운행. (일정 간격마다 한 번)


|        | 디바운스              | 스로틀             |
| ------ | ----------------- | --------------- |
| 실행 시점  | 이벤트가 **멈춘 뒤** 한 번 | 일정 **간격마다** 한 번 |
| 중간 이벤트 | 전부 무시(타이머 리셋)     | 간격 밖의 것만 실행     |
| 비유     | "입력 끝나면 검색"       | "0.3초마다 위치 체크"  |
| 대표 용도  | 검색어 입력, resize    | 스크롤, mousemove  |


=&gt; 판단 기준: **"마지막 상태만 중요"**하면 디바운스(검색어 최종값), **"진행 중에도 주기적 반응이 필요"**하면 스로틀(스크롤 위치). 같은 이벤트라도 뭘 원하느냐에 따라 선택이 갈린다.

</details>

<details class="orca-details">
<summary>실무에선 직접 구현할까, 라이브러리를 쓸까?</summary>

**실무에선 대부분 Lodash의 `_.debounce`·`_.throttle`을 쓴다.** 직접 구현한 버전은 교육용으로는 좋지만, 실제로 챙겨야 할 엣지 케이스가 많기 때문이다.

Lodash 버전이 추가로 다루는 것들:

- **leading / trailing 옵션**: "맨 앞에서도 한 번 실행할지, 맨 끝에서만 할지" 조절
- **maxWait**: 디바운스인데도 "최소 이만큼마다는 한 번 실행" 보장
- **cancel / flush**: 대기 중인 호출을 취소하거나 즉시 실행
- `this` 바인딩, 반환값 전달 등 세부 처리

```js
import { debounce, throttle } from 'lodash-es';

const onSearch = debounce(fn, 300);
const onScroll = throttle(fn, 300, { leading: true, trailing: false });
```

=&gt; "원리는 직접 구현으로 이해하되, 프로덕션에선 검증된 Lodash(또는 프레임워크 훅)를 쓴다"가 일반적이다. 면접에선 직접 구현을 물어보니 둘 다 알아두면 좋다.

</details>

<details class="orca-details">
<summary>React에서 디바운스를 쓰려면 왜 useRef·useCallback이 필요할까?</summary>

**리렌더링 때마다 타이머 id와 디바운스 함수가 새로 만들어지면 디바운스가 동작하지 않기 때문**이다.

함수 컴포넌트는 렌더링될 때마다 내부 코드가 다시 실행된다. 그래서 `debounce(fn, 300)`을 컴포넌트 본문에서 그냥 호출하면, **렌더링마다 새로운 디바운스 함수**가 생겨 `timerId`가 초기화된다. 결국 "이전 타이머 취소"가 안 돼서 디바운스가 깨진다.

해결은 두 가지다.

- `useRef` : 타이머 id를 렌더링 사이에도 유지되는 곳에 보관.
- `useCallback`(또는 `useMemo`) : 디바운스 함수 자체를 한 번만 만들어 재사용.

```js
function useDebounce(callback, delay) {
  const timerRef = useRef(null);
  return useCallback((...args) => {
    if (timerRef.current) clearTimeout(timerRef.current);
    timerRef.current = setTimeout(() => callback(...args), delay);
  }, [callback, delay]);
}
```

=&gt; 이건 24장 클로저와도 이어진다. 디바운스가 `timerId`를 **클로저로 붙잡아** 상태를 유지하는 구조인데, React에서는 그 "유지되는 저장소"를 `useRef`로 대신 마련해줘야 하는 것이다.

</details>



&nbsp;


# 42장. 비동기 프로그래밍

## 동기 처리와 비동기 처리

- **동기(synchronous) 처리**: 지금 실행 중인 작업이 **끝날 때까지** 다음 작업이 **대기**한다. 실행 순서는 보장되지만, 앞 작업이 오래 걸리면 뒤가 전부 멈춘다(블로킹).
- **비동기(asynchronous) 처리**: 지금 작업이 안 끝났어도 **다음 작업을 바로 실행**한다. 블로킹이 없지만, 실행 순서가 보장되지 않는다.

같은 구조의 코드를 동기/비동기로 각각 보면 차이가 선명하다.

```js
// 동기: sleep이 3초를 다 잡아먹고 나서야 bar 실행 (블로킹)
function sleep(func, delay) {
  const delayUntil = Date.now() + delay;
  while (Date.now() < delayUntil); // 3초간 CPU를 붙잡고 아무것도 못 함
  func();
}
sleep(() => console.log('foo'), 3000);
console.log('bar');
// 출력: (3초 뒤) foo → bar  ← bar가 3초나 기다림!
```

```js
// 비동기: setTimeout은 bar를 막지 않는다
setTimeout(() => console.log('foo'), 3000);
console.log('bar');
// 출력: bar → (3초 뒤) foo  ← bar가 즉시 실행됨
```

=&gt; 대표적인 비동기 처리 방식의 함수가 **타이머(`setTimeout`·`setInterval`), HTTP 요청, 이벤트 핸들러**다. 이들은 "오래 걸릴 수 있는 일"이라 메인 흐름을 막지 않도록 비동기로 동작한다.

<details class="orca-details">
<summary>블로킹이 왜 그렇게 치명적일까? 싱글 스레드라서?</summary>

**자바스크립트 엔진의 콜 스택이 하나뿐이라, 하나가 막히면 "모든 것"이 멈추기 때문**이다.

위 동기 예제의 `while` 루프가 도는 3초 동안, 브라우저는 그 어떤 것도 못 한다. 버튼 클릭도, 스크롤도, 애니메이션도, 심지어 화면 렌더링까지 전부 멈춘다. 사용자 눈에는 "페이지가 얼어붙은" 것처럼 보인다.

```js
// 이런 코드를 실행하면 3초간 페이지 전체가 먹통이 된다
const end = Date.now() + 3000;
while (Date.now() < end); // 클릭도, 스크롤도 안 먹힘
```

=&gt; 그래서 자바스크립트는 오래 걸리는 작업(타이머·네트워크)을 **절대 동기로 처리하지 않고** 비동기로 넘긴다. 싱글 스레드 언어에서 블로킹은 곧 "앱 전체 정지"라서, 비동기가 선택이 아니라 필수인 것이다.

</details>



## 이벤트 루프와 태스크 큐

**자바스크립트 엔진** (싱글 스레드, 딱 두 개의 공간)

- **콜 스택(call stack)**: 실행 컨텍스트가 쌓이는 곳. 함수가 호출되면 푸시, 끝나면 팝. **단 하나뿐**이라 한 번에 하나만 실행한다.
- **힙(heap)**: 객체가 저장되는 메모리 공간. 콜 스택의 실행 컨텍스트가 여기 객체들을 참조한다.

**브라우저(또는 Node.js)** (멀티 스레드)

- **Web API**: 타이머·HTTP 요청·DOM 이벤트 등 "오래 걸리는 일"을 엔진 대신 처리한다.
- **태스크 큐(task queue)**: Web API가 처리를 끝낸 **콜백 함수가 줄 서서 대기**하는 곳(선입선출 FIFO).
- **이벤트 루프(event loop)**: **"콜 스택이 비었는지"**를 끊임없이 확인하다가, 비면 태스크 큐에서 콜백을 꺼내 콜 스택으로 옮긴다.

=&gt; 즉, 자바스크립트 엔진은 소스코드의 **평가와 실행만** 담당하고, 비동기 처리의 나머지(타이머 세기, 네트워크 통신, 콜백을 큐에 넣기)는 전부 **브라우저**가 담당한다. 엔진은 싱글 스레드지만, 그걸 감싼 브라우저는 멀티 스레드라 이 분업이 가능하다.

<details class="orca-details">
<summary>&quot;Maximum call stack size exceeded&quot; 에러가 콜 스택이랑 무슨 관계일까?</summary>

콜 스택은 **크기가 유한**하다. 함수가 끝나지 않고 계속 쌓이기만 하면 스택이 꽉 차서 터지는데, 그게 바로 그 에러다.

```js
function recurse() {
  return recurse(); // 자기를 다시 호출 → 끝나지 않고 계속 쌓임
}
recurse(); // RangeError: Maximum call stack size exceeded
```

`recurse`가 호출될 때마다 실행 컨텍스트가 콜 스택에 푸시되는데, 반환(팝)이 없으니 수만 개가 쌓이다 한계(엔진마다 다르지만 보통 1만~수만)를 넘겨 터진다.

재밌는 건, **같은 재귀를 비동기로 쪼개면 안 터진다**는 점이다.

```js
function recurse() {
  setTimeout(recurse, 0); // 다음 호출을 '태스크 큐'로 미룸
}
recurse(); // 안 터짐! (무한히 돌지만 스택은 매번 비워짐)
```

`setTimeout`으로 미루면 `recurse`가 **콜 스택에서 완전히 빠져나간 뒤**에 다음 `recurse`가 큐에서 꺼내져 실행된다. 스택이 매번 비워지니 쌓이지 않는다.

=&gt; 그래서 아주 깊은 재귀나 대용량 반복 처리는 "비동기로 쪼개서(스택을 비워가며)" 돌리면 스택 오버플로우를 피할 수 있다. 콜 스택이 유한하다는 사실, 그리고 비동기가 그 스택을 비워준다는 원리가 여기서 실용적으로 쓰인다.

</details>

### setTimeout은 어떻게 동작하나 (단계별 추적)

```js
function foo() { console.log('foo'); }
function bar() { console.log('bar'); }

setTimeout(foo, 0); // 0ms인데도...
bar();
// 출력: bar → foo
```

`0ms`를 줬는데도 `bar`가 먼저 찍히는 이유를 단계로 쫓아보자.

1. 전역 코드가 콜 스택에 올라가 실행된다.
2. `setTimeout`이 호출되어 콜 스택에 푸시 → 실행된다.
3. `setTimeout`은 **콜백(foo)과 타이머 설정을 브라우저(Web API)에 넘기고 즉시 끝난다.** 콜 스택에서 팝.
4. 이제 **병행**으로 진행된다.
  - **브라우저**: 타이머(0ms, 실제론 최소 4ms)를 센 뒤, 콜백 `foo`를 **태스크 큐에 넣는다.**
  - **엔진**: 기다리지 않고 바로 `bar()`를 실행한다 → `'bar'` 출력 → 팝.
5. 전역 코드까지 다 끝나 **콜 스택이 완전히 빈다.**
6. **이벤트 루프**가 "콜 스택이 비었네!"를 감지하고, 태스크 큐에서 `foo`를 꺼내 콜 스택에 올린다 → `'foo'` 출력.

=&gt; 그래서 `foo`는 "0ms 뒤"가 아니라 "**콜 스택이 다 비워진 뒤**"에 실행된다. 비동기 콜백은 아무리 빨라도 **지금 실행 중인 동기 코드가 전부 끝나야** 비로소 실행 기회를 얻는다.

<details class="orca-details">
<summary>setTimeout(fn, 0)이 &quot;즉시&quot; 실행이 아니라는 게 핵심인데, 실무에서 이걸 어디에 쓸까?</summary>

**"지금 실행 중인 동기 코드가 다 끝난 직후로 실행을 미루고 싶을 때"** 쓴다. 지연이 목적이 아니라 **순서 미루기**가 목적이다.

대표적인 활용:

- **무거운 작업 쪼개기**: 긴 반복 작업을 `setTimeout(…, 0)`으로 잘게 나눠, 중간에 브라우저가 렌더링·클릭 처리를 할 틈을 준다(UI 안 멈춤).
- **DOM 반영 이후로 미루기**: DOM을 바꾼 직후 그 결과를 읽어야 할 때, 렌더링이 반영된 뒤로 코드를 미룬다.

```js
console.log('1');
setTimeout(() => console.log('2'), 0); // 지금 코드 다 끝난 뒤로 미룸
console.log('3');
// 1 → 3 → 2
```

=&gt; "0ms = 당장"이 아니라 "0ms = **콜 스택 비워진 다음 맨 처음**"이라는 의미. 이 특성을 알면 "왜 내 setTimeout(0)이 바로 안 돌지?"가 아니라 "일부러 뒤로 미루는 도구"로 쓸 수 있다.

</details>

<details class="orca-details">
<summary>&quot;자바스크립트가 싱글 스레드다&quot;랑 &quot;비동기로 동시에 처리한다&quot;는 모순 아닐까?</summary>

모순처럼 보이지만, **"무엇이" 싱글 스레드인지**를 구분하면 풀린다.

- **싱글 스레드인 것**: 자바스크립트 **엔진**(콜 스택). 우리가 짠 JS 코드는 한 줄씩, 하나씩만 실행된다. 이건 변함없는 사실이다.
- **멀티 스레드인 것**: 엔진을 감싼 **브라우저**. 타이머 세기, 네트워크 요청, 파일 읽기 등은 브라우저가 **별도 스레드에서 병행** 처리한다.

즉 "동시에 처리하는 것처럼 보이는" 일(타이머가 도는 동안 클릭도 먹히는 것)은 **엔진이 하는 게 아니라 브라우저가 대신** 해주는 것이다. 엔진은 그 결과(콜백)를 나중에 큐에서 하나씩 받아 실행할 뿐이다.

```
[엔진: 싱글 스레드]  ←→  [브라우저: 멀티 스레드]
   콜 스택 하나              타이머·네트워크 병행 처리
```

=&gt; 정리: **"자바스크립트(엔진)는 싱글 스레드가 맞다. 다만 비동기 처리는 엔진이 아니라 브라우저가 병행으로 해주고, 이벤트 루프가 그 둘을 이어준다."** 이 한 문장이 면접 답변의 핵심이다.

</details>

### 마이크로태스크 큐

- 태스크 큐 말고 **마이크로태스크 큐(microtask queue)**가 하나 더 있다. Promise의 후속 처리 메서드(`.then`·`.catch`·`.finally`)의 콜백이 여기에 담긴다.
- 핵심은 **마이크로태스크 큐가 태스크 큐보다 우선순위가 높다**는 것. 이벤트 루프는 콜 스택이 비면 **마이크로태스크 큐를 먼저 완전히 비운 뒤**에야 태스크 큐를 본다.

```js
console.log('1'); // 동기

setTimeout(() => console.log('2'), 0);        // 태스크 큐
Promise.resolve().then(() => console.log('3')); // 마이크로태스크 큐

console.log('4'); // 동기
// 출력: 1 → 4 → 3 → 2
```

`2`(setTimeout)와 `3`(Promise)이 둘 다 "나중 실행"이지만, **마이크로태스크인 `3`이 태스크인 `2`보다 먼저** 나온다.

=&gt; 실행 우선순위를 정리하면: **① 지금 실행 중인 동기 코드 → ② 마이크로태스크 큐(Promise 등) 전부 → ③ 태스크 큐(setTimeout 등) 하나.** 

<details class="orca-details">
<summary>왜 Promise(마이크로태스크)를 setTimeout(태스크)보다 먼저 처리할까?</summary>

**Promise 기반 비동기의 응답성을 높이고, 작업의 "연속성"을 보장하기 위해서**다.

마이크로태스크는 "지금 하던 일의 **직후 마무리**"에 가까운 성격이다. 예를 들어 `promise.then().then()`처럼 연결된 작업은 중간에 다른 작업(렌더링, 타이머)이 끼어들지 않고 **한 묶음으로** 처리되는 게 자연스럽다. 그래서 브라우저는 "콜 스택이 빌 때마다 마이크로태스크 큐를 **싹 비운 다음** 태스크 큐 하나를 처리"하는 규칙을 둔다.

```js
Promise.resolve()
  .then(() => console.log('A'))
  .then(() => console.log('B')); // A 다음에 바로, 끊김 없이 B
setTimeout(() => console.log('C'), 0);
// A → B → C  (Promise 체인이 setTimeout보다 먼저 다 끝남)
```

주의할 점도 있다. 마이크로태스크가 끝없이 새 마이크로태스크를 만들면, 태스크 큐(와 렌더링)가 영영 처리되지 못해 화면이 멈출 수 있다.

=&gt; "마이크로태스크 = 급한 마무리(Promise), 태스크 = 다음 차례의 일(타이머·이벤트)". 이 우선순위 때문에 `async/await`로 쓴 코드의 실행 순서가 `setTimeout`과 다르게 나오는 것이다.

</details>

<details class="orca-details">
<summary>렌더링은 이 큐들 사이 어디에서 일어날까?</summary>

대략 **"마이크로태스크를 다 비운 뒤, 다음 태스크로 넘어가기 전"** 타이밍에 브라우저가 렌더링(리플로우·리페인트, 38장) 기회를 갖는다.

이벤트 루프 한 바퀴를 거칠게 그리면 이렇다.

```
① 태스크 하나 실행 (예: 이벤트 핸들러)
② 마이크로태스크 큐 전부 비우기 (Promise 콜백들)
③ 필요하면 렌더링 (스타일 계산 → 레이아웃 → 페인트)
④ 다시 ①로
```

그래서 **동기 코드로 긴 루프를 돌리거나 마이크로태스크를 폭주시키면**, ③ 렌더링 차례가 안 와서 화면이 멈춘다. 반대로 무거운 작업을 `setTimeout`(태스크)으로 쪼개면, 그 사이사이에 ③ 렌더링이 끼어들 수 있어 UI가 살아있다.

=&gt; 41장에서 "애니메이션은 `requestAnimationFrame`"이라 한 것도 이 흐름과 연결된다. `requestAnimationFrame` 콜백은 ③ 렌더링 직전에 실행되도록 특별히 예약되기 때문이다. "왜 UI가 끊기는가"의 답이 전부 이 이벤트 루프 한 바퀴 안에 있다.

</details>

<details class="orca-details">
<summary>태스크 큐(매크로태스크)에는 setTimeout 말고 또 뭐가 들어갈까?</summary>

우리가 흔히 "태스크 큐"라 부르는 건 정확히는 **매크로태스크 큐(macrotask queue)**다. 여기엔 여러 종류의 비동기 작업이 들어간다.

- `setTimeout`·`setInterval` 콜백
- **DOM 이벤트 핸들러** (클릭, 스크롤 등)
- **네트워크 요청 완료** 콜백 (XHR의 `onload` 등)
- `MessageChannel`·`postMessage`
- (렌더링 관련) UI 이벤트

반대로 **마이크로태스크 큐**에 들어가는 것은 더 좁다.

- Promise의 `.then`·`.catch`·`.finally` 콜백
- `async/await`의 `await` 뒤 코드
- `queueMicrotask(fn)` — 직접 마이크로태스크를 예약하는 API
- `MutationObserver` 콜백

```js
// 직접 마이크로태스크를 큐에 넣을 수 있다
queueMicrotask(() => console.log('micro'));
setTimeout(() => console.log('macro'), 0);
console.log('sync');
// sync → micro → macro
```

=&gt; 핵심 규칙은 변함없다. **"매크로태스크를 하나 꺼내 실행 → 그 사이 쌓인 마이크로태스크를 전부 비움 → (필요시 렌더링) → 다음 매크로태스크 하나"**. 그래서 매크로태스크 사이사이에 마이크로태스크가 "싹" 처리된다. 어떤 API가 어느 큐를 쓰는지 알면 실행 순서를 정확히 예측할 수 있다.

</details>

<details class="orca-details">
<summary>Node.js의 이벤트 루프는 브라우저랑 똑같을까? (phases·process.nextTick)</summary>

큰 그림(싱글 스레드 + 이벤트 루프 + 콜백 큐)은 같지만, **세부 구조가 다르다.** 브라우저는 HTML 명세를, Node는 libuv라는 C 라이브러리를 따른다.

Node의 이벤트 루프는 여러 **단계(phase)**로 나뉘어 돈다.

```
timers      → setTimeout·setInterval 콜백
pending     → 일부 시스템 콜백
poll        → I/O(파일·네트워크) 콜백 대기·처리
check       → setImmediate 콜백
close       → close 이벤트 콜백
```

그리고 Node에는 브라우저에 없는 두 가지가 있다.

- `setImmediate(fn)` : "현재 단계가 끝나면 check 단계에서 바로 실행"하는 타이머. `setTimeout(fn, 0)`과 비슷하지만 더 즉각적일 때가 많다.
- `process.nextTick(fn)` : **마이크로태스크보다도 먼저** 실행되는, Node만의 최우선 큐. 매 단계 전환 직전에 `nextTick` 큐를 전부 비운다.

```js
// Node에서의 실행 순서 (대략)
setTimeout(() => console.log('timeout'), 0);
setImmediate(() => console.log('immediate'));
Promise.resolve().then(() => console.log('promise'));
process.nextTick(() => console.log('nextTick'));
// nextTick → promise → timeout/immediate (순서는 상황에 따라)
```

=&gt; 그래서 "자바스크립트 비동기"라고 뭉뚱그려도 **브라우저와 Node의 실행 순서는 미묘하게 다르다.** 프런트엔드만 하면 브라우저 모델(매크로/마이크로)만 알면 되지만, Node(백엔드)를 다룬다면 phase와 `process.nextTick`의 우선순위까지 알아야 디버깅이 된다.

</details>



# 43장. Ajax

## Ajax란?

- **Ajax(Asynchronous JavaScript and XML)**는 자바스크립트로 브라우저가 서버에 **비동기 방식으로 데이터를 요청**하고, 응답받은 데이터로 웹페이지를 **동적으로 갱신**하는 프로그래밍 방식이다.
- 브라우저가 제공하는 Web API인 **XMLHttpRequest 객체**를 기반으로 동작한다.

**전통적인 방식의 문제점** (Ajax 이전)

- 서버에서 **완성된 HTML 전체**를 매번 받아 렌더링했다.
- 변경이 없는 부분까지 다시 받으니 **불필요한 데이터 통신**이 많았다.
- 전체 페이지를 다시 그리니 **화면이 번쩍 깜박였다.**
- 동기 방식이라 응답이 올 때까지 **블로킹**되어 아무것도 못 했다.

**Ajax의 장점**

1. **필요한 데이터만** 받아 불필요한 통신이 없다.
2. 변경할 부분만 갱신하므로 **깜박임이 없다.**
3. **비동기**라서 요청 후에도 블로킹 없이 다른 작업을 할 수 있다.

=&gt; 즉 Ajax의 등장으로 웹은 "페이지를 통째로 바꾸는 문서"에서 "부분부분 살아 움직이는 애플리케이션(SPA)"으로 진화했다.

<details class="orca-details">
<summary>이름에 XML이 들어가는데, 요즘도 XML을 쓸까?</summary>

**거의 안 쓴다. 지금은 대부분 JSON을 쓴다.** "Ajax"라는 이름은 2000년대 초에 지어졌는데, 당시엔 서버와 주고받는 데이터 포맷으로 XML이 흔했다. 그래서 이름에 XML이 박혔다.

하지만 XML은 태그가 많아 무겁고 파싱이 번거로웠다. 그래서 자바스크립트 객체와 모양이 거의 같고 훨씬 가벼운 **JSON**이 표준처럼 자리 잡았다.

```xml
<!-- XML: 무겁다 -->
<user><name>Lee</name><age>20</age></user>
```
```json
// JSON: 가볍고 JS 객체와 닮았다
{ "name": "Lee", "age": 20 }
```

=&gt; 그래서 "Ajax"는 이제 "XML"이란 글자와 무관하게 "**JS로 서버와 비동기 통신하는 방식**"을 통칭하는 말로 굳어졌다. 이름은 역사의 흔적일 뿐, 실제 데이터는 JSON이 대세다.

</details>



## JSON

- **JSON(JavaScript Object Notation)**은 클라이언트와 서버 간 HTTP 통신을 위한 **텍스트 데이터 포맷**이다. 자바스크립트에 종속되지 않고 대부분의 언어에서 쓸 수 있다.

```json
{
  "name": "Lee",
  "age": 20,
  "alive": true,
  "hobby": ["traveling", "tennis"]
}
```

표기 규칙이 자바스크립트 객체 리터럴과 비슷하지만 더 엄격하다.

- **키는 반드시 큰따옴표(`"`)로** 묶어야 한다. (작은따옴표 불가)
- **문자열 값도 반드시 큰따옴표**로 묶어야 한다.

### JSON.stringify (직렬화)

- 객체를 네트워크로 전송하려면 **문자열로 바꿔야(직렬화)** 한다. 이때 `JSON.stringify`를 쓴다.

```js
const obj = { name: 'Lee', age: 20, alive: true, hobby: ['traveling', 'tennis'] };

const json = JSON.stringify(obj);
console.log(typeof json, json);
// string {"name":"Lee","age":20,"alive":true,"hobby":["traveling","tennis"]}

// 세 번째 인수로 들여쓰기(가독성)
const pretty = JSON.stringify(obj, null, 2);

// 두 번째 인수(replacer)로 특정 값 필터링
const filtered = JSON.stringify(obj, (key, value) =>
  typeof value === 'number' ? undefined : value // 숫자 프로퍼티 제외
);
```

### JSON.parse (역직렬화)

- 서버에서 받은 JSON 문자열을 자바스크립트에서 쓰려면 **객체로 되돌려야(역직렬화)** 한다. 이때 `JSON.parse`를 쓴다.

```js
const json = '{"name":"Lee","age":20,"hobby":["traveling","tennis"]}';
const obj = JSON.parse(json);
console.log(typeof obj, obj); // object {name: 'Lee', age: 20, hobby: Array(2)}
```

=&gt; 외우는 법: **stringify = 객체 → 문자열(보낼 때), parse = 문자열 → 객체(받을 때).** "보낼 땐 끈으로 묶고(string-ify), 받을 땐 풀어 파싱한다"고 기억하면 된다.

<details class="orca-details">
<summary>JSON.stringify로 함수나 undefined를 직렬화하면 어떻게 될까?</summary>

**JSON에 없는 타입은 조용히 사라지거나 변환된다.** JSON은 "순수한 데이터"만 표현하는 포맷이라, 함수·`undefined`·심벌 같은 건 담을 수 없기 때문이다.

```js
const obj = {
  name: 'Lee',
  age: undefined,       // 사라짐
  greet: function () {}, // 사라짐
  when: new Date(),      // 문자열로 변환됨
  nan: NaN,              // null로 변환됨
};
console.log(JSON.stringify(obj));
// {"name":"Lee","when":"2026-10-06T..."}  ← age, greet는 아예 없음
```

- **함수·undefined·심벌**: 객체 프로퍼티면 **키째로 제거**, 배열 요소면 `null`로.
- **Date 객체**: `toJSON`이 불려 **ISO 문자열**이 된다. 그래서 `JSON.parse`로 되돌려도 Date가 아니라 **문자열**이다.
- **NaN·Infinity**: `null`로 변환.

=&gt; 그래서 "`stringify` → `parse`를 거치면 원본과 완전히 같지 않을 수 있다"는 걸 알아야 한다. 특히 Date가 문자열이 되는 건 실무에서 자주 겪는 함정이다. (이 특성 때문에 `JSON.parse(JSON.stringify(obj))`로 깊은 복사를 하는 건 함수·Date가 있으면 위험하다 — `structuredClone`이 더 안전하다.)

</details>



## XMLHttpRequest

- 브라우저는 서버와 HTTP 통신을 하기 위한 Web API로 **XMLHttpRequest(XHR)** 객체를 제공한다. Ajax는 이 객체로 구현된다.

### 객체 생성과 HTTP 요청 전송

- `new XMLHttpRequest()`로 인스턴스를 만들고, `open`으로 요청을 **초기화**한 뒤, (필요하면 `setRequestHeader`로 헤더 설정 후) `send`로 **전송**한다.

```js
// 1. XHR 객체 생성
const xhr = new XMLHttpRequest();

// 2. 요청 초기화: open(method, url[, async])
xhr.open('GET', 'https://jsonplaceholder.typicode.com/todos/1');

// 3. 요청 헤더 설정 (반드시 open 이후에)
xhr.setRequestHeader('content-type', 'application/json');

// 4. 요청 전송
xhr.send();
```

- `open(method, url)` : HTTP 요청 메서드(`GET`·`POST`·`PUT`·`DELETE` 등)와 요청 URL을 지정한다.
- `send(body)` : 요청을 전송한다. `GET`은 보통 `send()`(본문 없음), `POST`는 `send(JSON.stringify(data))`로 **페이로드**를 담는다.
- `setRequestHeader` : 요청 헤더를 설정한다. 반드시 `open` **호출 이후**에 써야 한다.

```js
// POST 요청 예시: 데이터를 본문에 담아 전송
const xhr = new XMLHttpRequest();
xhr.open('POST', '/users');
xhr.setRequestHeader('content-type', 'application/json');
xhr.send(JSON.stringify({ name: 'Lee', age: 20 }));
```

### HTTP 응답 처리

- 요청을 보냈다고 응답이 바로 오는 게 아니다(비동기!). 그래서 **응답이 도착했을 때 실행될 콜백(이벤트 핸들러)**을 등록해두고 기다린다.
- `load` 이벤트(`onload`)를 쓰면, **요청이 성공적으로 완료됐을 때** 핸들러가 호출된다.

```js
const xhr = new XMLHttpRequest();
xhr.open('GET', 'https://jsonplaceholder.typicode.com/todos/1');
xhr.send();

// 응답이 도착하면 이 콜백이 실행된다 (비동기)
xhr.onload = () => {
  // status 200~299면 성공
  if (xhr.status === 200) {
    console.log(JSON.parse(xhr.response)); // 받은 JSON 문자열 → 객체
  } else {
    console.error('Error', xhr.status, xhr.statusText);
  }
};
```

`onload` 대신 `readystatechange` 이벤트로도 처리할 수 있다. 이건 요청 상태(`readyState`)가 바뀔 때마다 호출되므로, 완료 상태(`DONE`, 4)를 직접 확인해야 한다.

```js
xhr.onreadystatechange = () => {
  // 요청이 완료(DONE) 상태가 아니면 아직 처리하지 않음
  if (xhr.readyState !== XMLHttpRequest.DONE) return;

  if (xhr.status === 200) {
    console.log(JSON.parse(xhr.response));
  } else {
    console.error('Error', xhr.status, xhr.statusText);
  }
};
```

주요 상태·응답 관련 프로퍼티는 다음과 같다.


| 프로퍼티         | 설명                                |
| ------------ | --------------------------------- |
| `readyState` | 요청의 현재 상태(0~4). `DONE`(4)이면 응답 완료 |
| `status`     | HTTP 상태 코드(예: 200, 404)           |
| `statusText` | 상태 메시지(예: "OK", "Not Found")      |
| `response`   | 응답 본문                             |


<details class="orca-details">
<summary>onload랑 onreadystatechange, 뭘 쓰는 게 좋을까?</summary>

**onload가 더 간단하고 권장된다.** 둘의 차이는 "언제, 몇 번 호출되느냐"다.

- `onreadystatechange` : `readyState`가 바뀔 때마다 (여러 번) 호출된다. 그래서 매번 `if (xhr.readyState !== DONE) return;`으로 완료 상태를 걸러내야 한다. 번거롭다.
- `onload` : 요청이 **완료됐을 때 딱 한 번** 호출된다. `readyState`를 확인할 필요가 없어 코드가 깔끔하다.

```js
// readystatechange: 상태 체크 필요
xhr.onreadystatechange = () => {
  if (xhr.readyState !== XMLHttpRequest.DONE) return;
  // ...
};

// onload: 바로 성공 처리 (권장)
xhr.onload = () => {
  if (xhr.status === 200) { /* ... */ }
};
```

=&gt; "완료 시점만 신경 쓰면 되는" 대부분의 경우 `onload`로 충분하다. `onreadystatechange`는 로딩 진행 상황을 단계별로 추적해야 하는 특수한 경우에나 쓴다.

</details>

<details class="orca-details">
<summary>status로 200을 확인하는데, 왜 콜백(비동기)으로 받아야 할까? 그냥 변수에 담으면 안 될까?</summary>

**서버 응답은 "언제 올지 모르는 미래의 값"이라, 요청 직후엔 아직 존재하지 않기 때문**이다.

```js
const xhr = new XMLHttpRequest();
xhr.open('GET', '/data');
xhr.send();
console.log(xhr.response); // '' (빈 값! 아직 응답이 안 옴)
```

`send()` 직후에 `xhr.response`를 읽으면 빈 값이다. 네트워크 요청은 수십 ms~수 초가 걸리는데, 자바스크립트는 그걸 기다리지 않고(42장 비동기!) 바로 다음 줄로 넘어가기 때문이다. 응답이 실제로 도착하는 건 한참 뒤, 콜 스택이 다 비워지고 이벤트 루프가 콜백을 실행할 때다.

그래서 "응답이 도착하면 그때 실행해줘"라고 **콜백(`onload`)을 미리 등록**해두는 것이다.

=&gt; 이게 바로 42장 이벤트 루프와 직결된다. 그리고 이렇게 콜백으로 처리하다 보면, 요청이 요청을 부르는 "**콜백 지옥(callback hell)**"에 빠지기 쉽다. 그 문제를 우아하게 푸는 게 45장의 **Promise**, 46장의 **async/await**다. Ajax를 알아야 Promise가 왜 필요한지가 보인다.

</details>

<details class="orca-details">
<summary>요즘은 XHR 대신 fetch나 axios를 쓴다던데, 그래도 XHR을 배워야 할까?</summary>

**원리 이해를 위해 알아둘 가치는 있지만, 실무 코드는 `fetch`/`axios`로 쓴다.**

XHR은 문법이 번거롭고 콜백 기반이라 중첩이 심해진다. 그래서 현대 브라우저는 **Promise 기반**의 `fetch`를 내장 제공하고, 실무에선 `axios` 같은 라이브러리도 많이 쓴다.

```js
// XHR (옛날 방식, 콜백)
const xhr = new XMLHttpRequest();
xhr.open('GET', '/todos/1');
xhr.onload = () => console.log(JSON.parse(xhr.response));
xhr.send();

// fetch (Promise 기반, 현대)
fetch('/todos/1')
  .then((res) => res.json())
  .then((data) => console.log(data));

// async/await (가장 읽기 좋음)
const res = await fetch('/todos/1');
const data = await res.json();
```

=&gt; XHR → fetch → async/await로 갈수록 "비동기 코드가 동기 코드처럼 읽히게" 발전한 것이다. XHR을 배우는 이유는, 이 모든 게 결국 "Ajax 요청을 보내고 응답을 비동기로 받는다"는 **같은 뼈대** 위에 있다는 걸 이해하기 위해서다. 뼈대를 알면 `fetch`의 `.then`이 왜 필요한지가 자연스럽다.

</details>

<details class="orca-details">
<summary>"콜백 지옥"이 실제로 어떻게 생기고, 왜 Promise로 가게 될까?</summary>

콜백 지옥(callback hell)은 **"A 응답이 와야 B를 요청하고, B가 와야 C를 요청하는" 순차 비동기 작업을 콜백으로만 엮을 때** 생기는, 오른쪽으로 계단처럼 깊어지는 중첩 구조다.

예를 들어 "유저 조회 → 그 유저의 글 조회 → 그 글의 댓글 조회"를 XHR로 하면 이렇게 된다.

```js
getUser(1, (user) => {
  getPosts(user.id, (posts) => {
    getComments(posts[0].id, (comments) => {
      console.log(comments);
      // ...여기서 또 요청하면 계속 오른쪽으로 깊어진다
    }, onError);
  }, onError);
}, onError);
```

문제가 두 가지다.
1. **가독성**: 들여쓰기가 계속 깊어져 "파멸의 피라미드(pyramid of doom)"가 된다.
2. **에러 처리**: 단계마다 `onError`를 일일이 넘겨야 하고, 한곳에서 에러를 모아 처리하기 어렵다.

같은 걸 Promise 체인으로 바꾸면 중첩이 **평평**해지고 에러도 `.catch` 하나로 모인다.

```js
getUser(1)
  .then((user) => getPosts(user.id))
  .then((posts) => getComments(posts[0].id))
  .then((comments) => console.log(comments))
  .catch(onError); // 어느 단계에서 터져도 여기로
```

=&gt; 즉 콜백 자체가 나쁜 게 아니라, **"순차적으로 이어지는 여러 비동기 작업"**을 콜백으로 엮을 때 구조가 무너진다. 그걸 "중첩 대신 체인"으로 펴주는 게 45장 Promise이고, 그것마저 동기 코드처럼 보이게 하는 게 46장 async/await다. 43장의 XHR 콜백을 직접 겪어봐야 이 발전의 필요성이 체감된다.

</details>

<details class="orca-details">
<summary>CORS 에러는 왜 뜨고, 브라우저는 내부적으로 어떻게 검사할까?</summary>

Ajax를 하다 보면 반드시 만나는 게 **CORS(Cross-Origin Resource Sharing) 에러**다. 뿌리는 브라우저의 보안 규칙인 **동일 출처 정책(Same-Origin Policy)**이다.

**출처(origin)**는 `프로토콜 + 호스트 + 포트` 세 가지가 모두 같아야 "같은 출처"다.

```
https://a.com        ← 기준
https://a.com/path   → 같은 출처 (경로는 무관)
http://a.com         → 다른 출처 (프로토콜 다름)
https://b.com        → 다른 출처 (호스트 다름)
https://a.com:8080   → 다른 출처 (포트 다름)
```

동일 출처 정책은 **다른 출처의 리소스에 대한 Ajax 요청을 기본적으로 차단**한다. 악성 사이트가 내 로그인 세션으로 내 은행 API를 몰래 호출하는 것 등을 막기 위해서다. CORS는 이 차단을 **서버가 허락하면 풀어주는** 공식 우회 메커니즘이다.

동작은 이렇다. 브라우저가 다른 출처로 요청을 보낼 때,
1. **단순 요청(GET 등)**: 요청을 보내되, 응답 헤더에 `Access-Control-Allow-Origin`이 내 출처(또는 `*`)를 허용하는지 확인한다. 없으면 **응답을 받아놓고도 자바스크립트에 넘겨주지 않고** CORS 에러를 던진다.
2. **예비 요청(preflight)**: `PUT`·`DELETE`이거나 커스텀 헤더가 있는 등 "복잡한" 요청이면, 본 요청 전에 `OPTIONS` 메서드로 **"이 요청 해도 돼?"를 먼저 물어본다.** 서버가 허용 헤더로 답해야 본 요청이 나간다.

```
# preflight (브라우저가 자동으로 보냄)
OPTIONS /todos
# 서버 응답이 이걸 포함해야 통과
Access-Control-Allow-Origin: https://myapp.com
Access-Control-Allow-Methods: GET, POST, PUT, DELETE
```

=&gt; 핵심은 **CORS는 "서버"가 허용 헤더로 풀어주는 것**이지, 프런트 코드로 끄는 게 아니라는 점이다. 그래서 "CORS 에러"는 대부분 **백엔드 설정** 문제다. (로컬 개발 땐 프록시로 같은 출처인 척 우회하기도 한다.) 그리고 이건 **브라우저만의 규칙**이라, 서버 간 통신이나 Postman에서는 CORS가 없다.

</details>

<details class="orca-details">
<summary>요청을 중간에 취소하거나 타임아웃을 걸 수 있을까? (abort·timeout·progress)</summary>

있다. 네트워크 요청은 오래 걸릴 수 있어서, **진행 상황을 추적하거나 중간에 끊는** 제어가 필요하다.

**XHR의 요청 제어**

```js
const xhr = new XMLHttpRequest();
xhr.open('GET', '/big-file');
xhr.timeout = 3000;                 // 3초 넘으면 자동 중단
xhr.ontimeout = () => console.log('시간 초과');
xhr.onprogress = (e) => {           // 다운로드 진행률
  if (e.lengthComputable) console.log(`${(e.loaded / e.total) * 100}%`);
};
xhr.send();

// 사용자가 '취소'를 누르면
xhr.abort();                        // 진행 중인 요청을 즉시 중단
```

**fetch의 요청 취소 (AbortController)**

`fetch`는 더 간결한 대신 취소를 `AbortController`라는 별도 객체로 한다.

```js
const controller = new AbortController();
fetch('/big-file', { signal: controller.signal })
  .then((res) => res.json())
  .catch((e) => {
    if (e.name === 'AbortError') console.log('취소됨');
  });

controller.abort(); // 요청 취소 → fetch가 AbortError로 reject
```

=&gt; 이 "취소" 기능은 실무에서 아주 중요하다. 예를 들어 **검색 자동완성**에서, 사용자가 빠르게 타이핑하면 이전 요청들을 `abort`로 끊어야 "방금 친 글자"의 결과만 받는다(경쟁 상태 방지). React에서 컴포넌트가 언마운트될 때 진행 중인 요청을 취소하는 것도 같은 맥락이다. "요청은 보내고 끝"이 아니라 "추적하고 끊을 수 있는 대상"이라는 걸 알아두자.

</details>


# 44장. REST API

## REST란?

- **REST(REpresentational State Transfer)**는 HTTP를 기반으로 **클라이언트가 서버의 리소스에 접근하는 방식을 규정한 아키텍처**다. HTTP/1.0·1.1 스펙 작성에 참여한 로이 필딩이 2000년에 제안했다.
- 당시 웹이 HTTP의 장점을 제대로 못 살리던 상황을 개선하려고, HTTP를 **"의도에 맞게" 제대로 쓰자**는 취지로 나왔다.

용어를 정리하면 이렇다.

- **REST**: 리소스 접근 방식을 규정한 **아키텍처(설계 원칙)**.
- **REST API**: 그 REST를 기반으로 구현한 **서비스 API**.
- **RESTful**: REST 원칙을 **잘 지킨** 상태를 가리키는 말.

=&gt; 핵심은 **"REST API만 보고도 요청의 내용을 이해할 수 있어야 한다"**는 것이다. 주소(URI)와 메서드만 봐도 "무엇을, 어떻게 하려는 요청인지"가 드러나는 자체 표현 구조를 지향한다.

<details class="orca-details">
<summary>REST랑 RESTful은 뭐가 다를까? 실무 API는 다 RESTful할까?</summary>

**REST는 "원칙", RESTful은 "그 원칙을 잘 지킨 상태"**를 뜻한다. 그리고 솔직히 말하면, **실무의 많은 API는 완벽하게 RESTful하지 않다.**

로이 필딩의 REST에는 사실 꽤 엄격한 조건들(특히 HATEOAS — 응답에 다음 행동으로 갈 링크를 포함하는 것)이 있는데, 이걸 전부 지키는 API는 드물다. 그래서 업계에서 "REST API"라고 부르는 것은 대개 **"URI로 자원을 표현하고, HTTP 메서드로 행위를 표현하는"** 핵심 두 원칙만 지킨 수준이다.

=&gt; 그래서 "엄밀히 RESTful하냐"를 따지기보다, **"주소와 메서드만 봐도 의도가 읽히게 일관되게 설계했는가"**를 실무 기준으로 삼으면 된다. 최근엔 REST의 한계(과도/과소 fetching 등)를 보완하려 **GraphQL** 같은 대안도 쓰이지만, REST가 여전히 가장 보편적인 표준이다.

</details>



## REST API의 구성

- REST API는 **자원, 행위, 표현** 세 가지 요소로 구성된다. "무엇을(자원), 어떻게(행위), 어떤 내용으로(표현)" 요청하는지를 나눠 담는 것이다.


| 구성 요소                  | 내용             | 표현 방법              |
| ---------------------- | -------------- | ------------------ |
| **자원(resource)**       | 요청의 대상(무엇을)    | **URI**(엔드포인트)     |
| **행위(verb)**           | 자원에 대한 조작(어떻게) | **HTTP 요청 메서드**    |
| **표현(representation)** | 행위의 구체적 내용     | **페이로드**(요청/응답 본문) |


```
DELETE  /todos/1
───┬──  ───┬────
  행위      자원
(HTTP 메서드) (URI)
```

=&gt; 예를 들어 "1번 할 일을 삭제해줘"라는 요청은 **자원(`/todos/1`) + 행위(`DELETE`)**로 표현된다. 주소와 메서드가 각자 역할을 나눠 가진다는 게 핵심이다.



## REST API 설계 원칙

REST API 설계의 중심 규칙은 딱 두 가지다. 이 둘만 지켜도 "RESTful하다"고 할 수 있다.

### ① URI는 리소스(자원)를 표현해야 한다

- URI는 **무엇을(리소스)**에 집중해야 한다. 리소스 이름은 **동사가 아니라 명사**로 쓴다.
- `get`, `show`, `delete` 같은 **행위를 나타내는 동사를 URI에 넣으면 안 된다.** 행위는 다음 원칙(HTTP 메서드)이 담당하기 때문이다.

```
# 나쁜 예 (URI에 행위가 들어감)
GET /getTodos/1
GET /todos/show/1

# 좋은 예 (URI는 리소스만)
GET /todos/1
```

### ② 리소스에 대한 행위는 HTTP 요청 메서드로 표현한다

- **어떻게(행위)**는 URI가 아니라 **HTTP 메서드**로 표현한다. 조회는 `GET`, 생성은 `POST`, 삭제는 `DELETE`처럼.

```
# 나쁜 예 (URI로 행위를 표현)
GET /todos/delete/1

# 좋은 예 (행위는 메서드로)
DELETE /todos/1
```

주요 HTTP 요청 메서드 5가지는 데이터 처리의 기본 4가지(CRUD)에 대응된다.


| 메서드        | 종류    | 목적         | CRUD   | 페이로드 | 멱등성 |
| ---------- | ----- | ---------- | ------ | :----: | :---: |
| **GET**    | 조회    | 리소스 취득     | Read   | X    | O   |
| **POST**   | 생성    | 리소스 생성     | Create | O    | X   |
| **PUT**    | 전체 교체 | 리소스 전체를 교체 | Update | O    | O   |
| **PATCH**  | 부분 수정 | 리소스 일부만 수정 | Update | O    | X   |
| **DELETE** | 삭제    | 리소스 삭제     | Delete | X    | O   |


<details class="orca-details">
<summary>PUT이랑 PATCH, 둘 다 수정인데 뭐가 다를까?</summary>

**PUT은 "통째로 교체", PATCH는 "일부만 수정"**이다. 이 차이가 실무에서 꽤 중요하다.

`{ id: 1, content: 'HTML', completed: false }`라는 할 일을 수정한다고 하자.

- **PUT**: 리소스 **전체를 보낸 데이터로 덮어쓴다.** 그래서 일부 필드만 보내면 **나머지가 사라질 수 있다.**

```js
// PUT: content만 보내면 completed가 날아갈 수 있음 (전체 교체니까)
fetch('/todos/1', { method: 'PUT', body: JSON.stringify({ content: '수정' }) });
// 결과: { content: '수정' }  ← id, completed 소실!
```

- **PATCH**: 보낸 필드만 **부분적으로 바꾼다.** 나머지는 그대로 유지된다.

```js
// PATCH: completed만 바꾸고 나머지는 유지
fetch('/todos/1', { method: 'PATCH', body: JSON.stringify({ completed: true }) });
// 결과: { id: 1, content: 'HTML', completed: true }  ← 나머지 유지
```

=&gt; "전체를 새 걸로 갈아끼우면 PUT, 몇 개 필드만 고치면 PATCH". 실무에선 안전하게 **PATCH를 더 많이** 쓴다. PUT을 쓸 땐 반드시 전체 데이터를 다 보내야 한다는 걸 잊지 말자.

</details>

<details class="orca-details">
<summary>표에 나온 &quot;멱등성&quot;이 뭘까? 왜 POST·PATCH만 멱등이 아닐까?</summary>

**멱등성(idempotence)은 "같은 요청을 여러 번 보내도 결과(서버 상태)가 똑같은" 성질**이다.

- **GET**: 몇 번을 조회해도 서버 상태가 안 바뀜 → 멱등 O
- **PUT**: `{id:1, content:'A'}`로 10번 교체해도 결과는 똑같이 `content:'A'` → 멱등 O
- **DELETE**: 1번을 10번 지워도 "1번이 없는 상태"로 동일 → 멱등 O
- **POST**: 생성 요청을 10번 보내면 리소스가 **10개 생김** → 멱등 X (그래서 결제 버튼 중복 클릭이 위험)
- **PATCH**: 구현에 따라 다름. 예를 들어 "조회수 +1" 같은 상대적 수정은 보낼 때마다 결과가 달라짐 → 보통 멱등 X로 분류

```js
// POST를 두 번 보내면 → 같은 할 일이 2개 생긴다 (멱등 X)
fetch('/todos', { method: 'POST', body: ... }); // id:4 생성
fetch('/todos', { method: 'POST', body: ... }); // id:5 또 생성!
```

=&gt; 멱등성이 중요한 이유는 **안전한 재시도** 때문이다. 네트워크가 끊겨 응답을 못 받았을 때, 멱등한 요청(GET·PUT·DELETE)은 그냥 다시 보내도 안전하지만, POST는 중복 생성 위험이 있어 재시도에 주의해야 한다. "결제·주문 중복 방지"가 바로 이 POST 비멱등성 때문에 생기는 실무 이슈다.

</details>

<details class="orca-details">
<summary>"안전한(safe) 메서드"는 멱등성이랑 다른 걸까? GET은 왜 캐싱될까?</summary>

**안전성(safety)과 멱등성(idempotence)은 비슷해 보이지만 다른 개념**이다.

- **안전한 메서드**: 서버의 상태를 **전혀 변경하지 않는**(읽기 전용) 메서드. → **GET**, HEAD, OPTIONS
- **멱등한 메서드**: 여러 번 호출해도 **결과 상태가 같은** 메서드. → GET, PUT, DELETE

관계를 보면 **"안전하면 반드시 멱등"**이다(상태를 안 바꾸니 몇 번을 불러도 같다). 하지만 역은 성립하지 않는다. PUT·DELETE는 멱등이지만 상태를 바꾸므로 안전하지 않다.

| 메서드 | 안전(safe) | 멱등(idempotent) |
| --- | :---: | :---: |
| GET | O | O |
| PUT | X | O |
| DELETE | X | O |
| POST | X | X |
| PATCH | X | X |

이 "안전성"이 실무에서 중요한 이유가 **캐싱**이다. GET은 상태를 안 바꾸니 브라우저·CDN·프록시가 **응답을 캐시해 재사용**해도 안전하다. 그래서 GET 응답에는 `Cache-Control`·`ETag` 같은 캐시 헤더를 붙여 성능을 높인다.

```
# GET 응답: 캐시 가능 (안전한 메서드라서)
Cache-Control: max-age=3600
ETag: "abc123"
```

=&gt; 그래서 **"조회는 반드시 GET으로"** 하는 게 중요하다. 만약 조회를 POST로 하면(간혹 그런 설계가 있다) 캐싱·재시도·북마크 같은 HTTP의 기본 이점을 전부 잃는다. "상태를 안 바꾸면 GET, 바꾸면 나머지"라는 구분이 캐싱 전략의 출발점이다.

</details>

<details class="orca-details">
<summary>그럼 /todos/1/complete 처럼 &quot;완료 처리&quot;는 어떻게 설계할까?</summary>

REST를 엄격히 지키면 **상태 변경도 리소스로 보고 PATCH로** 처리한다. 하지만 현실에선 종종 타협한다.

- **RESTful한 방식**: "완료"를 `completed` 필드의 변경으로 본다.

```
PATCH /todos/1   body: { "completed": true }
```

- **현실에서 자주 보는 방식**: 복잡한 동작(결제, 발송, 승인 등)은 동사형 엔드포인트를 쓰기도 한다.

```
POST /todos/1/complete   ← 엄밀히는 비 RESTful하지만 의도가 명확
POST /orders/1/cancel
```

순수주의 관점에선 후자가 "URI에 행위(동사)를 넣지 말라"는 원칙 위반이다. 하지만 "주문 취소"처럼 단순 CRUD로 표현하기 애매한 **행위 중심 동작**은, 억지로 PATCH로 욱여넣기보다 명확한 동사 엔드포인트가 팀에 더 친절할 때가 있다.

=&gt; "**기본은 명사 + HTTP 메서드(RESTful), 단순 CRUD로 표현이 어려운 복잡한 행위는 실용적으로 타협**"이 현실적인 기준이다. 원칙을 알되 맹신하지 않는 게 좋은 설계다.

</details>



## JSON Server를 이용한 실습

인데 여긴 그냥... 거의 받아쓰기 ㅋㅋ

- **JSON Server**는 `json` 파일 하나로 **가상의 REST API 서버**를 뚝딱 만들어주는 도구다. 백엔드 서버가 아직 없을 때, 프런트엔드가 실제 서버처럼 요청을 연습할 수 있다.

```bash
# 설치
npm install json-server --save-dev
```

```json
// db.json — 이 파일이 데이터베이스 역할을 한다
{
  "todos": [
    { "id": 1, "content": "HTML", "completed": false },
    { "id": 2, "content": "CSS", "completed": true }
  ]
}
```

```bash
# 서버 실행 (watch로 파일 변경 감지)
json-server --watch db.json
```

이제 `http://localhost:3000/todos`로 CRUD 요청을 보낼 수 있다.

**GET — 조회**

```js
// 전체 조회: GET /todos  / 특정 조회: GET /todos/1
const xhr = new XMLHttpRequest();
xhr.open('GET', 'http://localhost:3000/todos');
xhr.send();
xhr.onload = () => {
  if (xhr.status === 200) console.log(JSON.parse(xhr.response));
};
```

**POST — 생성** (페이로드 필요, 성공 시 상태 코드 201)

```js
const xhr = new XMLHttpRequest();
xhr.open('POST', 'http://localhost:3000/todos');
xhr.setRequestHeader('content-type', 'application/json'); // 페이로드 타입 지정
xhr.send(JSON.stringify({ id: 3, content: 'React', completed: false }));
xhr.onload = () => {
  if (xhr.status === 201) console.log(JSON.parse(xhr.response));
};
```

**PUT — 전체 교체** / **PATCH — 부분 수정**

```js
// PUT: id 제외 전체를 교체
xhr.open('PUT', 'http://localhost:3000/todos/1');
xhr.setRequestHeader('content-type', 'application/json');
xhr.send(JSON.stringify({ id: 1, content: 'HTML 학습', completed: true }));

// PATCH: 일부 필드만 수정
xhr.open('PATCH', 'http://localhost:3000/todos/1');
xhr.setRequestHeader('content-type', 'application/json');
xhr.send(JSON.stringify({ completed: true }));
```

**DELETE — 삭제** (페이로드 없음)

```js
const xhr = new XMLHttpRequest();
xhr.open('DELETE', 'http://localhost:3000/todos/1');
xhr.send();
```

=&gt; **① 생성 → ② open(메서드+URL) → ③ (페이로드 있으면) 헤더 설정 + send(JSON) → ④ onload로 응답 처리.** 달라지는 건 **메서드와 페이로드 유무**뿐이다.

<details class="orca-details">
<summary>POST는 왜 200이 아니라 201을 반환할까? 상태 코드는 어떤 게 있을까?</summary>

**201(Created)은 "새 리소스가 생성됨"을 명확히 알리는 코드**다. 그냥 200(OK)보다 의도가 구체적이다.

HTTP 상태 코드는 첫 자리로 큰 분류를 나눈다.

- **2xx 성공**: `200 OK`(일반 성공), `201 Created`(생성됨, POST 응답), `204 No Content`(성공했으나 본문 없음, DELETE 응답에 자주)
- **3xx 리다이렉션**: `301`(영구 이동), `304 Not Modified`(캐시 사용)
- **4xx 클라이언트 오류**: `400`(잘못된 요청), `401`(인증 필요), `403`(권한 없음), `404`(리소스 없음)
- **5xx 서버 오류**: `500`(서버 내부 오류), `503`(서버 과부하)

```js
xhr.onload = () => {
  if (xhr.status >= 200 && xhr.status < 300) {
    // 2xx면 성공
  } else if (xhr.status === 404) {
    // 리소스 없음
  }
};
```

=&gt; 그래서 응답 처리할 때 "200만 성공"으로 보면 201·204를 놓친다. `status >= 200 && status < 300` 범위로 성공을 판단하는 게 안전하다. 상태 코드는 "요청이 어떻게 됐는지"를 숫자로 알려주는 공통 언어라 꼭 익혀둬야 한다.

</details>

<details class="orca-details">
<summary>HTTP 헤더에는 뭘 담고, 인증은 어떻게 흘러갈까?</summary>

HTTP 요청·응답은 **헤더(메타데이터) + 본문(실제 데이터)**으로 나뉜다. 헤더는 "이 요청/응답을 어떻게 다뤄야 하는지"를 알려주는 쪽지다. REST API에서 자주 쓰는 헤더는 이렇다.

- `Content-Type` : 내가 **보내는** 본문의 형식. `application/json`이면 "본문은 JSON이야".
- `Accept` : 내가 **받고 싶은** 응답 형식. `application/json`이면 "JSON으로 답해줘".
- `Authorization` : **인증 정보**. 보통 토큰을 담는다. `Bearer <토큰>` 형식.

실무에서 가장 자주 다루는 게 **토큰 인증**이다. 흐름은 이렇다.

```
1. 로그인:  POST /login  { id, password }
            → 서버가 검증 후 '토큰(JWT 등)'을 발급해 응답

2. 이후 요청마다 그 토큰을 Authorization 헤더에 담아 보냄
   GET /todos
   Authorization: Bearer eyJhbGciOi...   ← "나 이 사람 맞아"의 증명
```

```js
fetch('/todos', {
  headers: {
    'Content-Type': 'application/json',
    Authorization: `Bearer ${token}`, // 저장해둔 토큰을 매 요청에 첨부
  },
});
```

=&gt; 그래서 "로그인 후 다른 API가 401(인증 필요)을 뱉는다"면, 대개 **토큰을 헤더에 안 실었거나 만료된** 경우다. 헤더는 눈에 잘 안 띄지만 인증·형식 협상·캐싱 같은 HTTP의 핵심 동작이 전부 여기서 이뤄진다. 개발자 도구 Network 탭에서 요청/응답 헤더를 열어보는 습관이 중요하다.

</details>

<details class="orca-details">
<summary>REST의 "무상태성(Stateless)"이 무슨 뜻이고, 왜 토큰을 매번 보낼까?</summary>

REST의 중요한 제약 중 하나가 **무상태성(Statelessness)**이다. **서버가 클라이언트의 "이전 요청 상태"를 기억하지 않는다**는 원칙이다.

즉, 모든 요청은 **그 요청 하나만으로 처리에 필요한 정보를 전부 담고 있어야** 한다. 서버는 "아까 이 사람이 로그인했었지" 같은 걸 기억하지 않으므로, 클라이언트가 **매 요청마다 자신이 누구인지(토큰)를 들고 와야** 한다.

```
# 무상태: 각 요청이 독립적. 서버는 요청 간 기억이 없음
GET /todos   Authorization: Bearer <토큰>   ← 매번 토큰 첨부
GET /posts   Authorization: Bearer <토큰>   ← 또 첨부 (아까 보낸 건 서버가 기억 안 함)
```

이게 번거로워 보여도 큰 장점이 있다.
- **확장성(scalability)**: 서버가 상태를 안 들고 있으니, 요청을 **아무 서버에나** 보내도 된다. 서버를 여러 대로 늘리기(스케일 아웃) 쉽다.
- **안정성**: 서버가 죽었다 살아나도 "잃어버릴 세션 상태"가 없다.

=&gt; 전통적인 **세션 방식**(서버가 로그인 상태를 메모리에 기억)은 이 무상태 원칙에 어긋난다. 그래서 REST API에서는 상태를 서버가 아닌 **토큰(JWT)에 담아 클라이언트가 들고 다니는** 방식을 선호한다. "서버는 기억하지 않는다 → 그래서 매번 증명해야 한다"가 토큰 인증의 철학적 뿌리다.

</details>

<details class="orca-details">
<summary>목록이 수천 개면? 페이지네이션·필터·정렬은 어떻게 설계할까?</summary>

`GET /todos`가 수천 개를 한 번에 다 내려주면 느리고 비효율적이다. 그래서 **쿼리 스트링(`?key=value`)**으로 "어떻게 잘라서 줄지"를 지정한다. 이건 리소스(URI)가 아니라 "조회 조건"이라 쿼리로 표현하는 게 RESTful하다.

```
# 페이지네이션: 11~20번째 항목
GET /todos?offset=10&limit=10
GET /todos?page=2&size=10        ← 페이지 번호 방식

# 필터링: 완료된 것만
GET /todos?completed=true

# 정렬: 생성일 내림차순
GET /todos?sort=createdAt&order=desc

# 조합도 가능
GET /todos?completed=false&sort=priority&order=desc&page=1&size=20
```

핵심 구분은 이렇다.
- **경로(path)**: "무엇을"을 가리키는 **리소스 식별**. `/todos/1`
- **쿼리(query)**: "그중 어떻게 걸러·정렬·자를지"의 **조회 옵션**. `?completed=true`

=&gt; 그래서 "완료된 할 일"은 `/completedTodos`(새 리소스처럼) 가 아니라 `/todos?completed=true`(todos를 거른 것)로 설계하는 게 맞다. 대용량 목록 API는 거의 항상 페이지네이션이 필수인데, 커서 기반(`?cursor=...`) vs 오프셋 기반(`?offset=...`)의 트레이드오프까지 알면 더 깊이 설계할 수 있다. (커서 방식이 대용량·실시간에 유리하다.)

</details>


