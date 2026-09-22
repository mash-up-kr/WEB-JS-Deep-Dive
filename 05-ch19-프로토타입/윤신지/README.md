# 19장. 프로토타입

> 자바스크립트는 명령형, 함수형, 프로토타입 기반 객체지향 프로그래밍을 지원하는 멀티 패러다임 언어이며, 자바스크립트를 이루고 있는 거의 **"모든 것"이 객체**다.


## 객체지향 프로그래밍

객체지향은 프로그램을 **객체(독립적 단위)의 집합**으로 표현하는 패러다임이다.

#### 속성이란

대상을 다른 대상과 구별해 주는 특징·성질. 사람이라면 이름·나이·성별 등.

#### 추상화란

여러 속성 중 **필요한 것만 간추려 표현**하는 것. 세부 구현은 감추고 핵심만 드러내 코드를 단순하게 만든다.

#### 객체란

- 상태 데이터(**프로퍼티**)와 동작(**메서드**)을 하나로 묶은 복합 자료구조.

```js
const circle = {
  radius: 5,              // 프로퍼티 (상태)
  getArea() {            // 메서드 (동작)
    return Math.PI * this.radius ** 2;
  },
};
```

<details class="orca-details">
<summary>프로퍼티랑 메서드가 뭐가 달라?</summary>

- **프로퍼티**: 객체의 **상태(값)**. "반지름은 5".
- **메서드**: 객체의 **동작(기능)**. 사실 "값이 함수인 프로퍼티"다.

=&gt; 명사(무엇을 가졌나)는 프로퍼티, 동사(무엇을 하나)는 메서드. 상속으로 아끼려는 대상은 주로 **모든 인스턴스가 똑같이 쓰는 메서드**다.

</details>



## 상속과 프로토타입

**📌 한눈에 정리**

- 생성자 함수 안에서 메서드를 만들면 **인스턴스마다 똑같은 메서드가 복사**돼 메모리가 낭비된다.
- 공유할 메서드는 생성자의 `prototype`에 **하나만** 올리고, 모든 인스턴스가 **상속**받아 쓴다.
- 결국 인스턴스는 **자기 상태(프로퍼티)만** 갖고, 공통 동작은 부모에서 빌려 쓴다.

```js
function Circle(radius) {
  this.radius = radius;
}
// prototype에 딱 하나만 올려 모든 인스턴스가 공유
Circle.prototype.getArea = function () {
  return Math.PI * this.radius ** 2;
};

const c1 = new Circle(1);
const c2 = new Circle(2);
console.log(c1.getArea === c2.getArea); // true (하나를 공유)
```

만약 `getArea`를 생성자 안에서 `this.getArea = ...`로 정의했다면 `c1.getArea === c2.getArea`는 `false`가 된다. 내용은 같아도 인스턴스마다 별개의 함수 객체가 생기기 때문이다.

<details class="orca-details">
<summary>메서드가 &quot;중복 생성&quot;되는 게 왜 문제야? 똑같은 함수잖아</summary>

내용은 같아도 자바스크립트에겐 **각각 다른 함수 객체**다(그래서 `===`가 `false`). 원을 10만 개 만들면 똑같은 계산 기능이 10만 벌 복사돼 메모리에 쌓인다.

=&gt; 상태(radius)는 각자 달라야 하니 어쩔 수 없지만, 동작은 다를 이유가 없다. "**상태는 각자, 동작은 공유**"의 창고가 프로토타입이다.

</details>

<details class="orca-details">
<summary>프로토타입이 결국 뭐야? 부모 객체라고 보면 돼?</summary>

절반은 맞다. **프로토타입 = 다른 객체에 공유 프로퍼티를 제공하는 상위(부모) 객체**다.

다만 클래스 기반 언어처럼 "설계도를 복제"하는 게 아니라, **실제 메모리에 존재하는 부모 객체를 링크(`[[Prototype]]`)로 연결**해두고 필요할 때 타고 올라가 빌려 쓴다. 이 "타고 올라가기"가 뒤의 **프로토타입 체인**이다.

</details>



## 프로토타입 객체

**📌 한눈에 정리**

- 모든 객체는 `[[Prototype]]`이라는 **숨은 링크**로 부모(프로토타입)를 가리킨다.
- 이 링크를 다루는 부품 3개:
  - `__proto__` : **객체**가 자기 부모에게 가는 문 (모든 객체)
  - `prototype` : **생성자 함수**가 "자식에게 물려줄 프로토타입"을 담는 주머니 (함수만)
  - `constructor` : **프로토타입**이 "날 만든 생성자"를 가리키는 이름표
