# 35장. 스프레드 문법

스프레드 문법(spread syntax, 전개 문법) `...`은 ES6에서 도입된, 실무에서 하루에도 몇 번씩 쓰게 되는 문법이다. 배열·문자열 같은 **하나로 뭉쳐 있는 값들의 묶음을 펼쳐서** 개별적인 값들로 흩어 놓는다.



## 스프레드 문법이란?

- 스프레드 문법 `...`은 **하나로 뭉쳐 있는 여러 값들의 집합을 펼쳐서(전개해서)** 개별적인 값들의 목록으로 만든다.
- 스프레드 문법을 쓸 수 있는 대상은 **이터러블(Iterable)**에 한정된다. (`Array`, `String`, `Map`, `Set`, `arguments` 등)

```js
// ...[1, 2, 3]은 [1, 2, 3]을 개별 요소로 분리한다 → 1, 2, 3
console.log(...[1, 2, 3]); // 1 2 3

// 문자열도 이터러블이라 가능
console.log(...'Hello'); // H e l l o

// Map, Set도 이터러블
console.log(...new Set([1, 2, 3])); // 1 2 3
```

여기서 꼭 짚어야 할 점. **스프레드 문법의 결과는 "값"이 아니다.** `1, 2, 3`처럼 쉼표로 나열된 **값들의 목록**일 뿐이라, 하나의 값으로 변수에 할당할 수 없다.

```js
// SyntaxError: 값의 목록은 변수에 담을 수 없다
// const list = ...[1, 2, 3];
```

=&gt; 그래서 스프레드 문법은 **쉼표로 구분한 값의 목록을 쓸 수 있는 문맥**에서만 사용할 수 있다. 구체적으로 세 곳이다. ① 함수 호출문의 인수 목록, ② 배열 리터럴 내부, ③ 객체 리터럴 내부.

<details class="orca-details">
<summary>스프레드 문법의 결과가 &quot;값이 아니다&quot;라는 게 무슨 뜻일까?</summary>

`...[1, 2, 3]`은 그 자체로 어떤 하나의 값(배열이나 객체)이 되는 게 아니라, **`1, 2, 3`이라는 "펼쳐진 목록"**이 된다는 뜻이다. 목록은 독립적으로 존재할 수 없고, 항상 "목록을 받아주는 자리" 안에 있어야 한다.

```js
const list = ...[1, 2, 3]; // SyntaxError (목록은 값이 아니라 할당 불가)

const arr = [...[1, 2, 3]]; // OK (배열 리터럴이 목록을 받아줌)
foo(...[1, 2, 3]);          // OK (함수 인수 자리가 목록을 받아줌)
```

=&gt; 쉽게 말해, `...`는 **"괄호/대괄호/중괄호 안에서 내용물을 쏟아붓는" 도구**이지, 그 자체로 결과물을 만드는 연산자가 아니다. 그래서 쏟아부을 그릇(함수 호출·배열·객체 리터럴)이 꼭 있어야 한다.

</details>

<details class="orca-details">
<summary>이터러블이 아니면 스프레드를 못 쓸까? 일반 객체는?</summary>

**배열 자리(`[...obj]`)나 함수 인수 자리(`f(...obj)`)에서는 이터러블만** 펼칠 수 있다. 일반 객체는 이터러블이 아니라서 거기선 에러가 난다.

```js
console.log([...{ a: 1 }]); // TypeError: object is not iterable
```

**단, 객체 리터럴 안(`{...obj}`)에서는 예외다.** 이건 스프레드 "문법"이 아니라 별도로 추가된 **스프레드 프로퍼티** 제안이라, 일반 객체도 펼칠 수 있다(아래 35.3에서 다룸).

```js
console.log({ ...{ a: 1 } }); // { a: 1 } ← 객체 리터럴에선 OK
```

=&gt; 정리하면 "**배열·함수 인수 자리 = 이터러블만**, **객체 리터럴 자리 = 일반 객체도 OK**". 둘은 뿌리가 다른 기능이라 규칙도 다르다.

</details>



## 함수 호출문의 인수 목록에서 사용하는 경우

- 배열 같은 이터러블을 **펼쳐서 함수의 개별 인수로** 전달할 때 쓴다.

가장 흔한 예가 `Math.max`다. `Math.max`는 숫자 인수들을 받아 최댓값을 반환하는데, **배열을 통째로 넘기면** 숫자가 아니라서 `NaN`이 된다.

```js
const arr = [1, 2, 3];

// 배열을 그대로 넘기면 안 됨
Math.max(arr);     // NaN
// 스프레드로 펼쳐서 넘겨야 함 → Math.max(1, 2, 3)
Math.max(...arr);  // 3
```

<details class="orca-details">
<summary>스프레드 나오기 전엔 Math.max(...arr)를 어떻게 했을까?</summary>

**`Function.prototype.apply`**를 썼다. `apply`는 두 번째 인수로 **배열을 받아** 그걸 개별 인수로 펼쳐 함수를 호출해준다(22장).

```js
// 스프레드 이전 (ES5)
Math.max.apply(null, [1, 2, 3]); // 3

// 스프레드 이후 (ES6)
Math.max(...[1, 2, 3]); // 3
```

둘은 같은 일을 하지만 스프레드가 훨씬 직관적이다. `apply`는 첫 인수로 `this`(여기선 안 쓰니 `null`)를 억지로 넘겨야 해서 의도가 덜 드러난다.

=&gt; 그래서 "배열을 개별 인수로 펼쳐 넘기는" 용도의 `apply`는 사실상 스프레드로 대체됐다. (`apply`는 이제 `this`를 바꿔야 하는 특수한 경우에만 쓴다.)

</details>

<details class="orca-details">
<summary>Rest 파라미터랑 똑같이 생겼는데, 이건 왜 스프레드일까?</summary>

**`...`가 어디에 쓰였느냐(위치)**로 구분한다. 생김새는 같지만 역할은 정반대다.

```js
// 스프레드: "함수를 호출"하는 쪽에서 배열을 펼친다 (풀기)
Math.max(...[1, 2, 3]);

// Rest 파라미터: "함수를 정의"하는 쪽에서 인수를 모은다 (묶기)
function foo(...rest) { console.log(rest); } // [1, 2, 3]
foo(1, 2, 3);
```

- **함수를 호출하는 자리**(괄호 안에 값을 넣는 곳)에 있으면 → **스프레드**(펼치기)
- **함수를 정의하는 자리**(매개변수 선언)에 있으면 → **Rest 파라미터**(모으기)

=&gt; "값을 넘기는 쪽이면 스프레드, 값을 받는 쪽이면 Rest." 26장에서 본 Rest 파라미터와 짝을 이루는 개념이다.

</details>



## 배열 리터럴 내부에서 사용하는 경우

배열 리터럴 `[]` 안에서 스프레드를 쓰면, 기존의 번거로운 배열 메서드들을 훨씬 간결하게 대체할 수 있다.

### concat 대체 — 배열 합치기

```js
const arr1 = [1, 2];
const arr2 = [3, 4];

// ES5: concat
arr1.concat(arr2); // [1, 2, 3, 4]

// ES6: 스프레드
[...arr1, ...arr2]; // [1, 2, 3, 4]
```

### splice 대체 — 중간에 끼워넣기

```js
const arr = [1, 4];

// ES5: splice로 중간 삽입 (원본 변경, 문법도 복잡)
// arr.splice(1, 0, 2, 3);

// ES6: 스프레드로 새 배열 생성 (원본 유지)
[arr[0], ...[2, 3], arr[1]]; // [1, 2, 3, 4]
```

### 배열 복사

```js
const origin = [1, 2, 3];

// ES5: slice()
const copy1 = origin.slice();

// ES6: 스프레드
const copy2 = [...origin];
console.log(copy2);         // [1, 2, 3]
console.log(copy2 === origin); // false (새 배열)
```

### 이터러블을 배열로 변환

```js
// 문자열 → 배열
[...'Hello']; // ['H', 'e', 'l', 'l', 'o']

// Set → 배열 (중복 제거 관용구!)
[...new Set([1, 1, 2, 3, 3])]; // [1, 2, 3]
```

<details class="orca-details">
<summary>스프레드 복사 [...arr]도 slice처럼 &quot;얕은 복사&quot;일까?</summary>

**그렇다.** `[...arr]`도 `slice()`와 똑같이 **얕은 복사(shallow copy)**다. 배열 껍데기는 새로 만들지만, 요소가 객체면 그 객체는 **참조만 복사**된다.

```js
const origin = [{ x: 1 }, { x: 2 }];
const copy = [...origin];

console.log(copy === origin);       // false (배열은 새것)
console.log(copy[0] === origin[0]); // true  (안의 객체는 같은 것!)

copy[0].x = 100;
console.log(origin[0].x); // 100 ← 원본 객체도 바뀜!
```

