# 모던 자바스크립트 Deep Dive 35 ~ 38장 정리

> 35장 스프레드 문법 · 36장 디스트럭처링 할당 · 37장 Set과 Map · 38장 브라우저의 렌더링 과정

## 35장 스프레드 문법

### 스프레드 문법

- `...`은 뭉쳐 있는 여러 값들의 집합을 펼쳐서 **개별적인 값들의 목록**으로 만든다.
- 대상은 `for...of`로 순회할 수 있는 **이터러블**(`Array`, `String`, `Map`, `Set`, `arguments` 등)에 한정된다.
- 결과는 **값이 아니라 값들의 목록**이다 ⇒ 변수에 할당할 수 없고, 쉼표로 구분한 목록을 쓰는 곳(함수 인수, 배열 리터럴, 객체 리터럴)에서만 쓸 수 있다.

```js
console.log(...[1, 2, 3]); // 1 2 3
console.log(...'Hi'); // H i

const list = ...[1, 2, 3]; // SyntaxError ← 값이 아니므로 할당 불가
```

### 사용처

**① 함수 인수 목록**

```js
const arr = [1, 2, 3];

Math.max(arr); // NaN
Math.max.apply(null, arr); // 3 ← ES5
Math.max(...arr); // 3 ← ES6
```

- Rest 파라미터와 모양은 같지만 **반대 개념**이다. Rest는 인수 목록을 **배열로 모으고**, 스프레드는 배열을 **목록으로 펼친다**.

**② 배열 리터럴** — `concat`, `slice`, `splice`를 대체

```js
const merged = [...[1, 2], ...[3, 4]]; // [1, 2, 3, 4] ← concat 대체

const origin = [{ a: 1 }];
const copy = [...origin]; // slice 대체
console.log(copy === origin); // false
console.log(copy[0] === origin[0]); // true ← 얕은 복사
```

- 이터러블이 아닌 유사 배열 객체는 펼칠 수 없다 ⇒ `Array.from` 사용

**③ 객체 리터럴 (스프레드 프로퍼티, ES2018)**

- 일반 객체도 대상으로 허용한다. `Object.assign`을 대체하며, 키가 겹치면 **뒤에 위치한 프로퍼티가 우선**한다.

```js
const merged = { ...{ x: 1, y: 2 }, ...{ y: 10, z: 3 } }; // { x: 1, y: 10, z: 3 }
const changed = { ...{ x: 1, y: 2 }, y: 100 }; // { x: 1, y: 100 }
```

⇒ 리액트에서 상태를 불변하게 업데이트하는 `{ ...state, key: value }` 패턴이 이것

## 36장 디스트럭처링 할당

### 디스트럭처링 할당

- 이터러블이나 객체를 **비구조화하여 1개 이상의 변수에 개별적으로 할당**하는 것. 필요한 값만 꺼낼 때 유용하다.

### 배열 디스트럭처링

- 우변은 **이터러블**, 할당 기준은 **인덱스(순서)**

```js
const [a, b] = [1, 2]; // 1 2
const [c, d] = [1]; // 1 undefined ← 개수가 달라도 된다
const [e, , f] = [1, 2, 3]; // 1 3 ← 건너뛰기
const [g, h = 10] = [1]; // 1 10 ← 기본값 (undefined일 때만 적용)
const [x, ...rest] = [1, 2, 3]; // 1 [2, 3] ← Rest 요소는 마지막에

let m = 1, n = 2;
[m, n] = [n, m]; // swap → 2 1
```

### 객체 디스트럭처링

- 우변은 **객체**, 할당 기준은 **프로퍼티 키**(순서 무관)
- `{ lastName }`은 `{ lastName: lastName }`의 축약 표현이다 ⇒ 다른 이름으로 받으려면 `{ key: newName }`