- 즉 **객체 ↔ 프로토타입 ↔ 생성자 함수**가 이 셋으로 서로 연결된다.

<details class="orca-details">
<summary>`[[Prototype]]` 이중 대괄호는 뭐야? 코드에서 써?</summary>

`[[...]]`는 엔진이 내부적으로만 쓰는 **숨은 슬롯** 표기다. `obj.[[Prototype]]`처럼 직접 못 쓴다. 대신 `__proto__`(또는 `Object.getPrototypeOf`)라는 문으로 간접 접근한다.

=&gt; `**[[Prototype]]`은 실체(숨은 링크), `__proto__`는 그 실체를 여닫는 손잡이.**

</details>

### **proto** 접근자 프로퍼티

`__proto__`는 값을 담은 보통 프로퍼티가 아니라, **읽고 쓸 때마다 함수가 실행되는 접근자 프로퍼티**(`get`/`set`)다. 그리고 객체가 직접 소유한 게 아니라 `Object.prototype`에서 **상속받아** 쓴다.

```js
const obj = {};
console.log(obj.hasOwnProperty('__proto__')); // false (상속받은 것)
console.log({}.__proto__ === Object.prototype); // true
```

`__proto__`가 굳이 접근자(함수)로 감싸인 이유는 **위험한 프로토타입 연결을 쓰는 순간(set) 막기 위해서**다. 대표적으로 서로가 서로의 부모가 되는 상호 참조를 차단한다.

```js
const parent = {};
const child = {};
child.__proto__ = parent;
parent.__proto__ = child; // TypeError: Cyclic __proto__ value
```

이렇게 접근자 프로퍼티를 통해 접근·교체하도록 만든 이유는, 상호 참조로 프로토타입 체인이 생성되는 것을 방지하기 위해서다.

<details class="orca-details">
<summary>왜 순환(상호 참조) 프로토타입 체인이 생기면 안 될까?</summary>

프로토타입 체인은 **한쪽 방향으로만 흐르는 단방향 사슬**이어야 한다. 프로퍼티를 못 찾으면 부모 → 조부모로 올라가다가, 맨 끝(종점, `Object.prototype` → `null`)에서 검색을 멈춘다.

그런데 `child`의 부모가 `parent`, `parent`의 부모가 다시 `child`가 되면 —

```
child → parent → child → parent → child → ... (끝이 없음)
```

**종점이 사라져** 사슬이 뱅뱅 돈다. 이 상태에서 없는 프로퍼티를 찾으면 "부모로 올라가기"가 영원히 반복돼 **무한 루프**에 빠지고, 브라우저가 멈춘다.

=&gt; 그래서 엔진은 `set __proto__` 시점에 "이 연결이 사이클을 만드나?"를 검사해 `TypeError`로 막는다. 순환을 허용하면 검색이 끝날 수 없으니 원천 차단하는 것이다.

</details>

<details class="orca-details">
<summary>그럼 코드에서 `__proto__`를 직접 써도 돼?</summary>

권장하지 않는다. `Object.create(null)`로 만든 객체엔 `__proto__`가 아예 없을 수 있기 때문이다.

=&gt; 프로토타입을 **읽을 땐 `Object.getPrototypeOf(obj)`**, **바꿀 땐 `Object.setPrototypeOf(obj, proto)`**를 쓰는 게 안전하다.

</details>

### 함수 객체의 prototype 프로퍼티

함수 객체만 갖는 `prototype`은 **생성자가 만들 인스턴스의 프로토타입**을 가리킨다. 그래서 `new`로 호출할 수 없는 화살표 함수·축약 메서드(non-constructor)는 `prototype`이 없다.

```js
function Person(name) { this.name = name; }
const me = new Person('Lee');

// 생성자의 prototype과 인스턴스의 __proto__는 같은 프로토타입을 가리킨다.
console.log(Person.prototype === me.__proto__); // true
```

`__proto__`와 `prototype`은 같은 프로토타입을 가리키지만 **주체가 다르다.** `__proto__`는 모든 객체가 자기 부모에 접근하려고, `prototype`은 생성자가 자식의 부모를 지정하려고 쓴다.

### 프로토타입의 constructor 프로퍼티

프로토타입의 `constructor`는 자신을 만든 **생성자 함수**를 가리킨다. 이 연결은 함수 객체가 생성될 때 맺어진다.

