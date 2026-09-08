# 모던 자바스크립트 Deep Dive 19장 정리

19장은 "객체가 코드를 어떻게 공유하는가"를 다룬다. 17장에서 생성자 함수로 객체를 찍어내는 방법을 봤다면, 여기서는 찍어낸 객체들이 메서드를 중복 없이 나눠 쓰는 구조를 본다. 상속·프로토타입 체인·섀도잉·교체까지 전부 하나의 그림에서 파생된다.

## 19.1 객체지향 프로그래밍

객체지향은 실체를 속성의 집합으로 표현하려는 발상에서 출발한다. 실체가 가진 무수한 속성 중 프로그램에 필요한 것만 골라내는 것이 **추상화**이고, 골라낸 속성(상태)과 그 상태를 조작하는 동작을 하나로 묶은 것이 **객체**다.

```js
const person = {
  // 상태(state)
  name: 'Lee',
  address: 'Seoul',

  // 동작(behavior)
  sayHello() {
    console.log(`Hi! My name is ${this.name}`);
  },
};
```

⇒ 객체 = 상태 + 동작. 이 관점에서 프로토타입은 "여러 객체가 **동작만** 공유하는 방법"이다. 상태는 각자 갖고 동작은 하나만 둔다.

## 19.2 상속과 프로토타입

상속의 목적은 명확하다. 중복 제거, 즉 메모리 절약이다.

### 메서드가 인스턴스마다 중복 생성된다

```js
function Circle(radius) {
  this.radius = radius;
  this.getArea = function () {
    return Math.PI * this.radius ** 2;
  };
}

const circle1 = new Circle(1);
const circle2 = new Circle(2);

console.log(circle1.getArea === circle2.getArea); // false
```

생성자 함수는 호출될 때마다 `this.getArea = function () {...}`를 실행한다. 인스턴스를 100개 만들면 내용이 동일한 함수 객체가 100개 생긴다. 상태(`radius`)는 인스턴스마다 달라야 하니 중복이 맞지만, 동작까지 중복될 이유는 없다.

### 프로토타입에 메서드를 하나만 둔다

```js
function Circle(radius) {
  this.radius = radius; // 상태는 인스턴스가 각자 소유
}

// 동작은 프로토타입에 하나만
Circle.prototype.getArea = function () {
  return Math.PI * this.radius ** 2;
};

const circle1 = new Circle(1);
const circle2 = new Circle(2);

console.log(circle1.getArea === circle2.getArea); // true
console.log(circle1.getArea()); // 3.141592653589793
console.log(circle2.getArea()); // 12.566370614359172
```

`Circle`이 생성한 모든 인스턴스는 `Circle.prototype`을 자신의 프로토타입(부모 객체)으로 상속받는다. `circle1`은 `getArea`를 소유하지 않지만 자신의 것처럼 사용한다.

⇒ 함수는 하나인데 호출한 인스턴스에 따라 `this`가 달라진다. **코드는 공유하고 데이터는 분리**하는 것이 성립하는 이유가 `this`의 동적 바인딩이다.

## 19.3 프로토타입 객체

모든 객체는 `[[Prototype]]`이라는 내부 슬롯을 가진다. 값은 프로토타입 객체의 참조이며, 프로토타입이 없는 객체는 `null`이다. 내부 슬롯이라 직접 접근할 수 없고 `__proto__` 접근자 프로퍼티로 간접 접근한다.

### `__proto__` 접근자 프로퍼티

```js
const obj = {};
const parent = { x: 1 };

console.log(obj.__proto__); // getter 호출 — Object.prototype

obj.__proto__ = parent;     // setter 호출 — 프로토타입 교체

console.log(obj.x); // 1
```

**1. 접근자 프로퍼티다.** 값을 저장한 데이터 프로퍼티가 아니라 `[[Get]]` / `[[Set]]`에 대응하는 함수 쌍이다.

```js
console.log(Object.getOwnPropertyDescriptor(Object.prototype, '__proto__'));
// {get: ƒ, set: ƒ, enumerable: false, configurable: true}
```

**2. 객체가 직접 소유한 프로퍼티가 아니다.** `Object.prototype`의 프로퍼티를 상속받아 쓰는 것이다.

```js
const person = { name: 'Lee' };

console.log(person.hasOwnProperty('__proto__')); // false
console.log('__proto__' in person);              // true
```

**3. 순환 프로토타입 체인을 막는다.**