```js
const user = { firstName: 'Ungmo', lastName: 'Lee' };

const { lastName, firstName } = user; // Lee Ungmo
const { lastName: ln } = user; // 다른 이름으로 → ln = 'Lee'
const { age = 20 } = user; // 기본값 → 20
const { firstName: fn, ...rest } = user; // Rest 프로퍼티 → rest = { lastName: 'Lee' }
```

- **매개변수**와 **중첩 객체**에도 쓸 수 있다.

```js
// 매개변수 → 리액트 컴포넌트의 props 받기와 같은 패턴
function printTodo({ content, completed }) {
  console.log(`${content}: ${completed ? '완료' : '비완료'}`);
}
printTodo({ id: 1, content: 'HTML', completed: true }); // HTML: 완료

// 중첩 객체
const { address: { city } } = { name: 'Lee', address: { city: 'Seoul' } };
console.log(city); // 'Seoul'
```

### 정리 — 배열 vs 객체 디스트럭처링

| | 배열 디스트럭처링 | 객체 디스트럭처링 |
| --- | --- | --- |
| 우변 대상 | 이터러블 | 객체 |
| 할당 기준 | 인덱스 (순서) | 프로퍼티 키 (순서 무관) |
| 기본값 | `[a = 1]` | `{ a = 1 }` |
| 나머지 | `[a, ...rest]` | `{ a, ...rest }` |

## 37장 Set과 Map

### Set

- **중복되지 않는 유일한 값들의 집합**. 수학적 집합을 구현하기 위한 자료구조다.

| 구분 | 배열 | Set |
| --- | --- | --- |
| 중복 값 허용 | O | X |
| 순서에 의미 | O | X |
| 인덱스 접근 | O | X |

- 생성자에 이터러블을 넘기면 중복이 제거된다 ⇒ **배열 중복 제거**에 자주 쓴다.
- `NaN`과 `NaN`, `+0`과 `-0`을 **같은 값**으로 본다.

```js
const uniq = (array) => [...new Set(array)];
uniq([2, 1, 2, 3, 3]); // [2, 1, 3]

const set = new Set();
set.add(1).add(2).add(2); // add는 Set을 반환 → 체이닝 가능, 중복은 무시
set.has(1); // true
set.delete(1); // true ← 불리언 반환 → 체이닝 불가, 인덱스가 아니라 값을 넘긴다
set.size; // 1 ← getter만 있는 접근자 프로퍼티라 할당해도 안 바뀜
set.clear();
```

- **이터러블**이라 `for...of`, 스프레드, 배열 디스트럭처링이 가능하다. 순회 순서는 추가된 순서
- 집합 연산은 스프레드 + `filter`로 구현한다. (ES2025부터 `union`, `intersection`, `difference` 등이 표준 메서드로 추가됨)

```js
const setA = new Set([1, 2, 3, 4]);
const setB = new Set([2, 4]);

new Set([...setA].filter((v) => setB.has(v))); // 교집합 {2, 4}
new Set([...setA, ...setB]); // 합집합 {1, 2, 3, 4}
new Set([...setA].filter((v) => !setB.has(v))); // 차집합 {1, 3}
```

### Map

- **키와 값의 쌍으로 이루어진 컬렉션**. 객체와 비슷하지만

| 구분 | 객체 | Map |
| --- | --- | --- |
| 키로 쓸 수 있는 값 | 문자열, 심벌 | **객체를 포함한 모든 값** |
| 이터러블 | X | O |
| 요소 개수 | `Object.keys(obj).length` | `map.size` |

```js
const lee = { name: 'Lee' };
const kim = { name: 'Kim' };

const map = new Map([['key1', 'value1']]); // [키, 값] 쌍의 이터러블로 생성
map.set(lee, 'developer').set(kim, 'designer'); // 객체도 키가 된다, 체이닝 가능
map.get(lee); // 'developer'
map.has(kim); // true
map.delete(kim); // true

// 일반 객체에 객체를 키로 쓰면 '[object Object]' 문자열로 변환되어 덮어써진다
const obj = {};
obj[lee] = 'developer';
obj[kim] = 'designer';
console.log(obj); // { '[object Object]': 'designer' }
```

