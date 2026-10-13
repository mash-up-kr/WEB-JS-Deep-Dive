## 26장 ES6 함수의 추가 기능

### 함수의 구분

- ES6 이전에는 모든 함수가 **일반 함수로도, 생성자 함수로도 호출 가능** (callable + constructor)
    - 객체에 바인딩된 메서드나 콜백 함수도 `new`로 호출할 수 있었음
    - 호출될 일 없는 prototype 객체까지 생성하니까 **성능상 손해**, 실수 가능성도 있음
    (메모리 낭비, 생성 비용과 GC 부담)
- ES6에서 사용 목적에 따라 3가지로 명확히 구분

| ES6 함수 구분 | constructor | prototype | super | arguments |
| --- | --- | --- | --- | --- |
| 일반 함수 | O | O | X | O |
| 메서드 | X | X | O | O |
| 화살표 함수 | X | X | X | X |
- 일반 함수는 함수 선언문이나 함수 표현식으로 정의한 함수 → ES6 이전 함수와 동일

### 메서드

- ES6 사양에서 메서드는 **메서드 축약 표현으로 정의된 함수만** 의미

```jsx
const obj = {
  foo() { return 'foo'; },           // 메서드 O
  bar: function () { return 'bar'; } // 일반 함수 (메서드 X)
};

new obj.foo(); // TypeError
new obj.bar(); // bar {}
```

- **non-constructor** → `new`로 호출 불가
- prototype 프로퍼티가 없고 프로토타입도 생성하지 않음
- 자신을 바인딩한 객체를 가리키는 내부 슬롯 **`[[HomeObject]]`** 를 가짐 → `super` 참조 가능
- 그래서 메서드를 정의할 땐 프로퍼티에 익명 함수를 할당하지 말고 **메서드 축약 표현** 사용 권장

### 화살표 함수

#### 정의 문법

- 함수 표현식으로만 정의 가능 (함수 선언문 X)
- 매개변수가 1개면 소괄호 생략 가능, 없으면 `()` 생략 불가
- 몸체가 **하나의 표현식**이면 중괄호 생략 가능 → 그 값이 **암묵적으로 반환**
    - 표현식이 아닌 문(ex. `const x = 1`)은 중괄호 생략 불가
- **객체 리터럴을 반환하려면 소괄호로 감싸야 함**

```jsx
const create = (id, content) => ({ id, content });
```

- 즉시 실행 함수로도 사용 가능
- 고차 함수에 콜백으로 넘길 때 간결해서 유용

#### 일반 함수와의 차이

- **non-constructor** → `new` 호출 불가, prototype 프로퍼티 없음
- **중복된 매개변수 이름 선언 불가** (일반 함수는 non-strict에서 가능)
- 함수 자체의 **this, arguments, super, new.target 바인딩이 없음**
    - 참조하면 스코프 체인을 통해 **상위 스코프의 것**을 참조

#### this

- 화살표 함수가 등장한 주된 이유가 **콜백 함수 내부의 this 문제** 해결
    - 일반 함수로 호출되는 콜백의 this는 전역 객체(클래스 안이면 strict라 undefined)라서 메서드의 this와 달라짐
    - ES6 이전 해결법: `that = this`, `map` 등의 두 번째 인수(thisArg), `bind`
- 화살표 함수의 this = **상위 스코프의 this** → **렉시컬 this**
    - 정의된 위치에 의해 this가 결정됨
- 화살표 함수끼리 중첩되면 스코프 체인상 가장 가까운 **화살표 함수가 아닌 함수**의 this 참조
- 전역에서 정의하면 this는 전역 객체
- `call`, `apply`, `bind`로 **this를 교체할 수 없음** (호출 자체는 가능)
- 주의
    - **메서드를 화살표 함수로 정의하지 말 것** → this가 객체가 아니라 상위 스코프(전역) 가리킴
    - **프로토타입에 화살표 함수 할당도 X**, 같은 이유
    - 클래스 필드에 화살표 함수를 할당하면 this는 인스턴스를 가리키긴 하지만, 프로토타입 메서드가 아니라 **인스턴스 메서드**가 됨 → 메서드 축약 표현 권장