=&gt; 원시값만 든 배열이면 `[...arr]`로 충분하지만, 객체가 든 배열을 진짜 독립적으로 복사하려면 **깊은 복사**(`structuredClone(arr)`)가 필요하다. 27장 `slice`에서 본 것과 완전히 같은 함정이다.

</details>

<details class="orca-details">
<summary>Array.from이랑 [...iterable], 둘 다 이터러블을 배열로 만드는데 뭐가 다를까?</summary>

이터러블 변환은 둘이 같지만, **"유사 배열 객체"**를 다룰 때 갈린다.

- **스프레드 `[...x]`**: `x`가 **이터러블**이어야만 동작한다. 유사 배열 객체(이터러블이 아닌)엔 못 쓴다.
- **`Array.from(x)`**: **이터러블 + 유사 배열 객체** 둘 다 변환할 수 있다. 게다가 두 번째 인수로 map 콜백도 받는다.

```js
const arrayLike = { 0: 'a', 1: 'b', length: 2 }; // 이터러블 아님

[...arrayLike];        // TypeError: not iterable
Array.from(arrayLike); // ['a', 'b'] ← 성공
```

=&gt; "이터러블(배열·문자열·Set 등)이면 `[...x]`가 간결하고, 유사 배열 객체이거나 변환과 동시에 매핑이 필요하면 `Array.from`". 옛날 `arguments`(유사 배열)도 `Array.from`으로 배열화한다.

</details>



## 객체 리터럴 내부에서 사용하는 경우

- 객체 리터럴 `{}` 안에서도 스프레드를 쓸 수 있다. 이건 **스프레드 프로퍼티(spread properties)**라는 별도 제안으로, 객체를 펼쳐 새 객체로 합치거나 복사할 때 쓴다.

```js
const obj1 = { a: 1, b: 2 };
const obj2 = { c: 3 };

// 객체 병합
const merged = { ...obj1, ...obj2 }; // { a: 1, b: 2, c: 3 }

// 객체 복사
const copy = { ...obj1 }; // { a: 1, b: 2 }

// 특정 프로퍼티 덮어쓰기(갱신)
const updated = { ...obj1, b: 100 }; // { a: 1, b: 100 }
```

기존에는 `Object.assign`으로 하던 일을 더 간결하게 표현한다.

```js
// ES5~: Object.assign
Object.assign({}, obj1, obj2); // { a: 1, b: 2, c: 3 }
// 스프레드 프로퍼티
{ ...obj1, ...obj2 };
```

<details class="orca-details">
<summary>프로퍼티가 겹치면 어떻게 될까? 순서가 중요할까?</summary>

**겹치면 "나중에 나온 것"이 이긴다.** 그래서 순서가 매우 중요하다.

```js
const base = { a: 1, b: 2 };

{ ...base, b: 100 }; // { a: 1, b: 100 } ← 뒤의 b가 이김
{ b: 100, ...base }; // { a: 1, b: 2 }   ← base의 b가 덮어씀!
```

이 성질 덕분에 **"기본값 위에 사용자 값 덮어쓰기"** 패턴을 자주 쓴다.

```js
const defaults = { theme: 'light', size: 'm' };
const userOptions = { size: 'l' };
const config = { ...defaults, ...userOptions };
// { theme: 'light', size: 'l' } ← 기본값 깔고 사용자 값으로 덮음
```

=&gt; "기본값을 먼저, 덮어쓸 값을 나중에." React에서 props나 state를 갱신할 때 `{ ...state, changed: value }` 패턴이 바로 이것이다.

</details>

<details class="orca-details">
<summary>객체 스프레드도 얕은 복사일까? 중첩 객체는?</summary>

**얕은 복사 맞다.** 최상위 프로퍼티만 복사하고, 값이 객체면 참조만 복사된다.

```js
const origin = { user: { name: 'Lee' } };
const copy = { ...origin };

copy.user.name = 'Kim';
console.log(origin.user.name); // Kim ← 중첩 객체는 공유됨!
```

`copy`의 `user`와 `origin`의 `user`는 **같은 객체**를 가리킨다. 그래서 중첩된 값을 바꾸면 원본도 바뀐다.

중첩 객체까지 안전하게 바꾸려면 그 단계도 펼쳐야 한다.

```js
const copy = { ...origin, user: { ...origin.user, name: 'Kim' } };
// 이제 origin.user는 안전
```

=&gt; 깊은 구조라면 단계마다 스프레드를 겹쳐 쓰거나, `structuredClone`으로 깊은 복사를 한다. "스프레드 = 한 겹만 복사"를 항상 기억하자.

</details>

<details class="orca-details">
<summary>Object.assign이랑 객체 스프레드, 완전히 같을까?</summary>

거의 같지만 **미묘한 차이**가 있다.

1. **`Object.assign`은 대상 객체를 변경(mutate)**한다. 첫 인수에 병합하므로, 빈 객체 `{}`를 안 넣으면 원본이 바뀐다.

```js
Object.assign(target, source); // target이 변경됨!
const merged = { ...a, ...b };  // 항상 새 객체, 원본 안전
```

2. **setter 동작 차이**: `Object.assign`은 대상의 setter를 호출하지만, 스프레드는 그냥 값을 복사해 새 프로퍼티로 정의한다.

=&gt; 실무에선 "새 객체를 만든다"는 의도가 분명하고 원본을 안 건드리는 **스프레드가 더 안전**해서 선호된다. `Object.assign`은 기존 객체에 병합해야 하는 특수한 경우에 쓴다.

</details>



## Rest 파라미터와의 차이

생김새가 똑같은 `...`이지만, 스프레드 문법과 Rest 파라미터는 **정반대 방향**의 기능이다. 이걸 헷갈리지 않는 게 이 장의 핵심이다.

- **스프레드 문법**: 하나로 뭉친 값을 **개별 값으로 펼친다.** (풀기) — 값을 넘기는 자리(함수 호출·리터럴)에서 쓴다.
- **Rest 파라미터**: 흩어진 개별 값을 **하나의 배열로 모은다.** (묶기) — 함수를 정의하는 매개변수 자리에서 쓴다.

```js
// Rest 파라미터: 개별 인수들을 rest 배열로 "모은다"
function foo(...rest) {
  console.log(rest); // [1, 2, 3]
}

// 스프레드 문법: 배열을 개별 인수로 "펼친다"
foo(...[1, 2, 3]); // 위 foo에 1, 2, 3을 개별로 넘김

// 한 줄에 둘 다 등장할 수도 있다
function bar(x, y, ...rest) { // rest: 모으기
  return [x, y, ...rest];     // ...rest: 펼치기
}
```

| 구분 | 스프레드 문법 | Rest 파라미터 |
| --- | --- | --- |
| 역할 | 펼치기(전개) | 모으기(수집) |
| 방향 | 배열 → 개별 값 | 개별 값 → 배열 |
| 쓰는 곳 | 함수 호출문·배열/객체 리터럴 | 함수 매개변수 |

<details class="orca-details">
<summary>한 줄에 ...가 두 번 나오면 어떻게 구분할까?</summary>

**위치를 보면 된다.** `function bar(x, y, ...rest) { return [x, y, ...rest]; }`를 뜯어보자.

- `bar(x, y, ...rest)`의 `...rest` → **매개변수를 선언하는 자리** → **Rest**(모으기)
- `[x, y, ...rest]`의 `...rest` → **배열 리터럴 안, 값을 넣는 자리** → **스프레드**(펼치기)

```js
bar(1, 2, 3, 4);
// Rest가 [3, 4]를 모음 → rest = [3, 4]
// 스프레드가 rest를 펼침 → [1, 2, 3, 4] 반환
```

=&gt; "선언하는 쪽(매개변수)이면 Rest, 사용하는 쪽(호출·리터럴)이면 스프레드." 같은 `...`라도 어느 자리에 있는지만 보면 100% 구분된다.

</details>



# 36장. 디스트럭처링 할당

디스트럭처링 할당(destructuring assignment, 구조 분해 할당)은 **구조화된 배열이나 객체를 분해(destructuring)해서, 그 안의 값들을 여러 변수에 한 번에 꺼내 담는** 문법이다. 35장 스프레드가 "펼쳐서 넣기"였다면, 디스트럭처링은 반대로 "분해해서 꺼내기"에 가깝다.



## 디스트럭처링 할당이란?

- 디스트럭처링 할당은 구조화된 배열(이터러블) 또는 객체를 분해해, **1개 이상의 변수에 개별적으로 할당**하는 것이다.
- 배열이나 객체에서 **필요한 값만 추출**해서 변수에 담을 때 유용하다.