- **이터러블**이며 `keys()`, `values()`, `entries()`를 제공한다. 순회 순서는 추가된 순서

```js
for (const [key, value] of map) {
  console.log(key, value); // 'key1' 'value1' / {name: 'Lee'} 'developer'
}
```

### 정리 — 언제 무엇을 쓰는가

| 상황 | 선택 |
| --- | --- |
| 중복 제거, 존재 여부 확인(`has`)이 잦다 | Set |
| 키에 객체 등 문자열이 아닌 값을 써야 한다 / 추가·삭제·순회가 잦다 | Map |
| 고정된 형태의 데이터, JSON 직렬화가 필요하다 | 일반 객체 |

## 38장 브라우저의 렌더링 과정

### 브라우저의 렌더링 과정

- 자바스크립트는 브라우저에서 HTML, CSS와 함께 실행된다. 그래서 브라우저가 텍스트 문서를 **어떻게 해석해서 화면에 그리는지** 알아야 JS를 효율적으로 쓸 수 있다.
- **파싱**: 텍스트 문서를 토큰으로 쪼개고(어휘 분석), 문법적 의미와 구조를 반영해 트리 자료구조로 만드는 과정
- **렌더링**: HTML, CSS, JS 문서를 파싱해서 브라우저 화면에 시각적으로 출력하는 것

```text
HTML ──파싱──▶ DOM ─────┐
                        ├──▶ 렌더 트리 ──▶ 레이아웃 ──▶ 페인트
CSS  ──파싱──▶ CSSOM ───┘        ▲
                                 │ DOM API로 변경
JS   ──파싱──▶ AST ──▶ 바이트코드 ──▶ 실행
```

1. 렌더링에 필요한 리소스(HTML, CSS, JS, 이미지, 폰트)를 **요청**하고 **응답**받는다.
2. 렌더링 엔진이 HTML, CSS를 파싱해서 **DOM**, **CSSOM**을 만들고 **렌더 트리**로 결합한다.
3. JS 엔진이 JS를 파싱해서 **AST** → 바이트코드로 바꿔 실행한다. 이때 DOM API로 DOM/CSSOM을 바꿀 수 있고, 바뀌면 다시 렌더 트리로 결합된다.
4. 렌더 트리로 요소의 위치·크기를 계산(**레이아웃**)하고 화면에 그린다(**페인트**).

### 요청과 응답

- 주소창에 URL 입력 → 호스트 이름이 **DNS를 통해 IP 주소로 변환** → 해당 서버에 요청

```text
https://poiemaweb.com:443/docs/index.html?q=1#top
└─┬─┘   └─────┬──────┘└┬┘└──────┬───────┘└─┬─┘└┬┘
scheme       host    port     path      query fragment
```

- 경로 없이 루트(`/`)로 요청하면 서버는 보통 암묵적으로 `index.html`을 응답한다.
- 주소창만이 요청 수단은 아니다. JS로도 동적으로 요청할 수 있다. (43장 Ajax, 44장 REST API)
- HTML 파싱 중 `link`, `img`, `script` 같은 외부 리소스 태그를 만나면 **파싱을 잠시 멈추고** 해당 리소스를 요청한다.

### HTTP 1.1과 HTTP 2.0

- **HTTP/1.1**: 커넥션당 요청/응답 **1개**. 리소스가 많을수록 응답 시간도 그만큼 늘어난다.
- **HTTP/2**: 커넥션당 요청/응답 **여러 개**(다중화). HTTP/1.1보다 페이지 로드가 약 50% 빠르다고 알려져 있다.

```text
HTTP/1.1  [index.html]──▶[style.css]──▶[app.js]──▶[img.png]   ← 순차

HTTP/2    [index.html]
          [style.css ]  ← 한 커넥션에서 동시에
          [app.js    ]
          [img.png   ]
```

