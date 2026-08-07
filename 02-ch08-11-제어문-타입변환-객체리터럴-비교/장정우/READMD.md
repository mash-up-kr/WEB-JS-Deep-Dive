# 모던 자바스크립트 Deep Dive 8~11장 정리

## 08장. 제어문

제어문(control flow statement)은 조건에 따라 코드 블록을 실행(조건문)하거나 반복 실행(반복문)할 때 사용한다.
제어문은 코드의 흐름을 이해하는데 어렵게 해, 가독성을 해치는 단점이 있다.

(이터러블 고차 함수에 대해서는 34장에서..)

### 블록문

0개 이상의 문을 중괄호로 묶은 것으로, 코드 블록 또는 블록이라고 부르기도 한다.
자바스크립트는 블록문을 하나의 실행 단위로 취급한다.

```js
// 블록문
{
  var foo = 10;
}

// 제어문
var x = 1;
if (x < 10) {
  x++;
}

// 함수 선언문
function sum(a, b) {
  return a + b;
}
```

### 조건문

주어진 조건식의 평가 결과에 따라 코드 블록의 실행을 결정한다.
js는 `if...else`문과 `switch`문 2가지 조건문을 제공한다.

```js
if (조건식1) {
  // 조건식1이 참이면 이 코드 블록이 실행된다.
} else if (조건식2) {
  // 조건식2가 참이면 이 코드 블록이 실행된다.
} else {
  // 조건식1과 조건식2가 모두 거짓이면 이 코드 블록이 실행된다.
}
```

```js
switch (표현식) {
  case 표현식1:
    // switch 문의 표현식과 표현식1이 일치하면 실행될 문
    break;

  case 표현식2:
    // switch 문의 표현식과 표현식2가 일치하면 실행될 문
    break;

  default:
    // 일치하는 case 문이 없을 때 실행될 문
}
```

#### Early return

함수의 앞부분에서 예외적인 조건을 먼저 걸러내고 즉시 return으로 빠져나가는 패턴. 
이때 앞에 두는 조건 검사를 **guard clause(보호 절)** 라고 한다.


```jsx
// 요런 걸
function UserProfile({ userId }) {
  const { data, isLoading, isError, error } = useQuery(['user', userId], fetchUser);

  return (
    <div>
      {isLoading ? (
        <Spinner />
      ) : isError ? (
        <ErrorMessage error={error} />
      ) : (
        <>
          <Avatar src={data.avatarUrl} />
          <h1>{data.name}</h1>
          <p>{data.bio}</p>
        </>
      )}
    </div>
  );
}

// 요렇게
function UserProfile({ userId }) {
  const { data, isLoading, isError, error } = useQuery(['user', userId], fetchUser);

  if (isLoading) return <Spinner />;
  if (isError) return <ErrorMessage error={error} />;

  return (
    <div>
      <Avatar src={data.avatarUrl} />
      <h1>{data.name}</h1>
      <p>{data.bio}</p>
    </div>
  );
}
```

Early return이 주는 장점
1. 타입 내로잉
   - TypeScript + React Query 조합에서 isLoading과 isError를 return으로 걸러내면, 그 아래에선 data의 타입을 `User | undefined`가 아니라 `User`로 좁힐 수 있다.
   - 데이터가 빈 상황에 대한 방지
2. 각 상태의 UI를 독립적으로 관리할 수 있다.
   - 위 코드에서 `isLoading` 상태의 `<Spinner />`의 UI를 바꾼다고 해서 다른 부분에 영향을 잘 끼치지 않음.
3. 가독성 향상.

다만, Early return도 방식 중 하나일 뿐 항상 좋은 건 아님.
1. 남용하면 조건 나열이 되어버림
   - 뭐.. 물론 이렇게 짜는 사람은 없겠지만
   - ```jsx
     function OrderDetail({ order }) {
        if (!order) return <Empty />;
        if (order.status === 'draft') return <Empty />;
        if (order.status === 'deleted') return <Empty />;
        if (order.status === 'expired') return <Empty />;
        if (order.items.length === 0) return <Empty />;
        
        if (order.status === 'canceled') return <CanceledNotice order={order} />;
        if (order.status === 'refunded') return <CanceledNotice order={order} />;
        if (order.status === 'refund_pending') return <CanceledNotice order={order} />;
        
        return <OrderContent order={order} />;
     }
          
     // 이런 경우엔 조건을 의미 단위로 묶어서 처리해준다.
     const HIDDEN_STATUSES = new Set(['draft', 'deleted', 'expired']);
       const CANCELED_STATUSES = new Set(['canceled', 'refunded', 'refund_pending']);
        
       function OrderDetail({ order }) {
       if (!order || HIDDEN_STATUSES.has(order.status) || order.items.length === 0) {
        return <Empty />;
       }
       if (CANCELED_STATUSES.has(order.status)) {
         return <CanceledNotice order={order} />;
       }
       return <OrderContent order={order} />;
     }
      ```