```js
const parent = {};
const child = {};

child.__proto__ = parent;
parent.__proto__ = child; // TypeError: Cyclic __proto__ value
```

프로토타입 체인은 단방향 링크드 리스트여야 한다. 순환이 생기면 프로퍼티 검색이 무한 루프에 빠지므로 setter가 이를 검사한다. `__proto__`가 접근자 프로퍼티인 이유가 이것이다. 단순 데이터 프로퍼티였다면 검사를 넣을 수 없다.

**4. 코드에서 직접 쓰는 것은 권장되지 않는다.** 모든 객체가 `__proto__`를 쓸 수 있는 것이 아니기 때문이다.

```js
const obj = Object.create(null); // 체인의 종점에 위치하는 객체

console.log(obj.__proto__);              // undefined — 접근자를 상속받지 못했다
console.log(Object.getPrototypeOf(obj)); // null — 이건 동작한다
```

| 목적 | 권장 | 비권장 |
| --- | --- | --- |
| 프로토타입 취득 | `Object.getPrototypeOf(obj)` | `obj.__proto__` |
| 프로토타입 교체 | `Object.setPrototypeOf(obj, parent)` | `obj.__proto__ = parent` |

`__proto__`는 원래 표준이 아니었지만 브라우저 호환성 때문에 ES6에서 부록(Annex B)으로 편입됐다. 레거시 호환용이라는 뜻이므로 새 코드에서는 쓰지 않는다.

### 함수 객체의 `prototype` 프로퍼티

**함수 객체만** `prototype` 프로퍼티를 소유한다. 생성자 함수가 생성할 인스턴스의 프로토타입을 가리킨다.

```js
console.log((function () {}).hasOwnProperty('prototype')); // true
console.log({}.hasOwnProperty('prototype'));               // false
```

non-constructor는 `prototype`을 갖지 않는다. `new`와 함께 호출할 수 없어 인스턴스를 만들지 않으므로 애초에 생성되지 않는다.

```js
const Arrow = () => {};
console.log(Arrow.hasOwnProperty('prototype')); // false
console.log(Arrow.prototype);                   // undefined

const obj = { foo() {} }; // ES6 메서드 축약 표현
console.log(obj.foo.hasOwnProperty('prototype')); // false
```

`__proto__`와 `prototype`은 같은 객체를 가리키지만 소유 주체와 사용 목적이 다르다.

| 구분 | 소유 | 값 | 사용 주체 | 목적 |
| --- | --- | --- | --- | --- |
| `__proto__` | 모든 객체 | 프로토타입의 참조 | 모든 객체 | 객체가 **자신의** 프로토타입에 접근·교체 |
| `prototype` | constructor | 프로토타입의 참조 | 생성자 함수 | 생성자가 **생성할 인스턴스의** 프로토타입 할당 |

```js
function Person(name) {
  this.name = name;
}

const me = new Person('Lee');

console.log(Person.prototype === me.__proto__); // true
```

### 프로토타입의 `constructor` 프로퍼티

모든 프로토타입은 자신을 참조하는 생성자 함수를 가리키는 `constructor`를 갖는다. 함수 객체가 생성될 때 자동으로 연결된다.

```js
function Person(name) {
  this.name = name;
}

const me = new Person('Lee');

console.log(me.constructor === Person); // true
```

`me`에는 `constructor`가 없지만 `Person.prototype`에서 상속받아 쓴다. 즉 인스턴스에서 자신을 만든 생성자 함수를 역추적할 수 있다. 이 연결은 19.9에서 끊어질 수 있다.

```text
   Person (생성자 함수)                    me (인스턴스)
   ┌──────────────┐                      ┌──────────────┐
   │  prototype ──┼──┐              ┌────┼── __proto__  │
   └──────────────┘  │              │    │  name: 'Lee' │
                     ▼              │    └──────────────┘
              Person.prototype ─────┘
              ┌──────────────┐
              │ constructor ─┼──▶ Person
              └──────────────┘
```

## 19.4 리터럴로 생성된 객체의 생성자 함수와 프로토타입

```js
const obj = {};
console.log(obj.constructor === Object); // true

function foo() {}
console.log(foo.constructor === Function); // true

const arr = [];
console.log(arr.constructor === Array); // true

const regexp = /is/gi;
console.log(regexp.constructor === RegExp); // true
```