### HTML 파싱과 DOM 생성

- 서버가 응답한 HTML은 그냥 **문자열 텍스트**다. 화면에 그리려면 브라우저가 이해할 수 있는 **자료구조(객체)** 로 바꿔서 메모리에 올려야 한다. ⇒ 그게 **DOM**

```html
<!DOCTYPE html>
<html lang="ko">
  <head>
    <meta charset="UTF-8" />
    <link rel="stylesheet" href="style.css" />
  </head>
  <body>
    <ul>
      <li id="apple">Apple</li>
      <li id="banana">Banana</li>
      <li id="orange">Orange</li>
    </ul>
    <script src="app.js"></script>
  </body>
</html>
```

```text
바이트 ──▶ 문자 ──▶ 토큰 ──▶ 노드 ──▶ DOM

3c 62 6f 64 79 3e ...           ← 1. 바이트 (서버는 2진수로 응답)
<body>...                       ← 2. 문자 (meta charset 기준으로 인코딩, ex. UTF-8)
StartTag:body  Text  EndTag     ← 3. 토큰 (문법적 의미를 갖는 최소 단위)
[body 노드] [텍스트 노드]          ← 4. 노드 (토큰을 객체로 변환)
html ─ head ─ meta / link       ← 5. DOM (노드의 부자 관계를 트리로)
     └ body ─ ul ─ li × 3
            └ script
```

1. 서버가 HTML 파일을 **바이트**로 응답
2. `meta` 태그의 `charset`(ex. UTF-8) 기준으로 **문자열**로 변환
3. 문자열을 **토큰**으로 분해
4. 토큰을 객체로 바꿔 **노드** 생성 (문서 / 요소 / 어트리뷰트 / 텍스트 노드)
5. HTML 요소의 중첩 관계를 반영해 노드를 **트리**로 구성 ⇒ **DOM**

⇒ 정리하면 **DOM은 HTML 문서를 파싱한 결과물**이다.

### CSS 파싱과 CSSOM 생성

- HTML을 한 줄씩 파싱하다가 `link`나 `style` 태그를 만나면 **DOM 생성을 멈춘다.**
- CSS도 HTML과 같은 과정(바이트 → 문자 → 토큰 → 노드)을 거쳐 **CSSOM**을 만들고, 끝나면 멈췄던 지점부터 HTML 파싱을 이어간다.
- CSSOM은 CSS의 **상속을 반영**해서 만들어진다.

```css
body {
  font-size: 18px;
}
ul {
  list-style-type: none;
}
```

```text
body  { font-size: 18px }
 └ ul { font-size: 18px; list-style-type: none }   ← body의 font-size를 상속
    └ li { font-size: 18px; list-style-type: none }
```

### 렌더 트리 생성

- DOM + CSSOM ⇒ **렌더 트리**. 렌더링을 위한 트리라서 **화면에 그려지는 노드만** 남긴다.
  - `meta`, `script` 같이 화면에 안 그려지는 노드 제외
  - `display: none`인 노드 제외

```text
       DOM                      CSSOM
html                       body { font-size }
 ├ head ─ meta / link       └ ul { list-style }
 └ body                         └ li
    ├ ul ─ li × 3
    └ script
            \                /
             ▼              ▼
              렌더 트리
       body (font-size: 18px)
        └ ul (list-style: none)
           ├ li "Apple"
           ├ li "Banana"
           └ li "Orange"
   ← head, meta, link, script 는 제외
```

- 렌더 트리 → **레이아웃**(위치·크기 계산) → **페인트**(픽셀로 그리기)
- 이 과정은 한 번으로 끝나지 않는다. 아래 경우에 레이아웃과 페인트가 다시 실행된다.
  - JS로 노드 추가/삭제
  - 브라우저 창 리사이징
  - `width`, `height`, `margin`, `padding`, `display`, `position` 등 레이아웃에 영향을 주는 스타일 변경