```jsx
const person = {
  name: 'Lee',
  sayHi: () => console.log(`Hi ${this.name}`) // this는 전역 객체
};
```

#### super

- 자체 super 바인딩 없음 → 상위 스코프의 super 참조
- 클래스 필드에 할당한 화살표 함수 안의 super는 constructor의 super를 가리킴

#### arguments

- 자체 arguments 바인딩 없음 → 상위 스코프의 arguments 참조
- 자기 자신에게 전달된 인수 목록을 확인할 수 없음 → 가변 인자 함수가 필요하면 **Rest 파라미터** 사용

### Rest 파라미터

```jsx
function foo(param, ...rest) {
  console.log(param); // 1
  console.log(rest);  // [2, 3, 4]
}
foo(1, 2, 3, 4);
```

- 매개변수 이름 앞에 `...` → 전달된 인수 목록을 **배열로** 받음
- 일반 매개변수와 같이 쓰면 앞에서부터 순서대로 할당되고 나머지가 Rest 파라미터로
- **반드시 마지막**에 와야 하고, **단 하나만** 선언 가능
- 함수의 `length` 프로퍼티에 영향 없음
- arguments와 비교
    - arguments는 **유사 배열 객체**라 배열 메서드를 쓰려면 `call`/`apply`로 변환 필요
    - Rest 파라미터는 **진짜 배열**이라 바로 배열 메서드 사용 가능
    - 화살표 함수는 arguments가 없으니 가변 인자를 받으려면 Rest 파라미터 필수

### 매개변수 기본값

- 인수를 덜 전달해도 에러가 안 나고, 전달되지 않은 매개변수는 `undefined`
    - ES6 이전엔 `a = a || 0` 같은 방어 코드가 필요했음

```jsx
function sum(x = 0, y = 0) {
  return x + y;
}
sum(1); // 1
```

- 기본값은 **인수를 전달하지 않았거나 `undefined`를 전달한 경우에만** 적용 (`null`은 적용 X)
- Rest 파라미터에는 기본값 지정 불가
- 기본값은 함수의 `length`와 `arguments` 객체에 영향 없음

---

## 27장 배열

### 배열이란

- 여러 개의 값을 순차적으로 나열한 자료구조
- **요소**: 배열이 가진 값, 자바스크립트의 모든 값이 요소가 될 수 있음
- **인덱스**: 요소의 위치, 0 이상의 정수
- **length 프로퍼티**: 요소의 개수
- 배열이라는 타입은 없음 → **배열은 객체** (`typeof [] === 'object'`)
- 생성 방법: 배열 리터럴, `Array` 생성자 함수, `Array.of`, `Array.from`
- 일반 객체와의 차이

| 구분 | 객체 | 배열 |
| --- | --- | --- |
| 구조 | 프로퍼티 키와 값 | 인덱스와 요소 |
| 값의 참조 | 프로퍼티 키 | 인덱스 |
| 값의 순서 | X | O |
| length | X | O |
- 값의 순서와 length가 있어서 **순차적으로, 역순으로, 특정 위치부터** 접근 가능

### 자바스크립트 배열은 배열이 아니다

- 일반적인 자료구조의 배열 = **밀집 배열 (dense array)**
    - 동일한 크기의 메모리 공간이 빈틈없이 연속으로 나열
    - 인덱스로 한 번에 접근 가능 → 임의 접근 **O(1)**
    - 대신 정렬 안 된 배열에서 값 검색은 O(n), 요소 삽입/삭제 시 뒤 요소들을 이동해야 함
- 자바스크립트 배열
    - 요소의 메모리 공간이 **같은 크기가 아니어도 되고, 연속적이지 않을 수도 있음** → **희소 배열 (sparse array)**
    - 정확히는 **일반적인 배열의 동작을 흉내 낸 특수한 객체**
    - 인덱스를 나타내는 문자열을 프로퍼티 키로, length 프로퍼티를 갖는 객체