```js
function Person(name) { this.name = name; }
const me = new Person('Lee');
console.log(me.constructor === Person); // true (Person.prototype에서 상속)
```

<details class="orca-details">
<summary>**proto** · prototype · constructor 이름이 헷갈려. 한 번에 정리해줘</summary>

`Person`으로 `me`를 만들었다고 하자.


| 이름            | 누가 가짐  | 무엇을 가리킴        | 비유               |
| ------------- | ------ | -------------- | ---------------- |
| `prototype`   | 생성자 함수 | 자식이 물려받을 프로토타입 | 엄마의 "물려줄 보따리"    |
| `__proto__`   | 모든 객체  | 자기 부모          | 자식이 "엄마에게 가는 문"  |
| `constructor` | 프로토타입  | 자길 만든 생성자      | 보따리의 "만든 사람" 이름표 |


```js
me.__proto__ === Person.prototype;        // true (자식 → 보따리)
Person.prototype.constructor === Person;  // true (보따리 → 만든 사람)
```

=&gt; **자식은 `__proto__`로 부모를 찾고, 생성자는 `prototype`으로 자식의 부모를 지정하고, 부모는 `constructor`로 자길 만든 생성자를 기억한다.**

</details>



## 리터럴 표기법에 의해 생성된 객체의 생성자 함수와 프로토타입

**📌 한눈에 정리**

- `{}`, `[]`, `function(){}`처럼 `new` 없이 리터럴로 만든 객체도 **프로토타입과 생성자 함수가 있다.**
- 다만 그 생성자는 상속을 위해 존재하는 **가상적인 짝**이다. (`{}`의 짝은 `Object`)
- 프로토타입과 생성자 함수는 **언제나 세트**라 리터럴도 예외가 아니다.

```js
const obj = {};
console.log(obj.constructor === Object); // true (짝은 Object)
```


| 리터럴            | 생성자 함수   | 프로토타입              |
| -------------- | -------- | ------------------ |
| `{}`           | Object   | Object.prototype   |
| `function(){}` | Function | Function.prototype |
| `[]`           | Array    | Array.prototype    |
| `/is/gi`       | RegExp   | RegExp.prototype   |


<details class="orca-details">
<summary>&quot;가상적인 생성자 함수&quot;가 무슨 말이야?</summary>

`{}`는 `new Object()`로 직접 만든 게 아닌데도 생성자가 `Object`로 잡힌다. 실제로 `Object()`를 호출한 건 아니지만, **상속을 위해 프로토타입이 필요하고 프로토타입은 늘 생성자와 짝**이라, 개념상 짝지어진 생성자를 "가상적"이라 부르는 것이다.

</details>

<details class="orca-details">
<summary>`{}` 랑 `new Object()` 는 완전히 똑같아?</summary>

결과물(프로토타입)은 사실상 같다. 둘 다 내부적으로 `OrdinaryObjectCreate`를 호출해 `Object.prototype`을 상속한다.

차이는 절차다. `new Object(123)`처럼 인수를 주면 래퍼 객체를 반환하는 분기가 있다.

=&gt; 실무에선 그냥 **리터럴 `{}`**를 쓴다. `new Object()`는 길고 인수에 따라 엉뚱해질 수 있어 거의 안 쓴다.

</details>



## 프로토타입의 생성 시점

**📌 한눈에 정리**

- 프로토타입은 **생성자 함수가 만들어지는 순간 같이 만들어진다.** (인스턴스보다 먼저 존재)
- **사용자 정의 생성자** → 함수 정의가 평가되는 시점.
- **빌트인 생성자**(`Object`·`Array`…) → 전역 객체 생성 시점(코드 실행 전).

### 사용자 정의 생성자 함수와 프로토타입 생성 시점

`new`로 호출 가능한 함수(constructor)는 함수 객체가 생성될 때 프로토타입도 함께 생긴다.

```js
console.log(Person.prototype); // {constructor: ƒ} ← new 하기도 전에 이미 있다
function Person(name) { this.name = name; }
```

### 빌트인 생성자 함수와 프로토타입 생성 시점

빌트인 생성자는 **전역 객체가 생성되는 시점**에 이미 만들어져 있다. 그래서 코드를 실행하기 전부터 `Array.prototype` 등이 존재한다.

<details class="orca-details">
<summary>&quot;엄마가 아기보다 먼저 생긴다&quot;는 게 코드로 무슨 뜻이야?</summary>

