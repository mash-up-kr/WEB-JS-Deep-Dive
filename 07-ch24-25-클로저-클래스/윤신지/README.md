# 24장. 클로저

클로저는 자바스크립트 고유의 개념이 아니라, 함수를 일급 객체로 취급하는 함수형 프로그래밍 언어(하스켈, 리스프, 스칼라 등)에서 사용되는 중요한 특성이다. 

> A closure is the combination of a function and the lexical environment within which that function was declared. (MDN)
>
> 클로저는 함수와 그 함수가 선언된 렉시컬 환경과의 조합이다.

=&gt; 즉 클로저를 이해하려면 먼저 **"함수가 선언된(정의된) 렉시컬 환경"**을 이해해야 한다. 그래서 이 장은 23장의 렉시컬 환경 이야기를 그대로 이어받는다. 렉시컬 스코프 → `[[Environment]]` → 클로저 순서로, "함수가 어떻게 자기가 태어난 환경을 기억하는가"를 따라가면 된다.



## 렉시컬 스코프

- 자바스크립트는 함수를 **어디서 호출**했는지가 아니라 **어디에 정의**했는지에 따라 상위 스코프를 정한다. 이를 **렉시컬 스코프(정적 스코프)**라 한다.
- 즉 상위 스코프는 함수 정의가 평가되는 시점에 **정적으로 결정**되고, 호출 위치는 아무 영향도 주지 않는다.

```js
const x = 1;

function foo() {
  const x = 10;
  bar(); // bar를 foo 안에서 "호출"
}

function bar() {
  console.log(x);
}

foo(); // 1
bar(); // 1
```

`bar`는 `foo` 안에서 호출됐지만 결과는 둘 다 `1`이다. `bar`가 **전역에서 정의**되었기 때문에, 상위 스코프는 언제나 전역 스코프다. "누가 불렀나"가 아니라 "어디서 태어났나"가 기준이다.

<details class="orca-details">
<summary>렉시컬 스코프랑 동적 스코프는 뭐가 다를까?</summary>

두 방식은 **상위 스코프를 결정하는 기준**이 정반대다.

- **렉시컬(정적) 스코프**: 함수를 **정의한 위치**로 상위 스코프 결정. (자바스크립트, 대부분의 언어)
- **동적 스코프**: 함수를 **호출한 위치**로 상위 스코프 결정.

위 예제를 동적 스코프로 해석했다면 `foo()` 안에서 부른 `bar()`는 `foo`의 `x`인 `10`을 봤을 것이다. 하지만 자바스크립트는 렉시컬 스코프라 정의 위치 기준으로 `1`이 나온다.

=&gt; "정적"이라 부르는 이유는, 코드를 실행하지 않고 **작성된 모양만 봐도** 상위 스코프를 알 수 있기 때문이다.

</details>



## 함수 객체의 내부 슬롯 [[Environment]]

- 함수가 렉시컬 스코프를 "기억"하는 방법이 바로 이 슬롯이다. 함수는 자신이 **정의된 환경(상위 스코프의 참조)**을 내부 슬롯 `[[Environment]]`에 저장한다.
- 이 값은 함수가 **호출될 때** 생성되는 함수 렉시컬 환경의 **"외부 렉시컬 환경에 대한 참조"** 값으로 그대로 쓰인다.
- 함수가 존재하는 한, 이 상위 스코프 참조도 계속 살아있다. → 이게 클로저의 씨앗이다.

```js
const x = 1;

function foo() {
  const x = 10;
  // bar의 [[Environment]]는 정의 시점(전역)의 렉시컬 환경을 가리킨다.
  bar();
}

function bar() {
  console.log(x);
}
```

함수 정의가 평가되어 함수 객체를 생성할 때, 자바스크립트 엔진은 **정의된 위치의 렉시컬 환경**을 함수 객체의 `[[Environment]]`에 저장한다. `foo`와 `bar`는 전역에서 정의됐으니 둘의 `[[Environment]]`는 전역 렉시컬 환경을 가리킨다.

<details class="orca-details">
<summary>`[[Environment]]`랑 &quot;외부 렉시컬 환경에 대한 참조&quot;는 같은 걸까?</summary>

거의 같은 걸 두 시점에서 부르는 이름이다.

- 함수 **정의** 시점: 상위 스코프를 함수 객체의 `[[Environment]]`에 **저장**해 둔다.
- 함수 **호출** 시점: 함수 렉시컬 환경이 새로 생기는데, 그 "외부 렉시컬 환경 참조" 칸에 아까 저장해 둔 `[[Environment]]`를 **복사해 꽂는다.**

=&gt; 즉 `[[Environment]]`는 "미리 적어둔 부모 주소", 외부 렉시컬 환경 참조는 "호출할 때 그 주소를 실제로 연결한 것". 이 연결이 **스코프 체인**의 실체다.

</details>

<details class="orca-details">
<summary>그럼 함수는 상위 스코프를 언제 기억할까?</summary>

**정의(평가)될 때** 기억한다. 호출 때가 아니다.

그래서 위 예제에서 `bar`를 `foo` 안에서 호출하든 전역에서 호출하든, `bar`가 기억하는 상위 스코프는 이미 "정의 시점(전역)"으로 굳어져 있다. 호출은 그 굳어진 기억을 꺼내 쓸 뿐이다.

=&gt; "정의 시점에 상위 스코프가 정적으로 결정된다"는 렉시컬 스코프의 원리가, 구현 레벨에서는 "정의 시점에 `[[Environment]]`에 저장한다"로 나타나는 것이다.

</details>



## 클로저와 렉시컬 환경