- 성능 차이
    - 해시 테이블로 구현된 객체라 **인덱스 접근은 일반 배열보다 느림**
    - 대신 **요소 삽입/삭제는 일반 배열보다 빠름**
    - 엔진이 배열을 일반 객체와 구별해서 더 배열처럼 동작하도록 최적화해 둠

### length 프로퍼티와 희소 배열

- 0 이상의 정수, 최대값은 2³² - 1
- 요소를 추가/삭제하면 자동 갱신
- length에 명시적으로 값을 할당할 수 있음
    - 현재보다 **작은 값** → 배열이 실제로 줄어듦
    - 현재보다 **큰 값** → length만 바뀌고 실제 배열 길이는 안 늘어남 (`empty`로 표시될 뿐, 메모리 확보 X)
- 희소 배열은 length와 실제 요소 개수가 일치하지 않음 (length ≥ 실제 요소 개수)
- **희소 배열은 사용하지 말 것**, 배열에는 **같은 타입의 요소를 연속적으로** 두는 게 최선

### 배열 생성

#### 배열 리터럴

- `[1, 2, 3]`, 가장 일반적
- `[1, , 3]`처럼 요소를 생략하면 희소 배열

#### Array 생성자 함수

- 인수에 따라 동작이 다름
    - 숫자 1개 → **그 값을 length로 갖는 희소 배열** (`new Array(10)`)
        - 0 ~ 2³² - 1 범위를 벗어나면 RangeError
    - 인수 없음 → 빈 배열
    - 인수 2개 이상 또는 숫자가 아닌 1개 → 인수를 요소로 갖는 배열
- `new` 없이 호출해도 동일하게 동작

#### Array.of (ES6)

- 인수를 **요소로** 갖는 배열 생성
- 숫자 1개를 넘겨도 요소로 취급 → `Array.of(1)` → `[1]`

#### Array.from (ES6)

- **유사 배열 객체 또는 이터러블 객체**를 배열로 변환

```jsx
Array.from({ length: 2, 0: 'a', 1: 'b' }); // ['a', 'b']
Array.from('Hello');                        // ['H', 'e', 'l', 'l', 'o']
Array.from({ length: 3 }, (_, i) => i);     // [0, 1, 2]
```

- 두 번째 인수로 콜백을 넘기면 콜백 반환값으로 요소를 채움
- 참고
    - **유사 배열 객체**: 인덱스로 접근 가능하고 length를 가진 객체 → for 문으로 순회 가능
    - **이터러블 객체**: `Symbol.iterator`를 구현한 객체 → for...of, 스프레드, 디스트럭처링 가능 (Array, String, Map, Set, DOM 컬렉션, arguments 등)

### 요소의 참조, 추가, 갱신, 삭제

- 참조: 대괄호 안에 인덱스
    - 존재하지 않는 요소 참조 → `undefined` (객체에 없는 프로퍼티 참조와 같은 원리)
- 추가/갱신
    - 없는 인덱스에 할당하면 요소 추가, length 자동 갱신
    - 현재 length보다 큰 인덱스에 할당하면 희소 배열이 됨
    - 이미 있는 인덱스에 할당하면 갱신
    - 인덱스는 정수여야 함 (정수 형태 문자열도 가능)
    - 정수가 아닌 값을 인덱스처럼 쓰면 **요소가 아니라 프로퍼티가 생성**되고 length에 영향 없음
- 삭제
    - 배열도 객체라 `delete` 가능하지만 **희소 배열이 됨** (length 그대로)
    - 그러니 `delete` 말고 **`splice`** 사용

### 배열 메서드

- 크게 두 종류
    - **원본 배열을 직접 변경**하는 메서드 (mutator)
    - 원본은 그대로 두고 **새로운 배열을 반환**하는 메서드 (accessor)
- 원본을 변경하는 건 부수 효과가 있으니 **가급적 원본을 변경하지 않는 메서드** 사용
    - 특히 React state는 원본을 바꾸면 변경 감지가 안 되니까 이 구분이 실제로 중요함

#### 원본 변경 여부 한눈에