```js
// 디스트럭처링 없이 (번거로움)
const arr = [1, 2, 3];
const one = arr[0];
const two = arr[1];

// 디스트럭처링으로 (한 줄)
const [one, two, three] = [1, 2, 3];
console.log(one, two, three); // 1 2 3
```

=&gt; 값을 하나씩 인덱스/키로 꺼내 변수에 담던 반복 작업을, **구조만 맞춰주면 한 번에** 끝낼 수 있다는 게 핵심이다.



## 배열 디스트럭처링 할당

- 할당 대상은 **이터러블**이어야 하고, 할당 기준은 배열의 **인덱스(순서)**다.
- 왼쪽에 배열 리터럴 형태로 변수를 나열하면, 오른쪽 배열의 요소가 **순서대로** 할당된다.

```js
const arr = [1, 2, 3];
const [one, two, three] = arr;
console.log(one, two, three); // 1 2 3
```

변수 개수와 요소 개수가 **일치하지 않아도 된다.** 남는 변수는 `undefined`, 남는 요소는 무시된다.

```js
const [a, b] = [1, 2, 3]; // a=1, b=2 (3은 버려짐)
const [c, d] = [1];       // c=1, d=undefined

// 필요 없는 요소는 건너뛸 수도 있다 (쉼표만)
const [, second, , fourth] = [1, 2, 3, 4]; // second=2, fourth=4
```

### 기본값

- 할당될 값이 `undefined`일 때를 대비해 **기본값**을 설정할 수 있다.

```js
const [a, b, c = 3] = [1, 2];     // a=1, b=2, c=3 (기본값)
const [x, y = 10] = [1, 2];       // x=1, y=2 (값이 있으면 기본값 무시)
```

### Rest 요소

- 스프레드처럼 생긴 `...`를 마지막 변수에 붙이면, **나머지 요소를 배열로** 모은다. (26장 Rest 파라미터와 같은 원리)
- Rest 요소는 반드시 **마지막**에 와야 한다.

```js
const [first, ...rest] = [1, 2, 3, 4];
console.log(first); // 1
console.log(rest);  // [2, 3, 4]
```

<details class="orca-details">
<summary>변수를 두 번 교환(swap)할 때 디스트럭처링이 편하다던데?</summary>

**임시 변수 없이 한 줄로** 두 변수의 값을 맞바꿀 수 있다. 디스트럭처링의 대표적인 활용이다.

```js
let a = 1, b = 2;

// 옛날 방식: 임시 변수 필요
// let temp = a; a = b; b = temp;

// 디스트럭처링: 한 줄로
[a, b] = [b, a];
console.log(a, b); // 2 1
```

오른쪽 `[b, a]`가 먼저 `[2, 1]`로 평가된 뒤, 왼쪽 `[a, b]`에 순서대로 할당되어 값이 교환된다.

=&gt; 임시 변수가 사라져 코드가 깔끔하고 의도가 분명하다. 다만 성능이 극도로 중요한 루프에서는 임시 변수 방식이 미세하게 빠를 수 있으니, 거기선 상황을 봐서 선택한다.

</details>

<details class="orca-details">
<summary>Rest 요소랑 35장 스프레드랑 똑같이 생겼는데 뭐가 다를까?</summary>

**방향이 반대**다. 35장에서 본 "펼치기 vs 모으기"가 여기서도 똑같이 적용된다.

```js
// 스프레드(펼치기): 값을 넣는 쪽
const arr = [...[1, 2], 3]; // [1, 2, 3] 으로 펼침

// Rest 요소(모으기): 값을 받는 쪽(할당 왼쪽)
const [head, ...tail] = [1, 2, 3]; // 나머지를 tail에 모음
```

- **할당의 왼쪽(값을 받는 자리)**에 있으면 → **Rest 요소**(모으기)
- **값을 만드는 자리(리터럴·함수 호출)**에 있으면 → **스프레드**(펼치기)

=&gt; 함수의 Rest 파라미터(26장), 배열 디스트럭처링의 Rest 요소가 모두 "받는 쪽에서 모으는" 같은 개념이다. 위치만 보면 구분된다.

</details>

<details class="orca-details">
<summary>문자열이나 Set도 배열 디스트럭처링이 될까?</summary>

**된다.** 배열 디스트럭처링의 대상은 "배열"이 아니라 **이터러블**이기 때문이다. 문자열·Set·Map 등 이터러블이면 전부 분해할 수 있다.

```js
const [a, b, c] = 'abc';          // a='a', b='b', c='c'
const [x, y] = new Set([1, 2, 3]); // x=1, y=2
```

=&gt; "순서가 있는 이터러블이면 배열 디스트럭처링으로 앞에서부터 꺼낼 수 있다"고 이해하면 된다. 반대로 일반 객체(이터러블 아님)는 배열 디스트럭처링이 안 되고, 객체 디스트럭처링(`{}`)을 써야 한다.

</details>



## 객체 디스트럭처링 할당

- 할당 대상은 **객체**여야 하고, 할당 기준은 **프로퍼티 키**다. (배열처럼 순서가 아니다!)
- 왼쪽에 중괄호 `{}`로 변수를 나열하면, 오른쪽 객체에서 **같은 이름의 프로퍼티 값**이 할당된다.

```js
const user = { name: 'Lee', age: 20 };

// 프로퍼티 키와 같은 이름의 변수에 할당 (순서 무관)
const { name, age } = user;
console.log(name, age); // Lee 20

// 순서가 달라도 키 이름으로 매칭되므로 OK
const { age, name } = user; // 정상 동작
```

### 변수 이름 바꾸기

- 프로퍼티 키와 **다른 이름**의 변수에 담고 싶으면 `키: 새이름` 형태로 쓴다.

```js
const user = { name: 'Lee', age: 20 };
const { name: userName, age: userAge } = user;
console.log(userName, userAge); // Lee 20
// console.log(name); // ReferenceError (name은 선언 안 됨)
```

### 기본값

- 배열과 마찬가지로 기본값을 줄 수 있다.

```js
const { name = '미정', age = 0 } = { name: 'Lee' };
console.log(name, age); // Lee 0

// 이름 변경 + 기본값 동시에
const { name: userName = '익명' } = {};
console.log(userName); // 익명
```

### 함수 매개변수에서 사용

- 함수가 객체를 인자로 받을 때, **매개변수 자리에서 바로 분해**하면 필요한 프로퍼티만 깔끔하게 꺼낼 수 있다.

```js
// 분해 전
function printUser(user) {
  console.log(user.name, user.age);
}

// 매개변수에서 디스트럭처링
function printUser({ name, age }) {
  console.log(name, age);
}
printUser({ name: 'Lee', age: 20 }); // Lee 20

// 기본값까지 조합 (인자 안 넘겨도 안전)
function greet({ name = '손님' } = {}) {
  console.log(`안녕하세요, ${name}님`);
}
greet();                 // 안녕하세요, 손님님
greet({ name: 'Kim' });  // 안녕하세요, Kim님
```

### Rest 프로퍼티

- 객체에서도 `...`로 **나머지 프로퍼티를 새 객체로** 모을 수 있다. 반드시 마지막에 위치한다.

```js
const { name, ...rest } = { name: 'Lee', age: 20, city: 'Seoul' };
console.log(name); // Lee
console.log(rest); // { age: 20, city: 'Seoul' }
```

### 중첩 객체

- 깊은 구조의 객체도 구조를 그대로 본떠서 분해할 수 있다.

```js
const user = {
  name: 'Lee',
  address: { city: 'Seoul', zipcode: '12345' },
};

// 중첩된 city를 직접 꺼냄
const { address: { city } } = user;
console.log(city); // Seoul
```

<details class="orca-details">
<summary>배열은 순서로, 객체는 키로 매칭한다는 게 왜 중요할까?</summary>

**분해할 때 "무엇을 기준으로 꺼내느냐"가 완전히 달라서**, 둘을 헷갈리면 원하는 값을 못 꺼낸다.

```js
// 배열: 순서(인덱스)로 매칭 → 이름은 내 맘대로
const [first, second] = [1, 2]; // first=1, second=2 (순서대로)

// 객체: 프로퍼티 키로 매칭 → 이름이 키와 같아야 함
const { age, name } = { name: 'Lee', age: 20 };
// 순서를 바꿔도 키로 찾으니 age=20, name='Lee' (정상)
const { x } = { name: 'Lee' };
console.log(x); // undefined (x라는 키가 없으니)
```

- **배열**: 이름은 자유롭지만 **순서**가 중요하다.
- **객체**: 순서는 자유롭지만 **이름(키)**이 정확히 맞아야 한다.

