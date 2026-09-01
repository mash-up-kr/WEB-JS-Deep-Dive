# 19장 프로토타입

자바스크립트는 명령형, 함수형, 프로토타입 기반 객체지향을 아우르는 멀티 패러다임 언어다. 클래스 기반이 아니라 **프로토타입 기반** 객체지향 언어이며, ES6의 class도 프로토타입 기반을 감춘 문법적 설탕에 가깝다(완전히 동일하진 않음).

원시 타입 값을 제외한 나머지 값(객체, 함수, 배열, 정규표현식 등)은 모두 객체다.

---

## 19.1 객체지향 프로그래밍

실체를 인식하고 그 속성 중 필요한 것만 간추려 표현하는 것을 **추상화(abstraction)** 라 한다.

객체는 **상태(state)** 를 나타내는 프로퍼티와, 상태를 조작하는 **동작(behavior)** 인 메서드를 하나의 논리적 단위로 묶은 자료구조다.

```js
const person = {
  name: 'Lee',
  sayHello() {
    console.log(`Hi! My name is ${this.name}`);
  }
};
```

---

## 19.2 상속과 프로토타입

**상속(inheritance)** 은 어떤 객체의 프로퍼티/메서드를 다른 객체가 그대로 사용하는 것. 중복 제거가 핵심 목적이다.

### 문제: 생성자 함수 내부에 메서드를 정의하면

```js
function Circle(radius) {
  this.radius = radius;
  this.getArea = function () {
    return Math.PI * this.radius ** 2;
  };
}

const c1 = new Circle(1);
const c2 = new Circle(2);

console.log(c1.getArea === c2.getArea); // false
```

인스턴스를 생성할 때마다 동일한 동작을 하는 함수 객체가 중복 생성된다. 메모리 낭비 + 퍼포먼스 악화.

### 해결: 프로토타입에 정의

```js
function Circle(radius) {
  this.radius = radius;
}

Circle.prototype.getArea = function () {
  return Math.PI * this.radius ** 2;
};

const c1 = new Circle(1);
const c2 = new Circle(2);

console.log(c1.getArea === c2.getArea); // true
```

`Circle` 생성자 함수가 생성한 모든 인스턴스는 `Circle.prototype`의 모든 프로퍼티와 메서드를 상속받는다. 인스턴스는 자신의 고유한 상태(radius)만 관리하면 된다.

---

## 19.3 프로토타입 객체

모든 객체는 `[[Prototype]]` 이라는 **내부 슬롯**을 가지며, 여기에 프로토타입의 참조가 저장된다. 값은 객체 생성 방식에 따라 결정된다.

`[[Prototype]]` 내부 슬롯에는 직접 접근할 수 없지만, `__proto__` 접근자 프로퍼티나 `Object.getPrototypeOf`로 간접 접근할 수 있다.

### 19.3.1 `__proto__` 접근자 프로퍼티

**① `__proto__`는 접근자 프로퍼티다.**

`[[Get]]` / `[[Set]]` 프로퍼티 어트리뷰트로 구성된 접근자 프로퍼티다. getter/setter 함수로 프로토타입을 취득하거나 할당한다.

```js
const obj = {};
const parent = { x: 1 };

obj.__proto__;          // getter 호출
obj.__proto__ = parent; // setter 호출

console.log(obj.x); // 1
```

**② `__proto__`는 상속을 통해 사용된다.**

`__proto__`는 객체가 직접 소유하는 프로퍼티가 아니라 `Object.prototype`의 프로퍼티다. 모든 객체는 상속을 통해 `Object.prototype.__proto__`를 사용할 수 있다.

```js
const person = { name: 'Lee' };
console.log(person.hasOwnProperty('__proto__')); // false
console.log(Object.getOwnPropertyDescriptor(Object.prototype, '__proto__'));
// { get: f, set: f, enumerable: false, configurable: true }
```

**③ `__proto__`를 통해 프로토타입에 접근하는 이유**

프로토타입 체인은 **단방향 링크드 리스트**로 구현되어야 한다. 상호 참조로 순환 참조하는 체인이 만들어지면 프로퍼티 검색 시 무한 루프에 빠진다. `__proto__` 접근자 프로퍼티는 setter에서 이를 체크해 에러를 던진다.