- **외부 함수보다 중첩 함수가 더 오래 살아남고, 그 중첩 함수가 외부 함수의 변수를 참조**하면 — 이미 끝난 외부 함수의 변수에 계속 접근할 수 있다. 이 중첩 함수를 **클로저**라 한다.
- 외부 함수의 실행 컨텍스트는 스택에서 사라지지만, **렉시컬 환경은 중첩 함수의 `[[Environment]]`가 붙잡고 있어** 가비지 컬렉션되지 않는다.
- 클로저가 참조하는 상위 스코프의 변수를 **자유 변수(free variable)**라 한다.

```js
const x = 1;

function outer() {
  const x = 10;
  const inner = function () { console.log(x); };
  return inner; // 함수를 반환
}

// outer를 호출하면 inner를 반환하고 outer의 생명 주기는 끝난다.
const innerFunc = outer();
innerFunc(); // 10
```

`outer`는 이미 실행을 마치고 실행 컨텍스트 스택에서 제거됐다. 그런데도 `innerFunc()`는 `outer`의 지역 변수 `x`(값 10)에 접근한다. `outer`의 렉시컬 환경이 살아있기 때문이다.

이유는 이렇다. `inner`의 `[[Environment]]`가 `outer`의 렉시컬 환경을 참조하고 있고, 반환된 `inner`(= `innerFunc`)가 살아있는 한 그 참조도 유지된다. **참조되는 렉시컬 환경은 가비지 컬렉션 대상이 아니므로** `x`도 사라지지 않는다.

<details class="orca-details">
<summary>MDN 정의 &quot;함수와 렉시컬 환경의 조합&quot;을 쉽게 풀면?</summary>

**"함수 + 그 함수가 기억하는 주변 변수들을 통째로 싸 들고 다니는 것"**이 클로저다.

`inner`라는 함수 하나만 반환된 것 같지만, 사실은 `inner`가 **자기가 태어난 방(outer의 렉시컬 환경)의 열쇠까지 같이** 들고 나온 것이다. 그래서 밖에서 불러도 그 방 안의 `x`를 꺼내 쓸 수 있다.

=&gt; "함수(inner) + 렉시컬 환경(outer의 변수들)"의 세트가 클로저. 함수만 달랑 있는 게 아니라 **환경까지 묶여 있다**는 게 핵심이다.

</details>

<details class="orca-details">
<summary>outer 실행이 끝났는데 x는 왜 안 사라질까? GC는 뭐 하고?</summary>

가비지 컬렉터는 "함수 실행이 끝나는 순간"이 아니라 **"더 이상 아무도 참조하지 않을 때"** 메모리를 회수한다.

`outer()`가 끝나면 보통은 그 지역 변수들이 회수 대상이 된다. 그런데 반환된 `inner`가 `outer`의 렉시컬 환경을 계속 참조하고 있으니, GC 입장에선 "아직 쓰는 사람이 있네" 하고 회수하지 않는다.

```js
const innerFunc = outer(); // innerFunc가 살아있는 한
innerFunc();               // outer의 x도 살아있다
```

=&gt; 즉 `x`가 안 사라지는 건 버그가 아니라, "참조가 남아있으면 회수하지 않는다"는 GC 규칙이 만들어내는 **의도된 동작**이다. 이게 클로저의 심장이다.

</details>

<details class="orca-details">
<summary>그럼 중첩 함수는 다 클로저인가?</summary>

아니다. 이론적으론 모든 함수가 상위 스코프를 기억하지만, 실무에서 **클로저라고 부르려면 두 조건**을 만족해야 한다.

1. 중첩 함수가 **상위 스코프의 식별자를 참조**하고,
2. 중첩 함수가 **외부 함수보다 더 오래 살아남을** 것.

상위 변수를 하나도 안 쓰거나, 외부 함수 안에서 바로 소멸하는 중첩 함수는 클로저라 부르지 않는다(엔진이 최적화로 상위 스코프를 아예 안 들고 있기도 하다).

=&gt; 한마디로 **"바깥 변수를 붙잡은 채, 바깥 함수보다 오래 사는 함수"**만 클로저다.

</details>

<details class="orca-details">
<summary>자유 변수(free variable)가 뭘까?</summary>

클로저가 **바깥 스코프에서 끌어다 쓰는 변수**를 자유 변수라 한다. 위 예제의 `x`가 그것이다.

"자유"라는 말은 "이 함수 자신의 매개변수도, 자신의 지역 변수도 아닌, 바깥에서 자유롭게 데려온 변수"라는 뉘앙스다. 클로저는 곧 **"자유 변수에 묶여 있는 함수"**라고도 표현한다.

</details>

<details class="orca-details">
<summary>클로저는 결국 메모리를 계속 잡고 있는 건데, 그럼 메모리 누수 아닌가?</summary>

**필요해서 붙잡고 있는 건 누수가 아니다.** 누수는 "안 쓰는데도 못 놓는 것"을 말한다. 클로저가 자유 변수를 붙잡는 건 그 변수를 실제로 쓰기 때문이라 정상이다.

다만 **다 쓴 클로저의 참조를 계속 들고 있으면** 그건 누수가 된다. 예를 들어 이벤트 핸들러로 등록한 클로저를 `removeEventListener` 없이 방치하면, 그 클로저가 붙잡은 큰 객체까지 회수가 안 된다.

```js
// 다 쓴 클로저는 참조를 끊어주면 GC가 회수한다
innerFunc = null;
```

=&gt; 그래서 클로저 자체를 겁낼 필요는 없고, **"수명이 끝난 클로저의 참조를 제때 끊는다"**는 습관만 있으면 된다.

</details>



## 클로저의 활용

- 클로저는 **상태(state)를 안전하게 은닉하고, 특정 함수에게만 변경을 허용**할 때 쓴다.
- 상태를 전역 변수로 두면 아무나 바꿀 수 있어 위험한데, 클로저 안에 가두면 **오직 반환된 함수만** 그 상태를 만질 수 있다.