`obj`는 `Object` 생성자 함수로 만든 것이 아닌데도 `constructor`가 `Object`를 가리킨다. `Object` 생성자에 인수를 전달하지 않으면 내부적으로 추상 연산 `OrdinaryObjectCreate`를 호출해 `Object.prototype`을 프로토타입으로 갖는 빈 객체를 만드는데, 객체 리터럴이 평가될 때도 동일하게 `OrdinaryObjectCreate`가 호출되기 때문이다.

⇒ 생성 과정에는 차이가 있지만(`new.target` 확인 여부, 프로퍼티 추가 방식) 결과물의 프로토타입은 같다. 프로토타입은 생성자 함수와 **언제나 쌍으로 존재**하므로, 리터럴로 만든 객체도 가상적인 생성자 함수를 갖는다고 보면 된다.

| 리터럴 표기법 | 생성자 함수 | 프로토타입 |
| --- | --- | --- |
| `{}` | `Object` | `Object.prototype` |
| `function () {}` | `Function` | `Function.prototype` |
| `[]` | `Array` | `Array.prototype` |
| `/x/` | `RegExp` | `RegExp.prototype` |

## 19.5 프로토타입의 생성 시점

프로토타입은 **생성자 함수가 생성되는 시점에 함께 생성된다.** 둘은 언제나 쌍으로 존재하기 때문이다.

### 사용자 정의 생성자 함수

```js
// 함수 선언문은 런타임 이전에 평가된다 → 이 시점에 프로토타입도 이미 존재
console.log(Person.prototype); // {constructor: ƒ}

function Person(name) {
  this.name = name;
}
```

생성된 프로토타입은 `constructor`만 갖는 객체이고, 그 프로토타입은 `Object.prototype`이다.

```js
console.log(Object.getPrototypeOf(Person.prototype) === Object.prototype); // true
```

non-constructor는 프로토타입이 생성되지 않는다.

### 빌트인 생성자 함수

`Object`, `String`, `Number`, `Function`, `Array`, `RegExp`, `Date`, `Promise` 등은 **전역 객체가 생성되는 시점에** 생성되고, 이때 프로토타입도 함께 만들어져 `prototype` 프로퍼티에 바인딩된다.

전역 객체는 코드 실행 이전에 엔진이 가장 먼저 생성하는 특수 객체다. 브라우저는 `window`, Node.js는 `global`이며 표준은 `globalThis`다.

```text
전역 객체 생성 → 빌트인 생성자 함수 + 프로토타입 생성
      ↓
사용자 정의 생성자 함수 평가 → 해당 프로토타입 생성
      ↓
new 호출 → 인스턴스 생성 → [[Prototype]]에 프로토타입 할당
```

⇒ 객체가 생성되기 이전에 프로토타입은 이미 객체화되어 존재한다. 그래서 객체는 생성 즉시 상속받을 수 있다.

## 19.6 객체 생성 방식과 프로토타입의 결정

객체 생성 방식은 여러 가지지만 모두 `OrdinaryObjectCreate`를 호출한다. 이 연산은 인수로 전달받은 객체를 새 객체의 `[[Prototype]]`에 할당한다. 즉 프로토타입은 전달되는 인수에 의해 결정되고, 그 인수는 생성 방식에 따라 결정된다.

**객체 리터럴 / `Object` 생성자 함수** — 프로토타입은 `Object.prototype`. 차이는 프로퍼티 추가 방식뿐이다.

```js
const obj1 = { x: 1 };
const obj2 = new Object();
obj2.x = 1;

console.log(obj1.constructor === Object, obj2.constructor === Object); // true true
console.log(obj1.hasOwnProperty('x')); // true — Object.prototype에서 상속
```

**생성자 함수** — 프로토타입은 `생성자함수.prototype`.

```js
function Person(name) {
  this.name = name;
}

Person.prototype.sayHello = function () {
  console.log(`Hi! My name is ${this.name}`);
};

const me = new Person('Lee');
const you = new Person('Kim');

me.sayHello();  // Hi! My name is Lee
you.sayHello(); // Hi! My name is Kim
```

객체 리터럴로 만든 객체는 다양한 빌트인 메서드를 상속받지만, 생성자 함수로 만든 객체의 프로토타입은 `constructor`만 있는 단출한 객체다. 여기에 프로퍼티를 추가하면 모든 인스턴스가 즉시 상속받는다.

⇒ 프로토타입은 동적으로 변경 가능한 살아 있는 객체다.

## 19.7 프로토타입 체인