- 원본 변경: `push`, `pop`, `unshift`, `shift`, `splice`, `reverse`, `fill`, `sort`
- 새 배열 반환: `concat`, `slice`, `flat`, `map`, `filter`, `flatMap`
- 참고: `toReversed`, `toSorted`, `toSpliced`, `with`는 원본 변경 메서드의 **새 배열 반환 버전**

#### Array.isArray

- 배열이면 true (`Array.isArray({ 0: 1, length: 1 })` → false)
- 활용: 배열일 수도, 단일 값일 수도 있는 데이터를 배열로 통일

```jsx
const toArray = (value) => (Array.isArray(value) ? value : [value]);

toArray('a');        // ['a']
toArray(['a', 'b']); // ['a', 'b']
```

#### indexOf

- 인수로 전달한 요소의 **인덱스** 반환, 여러 개면 첫 번째 것
- 없으면 **1**
- 두 번째 인수로 검색 시작 인덱스 지정
- `NaN` 포함 여부는 확인 불가
- 포함 여부만 확인할 거면 ES7의 **`includes`*가 가독성이 더 좋음
- 활용: 값 자체보다 **위치**가 필요할 때

```jsx
// 현재 탭 기준으로 다음 탭 구하기
const tabs = ['home', 'search', 'mypage'];
const next = tabs[(tabs.indexOf(current) + 1) % tabs.length];
```

#### push / pop

- `push`: 마지막에 요소 추가, **변경된 length 반환**, 원본 변경
    - `arr[arr.length] = x`가 더 빠름
    - 원본을 안 건드리려면 스프레드 `[...arr, x]` 권장
- `pop`: 마지막 요소 **제거 후 반환**, 빈 배열이면 undefined, 원본 변경
- push + pop → **스택** 구현
- 활용: 되돌리기(undo), 방문 기록처럼 **마지막에 넣은 걸 먼저 꺼내는** 구조

```jsx
const history = [];
history.push(state);             // 변경 전 상태 저장
const prevState = history.pop(); // 되돌리기

// React state에서는 push 대신
setItems(prev => [...prev, newItem]);
```

#### unshift / shift

- `unshift`: 선두에 요소 추가, 변경된 length 반환, 원본 변경 → 스프레드 `[x, ...arr]` 권장
- `shift`: 첫 요소 제거 후 반환, 빈 배열이면 undefined, 원본 변경
- push + shift → **큐** 구현
- 활용: 토스트 알림, 업로드 대기열처럼 **먼저 들어온 걸 먼저 처리**하는 구조

```jsx
const queue = [];
queue.push(task);              // 대기열 뒤에 추가
const current = queue.shift(); // 앞에서 하나 꺼내 처리
```

- shift/unshift는 나머지 요소 인덱스를 전부 당기거나 밀어야 해서 큰 배열에선 느림

#### concat

- 인수로 전달된 값을 마지막에 추가한 **새 배열** 반환, 원본 변경 X
- 인수가 배열이면 **해체해서** 요소로 추가 (push/unshift는 배열을 그대로 하나의 요소로 추가)
- 스프레드 문법으로 대체 가능 → 일관성 있게 스프레드 권장
- 활용: 무한 스크롤에서 다음 페이지 데이터 이어 붙이기

```jsx
setProducts(prev => prev.concat(nextPage));
// 또는
setProducts(prev => [...prev, ...nextPage]);
```

#### splice

- `splice(start, deleteCount, ...items)`, **원본 변경**
    - `start`: 제거 시작 인덱스 (음수면 뒤에서부터)
    - `deleteCount`: 제거할 개수, 0이면 제거 없이 삽입만, 생략하면 start부터 끝까지 제거
    - `items`: 제거한 자리에 삽입할 요소
- **제거한 요소들의 배열**을 반환
- 특정 요소 하나 제거: `indexOf`로 인덱스 찾고 `splice`
- 중복된 요소 전부 제거: `filter` 사용
- 활용: 드래그 앤 드롭으로 **순서 변경**, 원본 보호하려고 복사본에 적용

```jsx
const reorder = (list, from, to) => {
  const copy = [...list];
  const [moved] = copy.splice(from, 1); // 빼고
  copy.splice(to, 0, moved);            // 원하는 위치에 끼워 넣기
  return copy;
};
```