=&gt; "배열은 자리로 꺼내고, 객체는 이름표로 꺼낸다"고 기억하면 된다. 그래서 객체에서 다른 이름으로 받고 싶으면 `키: 새이름`으로 명시해야 하는 것이다.

</details>

<details class="orca-details">
<summary>const { name: userName } = user; 여기서 name은 변수가 아닐까?</summary>

**아니다.** `name: userName`에서 **`name`은 "찾을 프로퍼티 키"이고, 실제 선언되는 변수는 `userName`**이다. 이게 헷갈리기 쉬운 포인트다.

```js
const user = { name: 'Lee' };
const { name: userName } = user;

console.log(userName); // Lee  ← 이게 변수
console.log(name);     // ReferenceError: name is not defined
```

`name`을 변수로 착각하기 쉬운데, 콜론 앞은 "객체에서 찾을 키", 콜론 뒤가 "그 값을 담을 변수"다. 객체 리터럴에서 `{ 키: 값 }`을 쓰던 것과 방향이 반대라 처음엔 헷갈린다.

=&gt; "객체 **만들 때**는 `{ 키: 값 }`, 객체 **분해할 때**는 `{ 키: 변수 }`". 분해에서는 콜론 뒤가 변수라는 걸 기억하자.

</details>

<details class="orca-details">
<summary>React에서 디스트럭처링을 왜 그렇게 많이 쓸까?</summary>

**props·state·훅 반환값이 전부 객체나 배열이라서**, 디스트럭처링이 거의 필수처럼 쓰인다.

```js
// props 분해 (매개변수에서 바로)
function Profile({ name, age }) {
  return {name} ({age})

;
}

// useState는 배열을 반환 → 배열 디스트럭처링
const [count, setCount] = useState(0);

// 여러 값을 객체로 반환하는 커스텀 훅
const { data, loading, error } = useFetch('/api');
```

`useState`가 **배열**을 반환하는 건, 사용자가 `[count, setCount]`처럼 **이름을 자유롭게** 짓게 하려는 의도다(배열은 순서로 매칭하니까). 반대로 여러 값을 반환하는 커스텀 훅은 보통 **객체**로 줘서, 필요한 것만 이름으로 골라 꺼내게 한다.

=&gt; "순서가 중요하고 이름을 자유롭게 → 배열 반환", "필요한 것만 골라 쓰게 → 객체 반환"이라는 설계 감각이 디스트럭처링과 맞물려 있다. React를 쓴다면 이 둘을 구분해 쓰는 눈이 중요하다.

</details>


# 37장. Set과 Map

ES6에서 추가된 `Set`과 `Map`은 각각 배열과 객체를 **대체·보완**하는 새로운 자료구조다. 배열·객체로도 비슷한 걸 할 수 있지만, 이 둘은 특정 상황에서 더 명확하고 안전하다.



## Set이란?

- `Set` 객체는 **중복되지 않는 유일한 값들의 집합**이다. 수학의 집합을 자료구조로 구현한 것에 가깝다.
- 배열과 비슷해 보이지만 다음 두 가지가 결정적으로 다르다.


| 구분            | 배열  | Set            |
| ------------- | --- | -------------- |
| 중복 허용         | O   | **X (중복 불가)**  |
| 요소 순서(인덱스) 접근 | O   | **X (인덱스 없음)** |


```js
const set = new Set([1, 2, 2, 3, 3, 3]);
console.log(set); // Set(3) {1, 2, 3} ← 중복이 제거됨
```

=&gt; "중복을 허용하지 않는다"가 Set의 정체성이다. 그래서 **배열 중복 제거**에 가장 많이 쓰인다.

<details class="orca-details">
<summary>Set의 중복 판단 기준은 뭘까? === 랑 같을까?</summary>

거의 같지만 `**NaN` 하나가 다르다.** Set은 **SameValueZero**라는 비교 방식을 쓰는데, `===`와 딱 하나 차이가 있다.

- `===`에서는 `NaN === NaN`이 `false`라, 보통은 NaN을 "다른 값"으로 본다.
- Set(SameValueZero)은 `**NaN`과 `NaN`을 같은 값으로** 취급한다. 그래서 NaN을 여러 개 넣어도 하나만 남는다.

```js
const set = new Set();
set.add(NaN).add(NaN);
console.log(set.size); // 1 ← NaN끼리 중복으로 봄

// 객체는 참조로 비교 → 내용이 같아도 다른 객체
set.add({}).add({});
console.log(set.size); // 3 ← 서로 다른 객체 2개 추가됨
```

=&gt; "원시값은 값으로(NaN 포함), 객체는 참조로" 중복을 판단한다. 내용이 같은 객체 두 개는 Set에서 중복으로 안 본다는 점을 주의하자.

</details>

### Set 객체의 생성

- `new Set()`으로 생성한다. 인수로 **이터러블**을 넘기면 그 요소로 Set을 채운다(중복은 자동 제거).

```js
const set1 = new Set();            // 빈 Set
const set2 = new Set([1, 2, 3]);   // Set(3) {1, 2, 3}
const set3 = new Set('hello');     // Set(4) {'h', 'e', 'l', 'o'} (l 중복 제거)
```

### size — 요소 개수 확인

- 요소 개수는 `size` 프로퍼티로 확인한다. (배열의 `length`에 대응)
- `size`는 **getter만 있는** 접근자 프로퍼티라, 값을 할당해서 크기를 바꿀 수 없다.

```js
const set = new Set([1, 2, 3]);
console.log(set.size); // 3
// set.size = 10; // 무시됨 (변경 불가)
```

### add — 요소 추가

- `add(value)`로 요소를 추가한다. **추가된 Set 자신을 반환**하므로 **체이닝**이 가능하다. 중복 값은 조용히 무시된다.

```js
const set = new Set();
set.add(1).add(2).add(2).add(3); // 체이닝
console.log(set); // Set(3) {1, 2, 3}
```

### has — 요소 존재 확인

- `has(value)`로 특정 값이 있는지 불리언으로 확인한다.

```js
const set = new Set([1, 2, 3]);
console.log(set.has(2)); // true
console.log(set.has(5)); // false
```

### delete / clear — 요소 삭제

- `delete(value)`: 특정 **값**을 삭제하고, 성공 여부를 불리언으로 반환. (인덱스가 아니라 값으로 지운다)
- `clear()`: 모든 요소를 **일괄 삭제**. 반환값은 `undefined`.

```js
const set = new Set([1, 2, 3]);
set.delete(2);   // true → Set(2) {1, 3}
set.delete(10);  // false (없는 값)
set.clear();     // Set(0) {}
```

### forEach — 요소 순회

- `forEach`로 순회한다. 콜백은 `(value, value2, set)`을 받는데, **첫 번째와 두 번째 인수가 같은 값**이다.

```js
const set = new Set([1, 2, 3]);
set.forEach((v, v2, s) => console.log(v, v2));
// 1 1 / 2 2 / 3 3

// Set은 이터러블이라 for...of, 스프레드, 디스트럭처링도 된다
for (const value of set) console.log(value); // 1, 2, 3
console.log([...set]); // [1, 2, 3]
```

<details class="orca-details">
<summary>Set의 forEach는 왜 value를 두 번(v, v2) 줄까?</summary>

**배열 `forEach`와 콜백 모양(인수 3개)을 맞추려는 설계** 때문이다.

배열 `forEach`의 콜백은 `(요소값, 인덱스, 배열)`을 받는다. 그런데 Set은 인덱스가 없다. 그렇다고 인수 구조를 다르게 하면 혼란스러우니, **"인덱스 자리에 그냥 값을 한 번 더" 넣어** `(value, value, set)` 형태로 통일한 것이다.

```js
// 배열:  (요소, 인덱스, 배열)
// Set:   (값,   값,     Set)   ← 두 번째도 값
// Map:   (값,   키,     Map)   ← Map은 두 번째가 키
```

=&gt; 즉 Set의 `v2`는 의미가 있어서가 아니라 "자리를 맞추려고" 있는 것이다. 보통은 첫 번째 인수만 쓰면 된다. (Map은 두 번째가 키라서 실제로 의미가 있다.)

</details>

### 집합 연산

Set으로 수학의 **교집합·합집합·차집합·부분집합** 같은 집합 연산을 구현할 수 있다.