```js
// 카운트 상태를 클로저에 가둔다
const increase = (function () {
  let num = 0; // 외부에서 직접 접근 불가 (자유 변수)
  return function () {
    return ++num;
  };
}());

console.log(increase()); // 1
console.log(increase()); // 2
console.log(increase()); // 3
```

`num`은 즉시 실행 함수 안에 갇혀 있어 밖에서 `num`을 직접 건드릴 방법이 없다. 오직 반환된 함수를 통해서만 `++num`이 일어난다. 전역 변수였다면 누구나 `num = 100`으로 망칠 수 있었을 것이다.

고차 함수로 증가/감소 같은 동작을 주입할 수도 있다.

```js
function makeCounter(aux) {
  let counter = 0; // 각 makeCounter 호출마다 독립적인 렉시컬 환경
  return function () {
    counter = aux(counter);
    return counter;
  };
}

function increase(n) { return ++n; }
function decrease(n) { return --n; }

const increaser = makeCounter(increase);
console.log(increaser()); // 1
console.log(increaser()); // 2

// 별도로 호출하면 완전히 독립된 counter를 갖는다
const decreaser = makeCounter(decrease);
console.log(decreaser()); // -1
```

<details class="orca-details">
<summary>왜 굳이 즉시 실행 함수로 감싸는 걸까?</summary>

**상태를 딱 한 번만 초기화하고, 그 상태를 바깥에서 못 건드리게** 하려고.

`num`을 그냥 전역에 두면 초기화도 여러 번 될 수 있고 누구나 접근한다. 즉시 실행 함수로 감싸면 `num`은 그 함수 스코프 안에 갇히고, 함수는 딱 한 번 실행되며, 반환된 클로저만 `num`에 접근할 수 있다.

=&gt; "초기화는 한 번, 접근은 통제된 창구로만." 이게 IIFE + 클로저 조합의 목적이다.

</details>

<details class="orca-details">
<summary>makeCounter를 두 번 호출하면 counter를 공유할까?</summary>

**공유하지 않는다.** 함수를 호출할 때마다 **새로운 렉시컬 환경**이 만들어지기 때문이다.

```js
const increaser = makeCounter(increase); // counter A
const decreaser = makeCounter(decrease); // counter B (완전 별개)
```

`increaser`와 `decreaser`는 각자 자기만의 `counter`를 붙잡은 별개의 클로저다. 그래서 한쪽을 올려도 다른 쪽엔 영향이 없다.

=&gt; "팩토리 함수를 여러 번 호출하면 상태도 여러 벌 생긴다"는 게 클로저 기반 상태 관리의 기본 성질이다.

</details>

<details class="orca-details">
<summary>React의 useState나 훅이 클로저랑 무슨 관계가 있을까?</summary>

관계가 아주 깊다. React 함수 컴포넌트와 훅은 **클로저 위에서 돌아간다.**

`useState`가 돌려주는 `setState`나, 이벤트 핸들러 안에서 참조하는 `state` 값은 전부 **그 렌더 시점의 렉시컬 환경을 붙잡은 클로저**다. 그래서 "오래된 state를 참조하는" 유명한 **stale closure(낡은 클로저)** 문제가 생긴다.

```js
function Counter() {
  const [count, setCount] = useState(0);
  useEffect(() => {
    const id = setInterval(() => {
      // 이 클로저는 처음 렌더의 count(0)를 계속 붙잡고 있다
      console.log(count);
    }, 1000);
    return () => clearInterval(id);
  }, []); // 의존성이 비어 있어 count가 갱신 안 됨
}
```

=&gt; `useEffect`의 의존성 배열, `useCallback`, 함수형 업데이트(`setCount(c => c + 1)`) 같은 게 전부 이 클로저 특성을 다루기 위한 장치다. 클로저를 이해하면 이 훅들의 동작이 한 번에 이해된다.

</details>



## 캡슐화와 정보 은닉

- **캡슐화**: 객체의 상태(프로퍼티)와 동작(메서드)을 하나로 묶는 것. 여기에 **정보 은닉**(외부에 감출 것은 감추기)까지 더한다.
- 자바스크립트는 `public`/`private` 같은 **접근 제한자가 없어** 기본적으로 모든 프로퍼티가 public이다.
- **클로저로 private을 흉내** 낼 수 있지만, "프로토타입 메서드 + 개별 상태 유지"는 클로저만으로는 깔끔하게 안 된다.

```js
function Person(name, age) {
  this.name = name; // public
  let _age = age;   // private (클로저로 은닉)

  // 인스턴스 메서드: _age에 접근하는 클로저
  this.sayHi = function () {
    console.log(`Hi! My name is ${this.name}. I am ${_age}.`);
  };
}

const me = new Person('Lee', 20);
me.sayHi();        // Hi! My name is Lee. I am 20.
console.log(me.name); // Lee
console.log(me._age); // undefined (밖에서 접근 불가)
```

`_age`는 생성자 함수의 지역 변수라 밖에서 못 본다. `sayHi`가 클로저라 `_age`에 접근할 수 있을 뿐이다. 다만 이 방식은 **인스턴스마다 `sayHi`가 중복 생성**되는 단점이 있다(프로토타입에 올리지 못하니까).

그럼 `sayHi`를 프로토타입 메서드로 올리면? 이번엔 `_age`를 공유하게 돼서 개별 상태가 깨진다.

```js
const Person = (function () {
  let _age = 0; // 모든 인스턴스가 공유하는 하나의 변수!

  function Person(name, age) {
    this.name = name;
    _age = age;
  }
  Person.prototype.sayHi = function () {
    console.log(`Hi! My name is ${this.name}. I am ${_age}.`);
  };
  return Person;
}());

const me = new Person('Lee', 20);
const you = new Person('Kim', 30);

me.sayHi();  // Hi! My name is Lee. I am 30. ← _age가 30으로 덮어써짐!
you.sayHi(); // Hi! My name is Kim. I am 30.
```