```js
const parent = {};
const child = {};

child.__proto__ = parent;
parent.__proto__ = child; // TypeError: Cyclic __proto__ value
```

**④ 코드 내에서 직접 사용하는 것은 권장하지 않는다.**

모든 객체가 `__proto__`를 사용할 수 있는 건 아니다. `Object.create(null)`로 생성한 객체는 `Object.prototype`을 상속받지 않는다.

```js
const obj = Object.create(null);
console.log(obj.__proto__); // undefined
console.log(Object.getPrototypeOf(obj)); // null
```

→ 프로토타입 취득은 `Object.getPrototypeOf`, 교체는 `Object.setPrototypeOf`를 사용한다.

### 19.3.2 함수 객체의 prototype 프로퍼티

**함수 객체만이** `prototype` 프로퍼티를 소유한다. 이 프로퍼티는 생성자 함수가 생성할 **인스턴스의 프로토타입**을 가리킨다.

```js
(function () {}).hasOwnProperty('prototype'); // true
({}).hasOwnProperty('prototype');            // false
```

단, non-constructor는 `prototype` 프로퍼티를 소유하지 않는다.

```js
const Person = name => { this.name = name; };
console.log(Person.hasOwnProperty('prototype')); // false

const obj = { foo() {} }; // ES6 메서드 축약 표현
console.log(obj.foo.hasOwnProperty('prototype')); // false
```

`__proto__`와 `prototype`은 **동일한 프로토타입을 가리키지만 사용 주체가 다르다.**

| 구분 | 소유 | 값 | 사용 주체 | 사용 목적 |
|---|---|---|---|---|
| `__proto__` | 모든 객체 | 프로토타입의 참조 | 모든 객체 | 자신의 프로토타입에 접근/교체 |
| `prototype` | constructor | 프로토타입의 참조 | 생성자 함수 | 생성할 인스턴스의 프로토타입 할당 |

```js
function Person(name) { this.name = name; }
const me = new Person('Lee');

console.log(Person.prototype === me.__proto__); // true
```

### 19.3.3 프로토타입의 constructor 프로퍼티와 생성자 함수

모든 프로토타입은 `constructor` 프로퍼티를 갖는다. 이는 `prototype` 프로퍼티로 자신을 참조하고 있는 **생성자 함수**를 가리킨다. 이 연결은 함수 객체가 생성될 때 이뤄진다.

```js
function Person(name) { this.name = name; }
const me = new Person('Lee');

console.log(me.constructor === Person); // true
```

`me` 객체에는 `constructor` 프로퍼티가 없지만, 프로토타입인 `Person.prototype`의 것을 상속받아 사용한다.

---

## 19.4 리터럴 표기법에 의해 생성된 객체의 생성자 함수와 프로토타입

리터럴 표기법으로 생성한 객체도 프로토타입이 존재하지만, 그 `constructor` 프로퍼티가 가리키는 생성자 함수가 반드시 그 객체를 생성한 것은 아니다.

```js
const obj = {};
console.log(obj.constructor === Object); // true
```

`obj`는 객체 리터럴로 생성됐지만 `Object` 생성자 함수와 연결되어 있다. 하지만 `Object` 생성자 함수 호출과 객체 리터럴의 평가는 **추상 연산 `OrdinaryObjectCreate`를 호출해 빈 객체를 생성하는 점은 동일하나, new.target 확인이나 프로퍼티를 추가하는 처리 등 세부 내용은 다르다.** 즉 객체 리터럴로 생성된 객체는 `Object` 생성자 함수가 생성한 객체가 아니다.

그럼에도 이렇게 연결해두는 이유는 **프로토타입과 생성자 함수는 단독으로 존재할 수 없고 언제나 쌍(pair)으로 존재**하기 때문이다. 리터럴 표기법으로 생성된 객체는 가상적인 생성자 함수를 갖는다고 이해하면 된다.