#### slice

- `slice(start, end)`: start부터 **end 직전까지** 복사한 새 배열 반환, 원본 변경 X
- 음수면 뒤에서부터, end 생략하면 끝까지
- 인수를 모두 생략하면 원본의 복사본 반환 → **얕은 복사**
- 유사 배열 객체를 배열로 변환할 때 쓰였음 (`Array.prototype.slice.call(arguments)`) → 지금은 `Array.from`이 간편
- 활용: 페이지네이션, 개수 제한, 최근 N개

```jsx
// 페이지네이션
const pageItems = items.slice((page - 1) * size, page * size);

// 최근 검색어: 중복 제거 + 맨 앞 추가 + 최대 10개
const updated = [keyword, ...recent.filter(k => k !== keyword)].slice(0, 10);

// 마지막 3개
logs.slice(-3);
```

#### join

- 모든 요소를 문자열로 변환 후 구분자로 연결한 문자열 반환
- 구분자 기본값은 콤마 `','`
- 활용: 조건부 className, 경로 조합, 태그 나열

```jsx
const className = ['btn', isActive && 'active', size].filter(Boolean).join(' ');

const path = ['api', 'users', userId].join('/'); // 'api/users/42'

tags.join(', '); // 'react, js, css'
```

#### reverse

- 순서를 뒤집음, **원본 변경**, 뒤집힌 배열 반환
- 활용: 오래된 순으로 온 데이터를 최신순으로 보여줄 때
- 주의: props나 state 배열에 바로 쓰면 원본이 뒤집혀 버림

```jsx
const latestFirst = [...comments].reverse(); // 복사 후 뒤집기
// ES2023: comments.toReversed()
```

#### fill (ES6)

- 인수로 받은 값으로 요소를 채움, **원본 변경**
- 두 번째, 세 번째 인수로 시작 / 끝(미포함) 인덱스 지정
- 모든 요소를 같은 값으로만 채울 수 있음 → 요소마다 다른 값이 필요하면 `Array.from` 콜백 사용
- 활용: 스켈레톤 UI처럼 **개수만큼 빈 항목** 렌더링

```jsx
Array(6).fill(null).map((_, i) => <Skeleton key={i} />);
```

- `Array(6)`만 쓰면 희소 배열이라 `map`이 순회를 안 함 → `fill`로 실제 요소를 채워야 동작
- 함정: 객체나 배열로 채우면 **전부 같은 참조**

```jsx
const grid = Array(3).fill([]);
grid[0].push(1); // [[1], [1], [1]]

const grid2 = Array.from({ length: 3 }, () => []); // 각각 새 배열
```

#### includes (ES7)

- 특정 요소 포함 여부를 boolean으로 반환
- 두 번째 인수로 검색 시작 인덱스
- indexOf와 달리 **`NaN`도 확인 가능**
- 활용: 허용 값 체크, 여러 개 `||` 비교 대체

```jsx
const SUPPORTED = ['ko', 'en', 'ja'];
const lang = SUPPORTED.includes(userLang) ? userLang : 'en';

// status === 'a' || status === 'b' || status === 'c' 대신
['a', 'b', 'c'].includes(status);
```

#### flat (ES10)

- 중첩 배열을 평탄화
- 평탄화 깊이 기본값 1, `Infinity` 넘기면 끝까지 평탄화

```jsx
[1, [2, [3, [4]]]].flat();         // [1, 2, [3, [4]]]
[1, [2, [3, [4]]]].flat(Infinity); // [1, 2, 3, 4]
```

- 활용: 카테고리별로 묶인 데이터를 한 목록으로

```jsx
const allProducts = categories.map(c => c.products).flat();
// 이 패턴은 flatMap으로 한 번에: categories.flatMap(c => c.products)
```

### 배열 고차 함수

- **고차 함수**: 함수를 인수로 전달받거나 함수를 반환하는 함수
- 외부 상태 변경이나 가변 데이터를 피하고 **불변성**을 지향하는 함수형 프로그래밍 기반
    - 조건문과 반복문을 줄여 가독성을 높이고, 변수 사용을 억제해서 상태 변경을 피함