```js
function Person(name) {
  this.name = name;
}

Person.prototype.sayHello = function () {
  console.log(`Hi! My name is ${this.name}`);
};

const me = new Person('Lee');

console.log(me.hasOwnProperty('name')); // true
```

`me`도 `Person.prototype`도 `hasOwnProperty`를 소유하지 않는데 호출된다.

```js
console.log(Object.getPrototypeOf(me) === Person.prototype);               // true
console.log(Object.getPrototypeOf(Person.prototype) === Object.prototype); // true
console.log(Object.getPrototypeOf(Object.prototype));                      // null
```

프로퍼티에 접근할 때 엔진은 다음 순서로 검색한다.

1. 해당 객체에 프로퍼티가 있는지 검색한다.
2. 없으면 `[[Prototype]]`의 참조를 따라 부모 프로토타입으로 이동해 검색한다.
3. 순차적으로 반복한다.
4. 종점까지 갔는데도 없으면 `undefined`를 반환한다. **에러가 아니다.**

```text
        me                Person.prototype        Object.prototype        null
   ┌──────────┐            ┌─────────────┐      ┌─────────────────┐
   │ name     │  [[Proto]] │ constructor │[[Proto]] hasOwnProperty │[[Proto]]
   │          │ ─────────▶ │ sayHello    │─────▶│ toString        │──────▶ (종점)
   └──────────┘            └─────────────┘      │ isPrototypeOf   │
                                                 └─────────────────┘
```

`Object.prototype`이 프로토타입 체인의 **종점**이며, 그 `[[Prototype]]`은 `null`이다. 그래서 모든 객체가 `toString`, `hasOwnProperty`, `valueOf`를 상속받는다.

```js
console.log(me.foo); // undefined — 에러가 아니다
```

### 프로토타입 체인과 스코프 체인

| | 검색 대상 | 방향 |
| --- | --- | --- |
| 스코프 체인 | 식별자(변수·함수 이름) | 중첩된 스코프를 따라 위로 |
| 프로토타입 체인 | 프로퍼티 | `[[Prototype]]`을 따라 위로 |

```js
me.hasOwnProperty('name');
```

이 한 줄에서 두 체인이 협력한다. 먼저 스코프 체인에서 식별자 `me`를 검색하고, 그다음 `me`의 프로토타입 체인에서 `hasOwnProperty`를 검색한다.

⇒ 두 체인은 별개로 도는 것이 아니라 식별자와 프로퍼티 검색에 함께 쓰인다.

## 19.8 오버라이딩과 프로퍼티 섀도잉

```js
const Person = (function () {
  function Person(name) {
    this.name = name;
  }

  Person.prototype.sayHello = function () {
    console.log(`Hi! My name is ${this.name}`);
  };

  return Person;
})();

const me = new Person('Lee');

// 인스턴스 메서드 추가
me.sayHello = function () {
  console.log(`Hey! My name is ${this.name}`);
};

me.sayHello(); // Hey! My name is Lee
```

프로퍼티를 추가할 때 체인 상에 동일한 이름이 있어도 프로토타입을 덮어쓰지 않고 인스턴스에 새로 추가된다. 그 결과 인스턴스 메서드가 프로토타입 메서드를 가린다.

- **오버라이딩**: 상위에서 정의한 메서드를 하위에서 재정의해 사용하는 방식
- **프로퍼티 섀도잉**: 상속 관계에 의해 프로퍼티가 가려지는 현상

오버로딩(매개변수 타입·개수가 다른 동일 이름의 메서드를 여러 개 두는 것)과는 다르며, 자바스크립트는 오버로딩을 문법적으로 지원하지 않는다.

```js
delete me.sayHello;
me.sayHello(); // Hi! My name is Lee — 프로토타입 메서드가 다시 보인다

delete me.sayHello;
me.sayHello(); // Hi! My name is Lee — 프로토타입 메서드는 지워지지 않는다
```

하위 객체를 통해 프로토타입의 프로퍼티를 변경·삭제하는 것은 불가능하다. 변경·삭제하려면 프로토타입에 직접 접근해야 한다.

```js
Person.prototype.sayHello = function () { /* 변경 */ };
delete Person.prototype.sayHello;
```

⇒ 프로토타입 체인은 **get 접근은 허용하지만 set 접근은 허용하지 않는다.** `obj.x = 1`은 언제나 `obj` 자신에게 프로퍼티를 만든다.