2. guard 안에 비즈니스 로직을 섞지 않도록
```js
getTotal() {
   if (this.discount === null) return total; // 처음엔 무해한 null guard였는데
   ...
}
```

여기까지는 평범한 guard처럼 보이는데, 요구사항이 "할인 정책이 없으면 1,000원 할인"으로 바뀌는 순간:

```js
if (this.discount === null) return total - 1000; // guard가 비즈니스 로직이 됨
```

#### fall-through

switch문에서 조건과 일치하는 case를 실행한 뒤 탈출하지 않고, 이후의 case와 default까지 연이어 실행하는 현상을 말한다.

```js
// 월을 영어로 변환한다. (11 → 'November')
var month = 11;
var monthName;

switch (month) {
 case 1:
   monthName = 'January';
 case 2:
   monthName = 'February';
 case 3:
   monthName = 'March';
 case 4:
   monthName = 'April';
 case 5:
   monthName = 'May';
 case 6:
   monthName = 'June';
 case 7:
   monthName = 'July';
 case 8:
   monthName = 'August';
 case 9:
   monthName = 'September';
 case 10:
   monthName = 'October';
 case 11:
   monthName = 'November';
 case 12:
   monthName = 'December';
 default:
   monthName = 'Invalid month';
}

console.log(monthName);
```

책에 나온 예제 외에도 이런식으로도 활용 가능하다.

```js
// 같은 실행을 하는 것 끼리 의도 적으로 묶는다.
  switch (value) {
   case 'A':
   case 'B':
      doSomething();
      break;
}
```

간단한 switch case 문의 경우엔 이런 활용을 하는 것도 좋지만, fall-through가 많아 진다면 그 흐름을 파악하는 것 또한 고역이기에 개인적으로는 잘 활용하지 않는 것 같다.
또한 의도적인 fall-through인지 실수인지 알기 어렵다.

#### 조건문에서 hook을 쓰면 안되는 이유

- 훅이 최상위가 아닌 제어문으로 감싸지게 된다면, 렌더링마다 훅의 호출 순서나 개수가 달라져 상태와 이펙트가 잘못 연결될 수 있다. 따라서 훅은 컴포넌트나 커스텀 훅의 최상위에서 항상 같은 순서로 호출되어야 한다.
- React는 훅의 호출 순서를 상태의 식별자로 사용하기 때문.

### 반복문

> 그냥 반복문.

## 09장. 타입 변환과 단축 평가

> 자세한 내용은 책에. 이건 뭐 인간 인터프리터 되기 급의 암기

### 타입 변환

타입 변환에는 2가지가 있다.
- 명시적 타입 변환 (타입 캐스팅)
- 암묵적 타입 변환 (타입 강제 변환)


타입 변환은 기존의 원시 값을 사용해 다른 타입의 새로운 원시 값을 생성하는 것이다.


js는 가급적 에러를 발생 시키지 않도록 암묵적 타입 변환을 통해 표현식을 평가한다.
- 암묵적 타입 변환이 발생하면 문자열, 숫자, 불리언 같은 원시 타입 중 하나로 자동 변환한다.


단축 평가에서는 피연산자를 불리언 값으로 평가해 실행 흐름을 결정하지만, 결과는 불리언으로 변환하지 않고 결과를 결정한 피연산자 자체를 반환한다.
```js
  Boolean('Cat'); // true: 명시적으로 불리언 반환
  !'Cat';         // false: 불리언 반환
  'Cat' && 'Dog'; // 'Dog': 피연산자 반환
  'Cat' || 'Dog'; // 'Cat': 피연산자 반환
   0 && '실행';    // 0
   '' || '기본값'; // '기본값'
```
| ![short-circuit evaluation.png](images/short-circuit%20evaluation.png) |
|:----------------------------------------------------------------------:|
|                               단축 평가 조건표                                |

## 10장. 객체 리터럴

### 객체 

⭐️자바스크립트를 구선하는 거의 "모든 것"이 객체다. (원시 값 제외)