```js
const setA = new Set([1, 2, 3, 4]);
const setB = new Set([2, 4]);

// 교집합 (A와 B 둘 다에 있는 것)
const intersection = new Set([...setA].filter(v => setB.has(v)));
console.log(intersection); // Set(2) {2, 4}

// 합집합 (A 또는 B에 있는 것)
const union = new Set([...setA, ...setB]);
console.log(union); // Set(4) {1, 2, 3, 4}

// 차집합 (A에는 있지만 B에는 없는 것)
const difference = new Set([...setA].filter(v => !setB.has(v)));
console.log(difference); // Set(2) {1, 3}

// 부분집합 (B가 A에 포함되는가)
const isSubset = [...setB].every(v => setA.has(v));
console.log(isSubset); // true
```

<details class="orca-details">
<summary>요즘은 집합 연산을 더 간단하게 할 수 있다던데?</summary>

맞다. **ES2024에서 Set에 집합 연산 메서드가 정식 추가**됐다. 위처럼 `[...set].filter(...)`로 손수 만들던 걸 메서드 하나로 해결한다.

```js
const a = new Set([1, 2, 3, 4]);
const b = new Set([2, 4]);

a.intersection(b);       // Set {2, 4}   교집합
a.union(b);              // Set {1,2,3,4} 합집합
a.difference(b);         // Set {1, 3}   차집합
a.symmetricDifference(b);// 대칭 차집합 (양쪽 중 한쪽에만)
b.isSubsetOf(a);         // true  부분집합
a.isSupersetOf(b);       // true  상위집합
a.isDisjointFrom(b);     // false 서로소 여부
```

=&gt; 최신 브라우저·Node라면 이 메서드들을 쓰는 게 훨씬 깔끔하다. 다만 지원 환경을 확인해야 하고, 책이 나온 시점엔 없던 기능이라 책에선 `filter`·스프레드 방식으로 설명한다. "원리는 filter/스프레드, 최신 문법은 메서드"로 알아두면 좋다.

</details>

<details class="orca-details">
<summary>배열 중복 제거는 Set이 제일 좋을까? filter랑 비교하면?</summary>

**대부분 Set이 가장 간결하고 빠르다.**

```js
const arr = [1, 1, 2, 3, 3];

// Set 방식 (권장)
const unique1 = [...new Set(arr)]; // [1, 2, 3]

// filter + indexOf 방식 (느림)
const unique2 = arr.filter((v, i) => arr.indexOf(v) === i);
```

`filter + indexOf`는 각 요소마다 `indexOf`로 배열 전체를 다시 뒤지므로 O(n²)이다. 반면 Set은 내부적으로 해시 기반이라 추가·조회가 평균 O(1)이고, 전체 중복 제거가 O(n)이다.

=&gt; "원시값 배열의 중복 제거"는 `[...new Set(arr)]`가 거의 정답이다. 단, **객체 배열**은 참조로 비교하니(내용 같아도 다른 객체) 이 방법으로 중복 제거가 안 된다 — 그땐 키를 뽑아 Map/객체로 거르는 식으로 접근해야 한다.

</details>



## Map이란?

- `Map` 객체는 **키-값 쌍의 모음**이다. 객체와 비슷하지만 다음이 다르다.


| 구분          | 객체                        | Map                |
| ----------- | ------------------------- | ------------------ |
| 키로 쓸 수 있는 값 | 문자열·심벌만                   | **모든 값(객체·함수 포함)** |
| 이터러블        | X                         | **O**              |
| 요소 개수 확인    | `Object.keys(obj).length` | `**size` 프로퍼티**    |
| 순서 보장       | (대체로) O                   | **O (삽입 순서)**      |


```js
const map = new Map();
const key = { id: 1 };
map.set(key, 'object key!'); // 객체를 키로 사용
console.log(map.get(key));   // 'object key!'
```

=&gt; "객체도 키로 쓸 수 있다"와 "size·이터러블로 다루기 편하다"가 Map의 핵심 강점이다.

### Map 객체의 생성

- `new Map()`으로 생성한다. 인수로 `**[키, 값]` 쌍의 배열**(또는 이터러블)을 넘겨 초기화할 수 있다.

```js
const map1 = new Map();
const map2 = new Map([
  ['key1', 'value1'],
  ['key2', 'value2'],
]);
console.log(map2); // Map(2) {'key1' => 'value1', 'key2' => 'value2'}
```

### size — 요소 개수 확인

```js
const map = new Map([['a', 1], ['b', 2]]);
console.log(map.size); // 2
```

### set / get — 요소 추가·취득

- `set(key, value)`: 요소를 추가한다. **Map 자신을 반환**하므로 체이닝 가능.
- `get(key)`: 키로 값을 가져온다. 없으면 `undefined`.

```js
const map = new Map();
map.set('name', 'Lee').set('age', 20); // 체이닝

console.log(map.get('name')); // Lee
console.log(map.get('xxx'));  // undefined (없는 키)
```

### has — 요소 존재 확인

```js
const map = new Map([['name', 'Lee']]);
console.log(map.has('name')); // true
console.log(map.has('age'));  // false
```

### delete / clear — 요소 삭제

- `delete(key)`: 특정 키의 요소 삭제(성공 여부 반환). `clear()`: 전부 삭제.

```js
const map = new Map([['a', 1], ['b', 2]]);
map.delete('a'); // true → Map(1) {'b' => 2}
map.clear();     // Map(0) {}
```

### forEach / 요소 순회

- `forEach` 콜백은 `(value, key, map)`을 받는다. Set과 달리 **두 번째 인수가 "키"**라 실제로 의미가 있다.
- Map은 이터러블이라 `keys()`, `values()`, `entries()`와 `for...of`로 순회할 수 있다.

```js
const map = new Map([['name', 'Lee'], ['age', 20]]);

map.forEach((value, key) => console.log(key, value));
// name Lee / age 20

for (const [key, value] of map) console.log(key, value); // 디스트럭처링
for (const key of map.keys()) console.log(key);          // name, age
for (const value of map.values()) console.log(value);    // Lee, 20
```

<details class="orca-details">
<summary>그냥 객체 쓰면 되지, 언제 Map을 써야 할까?</summary>

**"키가 문자열이 아니거나, 자주 추가·삭제·개수 확인을 하거나, 순회가 잦을 때"** Map이 낫다.

객체 대신 Map이 유리한 경우:

- **키가 문자열·심벌이 아닐 때** → 객체는 키를 문자열로 강제 변환하지만 Map은 객체·함수도 키로 그대로 쓴다.
- **동적으로 키-값을 자주 넣고 빼고 셀 때** → `map.size`로 바로 개수를 알고, 추가·삭제가 최적화돼 있다.
- **순회가 잦을 때** → Map은 이터러블이라 `for...of`로 바로 돌 수 있다.

```js
const obj = {};
obj[1] = 'a';      // 키 1이 문자열 '1'로 변환됨
console.log(obj['1'] === obj[1]); // true (헷갈림)

const map = new Map();
map.set(1, 'a');   // 숫자 1 그대로 키
console.log(map.get(1)); // 'a'
```

반대로 **고정된 구조의 데이터, JSON 직렬화가 필요한 데이터**는 그냥 객체가 낫다(Map은 `JSON.stringify`로 바로 직렬화가 안 됨).

=&gt; "구조가 고정된 레코드는 객체, 동적인 키-값 저장소·캐시·룩업 테이블은 Map"이라고 기준을 잡으면 된다.

</details>

<details class="orca-details">
<summary>객체를 키로 쓸 수 있다는 게 왜 유용할까?</summary>

**어떤 객체에 "부가 정보"를 외부에서 연결해두고 싶을 때** 유용하다. 그 객체를 건드리지 않고도 메타데이터를 매달 수 있다.

```js
const user1 = { name: 'Lee' };
const user2 = { name: 'Kim' };

const lastLogin = new Map();
lastLogin.set(user1, '2026-10-01'); // 객체 자체가 키
lastLogin.set(user2, '2026-09-28');

console.log(lastLogin.get(user1)); // '2026-10-01'
```

`user1`에 `lastLogin` 프로퍼티를 직접 추가하지 않고도, 바깥 Map에서 그 객체와 연결된 정보를 관리할 수 있다. DOM 요소에 데이터를 연결하거나, 함수 호출 결과를 캐싱(메모이제이션)할 때 흔히 쓴다.

=&gt; 참고로 "키로 쓴 객체가 사라지면 메모리도 자동 정리되길" 원한다면 **WeakMap**을 쓴다(아래 토글). 일반 Map은 키 객체를 계속 붙잡고 있어 가비지 컬렉션을 막는다.

</details>

<details class="orca-details">
<summary>WeakSet, WeakMap은 뭘까? 그냥 Set/Map이랑 뭐가 다를까?</summary>

**키(또는 값)로 쓴 객체를 "약하게" 참조**해서, 그 객체를 다른 데서 더 안 쓰면 **가비지 컬렉션이 회수**하도록 둔 버전이다.