=&gt; 즉 클로저만으로는 **"메서드는 공유(프로토타입) + 상태는 인스턴스별로 은닉"**을 동시에 달성하기 어렵다. 이것이 클로저 기반 정보 은닉의 한계다.

<details class="orca-details">
<summary>왜 프로토타입 메서드로 올리면 _age가 공유돼 버릴까?</summary>

`_age`가 **즉시 실행 함수의 스코프에 딱 하나만** 존재하기 때문이다. 프로토타입 메서드 `sayHi`도 그 하나의 `_age`를 참조하는 하나의 클로저다.

그래서 `new Person('Kim', 30)`을 부르는 순간 `_age = 30`이 실행되어, 앞서 만든 `me`가 참조하던 `_age`까지 30으로 바뀐다. 인스턴스는 여러 개지만 `_age`는 한 개니까.

=&gt; "인스턴스별 상태"가 되려면 `_age`가 인스턴스마다 따로 있어야 하는데, 프로토타입 방식에선 하나로 공유되니 근본적으로 안 맞는 것이다.

</details>

<details class="orca-details">
<summary>그냥 `#` private 필드 쓰면 되잖아?</summary>

맞다. **ES2022의 클래스 private 필드(`#`)**가 이 한계를 언어 차원에서 깔끔하게 해결했다.

```js
class Person {
  #age; // 진짜 private 필드
  constructor(name, age) {
    this.name = name;
    this.#age = age;
  }
  sayHi() { // 프로토타입 메서드인데도
    console.log(`Hi! ${this.name}, ${this.#age}`); // 인스턴스별 #age 접근 OK
  }
}

const me = new Person('Lee', 20);
console.log(me.name); // Lee
// console.log(me.#age); // SyntaxError: 밖에서 접근 불가
```

`#age`는 인스턴스마다 따로 존재하면서 밖에서는 접근이 문법적으로 차단된다. 클로저로 흉내 내던 걸 언어가 정식 지원하는 셈이다.

=&gt; 그래도 클로저 기반 은닉을 배우는 이유는, (1) `#`이 없던 시절 코드/라이브러리가 여전히 많고, (2) 클래스가 아닌 함수·모듈 스코프에서의 은닉 원리가 결국 같은 클로저이기 때문이다.

</details>



## 자주 발생하는 실수

- 클로저를 오해해서 나오는 대표 버그가 `**var`로 도는 반복문 안에서 함수를 만드는 경우**다.
- `var`는 함수 레벨 스코프라 반복문의 `i`가 **하나뿐**이고, 모든 함수가 그 하나를 참조해 **끝값만** 나온다.
- 해결: `**let`(블록 스코프)** 쓰기 / IIFE로 값 가두기 / 고차 함수.

```js
var funcs = [];

for (var i = 0; i < 3; i++) {
  funcs[i] = function () { return i; };
}

console.log(funcs[0]()); // 3
console.log(funcs[1]()); // 3
console.log(funcs[2]()); // 3
```

`var i`는 전역(또는 함수) 변수 하나다. 반복이 끝나면 `i`는 3이 되고, 세 함수가 전부 그 **같은 `i` 하나**를 가리키니 모두 3이 나온다.

**해결 1 —** `let`**(가장 권장되는 방식)**

```js
const funcs = [];

for (let i = 0; i < 3; i++) {
  funcs[i] = function () { return i; };
}

console.log(funcs[0]()); // 0
console.log(funcs[1]()); // 1
console.log(funcs[2]()); // 2
```

`let`은 블록 레벨 스코프라, 반복할 때마다 **새로운 렉시컬 환경**이 생겨 그 시점의 `i`를 각각 캡처한다.

**해결 2 — 즉시 실행 함수로 값 가두기**

```js
for (var i = 0; i < 3; i++) {
  funcs[i] = (function (id) {
    return function () { return id; };
  }(i)); // 현재 i를 id로 복사해 가둔다
}
```

**해결 3 — 고차 함수(함수형)**

```js
const funcs = Array.from(new Array(3), (_, i) => () => i);
funcs.forEach(f => console.log(f())); // 0, 1, 2
```

<details class="orca-details">
<summary>var는 왜 3,3,3이고 let은 왜 0,1,2일까? 실행 컨텍스트로 설명하면?</summary>



</details>

# 25장. 클래스

자바스크립트는 프로토타입 기반 객체지향 언어라, 사실 클래스 없이도 생성자 함수와 프로토타입만으로 상속을 구현할 수 있다(19장). 하지만 클래스 기반 언어에 익숙한 개발자에겐 이 방식이 낯설다. 그래서 ES6에서 **클래스(class)**가 도입됐다.

=&gt; **"클래스는 그냥 생성자 함수를 예쁘게 쓴 것(문법적 설탕)일 뿐일까, 아니면 새로운 무언가일까?"** 결론부터 말하면 "거의 문법적 설탕이지만, 몇 가지 진짜 차이가 있다"이다. 그 차이가 뭔지를 따라가는 게 25장이다.



## 클래스는 프로토타입의 문법적 설탕인가?

- 클래스도 결국 **함수**이고, 내부적으로는 프로토타입 기반으로 동작한다. 그래서 "문법적 설탕(syntactic sugar)"에 가깝다.
- 하지만 생성자 함수와 **똑같지는 않다.** 다음 5가지 차이가 있다.