| 리터럴 표기법 | 생성자 함수 | 프로토타입 |
|---|---|---|
| 객체 리터럴 | Object | Object.prototype |
| 함수 리터럴 | Function | Function.prototype |
| 배열 리터럴 | Array | Array.prototype |
| 정규 표현식 리터럴 | RegExp | RegExp.prototype |

---

## 19.5 프로토타입의 생성 시점

프로토타입은 **생성자 함수가 생성되는 시점에 더불어 생성된다.** 프로토타입과 생성자 함수는 언제나 쌍으로 존재하기 때문이다.

### 19.5.1 사용자 정의 생성자 함수와 프로토타입 생성 시점

constructor는 함수 정의가 평가되어 함수 객체를 생성하는 시점에 프로토타입도 더불어 생성된다.

```js
console.log(Person.prototype); // {constructor: f}

function Person(name) {
  this.name = name;
}
```

함수 선언문은 런타임 이전에 먼저 평가되어 함수 객체가 되므로, 프로토타입도 이때 함께 생성된다.

생성된 프로토타입은 오직 `constructor` 프로퍼티만 갖는 객체이며, 이 프로토타입의 프로토타입은 `Object.prototype`이다.

> 화살표 함수, ES6 메서드 축약 표현 같은 non-constructor는 프로토타입이 생성되지 않는다.

### 19.5.2 빌트인 생성자 함수와 프로토타입 생성 시점

`Object`, `String`, `Number`, `Function`, `Array`, `RegExp`, `Date`, `Promise` 등 빌트인 생성자 함수는 **전역 객체가 생성되는 시점**에 생성된다. 이때 프로토타입도 함께 생성되며, 생성된 프로토타입은 빌트인 생성자 함수의 `prototype` 프로퍼티에 바인딩된다.

> 전역 객체: 브라우저에서는 `window`, Node.js에서는 `global` (ES11부터 `globalThis`로 통일).

정리하면, 객체가 생성되기 이전에 생성자 함수와 프로토타입은 이미 객체화되어 존재한다. 이후 객체를 생성하면 프로토타입은 생성된 객체의 `[[Prototype]]` 내부 슬롯에 할당된다.

---

## 19.6 객체 생성 방식과 프로토타입의 결정

객체 생성 방식은 다양하지만, 세부적인 차이는 있어도 모두 추상 연산 `OrdinaryObjectCreate`에 의해 생성된다는 공통점이 있다.

`OrdinaryObjectCreate`는 생성할 객체의 프로토타입을 인수로 전달받고, 빈 객체를 생성한 뒤 인수로 전달된 프로토타입을 `[[Prototype]]` 내부 슬롯에 할당한다. 즉 **프로토타입은 `OrdinaryObjectCreate`에 전달되는 인수에 의해 결정**되고, 이 인수는 객체가 생성되는 시점에 객체 생성 방식에 의해 결정된다.

### 19.6.1 객체 리터럴에 의해 생성된 객체의 프로토타입

`OrdinaryObjectCreate`에 `Object.prototype`이 전달된다.

```js
const obj = { x: 1 };

console.log(obj.constructor === Object); // true
console.log(obj.hasOwnProperty('x'));    // true
```

### 19.6.2 Object 생성자 함수에 의해 생성된 객체의 프로토타입

객체 리터럴과 동일하게 `Object.prototype`이 전달된다. 차이는 프로퍼티 추가 방식이다. 객체 리터럴은 리터럴 내부에 프로퍼티를 추가하지만, `Object` 생성자 함수 방식은 빈 객체를 생성한 후 프로퍼티를 추가해야 한다.

```js
const obj = new Object();
obj.x = 1;
```

### 19.6.3 생성자 함수에 의해 생성된 객체의 프로토타입

`OrdinaryObjectCreate`에 **생성자 함수의 `prototype` 프로퍼티에 바인딩되어 있는 객체**가 전달된다.

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

`Object.prototype`과 달리 사용자 정의 생성자 함수의 프로토타입에는 프로퍼티가 `constructor` 하나뿐이지만, 프로퍼티를 추가/삭제할 수 있고 이는 즉각 프로토타입 체인에 반영된다.

---

## 19.7 프로토타입 체인