- **Map/Set**: 키·값을 강하게 붙잡는다. 그래서 키로 쓴 객체는 Map이 살아있는 한 메모리에서 안 사라진다 → 관리 안 하면 메모리 누수.
- **WeakMap/WeakSet**: 객체를 약하게만 참조한다. 그 객체를 가리키는 다른 참조가 없어지면 엔진이 알아서 회수한다.

```js
let user = { name: 'Lee' };
const wm = new WeakMap();
wm.set(user, 'data');

user = null; // 이제 user 객체를 아무도 안 씀
// → WeakMap의 해당 항목도 언젠가 자동으로 사라짐 (메모리 안전)
```

대신 제약이 있다. **키는 반드시 객체여야 하고**(원시값 불가), **순회·size가 불가능**하다(언제 사라질지 모르니). 그래서 "객체에 부가 정보를 임시로 매달되, 객체가 죽으면 같이 치워지길" 원하는 캐시·메타데이터 용도에 쓴다.

=&gt; "순회가 필요하고 생명주기를 내가 관리" → Map/Set, "객체에 몰래 정보만 붙이고 메모리는 자동 정리" → WeakMap/WeakSet. (책 38장 이후 또는 심화 주제로 다뤄진다.)

</details>



# 38장. 브라우저의 렌더링 과정

 브라우저는 **① 서버에 리소스를 요청·응답받고 → ② HTML을 파싱해 DOM, CSS를 파싱해 CSSOM을 만들고 → ③ 둘을 합쳐 렌더 트리를 만든 뒤 → ④ 레이아웃을 계산하고 → ⑤ 화면에 페인팅**한다. 그 사이사이 자바스크립트가 끼어들어 DOM을 바꾸면 이 과정이 다시 일어난다(리플로우·리페인트). 

> `파싱(parsing)` 텍스트 문서를 읽어들여, 실행하기 좋도록 문법적 의미와 구조를 반영한 자료구조(파스 트리)를 생성하는 것.
>
> `렌더링(rendering)` HTML·CSS·자바스크립트로 작성된 문서를 파싱하여 브라우저에 **시각적으로 출력**하는 것.



## 요청과 응답

- 브라우저의 핵심 기능은 **필요한 리소스를 서버에 요청(request)하고, 응답(response)받아 화면에 렌더링**하는 것이다.
- 주소창에 URL을 입력하면, URL의 호스트 이름이 **DNS**를 통해 IP 주소로 변환되고, 그 IP의 서버로 요청이 전송된다.

리소스 요청은 명시적으로 하지 않아도 자동으로 여러 번 일어난다. 예를 들어 `index.html`을 받아 파싱하다가 `link`, `img`, `script` 태그를 만나면, 브라우저가 알아서 CSS·이미지·자바스크립트 파일을 추가로 요청한다.

```html
<!-- 이 태그들을 만날 때마다 브라우저가 해당 리소스를 추가로 요청한다 -->
<link rel="stylesheet" href="style.css" />
<img src="logo.png" />
<script src="app.js"></script>
```

=&gt; 즉 HTML 파싱과 리소스 요청은 **섞여서** 일어난다. 모든 요청·응답은 개발자 도구의 **Network 패널**에서 확인할 수 있다.

<details class="orca-details">
<summary>URL을 입력하면 화면이 뜨기까지 정확히 무슨 일이 일어날까?</summary>

유명한 면접 질문 "브라우저에 URL을 치면 무슨 일이 일어나나요?"의 요약이다.

1. **DNS 조회**: `example.com` 같은 도메인을 실제 서버 **IP 주소**로 변환한다.
2. **TCP 연결**: 그 IP와 연결을 맺는다(HTTPS면 TLS 핸드셰이크로 암호화 채널까지).
3. **HTTP 요청**: 브라우저가 `GET /` 같은 요청을 보낸다.
4. **서버 응답**: 서버가 `index.html`을 보내준다.
5. **렌더링**: 받은 HTML을 파싱하며, 중간에 만난 CSS·JS·이미지를 추가 요청하고, 이 장에서 배울 렌더링 과정을 거쳐 화면을 그린다.

=&gt; 이 장은 그중 **5번(렌더링)**에 집중한다. 1~4번은 네트워크 영역이고, 5번이 브라우저 렌더링 엔진의 일이다.

</details>



## HTTP 1.1과 HTTP 2.0

- **HTTP**는 웹에서 브라우저와 서버가 데이터를 주고받는 통신 규약(프로토콜)이다. 버전에 따라 성능 차이가 크다.

**HTTP/1.1**

- 기본적으로 **커넥션당 하나의 요청과 응답**만 처리한다.
- 여러 리소스를 동시에 보낼 수 없어서, 요청할 리소스 개수가 많을수록 응답 지연이 쌓인다.

**HTTP/2**

- **커넥션당 여러 요청·응답을 동시에** 처리(멀티플렉싱)한다.
- 그 덕에 같은 리소스를 받아도 HTTP/1.1보다 페이지 로드가 대략 50%가량 빠르다고 알려져 있다.

=&gt; 그래서 리소스를 하나로 합치거나(번들링) 개수를 줄이는 최적화가 HTTP/1.1 시절엔 특히 중요했다. HTTP/2에서는 동시 전송이 되므로 그런 압박이 줄었다.

<details class="orca-details">
<summary>HTTP/1.1의 &quot;Head of Line Blocking&quot;이 뭘까?</summary>

**앞선 요청이 늦어지면 뒤의 요청들이 줄줄이 막히는** 현상이다. HTTP/1.1의 고질적 한계다.

HTTP/1.1은 한 커넥션에서 요청을 보낸 순서대로 응답을 받아야 한다. 그래서 맨 앞 요청의 응답이 느리면, 그 뒤에 줄 서 있는 요청들도 전부 기다려야 한다. 마치 마트 계산대에서 앞사람이 느리면 뒷사람 전부 묶이는 것과 같다.

HTTP/2는 하나의 커넥션 안에서 요청·응답을 **잘게 쪼개 병렬로(멀티플렉싱)** 주고받아 이 문제를 해결했다.

=&gt; "HTTP/1.1은 한 줄로 순서대로, HTTP/2는 여러 개를 동시에"가 핵심 차이다. 그래서 작은 파일 여러 개를 받아야 할 때 HTTP/2의 이점이 크다.

</details>



## HTML 파싱과 DOM 생성

- 서버가 보내준 HTML 문서는 그냥 **순수한 텍스트(바이트)**다. 브라우저가 이걸 이해하고 조작하려면, 메모리에 올릴 수 있는 **객체 자료구조(DOM)**로 바꿔야 한다.

그 변환 과정은 다음 단계를 거친다.

```
바이트(Bytes) → 문자(Characters) → 토큰(Tokens) → 노드(Nodes) → DOM
```

1. **바이트 → 문자**: 서버가 바이트로 보낸 HTML을, `meta` 태그의 `charset`(예: UTF-8)에 따라 문자열로 변환한다.
2. **문자 → 토큰**: 문자열을 문법적 의미를 갖는 최소 단위인 **토큰**(`<html>`, `<body>` 등)으로 분해한다.
3. **토큰 → 노드**: 각 토큰을 객체인 **노드(DOM Node)**로 변환한다.
4. **노드 → DOM**: HTML 요소의 중첩(부자) 관계를 반영해 노드들을 **트리 구조**로 구성한다.

=&gt; 이렇게 만들어진 트리가 **DOM(Document Object Model)**이다. 즉, **DOM은 HTML 문서를 파싱한 결과물**이며, HTML 문서의 구조와 내용을 자바스크립트로 조작할 수 있게 해주는 자료구조다.

<details class="orca-details">
<summary>DOM이랑 HTML은 같은 거 아닐까? 뭐가 다를까?</summary>

**HTML은 "설계도(텍스트)", DOM은 그 설계도로 지은 "건물(메모리 속 객체 트리)"**이다. 둘은 다르다.

- **HTML**: 우리가 작성한 **정적인 텍스트 파일**. 서버에 저장돼 있고 바뀌지 않는다.
- **DOM**: 브라우저가 HTML을 파싱해 **메모리에 만든 객체 트리**. 자바스크립트로 실시간 조작이 가능하다.

그래서 자바스크립트로 `document.body.style.color = 'red'`를 하면 **DOM은 바뀌지만, 원본 HTML 파일은 그대로**다. 개발자 도구의 Elements 탭에서 보는 건 HTML 원문이 아니라 현재 DOM 상태다.

```js
// HTML 파일은 그대로, 메모리 속 DOM만 변경됨
document.querySelector('h1').textContent = '바뀐 제목';
```