| 구분          | 클래스                            | 생성자 함수       |
| ----------- | ------------------------------ | ------------ |
| `new` 없이 호출 | **에러**                         | 일반 함수로 실행됨   |
| 상속          | `extends`·`super` 지원           | 미지원(직접 구현해야) |
| 호이스팅        | 호이스팅 안 되는 것처럼 동작(TDZ)          | 함수 호이스팅      |
| strict mode | **암묵적으로 항상 적용**                | 옵션           |
| 메서드 열거      | `[[Enumerable]]` false(열거 안 됨) | 열거됨          |


=&gt; 그래서 책은 클래스를 **"단순한 문법적 설탕이 아니라 새로운 객체 생성 메커니즘"**이라고 정리한다. 프로토타입 위에 얹혔지만, 더 엄격하고 편리한 규칙을 얹은 것이다.

<details class="orca-details">
<summary>그럼 클래스는 생성자 함수랑 거의 똑같은 거 아닐까?</summary>

동작 원리(프로토타입 기반)는 같지만, **더 안전하고 엄격하게** 만들어졌다는 게 다르다.

예를 들어 생성자 함수는 실수로 `new` 없이 호출하면 `this`가 전역 객체가 되어 조용히 버그를 만든다. 클래스는 아예 에러를 던져 그 실수를 막는다. 상속도 생성자 함수는 `Object.create`나 `call`로 손수 엮어야 했지만, 클래스는 `extends`·`super`로 언어가 지원한다.

=&gt; "겉포장만 바뀐 게 아니라, 위험한 실수를 막고 상속을 정식 지원하는 등 규칙이 더 튼튼해진 것"이라고 보면 된다.

</details>

<details class="orca-details">
<summary>왜 클래스 메서드는 열거가 안 되게 만들었을까?</summary>

메서드는 "동작"이지 "데이터"가 아니기 때문이다. `for...in`이나 `Object.keys`로 객체를 순회하는 건 보통 **데이터(프로퍼티)**를 훑으려는 것인데, 여기에 `sayHi` 같은 메서드까지 딸려 나오면 방해가 된다.

그래서 클래스의 메서드는 `[[Enumerable]]`이 `false`로 설정돼 열거 대상에서 빠진다. `Object.prototype`의 빌트인 메서드들이 열거되지 않는 것과 같은 이유다(19장 참고).

</details>



## 클래스 정의

- 클래스는 `class` 키워드로 정의하고, 이름은 파스칼 케이스(`Person`)를 쓴다.
- 클래스는 **일급 객체**라 변수에 담고, 함수 인자·반환값으로 쓸 수 있다.

```js
// 클래스 선언문
class Person {}

// 클래스 표현식 (익명 / 기명)
const Person = class {};
const Person = class MyClass {};
```

클래스 몸체에는 **constructor(생성자), 프로토타입 메서드, 정적 메서드** 이렇게 세 종류의 메서드만 정의할 수 있다.

<details class="orca-details">
<summary>클래스도 &quot;함수&quot;라던데, typeof를 찍으면 뭐가 나올까?</summary>

`'function'`이 나온다. 클래스는 사실 함수이기 때문이다.

```js
class Person {}
console.log(typeof Person); // function
```

정확히는, 클래스는 평가되어 **함수 객체가 되고**, `new`로 호출 가능한(constructor) 함수다. 그래서 `Person.prototype`도 있고 프로토타입 메서드도 거기에 올라간다. "클래스 = 프로토타입 기반"이라는 말의 근거가 이거다.

</details>



## 클래스 호이스팅

- 클래스는 런타임 이전에 평가되어 함수 객체가 되지만, `let`·`const`처럼 **TDZ(일시적 사각지대)**에 빠진다.
- 그래서 선언 전에 접근하면 **호이스팅이 안 된 것처럼** `ReferenceError`가 난다.

```js
console.log(Person); // ReferenceError
class Person {}
```

<details class="orca-details">
<summary>클래스는 호이스팅이 아예 안 되는 거 아닐까?</summary>

호이스팅은 **된다.** 다만 `let`·`const`와 똑같이 **선언 전 접근이 막힌** 상태(TDZ)라, 마치 호이스팅이 없는 것처럼 보일 뿐이다.

호이스팅이 정말 없다면 아래에서 전역 `Person`이 검색돼야 하는데, 실제로는 TDZ 때문에 에러가 난다.

```js
const Person = '';
{
  console.log(Person); // ReferenceError (var였다면 '')
  class Person {}
}
```

=&gt; "클래스 선언문도 런타임 이전에 평가는 되지만, 초기화 전이라 접근이 막힌다"가 정확한 표현이다.

</details>



## 인스턴스 생성

- 클래스는 생성자 함수이며, 반드시 `**new` 연산자와 함께** 호출해 인스턴스를 만든다.
- `new` 없이 호출하면 에러가 난다.

```js
class Person {}

const me = new Person();
console.log(me); // Person {}

// Person(); // TypeError: Class constructor Person cannot be invoked without 'new'
```



## 메서드

### constructor

- 인스턴스를 생성하고 **초기화**하는 특수한 메서드다. 클래스 몸체에 **딱 하나만** 둘 수 있다(생략 시 빈 constructor가 암묵 정의).
- `this`에 프로퍼티를 추가해 인스턴스를 초기화하고, 별도 반환문 없이 `**this`를 암묵 반환**한다.

```js
class Person {
  constructor(name) {
    this.name = name; // 인스턴스 프로퍼티 초기화
  }
}
console.log(new Person('Lee')); // Person {name: "Lee"}
```

<details class="orca-details">
<summary>constructor에서 return을 쓰면 어떻게 될까?</summary>

경우에 따라 다르다.

- **객체를 반환**하면 → 그 객체가 반환되고, 암묵적 `this` 반환이 **무시**된다.
- **원시값을 반환**하면 → 그 반환은 **무시**되고 `this`가 반환된다.