객체의 프로퍼티에 접근할 때 해당 객체에 프로퍼티가 없으면, `[[Prototype]]` 내부 슬롯의 참조를 따라 부모 역할을 하는 프로토타입의 프로퍼티를 순차적으로 검색한다. 이를 **프로토타입 체인**이라 하며, 자바스크립트가 객체지향 프로그래밍의 상속을 구현하는 메커니즘이다.

```js
function Person(name) { this.name = name; }
Person.prototype.sayHello = function () { /* ... */ };

const me = new Person('Lee');

console.log(me.hasOwnProperty('name')); // true
```

`me` → `Person.prototype` → `Object.prototype` 순으로 검색한다. `hasOwnProperty`는 `Object.prototype`의 메서드이며, 이때 `this`에는 호출한 객체 `me`가 바인딩된다.

**`Object.prototype`은 프로토타입 체인의 종점(end of prototype chain)** 이다. 이 객체의 `[[Prototype]]` 내부 슬롯 값은 `null`이다. 종점에서도 프로퍼티를 검색할 수 없으면 `undefined`를 반환한다. **에러가 발생하지 않는다는 점에 주의.**

```js
console.log(me.foo); // undefined
```

### 프로토타입 체인 vs 스코프 체인

- **스코프 체인**: 식별자 검색을 위한 메커니즘
- **프로토타입 체인**: 프로퍼티 검색을 위한 메커니즘

```js
me.hasOwnProperty('name');
```

먼저 스코프 체인에서 식별자 `me`를 검색하고(전역에서 선언됐으므로 전역 스코프에서 검색), `me` 객체의 프로토타입 체인에서 `hasOwnProperty` 메서드를 검색한다. 두 체인은 서로 협력하여 식별자와 프로퍼티를 검색한다.

---

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
}());

const me = new Person('Lee');

// 인스턴스 메서드 추가
me.sayHello = function () {
  console.log(`Hey! My name is ${this.name}`);
};

me.sayHello(); // Hey! My name is Lee
```

프로토타입 메서드를 인스턴스 메서드가 **오버라이딩**했고, 프로토타입 메서드는 가려진다. 이처럼 상속 관계에 의해 프로퍼티가 가려지는 현상을 **프로퍼티 섀도잉(property shadowing)** 이라 한다.

> **오버라이딩**: 상위 클래스가 가진 메서드를 하위 클래스가 재정의하여 사용하는 방식
> **오버로딩**: 함수 이름은 동일하나 매개변수의 타입/개수가 다른 메서드를 구현하고 매개변수에 의해 메서드를 구별하여 호출하는 방식. 자바스크립트는 지원하지 않지만 `arguments` 객체로 흉내 낼 수 있다.

프로퍼티 삭제도 마찬가지다.

```js
delete me.sayHello; // 인스턴스 메서드 삭제
me.sayHello();      // Hi! My name is Lee (프로토타입 메서드 호출)

delete me.sayHello; // 프로토타입 메서드는 삭제되지 않음
me.sayHello();      // Hi! My name is Lee
```

**하위 객체를 통해 프로토타입의 프로퍼티를 변경/삭제하는 것은 불가능하다.** 즉 하위 객체를 통한 프로토타입 접근은 get은 허용되나 set은 허용되지 않는다. 프로토타입 프로퍼티를 변경/삭제하려면 프로토타입에 직접 접근해야 한다.

```js
Person.prototype.sayHello = function () { /* ... */ };
delete Person.prototype.sayHello;
```

---

## 19.9 프로토타입의 교체

프로토타입은 임의의 다른 객체로 변경할 수 있다. 부모 객체인 프로토타입을 동적으로 변경할 수 있다는 뜻이며, 이를 이용해 객체 간 상속 관계를 동적으로 변경할 수 있다.

### 19.9.1 생성자 함수에 의한 프로토타입의 교체

```js
const Person = (function () {
  function Person(name) {
    this.name = name;
  }

  // 프로토타입 교체
  Person.prototype = {
    sayHello() {
      console.log(`Hi! My name is ${this.name}`);
    }
  };

  return Person;
}());

const me = new Person('Lee');