=&gt; "HTML을 새로고침하면 원래대로 돌아온다"는 게 이 차이를 보여준다. DOM은 HTML에서 태어나지만, 그 뒤로는 독립적으로 변하는 살아있는 구조다.

</details>



## CSS 파싱과 CSSOM 생성

- 렌더링 엔진이 HTML을 위에서부터 순차 파싱하다가 CSS를 로드하는 `link` 태그나 `style` 태그를 만나면, **DOM 생성을 잠시 멈추고** CSS를 파싱하기 시작한다.
- CSS도 HTML과 똑같은 과정(바이트 → 문자 → 토큰 → 노드 → **CSSOM**)을 거쳐 객체 트리로 만들어진다.

```
바이트 → 문자 → 토큰 → 노드 → CSSOM
```

CSS 파싱이 끝나면, 멈췄던 지점부터 **HTML 파싱을 다시 이어간다.** 이렇게 만들어진 것이 **CSSOM(CSS Object Model)**이다. CSSOM은 CSS의 상속 관계까지 반영한 트리 구조다.

=&gt; 정리하면, HTML은 DOM으로, CSS는 CSSOM으로. 둘 다 "텍스트 → 객체 트리"라는 똑같은 변환을 거친다.

<details class="orca-details">
<summary>CSS가 &quot;렌더링을 막는다(render blocking)&quot;는 말이 무슨 뜻일까?</summary>

**CSSOM이 완성될 때까지 브라우저가 화면을 그리지 못한다**는 뜻이다. 그래서 CSS는 "렌더 블로킹 리소스"라 불린다.

이유는 분명하다. 스타일이 덜 로드된 채 화면을 그리면, 잠깐 **스타일 없는 날것의 페이지(FOUC, Flash of Unstyled Content)**가 번쩍 보였다가 바뀐다. 사용자 경험이 나쁘다. 그래서 브라우저는 CSSOM이 완성되어 렌더 트리를 만들 수 있을 때까지 페인팅을 미룬다.

=&gt; 그래서 CSS는 **`<head>` 안에서 일찍 로드**하는 게 좋다. 그래야 CSSOM을 빨리 만들어 화면 출력 지연을 줄인다. (반대로 자바스크립트는 뒤에서 보듯 `body` 끝이나 `defer`가 좋다.) "CSS는 위, JS는 아래"의 근거가 이것이다.

</details>



## 렌더 트리 생성

- 브라우저 렌더링 엔진은 DOM과 CSSOM을 **결합**하여 **렌더 트리(render tree)**를 생성한다.
- 렌더 트리는 **화면에 실제로 그려지는 노드만** 포함한다.

그래서 다음은 렌더 트리에서 제외된다.

- 화면에 시각적으로 나타나지 않는 노드: `meta`, `script` 태그 등
- CSS로 `display: none` 처리된 노드

```
DOM + CSSOM  →  렌더 트리(화면에 보이는 것만)
```

완성된 렌더 트리는 이후 두 단계의 입력으로 쓰인다.

1. **레이아웃(layout, 리플로우)**: 각 노드가 화면의 **어디에, 얼마나 큰 크기로** 배치될지 위치와 크기를 계산한다.
2. **페인트(paint)**: 계산된 레이아웃을 바탕으로 실제 **픽셀을 화면에 그린다**(색·테두리·그림자 등).

=&gt; 여기까지가 "처음 화면을 그리는" 과정이다. 문제는, 이 과정이 **단 한 번으로 끝나지 않는다**는 것이다. 자바스크립트가 DOM을 바꾸면 처음부터 다시 일어난다.

<details class="orca-details">
<summary>display: none이랑 visibility: hidden은 렌더 트리에서 똑같이 취급될까?</summary>

**다르다.** 이 차이가 렌더 트리 포함 여부를 가른다.

- `**display: none**`: 렌더 트리에서 **아예 제외**된다. 공간도 차지하지 않고, 그려지지도 않는다. (레이아웃에 없음)
- `**visibility: hidden**`: 렌더 트리에는 **포함**된다. 다만 보이지 않을 뿐, **공간은 그대로 차지**한다.

```css
.a { display: none; }      /* 사라짐 + 자리도 없음 */
.b { visibility: hidden; } /* 안 보임 + 자리는 유지 */
```

=&gt; "완전히 없애고 자리까지 없애려면 `display: none`, 안 보이게만 하고 자리는 남기려면 `visibility: hidden`". 참고로 `opacity: 0`도 `visibility: hidden`처럼 자리를 차지하지만, 이쪽은 클릭 같은 이벤트가 먹힌다는 차이가 있다.

</details>



## 자바스크립트 파싱과 실행

- 렌더링 엔진이 HTML을 파싱하다가 `script` 태그를 만나면, **DOM 생성을 멈추고 제어권을 자바스크립트 엔진에 넘긴다.**
- 자바스크립트 엔진은 코드를 파싱·실행한 뒤, 다시 렌더링 엔진에 제어권을 돌려주고 HTML 파싱을 이어간다.

자바스크립트 엔진의 처리 흐름은 이렇다.

1. **토크나이징(tokenizing)**: 소스코드 문자열을 의미 있는 최소 단위인 **토큰**으로 분해한다. (어휘 분석)
2. **파싱(parsing)**: 토큰들을 문법 구조에 맞춰 **AST(Abstract Syntax Tree, 추상 구문 트리)**로 만든다. (구문 분석)
3. **바이트코드 생성·실행**: AST를 엔진이 실행할 수 있는 중간 코드인 **바이트코드**로 변환해 실행한다.

=&gt; HTML이 DOM으로, CSS가 CSSOM으로 변환됐듯, 자바스크립트는 **AST**로 변환되어 실행된다. "텍스트 → 토큰 → 트리"라는 흐름은 셋 다 똑같다.

그리고 자바스크립트는 **DOM API**를 통해 이미 만들어진 DOM·CSSOM을 동적으로 조작할 수 있다. 바로 이 조작이 다음 절의 리플로우·리페인트를 일으킨다.

```js
// DOM API로 DOM을 조작 → 렌더링에 영향
document.getElementById('app').textContent = 'Hello';
```

<details class="orca-details">
<summary>AST가 뭘까? 왜 트리로 만들까?</summary>

**코드의 문법 구조를 나무 모양으로 표현한 자료구조**다. 컴파일러·엔진이 코드를 "이해"하기 위한 중간 형태다.

예를 들어 `const x = 1 + 2;`는 사람에겐 그냥 글자지만, 엔진은 이걸 "변수 선언문이고, 이름은 x, 값은 1과 2의 덧셈"이라는 **구조**로 파악해야 실행할 수 있다. 그 구조를 트리로 표현한 게 AST다.

```
VariableDeclaration
 └ VariableDeclarator
    ├ Identifier: x
    └ BinaryExpression: +
       ├ 1
       └ 2
```

=&gt; AST는 엔진 내부 얘기지만, 우리가 쓰는 도구의 바탕이기도 하다. **Babel**(트랜스파일), **ESLint**(코드 검사), **Prettier**(포매팅)가 전부 코드를 AST로 바꿔 분석·변형한다. "코드를 코드로 다루는" 도구들은 다 AST 위에서 돈다.

</details>



## 리플로우와 리페인트

- 자바스크립트가 DOM API로 DOM이나 CSSOM을 바꾸면, 변경된 내용으로 **렌더 트리가 다시 만들어지고**, 그에 따라 레이아웃과 페인트가 다시 일어난다.
- 이 "다시 그리기"가 **리플로우**와 **리페인트**다.

**리플로우(reflow)**

- **레이아웃을 다시 계산**하는 것. 요소의 위치·크기·구조가 바뀔 때 일어난다.
- 예: 노드 추가·삭제, 요소의 `width`·`height`·`margin`·`padding` 변경, 폰트 변경, 창 리사이징 등.

**리페인트(repaint)**

- 재계산된 렌더 트리를 바탕으로 **화면을 다시 칠하는** 것.
- 예: `color`, `background-color`, `visibility`처럼 **레이아웃에 영향 없는** 스타일만 바뀔 때.

=&gt; 중요한 포인트: **리플로우가 일어나면 리페인트는 반드시 뒤따른다**(위치·크기가 바뀌었으니 다시 칠해야 하니까). 하지만 리페인트만 단독으로 일어날 수도 있다(색만 바뀐 경우). 그래서 **리플로우가 리페인트보다 비싸다.**

<details class="orca-details">
<summary>리플로우/리페인트를 줄이려면 실무에서 뭘 조심해야 할까?</summary>

둘 다 비싼 작업이라, 자주 일으키면 화면이 버벅인다. 대표적인 줄이기 전략들이다.