```js
class A {
  constructor() { this.a = 1; return { b: 2 }; }
}
console.log(new A()); // { b: 2 } ← 객체는 살아남음

class B {
  constructor() { this.a = 1; return 100; }
}
console.log(new B()); // B { a: 1 } ← 원시값 반환은 무시
```

=&gt; 그래서 constructor에서 의도적으로 `return`을 쓰는 건 혼란만 주니, 기본적으로 쓰지 않는 게 좋다.

</details>

### 프로토타입 메서드

- 클래스 몸체에 그냥 정의한 메서드는 **자동으로 `prototype`에 올라간다.** 생성자 함수처럼 `Class.prototype.method = ...`를 안 써도 된다.

```js
class Person {
  constructor(name) { this.name = name; }
  sayHi() { console.log(`Hi! ${this.name}`); } // 프로토타입 메서드
}

const me = new Person('Lee');
me.sayHi(); // Hi! Lee
console.log(Object.getPrototypeOf(me) === Person.prototype); // true
```

### 정적 메서드

- `static` 키워드로 정의하며, 인스턴스가 아니라 **클래스로 직접 호출**한다. 인스턴스로는 호출할 수 없다.

```js
class Person {
  static sayHello() { console.log('Hello!'); }
}

Person.sayHello();     // Hello!
// new Person().sayHello(); // TypeError
```

### 정적 메서드와 프로토타입 메서드의 차이

- **프로토타입 체인**: 정적 메서드는 클래스에, 프로토타입 메서드는 인스턴스에 붙는다.
- **호출**: 정적은 클래스로, 프로토타입은 인스턴스로.
- **this**: 정적은 클래스를, 프로토타입은 인스턴스를 가리킨다. 그래서 **정적 메서드는 인스턴스 프로퍼티를 참조할 수 없다.**

<details class="orca-details">
<summary>그래서 언제 static을 쓰는 게 좋을까?</summary>

**메서드 안에서 `this`, 즉 인스턴스 상태가 필요 없을 때** 정적으로 두는 게 좋다.

```js
class Square {
  static area(w, h) { return w * h; }   // 인스턴스 없이 쓰는 유틸 → static
}
console.log(Square.area(10, 10)); // 100
```

반대로 `this.width`처럼 인스턴스 상태를 다뤄야 하면 프로토타입 메서드다. `Math.max`, `Number.isInteger`처럼 애플리케이션 전역에서 쓰는 유틸리티 함수를 하나로 묶을 때 정적 메서드가 딱이다.

=&gt; "this가 필요 없으면 static, 필요하면 프로토타입"이라는 기준은 19장 정적 메서드와 완전히 같다.

</details>

<details class="orca-details">
<summary>클래스 메서드에 콤마도 없고 function도 없던데, 특징이 뭘까?</summary>

클래스 몸체의 메서드는 다음 특징을 갖는다.

1. `function` 키워드 생략(메서드 축약 표현).
2. 메서드 사이에 콤마 불필요.
3. 암묵적으로 **strict mode**로 실행(해제 불가).
4. `[[Enumerable]]`이 `false`라 열거 안 됨.
5. **non-constructor** — `new`로 호출할 수 없다.

=&gt; 특히 5번 때문에 `new me.sayHi()` 같은 건 에러다. 메서드는 인스턴스를 찍어내는 용도가 아니니까.

</details>



## 클래스의 인스턴스 생성 과정

`new Class()`를 호출하면 다음 3단계를 거친다.

1. **인스턴스 생성과 this 바인딩** — 빈 객체(인스턴스)를 만들고, 그 프로토타입을 `Class.prototype`으로 설정한 뒤 `this`에 바인딩한다. (constructor 코드 실행 전)
2. **인스턴스 초기화** — constructor 내부 코드가 실행되며 `this`에 프로퍼티를 추가한다.
3. **인스턴스 반환** — 완성된 인스턴스(`this`)가 암묵적으로 반환된다.

```js
class Person {
  constructor(name) {
    console.log(this);        // Person {} ← 1단계에서 이미 빈 인스턴스
    console.log(Object.getPrototypeOf(this) === Person.prototype); // true
    this.name = name;         // 2단계 초기화
    // 3단계: this 암묵 반환
  }
}
```

<details class="orca-details">
<summary>this는 언제 인스턴스에 바인딩될까? constructor 실행 전일까 후일까?</summary>

**constructor 코드가 실행되기 전(1단계)**에 이미 바인딩된다.

그래서 constructor 첫 줄에서 `console.log(this)`를 찍어보면 아직 아무 프로퍼티도 없는 `Person {}`이 나온다. 빈 인스턴스가 먼저 만들어지고 `this`에 연결된 다음, constructor가 그 `this`를 채워나가는 순서다.

=&gt; "빈 그릇(인스턴스)을 먼저 만들어 this에 쥐여주고 → constructor가 그 그릇을 채운다"고 이해하면 된다.

</details>



## 프로퍼티

### 인스턴스 프로퍼티

- constructor 내부에서 `this.x = ...`로 정의한다. 언제나 **public**이다.

```js
class Person {
  constructor(name) { this.name = name; }
}
console.log(new Person('Lee').name); // Lee
```

### 접근자 프로퍼티

- 자체 값 없이 **getter/setter** 함수로 이뤄진 프로퍼티다. `get`/`set` 키워드로 정의하며 프로토타입에 올라간다.

```js
class Person {
  constructor(first, last) { this.first = first; this.last = last; }
  get fullName() { return `${this.first} ${this.last}`; }
  set fullName(name) { [this.first, this.last] = name.split(' '); }
}

const me = new Person('Ungmo', 'Lee');
console.log(me.fullName); // Ungmo Lee (getter)
me.fullName = 'Heegun Lee'; // setter
console.log(me.first);    // Heegun
```