`new Person()`을 한 번도 안 했는데 `Person.prototype`이 이미 있다는 뜻이다. 함수 선언문은 런타임 이전에 평가되고, 그때 프로토타입도 짝으로 생긴다.

=&gt; 부모가 먼저 준비돼 있어야 자식이 태어나며 부모를 가리킬 수 있으니 순서가 자연스럽다.

</details>

<details class="orca-details">
<summary>화살표 함수·축약 메서드는 왜 prototype이 없을까?</summary>

`prototype`은 `**new`로 인스턴스를 찍어낼 수 있을 때만** 의미가 있다. non-constructor(화살표 함수, 축약 메서드)는 `new` 호출이 불가능 → 인스턴스를 안 만듦 → 물려줄 프로토타입도 불필요.

```js
const Arrow = () => {};
console.log(Arrow.prototype); // undefined
```

=&gt; 버그가 아니라 설계다. "자식을 낳지 않는 함수에 보따리를 쥐여줄 이유가 없다."

</details>



## 객체 생성 방식과 프로토타입의 결정

**📌 한눈에 정리**

- 객체 생성법은 여럿(리터럴, `Object()`, 생성자, `Object.create`, `class`)이지만, 속은 전부 **`OrdinaryObjectCreate`**라는 한 절차를 거친다.
- 이 절차에 **"부모로 삼을 프로토타입"을 인수로 넘기고**, 그게 곧 객체의 프로토타입이 된다.
- 즉 **어떤 방식으로 만드나 = 누가 부모가 되나**. (리터럴/`Object()` → `Object.prototype`, 생성자 → 그 생성자의 `prototype`)

### 객체 리터럴에 의해 생성된 객체의 프로토타입

리터럴로 만든 객체의 프로토타입은 `Object.prototype`이다. 그래서 `constructor`, `hasOwnProperty` 등을 상속받아 쓴다.

```js
const obj = { x: 1 };
console.log(obj.hasOwnProperty('x')); // true (Object.prototype에서 상속)
```

### Object 생성자 함수에 의해 생성된 객체의 프로토타입

`new Object()`도 프로토타입은 `Object.prototype`이다. 리터럴과의 차이는 **프로퍼티를 나중에 추가**해야 한다는 것뿐이다.

### 생성자 함수에 의해 생성된 객체의 프로토타입

생성자로 만든 객체의 프로토타입은 그 생성자의 `prototype`이다. 프로토타입도 객체라 프로퍼티를 추가하면 **모든 인스턴스에 즉시 반영**된다.

```js
function Person(name) { this.name = name; }
Person.prototype.sayHello = function () {
  console.log(`Hi! ${this.name}`);
};
new Person('Lee').sayHello(); // Hi! Lee
```

<details class="orca-details">
<summary>&quot;추상 연산 OrdinaryObjectCreate&quot;가 대체 뭐야?</summary>

명세서가 엔진 동작을 설명하는 **가상의 함수**다. 실제 코드로 존재하진 않는다. 하는 일은 대략:

```
1. 빈 객체 생성
2. (있으면) 프로퍼티 추가
3. [[Prototype]] 슬롯에 인수로 받은 proto 꽂기
4. 반환
```

=&gt; 우리가 어떤 문법으로 만들든 엔진은 이 부품을 호출하고 **3번에서 넘길 부모(proto)만 다르게** 정한다. "문법은 껍데기, 속은 같은 부품 + 부모만 교체."

</details>



## 프로토타입 체인

**📌 한눈에 정리**

- 프로퍼티를 찾을 때 자기 자신에 없으면 `[[Prototype]]`을 타고 **부모 → 조부모**로 올라가며 찾는다. 이 사슬이 프로토타입 체인.
- 종점은 언제나 `Object.prototype`이고 그 부모는 `null`. 여기까지 못 찾으면 `undefined`(에러 아님).
- **프로토타입 체인 = 상속·프로퍼티 검색** / **스코프 체인 = 식별자(변수) 검색**. 둘은 협력한다.

```js
function Person(name) { this.name = name; }
const me = new Person('Lee');

// me엔 없지만 Object.prototype까지 타고 올라가 찾는다
console.log(me.hasOwnProperty('name')); // true
// me → Person.prototype → Object.prototype → null
```

> `프로토타입 체인` 객체의 프로퍼티에 접근할 때 없으면 `[[Prototype]]`의 참조를 따라 부모의 프로퍼티를 순차 검색하는 것.

<details class="orca-details">
<summary>&quot;종점&quot;이 뭐야? 없는 프로퍼티를 찾아도 왜 에러가 안 나?</summary>