## 19.9 프로토타입의 교체

프로토타입은 임의의 다른 객체로 변경할 수 있다. 부모 객체를 동적으로 바꿀 수 있다는 뜻이다.

### 생성자 함수에 의한 교체

```js
const Person = (function () {
  function Person(name) {
    this.name = name;
  }

  Person.prototype = {
    sayHello() {
      console.log(`Hi! My name is ${this.name}`);
    },
  };

  return Person;
})();

const me = new Person('Lee');

console.log(me.constructor === Person); // false
console.log(me.constructor === Object); // true
```

교체한 객체 리터럴에는 `constructor`가 없다. 그래서 체인을 따라 올라가 `Object.prototype`의 `constructor`를 찾은 것이고, 생성자 함수와 프로토타입의 연결이 파괴됐다. 되살리려면 명시적으로 추가한다.

```js
Person.prototype = {
  constructor: Person, // 연결 복구
  sayHello() {
    console.log(`Hi! My name is ${this.name}`);
  },
};
```

### 인스턴스에 의한 교체

```js
function Person(name) {
  this.name = name;
}

const me = new Person('Lee');

const parent = {
  sayHello() {
    console.log(`Hi! My name is ${this.name}`);
  },
};

Object.setPrototypeOf(me, parent); // me.__proto__ = parent 와 동일

me.sayHello(); // Hi! My name is Lee

console.log(me.constructor === Person);                    // false
console.log(Person.prototype === Object.getPrototypeOf(me)); // false
```

| | `생성자함수.prototype` | 인스턴스의 `constructor` 연결 |
| --- | --- | --- |
| 생성자 함수로 교체 | 새 프로토타입을 가리킴 | 끊어짐 |
| 인스턴스로 교체 | 여전히 원래 것을 가리킴 | 끊어짐 |

인스턴스로 교체하면 `Person.prototype`과 실제 인스턴스의 프로토타입이 서로 다른 객체를 가리키게 되어 관계가 더 꼬인다.

⇒ 프로토타입 교체는 쓰지 않는다. 관계가 파괴되어 추적이 어렵고 `Object.setPrototypeOf`는 엔진 최적화를 무력화해 성능도 나쁘다. 상속 관계를 인위적으로 설정하려면 `Object.create`나 ES6 `class`의 `extends`(25장)를 쓴다.

## 확인할 내용

- `__proto__`와 `prototype`의 소유 주체 구분
- 프로토타입 체인에서 get은 되지만 set은 되지 않는 이유
- 프로토타입 교체가 `constructor` 연결을 파괴하는 지점
- 리터럴로 만든 객체에도 생성자 함수와 프로토타입 쌍이 존재하는 이유

## 19장 핵심 흐름

```text
프로토타입 = 중복 제거(메모리) + 상속
  ├─ 상태는 인스턴스가, 동작은 프로토타입이 소유 (this 동적 바인딩으로 성립)
  │
  ├─ 연결 구조
  │    ├─ [[Prototype]] 내부 슬롯  → __proto__ 접근자로 간접 접근 (권장: getPrototypeOf)
  │    ├─ prototype              → constructor만 소유, 생성할 인스턴스의 프로토타입
  │    └─ constructor            → 프로토타입 → 생성자 함수 역참조
  │
  ├─ 생성 시점: 생성자 함수 생성 시 프로토타입도 함께 생성 (인스턴스보다 먼저 존재)
  │
  ├─ 프로토타입 체인 = 프로퍼티 검색 메커니즘
  │    ├─ 종점은 Object.prototype ([[Prototype]] === null)
  │    ├─ 끝까지 없으면 undefined (에러 아님)
  │    ├─ get O / set X → 하위 객체 할당은 섀도잉만 발생
  │    └─ 스코프 체인(식별자)과 협력해 동작
  │
  └─ 교체: 생성자 함수 교체 → prototype은 갱신 / constructor 연결 파괴
           인스턴스 교체     → prototype 갱신 안 됨 / constructor 연결 파괴
           둘 다 사용 금지, 상속은 class extends (25장)
```

`class`는 프로토타입 기반 상속을 감싼 문법에 가깝다. 다만 `new` 없이 호출 불가, 호이스팅 동작 차이, 메서드의 `[[Enumerable]]`이 `false`라는 점 등에서 완전히 동일하지는 않다. 22장 `this`, 23장 실행 컨텍스트, 25장 클래스로 이어진다.