<details>
<summary>JS로 객체를 생성하는 방법</summary>

### JS로 객체를 생성하는 방법

#### 1. 객체 리터럴

가장 간단하고 자주 사용하는 방법이다.

```js
const user = {
  name: 'Lee',
  age: 20,
  sayHello() {
    console.log(`안녕하세요. ${this.name}입니다.`);
  }
};

user.sayHello(); // 안녕하세요. Lee입니다.
```

#### 2. `Object` 생성자 함수

`new Object()`로 빈 객체를 만든 뒤 프로퍼티를 추가한다.

```js
const user = new Object();

user.name = 'Kim';
user.age = 30;

console.log(user); // { name: 'Kim', age: 30 }
```

일반적으로는 객체 리터럴을 더 많이 사용한다.

#### 3. 생성자 함수

같은 구조의 객체를 여러 개 만들 때 사용한다.

```js
function User(name, age) {
  this.name = name;
  this.age = age;
}

const user1 = new User('Lee', 20);
const user2 = new User('Kim', 30);
```

`new`를 사용하면 `User` 함수가 생성자 함수로 동작한다.

#### 4. `Object.create` 메서드

특정 객체를 프로토타입으로 지정해 새로운 객체를 만든다.

```js
const userPrototype = {
  sayHello() {
    console.log(`안녕하세요. ${this.name}입니다.`);
  }
};

const user = Object.create(userPrototype);
user.name = 'Park';

user.sayHello(); // 안녕하세요. Park입니다.
```

#### 5. 클래스

ES6에서 도입된 문법으로, 생성자 함수와 프로토타입 기반 객체 생성을 읽기 쉽게 표현한다.

```js
class User {
  constructor(name, age) {
    this.name = name;
    this.age = age;
  }

  sayHello() {
    console.log(`안녕하세요. ${this.name}입니다.`);
  }
}

const user = new User('Choi', 25);
user.sayHello(); // 안녕하세요. Choi입니다.
```

</details>

## 11장. 원시 값과 객체의 비교

원시 타입 
- String: `'hello'`
- Number: `42`, `3.14`
- Boolean: `true`, `false`
- `undefined`: 값이 할당되지 않은 상태
- `null`: 의도적으로 값이 없음을 나타냄
- Symbol: `Symbol('id')`
- BigInt: `123n`

객체 타입
- Object: `{ name: 'Lee' }`
- Array: `[1, 2, 3]`
- Function: `function () {}`
- RegExp: `/js/`

원시 타입과 객체 타입은 크게 세 가지 측면에서 다르다.
- 원시 타입의 값은 **변경 불가능한 값**이다.
```js
  // immutability와 재할당을 잘 분리해서 생각하자.
  let text = 'cat';
  text[0] = 'C';
  console.log(text); // 'cat'
```
- 이에 비해 객체 타입의 값은 **변경 가능한 값**이다.
```js
const user = { name: 'Lee' };
user.name = 'Kim';
user = {}; // 변수 재할당은 불가능
```

### Call by value와 Call by sharing

> 객체를 함수에 전달하면 객체 자체가 복사되는 것이 아니라, 같은 객체를 가리키는 참조값이 복사된다.  
> 따라서 함수 내부에서 객체를 변경하면 원본에도 영향을 주지만, 매개변수 자체를 재할당해도 원래 변수는 바뀌지 않는다.

자바스크립트는 함수의 인자를 값으로 전달한다.

```js
let number = 10;

function changeValue(value) {
  value = 20;
}

changeValue(number);
console.log(number); // 10
```

객체를 전달할 때는 객체 자체가 복사되는 것이 아니라 객체를 가리키는 참조값이 복사된다.

```js
const user = { name: 'Lee' };

function changeName(user) {
  user.name = 'Kim';
}

changeName(user);
console.log(user.name); // 'Kim'
```

따라서 객체 내부는 변경할 수 있지만, 함수 안에서 매개변수 자체를 다른 객체로 재할당해도 원래 변수는 바뀌지 않는다.

```js
const user = { name: 'Kim' };

function replaceUser(user) {
  user = { name: 'Park' };
}

replaceUser(user);
console.log(user.name); // 'Kim'
```

이를 `Pass by sharing`이라고 설명하기도 한다.

### 얕은 복사와 깊은 복사

#### 얕은 복사

객체의 최상위 프로퍼티만 복사한다. 중첩된 객체는 원본과 공유한다.