종점은 사슬의 맨 끝 정거장 `Object.prototype`(그 부모는 `null`). 여기까지 뒤져도 없으면 자바스크립트는 조용히 `undefined`를 준다. **"못 찾음 ≠ 에러"**.

```js
console.log(me.foo); // undefined (에러 아님)
```

=&gt; 편하지만, 오타를 내도 `undefined`가 흘러가 버그를 늦게 발견하는 양날의 검. 그래서 `in`/`hasOwnProperty`로 존재를 확인한다.

</details>

<details class="orca-details">
<summary>배열도 Object.prototype 메서드를 쓸 수 있어?</summary>

그렇다. 배열의 체인은 `[] → Array.prototype → Object.prototype → null`이라, `push`·`map`(Array) 위에 `hasOwnProperty`·`toString`(Object)도 상속받는다.

=&gt; 함수·배열·정규식 전부 각자 프로토타입을 거쳐 결국 `Object.prototype`으로 수렴한다. 이게 "거의 모든 것이 객체"의 실체다.

</details>

<details class="orca-details">
<summary>체인이 길어지면 성능에 영향 있어?</summary>

이론상 있다(한 단계씩 올라가며 찾으므로). 하지만 실무에선 걱정할 일이 거의 없다. 보통 체인은 2~3단계고, V8은 인라인 캐시·히든 클래스로 반복 접근을 최적화한다.

=&gt; 오히려 프로토타입을 이상하게 교체해 **엔진 최적화를 깨는 쪽**이 성능에 더 나쁘다.

</details>



## 오버라이딩과 프로퍼티 섀도잉

**📌 한눈에 정리**

- 프로토타입과 **같은 이름**을 인스턴스에 추가하면 덮어쓰는 게 아니라 인스턴스에 **새로 생긴다.**
- 그 결과 인스턴스 것이 먼저 검색돼 프로토타입 것을 **가린다** → **섀도잉**(가림), **오버라이딩**(재정의).
- 인스턴스로는 프로토타입 프로퍼티를 **읽기(get)만** 되고 **변경·삭제(set/delete)는 안 된다.** 진짜 바꾸려면 프로토타입에 직접 접근.

```js
me.sayHello = function () { console.log('Hey!'); }; // 인스턴스에 추가
me.sayHello(); // Hey! (인스턴스 것이 이김 → 프로토타입 것은 가려짐)

delete me.sayHello; // 인스턴스 것만 지워짐
me.sayHello(); // 다시 프로토타입 메서드 (delete me.sayHello로는 못 지움)
```

<details class="orca-details">
<summary>오버라이딩이랑 섀도잉은 같은 말이야, 다른 말이야?</summary>

같은 사건을 두 관점에서 부르는 이름이다. **인스턴스 관점 = 오버라이딩**("부모 메서드를 내 걸로 재정의"), **프로토타입 관점 = 섀도잉**("내 메서드가 가려짐").

=&gt; **덮어쓰는 동작 = 오버라이딩, 그 결과 가려지는 현상 = 섀도잉.** 원본은 그대로 있어 인스턴스 것만 걷어내면 다시 보인다.

</details>

<details class="orca-details">
<summary>왜 인스턴스로는 프로토타입 프로퍼티를 &quot;읽기만&quot; 될까?</summary>

get과 set의 규칙이 다르다. **읽기**는 체인을 타고 올라가 부모 값을 빌려온다. **쓰기**는 체인을 안 타고 **무조건 그 객체 자신에게** 만든다.

```js
me.name = 'Kim'; // Person.prototype.name을 바꾸는 게 아니라
console.log(me.hasOwnProperty('name'));  // true (내 것이 생김)
console.log(Person.prototype.name);      // 그대로
```

=&gt; 덕분에 한 인스턴스가 실수로 프로토타입을 건드려 **형제 전부에 영향 주는 사고**가 막힌다.

</details>



## 프로토타입의 교체

**📌 한눈에 정리**

- 부모(프로토타입)를 나중에 다른 객체로 갈아끼울 수 있다.
  - **생성자로 교체**: `Person.prototype = {...}` → 앞으로 만들 인스턴스에 적용.
  - **인스턴스로 교체**: `Object.setPrototypeOf(me, {...})` → 이미 만든 객체에 적용.
- 둘 다 `constructor` 연결이 끊어진다.
- **결론: 직접 교체는 번거롭고 위험하니 쓰지 말 것.** 상속이 필요하면 `Object.create`나 `class`.

