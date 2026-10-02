# 35~38ch 스프레드 · 디스트럭처링 · Set/Map · 브라우저 렌더링

- 35~37장은 React에서 **매일 쓰는 문법들의 정체**
- 38장은 그 코드가 **화면이 되기까지의 과정** = React가 왜 존재하는지에 대한 답

## 들어가기 전에,, 네 장을 어떻게 묶어볼까?

책 내용을 그대로 옮기기보다는, React 쓰면서 한 번쯤 마주쳤을 법한 이야기 위주로 정리해봄

|  | 35~37장 | 38장 |
| --- | --- | --- |
| 성격 | **매일 쓰는 문법의 정체** | **코드가 화면이 되기까지** |
| React 연결 | 불변 업데이트, props, state | Virtual DOM, CSR/SSR, 빌드 결과물 |
| 재밌는 포인트 | 매일 쓰는데 왜 이렇게 생겼는지 몰랐던 것 | React는 왜 만들어졌을까 |

이번에 주로 공유하고 싶었던 건 결국
**"스프레드·디스트럭처링·Set/Map은 React 불변 업데이트의 도구이고, 그 도구가 왜 필요한지는 38장 렌더링 과정이 설명해준다"** 는 것

### 앞서 우리가 학습한 것과 어떻게 연계를 하면 좋을까?

| 35~38장의 내용 | 연결되는 기존 개념 |
| --- | --- |
| 스프레드는 이터러블만 가능 | **34장 이터러블** |
| 스프레드가 push, concat을 대체 | **27장** 저자의 "스프레드가 낫다" 예고 회수 |
| 스프레드는 얕은 복사 | **11장** 원시값과 객체 (참조) |
| Set, Map을 mutate하면 리렌더 X | **27장** mutator vs accessor |
| 디스트럭처링 기본값은 undefined일 때만 | **9장** `??` (null 병합) |
| Map은 프로토타입 오염이 없음 | **19장** `Object.create(null)` |
| WeakMap으로 메모리 누수 방지 | **24장** 클로저와 메모리 |
| 리플로우가 비싸다 | React Virtual DOM이 생긴 이유 |

---

## 35ch 스프레드 - React 불변 업데이트의 주인공

27장에서 저자가 push, unshift, concat 설명할 때마다 "스프레드 문법이 낫다"고 계속 예고했었는데, 그게 여기서 회수됨ㅎ

### 🔥 얕은 복사의 함정 - 이전 state까지 바뀌어버림

```javascript
const [user, setUser] = useState({
  name: 'Lee',
  address: { city: 'Seoul' },
});

// ❌ 이러면?
const next = { ...user };
next.address.city = 'Busan';
setUser(next);
```

- 리렌더는 되는데, **이전 state까지 같이 바뀌어버림**
- 스프레드는 **1단계만 복사**하기 때문 → address는 같은 객체를 가리킴

```javascript
next === user;                  // false ← 바깥은 새 객체
next.address === user.address;  // true  ← 안쪽은 공유!
```

- 올바른 방법은 바뀌는 경로를 전부 펼치기

```javascript
setUser({
  ...user,
  address: { ...user.address, city: 'Busan' },
});
```

> 💡 중첩이 3단계면 스프레드도 3번…?
> 이게 너무 귀찮아서 나온 게 **Immer**임
>
> ```javascript
> setUser(produce(draft => { draft.address.city = 'Busan'; }));
> ```
>
> → Redux Toolkit이 Immer를 기본으로 내장한 이유도 같은 맥락! 불변성은 지키고 싶은데 문법이 너무 장황하니까

### 🔥 엥 왜 `[...obj]`는 에러인데 `{...obj}`는 됨?

```javascript
const obj = { a: 1 };

const arr = [...obj];   // ❌ TypeError: obj is not iterable
const copy = { ...obj }; // ✅
```

- 같은 `...`인데 사실 **다른 문법**임
  - `[...x]`, `f(...x)` → **이터러블**만 가능 (34장 이터레이션 프로토콜)
  - `{...x}` → **객체 스프레드**, 별도 제안으로 나중에 추가됨 (ES2018)
- 그래서 일반 객체는 배열이나 함수 인수 안에서는 펼칠 수 없음

---

## 36ch 디스트럭처링 - props와 useState의 정체

### 🔥 useState는 왜 배열을 반환할까?

```javascript
const [count, setCount] = useState(0);
```

만약 객체를 반환했다면?