⇒ 리렌더링은 **비용이 큰 작업**이다. 가능하면 자주 일어나지 않게 해야 한다.

### 자바스크립트 파싱과 실행

- DOM은 HTML의 구조만 담은 게 아니라, 요소와 스타일을 바꿀 수 있는 인터페이스인 **DOM API**도 제공한다. JS는 이걸로 DOM을 동적으로 조작한다.
- HTML 파싱 중 `script` 태그를 만나면 CSS 때와 마찬가지로 **DOM 생성을 멈추고 JS 엔진에 제어권을 넘긴다.** JS 실행이 끝나면 다시 렌더링 엔진으로 돌아와 멈춘 지점부터 이어간다.
- JS 파싱과 실행은 렌더링 엔진이 아니라 **JS 엔진**(V8, SpiderMonkey, JavaScriptCore 등)이 담당한다.
- 렌더링 엔진이 DOM/CSSOM을 만들듯, JS 엔진은 **AST**를 만들고 이를 바이트코드로 바꿔 실행한다.

```text
소스코드 ──▶ 토크나이징 ──▶ 파싱 ──▶ AST ──▶ 바이트코드 ──▶ 실행
```

**① 토크나이징** — 소스코드를 어휘 분석해서 **토큰**으로 분해

```js
var x = 1;
// → [var] [x] [=] [1] [;]
//    키워드 식별자 연산자 리터럴 구두점
```

**② 파싱** — 토큰을 구문 분석해서 **AST**(추상적 구문 트리) 생성