console.log(me.constructor === Person); // false
console.log(me.constructor === Object); // true
```

객체 리터럴에는 `constructor` 프로퍼티가 없다. 따라서 `me`의 `constructor`는 프로토타입 체인을 따라 `Object.prototype`의 `constructor`를 찾아낸다. 즉 **프로토타입을 교체하면 constructor 프로퍼티와 생성자 함수 간의 연결이 파괴된다.**

되살리려면 `constructor` 프로퍼티를 명시적으로 추가한다.

```js
Person.prototype = {
  constructor: Person,
  sayHello() { /* ... */ }
};
```

### 19.9.2 인스턴스에 의한 프로토타입의 교체

인스턴스의 `__proto__` 접근자 프로퍼티 또는 `Object.setPrototypeOf`로도 교체할 수 있다.

```js
function Person(name) { this.name = name; }

const me = new Person('Lee');

const parent = {
  sayHello() {
    console.log(`Hi! My name is ${this.name}`);
  }
};

Object.setPrototypeOf(me, parent);
// me.__proto__ = parent; 와 동일하게 동작

me.sayHello(); // Hi! My name is Lee

console.log(me.constructor === Person); // false
console.log(me.constructor === Object); // true
```

두 방식의 차이는 **생성자 함수의 `prototype` 프로퍼티가 교체된 프로토타입을 가리키는가**이다.

| | constructor 연결 | 생성자 함수의 prototype 프로퍼티 |
|---|---|---|
| 생성자 함수에 의한 교체 | 파괴됨 | 교체된 프로토타입을 가리킴 |
| 인스턴스에 의한 교체 | 파괴됨 | 이전 프로토타입을 그대로 가리킴 |

```js
console.log(Person.prototype === Object.getPrototypeOf(me)); // false
console.log(parent.constructor === Person); // false
```

프로토타입 교체를 통해 상속 관계를 동적으로 변경하는 것은 번거롭다. **직접 상속이나 ES6의 class를 사용하는 편이 낫다.**

---

## 19.10 instanceof 연산자

```
객체 instanceof 생성자함수
```

우변의 생성자 함수의 `prototype`에 바인딩된 객체가 좌변 객체의 **프로토타입 체인 상에 존재하면** `true`, 아니면 `false`.

```js
function Person(name) { this.name = name; }
const me = new Person('Lee');

const parent = {};
Object.setPrototypeOf(me, parent);

console.log(Person.prototype === parent); // false
console.log(parent.constructor === Person); // false

console.log(me instanceof Person); // false
console.log(me instanceof Object); // true
```

`Person.prototype`이 `me`의 프로토타입 체인에 없으므로 `false`. 여기서 `Person.prototype`을 `parent`로 바인딩하면 다시 `true`가 된다.

```js
Person.prototype = parent;
console.log(me instanceof Person); // true
```

**instanceof는 `constructor` 프로퍼티와 무관하다.** 생성자 함수의 `prototype`에 바인딩된 객체가 프로토타입 체인 상에 존재하는지만 확인한다.

instanceof를 함수로 표현하면 대략 이렇다.

```js
function isInstanceof(instance, constructor) {
  const prototype = Object.getPrototypeOf(instance);

  if (prototype === null) return false;

  return prototype === constructor.prototype || isInstanceof(prototype, constructor);
}
```

---

## 19.11 직접 상속

### 19.11.1 Object.create에 의한 직접 상속

`Object.create`는 명시적으로 프로토타입을 지정하여 새로운 객체를 생성한다. 다른 객체 생성 방식과 마찬가지로 `OrdinaryObjectCreate`를 호출한다.

```js
Object.create(prototype[, propertiesObject])
```

- 첫 번째 매개변수: 생성할 객체의 프로토타입으로 지정할 객체
- 두 번째 매개변수(옵션): 프로퍼티 키와 프로퍼티 디스크립터 객체로 이뤄진 객체

```js
// 프로토타입이 null인 객체 (체인의 종점)
let obj = Object.create(null);
console.log(Object.getPrototypeOf(obj) === null); // true

// obj = {}; 와 동일
obj = Object.create(Object.prototype);