```js
const original = {
  name: 'Lee',
  address: { city: 'Seoul' }
};

const copied = { ...original };
copied.address.city = 'Busan';

console.log(original.address.city); // 'Busan'
```

#### 깊은 복사

중첩된 객체까지 별도로 복사한다.

```js
const copied = structuredClone(original);

copied.address.city = 'Daegu';

console.log(original.address.city); // 'Busan'
```

### `Object.freeze`와 `Object.seal`

두 메서드 모두 객체의 변경을 제한하지만, 제한 범위가 다르다.

#### `Object.seal`

기존 프로퍼티의 추가와 삭제를 막는다. 하지만 기존 프로퍼티의 값은 변경할 수 있다.

```js
const user = Object.seal({
  name: 'Lee'
});

user.name = 'Kim'; // 가능
user.age = 20;     // 추가되지 않음
delete user.name;  // 삭제되지 않음

```

#### `Object.freeze`

프로퍼티의 추가·삭제·재할당을 모두 막는다.

```js
const user = Object.freeze({
  name: 'Lee'
});

user.name = 'Kim'; // 변경되지 않음
user.age = 20;     // 추가되지 않음
delete user.name;  // 삭제되지 않음

// 주로 상수값에 대한 변경을 외부에서 허락하지 않을때, source of truth 같은 걸 지향할때 사용한듯
export const ERROR = Object.freeze({
   EMPTY: '값을 입력해주세요.',
   VALID: '유효한 값을 입력해주세요.',
   OVER_MAX_LENGTH: '자동차 이름은 5자 이하로 입력해주세요.',
   OVER_MIN_COUNT: '시도 횟수는 1회 이상으로 입력해주세요.',
});
```

#### 프로퍼티 속성 차이

- `writable`: 프로퍼티 값의 재할당 가능 여부
- `configurable`: 프로퍼티 삭제 및 속성 변경 가능 여부

```js
const sealed = Object.seal({ name: 'Lee' });
const frozen = Object.freeze({ name: 'Lee' });

Object.getOwnPropertyDescriptor(sealed, 'name');
// { value: 'Lee', writable: true, configurable: false, enumerable: true }

Object.getOwnPropertyDescriptor(frozen, 'name');
// { value: 'Lee', writable: false, configurable: false, enumerable: true }
```

둘 다 중첩 객체까지 자동으로 처리하지 않는 얕은 동작이다.

### 복사와 객체 변경 제한의 관계

얕은 복사와 깊은 복사는 객체를 얼마나 분리해서 복사하는지에 대한 개념이고,
`Object.seal`과 `Object.freeze`는 객체의 변경을 어디까지 제한할지에 대한 기능이다.

```js
const original = {
  name: 'Lee',
  address: { city: 'Seoul' }
};

const shallow = { ...original };
const deep = structuredClone(original);

shallow.address.city = 'Busan';

console.log(original.address.city); // 'Busan'
console.log(deep.address.city);     // 'Seoul'
```

얕은 복사는 중첩 객체를 공유하기 때문에 복사본을 변경하면 원본에도 영향을 준다.
깊은 복사는 중첩 객체까지 별도로 만들기 때문에 서로 독립적이다.

```js
const shallow = Object.freeze({ ...original });
shallow.name = 'Kim';           // 변경되지 않음
shallow.address.city = 'Daegu'; // 중첩 객체는 변경 가능

const deep = Object.freeze(structuredClone(original));
deep.name = 'Park';             // 변경되지 않음
```

하지만 `Object.freeze`도 얕게 동작하므로, 중첩 객체까지 보호하려면 별도의 deep freeze 처리가 필요하다.

- `Object.isSealed(value)`: 객체가 `seal` 상태인지 확인
- `Object.isFrozen(value)`: 객체가 `freeze` 상태인지 확인

```js
const user = Object.freeze({ name: 'Lee' });

Object.isSealed(user); // true
Object.isFrozen(user); // true
```

`freeze`된 객체는 `seal`의 조건도 만족하므로 두 결과가 모두 `true`가 된다.

### 라이브러리를 이용한 복사

`es-toolkit`은 얕은 복사와 깊은 복사를 함수로 제공한다.

```ts
import { clone, cloneDeep } from 'es-toolkit';

const shallow = clone(original);
const deep = cloneDeep(original);
```

- `clone`: 얕은 복사
- `cloneDeep`: 깊은 복사

정리하면, 얕은 복사는 중첩 객체를 공유하고, 깊은 복사는 중첩 객체까지 복사한다.  
`Object.freeze`는 최상위 객체의 변경만 막는다.