- AST는 엔진만 쓰는 게 아니다. TypeScript, Babel, Prettier 같은 도구도 AST로 동작한다. ([AST Explorer](https://astexplorer.net)에서 확인 가능)

```text
VariableDeclaration (kind: "var")
 └ VariableDeclarator
    ├ id: Identifier (name: "x")
    └ init: Literal (value: 1)
```

**③ 코드 생성과 실행** — AST를 **바이트코드**로 변환해서 인터프리터가 실행

- V8은 자주 쓰이는 코드를 터보팬(TurboFan)이 최적화된 머신 코드로 컴파일한다. 사용 빈도가 줄면 다시 디옵티마이징하기도 한다.

> 23장 실행 컨텍스트와 연결) 여기서 말하는 "실행"이 23장의 **소스코드 평가 → 소스코드 실행** 과정이다.

### 리플로우와 리페인트

- JS가 DOM API로 DOM/CSSOM을 바꾸면 렌더 트리에 다시 결합되고, 레이아웃과 페인트를 거쳐 다시 그려진다.
  - **리플로우**: 레이아웃 계산을 다시 하는 것. 노드 추가/삭제, 크기·위치 변경, 리사이징처럼 **레이아웃에 영향이 있을 때만** 발생
  - **리페인트**: 렌더 트리를 기반으로 다시 그리는 것
- 둘이 항상 같이 일어나는 건 아니다. 레이아웃과 상관없는 변경이면 **리페인트만** 일어난다.

```js
const $box = document.querySelector('.box');

// 레이아웃에 영향 O → 리플로우 + 리페인트
$box.style.width = '200px';
$box.style.margin = '10px';

// 레이아웃에 영향 X → 리페인트만
$box.style.color = 'red';
$box.style.backgroundColor = 'blue';
```

- 그러면 스타일을 바꿀 때마다 DOM/CSSOM을 처음부터 다시 만드는가? → **아니다.** 바뀐 요소와 영향받는 요소의 스타일만 다시 계산하고, 필요한 단계부터 다시 실행한다.

```text
box.style.width = '500px'
  → Style 재계산 → Layout(리플로우) → Paint(리페인트) → Composite
     └ box뿐 아니라 부모·형제의 위치까지 다시 계산될 수 있다

box.style.transform = 'translateX(100px)'
  → Style 재계산 → Composite   ← 조건이 맞으면 Layout·Paint를 건너뛴다
```

⇒ `transform`, `opacity`는 레이어를 **합성(composite)** 하는 단계만으로 처리될 수 있다. 그래서 애니메이션은 `top`/`left`보다 `transform`이 유리하다.

### 자바스크립트 파싱에 의한 HTML 파싱 중단

- 렌더링 엔진과 JS 엔진은 **병렬이 아니라 직렬로** 파싱한다. 위에서 아래로 순서대로 HTML, CSS, JS를 처리한다.
  ⇒ 그래서 **`script` 태그의 위치**가 중요하다. 위치에 따라 HTML 파싱이 막혀 DOM 생성이 늦어질 수 있다.

```html
<html>
  <head>
    <script>
      // 아직 아래 HTML을 파싱하지 않았다 → DOM에 #apple이 없다
      const $apple = document.getElementById('apple'); // null
      $apple.style.color = 'red'; // TypeError: Cannot read properties of null (reading 'style')
    </script>
  </head>
  <body>
    <ul>
      <li id="apple">Apple</li>
      <li id="banana">Banana</li>
      <li id="orange">Orange</li>
    </ul>
  </body>
</html>
```

- 해결: `script`를 **`body` 맨 아래**에 둔다.
  1. JS가 실행될 때 이미 DOM이 완성되어 있으니 DOM 조작 에러가 없다.
  2. JS 로드/실행 때문에 HTML 렌더링이 밀리지 않아 페이지 로딩이 빨라진다.

```html
<body>
  <ul>
    <li id="apple">Apple</li>
    <li id="banana">Banana</li>
    <li id="orange">Orange</li>
  </ul>
  <script>
    // DOM 생성이 끝난 뒤에 실행되므로 정상 동작
    const $apple = document.getElementById('apple');
    $apple.style.color = 'red';
  </script>
</body>
```

### script 태그의 async / defer 어트리뷰트

- 파싱이 막히는 문제를 근본적으로 해결하기 위해 HTML5부터 `async`, `defer`가 추가됐다.
- `src`로 **외부 JS 파일을 로드할 때만** 쓸 수 있다. 인라인 스크립트에는 못 쓴다.
- 둘 다 HTML 파싱과 JS 로드가 **동시에** 진행된다. 차이는 **JS를 언제 실행하느냐**다.

```html
<script async src="extern.js"></script>
<script defer src="extern.js"></script>
```

- **`async`**: 로드가 끝나는 **즉시 실행**. 실행하는 동안은 HTML 파싱이 멈춘다. 먼저 로드된 것부터 실행하므로 **순서 보장 X**
- **`defer`**: **HTML 파싱이 끝난 뒤** 실행(`DOMContentLoaded` 직전). 작성한 **순서대로** 실행 ⇒ DOM이 완성된 뒤 실행해야 하는 JS에 적합

```text
기본 <script>
HTML 파싱 ████████▌                      ▐████████
JS 로드           ▐▓▓▓▓▓▓▓▌
JS 실행                   ▐░░░░░░░░▌
                  └─ HTML 파싱 중단(블로킹) ─┘

<script async>
HTML 파싱 ████████████████▌        ▐██████████
JS 로드     ▐▓▓▓▓▓▓▓▓▓▓▓▓▓▌
JS 실행                   ▐░░░░░░░░▌  ← 로드 끝나면 바로 실행, 이때 파싱 중단

<script defer>
HTML 파싱 ██████████████████████████████████▌
JS 로드     ▐▓▓▓▓▓▓▓▓▓▓▓▓▓▌
JS 실행                                     ▐░░░░░░░░▌  ← 파싱 끝난 뒤 실행
```

| 방식 | JS 로드 | 실행 시점 | HTML 파싱 중단 | 순서 보장 |
| --- | --- | --- | --- | --- |
| 기본 `<script>` | 파싱 멈추고 로드 | 로드 완료 즉시 | O (로드 + 실행) | O |
| `<script async>` | 파싱과 동시에 | 로드 완료 즉시 | O (실행 동안만) | X |
| `<script defer>` | 파싱과 동시에 | HTML 파싱 완료 후 | X | O |

### 더 깊게 — 렌더링은 어떤 스레드에서 일어나는가

> 책 범위 밖. 42장(비동기 프로그래밍, 이벤트 루프)과 이어진다.

- 브라우저는 하나의 프로그램이 아니라 **여러 프로세스와 스레드**로 동작한다. (Chromium 기준으로 단순화)

```text
Browser Process ── UI, 네트워크, 탭 관리
Renderer Process (탭마다)
 ├ Main Thread        ← HTML 파싱, DOM, JS 실행, Style, Layout, Paint
 ├ Compositor Thread  ← 레이어 합성 (Composite)
 └ Raster Threads     ← 실제 픽셀 만들기
GPU Process ── 화면 출력
```

- **페이지의 JS와 렌더링(Style, Layout, Paint)은 같은 Main Thread를 나눠 쓴다.** JS가 오래 돌면 렌더링도, 클릭 처리도 멈춘다.
  ⇒ "JS는 싱글 스레드다"를 렌더링 관점에서 보면 이 뜻이다.

```js
button.onclick = () => {
  while (true) {} // Main Thread를 JS가 붙잡고 있다 → Style/Layout/Paint, 다음 클릭 모두 ❌
};
```

- 그럼 `fetch`, `setTimeout`은 멀티스레드인가? → **아니다.** JS는 브라우저에 일을 맡기기만 하고, 끝나면 콜백이 이벤트 루프를 통해 **다시 Main Thread에서 실행**된다. ⇒ **비동기 ≠ 병렬**
- JS를 진짜 병렬로 돌리려면 **Web Worker**. 개발자가 만드는 **별도의 JS 실행 환경**(별도 콜 스택)이다.
  - Main Thread의 JS가 멀티스레드가 되는 게 아니라, 싱글 스레드 JS 실행 환경을 **하나 더** 만드는 것
  - **DOM 직접 접근 불가.** DOM은 Main Thread가 관리하기 때문 ⇒ `postMessage`로 결과를 넘기고 DOM 변경은 Main Thread에서

```js
// main.js
const worker = new Worker('./worker.js');
worker.postMessage(data); // 무거운 계산은 Worker에게
worker.onmessage = (e) => render(e.data); // 결과를 받아 Main Thread에서 DOM 반영

// worker.js
onmessage = (e) => postMessage(heavyCalculation(e.data)); // document 접근 불가
```

| 구분 | 실행 위치 | 특징 |
| --- | --- | --- |
| 일반 JS, 이벤트 핸들러, 콜백 | Main Thread | 렌더링과 Main Thread를 나눠 쓴다 |
| `fetch`, 타이머 등 Web API | 브라우저 내부 | JS는 맡기기만, 콜백은 다시 Main Thread에서 |
| Web Worker | 별도 Worker Thread | JS 병렬 실행 가능, DOM 접근 불가, `postMessage`로 통신 |

### 38장 핵심 흐름

```text
요청/응답      → HTTP/1.1은 순차, HTTP/2는 다중 요청/응답
HTML → DOM    → 바이트 → 문자 → 토큰 → 노드 → 트리
CSS  → CSSOM  → 같은 과정 + 상속 반영
DOM + CSSOM   → 렌더 트리 (화면에 그려지는 노드만) → 레이아웃 → 페인트
JS            → 파싱 중단 후 JS 엔진에 제어권 → 토크나이징 → AST → 바이트코드 → 실행
DOM 변경      → 바뀐 부분부터 재계산 → 리플로우 / 리페인트 (transform, opacity는 합성만)
script 위치   → 파싱이 직렬이라 중요 → body 맨 아래 or defer / async
Main Thread   → JS와 렌더링이 공유 → 무거운 JS는 Web Worker로 (비동기 ≠ 병렬)
```