### 생성자 함수에 의한 프로토타입의 교체

```js
function Person(name) { this.name = name; }
Person.prototype = { sayHello() {} }; // 통째로 교체

const me = new Person('Lee');
console.log(me.constructor === Person); // false ← 연결 끊김!
// 되살리려면 교체 객체에 constructor: Person 을 직접 넣어야 함
```

### 인스턴스에 의한 프로토타입의 교체

`Object.setPrototypeOf(me, parent)`(= `me.__proto__ = parent`)는 **이미 생성된 객체**의 부모를 바꾼다. 생성자 교체와 달리 생성자의 `prototype`은 그대로라 둘이 어긋날 수 있다.

<details class="orca-details">
<summary>왜 그렇게까지 직접 교체를 말릴까?</summary>

1. `constructor` 연결이 깨져 매번 수동 복구가 필요하다.
2. 생성자의 `prototype`과 인스턴스의 `__proto__`가 어긋나 추적이 어렵다.
3. 런타임에 부모를 갈아끼우면 V8의 히든 클래스 최적화가 깨져 느려진다(MDN도 경고).

=&gt; "이미 만든 객체의 부모 갈아끼우기"는 안티패턴. 상속은 처음부터 `Object.create`나 `class extends`로.

</details>

<details class="orca-details">
<summary>constructor 연결이 끊기면 실제로 뭐가 문제야?</summary>

`me.constructor`로 "누가 날 만들었나"를 되짚는 코드나, 그걸 이용해 같은 종류의 객체를 새로 만드는 패턴이 오작동한다. 또 디버깅 때 인스턴스의 정체를 파악하기 어려워진다.

=&gt; 다만 `instanceof`는 `constructor`가 아니라 `prototype` 체인을 보므로 별개다(아래 절 참고).

</details>



## instanceof 연산자

**📌 한눈에 정리**

- `객체 instanceof 생성자함수` → **생성자의 `prototype`이 객체의 프로토타입 체인에 있으면 `true`.**
- `constructor`(이름표)가 아니라 **체인에 `prototype` 객체가 끼어 있는지**를 본다.
- 그래서 프로토타입 교체로 `constructor`가 끊겨도, `prototype`만 체인에 있으면 `true`.

```js
function Person(name) { this.name = name; }
const me = new Person('Lee');
console.log(me instanceof Person); // true
console.log(me instanceof Object); // true (체인 끝 Object.prototype)
```

<details class="orca-details">
<summary>me instanceof Object 는 왜 true야? me는 Person으로 만들었잖아</summary>

`instanceof`는 "직접 만들었나?"가 아니라 **"이 생성자의 `prototype`이 내 조상 줄에 있나?"**를 묻는다. `me`의 조상 줄은 `me → Person.prototype → Object.prototype → null`이라 둘 다 있다.

=&gt; "나는 엄마의 자식이자 할머니의 손주". 거의 모든 객체가 `instanceof Object`가 `true`인 이유다.

</details>

<details class="orca-details">
<summary>me instanceof Person 이랑 me.constructor === Person 은 뭐가 달라?</summary>

- `instanceof`: `Person.prototype`이 **체인 어딘가**에 있는지(상속 관계 전체).
- `constructor === Person`: 체인에서 찾은 `constructor` 값이 **정확히 Person인지**(단순 비교).

프로토타입을 교체하면 `constructor`는 쉽게 깨지지만 `instanceof`는 `prototype`만 이어주면 유지된다.

=&gt; "이 객체가 그 생성자의 인스턴스냐"는 보통 `**instanceof**`. 단 iframe 등 컨텍스트가 다르면 오작동하니 배열은 `Array.isArray()`를 쓴다.

</details>



## 직접 상속

**📌 한눈에 정리**

- **직접 상속** = 객체를 만들 때 **부모(프로토타입)를 콕 집어** 지정.
- `Object.create(proto)` → `new` 없이 원하는 부모로 생성.
- ES6부터는 리터럴 안에서 `__proto__: proto`로 더 간단히.
- 프로토타입 교체보다 **깔끔·안전**해 손수 상속할 땐 이 방식.

### Object.create에 의한 직접 상속

```js
const myProto = { x: 10 };
const obj = Object.create(myProto); // obj → myProto → Object.prototype → null
console.log(obj.x); // 10
console.log(Object.getPrototypeOf(obj) === myProto); // true
```

장점: `new` 없이 생성 / 프로토타입을 지정하며 생성 / 리터럴 객체도 상속 가능.