- 대부분 콜백에 **(요소값, 인덱스, 배열 자체)** 를 전달
- 대부분 두 번째 인수로 콜백 내부에서 쓸 **this(thisArg)** 를 받음 → 화살표 함수를 쓰면 필요 없음

#### sort

- **원본 변경**, 정렬된 배열 반환, 기본은 오름차순
- 기본 정렬은 요소를 **문자열로 변환해서 유니코드 코드 포인트 순**으로 정렬
    - 그래서 숫자 정렬 시 `[2, 10, 1].sort()` → `[1, 10, 2]`
    - **숫자 정렬엔 비교 함수 필수**

```jsx
points.sort((a, b) => a - b); // 오름차순
points.sort((a, b) => b - a); // 내림차순
```

- 비교 함수 반환값이 0보다 작으면 a 먼저, 크면 b 먼저, 0이면 그대로
- 객체 배열은 비교 함수로 특정 프로퍼티 기준 정렬
- ES10부터 **stable sort** 보장 (timsort)

#### forEach

- for 문 대체용, 요소 순회하며 콜백 호출
- 반환값은 항상 **undefined**
- 원본을 직접 변경하진 않지만 콜백에서 원본을 변경하는 건 가능
- **break, continue 사용 불가** → 중간에 멈출 수 없고 모든 요소 순회
- 희소 배열의 존재하지 않는 요소는 순회 대상에서 제외 (map, filter, reduce 등도 동일)
- 성능은 for 문보다 떨어지지만 가독성이 좋음 → 대량 데이터나 시간이 중요한 경우가 아니면 권장

#### map

- 콜백의 **반환값들로 구성된 새 배열** 반환, 원본 변경 X
- 원본 배열과 반환 배열의 length가 같음 → **1:1 매핑**
- forEach와 차이
    - forEach: 반복문 대체, 반환값 undefined
    - map: 요소값을 다른 값으로 매핑한 **새 배열 생성**이 목적

#### filter

- 콜백 반환값이 **true인 요소만** 추출한 새 배열 반환, 원본 변경 X
- 반환 배열 length ≤ 원본 length
- 특정 요소 제거에도 활용 (중복된 요소면 전부 제거됨)

#### reduce

```jsx
[1, 2, 3, 4].reduce((acc, cur) => acc + cur, 0); // 10
```

- 콜백에 **(누적값, 현재 요소, 인덱스, 배열)** 전달
- 두 번째 인수는 **초기값**
- 콜백 반환값이 다음 순회의 누적값이 되고, 최종적으로 **하나의 결과값** 반환
- 활용: 합계, 평균, 최대값, 요소 중복 횟수 세기, 중첩 배열 평탄화, 중복 제거
    - 근데 최대값은 `Math.max`, 평탄화는 `flat`, 중복 제거는 `filter`나 `Set`이 더 직관적
- **초기값은 항상 전달하는 게 안전**
    - 빈 배열에 초기값 없이 호출하면 TypeError
    - 객체 배열의 특정 프로퍼티를 합산할 때 초기값이 없으면 첫 순회에서 객체가 누적값이 되어버림

#### some / every

- `some`: 하나라도 true면 true, **빈 배열이면 false**
- `every`: 모두 true여야 true, **빈 배열이면 true**

#### find / findIndex (ES6)

- `find`: 콜백 반환값이 true인 **첫 번째 요소** 반환, 없으면 undefined
    - filter는 배열을 반환, find는 **요소를 반환**
- `findIndex`: 조건을 만족하는 첫 번째 요소의 **인덱스** 반환, 없으면 -1

#### flatMap (ES10)

- map으로 새 배열을 만든 뒤 1단계 평탄화 → `map` + `flat(1)`
- 평탄화 깊이 지정 불가, 1단계만

```jsx
['hello', 'world'].flatMap(str => str.split(''));
// ['h', 'e', 'l', 'l', 'o', 'w', 'o', 'r', 'l', 'd']
```