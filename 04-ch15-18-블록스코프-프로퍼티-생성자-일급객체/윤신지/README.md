![모던 자바스크립트 딥다이브](https://cdn.inflearn.com/public/courses/327974/cover/3b014384-8b3e-4f66-a4de-a94ffff11f58/Modern%20Javascript%20Deep%20Dive.png?w=736)

   [*모던 자바스크립트 Deep Dive](https://www.yes24.com/Product/Goods/92742567)을 토대로 공부한 것을 정리한 내용으로, 모든 인용문은 모던 자바스크립트 Deep Dive의 문구를 인용한 것입니다.*

# 15장. let, const 키워드와 블록 레벨 스코프

## var 키워드로 선언한 변수의 문제점

ES5까지 변수는 오직 `var`  키워드를 이용해서만 선언할 수 있었다. 이러면 무슨 문제가 있을까?



### 변수 중복 선언 허용

```javascript
var x = 1;
var y = 1;

var x = 100; // 초기화문이 있는 중복 선언
var y; // 초기화문이 없는 중복 선언

console.log(x); // 100 : 중복 선언으로 변수 값 변경됨
console.log(y); // 1 : 변수 값 변경되지 않음
```

위 예제에서 `var`로 선언한 `x` 변수와 `y` 변수 모두 중복 선언되었다. 중복 선언된 변수는 **초기화문**의 유무에 따라 다르게 동작한다. 



- 초기화문이 있는 변수 선언문 : var 키워드가 없는 것처럼 동작하여 새로운 값이 할당됨
- 초기화문이 없는 변수 선언문 : 변수 선언문이 무시됨

=&gt; 동일한 이름의 변수가 이미 선언되어 있는 것을 모르고 변수를 중복 선언 + 값을 할당했다면 먼저 선언된 변수 값이 변경된느 부작용이 발생



### 함수 레벨 스코프

var 키워드로 선언한 변수는 오직 **함수**의 코드 블록만을 **지역 스코프**로 인정한다.   
=&gt; 함수 외부에서 아무리 코드 블록에 감싸더라도, var 키워드로 선언한 변수는 **전역 변수**가 된다. 

```js
var x = 1;

if(true) { 
  var x = 10; // 함수가 아닌 코드 블록에서 선언되어 전역 변수가 되어 버린 x
}

console.log(x); // 10

// =========================================================================

var i = 10;

for(var i = 0; i < 5; i++) {
  console.log(i); // 0 1 2 3 4
}

console.log(i); // 5 : 함수가 아닌 코드 블록에서 선언되어 전역 변수가 되어 버린 i
```

함수 레벨 스코프는 **전역변수를 남발할 가능성**을 높여 의도치 않은 중복 선언이 발생하기 쉽다. 



<details class="orca-details">
<summary>ES6 이전에는 블록 레벨 스코프를 어떻게 흉내 냈을까?</summary>

즉시 실행 함수(IIFE)를 사용했다. 함수는 유일하게 스코프를 만드는 문법이었기 때문에, 블록이 필요한 자리에 함수를 하나 만들어서 감싸는 방식이 관용구처럼 쓰였다.

```js
(function () {
  var temp = '외부에 노출되지 않는 변수';
})();
```

이 패턴이 확장된 것이 바로 **모듈 패턴**이고, jQuery 플러그인이나 UMD 번들 코드가 `(function (global) { ... })(this)` 형태로 시작하는 이유도 여기에 있다. 지금은 블록과 모듈이 그 역할을 대신하지만, 오래된 라이브러리 코드나 번들러가 만들어낸 결과물을 읽을 때 여전히 마주치게 되므로 눈에 익혀 둘 가치가 있다.

</details>

<details class="orca-details">
<summary>for문과 setTimeout을 함께 쓸 때 var는 왜 항상 마지막 값만 출력될까?</summary>

`var i`는 함수 레벨 스코프이므로 반복문이 몇 번을 돌든 **바인딩은 단 하나**다. 콜백들은 모두 그 하나의 `i`를 참조하는 클로저이고, 콜백이 실제로 실행되는 시점에는 이미 반복이 끝나 `i`가 최종값이 되어 있다.

```js
for (var i = 0; i < 3; i++) setTimeout(() => console.log(i)); // 3, 3, 3
for (let i = 0; i < 3; i++) setTimeout(() => console.log(i)); // 0, 1, 2
```

여기서 한 걸음 더 들어가면 좋은 질문은 **"그럼 let은 왜 되는가"** 다. 스펙상 `for (let ...)` 문은 반복마다 새로운 렉시컬 환경을 만들고 직전 반복의 값을 복사해 넣는다(`CreatePerIterationEnvironment`). 즉 `let`이 마법을 부리는 게 아니라, for문이 `let`을 만났을 때만 수행하는 별도의 절차가 스펙에 정의되어 있는 것이다. 이 사실을 알면 `for (const i = 0; i < 3; i++)`가 왜 증감식에서 에러가 나는지도 함께 설명된다.

</details>



### 변수 호이스팅

변수 호이스팅에 의해 var 키워드로 선언한 변수는 변수 선언문 이전에 참조할 수 있다.  
=&gt; 에러는 발생하지 않지만, 프로그램의 흐름상 맞지 않을 뿐더러 가독성을 떨어뜨리고 오류를 발생시킬 여지를 남긴다.

```js
console.log(foo); // undefined

foo = 123;
console.log(foo); // 123

var foo;
```

위 예제에서 `foo` 변수는 선언문보다 앞서 참조했음에도 에러가 발생하지 않고 undefined가 출력된다. 변수 선언문 이전에 변수를 참조하는 것은 변수 호이스팅에 의해 에러를 발생시키지는 않지만, 프로그램의 흐름상 맞지 않고 가독성을 떨어뜨린다.

이러한 var 키워드의 단점을 보완하기 위해 ES6에서 `let`과 `const` 키워드가 도입되었다.



## let 키워드

### 변수 중복 선언 금지

var 키워드는 이름이 동일한 변수를 중복 선언해도 아무런 에러가 발생하지 않았다. 하지만 **let 키워드로 이름이 같은 변수를 중복 선언하면 문법 에러(SyntaxError)가 발생**한다.

```js
var foo = 123;
var foo = 456; // 중복 선언 허용

let bar = 123;
let bar = 456; // SyntaxError: Identifier 'bar' has already been declared
```



<details class="orca-details">
<summary>let 중복 선언은 왜 ReferenceError가 아니라 SyntaxError일까?</summary>

중복 선언은 **코드를 실행해 보지 않아도 파싱 단계에서 판단할 수 있는 오류**이기 때문이다. 스펙에서는 이런 종류의 오류를 Early Error라고 부르며, 해당 스크립트/함수 본문은 **단 한 줄도 실행되지 않고** 통째로 에러가 난다.

```js
console.log('이 줄도 실행되지 않는다');
let x = 1;
let x = 2; // SyntaxError
```

디버깅할 때 "분명히 앞부분은 실행됐어야 하는데 아무것도 안 찍힌다" 싶으면 SyntaxError를 의심하면 된다. 반대로 TDZ 위반은 실행 중에 발생하는 ReferenceError이므로 그 앞 코드는 정상적으로 실행된다.

</details>



### 블록 레벨 스코프

var 키워드로 선언한 변수는 **함수 레벨 스코프**를 따른다. 하지만 let 키워드로 선언한 변수는 모든 코드 블록(함수, if문, for문, while문, try/catch문 등)을 **지역 스코프**로 인정하는 **블록 레벨 스코프**를 따른다.

```js
let foo = 1; // 전역 변수

{
  let foo = 2; // 지역 변수
  let bar = 3; // 지역 변수
}

console.log(foo); // 1
console.log(bar); // ReferenceError: bar is not defined
```

코드 블록 내에서 선언된 `foo`, `bar` 변수는 모두 지역 변수이다. 전역에서 선언된 `foo` 변수와 코드 블록 내에서 선언된 `foo` 변수는 별개의 변수이다. `bar` 변수 또한 블록 내에서만 유효한 지역 변수이므로 전역에서는 참조할 수 없다.



### 변수 호이스팅

var 키워드로 선언한 변수와 달리 **let 키워드로 선언한 변수는 변수 호이스팅이 발생하지 않는 것처럼 동작**한다.

```js
console.log(foo); // ReferenceError: Cannot access 'foo' before initialization
let foo;
```

let 키워드로 선언한 변수를 선언문 이전에 참조하면 참조 에러(ReferenceError)가 발생한다. 얼핏 호이스팅이 발생하지 않는 것처럼 보이지만, **실제로는 호이스팅이 발생하고 있다.**



**선언 단계와 초기화 단계**

var 키워드로 선언한 변수는 런타임 이전에 자바스크립트 엔진에 의해 암묵적으로 **선언 단계**와 **초기화 단계**가 한 번에 진행된다.

```js
console.log(foo); // undefined

var foo;
console.log(foo); // undefined

foo = 1;
console.log(foo); // 1
```

반면 **let 키워드로 선언한 변수는 선언 단계와 초기화 단계가 분리되어 진행**된다.  
런타임 이전에 자바스크립트 엔진에 의해 암묵적으로 선언 단계가 먼저 실행되지만, 초기화 단계는 변수 선언문에 도달했을 때 실행된다.

```js
// 런타임 이전에 선언 단계가 실행됨. 아직 초기화되지 않음
// 초기화 이전의 일시적 사각지대에서는 변수를 참조할 수 없다
console.log(foo); // ReferenceError

let foo; // 변수 선언문에서 초기화 단계가 실행됨
console.log(foo); // undefined

foo = 1; // 할당문에서 할당 단계가 실행됨
console.log(foo); // 1
```

초기화 단계가 실행되기 이전, 즉 자바스크립트 엔진이 변수 선언문을 만나기 이전까지 변수를 참조할 수 없다.  
=&gt; 스코프의 시작 지점부터 초기화 시작 지점까지 변수를 참조할 수 없는 이 구간을 **일시적 사각지대(TDZ, Temporal Dead Zone)**라고 부른다.



> `TDZ (Temporal Dead Zone)` 스코프의 선두에서부터 변수의 초기화(선언문) 지점까지의 구간. 이 구간에서 변수를 참조하면 ReferenceError가 발생한다.

결국 let 키워드로 선언한 변수는 **호이스팅이 발생하지 않는 것처럼 보이지만, 실제로는 호이스팅이 발생**한다.

```js
let foo = 1; // 전역 변수

{
  console.log(foo); // ReferenceError: Cannot access 'foo' before initialization
  let foo = 2; // 지역 변수
}
```

만약 호이스팅이 발생하지 않는다면 위 예제의 `console.log(foo)`는 전역 변수 `foo`의 값 1을 출력해야 한다. 하지만 let 키워드로 선언한 변수 역시 여전히 호이스팅이 발생하기 때문에 참조 에러가 발생한다.

<details class="orca-details">
<summary>let은 호이스팅이 발생하는데 왜 &quot;호이스팅이 발생하지 않는 것처럼&quot; 동작한다고 할까?</summary>

호이스팅의 본질은 "변수 선언이 스코프의 선두로 끌어 올려진 것처럼 동작하는 것"이다. let도 이 점에서는 var와 동일하게, 선언이 스코프의 선두로 끌어 올려져 **선언 단계**가 먼저 처리된다. 다만 var는 선언과 동시에 undefined로 **초기화**까지 끝내버리기 때문에 선언문 이전에 참조해도 undefined가 나온다. 반면 let은 선언 단계와 초기화 단계를 **분리**해서, 실제 선언문에 도달하기 전까지는 초기화되지 않은 상태(TDZ)로 남겨둔다. 그래서 참조하면 에러가 난다. 즉 "호이스팅이 아예 없는 것"이 아니라, "초기화가 지연되어 마치 없는 것처럼 보이는 것"이라고 이해하는 것이 정확하다.

</details>



### 전역 객체와 let

var 키워드로 선언한 전역 변수와 전역 함수, 그리고 선언하지 않은 변수에 값을 할당한 암묵적 전역은 전역 객체 window의 프로퍼티가 된다. 하지만 **let 키워드로 선언한 전역 변수는 전역 객체의 프로퍼티가 아니다.**

```js
// 브라우저 환경 기준
var x = 1;
console.log(window.x); // 1
console.log(x); // 1

let y = 2;
console.log(window.y); // undefined
console.log(y); // 2
```

let 키워드로 선언한 전역 변수는 보이지 않는 개념적인 블록(전역 렉시컬 환경의 선언적 환경 레코드) 내에 존재하게 된다. 따라서 window.y와 같이 접근할 수 없다.



## const 키워드

const 키워드는 상수(constant)를 선언하기 위해 사용한다. 하지만 반드시 상수만을 위해 사용하는 것은 아니다. const 키워드의 특징은 let 키워드와 대부분 동일하므로 차이점을 중심으로 살펴본다.

### 선언과 초기화

**const 키워드로 선언한 변수는 반드시 선언과 동시에 초기화해야 한다.**

```js
const foo = 1;

const bar; // SyntaxError: Missing initializer in const declaration
```

const 키워드로 선언한 변수도 let 키워드와 마찬가지로 블록 레벨 스코프를 가지며, 변수 호이스팅이 발생하지 않는 것처럼 동작한다.

```js
{
  console.log(foo); // ReferenceError: Cannot access 'foo' before initialization
  const foo = 1;
  console.log(foo); // 1
}

console.log(foo); // ReferenceError: foo is not defined
```



### 재할당 금지

var 또는 let 키워드로 선언한 변수는 재할당이 자유롭지만, **const 키워드로 선언한 변수는 재할당이 금지된다.**

```js
const foo = 1;
foo = 2; // TypeError: Assignment to constant variable.
```



### 상수

const 키워드로 선언한 변수에 원시 값을 할당한 경우, 원시 값은 변경할 수 없는 값(immutable value)이고 const 키워드에 의해 재할당이 금지되므로 할당된 값을 변경할 수 있는 방법은 없다. 이러한 특성을 이용해 **상수를 표현**하는 데 const 키워드를 사용한다.

상수는 상태 유지와 가독성, 유지보수의 편의를 위해 적극적으로 사용해야 한다.

```js
// 세율을 의미하는 0.1은 변경될 수 없는 상수로서 사용된다
// 변수 이름을 대문자로 선언해 상수임을 명확히 나타낸다
const TAX_RATE = 0.1;

// 세전 가격
let preTaxPrice = 100;

// 세후 가격
let afterTaxPrice = preTaxPrice + (preTaxPrice * TAX_RATE);

console.log(afterTaxPrice); // 110
```

일반적으로 상수의 이름은 **대문자로 선언**해 상수임을 명확히 나타낸다. 여러 단어로 이루어진 경우 언더스코어(_)로 구분해서 스네이크 케이스로 표현하는 것이 일반적이다.



### const 키워드와 객체

const 키워드로 선언한 변수에 **객체를 할당한 경우 값을 변경할 수 있다.** const 키워드는 재할당을 금지할 뿐 "불변"을 의미하지는 않는다.

```js
const person = {
  name: 'Lee'
};

// 객체는 변경 가능한 값이다. 따라서 재할당 없이 변경이 가능하다
person.name = 'Kim';

console.log(person); // {name: "Kim"}
```

const 키워드는 재할당을 금지할 뿐, 프로퍼티의 동적 생성, 삭제, 프로퍼티 값의 변경을 통해 객체를 변경하는 것은 가능하다.  
=&gt; 이때 객체가 변경되더라도 변수에 할당된 참조 값은 변경되지 않는다.

<details class="orca-details" open>
<summary>const로 선언한 객체를 정말 변경 불가능하게 만들려면 어떻게 할까?</summary>

`const`가 막는 것은 변수가 가리키는 **참조 값의 재할당**뿐이다. 객체 내부의 프로퍼티는 얼마든지 바꿀 수 있다. 객체 자체를 얼려버리고 싶다면 `Object.freeze()`를 사용한다. 다만 이것도 얕은(shallow) 동결이라 중첩된 객체까지는 얼리지 못하므로, 깊은 동결이 필요하면 재귀적으로 `Object.freeze()`를 적용하거나 불변 라이브러리(Immer 등)를 사용한다. 참고로 `Object.freeze()`로 얼린 객체를 변경하려 하면 strict mode에서는 에러가 나지만, 일반 모드에서는 조용히 무시되므로 주의가 필요하다.

</details>



## var vs. let vs. const

변수 선언에는 기본적으로 **const**를 사용하고, let은 재할당이 필요한 경우에 한정해 사용하는 것이 좋다. var 키워드는 사용하지 않는 것이 바람직하다.

- **ES6를 사용한다면 var 키워드는 사용하지 않는다.**
- **재할당이 필요한 경우에 한정해 let 키워드를 사용한다.** 이때 변수의 스코프는 최대한 좁게 만든다.
- **변경이 발생하지 않고 읽기 전용으로 사용하는(재할당이 필요 없는 상수) 원시 값과 객체에는 const 키워드를 사용한다.** const 키워드는 재할당을 금지하므로 var, let보다 안전하다.

일단 변수를 선언할 때는 const 키워드를 사용하고, 이후에 재할당이 필요하다면 그때 let 키워드로 변경해도 늦지 않다. 재할당이 필요 없는 상수를 const 키워드로 선언하면 유지보수성이 향상된다.

<details class="orca-details">
<summary>&quot;일단 const로 시작하라&quot;는 관례가 실무에서 주는 이점이 더 있을까?...</summary>

가장 큰 이점은 **의도의 명시**이다. `const`로 선언된 변수를 보면 "이 값은 이 스코프 안에서 다시 바뀌지 않는다"는 것을 한눈에 알 수 있어, 코드를 읽을 때 추적해야 할 상태의 수가 줄어든다. 반대로 `let`이 보이면 "아, 이건 어딘가에서 재할당되는구나" 하고 경계하게 된다. 즉 키워드 자체가 일종의 문서 역할을 한다. 또한 재할당을 실수로 했을 때 즉시 에러로 잡아주므로 버그를 조기에 발견할 수 있다. React 같은 환경에서 불변성을 중시하는 코딩 스타일과도 잘 맞아떨어진다.

</details>


# 16장. 프로퍼티 어트리뷰트

## 내부 슬롯과 내부 메서드

앞으로 나올 프로퍼티 어트리뷰트를 이해하기 위해서는 먼저 **내부 슬롯(internal slot)**과 **내부 메서드(internal method)**의 개념을 알아야 한다.

내부 슬롯과 내부 메서드는 자바스크립트 엔진의 구현 알고리즘을 설명하기 위해 ECMAScript 사양에서 사용하는 **의사 프로퍼티(pseudo property)**와 **의사 메서드(pseudo method)**이다. ECMAScript 사양에 등장하는 이중 대괄호(`[[...]]`)로 감싼 이름들이 내부 슬롯과 내부 메서드이다.

내부 슬롯과 내부 메서드는 자바스크립트 엔진의 내부 로직이므로 원칙적으로 직접 접근하거나 호출할 수 있는 방법을 제공하지 않는다.  
=&gt; 단, 일부 내부 슬롯과 내부 메서드에 한하여 간접적으로 접근할 수 있는 수단을 제공하기는 한다.

예를 들어 모든 객체는 `[[Prototype]]`이라는 내부 슬롯을 갖는다. 이 내부 슬롯은 원칙적으로 직접 접근할 수 없지만, `__proto__`를 통해 간접적으로 접근할 수 있다.

```js
const o = {};

// 내부 슬롯은 직접 접근할 수 없다
o.[[Prototype]] // Uncaught SyntaxError: Unexpected token '['

// 단, 일부 내부 슬롯과 내부 메서드에 한하여 간접적으로 접근할 수 있는 수단을 제공한다
o.__proto__ // Object.prototype
```



<details class="orca-details">
<summary>내부 슬롯·내부 메서드는 결국 뭘 위해 존재하는 개념일까?</summary>

**스펙을 기술하기 위한 가상의 장치**다. ECMAScript 스펙은 "객체는 이런 상태를 갖고, 이런 동작을 한다"를 설명해야 하는데, 그것을 자바스크립트 문법으로 표현하면 순환 정의가 되어버린다. 그래서 `[[Prototype]]`, `[[Extensible]]` 같은 **내부 상태**와 `[[Get]]`, `[[Set]]` 같은 **내부 동작**에 이름을 붙여 놓고, 실제 엔진 구현은 이 명세를 만족하기만 하면 되도록 만들었다.

따라서 "내부 슬롯은 개발자가 직접 접근할 수 없다"는 문장은 정보 은닉이라기보다 **"애초에 자바스크립트 값이 아니다"**에 가깝다. 다만 일부에는 간접 접근 수단이 마련되어 있는데, `[[Prototype]]`에 대한 `Object.getPrototypeOf()`, `[[Extensible]]`에 대한 `Object.isExtensible()`이 그렇다.

</details>



## 프로퍼티 어트리뷰트와 프로퍼티 디스크립터 객체

자바스크립트 엔진은 **프로퍼티를 생성할 때 프로퍼티의 상태를 나타내는 프로퍼티 어트리뷰트를 기본값으로 자동 정의**한다.

여기서 프로퍼티의 상태란 다음과 같다.

- 프로퍼티의 값(value)
- 값의 갱신 가능 여부(writable)
- 열거 가능 여부(enumerable)
- 재정의 가능 여부(configurable)

프로퍼티 어트리뷰트는 자바스크립트 엔진이 관리하는 내부 상태 값인 **내부 슬롯** `[[Value]]`, `[[Writable]]`, `[[Enumerable]]`, `[[Configurable]]`이다. 따라서 직접 접근할 수 없지만, `**Object.getOwnPropertyDescriptor` 메서드를 사용하여 간접적으로 확인**할 수 있다.

```js
const person = {
  name: 'Lee'
};

// 프로퍼티 어트리뷰트 정보를 제공하는 프로퍼티 디스크립터 객체를 반환한다
console.log(Object.getOwnPropertyDescriptor(person, 'name'));
// { value: "Lee", writable: true, enumerable: true, configurable: true }
```

`Object.getOwnPropertyDescriptor` 메서드는 첫 번째 매개변수에 객체의 참조를, 두 번째 매개변수에 프로퍼티 키를 문자열로 전달한다. 이때 **프로퍼티 어트리뷰트 정보를 제공하는 프로퍼티 디스크립터(PropertyDescriptor) 객체를 반환**한다. 존재하지 않는 프로퍼티나 상속받은 프로퍼티에 대한 프로퍼티 디스크립터를 요구하면 undefined가 반환된다.

ES8에서 도입된 `**Object.getOwnPropertyDescriptors` 메서드**는 모든 프로퍼티의 프로퍼티 어트리뷰트 정보를 제공하는 프로퍼티 디스크립터 객체들을 반환한다.

```js
const person = {
  name: 'Lee'
};

person.age = 20;

// 모든 프로퍼티의 프로퍼티 어트리뷰트 정보를 제공하는 디스크립터 객체들을 반환한다
console.log(Object.getOwnPropertyDescriptors(person));
/*
{
  name: { value: "Lee", writable: true, enumerable: true, configurable: true },
  age:  { value: 20,    writable: true, enumerable: true, configurable: true }
}
*/
```



## 데이터 프로퍼티와 접근자 프로퍼티

프로퍼티는 다음과 같이 두 가지로 구분할 수 있다.

- **데이터 프로퍼티(data property)** : 키와 값으로 구성된 일반적인 프로퍼티. 지금까지 살펴본 모든 프로퍼티는 데이터 프로퍼티다.
- **접근자 프로퍼티(accessor property)** : 자체적으로는 값을 갖지 않고, 다른 데이터 프로퍼티의 값을 읽거나 저장할 때 호출되는 접근자 함수(accessor function)로 구성된 프로퍼티

### 데이터 프로퍼티

데이터 프로퍼티는 다음과 같은 프로퍼티 어트리뷰트를 갖는다. 이 어트리뷰트는 자바스크립트 엔진이 프로퍼티를 생성할 때 기본값으로 자동 정의된다.


| 프로퍼티 어트리뷰트         | 디스크립터 객체의 프로퍼티 | 설명                                                                       |
| :------------------ | :-------------- | :------------------------------------------------------------------------ |
| `[[Value]]`        | value          | 프로퍼티 키를 통해 프로퍼티 값에 접근하면 반환되는 값                                           |
| `[[Writable]]`     | writable       | 프로퍼티 값의 변경 가능 여부(불리언). false인 경우 값을 변경할 수 없는 읽기 전용 프로퍼티가 된다              |
| `[[Enumerable]]`   | enumerable     | 프로퍼티의 열거 가능 여부(불리언). false인 경우 for...in 문이나 Object.keys 메서드 등으로 열거할 수 없다 |
| `[[Configurable]]` | configurable   | 프로퍼티의 재정의 가능 여부(불리언). false인 경우 프로퍼티의 삭제, 어트리뷰트 값의 변경이 금지된다              |


프로퍼티가 생성될 때 `[[Value]]`는 프로퍼티 값으로 초기화되며, 나머지 `[[Writable]]`, `[[Enumerable]]`, `[[Configurable]]`의 값은 **true로 초기화**된다. 이는 프로퍼티를 동적으로 추가해도 마찬가지이다.



### 접근자 프로퍼티

접근자 프로퍼티는 자체적으로는 값을 갖지 않고 **다른 데이터 프로퍼티의 값을 읽거나 저장할 때 사용하는 접근자 함수**로 구성된 프로퍼티다.


| 프로퍼티 어트리뷰트         | 디스크립터 객체의 프로퍼티 | 설명                                                                               |
| :------------------ | :-------------- | :-------------------------------------------------------------------------------- |
| `[[Get]]`          | get            | 접근자 프로퍼티를 통해 데이터 프로퍼티의 값을 읽을 때 호출되는 접근자 함수. getter 함수가 호출되고 그 결과가 프로퍼티 값으로 반환된다  |
| `[[Set]]`          | set            | 접근자 프로퍼티를 통해 데이터 프로퍼티의 값을 저장할 때 호출되는 접근자 함수. setter 함수가 호출되고 그 결과가 프로퍼티 값으로 저장된다 |
| `[[Enumerable]]`   | enumerable     | 데이터 프로퍼티의 `[[Enumerable]]`과 같다                                                   |
| `[[Configurable]]` | configurable   | 데이터 프로퍼티의 `[[Configurable]]`과 같다                                                 |


접근자 함수는 **getter/setter 함수**라고도 부른다. 접근자 프로퍼티는 getter와 setter 함수를 모두 정의할 수도 있고 하나만 정의할 수도 있다.

```js
const person = {
  // 데이터 프로퍼티
  firstName: 'Ungmo',
  lastName: 'Lee',

  // fullName은 접근자 함수로 구성된 접근자 프로퍼티다
  // getter 함수
  get fullName() {
    return `${this.firstName} ${this.lastName}`;
  },
  // setter 함수
  set fullName(name) {
    // 배열 디스트럭처링 할당
    [this.firstName, this.lastName] = name.split(' ');
  }
};

// 데이터 프로퍼티를 통한 프로퍼티 값의 참조
console.log(person.firstName + ' ' + person.lastName); // Ungmo Lee

// 접근자 프로퍼티를 통한 프로퍼티 값의 저장
// 접근자 프로퍼티 fullName에 값을 저장하면 setter 함수가 호출된다
person.fullName = 'Heegun Lee';
console.log(person); // {firstName: "Heegun", lastName: "Lee"}

// 접근자 프로퍼티를 통한 프로퍼티 값의 참조
// 접근자 프로퍼티 fullName에 접근하면 getter 함수가 호출된다
console.log(person.fullName); // Heegun Lee

// firstName은 데이터 프로퍼티다
console.log(Object.getOwnPropertyDescriptor(person, 'firstName'));
// { value: "Heegun", writable: true, enumerable: true, configurable: true }

// fullName은 접근자 프로퍼티다
console.log(Object.getOwnPropertyDescriptor(person, 'fullName'));
// { get: f, set: f, enumerable: true, configurable: true }
```

`firstName`, `lastName` 프로퍼티는 일반적인 데이터 프로퍼티다. 하지만 `fullName`은 접근자 함수, 즉 getter/setter 함수로 구성된 접근자 프로퍼티이다.

접근자 프로퍼티 `fullName`에 접근하면 내부적으로 `[[Get]]` 내부 메서드가 호출되어 다음과 같이 동작한다.

1. 프로퍼티 키가 유효한지 확인한다. 프로퍼티 키는 문자열 또는 심벌이어야 한다.
2. 프로토타입 체인에서 프로퍼티를 검색한다.
3. 검색된 fullName 프로퍼티가 데이터 프로퍼티인지 접근자 프로퍼티인지 확인한다.
4. 접근자 프로퍼티 fullName의 프로퍼티 어트리뷰트 `[[Get]]`의 값, 즉 getter 함수를 호출하여 그 결과를 반환한다.

<details class="orca-details">
<summary>데이터 프로퍼티인지 접근자 프로퍼티인지 구분하는 방법이 있나요?</summary>

`Object.getOwnPropertyDescriptor`로 프로퍼티 디스크립터 객체를 확인해보면 명확하게 구분된다. **데이터 프로퍼티의 디스크립터는 `value`, `writable`, `enumerable`, `configurable`** 프로퍼티를 갖는다. 반면 **접근자 프로퍼티의 디스크립터는 `get`, `set`, `enumerable`, `configurable`** 프로퍼티를 갖는다. 즉 `value`/`writable`이 있으면 데이터 프로퍼티, `get`/`set`이 있으면 접근자 프로퍼티라고 판단할 수 있다. 하나의 프로퍼티가 두 종류를 동시에 가질 수는 없다.

</details>



## 프로퍼티 정의

**프로퍼티 정의**란 새로운 프로퍼티를 추가하면서 프로퍼티 어트리뷰트를 명시적으로 정의하거나, 기존 프로퍼티의 프로퍼티 어트리뷰트를 재정의하는 것을 말한다.

`Object.defineProperty` 메서드를 사용하면 프로퍼티의 어트리뷰트를 정의할 수 있다. 인수로는 객체의 참조, 데이터 프로퍼티의 키인 문자열, 프로퍼티 디스크립터 객체를 전달한다.

```js
const person = {};

// 데이터 프로퍼티 정의
Object.defineProperty(person, 'firstName', {
  value: 'Ungmo',
  writable: true,
  enumerable: true,
  configurable: true
});

Object.defineProperty(person, 'lastName', {
  value: 'Lee'
  // 디스크립터 객체의 프로퍼티를 누락시키면 undefined, false가 기본값이다
});

let descriptor = Object.getOwnPropertyDescriptor(person, 'lastName');
console.log('lastName', descriptor);
// lastName { value: "Lee", writable: false, enumerable: false, configurable: false }

// [[Enumerable]] 값이 false인 경우
// 해당 프로퍼티는 for...in 문이나 Object.keys 등으로 열거할 수 없다
console.log(Object.keys(person)); // ["firstName"]

// [[Writable]] 값이 false인 경우 해당 프로퍼티의 [[Value]] 값을 변경할 수 없다
person.lastName = 'Kim'; // 에러는 발생하지 않고 무시된다 (strict mode에서는 에러)
console.log(person.lastName); // Lee
```

`**Object.defineProperty` 메서드로 프로퍼티를 정의할 때 디스크립터 객체의 프로퍼티를 일부 생략하면 다음과 같은 값이 기본값으로 적용된다.**


| 디스크립터 객체의 프로퍼티 | 대응하는 어트리뷰트         | 생략했을 때의 기본값 |
| :-------------- | :------------------ | :----------- |
| value          | `[[Value]]`        | undefined   |
| get            | `[[Get]]`          | undefined   |
| set            | `[[Set]]`          | undefined   |
| writable       | `[[Writable]]`     | false       |
| enumerable     | `[[Enumerable]]`   | false       |
| configurable   | `[[Configurable]]` | false       |


`Object.defineProperty` 메서드는 한 번에 하나의 프로퍼티만 정의할 수 있다. **여러 개의 프로퍼티를 한 번에 정의하려면 `Object.defineProperties` 메서드를 사용**한다.

```js
const person = {};

Object.defineProperties(person, {
  // 데이터 프로퍼티 정의
  firstName: {
    value: 'Ungmo',
    writable: true,
    enumerable: true,
    configurable: true
  },
  lastName: {
    value: 'Lee',
    writable: true,
    enumerable: true,
    configurable: true
  },
  // 접근자 프로퍼티 정의
  fullName: {
    get() {
      return `${this.firstName} ${this.lastName}`;
    },
    set(name) {
      [this.firstName, this.lastName] = name.split(' ');
    },
    enumerable: true,
    configurable: true
  }
});

person.fullName = 'Heegun Lee';
console.log(person); // {firstName: "Heegun", lastName: "Lee"}
```



<details class="orca-details">
<summary>클래스 필드가 defineProperty로 동작한다는 게 무슨 뜻일까?</summary>

클래스 필드(`class A { x = 1 }`)는 `this.x = 1`이라는 **할당(`[[Set]]`)이 아니라 `Object.defineProperty`에 해당하는 정의(`[[Define]]`)** 시맨틱으로 동작한다. 둘의 차이는 **상위 클래스에 같은 이름의 setter가 있을 때** 드러난다.

```js
class Base { set x(v) { console.log('setter 호출'); } }
class Derived extends Base { x = 1; }  // setter가 호출되지 않고 그냥 덮어씀
```

타입스크립트에서 `useDefineForClassFields` 옵션을 두고 논쟁이 있었던 이유가 바로 이것이다. 예전 TS는 필드를 할당으로 컴파일했기 때문에 표준과 동작이 달랐다. 실무에서 데코레이터나 MobX 같은 라이브러리와 클래스 필드를 함께 쓸 때 이 차이가 버그로 나타난다.

</details>

<details class="orca-details">
<summary>private 필드(#)는 프로퍼티 디스크립터로 들여다볼 수 있을까?</summary>

없다. `#x`는 프로퍼티가 아니라 **내부 슬롯에 가까운 별도의 저장소**라서 `Object.getOwnPropertyNames`, `Reflect.ownKeys`, `JSON.stringify` 어디에도 나타나지 않고, Proxy로도 가로챌 수 없다. 반면 `_x` 같은 관용적 네이밍은 그냥 일반 프로퍼티이므로 얼마든지 접근 가능하다.

이 성질을 이용해 **브랜드 체크**(진짜 그 클래스의 인스턴스인지 확인)를 하는 기법도 있다.

```js
class A {
  #brand;
  static isA(obj) { return #brand in obj; }  // ES2022
}
```

"자바스크립트에 진짜 private가 생겼다"는 말의 근거가 여기에 있다.

</details>



## 객체 변경 방지

객체는 변경 가능한 값이므로 재할당 없이 직접 변경할 수 있다. 즉 프로퍼티를 추가하거나 삭제할 수 있고, 프로퍼티 값을 갱신할 수 있으며, 프로퍼티 어트리뷰트를 재정의할 수도 있다.

자바스크립트는 객체의 변경을 방지하는 다양한 메서드를 제공하며, 이들은 **객체의 변경을 금지하는 강도가 다르다.**


| 구분       | 메서드                        | 프로퍼티 추가 | 프로퍼티 삭제 | 프로퍼티 값 읽기 | 프로퍼티 값 쓰기 | 프로퍼티 어트리뷰트 재정의 |
| :-------- | :-------------------------- | :-------: | :-------: | :---------: | :---------: | :--------------: |
| 객체 확장 금지 | `Object.preventExtensions` | ✕       | ○       | ○         | ○         | ○              |
| 객체 밀봉    | `Object.seal`              | ✕       | ✕       | ○         | ○         | ✕              |
| 객체 동결    | `Object.freeze`            | ✕       | ✕       | ○         | ✕         | ✕              |


### 객체 확장 금지

`Object.preventExtensions` 메서드는 객체의 확장을 금지한다. 확장이 금지된 객체는 **프로퍼티 추가가 금지**된다. 확장 가능한 객체인지 여부는 `Object.isExtensible` 메서드로 확인할 수 있다.

```js
const person = { name: 'Lee' };

// 확장이 가능한 객체다
console.log(Object.isExtensible(person)); // true

// 확장을 금지하여 프로퍼티 추가를 막는다
Object.preventExtensions(person);
console.log(Object.isExtensible(person)); // false

// 프로퍼티 추가는 금지된다 (strict mode에서는 에러)
person.age = 20; // 무시된다
console.log(person); // {name: "Lee"}

// 프로퍼티 삭제는 가능하다
delete person.name;
console.log(person); // {}
```



### 객체 밀봉

`Object.seal` 메서드는 객체를 밀봉한다. 밀봉된 객체는 **읽기와 쓰기만 가능**하다. 즉 프로퍼티 추가 및 삭제와 프로퍼티 어트리뷰트 재정의가 금지된다. 밀봉된 객체인지 여부는 `Object.isSealed` 메서드로 확인할 수 있다.

```js
const person = { name: 'Lee' };

// 밀봉되지 않은 객체다
console.log(Object.isSealed(person)); // false

// 밀봉하여 프로퍼티 추가, 삭제, 재정의를 금지한다
Object.seal(person);
console.log(Object.isSealed(person)); // true

// 밀봉된 객체는 configurable이 false다
console.log(Object.getOwnPropertyDescriptors(person));
// name: { value: "Lee", writable: true, enumerable: true, configurable: false }

// 프로퍼티 값 갱신은 가능하다
person.name = 'Kim';
console.log(person); // {name: "Kim"}
```



### 객체 동결

`Object.freeze` 메서드는 객체를 동결한다. 동결된 객체는 **읽기만 가능**하다. 즉 프로퍼티 추가 및 삭제, 프로퍼티 어트리뷰트 재정의, 프로퍼티 값 갱신이 모두 금지된다. 동결된 객체인지 여부는 `Object.isFrozen` 메서드로 확인할 수 있다.

```js
const person = { name: 'Lee' };

// 동결되지 않은 객체다
console.log(Object.isFrozen(person)); // false

// 동결하여 모든 변경을 금지한다
Object.freeze(person);
console.log(Object.isFrozen(person)); // true

// 동결된 객체는 writable과 configurable이 false다
console.log(Object.getOwnPropertyDescriptors(person));
// name: { value: "Lee", writable: false, enumerable: true, configurable: false }

// 프로퍼티 값 갱신이 금지된다 (strict mode에서는 에러)
person.name = 'Kim'; // 무시된다
console.log(person); // {name: "Lee"}
```



### 불변 객체

지금까지 살펴본 변경 방지 메서드들은 **얕은 변경 방지(shallow only)**로 직속 프로퍼티만 변경이 방지되고 **중첩 객체까지는 영향을 주지 못한다.**

```js
const person = {
  name: 'Lee',
  address: { city: 'Seoul' }
};

// 얕은 객체 동결
Object.freeze(person);

// 직속 프로퍼티만 동결되고 중첩 객체는 동결하지 못한다
console.log(Object.isFrozen(person)); // true
console.log(Object.isFrozen(person.address)); // false

// 중첩 객체는 변경이 가능하다
person.address.city = 'Busan';
console.log(person); // {name: "Lee", address: {city: "Busan"}}
```

**객체의 중첩 객체까지 동결하여 변경이 불가능한 읽기 전용의 불변 객체를 구현하려면 객체를 값으로 갖는 모든 프로퍼티에 대해 재귀적으로 `Object.freeze` 메서드를 호출**해야 한다.

```js
function deepFreeze(target) {
  // 객체가 아니거나 이미 동결된 객체는 무시하고, 그렇지 않으면 동결한다
  if (target && typeof target === 'object' && !Object.isFrozen(target)) {
    Object.freeze(target);
    // 모든 프로퍼티를 순회하며 재귀적으로 동결한다
    Object.keys(target).forEach(key => deepFreeze(target[key]));
  }
  return target;
}

const person = {
  name: 'Lee',
  address: { city: 'Seoul' }
};

// 깊은 객체 동결
deepFreeze(person);

console.log(Object.isFrozen(person)); // true
// 중첩 객체까지 동결한다
console.log(Object.isFrozen(person.address)); // true

person.address.city = 'Busan';
console.log(person); // {name: "Lee", address: {city: "Seoul"}}
```


# 17장. 생성자 함수에 의한 객체 생성

객체를 생성하는 방식에는 여러 가지가 있다. 가장 일반적이고 간단한 방법은 객체 리터럴을 사용하는 것이다. 이 외에도 다양한 방법으로 객체를 생성할 수 있는데, 이번 장에서는 **생성자 함수**를 사용해 객체를 생성하는 방식을 살펴본다.

## Object 생성자 함수

`new` 연산자와 함께 `Object` 생성자 함수를 호출하면 빈 객체를 생성하여 반환한다. 이후 프로퍼티나 메서드를 추가하여 객체를 완성할 수 있다.

```js
// 빈 객체의 생성
const person = new Object();

// 프로퍼티 추가
person.name = 'sozzang';
person.sayHello = function () {
  console.log('Hi! My name is ' + this.name);
};

console.log(person); // {name: "sozzang", sayHello: f}
person.sayHello(); // Hi! My name is sozzang
```

> `생성자 함수(constructor)` new 연산자와 함께 호출하여 객체(인스턴스)를 생성하는 함수  
> `인스턴스(instance)` 생성자 함수에 의해 생성된 객체

자바스크립트는 Object 생성자 함수 이외에도 String, Number, Boolean, Function, Array, Date, RegExp, Promise 등의 **빌트인(built-in) 생성자 함수**를 제공한다.

반드시 Object 생성자 함수를 사용해 빈 객체를 생성해야 하는 것은 아니다. 오히려 객체를 생성하는 방법으로 객체 리터럴을 사용하는 것이 더 간편하다.  
=&gt; 특별한 이유가 없다면 Object 생성자 함수는 그닥 유용하진 않다.



<details class="orca-details">
<summary>Object.create(null)로 만든 객체는 뭐가 다를까?</summary>

`[[Prototype]]`이 `null`이라서 `Object.prototype`**의 어떤 것도 상속받지 않는다.** `toString`, `hasOwnProperty`, `__proto__`가 전부 없는 순수한 키-값 저장소가 된다.

```js
const dict = Object.create(null);
dict.toString;          // undefined
'key' in dict;          // 여전히 동작 (in은 문법)
```

이게 왜 중요하냐면, 일반 객체를 딕셔너리로 쓰면 **상속받은 이름과 충돌**하기 때문이다. `obj['constructor']`나 `obj['toString']`이 사용자 데이터가 아닌데도 truthy로 나오는 사고가 실제로 일어난다. 다만 요즘은 이런 용도라면 `Map`**을 쓰는 것이 정석**이다. `Map`은 문자열이 아닌 키를 지원하고, `size`가 있고, 삽입 순서가 보장되며, 프로토타입 오염 걱정도 없다.

</details>



## 생성자 함수

### 객체 리터럴에 의한 객체 생성 방식의 문제점

객체 리터럴에 의한 객체 생성 방식은 직관적이고 간편하다. 하지만 **단 하나의 객체만 생성**한다.  
=&gt; 동일한 프로퍼티를 갖는 객체를 여러 개 생성해야 하는 경우, 매번 같은 프로퍼티를 기술해야 하기 때문에 비효율적이다.

```js
const circle1 = {
  radius: 5,
  getDiameter() {
    return 2 * this.radius;
  }
};

console.log(circle1.getDiameter()); // 10

const circle2 = {
  radius: 10,
  getDiameter() {
    return 2 * this.radius;
  }
};

console.log(circle2.getDiameter()); // 20
```

객체는 프로퍼티를 통해 객체 고유의 상태(state)를 표현하고, 메서드를 통해 상태 데이터를 참조하고 조작하는 동작(behavior)을 표현한다. 위 예제의 `circle1`, `circle2` 객체는 프로퍼티 값(radius)은 다르지만 메서드(getDiameter)는 완전히 동일하다.  
=&gt; 객체마다 프로퍼티 값이 다를 수는 있지만 메서드는 내용이 동일한 경우가 일반적인데, 객체 리터럴은 이런 동일한 구조의 객체를 여러 개 생성할 때 같은 코드를 반복해야 한다.

### 생성자 함수에 의한 객체 생성 방식의 장점

생성자 함수에 의한 객체 생성 방식은 마치 **객체(인스턴스)를 생성하기 위한 템플릿(클래스)처럼** 생성자 함수를 사용하여 프로퍼티 구조가 동일한 객체 여러 개를 간편하게 생성할 수 있다.

```js
// 생성자 함수
function Circle(radius) {
  // 생성자 함수 내부의 this는 생성자 함수가 생성할 인스턴스를 가리킨다
  this.radius = radius;
  this.getDiameter = function () {
    return 2 * this.radius;
  };
}

// 인스턴스의 생성
const circle1 = new Circle(5);  // 반지름이 5인 Circle 객체를 생성
const circle2 = new Circle(10); // 반지름이 10인 Circle 객체를 생성

console.log(circle1.getDiameter()); // 10
console.log(circle2.getDiameter()); // 20
```

> `this` 객체 자신의 프로퍼티나 메서드를 참조하기 위한 자기 참조 변수(self-referencing variable). this가 가리키는 값, 즉 this 바인딩은 함수 호출 방식에 따라 동적으로 결정된다.


| 함수 호출 방식    | this가 가리키는 값(this 바인딩) |
| :----------- | :---------------------- |
| 일반 함수로서 호출  | 전역 객체                  |
| 메서드로서 호출    | 메서드를 호출한 객체(마침표 앞의 객체) |
| 생성자 함수로서 호출 | 생성자 함수가 (미래에) 생성할 인스턴스 |


생성자 함수는 이름 그대로 객체(인스턴스)를 생성하는 함수다. 다만 **자바나 C++ 같은 클래스 기반 객체지향 언어의 생성자와는 다르게 형식이 정해져 있지 않고 일반 함수와 동일한 방법으로 정의**하며, `**new` 연산자와 함께 호출하면 해당 함수는 생성자 함수로 동작**한다. 만약 new 연산자와 함께 호출하지 않으면 생성자 함수가 아니라 일반 함수로 동작한다.

```js
// new 연산자와 함께 호출하지 않으면 일반 함수로서 호출된다
const circle3 = Circle(15);

// 일반 함수로서 호출된 Circle은 반환문이 없으므로 undefined를 반환한다
console.log(circle3); // undefined

// 일반 함수로서 호출된 Circle 내부의 this는 전역 객체를 가리킨다
console.log(radius); // 15
```



<details class="orca-details" open>
<summary>생성자 함수, 팩토리 함수, 클래스는 어떻게 골라 써야 할까?</summary>

- **생성자 함수**: ES6 이전 방식. 새로 작성할 이유는 거의 없지만 레거시 코드에서 만난다.
- **클래스**: 생성자 함수의 문법적 개선판이면서, `new` 강제·상속 문법·private 필드 등 실질적 기능이 추가됐다. 상속 계층이 필요하거나 라이브러리 API로 인스턴스를 노출할 때 적합하다.
- **팩토리 함수**: `new` 없이 호출하고 객체를 반환한다. 클로저로 은닉이 가능하고, `this` 관련 문제에서 자유롭다.

```js
const createCounter = (init = 0) => {
  let count = init;                          // 진짜 private
  return { increment: () => ++count, get value() { return count; } };
};
```

React가 클래스 컴포넌트에서 함수 컴포넌트+훅으로 옮겨간 흐름도 큰 틀에서는 같은 방향이다. `this`**를 다루지 않아도 되는 코드가 예측 가능하다**는 것이 핵심 동기다.

</details>



### 생성자 함수의 인스턴스 생성 과정

생성자 함수의 역할은 프로퍼티 구조가 동일한 인스턴스를 생성하기 위한 템플릿(클래스)으로서 동작하여 **① 인스턴스를 생성**하는 것과 **② 생성된 인스턴스를 초기화**(인스턴스 프로퍼티 추가 및 초기값 할당)하는 것이다.  
=&gt; 인스턴스를 생성하는 것은 필수, 생성된 인스턴스를 초기화하는 것은 옵션이다.

자바스크립트 엔진은 `new` 연산자와 함께 생성자 함수가 호출되면 암묵적으로 다음 과정을 거쳐 인스턴스를 생성하고 초기화한 후 암묵적으로 인스턴스를 반환한다.

**1️⃣ 인스턴스 생성과 this 바인딩**

암묵적으로 빈 객체가 생성된다. 이 빈 객체가 바로 생성자 함수가 생성한 인스턴스다. 그리고 암묵적으로 생성된 빈 객체, 즉 인스턴스는 **this에 바인딩**된다. 이 처리는 함수 몸체의 코드가 한 줄씩 실행되는 런타임 이전에 실행된다.

> `바인딩(binding)` 식별자와 값을 연결하는 과정. this 바인딩은 this(키워드로 분류되지만 식별자 역할을 함)와 this가 가리킬 객체를 바인딩하는 것이다.

**2️⃣ 인스턴스 초기화**

생성자 함수에 기술되어 있는 코드가 한 줄씩 실행되어 this에 바인딩되어 있는 인스턴스를 초기화한다. 즉, this에 바인딩되어 있는 인스턴스에 프로퍼티나 메서드를 추가하고 생성자 함수가 인수로 전달받은 초기값을 인스턴스 프로퍼티에 할당하여 초기화하거나 고정값을 할당한다. 이 처리는 개발자가 기술한다.

**3️⃣ 인스턴스 반환**

생성자 함수 내부의 모든 처리가 끝나면 완성된 인스턴스가 바인딩된 this가 암묵적으로 반환된다.

```js
function Circle(radius) {
  // 1. 암묵적으로 빈 객체(인스턴스)가 생성되고 this에 바인딩된다

  // 2. this에 바인딩되어 있는 인스턴스를 초기화한다
  this.radius = radius;
  this.getDiameter = function () {
    return 2 * this.radius;
  };

  // 3. 완성된 인스턴스가 바인딩된 this가 암묵적으로 반환된다
}

const circle = new Circle(1);
console.log(circle); // Circle {radius: 1, getDiameter: f}
```

만약 this가 아닌 다른 객체를 명시적으로 반환하면 return 문에 명시한 객체가 반환된다. 하지만 **명시적으로 원시 값을 반환하면 원시 값 반환은 무시되고 암묵적으로 this가 반환**된다.

```js
function Circle(radius) {
  this.radius = radius;

  // 명시적으로 객체를 반환하면 암묵적인 this 반환이 무시된다
  return {};
}

console.log(new Circle(1)); // {}

// ==================================================

function Circle2(radius) {
  this.radius = radius;

  // 명시적으로 원시 값을 반환하면 원시 값 반환은 무시되고 this가 반환된다
  return 100;
}

console.log(new Circle2(1)); // Circle2 {radius: 1}
```

이처럼 생성자 함수 내부에서 명시적으로 this가 아닌 다른 값을 반환하는 것은 생성자 함수의 기본 동작을 훼손한다.  
=&gt; **생성자 함수 내부에서 return 문은 반드시 생략해야 한다.**

<details class="orca-details">
<summary>객체는 반환하고 원시 값은 무시하는 규칙이 왜 이렇게 비대칭적일까?</summary>

생성자 함수의 목적은 "객체(인스턴스)를 만들어 반환하는 것"이라는 점에서 이해하면 자연스럽다. return으로 객체를 반환하면 "개발자가 의도적으로 다른 객체를 인스턴스로 쓰겠다"는 신호로 볼 수 있으니 그것을 존중한다. 반면 원시 값(숫자, 문자열 등)을 반환하면 "이건 인스턴스가 될 수 없는 값"이므로 실수로 간주하고 무시한 뒤 원래 만들려던 this를 반환한다. 즉 생성자 함수가 언제나 객체를 반환하도록 보장하기 위한 안전장치인 셈이다. 그래서 실무에서는 이런 예외 규칙에 의존하지 말고 아예 return 문을 쓰지 않는 게 좋다.

</details>



### 내부 메서드 [[Call]]과 [[Construct]]

함수 선언문 또는 함수 표현식으로 정의한 함수는 일반적인 함수로서 호출할 수 있는 것은 물론 생성자 함수로서 호출할 수도 있다. 생성자 함수로서 호출한다는 것은 new 연산자와 함께 호출하여 객체를 생성하는 것을 의미한다.

함수는 객체이지만 **일반 객체와는 다르다.** 일반 객체는 호출할 수 없지만 **함수는 호출할 수 있다.** 따라서 함수 객체는 일반 객체가 가지고 있는 내부 슬롯과 내부 메서드는 물론, 함수로서 동작하기 위해 함수 객체만을 위한 `[[Environment]]`, `[[FormalParameters]]` 등의 내부 슬롯과 `[[Call]]`, `[[Construct]]` 같은 내부 메서드를 추가로 가지고 있다.

- 함수가 **일반 함수로서 호출**되면 함수 객체의 내부 메서드 `[[Call]]`이 호출된다.
- 함수가 **new 연산자와 함께 생성자 함수로서 호출**되면 내부 메서드 `[[Construct]]`가 호출된다.

내부 메서드 `[[Call]]`을 갖는 함수 객체를 **callable**이라 하며, 내부 메서드 `[[Construct]]`를 갖는 함수 객체를 **constructor**, `[[Construct]]`를 갖지 않는 함수 객체를 **non-constructor**라고 부른다.

- callable : 호출할 수 있는 객체, 즉 함수
- constructor : 생성자 함수로서 호출할 수 있는 함수
- non-constructor : 생성자 함수로서 호출할 수 없는 함수

호출할 수 없는 객체는 함수 객체가 아니므로, **모든 함수 객체는 반드시 callable**이다. 하지만 모든 함수 객체가 `[[Construct]]`를 갖는 것은 아니다.  
=&gt; 함수 객체는 constructor일 수도 있고 non-constructor일 수도 있다.



### constructor와 non-constructor의 구분

자바스크립트 엔진은 함수 정의를 평가하여 함수 객체를 생성할 때 **함수 정의 방식에 따라 constructor와 non-constructor를 구분**한다.

- **constructor** : 함수 선언문, 함수 표현식, 클래스(클래스도 함수다)
- **non-constructor** : 메서드(ES6 메서드 축약 표현), 화살표 함수

```js
// 일반 함수 정의: 함수 선언문, 함수 표현식
function foo() {}
const bar = function () {};
// 프로퍼티 x의 값으로 할당된 것은 일반 함수로 정의된 함수다. 이는 메서드로 인정하지 않는다
const baz = {
  x: function () {}
};

// 일반 함수로 정의된 함수만이 constructor다
new foo();   // -> foo {}
new bar();   // -> bar {}
new baz.x(); // -> x {}

// 화살표 함수 정의
const arrow = () => {};
new arrow(); // TypeError: arrow is not a constructor

// 메서드 정의: ES6의 메서드 축약 표현만 메서드로 인정한다
const obj = {
  x() {}
};
new obj.x(); // TypeError: obj.x is not a constructor
```

주의할 것은, ECMAScript 사양에서 메서드로 인정하는 범위가 일반적인 의미의 메서드보다 좁다는 것이다. 함수가 어디에 할당되어 있는지가 아니라 **함수 정의 방식**에 따라 constructor와 non-constructor를 구분한다. 위 예제의 `baz.x`는 객체의 프로퍼티에 할당되어 있지만 메서드 축약 표현으로 정의되지 않았기 때문에 constructor다.



### new 연산자

일반 함수와 생성자 함수에 특별한 형식적 차이는 없다. **new 연산자와 함께 함수를 호출하면 해당 함수는 생성자 함수로 동작**한다. 함수 객체의 내부 메서드 `[[Call]]`이 호출되는 것이 아니라 `[[Construct]]`가 호출된다. 단, new 연산자와 함께 호출하는 함수는 non-constructor가 아닌 constructor이어야 한다.

```js
// 생성자 함수로서 정의하지 않은 일반 함수
function add(x, y) {
  return x + y;
}

// 생성자 함수로서 정의하지 않은 일반 함수를 new 연산자와 함께 호출
let inst = new add();
// 함수가 객체를 반환하지 않았으므로 반환문이 무시된다. 따라서 빈 객체가 생성되어 반환된다
console.log(inst); // {}
```

반대로 new 연산자 없이 생성자 함수를 호출하면 일반 함수로 호출된다. 즉, 함수 객체의 내부 메서드 `[[Construct]]`가 아니라 `[[Call]]`이 호출된다.

```js
// 생성자 함수
function Circle(radius) {
  this.radius = radius;
  this.getDiameter = function () {
    return 2 * this.radius;
  };
}

// new 연산자 없이 생성자 함수를 호출하면 일반 함수로서 호출된다
const circle = Circle(5);
console.log(circle); // undefined

// 일반 함수 내부의 this는 전역 객체 window를 가리킨다
console.log(radius); // 5
console.log(getDiameter()); // 10
```

일반 함수와 생성자 함수에 특별한 형식적 차이는 없으므로, 생성자 함수는 일반적으로 **첫 문자를 대문자로 기술하는 파스칼 케이스**로 명명하여 일반 함수와 구별할 수 있도록 노력한다.



### new.target

생성자 함수가 new 연산자 없이 호출되는 것을 방지하기 위해 파스칼 케이스 컨벤션을 사용한다 하더라도 실수는 언제나 발생할 수 있다. 이러한 위험성을 회피하기 위해 ES6에서는 **`new.target`**을 지원한다.

new.target은 this와 유사하게 constructor인 모든 함수 내부에서 암묵적인 지역 변수와 같이 사용되며 메타 프로퍼티라고 부른다.

- 함수가 **new 연산자와 함께 생성자 함수로서 호출**되면 함수 내부의 new.target은 **함수 자신**을 가리킨다.
- 함수가 **new 연산자 없이 일반 함수로서 호출**되면 함수 내부의 new.target은 **undefined**다.

따라서 함수 내부에서 new.target을 사용하여 new 연산자와 생성자 함수로서 호출했는지 확인하여 그렇지 않은 경우 new 연산자와 함께 재귀 호출을 통해 생성자 함수로서 호출할 수 있다.

```js
function Circle(radius) {
  // 이 함수가 new 연산자와 함께 호출되지 않았다면 new.target은 undefined다
  if (!new.target) {
    // new 연산자와 함께 생성자 함수를 재귀 호출하여 생성된 인스턴스를 반환한다
    return new Circle(radius);
  }

  this.radius = radius;
  this.getDiameter = function () {
    return 2 * this.radius;
  };
}

// new 연산자 없이 생성자 함수를 호출하여도 new.target을 통해 정상적으로 인스턴스를 생성한다
const circle = Circle(5);
console.log(circle.getDiameter()); // 10
```

**스코프 세이프 생성자 패턴(scope-safe constructor)**  
new.target은 ES6에서 도입된 최신 문법으로, IE에서는 지원하지 않는다. new.target을 사용할 수 없는 상황이라면 다음과 같은 스코프 세이프 생성자 패턴을 사용할 수 있다.

```js
function Circle(radius) {
  // 생성자 함수가 new 연산자와 함께 호출되면 함수의 선두에서 빈 객체를 생성하고
  // this에 바인딩한다. 이때 this와 Circle은 프로토타입에 의해 연결된다

  // new 연산자 없이 호출되면 this는 window를 가리키고 Circle과 연결되지 않는다
  if (!(this instanceof Circle)) {
    return new Circle(radius);
  }

  this.radius = radius;
  this.getDiameter = function () {
    return 2 * this.radius;
  };
}

const circle = Circle(5);
console.log(circle.getDiameter()); // 10
```

참고로 대부분의 빌트인 생성자 함수(Object, Function, Array 등)는 new 연산자와 함께 호출되었는지를 확인한 후 적절한 값을 반환한다.

- Object와 Function 생성자 함수는 new 연산자 없이 호출해도 new 연산자와 함께 호출했을 때와 동일하게 동작한다.
- String, Number, Boolean 생성자 함수는 new 연산자와 함께 호출했을 때 객체를 생성하여 반환하지만, new 연산자 없이 호출하면 문자열, 숫자, 불리언 값을 반환한다. 이를 통해 데이터 타입을 변환하기도 한다.

```js
let str = String(123);
console.log(str, typeof str); // 123 string

let num = Number('123');
console.log(num, typeof num); // 123 number

let bool = Boolean('true');
console.log(bool, typeof bool); // true boolean
```

<details class="orca-details">
<summary>new.target과 instanceof를 이용한 방어 코드, 요즘도 직접 짜야 할까?</summary>

일반적인 애플리케이션 코드에서는 거의 짤 일이 없다. ES6의 **class 문법을 쓰면 new 없이 호출하는 순간 자바스크립트 엔진이 알아서 TypeError를 던져주기 때문**이다. 즉 `class Circle {}`을 그냥 `Circle()`로 부르면 "Class constructor Circle cannot be invoked without 'new'"라는 에러가 나면서 실수를 원천 차단해준다. 그래서 new.target으로 직접 방어하는 패턴은 주로 라이브러리를 만들거나, class를 쓰지 못하는 레거시 환경을 지원해야 하거나, 팩토리 함수처럼 new 있이도 없이도 동작하게 만들고 싶을 때 쓴다. 결론적으로 개념은 알아두되, 실무 신규 코드에서는 class를 쓰는 것이 가장 안전하고 간결하다.

</details>

# 18장. 함수와 일급 객체

## 일급 객체

다음과 같은 조건을 만족하는 객체를 **일급 객체(first-class object)**라 한다.

1. 무명의 리터럴로 생성할 수 있다. 즉, 런타임에 생성이 가능하다.
2. 변수나 자료구조(객체, 배열 등)에 저장할 수 있다.
3. 함수의 매개변수에 전달할 수 있다.
4. 함수의 반환값으로 사용할 수 있다.

**자바스크립트의 함수는 위의 조건을 모두 만족하므로 일급 객체다.**

```js
// 1. 함수는 무명의 리터럴로 생성할 수 있다
// 2. 함수는 변수에 저장할 수 있다
// 런타임(할당 단계)에 함수 리터럴이 평가되어 함수 객체가 생성되고 변수에 할당된다
const increase = function (num) {
  return ++num;
};

const decrease = function (num) {
  return --num;
};

// 2. 함수는 객체에 저장할 수 있다
const auxs = { increase, decrease };

// 3. 함수의 매개변수에 전달할 수 있다
// 4. 함수의 반환값으로 사용할 수 있다
function makeCounter(aux) {
  let num = 0;

  return function () {
    num = aux(num);
    return num;
  };
}

// 3. 함수는 매개변수에게 함수를 전달할 수 있다
const increaser = makeCounter(auxs.increase);
console.log(increaser()); // 1
console.log(increaser()); // 2

const decreaser = makeCounter(auxs.decrease);
console.log(decreaser()); // -1
console.log(decreaser()); // -2
```

함수가 일급 객체라는 것은 **함수를 객체와 동일하게 사용할 수 있다**는 의미다. 객체는 값이므로 함수는 값과 동일하게 취급할 수 있다.  
=&gt; 따라서 함수는 값을 사용할 수 있는 곳(변수 할당문, 객체의 프로퍼티 값, 배열의 요소, 함수 호출의 인수, 함수 반환문)이라면 어디서든지 리터럴로 정의할 수 있으며 런타임에 함수 객체로 평가된다.

일급 객체로서 함수가 가지는 가장 큰 특징은 다음과 같다.

- 일반 객체와 같이 함수의 **매개변수에 전달**할 수 있다.
- 함수의 **반환값**으로 사용할 수 있다.

=&gt; 이는 함수형 프로그래밍을 가능하게 하는 자바스크립트의 장점 중 하나다.

함수는 객체이지만 일반 객체와는 차이가 있다. **일반 객체는 호출할 수 없지만 함수 객체는 호출할 수 있다.** 그리고 함수 객체는 일반 객체에는 없는 함수 고유의 프로퍼티를 소유한다.

<details class="orca-details">
<summary>함수가 일급 객체라는 사실이 실제로 어떤 코드를 가능하게 할까?</summary>

우리가 매일 쓰는 고차 함수들이 전부 여기서 나온다. `map`, `filter`, `reduce`, `forEach`처럼 함수를 인수로 받는 배열 메서드, `setTimeout(콜백, 1000)`처럼 콜백을 넘기는 비동기 처리, 이벤트 핸들러 등록(`addEventListener`), React의 컴포넌트(함수 컴포넌트 자체가 함수다)와 커스텀 훅이 반환하는 함수 등이 모두 "함수를 값처럼 주고받을 수 있다"는 전제 위에 있다. 만약 함수가 일급 객체가 아니었다면 함수를 변수에 담거나 다른 함수에 넘길 수 없어서, 우리가 아는 함수형 스타일의 코드는 대부분 불가능했을 것이다. 즉 일급 객체라는 개념은 단순한 이론이 아니라 자바스크립트 코드의 표현력을 떠받치는 근본 토대다.

</details>



## 함수 객체의 프로퍼티

함수는 객체다. 따라서 함수도 프로퍼티를 가질 수 있다. 브라우저 콘솔에서 `console.dir` 메서드를 사용하여 함수 객체 내부를 살펴보면, 일반 객체에는 없는 함수 객체 고유의 프로퍼티를 확인할 수 있다.

```js
function square(number) {
  return number * number;
}

console.dir(square);
```

`arguments`, `caller`, `length`, `name`, `prototype` 프로퍼티는 모두 함수 객체의 데이터 프로퍼티다. 이것들은 일반 객체에는 없는 함수 객체 고유의 프로퍼티다.  
=&gt; 단, `__proto__`는 접근자 프로퍼티이며 함수 객체 고유의 프로퍼티가 아니라 Object.prototype 객체의 프로퍼티를 상속받은 것이다.

```js
function square(number) {
  return number * number;
}

console.log(Object.getOwnPropertyDescriptors(square));
/*
{
  length:    { value: 1, writable: false, enumerable: false, configurable: true },
  name:      { value: "square", writable: false, enumerable: false, configurable: true },
  arguments: { value: null, writable: false, enumerable: false, configurable: false },
  caller:    { value: null, writable: false, enumerable: false, configurable: false },
  prototype: { value: {...}, writable: true, enumerable: false, configurable: false }
}
*/

// __proto__는 square 함수의 자체 프로퍼티가 아니라 Object.prototype의 프로퍼티를 상속받은 것이다
console.log(Object.getOwnPropertyDescriptor(square, '__proto__')); // undefined
console.log(Object.getOwnPropertyDescriptor(Object.prototype, '__proto__'));
// { get: f, set: f, enumerable: false, configurable: true }
```



### arguments 프로퍼티

함수 객체의 `arguments` 프로퍼티 값은 arguments 객체다. **arguments 객체는 함수 호출 시 전달된 인수(argument)들의 정보를 담고 있는 순회 가능한(iterable) 유사 배열 객체**이며, 함수 내부에서 지역 변수처럼 사용된다.  
=&gt; 즉, 함수 외부에서는 참조할 수 없다.

자바스크립트는 함수의 매개변수와 인수의 개수가 일치하는지 확인하지 않는다. 따라서 함수 호출 시 매개변수 개수만큼 인수를 전달하지 않아도 에러가 발생하지 않는다.

```js
function multiply(x, y) {
  console.log(arguments);
  return x * y;
}

console.log(multiply());        // NaN
console.log(multiply(1));       // NaN
console.log(multiply(1, 2));    // 2
console.log(multiply(1, 2, 3)); // 2
```

- **인수가 부족한 경우** : 매개변수는 undefined로 초기화된 상태를 유지한다.
- **인수가 초과된 경우** : 초과된 인수는 무시된다. 하지만 그냥 버려지는 것은 아니고, 모든 인수는 암묵적으로 **arguments 객체의 프로퍼티로 보관**된다.

arguments 객체는 인수를 프로퍼티 값으로 소유하며 프로퍼티 키는 인수의 순서를 나타낸다. arguments 객체의 `callee` 프로퍼티는 호출되어 arguments 객체를 생성한 함수, 즉 함수 자신을 가리키고 `length` 프로퍼티는 인수의 개수를 가리킨다.

arguments 객체는 **매개변수 개수를 확정할 수 없는 가변 인자 함수를 구현**할 때 유용하다.

```js
function sum() {
  let res = 0;

  // arguments 객체는 length 프로퍼티가 있는 유사 배열 객체이므로 for 문으로 순회할 수 있다
  for (let i = 0; i < arguments.length; i++) {
    res += arguments[i];
  }

  return res;
}

console.log(sum());        // 0
console.log(sum(1, 2));    // 3
console.log(sum(1, 2, 3)); // 6
```

> `유사 배열 객체(array-like object)` length 프로퍼티를 가진 객체로 for 문으로 순회할 수 있는 객체. 하지만 배열은 아니므로 배열 메서드를 사용하면 에러가 발생한다.

유사 배열 객체는 배열이 아니므로 배열 메서드를 사용할 경우 에러가 발생한다. 따라서 배열 메서드를 사용하려면 `Function.prototype.call`, `Function.prototype.apply`를 사용해 간접 호출해야 하는 번거로움이 있다.  
=&gt; 이러한 번거로움을 해결하기 위해 ES6에서는 **Rest 파라미터**를 도입했다.

```js
// ES6 Rest 파라미터
function sum(...args) {
  // Rest 파라미터 args는 순수한 배열이다
  return args.reduce((pre, cur) => pre + cur, 0);
}

console.log(sum(1, 2));       // 3
console.log(sum(1, 2, 3, 4)); // 10
```



<details class="orca-details">
<summary>rest 파라미터가 있는데 arguments를 여전히 알아야 할까?</summary>

새 코드에서 쓸 일은 거의 없지만, **레거시 코드를 읽고 고치기 위해** 필요하고 하네욥......... 특히 다음 두 가지 함정은 알아 둘 가치가 있다.

첫째, **화살표 함수에는** `arguments`**가 없다.** 상위 스코프의 것을 참조하므로, 일반 함수를 화살표 함수로 바꾸는 리팩터링에서 조용히 깨진다.

둘째, `arguments`는 배열이 아닌 **유사 배열 객체**라서 `map`, `filter`가 없다. 변환이 필요하다.

```js
const args = Array.from(arguments);   // 또는 [...arguments]
```

rest 파라미터(`...args`)는 처음부터 진짜 배열이고, 어떤 매개변수가 가변인지 시그니처에 드러나며, 화살표 함수에서도 동작한다. 대체하지 않을 이유가 없다.

</details>

<details class="orca-details">
<summary>sloppy mode에서 arguments가 매개변수와 연동된다는 게 무슨 말일까?</summary>

sloppy mode의 `arguments`는 매개변수와 **양방향으로 값이 연결(mapped)**되어 있다. 한쪽을 바꾸면 다른 쪽도 바뀐다.

```js
function f(a) {
  a = 2;
  console.log(arguments[0]); // sloppy: 2 / strict: 1
}
f(1);
```

strict mode에서는 이 연동이 끊긴다(unmapped). 또한 **기본값·rest·구조 분해 매개변수가 하나라도 있으면 strict가 아니어도 unmapped**가 된다. 스펙이 이렇게 복잡해진 건 기존 웹 호환성 때문인데, 결과적으로 "같은 코드가 모드에 따라 다르게 동작한다"는 최악의 상황이 만들어졌다. `arguments`를 피해야 할 또 하나의 이유다.

</details>

<details class="orca-details">
<summary>arguments.callee는 왜 금지됐을까?</summary>

익명 함수가 자기 자신을 재귀 호출하기 위해 쓰이던 기능이지만, strict mode에서 접근하면 TypeError가 난다. 이유는 두 가지다. **보안** 측면에서는 호출자 정보를 타고 올라가 스택을 들여다볼 수 있어 정보 유출 경로가 되고, **성능** 측면에서는 인라이닝과 꼬리 호출 최적화를 방해한다.

대안은 간단하다. **기명 함수 표현식**을 쓰면 된다.

```js
const factorial = function fact(n) {
  return n <= 1 ? 1 : n * fact(n - 1);   // fact는 함수 내부에서만 보이는 이름
};
```

이때 `fact`라는 이름은 함수 자신의 스코프 안에서만 유효하고 외부에서는 보이지 않는다는 점도 함께 알아두면 좋다.

</details>



### caller 프로퍼티

`caller` 프로퍼티는 ECMAScript 사양에 포함되지 않은 **비표준 프로퍼티**다. 이후 표준화될 예정도 없는 프로퍼티이므로 사용하지 말고 참고로만 알아두자. caller 프로퍼티는 함수 자신을 호출한 함수를 가리킨다.

```js
function foo(func) {
  return func();
}

function bar() {
  return 'caller : ' + bar.caller;
}

// 브라우저에서의 실행 결과
console.log(foo(bar)); // caller : function foo(func) {...}
console.log(bar());    // caller : null
```



<details class="orca-details">
<summary>얘 비표준이라는데 그렇게 중요한가?... 책에 왜 실렸지</summary>

**"왜 이런 기능이 제거되는가"를 보여주는 사례**라서다. `func.caller`는 자신을 호출한 함수를 참조하는데, 이를 반복하면 호출 스택 전체를 코드로 순회할 수 있다. 샌드박스 안의 코드가 자신을 호출한 바깥 코드의 함수 객체와 그 클로저에 접근할 수 있다는 뜻이므로, **보안 경계를 무너뜨리는 기능**이다.

그래서 strict mode 함수에서는 `caller`와 `arguments` 프로퍼티 접근이 TypeError를 던지도록 막혀 있다(과거에는 이를 "poisoned pill"이라 불렀다). 호출 스택 정보가 필요하다면 `new Error().stack`처럼 **문자열로만 제공되는 디버깅용 수단**을 쓰는 것이 현재의 방향이다.

</details>



### length 프로퍼티

함수 객체의 `length` 프로퍼티는 **함수를 정의할 때 선언한 매개변수의 개수**를 가리킨다.

```js
function foo() {}
console.log(foo.length); // 0

function bar(x) {
  return x;
}
console.log(bar.length); // 1

function baz(x, y) {
  return x * y;
}
console.log(baz.length); // 2
```

주의할 것은 **arguments 객체의 length 프로퍼티와 함수 객체의 length 프로퍼티의 값은 다를 수 있다**는 점이다. arguments 객체의 length 프로퍼티는 인자(argument)의 개수를 가리키고, 함수 객체의 length 프로퍼티는 매개변수(parameter)의 개수를 가리킨다.



### name 프로퍼티

함수 객체의 `name` 프로퍼티는 함수 이름을 나타낸다. name 프로퍼티는 ES6에서 정식 표준이 되었다.

name 프로퍼티는 ES5와 ES6에서 동작을 달리하므로 주의해야 한다. **익명 함수 표현식**의 경우 ES5에서 name 프로퍼티는 빈 문자열을 값으로 갖는다. 하지만 **ES6에서는 함수 객체를 가리키는 식별자(변수 이름)를 값으로 갖는다.**

```js
// 기명 함수 표현식
const namedFunc = function foo() {};
console.log(namedFunc.name); // foo

// 익명 함수 표현식
const anonymousFunc = function () {};
// ES5: name 프로퍼티는 빈 문자열을 값으로 갖는다
// ES6: name 프로퍼티는 함수 객체를 가리키는 변수 이름을 값으로 갖는다
console.log(anonymousFunc.name); // anonymousFunc

// 함수 선언문(Function declaration)
function bar() {}
console.log(bar.name); // bar
```

참고로 함수 이름과 함수 객체를 가리키는 식별자는 의미가 다르다는 점에 유의해야 한다. 함수를 호출할 때는 함수 이름이 아니라 함수 객체를 가리키는 식별자로 호출한다.



<details class="orca-details">
<summary>name은 읽기 전용인데 왜 바꿀 수 있을까?</summary>

`name`의 어트리뷰트는 `writable: false`지만 **`configurable: true`**다. 즉 할당은 막혀 있어도 `Object.defineProperty`로 재정의하는 것은 가능하다.

```js
const f = () => {};
f.name = 'x';                                  // 무시됨 (strict에서는 TypeError)
Object.defineProperty(f, 'name', { value: 'x' });
f.name; // 'x'
```

라이브러리가 함수를 래핑하면서 원본 이름을 유지하기 위해 실제로 쓰는 기법이다. `length`도 마찬가지 어트리뷰트를 갖는다.

</details>



### `__proto__` 접근자 프로퍼티

모든 객체는 `[[Prototype]]`이라는 내부 슬롯을 갖는다. `[[Prototype]]` 내부 슬롯은 객체지향 프로그래밍의 상속을 구현하는 프로토타입 객체를 가리킨다.

`__proto__` 프로퍼티는 `[[Prototype]]` 내부 슬롯이 가리키는 프로토타입 객체에 접근하기 위해 사용하는 **접근자 프로퍼티**다. 내부 슬롯에는 직접 접근할 수 없고 간접적인 접근 방법을 제공하는 경우에 한하여 접근할 수 있는데, `[[Prototype]]` 내부 슬롯에도 직접 접근할 수 없으며 `__proto__` 접근자 프로퍼티를 통해 간접적으로 프로토타입 객체에 접근할 수 있다.

```js
const obj = { a: 1 };

// 객체 리터럴 방식으로 생성한 객체의 프로토타입 객체는 Object.prototype이다
console.log(obj.__proto__ === Object.prototype); // true

// 객체 리터럴 방식으로 생성한 객체는 프로토타입 객체인 Object.prototype의 프로퍼티를 상속받는다
// hasOwnProperty 메서드는 Object.prototype의 메서드다
console.log(obj.hasOwnProperty('a'));         // true
console.log(obj.hasOwnProperty('__proto__')); // false
```

> `hasOwnProperty` 인수로 전달받은 프로퍼티 키가 객체 고유의 프로퍼티 키인 경우에만 true를 반환하고 상속받은 프로토타입의 프로퍼티 키인 경우 false를 반환한다.



<details class="orca-details">
<summary>**proto** 대신 getPrototypeOf를 쓰라는 이유는?</summary>

`__proto__`는 표준 본문이 아니라 **기존 브라우저 호환성을 위해 Annex B에 규정된 레거시 기능**이다. 게다가 프로퍼티처럼 생겼지만 실제로는 `Object.prototype`에 정의된 접근자이므로, `Object.create(null)`로 만든 객체에는 존재하지 않는다.

```js
Object.getPrototypeOf(obj);
Object.setPrototypeOf(obj, proto);
```

특히 `setPrototypeOf`(그리고 `__proto__` 할당)는 **엔진의 히든 클래스 최적화를 무효화**해서 성능을 크게 떨어뜨린다. 프로토타입은 객체를 만들 때 `Object.create`나 클래스로 한 번에 결정하고, 이후에는 바꾸지 않는 것이 원칙이다.

</details>



### prototype 프로퍼티

`prototype` 프로퍼티는 생성자 함수로 호출할 수 있는 함수 객체, 즉 **constructor만이 소유하는 프로퍼티**다. 일반 객체와 생성자 함수로 호출할 수 없는 non-constructor에는 prototype 프로퍼티가 없다.

```js
// 함수 객체는 prototype 프로퍼티를 소유한다
console.log((function () {}).hasOwnProperty('prototype')); // true

// 일반 객체는 prototype 프로퍼티를 소유하지 않는다
console.log(({}).hasOwnProperty('prototype')); // false
```

prototype 프로퍼티는 함수가 객체를 생성하는 생성자 함수로 호출될 때 **생성자 함수가 생성할 인스턴스의 프로토타입 객체**를 가리킨다.

<details class="orca-details">
<summary>**proto** 프로퍼티와 prototype 프로퍼티는 이름이 비슷한데 뭐가 다를까?</summary>

헷갈리기 딱 좋은 한 쌍인데, "누가 갖고 있느냐"로 구분하면 명확하다. `**prototype` 프로퍼티는 오직 함수(그중에서도 constructor)만 갖는다.** 이건 "내가 new로 인스턴스를 만들면, 그 인스턴스에게 물려줄 부모 객체는 바로 이거야"라고 미리 준비해둔 청사진 같은 것이다. 반면 `**__proto__`는 모든 객체가 갖는다.** 이건 "나(이 객체)의 실제 부모는 누구인가"를 가리키는, 이미 연결이 끝난 링크다. 그래서 `new Circle()`로 인스턴스를 만들면 `Circle.prototype === circle.__proto__` 관계가 성립한다. 즉 생성자 쪽에서 내려다보는 방향이 `prototype`, 인스턴스 쪽에서 올려다보는 방향이 `__proto__`라고 이해하면 된다. 참고로 `__proto__` 직접 사용은 권장되지 않고, 실무에서는 `Object.getPrototypeOf` / `Object.setPrototypeOf`를 쓴다. 프로토타입은 바로 다음 19장에서 자세히 다룬다.

</details>

<details class="orca-details">
<summary>화살표 함수에 prototype이 없다는 사실이 왜 중요할까?</summary>

`prototype`이 없다는 건 곧 **인스턴스를 만들 수 없다**는 뜻이므로, `new`로 호출할 수 없다는 17장의 내용과 정확히 이어진다. ES6 메서드 축약 표현도 마찬가지다.

```js
(() => {}).prototype;          // undefined
({ foo() {} }).foo.prototype;  // undefined
(function () {}).prototype;    // {constructor: f}
```

실무에서 이게 문제가 되는 대표적인 경우는 **클래스 메서드를 화살표 함수 필드로 정의할 때**다. 화살표 함수 필드는 `prototype`이 아니라 **각 인스턴스에 개별 생성**되므로 메모리를 더 쓰고, 프로토타입 체인에 없어서 서브클래스에서 `super.method()`로 호출할 수도 없다. 대신 `this` 바인딩이 자동으로 유지된다는 장점이 있어 트레이드오프를 알고 선택해야 한다.

```js
class A {
  handleClick = () => {};   // 인스턴스마다 생성, this 자동 바인딩
  handleClick2() {}         // 프로토타입에 한 번만, this 바인딩 필요
}
```

</details>