### 객체 리터럴 내부에서 __proto__에 의한 직접 상속

```js
const myProto = { x: 10 };
const obj = {
  y: 20,
  __proto__: myProto, // obj → myProto → Object.prototype → null
};
console.log(obj.x, obj.y); // 10 20
```

<details class="orca-details">
<summary>Object.create(proto)랑 new Person()이랑 뭐가 달라?</summary>

- `new Person()`: 부모가 `**Person.prototype`으로 고정** + 생성자 몸체(`this.name=...`)까지 **실행**.
- `Object.create(proto)`: 부모를 **내가 지정** + 생성자 실행 없이 "그 부모를 가진 빈 객체"만 생성.

=&gt; `new`는 "조립+초기화까지 해주는 편의 코스", `Object.create`는 "부모만 정해 빈 몸체를 뽑는 저수준 코스".

</details>

<details class="orca-details">
<summary>Object.create(null)로 &quot;부모 없는 객체&quot;를 왜 일부러 만들까?</summary>

`Object.prototype`조차 상속 안 하는 순수 빈 객체가 된다. **순수한 사전(dictionary)**으로 쓸 때 유용하다. 일반 객체는 물려받은 `constructor`·`hasOwnProperty` 등과 키가 충돌할 수 있는데, 이게 없으니 안전하다.

```js
const dict = Object.create(null);
console.log('constructor' in dict); // false (깨끗함)
```

=&gt; 외부 입력을 키로 받는 맵/캐시에 쓴다(요즘은 `Map`도 많이 씀). 대신 `hasOwnProperty`가 필요하면 `Object.prototype.hasOwnProperty.call(dict, key)`로 간접 호출.

</details>



## 정적 프로퍼티/메서드

**📌 한눈에 정리**

- **정적(static)** = 생성자 함수 **자신**이 가진 것. `Person.staticMethod()`로 호출.
- 프로토타입 체인에 없으므로 **인스턴스로는 호출 불가.**
- 판단 기준: 메서드 안에서 `**this`(인스턴스 상태)가 필요 없으면 정적**, 필요하면 프로토타입 메서드.

```js
function Person(name) { this.name = name; }
Person.staticMethod = function () { console.log('static'); };

Person.staticMethod();      // static
const me = new Person('Lee');
me.staticMethod();          // TypeError (인스턴스는 못 씀)
```

예: `Object.create`·`Array.isArray`(정적) ↔ `arr.push`·`obj.hasOwnProperty`(프로토타입).

<details class="orca-details">
<summary>정적 메서드는 언제 쓰는 게 좋아? 실무 예시</summary>

**특정 인스턴스 상태(this)에 의존하지 않는 "그 종류 전체의 유틸리티"**일 때.

```js
class Circle {
  constructor(r) { this.r = r; }
  getArea() { return Math.PI * this.r ** 2; }        // this 씀 → 프로토타입
  static isValidRadius(r) { return typeof r === 'number' && r > 0; } // 정적
}
Circle.isValidRadius(5); // 인스턴스 없이 호출
```

=&gt; 팩토리 메서드(`User.fromJSON`)·유효성 검사(`Validator.isEmail`)가 대표적 활용처.

</details>



## 프로퍼티 존재 확인

**📌 한눈에 정리**

- `**in**`(또는 `Reflect.has`) → **상속받은 것까지 포함**해 있으면 `true`. (`'toString' in obj` → `true`)
- `**hasOwnProperty**` → **객체 자신의 것만** 확인. 상속받은 건 `false`.

### in 연산자

```js
const person = { name: 'Lee' };
console.log('name' in person);      // true
console.log('toString' in person);  // true (Object.prototype에서 상속)
```

### Object.prototype.hasOwnProperty 메서드

```js
console.log(person.hasOwnProperty('name'));     // true
console.log(person.hasOwnProperty('toString')); // false (상속받은 것)
```

<details class="orca-details">
<summary>그래서 in이랑 hasOwnProperty 중 뭘 써?</summary>

"내가 진짜 넣은 프로퍼티냐"를 볼 땐 `**hasOwnProperty**`(실무에서 이게 더 잦다). "쓸 수 있냐(상속 포함)"는 `in`.

안전하게는 `Object.prototype.hasOwnProperty.call(obj, key)` 또는 ES2022의 `**Object.hasOwn(obj, key)**`.

</details>



## 프로퍼티 열거

**📌 한눈에 정리**