### 클래스 필드 정의

- 클래스 몸체에서 `name = 'Lee'`처럼 **필드를 직접 정의**할 수 있다(최신 표준). constructor 없이도 인스턴스 프로퍼티를 만든다.
- 단, 클래스 필드는 `**this`로만 접근**한다.

```js
class Person {
  name = 'Lee'; // 클래스 필드
  constructor() { console.log(this.name); } // Lee
}
```

<details class="orca-details">
<summary>클래스 필드 `name = 'Lee'`랑 constructor의 `this.name = ...`는 뭐가 다를까?</summary>

**결과는 같고, 쓰는 위치와 목적이 다르다.** 둘 다 인스턴스 프로퍼티를 만든다.

- **클래스 필드**: 초기값이 정해져 있거나 constructor 인자와 무관한 프로퍼티를 선언적으로 둘 때 깔끔하다.
- **constructor의 `this.x`**: 생성 시점의 **인자를 받아** 초기화할 때 쓴다.

```js
class Counter {
  count = 0;                 // 항상 0으로 시작 → 클래스 필드가 깔끔
  constructor(name) {
    this.name = name;        // 인자로 초기화 → constructor
  }
}
```

=&gt; 실무에선 둘을 섞어 쓴다. 고정 초기값은 클래스 필드, 인자 기반 초기화는 constructor.

</details>

<details class="orca-details">
<summary>클래스 필드에 화살표 함수를 할당하는 걸 종종 보던데 왜 그럴까?</summary>

`**this`를 인스턴스에 고정**하기 위해서다. 이벤트 핸들러로 메서드를 넘길 때 `this`가 풀리는 문제를 막는 기법이다.

```js
class App {
  count = 0;
  // 화살표 함수라 this가 언제나 이 인스턴스로 고정된다
  increase = () => { this.count++; };
}
const app = new App();
button.addEventListener('click', app.increase); // this 안 풀림
```

일반 프로토타입 메서드를 그냥 넘기면 호출 시점에 `this`가 바뀌어(22장) `undefined`가 되기 쉬운데, 클래스 필드 + 화살표 함수는 상위 스코프의 `this`를 캡처해 이 문제를 피한다.

=&gt; 다만 이 방식은 메서드가 **인스턴스마다 복사**돼 프로토타입 공유가 안 되니(19장), 남발하면 메모리 측면에선 손해다. 핸들러처럼 `this` 고정이 꼭 필요한 곳에만 쓰는 게 좋다.

</details>

### private 필드

- `#`을 붙이면 **private 필드**가 된다. 클래스 **내부에서만** 접근 가능하고, 밖에서 접근하면 문법 에러다.

```js
class Person {
  #name = '';
  constructor(name) { this.#name = name; }
  get name() { return this.#name; }
}

const me = new Person('Lee');
console.log(me.name);   // Lee (접근자로만)
// console.log(me.#name); // SyntaxError
```

<details class="orca-details">
<summary>24장에선 클로저로 private을 흉내 냈는데, `#`이랑 뭐가 다를까?</summary>

**목적은 같고, `#`은 언어가 정식 지원한다는 게 다르다.** 24장의 클로저 은닉은 "프로토타입 메서드 공유 + 인스턴스별 상태"를 동시에 못 하는 한계가 있었는데, `#` 필드는 그걸 깔끔하게 해결한다.

```js
class Person {
  #age; // 인스턴스마다 따로 존재
  constructor(name, age) { this.name = name; this.#age = age; }
  sayHi() { console.log(`${this.name}, ${this.#age}`); } // 프로토타입 메서드에서도 접근
}
```

`#age`는 인스턴스별로 존재하면서, 프로토타입 메서드에서도 접근되고, 밖에서는 문법적으로 차단된다. 클로저로는 셋을 동시에 만족하기 어려웠던 걸 `#`이 해낸다.

=&gt; 그래도 클로저 은닉을 배우는 이유는, `#`이 없던 시절 코드와 함수·모듈 스코프의 은닉 원리가 결국 같은 클로저이기 때문이다.

</details>

### static 필드

- `static` 키워드로 클래스 자신에 붙는 필드를 정의한다. static private(`static #x`)도 가능하다.

```js
class MyMath {
  static PI = 22 / 7;
  static #num = 10;
  static increment() { return ++MyMath.#num; }
}
console.log(MyMath.PI);          // 3.142857...
console.log(MyMath.increment()); // 11
```



## 상속에 의한 클래스 확장

- **상속**은 기존 클래스(수퍼클래스/부모)를 확장해 새 클래스(서브클래스/자식)를 만드는 것이다. `extends` 키워드로 구현한다.
- 프로토타입 기반 상속(19장)을 손수 엮던 것과 달리, `extends`·`super`로 **간결하게** 상속·확장할 수 있다.

```js
class Animal {
  constructor(age, weight) { this.age = age; this.weight = weight; }
  eat() { return 'eat'; }
  move() { return 'move'; }
}

class Bird extends Animal {
  fly() { return 'fly'; }
}

const bird = new Bird(1, 5);
console.log(bird.eat());  // eat (상속받음)
console.log(bird.fly());  // fly (자체)
console.log(bird instanceof Animal); // true
```

### 동적 상속

- `extends` 뒤에는 클래스뿐 아니라 `**[[Construct]]`를 갖는 함수 표현식**이 올 수 있다. 그래서 조건에 따라 상속 대상을 동적으로 정할 수 있다.

```js
function Base1() {}
class Base2 {}
let condition = true;

class Derived extends (condition ? Base1 : Base2) {}
console.log(new Derived() instanceof Base1); // true
```

### super 키워드