1. **DOM 변경을 모아서 한 번에.** 반복문에서 DOM을 조금씩 여러 번 바꾸면 리플로우가 그만큼 반복된다. `DocumentFragment`에 모았다가 한 번에 붙이거나, 요소를 떼서(`display:none`) 수정 후 다시 붙인다.

```js
// 나쁨: 루프마다 리플로우 유발 가능
for (let i = 0; i < 100; i++) list.appendChild(makeItem(i));

// 나음: fragment에 모아 한 번만 반영
const frag = document.createDocumentFragment();
for (let i = 0; i < 100; i++) frag.appendChild(makeItem(i));
list.appendChild(frag);
```

2. **레이아웃 대신 그리기 속성으로 애니메이션.** `top`·`left`·`width`(리플로우)보다 `transform`·`opacity`(리플로우 없이 GPU 처리)로 움직이면 훨씬 부드럽다.
3. **레이아웃 값 읽기와 쓰기를 섞지 않기.** `offsetHeight` 같은 값을 읽으면 브라우저가 최신 레이아웃을 강제 계산하는데(**강제 동기 리플로우**), 읽기·쓰기를 번갈아 하면 이게 반복된다. 읽기는 읽기끼리, 쓰기는 쓰기끼리 모은다.

=&gt; 한마디로 "**변경은 모아서, 애니메이션은 transform/opacity로, 읽고 쓰기는 분리**". 리플로우가 리페인트보다 비싸다는 걸 알면 어디를 아껴야 할지 보인다.

</details>



## 자바스크립트 파싱에 의한 HTML 파싱 중단

- 렌더링 엔진과 자바스크립트 엔진은 **병렬이 아니라 직렬로**, 위에서 아래로 순차적으로 동작한다.
- 그래서 HTML 파싱 도중 `script` 태그를 만나면, **그 자바스크립트를 로드·파싱·실행할 때까지 HTML 파싱이 멈춘다.** 이를 블로킹(blocking)이라 한다.

이것이 두 가지 문제를 일으킨다.

1. **DOM 조작 에러**: `script`가 `head`나 위쪽에 있으면, 아직 생성되지 않은 아래쪽 DOM 요소를 조작하려다 에러가 난다.
2. **렌더링 지연**: 자바스크립트 로드·실행이 끝날 때까지 뒤쪽 HTML이 파싱되지 않아, 화면 출력이 늦어진다.

그래서 전통적인 해결책은 `**script` 태그를 `body`의 가장 아래**에 두는 것이다.

```html
<!DOCTYPE html>
<html>
<head>
  <link rel="stylesheet" href="style.css" />  <!-- CSS는 위에 -->
</head>
<body>
  <div id="app"></div>
  <!-- JS는 body 맨 아래 → DOM 다 만든 뒤 실행 -->
  <script src="app.js"></script>
</body>
</html>
```

=&gt; 이렇게 두면, DOM이 모두 완성된 뒤에 자바스크립트가 실행되므로 ① DOM 조작 에러가 없고, ② HTML 파싱이 자바스크립트에 막히지 않아 화면이 더 빨리 뜬다.

<details class="orca-details">
<summary>자바스크립트가 왜 HTML 파싱을 막도록(blocking) 설계됐을까?</summary>

**자바스크립트가 파싱 중인 DOM을 바꿔버릴 수 있기 때문**이다. 그래서 안전하게 멈추는 것이다.

자바스크립트는 `document.write()`나 DOM API로 문서 구조 자체를 바꿀 수 있다. 만약 HTML 파싱과 자바스크립트 실행이 동시에 진행된다면, 브라우저가 "지금 파싱 중인 이 부분을 자바스크립트가 바꾸면 어쩌지?" 하는 충돌이 생긴다.

그래서 브라우저는 안전하게 **"자바스크립트를 만나면 파싱을 멈추고, 자바스크립트를 먼저 끝낸 뒤 이어서 파싱"**하도록 직렬로 처리한다.

=&gt; 즉 블로킹은 버그가 아니라 **일관성을 지키기 위한 의도된 설계**다. 다만 그 부작용(지연)을 줄이려고 다음 절의 `async`/`defer`가 나왔다.

</details>



## script 태그의 async와 defer 어트리뷰트

- HTML 파싱 블로킹 문제를 더 깔끔하게 풀기 위해, HTML5에서 `script` 태그에 **`async`**와 `**defer**` 어트리뷰트가 추가됐다.
- 둘 다 **HTML 파싱과 자바스크립트 파일 로드(다운로드)를 동시에(비동기로)** 진행한다는 공통점이 있다. 차이는 **"언제 실행하느냐"**다.

```html
<script async src="app.js"></script>
<script defer src="app.js"></script>
```

**일반 script** — 로드도 실행도 파싱을 막는다.

- HTML 파싱 중단 → JS 로드 → JS 실행 → HTML 파싱 재개.

**async** — 로드는 비동기, 하지만 **로드되면 즉시 실행**(이때 파싱 중단).

- HTML 파싱과 동시에 JS 로드 → **로드 끝나는 즉시** 파싱 멈추고 실행 → 다시 파싱.
- 여러 개면 **로드 완료 순서대로** 실행 → **실행 순서 보장 안 됨.**
- 서로 **독립적인** 스크립트(예: 분석 도구, 광고)에 적합.

**defer** — 로드는 비동기, 실행은 **HTML 파싱이 다 끝난 뒤**(DOM 완성 후).

- HTML 파싱과 동시에 JS 로드 → 파싱 완전히 끝난 뒤 실행.
- 여러 개면 **작성한 순서대로** 실행 → **순서 보장됨.**
- DOM 조작이 필요하고 순서도 중요한 일반 스크립트에 적합. **대개 이게 권장**된다.


| 방식      | 로드      | 실행 시점        | 순서 보장 | 적합한 경우         |
| ------- | ------- | ------------ | ----- | -------------- |
| 일반      | 파싱 중단   | 로드 직후(파싱 중단) | O     | -              |
| `async` | 비동기(병렬) | 로드되는 즉시      | **X** | 독립적 스크립트       |
| `defer` | 비동기(병렬) | HTML 파싱 완료 후 | **O** | DOM 조작·일반 스크립트 |


<details class="orca-details">
<summary>그래서 async랑 defer 중 뭘 써야 할까?</summary>

**대부분은 `defer`가 정답**이고, `async`는 특수한 경우에만 쓴다.

- **`defer`를 쓸 때(기본값처럼)**: 내 코드가 DOM을 조작하거나, 다른 스크립트와 **순서**가 중요할 때. HTML 파싱이 끝난 뒤 작성 순서대로 실행되니 안전하다. 사실상 "`body` 맨 아래에 `script`" 하던 걸 `head`에서 `defer`로 깔끔하게 대체할 수 있다.
- **`async`를 쓸 때**: 다른 코드·DOM과 **완전히 독립적**인 스크립트. 구글 애널리틱스, 광고 스크립트처럼 "언제 실행되든, 순서가 어떻든 상관없고, 빨리 로드돼서 빨리 돌면 좋은" 것들.

```html
<!-- 분석 도구: 독립적이라 async -->
<script async src="analytics.js"></script>
<!-- 내 앱 코드: 순서·DOM 중요하니 defer -->
<script defer src="lib.js"></script>
<script defer src="app.js"></script> <!-- lib.js 다음에 실행 보장 -->
```

=&gt; "**순서·DOM 상관있으면 defer, 완전 독립이면 async**". 참고로 `type="module"` 스크립트는 **기본적으로 defer처럼** 동작한다(20장에서 본 모듈의 특성).

</details>

<details class="orca-details">
<summary>그럼 이제 body 맨 아래에 script 두는 방식은 안 써도 될까?</summary>

**`defer`가 상위 호환이라, 새 코드라면 `head`에서 `defer`를 쓰는 게 더 낫다.**

`body` 맨 아래 배치의 한계는 "스크립트 **다운로드**가 HTML 파싱이 거의 끝난 뒤에야 시작된다"는 점이다. 파일을 받는 동안 또 기다려야 한다.

반면 `head`에 `defer`로 두면, **HTML을 파싱하는 동안 스크립트를 미리 병렬로 다운로드**해두고, 파싱이 끝나면 바로 실행한다. 다운로드 시간이 파싱 시간과 겹쳐 전체가 더 빠르다.

```html
<head>
  <script defer src="app.js"></script> <!-- 파싱과 동시에 미리 다운로드 -->
</head>
```

=&gt; 결론: "`body` 끝 배치"는 `defer`가 없던 시절의 기법이다. 지금은 **`head`에 `defer`**가 더 효율적이다. 다만 둘 다 "DOM 완성 후 실행"이라는 목적은 같으니, 레거시 환경이면 `body` 끝 배치도 여전히 유효하다.

</details>