```javascript
const { state: count, setState: setCount } = useState(0);
const { state: name, setState: setName } = useState('');  // 매번 이름을 바꿔줘야 함;
```

- 배열 디스트럭처링은 **위치(인덱스) 기준**이라 이름을 마음대로 지을 수 있음
- 같은 훅을 여러 번 쓰는 구조에 딱 맞는 것

<details>
<summary>그러면 커스텀 훅은 배열? 객체?</summary>

- 반환값이 2개이고 순서가 명확하면 → 배열 (`[value, setValue]`)
- 반환값이 많거나 일부만 골라 써야 하면 → 객체 (`{ data, error, isLoading }`)
- 객체 디스트럭처링은 프로퍼티 키 기준이라 순서 상관없이 필요한 것만 꺼낼 수 있음!

</details>

### 🔥 기본값이 안 먹히는 경우

```javascript
function Button({ size = 'md' }) { ... }

<Button size={undefined} />  // 'md' ✅
<Button size={null} />       // null ❌ 기본값 무시됨!
```

- 디스트럭처링 기본값은 **undefined일 때만** 적용됨
- API 응답이 null을 주면 기본값이 안 들어가서 버그가 남
- 9장의 `??`와 비교해보면 → `??`는 null도 걸러줌

```javascript
const size = props.size ?? 'md';  // null, undefined 둘 다 'md'
```

### 🔥 props 나머지 전달

```javascript
function Input({ label, error, ...rest }) {
  return <input {...rest} />;  // 나머지는 그대로 input에
}
```

- 36장 rest + 35장 스프레드가 한 줄에 다 들어있음
- 래퍼 컴포넌트 만들 때 매일 쓰는 패턴

---

## 37ch Set과 Map - React state로 쓰면 함정 투성이

### 🔥 Set을 state로 쓰면 리렌더가 안 됨

```javascript
const [selected, setSelected] = useState(new Set());

// ❌
selected.add(id);
setSelected(selected);  // 리렌더 안 됨
```

엥 분명 add로 넣었는데 왜?;

- `add`는 **같은 Set 안에 넣기만** 함 → 내용은 바뀌었는데 **주소는 그대로**
- React는 Set 안을 들여다보지 않고 **주소만 비교** (`Object.is`)

```plain text
React의 판단:
  이전 값: 0xA1
  새 값:   0xA1
  → "안 바뀌었네" → 리렌더 스킵
```

- 비유하면 상자에 물건을 넣고 **같은 상자를 다시 건네주는 것** → React는 "아까 그 상자네?" 하고 열어보지도 않음

```javascript
// ✅ 새 상자를 주기
setSelected(prev => new Set(prev).add(id));
```

- 사실 27장 배열 push 문제와 완전히 똑같음

```javascript
items.push(4);     setItems(items);        // ❌
selected.add(id);  setSelected(selected);  // ❌ ← 같은 실수
```

> 💡 delete는 함정이 하나 더 있음
>
> ```javascript
> // ❌ delete는 Set이 아니라 boolean을 반환함!
> setSelected(prev => new Set(prev).delete(id));  // state가 true/false가 됨
>
> // ✅
> setSelected(prev => {
>   const next = new Set(prev);
>   next.delete(id);
>   return next;
> });
> ```
>
> add는 Set 자신을 반환해서 체이닝이 되지만, delete는 true/false를 반환함

<details>
<summary>그럼 왜 굳이 Set을 쓸까?</summary>

- 체크박스 다중 선택에서 `selected.has(id)`는 O(1)
- 배열이면 `selected.includes(id)` → 항목마다 배열 전체를 훑음 (O(n))
- 목록이 1000개면 렌더마다 최대 100만 번 비교…
- 다만 항목이 수십 개 수준이면 배열이 더 단순할 수도 있음!

</details>

### 🔥 Map을 localStorage에 저장하면?

```javascript
const cache = new Map([['a', 1]]);
JSON.stringify(cache);  // '{}'  ← 텅 비었다!
```

- Map과 Set은 **JSON 직렬화가 안 됨**
- persist 미들웨어 쓰다가 새로고침하면 데이터가 날아가는 실무 사고 포인트

```javascript
JSON.stringify([...cache]);   // '[["a",1]]'  ← 배열로 바꿔서 저장
new Map(JSON.parse(str));     // 복원
```

### 🔥 객체는 순서를 보장할까?