// obj = { x: 1 }; 과 동일
obj = Object.create(Object.prototype, {
  x: { value: 1, writable: true, enumerable: true, configurable: true }
});

// 임의 객체를 프로토타입으로 지정
const myProto = { x: 10 };
obj = Object.create(myProto);
console.log(obj.x); // 10
```

**장점**

- new 연산자 없이도 객체를 생성할 수 있다
- 프로토타입을 지정하면서 객체를 생성할 수 있다
- 객체 리터럴에 의해 생성된 객체도 상속받을 수 있다

**주의**: `Object.create(null)`로 생성한 객체는 `Object.prototype`의 빌트인 메서드를 사용할 수 없다. 그래서 `Object.prototype`의 빌트인 메서드는 **간접 호출**하는 것이 권장된다.

```js
const obj = Object.create(null);
obj.a = 1;

// console.log(obj.hasOwnProperty('a')); // TypeError

console.log(Object.prototype.hasOwnProperty.call(obj, 'a')); // true
```

> ESLint에서도 `Object.prototype`의 빌트인 메서드를 객체가 직접 호출하는 것을 권장하지 않는다.

### 19.11.2 객체 리터럴 내부에서 `__proto__`에 의한 직접 상속

ES6에서는 객체 리터럴 내부에서 `__proto__` 접근자 프로퍼티를 사용하여 직접 상속을 구현할 수 있다.

```js
const myProto = { x: 10 };

const obj = {
  y: 20,
  __proto__: myProto
};

console.log(obj.x, obj.y);                       // 10 20
console.log(Object.getPrototypeOf(obj) === myProto); // true
```

---

## 19.12 정적 프로퍼티/메서드

**정적(static) 프로퍼티/메서드**는 생성자 함수로 인스턴스를 생성하지 않아도 참조/호출할 수 있는 프로퍼티/메서드다.

```js
function Person(name) { this.name = name; }

Person.prototype.sayHello = function () { /* ... */ };

// 정적 프로퍼티
Person.staticProp = 'static prop';

// 정적 메서드
Person.staticMethod = function () {
  console.log('staticMethod');
};

const me = new Person('Lee');

Person.staticMethod(); // staticMethod
me.staticMethod();     // TypeError: me.staticMethod is not a function
```

생성자 함수가 생성한 인스턴스는 자신의 프로토타입 체인에 속한 객체의 프로퍼티/메서드에 접근할 수 있지만, **정적 프로퍼티/메서드는 인스턴스의 프로토타입 체인에 속한 객체의 프로퍼티/메서드가 아니므로 인스턴스로 접근할 수 없다.**

`Object.create`는 정적 메서드, `Object.prototype.hasOwnProperty`는 프로토타입 메서드다.

```js
Object.create();                      // 정적 메서드
const obj = {}.hasOwnProperty('a');   // 프로토타입 메서드
```

**프로토타입 메서드가 `this`를 참조하지 않는다면 정적 메서드로 변경할 수 있다.** 정적 메서드는 인스턴스를 생성하지 않아도 호출할 수 있으므로 더 간편하다.

```js
function Foo() {}

// this를 참조하지 않는 프로토타입 메서드
Foo.prototype.x = function () { console.log('x'); };
const foo = new Foo();
foo.x(); // 인스턴스를 생성해야 호출 가능

// 정적 메서드로 변경
Foo.x = function () { console.log('x'); };
Foo.x(); // 인스턴스 생성 없이 호출
```

> 참고: `Object.prototype.hasOwnProperty` 같은 표기에서 `#`은 정적, `.prototype.`은 프로토타입 프로퍼티를 나타내는 관례가 있다. (`Object.prototype.isPrototypeOf` ↔ `Object#isPrototypeOf`)

---

## 19.13 프로퍼티 존재 확인

### 19.13.1 in 연산자

```js
key in object
```

객체 내에 특정 프로퍼티가 존재하는지 확인한다. **확인 대상 객체의 프로퍼티뿐 아니라 상속받은 모든 프로토타입의 프로퍼티를 확인한다는 점에 주의.**

```js
const person = { name: 'Lee', address: 'Seoul' };

console.log('name' in person);    // true
console.log('age' in person);     // false
console.log('toString' in person); // true (Object.prototype의 메서드)
```