- `**super()` 호출**: 수퍼클래스의 constructor를 호출한다.
- `**super.method()` 참조**: 수퍼클래스의 메서드를 호출한다.

```js
class Base {
  constructor(name) { this.name = name; }
  sayHi() { return `Hi! ${this.name}`; }
}

class Derived extends Base {
  constructor(name, age) {
    super(name);      // 부모 constructor 호출
    this.age = age;
  }
  sayHi() {
    return `${super.sayHi()} I'm ${this.age}`; // 부모 메서드 호출
  }
}

console.log(new Derived('Lee', 20).sayHi()); // Hi! Lee I'm 20
```

super 사용에는 규칙이 있다.

- 서브클래스에 constructor를 두면 **반드시 `super()`를 호출**해야 한다.
- `**super()` 호출 전에는 `this`를 참조할 수 없다.**

<details class="orca-details">
<summary>super()를 안 부르면, 그리고 super 전에 this를 쓰면 왜 에러가 날까?</summary>

서브클래스는 **자기 인스턴스를 스스로 만들지 않고, 수퍼클래스의 constructor(`super()`)가 만들어 주기 때문**이다.

상속 클래스의 인스턴스 생성 과정을 보면, `new Derived()`를 해도 실제로 빈 인스턴스를 만들어 `this`에 바인딩하는 건 **수퍼클래스의 constructor**다. 그러니 `super()`를 부르기 전에는 `this`가 아직 존재하지 않아서 참조할 수 없고, `super()`를 아예 안 부르면 인스턴스가 만들어지지 않아 에러가 난다.

```js
class Derived extends Base {
  constructor() {
    // this.a = 1; // ReferenceError: super 전에 this 사용 불가
    super();
    this.a = 1;    // OK
  }
}
```

=&gt; "부모가 그릇을 만들어 줘야 자식이 그 그릇을 채운다"는 순서라, super가 먼저일 수밖에 없다.

</details>

<details class="orca-details">
<summary>super.sayHi()는 어떻게 &quot;부모 메서드&quot;를 정확히 찾아갈까?</summary>

메서드가 자신이 정의된 객체를 기억하는 내부 슬롯 `**[[HomeObject]]**` 덕분이다.

`super`는 "`[[HomeObject]]`의 프로토타입"을 찾아가는데, ES6 **메서드 축약 표현으로 정의한 메서드만** 이 `[[HomeObject]]`를 갖는다. 그래서 `super`는 축약 메서드 안에서만 쓸 수 있다.

```js
const obj = {
  foo() {},            // [[HomeObject]] 있음 → super 사용 가능
  bar: function () {}, // [[HomeObject]] 없음 → super 불가
};
```

=&gt; 즉 `super`가 부모를 찾아가는 건 "호출 위치"가 아니라 "메서드가 정의된 객체(`[[HomeObject]]`)"를 기준으로 하기 때문에 정확하다.

</details>

### 상속 클래스의 인스턴스 생성 과정

`new ColorRectangle()`을 호출하면 대략 이런 순서로 흐른다.

1. **서브클래스가 `super()` 호출** — 서브클래스는 인스턴스 생성을 수퍼클래스에 위임한다.
2. **수퍼클래스가 인스턴스 생성 + this 바인딩** — 실제 인스턴스를 만든다. 단 `new`로 호출된 건 서브클래스라, 생성되는 인스턴스 타입과 `new.target`은 **서브클래스**다.
3. **수퍼클래스가 인스턴스 초기화** — 부모 constructor가 `this`를 채운다.
4. **서브클래스 constructor로 복귀** — `super`가 반환한 인스턴스가 `this`에 바인딩된다.
5. **서브클래스가 인스턴스 초기화** — 자식만의 프로퍼티를 추가한다.
6. **완성된 인스턴스 반환.**

```js
class Rectangle {
  constructor(w, h) {
    console.log(new.target); // ColorRectangle ← new으로 부른 건 자식
    this.w = w; this.h = h;
  }
}
class ColorRectangle extends Rectangle {
  constructor(w, h, color) {
    super(w, h);        // 여기서 위 Rectangle constructor 실행
    this.color = color; // 자식 초기화
  }
}
console.log(new ColorRectangle(2, 4, 'red')); // ColorRectangle {w:2,h:4,color:'red'}
```

### 표준 빌트인 생성자 함수 확장

- `Array`, `String`, `Number` 같은 **표준 빌트인 객체도 `extends`로 상속**해 나만의 메서드를 얹을 수 있다.

```js
class MyArray extends Array {
  uniq() { return this.filter((v, i, self) => self.indexOf(v) === i); }
  average() { return this.reduce((p, c) => p + c, 0) / this.length; }
}

const arr = new MyArray(1, 1, 2, 3);
console.log(arr.uniq());    // MyArray(3) [1, 2, 3]
console.log(arr.average()); // 1.75
```

<details class="orca-details">
<summary>빌트인 확장이 신기한데, 예전 ES5에선 왜 어려웠을까?</summary>

ES5에서 생성자 함수로 `Array`를 상속하려 하면, `Array` 생성자가 만든 객체는 진짜 배열의 내부 동작(예: `length` 자동 갱신)을 온전히 물려받지 못했다. `Array.apply` 같은 편법을 써도 반쪽짜리였다.

ES6 클래스의 `extends Array`는 **엔진이 진짜 배열(exotic object)로 인스턴스를 만들어** 주기 때문에, `length` 자동 갱신 같은 배열 고유 동작까지 그대로 상속된다. 그래서 `MyArray`가 `filter`·`map`을 쓰면 반환값도 `MyArray` 인스턴스가 된다.

=&gt; "클래스는 문법적 설탕일 뿐"이라기엔, 이런 **빌트인 확장이 제대로 되는 것**이 클래스만의 실질적 이점 중 하나다.

</details>