```javascript
const obj = { b: 1, a: 2, 1: 3 };
Object.keys(obj);  // ['1', 'b', 'a']  ← 숫자 키가 앞으로!

const map = new Map([['b', 1], ['a', 2], [1, 3]]);
[...map.keys()];   // ['b', 'a', 1]   ← 넣은 순서 그대로
```

- 객체는 **정수처럼 생긴 키를 먼저 정렬**함
- ID를 키로 쓰는 객체를 순회하면 순서가 바뀌는 버그가 날 수 있음 → 순서가 중요하면 Map

### 그래서 언제 Map, Set을 쓰면 좋을까?

| 원하는 것 | 선택 |
| --- | --- |
| 이게 있나 없나만 빠르게 알고 싶다 | **Set** |
| 중복 없이 모으고 싶다 | **Set** (`[...new Set(arr)]`) |
| 키가 문자열이 아니거나 자주 추가·삭제된다 | **Map** |
| 사용자 입력을 키로 쓴다 | **Map** (프로토타입 오염 X) |
| 고정된 모양의 데이터 (name, age 같은) | **객체** |
| 서버에 보내거나 저장해야 한다 | **객체 / 배열** |

### 그래서 실무에서는 어디에 쓰일까?

#### Set

- **중복 제거** — 가장 흔한 관용구 (35장 스프레드 + 37장 Set)

```javascript
const tags = ['react', 'js', 'react', 'css'];
const unique = [...new Set(tags)];  // ['react', 'js', 'css']
```

- **체크박스 다중 선택** — `selected.has(id)`로 바로 확인 (위 토글 참고)
- **무한 스크롤 중복 아이템 방지** — 다음 페이지 응답에 이전 아이템이 섞여 올 때

```javascript
const seenIds = new Set();
const newItems = fetched.filter(item => {
  if (seenIds.has(item.id)) return false;
  seenIds.add(item.id);
  return true;
});
```

- **권한·기능 플래그 확인**

```javascript
const permissions = new Set(['read', 'write']);
if (permissions.has('delete')) { ... }
```

#### Map

- **객체를 키로 쓰고 싶을 때** — 이건 Map만 가능함

```javascript
const meta = new Map();
const el = document.querySelector('#btn');
meta.set(el, { clicked: 0 });  // DOM 노드 자체가 키

const obj = {};
obj[el] = 1;  // ❌ 키가 '[object HTMLButtonElement]' 문자열로 바뀜
```

- 객체의 키는 문자열로 변환돼서, 서로 다른 버튼 요소가 전부 같은 키가 되어버림;
- **API 응답 캐시** — 삽입 순서가 보장되니까 오래된 것부터 지우는 LRU도 쉽게 만들 수 있음

```javascript
const cache = new Map();

async function fetchUser(id) {
  if (cache.has(id)) return cache.get(id);
  const data = await api.get(`/users/${id}`);
  cache.set(id, data);
  if (cache.size > 100) cache.delete(cache.keys().next().value);  // 가장 먼저 넣은 것 삭제
  return data;
}
```

- **사용자 입력을 키로 쓸 때** — 19장 프로토타입 오염 얘기

```javascript
const obj = {};
obj['constructor'];      // ƒ Object() ← 상속받은 게 튀어나옴

const map = new Map();
map.get('constructor');  // undefined ← 깨끗함
```

<details>
<summary>+ WeakMap은 언제?</summary>

- 일반 Map은 키를 붙잡고 있어서, DOM 노드가 화면에서 제거돼도 메모리에 계속 남음
- WeakMap은 키를 약하게 참조 → 노드 참조가 사라지면 항목도 자동으로 GC

```javascript
const clickCount = new WeakMap();
clickCount.set(el, 0);
// el이 DOM에서 제거되고 참조가 사라지면 → WeakMap 항목도 자동으로 GC
```

- 24장 클로저 메모리 누수와 같은 축!

</details>

### 반대로, 안 쓰는 게 나은 경우 (React 포함)

- **JSON으로 주고받거나 저장하는 데이터** → 직렬화가 안 됨 (위 localStorage 예시)
- **React state인데 항목이 적을 때** → 매번 `new Set(prev)`로 복사해야 해서 불변 업데이트가 번거로움. 수십 개 수준이면 배열이 코드도 더 단순함
- **고정된 구조의 레코드** → `{ name, age }` 같은 데이터는 그냥 객체가 자연스러움