ES6에서는 `Reflect.has`를 사용할 수도 있다. 동일하게 동작한다.

```js
console.log(Reflect.has(person, 'name')); // true
```

### 19.13.2 Object.prototype.hasOwnProperty 메서드

인수로 전달받은 프로퍼티 키가 **객체 고유의 프로퍼티 키인 경우에만** `true`를 반환한다. 상속받은 프로토타입의 프로퍼티 키인 경우 `false`.

```js
console.log(person.hasOwnProperty('name'));     // true
console.log(person.hasOwnProperty('age'));      // false
console.log(person.hasOwnProperty('toString')); // false
```

---

## 19.14 프로퍼티 열거

### 19.14.1 for...in 문

```js
for (변수선언문 in 객체) { ... }
```

객체의 프로퍼티 개수만큼 순회하며 변수선언문에서 선언한 변수에 프로퍼티 키를 할당한다.

**상속받은 프로토타입의 프로퍼티까지 열거한다.** 단, `[[Enumerable]]`이 `false`인 프로퍼티는 제외된다. `Object.prototype.toString` 같은 빌트인 메서드는 `[[Enumerable]]`이 `false`라 열거되지 않는다.

```js
const person = { name: 'Lee', address: 'Seoul', __proto__: { age: 20 } };

for (const key in person) {
  console.log(key + ': ' + person[key]);
}
// name: Lee
// address: Seoul
// age: 20
```

- 프로퍼티 키가 **심벌인 프로퍼티는 열거하지 않는다.**
- 열거 시 프로퍼티 **순서를 보장하지 않는다.** (대부분의 모던 브라우저는 순서를 보장하고 숫자 키에는 오름차순 정렬을 하기는 한다)

객체 자신의 프로퍼티만 열거하려면 `hasOwnProperty`로 필터링한다.

```js
for (const key in person) {
  if (!person.hasOwnProperty(key)) continue;
  console.log(key + ': ' + person[key]);
}
```

배열에는 `for...in` 대신 일반 `for` 문, `for...of`, `Array.prototype.forEach`를 사용하는 것이 좋다. 배열도 객체이므로 상속받은 프로퍼티나 추가한 프로퍼티까지 열거될 수 있다.

```js
const arr = [1, 2, 3];
arr.x = 10;

for (const i in arr) console.log(arr[i]); // 1 2 3 10

arr.forEach(v => console.log(v));   // 1 2 3
for (const value of arr) console.log(value); // 1 2 3
```

### 19.14.2 Object.keys/values/entries 메서드

객체 **자신의 고유 프로퍼티만** 열거하려면 이 메서드들을 사용하는 것이 권장된다.

```js
const person = { name: 'Lee', address: 'Seoul', __proto__: { age: 20 } };

console.log(Object.keys(person));    // ['name', 'address']
console.log(Object.values(person));  // ['Lee', 'Seoul']  (ES8)
console.log(Object.entries(person)); // [['name','Lee'], ['address','Seoul']]  (ES8)

Object.entries(person).forEach(([key, value]) => console.log(key, value));
```

---

## 정리

- 프로토타입은 `[[Prototype]]` 내부 슬롯에 저장되고, `__proto__`(접근자 프로퍼티) 또는 `Object.getPrototypeOf`로 접근한다.
- `prototype` 프로퍼티는 함수 객체(constructor)만 갖고, 생성할 인스턴스의 프로토타입을 가리킨다.
- 프로토타입과 생성자 함수는 항상 쌍으로 존재하며, 생성자 함수가 평가되는 시점에 함께 생성된다.
- 프로퍼티 검색은 프로토타입 체인을 따라 이뤄지고, 종점은 `Object.prototype`(그 프로토타입은 `null`)이다.
- 하위 객체를 통해 프로토타입 프로퍼티를 get은 할 수 있지만 set/delete는 할 수 없다.
- 프로토타입 교체는 `constructor` 연결을 파괴하므로, 직접 상속이나 class를 쓰는 게 낫다.
- `instanceof`는 `constructor`가 아니라 프로토타입 체인을 본다.