- `**for...in**` → 상속받은 프로퍼티까지 열거(단 `[[Enumerable]]`이 `true`인 것만, 심벌 제외). 내 것만 원하면 `hasOwnProperty` 필터 필요.
- `**Object.keys/values/entries**` → **객체 자신의** 열거 가능 프로퍼티만 배열로. 필터 불필요 → 기본 선택지.
- 배열은 `for...in` 말고 `for...of`·`forEach`.

### for...in 문

```js
const person = { name: 'Lee', address: 'Seoul', __proto__: { age: 20 } };

for (const key in person) {
  if (!person.hasOwnProperty(key)) continue; // 상속분 제외
  console.log(key); // name, address
}
```

### Object.keys/values/entries 메서드

```js
console.log(Object.keys(person));    // ["name", "address"]
console.log(Object.values(person));  // ["Lee", "Seoul"]
console.log(Object.entries(person)); // [["name","Lee"], ["address","Seoul"]]
```

<details class="orca-details">
<summary>for...in, Object.keys, for...of 언제 뭘 써?</summary>

- **배열·문자열·Map·Set 순회** → `for...of`(또는 `forEach`·`map`)
- **객체의 키·값 순회** → `Object.keys/values/entries`
- `**for...in**` → 상속분까지 훑어야 하는 특수한 경우만. 평소엔 거의 안 씀.

</details>

<details class="orca-details">
<summary>class가 있는데 왜 프로토타입을 깊게 공부해?</summary>

**ES6 `class`는 프로토타입을 감싼 문법적 설탕**이라, 속은 전부 프로토타입 체인으로 돈다.

```js
class Dog extends Animal { bark() {} }
const d = new Dog();
console.log(d.__proto__ === Dog.prototype);            // true
console.log(Dog.prototype.__proto__ === Animal.prototype); // true
```

=&gt; 상속·`this`·`super`·`instanceof`·버그 디버깅을 정확히 이해하려면 결국 프로토타입을 알아야 한다. `class`는 겉포장, 프로토타입은 **뿌리**다.

</details>



## 19장 한 장 정리


| 키워드                             | 한 줄 요약                                           |
| ------------------------------- | ------------------------------------------------ |
| **프로토타입**                       | 인스턴스들이 공유할 프로퍼티(주로 메서드)를 담는 부모 객체                |
| `**[[Prototype]]**`             | 객체마다 달린, 부모를 가리키는 숨은 링크(엔진 전용)                   |
| `**__proto__**`                 | 그 링크를 여닫는 손잡이(모든 객체)                             |
| `**prototype**`                 | 생성자가 "자식에게 물려줄 프로토타입"을 담는 프로퍼티(함수만)              |
| `**constructor**`               | 프로토타입이 "날 만든 생성자"를 가리키는 이름표                      |
| **프로토타입 체인**                    | 못 찾으면 부모→조부모로 올라가며 찾는 사슬(종점: `Object.prototype`) |
| **섀도잉**                         | 인스턴스에 같은 이름을 추가해 프로토타입 것이 가려짐                    |
| **프로토타입 교체**                    | 부모를 통째로 갈아끼우기 — `constructor` 깨져 비권장             |
| `**instanceof**`                | 생성자의 `prototype`이 내 체인에 있는지 확인                   |
| `**Object.create**`             | 부모를 직접 지정해 생성(직접 상속)                             |
| **정적 멤버**                       | 생성자 자신이 가진 것 — 인스턴스는 못 씀(`this` 불필요할 때)          |
| `**in` vs `hasOwnProperty`**    | 상속 포함 확인 vs 내 것만 확인                              |
| `**for...in` vs `Object.keys`** | 상속까지 열거 vs 내 것만 열거(권장)                           |


<details class="orca-details" open>
<summary>결국 한 문장으로 &quot;프로토타입이 뭐냐&quot;</summary>

**"인스턴스마다 복사하면 낭비인 공통 기능을, 부모 객체 한 곳에 두고 링크(`[[Prototype]]`)를 타고 올라가 빌려 쓰게 하는 구조."**

- 왜? → 중복(메모리 낭비) 제거
- 어떻게? → 숨은 링크를 타고 올라가며 검색(체인)
- 무엇으로? → `__proto__`(자식의 문)·`prototype`(부모 보따리)·`constructor`(이름표)

=&gt; 이 뼈대만 잡으면 교체·`instanceof`·정적 멤버·열거는 전부 "이 구조를 다루거나 확인하는 도구"로 꿰어진다.

</details>