> 💡 정리하면, React에서 Set/Map을 state로 쓰는 건 "조회가 잦고 항목이 많을 때"(체크박스 다중 선택 같은) 이득이 크고, 그 외에는 배열·객체가 더 단순한 경우가 많음
> → 판단 기준: "이게 있나 없나"를 자주 묻는가? 키가 문자열이 아닌가? 서버에 보내야 하는가?

> 💡 참고) 책에서는 교집합·합집합을 직접 구현하는데, 2024년부터 주요 브라우저가 내장 메서드를 지원함 (ES2025 표준)
>
> ```javascript
> const a = new Set([1, 2, 3]), b = new Set([2, 3, 4]);
> a.intersection(b);  // Set {2, 3}
> a.union(b);         // Set {1, 2, 3, 4}
> a.difference(b);    // Set {1}
> ```

---

## 38ch 브라우저 렌더링 - React는 왜 존재할까?

38장을 React 관점으로 읽으면 완전히 다른 장이 됨!

```plain text
HTML 파싱 → DOM
CSS 파싱  → CSSOM
              ↓
         렌더 트리
              ↓
     레이아웃 (리플로우)   ← 위치·크기 계산. 비쌈
              ↓
       페인트 (리페인트)    ← 픽셀 칠하기
```

### 🔥 Virtual DOM은 정말 빠를까?

```javascript
// DOM을 직접 1000번 건드리면 → 리플로우 위험
for (let i = 0; i < 1000; i++) list.appendChild(item);
```

- 리플로우가 비싸다는 게 React가 Virtual DOM을 만든 출발점
- 변경을 모아서 한 번에 반영하자!

> 💡 그런데 Svelte 만든 Rich Harris는 "Virtual DOM is pure overhead"라는 글을 씀
>
> - 잘 짠 직접 조작보다 **빠를 수는 없음** (비교 작업이 하나 더 추가되니까)
> - Virtual DOM의 진짜 가치는 "개발자가 신경 안 써도 적당히 빠르게" + 선언적 코드
>
> → 즉 **속도가 아니라 생산성의 기술**에 가까움

### 🔥 CSR 첫 화면은 왜 하얄까?

```html
<!-- Vite로 만든 React 앱의 index.html -->
<body>
  <div id="root"></div>   <!-- 이게 전부 -->
  <script type="module" src="/main.jsx"></script>
</body>
```

38장 흐름대로 따라가보면

```plain text
① HTML 받음 → <div id="root"></div>  ← 빈 화면
② JS 다운로드 대기 ...
③ JS 파싱·실행 ...
④ React가 DOM 생성
⑤ 그제야 화면 표시
```

- ①~④ 동안 하얀 화면 → 이게 **Next.js(SSR)가 존재하는 이유** 중 하나
- 서버에서 HTML을 완성해서 보내면 ①에서 바로 내용이 보임
- SSR을 "SEO 때문"이라고만 알았는데, 38장 관점에선 **첫 렌더링까지의 시간** 문제이기도 함

### 🔥 애니메이션에 top 쓰면 안 되는 이유

```css
.box { transition: top 0.3s; }        /* ❌ 매 프레임 리플로우 */
.box { transition: transform 0.3s; }  /* ✅ 레이아웃을 다시 안 함 */
```

- top, left, width 변경 → 레이아웃부터 다시 (리플로우 + 리페인트)
- transform, opacity 변경 → 레이아웃을 건너뛰고 합성(composite) 위주로 처리
- 38.7 리플로우/리페인트를 알면 왜 transform을 쓰라는지 이유가 보임

---

### 여기서 알아두면 좋을 것들

- 스프레드는 **얕은 복사** → 중첩된 객체는 바뀌는 경로마다 펼쳐야 함
- Set, Map도 결국 객체 → `add`, `set`, `delete`는 원본을 바꾸는 mutator
- React는 **주소만 비교**함 → 새 객체·새 배열·새 Set을 만들어야 리렌더
- Map, Set은 JSON 직렬화가 안 됨
- 디스트럭처링 기본값은 undefined일 때만 → null 대비는 `??`
- `type="module"` 스크립트는 defer처럼 동작

## 같이 얘기해보고 싶은 것

- Virtual DOM은 정말 빠른 걸까? 그럼 React를 쓰는 진짜 이유는 뭘까?
- 중첩 state 업데이트, Immer를 쓰시나요? 아니면 state를 평탄하게 설계하시나요?
- 커스텀 훅은 배열을 반환해야 할까, 객체를 반환해야 할까?
