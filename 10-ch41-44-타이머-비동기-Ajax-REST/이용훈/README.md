## 38장 브라우저의 렌더링 과정

- **파싱**: 텍스트 문서를 읽어서 토큰으로 분해하고, 문법적 의미와 구조를 반영한 **파스 트리**를 만드는 과정
- **렌더링**: HTML, CSS, JS로 작성된 문서를 파싱해서 브라우저에 시각적으로 출력하는 것

### 렌더링 과정 한눈에 ⭐

1. HTML, CSS, JS, 이미지, 폰트 등 리소스를 서버에 요청하고 응답받음
2. 렌더링 엔진이 HTML, CSS를 파싱 → **DOM**, **CSSOM** 생성 → 둘을 결합해 **렌더 트리** 생성
3. JS 엔진이 JS를 파싱 → AST 생성 → 바이트코드로 변환해 실행
   - 이때 JS가 DOM API로 DOM이나 CSSOM을 바꾸면, 다시 렌더 트리로 결합됨
4. 렌더 트리를 기반으로 **레이아웃**(위치, 크기 계산) → **페인트**(화면에 그리기)

### 요청과 응답

- 주소창에 URL 입력 → 호스트 이름이 **DNS**를 통해 IP 주소로 변환 → 해당 서버에 요청
- 경로 없이 루트로 요청하면 보통 `index.html`을 기본으로 응답
- 개발자 도구 Network 패널에서 요청/응답 확인 가능

### HTTP 1.1과 HTTP 2.0

- **HTTP/1.1**: 커넥션당 **하나의 요청과 응답만** 처리 → 리소스가 많으면 응답 시간 증가
- **HTTP/2**: 커넥션당 **여러 개의 요청과 응답** 처리 가능 → 1.1보다 약 50% 빠르다고 알려짐

### HTML 파싱과 DOM 생성

- 흐름: **바이트(2진수) → 문자 → 토큰 → 노드 → DOM**
  - 바이트는 `meta charset`에 지정된 인코딩 방식(UTF-8 등)으로 문자열 변환
  - 문자열을 문법적 의미를 갖는 최소 단위인 토큰으로 분해
  - 토큰을 객체로 변환해서 노드 생성
  - 노드들의 부자 관계를 반영해 트리 자료구조로 구성 → **DOM**
- DOM = HTML 문서를 파싱한 결과물

### CSS 파싱과 CSSOM 생성

- HTML 파싱 중 `link`나 `style` 태그를 만나면 **DOM 생성을 일시 중단**
- CSS 파일을 받아서 파싱 → **CSSOM** 생성 → 끝나면 HTML 파싱 재개
- CSSOM은 **CSS 상속을 반영**해서 만들어짐 (부모에 지정한 font-size 등이 자식에 반영)

### 렌더 트리 생성 ⭐

- DOM + CSSOM 결합 → **렌더링을 위한 트리**
- 화면에 렌더링되지 않는 노드는 포함 X
  - `meta`, `script` 태그 등
  - `display: none`인 노드
- 완성된 렌더 트리로 **레이아웃 계산 → 페인팅**
- 다음 상황에서 레이아웃 계산과 페인팅이 반복됨 (리렌더링)
  - JS로 노드 추가/삭제
  - 브라우저 창 리사이즈
  - 레이아웃에 영향을 주는 스타일 변경
- 리렌더링은 **비용이 크니까 빈번하게 발생하지 않도록 주의**

### 자바스크립트 파싱과 실행

- HTML 파싱 중 `script` 태그를 만나면 **DOM 생성을 일시 중단**하고 제어권을 JS 엔진에 넘김
- JS 엔진 처리 과정
  - **토크나이징**: 소스코드를 토큰으로 분해
  - **파싱**: 토큰들을 분석해서 **AST(추상적 구문 트리)** 생성
  - **코드 생성과 실행**: AST를 바이트코드로 변환해 인터프리터가 실행
    - V8은 자주 쓰이는 코드를 터보팬으로 최적화된 머신 코드로 컴파일, 사용 빈도가 줄면 다시 디옵티마이징

### 리플로우와 리페인트 ⭐

- JS가 DOM API로 DOM이나 CSSOM을 변경하면 렌더 트리가 다시 만들어지고 레이아웃, 페인트가 다시 일어남
- **리플로우**: 레이아웃을 **다시 계산**
  - 노드 추가/삭제, 요소 크기/위치 변경, 윈도우 리사이즈 등 레이아웃에 영향이 있을 때
- **리페인트**: 재결합된 렌더 트리 기반으로 **다시 그리기**
- 둘이 항상 같이 일어나는 건 아님 → 레이아웃에 영향 없는 변경(색상 등)은 **리페인트만** 발생
- 활용: 애니메이션은 `top`, `left` 대신 `transform`, `opacity`를 쓰는 게 좋다고 하는 이유가 이것 (리플로우를 피함)

### 자바스크립트 파싱에 의한 HTML 파싱 중단 ⭐

- 브라우저는 동기적으로 위에서 아래로 파싱 → script를 만나면 HTML 파싱이 멈춤
- 문제
  - DOM이 다 만들어지기 전에 JS가 DOM을 조작하면 요소를 못 찾아서 에러
  - JS 로딩/파싱/실행 시간만큼 렌더링이 지연됨
- 그래서 `script` 태그를 **body 요소의 가장 아래**에 두는 게 좋음

### script 태그의 async / defer 어트리뷰트 ⭐

- **src로 외부 JS를 불러올 때만** 사용 가능 (인라인 스크립트엔 적용 X)

|         | JS 로드            | JS 실행 시점                                    | 실행 순서  |
| ------- | ------------------ | ----------------------------------------------- | ---------- |
| 기본    | HTML 파싱 중단     | 로드 직후                                       | 순서대로   |
| `async` | HTML 파싱과 동시에 | **로드 완료 직후** (이때 HTML 파싱 중단)        | **보장 X** |
| `defer` | HTML 파싱과 동시에 | **HTML 파싱 완료 직후** (DOMContentLoaded 직전) | 순서대로   |

- `async`: 다른 스크립트나 DOM에 의존하지 않는 스크립트에 적합 (광고, 분석 스크립트 등)
- `defer`: DOM 생성이 끝난 후 실행돼야 하는 스크립트에 적합

```html
<script defer src="app.js"></script>
```

---

## 39장 DOM

- **DOM**: HTML 문서의 계층적 구조와 정보를 표현하고, 이를 제어할 수 있는 API(프로퍼티와 메서드)를 제공하는 **트리 자료구조**

### 노드

#### HTML 요소와 노드 객체

- HTML 요소 = 시작 태그 + 콘텐츠 + 종료 태그
- HTML 요소는 파싱되어 **요소 노드 객체**로 변환
  - 어트리뷰트 → 어트리뷰트 노드
  - 텍스트 콘텐츠 → 텍스트 노드
- HTML 요소의 중첩 관계로 부자 관계가 생기고, 이걸 반영해서 노드 객체들을 트리 구조로 구성 → **DOM 트리**

#### 노드 타입

- 총 12개, 중요한 건 4개
- **문서 노드**: DOM 트리 최상위 루트 노드, `document` 객체
  - 브라우저 렌더링 엔진이 HTML 문서 전체를 가리키는 객체로 생성 → `window.document`로 참조
  - HTML 문서당 하나만 존재
  - DOM 트리 노드들에 접근하기 위한 **진입점**
- **요소 노드**: HTML 요소를 가리키는 객체, 부자 관계로 문서 구조를 표현
- **어트리뷰트 노드**: HTML 어트리뷰트를 가리키는 객체
  - 요소 노드와 연결되어 있지만 부모 노드와 연결되어 있진 않음 → 요소 노드의 형제 X
  - 그래서 어트리뷰트 노드에 접근하려면 **먼저 요소 노드에 접근**해야 함
- **텍스트 노드**: HTML 요소의 텍스트를 가리키는 객체
  - 문서의 정보를 표현, 자식 노드를 가질 수 없는 **리프 노드**
- 그 외: 주석 노드, DocumentType 노드, DocumentFragment 노드 등

#### 노드 객체의 상속 구조

- 노드 객체도 자바스크립트 객체 → **프로토타입 기반 상속**

```
Object → EventTarget → Node → Element → HTMLElement → HTMLInputElement 등
                            → Document → HTMLDocument
                            → Attr
                            → CharacterData → Text
```

- 모든 노드 객체는 `EventTarget`을 상속 → `addEventListener` 등 이벤트 관련 기능 사용 가능
- 노드 타입에 따라 필요한 기능을 프로퍼티와 메서드로 제공 → 이게 **DOM API**

### 요소 노드 취득

#### getElementById

- `document.getElementById('id')`: `Document.prototype`의 메서드라 **document로만 호출**
- id가 같은 요소가 여러 개면 **첫 번째 하나**만 반환, 없으면 `null`
- id 값과 같은 이름의 **전역 변수가 암묵적으로 선언**되고 해당 노드가 할당됨
  - 같은 이름의 전역 변수가 이미 있으면 재할당되지 않음

#### getElementsByTagName / getElementsByClassName

- 태그 이름 / 클래스 이름으로 요소들을 찾아서 **HTMLCollection** 반환
- `Document.prototype`과 `Element.prototype` 둘 다에 있음
  - `document.getElementsByTagName('li')` → 문서 전체에서 탐색
  - `$ul.getElementsByTagName('li')` → 특정 요소의 **자손 중에서** 탐색
- `getElementsByTagName('*')` → 모든 요소
- `getElementsByClassName('a b')` → 공백으로 여러 클래스 지정 가능
- 없으면 빈 HTMLCollection 반환

#### querySelector / querySelectorAll ⭐

- **CSS 선택자**로 요소 취득
- `querySelector`: 첫 번째 요소 하나, 없으면 `null`
- `querySelectorAll`: 모든 요소를 **NodeList**로 반환, 없으면 빈 NodeList
- 선택자 문법이 틀리면 DOMException 에러
- `Document.prototype`, `Element.prototype` 둘 다에 있음
- getElementById 등보다 다소 느리지만 더 구체적인 조건으로, 일관된 방식으로 취득 가능
- 권장: **id가 있으면 getElementById, 나머지는 querySelector(All)**

#### 특정 요소 노드를 취득할 수 있는지 확인

- `Element.prototype.matches(선택자)`: 해당 선택자로 이 요소를 취득할 수 있는지 boolean 반환
- 이벤트 위임에 유용 (40장)

#### HTMLCollection과 NodeList ⭐

- 둘 다 DOM API가 여러 결과를 반환할 때 쓰는 **DOM 컬렉션 객체**
- 둘 다 **유사 배열 객체이면서 이터러블** → for 문, for...of, 스프레드 가능
- **HTMLCollection**: 노드 객체의 상태 변화를 실시간으로 반영하는 **살아 있는(live) 객체**
  - 순회하면서 클래스를 바꾸면 컬렉션에서 요소가 실시간으로 빠져서 일부만 바뀌는 문제

```jsx
const $elems = document.getElementsByClassName("red");

for (let i = 0; i < $elems.length; i++) {
  $elems[i].className = "blue"; // 바뀌는 즉시 컬렉션에서 빠짐 → 하나 건너뛰게 됨
}
```

- 해결: 역방향 순회, while 문 → **가장 좋은 건 배열로 변환해서 사용**
- **NodeList**: 대부분 실시간 반영 X (**non-live**, 정적 상태 유지)
  - 단, `childNodes`가 반환하는 NodeList는 live
  - `forEach`, `item`, `entries`, `keys`, `values` 메서드 제공
- 결론: **둘 다 배열로 바꿔서 쓰는 게 안전** → `[...collection]`, `Array.from(collection)`

### 노드 탐색

- 노드 탐색 프로퍼티는 모두 **접근자 프로퍼티**인데 setter가 없는 **읽기 전용**
- `parentNode`, `childNodes`, `firstChild`, `previousSibling` 등 → `Node.prototype`
- `children`, `firstElementChild`, `nextElementSibling` 등 → `Element.prototype`

#### 공백 텍스트 노드

- HTML 요소 사이의 스페이스, 탭, 줄바꿈 같은 공백 문자도 **텍스트 노드를 생성**
- 그래서 `firstChild`, `childNodes` 같은 프로퍼티는 공백 텍스트 노드까지 반환할 수 있음 → 주의

#### 자식 노드 탐색

| 텍스트 노드 포함        | 요소 노드만                 |
| ----------------------- | --------------------------- |
| `childNodes` (NodeList) | `children` (HTMLCollection) |
| `firstChild`            | `firstElementChild`         |
| `lastChild`             | `lastElementChild`          |

- 자식 노드 존재 확인
  - `hasChildNodes()`: 텍스트 노드 포함해서 확인
  - 요소 노드만 확인하려면 `children.length` 또는 `childElementCount`
- 요소 노드의 텍스트 노드는 자식이니까 `firstChild`로 접근

#### 부모 / 형제 노드 탐색

- 부모: `parentNode` (텍스트 노드는 리프 노드라 부모가 될 수 없음)
- 형제

| 텍스트 노드 포함  | 요소 노드만              |
| ----------------- | ------------------------ |
| `previousSibling` | `previousElementSibling` |
| `nextSibling`     | `nextElementSibling`     |

### 노드 정보 취득

- `nodeType`: 노드 타입을 상수로 반환
  - `Node.ELEMENT_NODE` = 1, `Node.TEXT_NODE` = 3, `Node.DOCUMENT_NODE` = 9
- `nodeName`: 노드 이름
  - 요소 노드는 대문자 태그 이름 (`'UL'`), 텍스트 노드는 `'#text'`, 문서 노드는 `'#document'`

### 요소 노드의 텍스트 조작

#### nodeValue

- 노드 객체의 값 = **텍스트 노드의 텍스트**
- 문서 노드, 요소 노드의 nodeValue는 `null`
- 요소의 텍스트를 바꾸려면: 텍스트 노드 취득(`firstChild`) → `nodeValue`에 할당

#### textContent ⭐

- 요소 노드의 텍스트 + **모든 자손 노드의 텍스트**를 한 번에 취득/변경 (HTML 마크업은 무시)
- 값을 할당하면 **모든 자식 노드를 제거**하고 할당한 문자열을 텍스트로 추가
  - 문자열에 HTML 마크업이 있어도 **파싱하지 않고 텍스트로** 취급 → XSS에 안전
- `innerText`는 쓰지 말 것
  - CSS를 고려해서 `visibility: hidden` 등 숨겨진 요소의 텍스트는 제외
  - CSS를 고려해야 해서 textContent보다 느림

### DOM 조작

- 새 노드를 생성해서 추가하거나 기존 노드를 삭제/교체하는 것
- DOM 조작은 **리플로우와 리페인트를 일으키니까 성능 최적화에 주의**

#### innerHTML ⭐

- 요소의 콘텐츠 영역 내 **HTML 마크업 문자열**을 취득/변경
- 할당하면 기존 자식 노드를 모두 제거하고, 문자열을 **파싱해서 DOM에 반영**
- 간단하고 편하지만 단점이 많음
  - **크로스 사이트 스크립팅(XSS)에 취약**
    - HTML5부터 innerHTML로 넣은 `script` 태그는 실행되지 않지만, `<img src="x" onerror="...">`처럼 이벤트 핸들러로 스크립트 실행 가능
    - 사용자 입력을 넣을 거면 **HTML 새니티제이션** 필요 (DOMPurify 라이브러리 등)
  - `+=`로 추가해도 **기존 자식 노드까지 전부 제거하고 다시 생성**함 → 비효율
  - 새 요소를 **삽입할 위치를 지정할 수 없음**

#### insertAdjacentHTML

- `insertAdjacentHTML(position, html문자열)`: 기존 요소는 그대로 두고 지정 위치에 삽입

```html
<!-- beforebegin -->
<div>
  <!-- afterbegin -->
  text
  <!-- beforeend -->
</div>
<!-- afterend -->
```

- innerHTML보다 효율적이고 빠르지만, 문자열을 파싱하므로 **XSS 취약점은 동일**

#### 노드 생성과 추가

- `document.createElement('li')`: 요소 노드 생성
  - 생성만 할 뿐 **DOM에 추가되진 않음**, 자식 노드도 없음
- `document.createTextNode('text')`: 텍스트 노드 생성
- `$parent.appendChild($child)`: **마지막 자식**으로 추가
- 요소에 텍스트만 넣을 거면 텍스트 노드 만들 필요 없이 `textContent`가 간편

```jsx
const $li = document.createElement("li");
$li.textContent = "Banana";
$fruits.appendChild($li);
```

#### 여러 노드 추가와 DocumentFragment ⭐

- 반복문으로 appendChild를 여러 번 하면 DOM이 그만큼 변경 → **리플로우, 리페인트 여러 번**
- 컨테이너 div에 모아서 한 번에 넣으면 불필요한 div가 DOM에 남음
- **DocumentFragment** 사용
  - `document.createDocumentFragment()`로 생성
  - 부모 노드가 없어서 DOM과 **별도로 존재**
  - DOM에 추가하면 **자신은 빠지고 자식 노드만** 추가됨 → DOM 변경 1번

```jsx
const $fragment = document.createDocumentFragment();

["Apple", "Banana", "Orange"].forEach((text) => {
  const $li = document.createElement("li");
  $li.textContent = text;
  $fragment.appendChild($li);
});

$fruits.appendChild($fragment); // 리플로우 1번
```

#### 노드 삽입 / 이동 / 복사 / 교체 / 삭제

- **삽입**
  - `appendChild(newNode)`: 마지막 자식으로
  - `insertBefore(newNode, childNode)`: childNode **앞에** 삽입
    - childNode는 반드시 호출한 노드의 자식이어야 함 (아니면 에러)
    - childNode가 `null`이면 마지막에 추가
- **이동**: 이미 DOM에 있는 노드를 appendChild/insertBefore 하면 **원래 위치에서 빠지고 새 위치로 이동**
- **복사**: `cloneNode(deep)`
  - `true`: 깊은 복사 (모든 자손 포함)
  - `false` 또는 생략: 얕은 복사 (노드 자신만, 자식이 없으니 텍스트 노드도 복사 X)
- **교체**: `$parent.replaceChild(newChild, oldChild)`
- **삭제**: `$parent.removeChild(child)` (child는 호출 노드의 자식이어야 함)

### 어트리뷰트

#### 어트리뷰트 노드와 attributes 프로퍼티

- HTML 어트리뷰트 하나당 어트리뷰트 노드 하나 생성
- 요소의 모든 어트리뷰트 노드는 **NamedNodeMap**(유사 배열, 이터러블)에 담기고, 요소 노드의 `attributes` 프로퍼티(읽기 전용)로 참조

#### HTML 어트리뷰트 조작

- `getAttribute(name)`: 값 취득
- `setAttribute(name, value)`: 값 변경/추가
- `hasAttribute(name)`: 존재 확인
- `removeAttribute(name)`: 삭제

#### HTML 어트리뷰트 vs DOM 프로퍼티 ⭐

- 요소 노드에는 HTML 어트리뷰트에 대응하는 **DOM 프로퍼티**도 있음 (`id`, `type`, `value` 등)
  - 처음엔 HTML 어트리뷰트 값으로 초기화됨
- 역할이 다름
  - **HTML 어트리뷰트**: 요소의 **초기 상태** 지정, 변하지 않음
  - **DOM 프로퍼티**: 요소의 **최신 상태** 관리 (사용자 입력 반영)

```html
<input id="user" type="text" value="lee" />
```

```jsx
// 사용자가 'kim'으로 바꿔 입력한 뒤
$input.getAttribute("value"); // 'lee' → 초기 상태
$input.value; // 'kim' → 최신 상태
```

- 사용자 입력과 관계없는 어트리뷰트(id 등)는 어트리뷰트와 DOM 프로퍼티가 **항상 같은 값으로 동기화**
- 대응 관계가 항상 1:1은 아님
  - `id` ↔ `id`: 1:1
  - `input`의 `value` 어트리뷰트 ↔ `value` 프로퍼티: 초기값만 같고 이후는 별개
  - `class` ↔ `className`, `classList`
  - `for` ↔ `htmlFor`
  - `td`의 `colspan`: 대응 프로퍼티 없음
  - `textContent`: 대응 어트리뷰트 없음
  - 어트리뷰트 이름은 대소문자 무관, 프로퍼티는 카멜 케이스 (`maxlength` → `maxLength`)
- 값 타입
  - `getAttribute`는 **항상 문자열**
  - DOM 프로퍼티는 문자열이 아닐 수도 있음 (checkbox의 `checked`는 **불리언**)

#### data 어트리뷰트와 dataset 프로퍼티

- `data-` 접두사를 붙인 **사용자 정의 어트리뷰트**
- `dataset` 프로퍼티로 접근, 이름은 **카멜 케이스로 변환**

```html
<li data-user-id="7621" data-role="admin">Lee</li>
```

```jsx
$li.dataset.userId; // '7621'
$li.dataset.role = "user"; // 변경
$li.dataset.score = "90"; // 없는 이름에 할당하면 data-score 어트리뷰트 추가
```

- 활용: 리스트 항목에 id를 심어두고, 이벤트 위임으로 클릭된 항목의 id를 꺼낼 때

### 스타일

#### 인라인 스타일 조작

- `style` 프로퍼티(CSSStyleDeclaration)로 **인라인 스타일**을 취득/변경
- CSS 프로퍼티는 **카멜 케이스** (`backgroundColor`), 케밥 케이스는 대괄호로 (`style['background-color']`)
- 단위가 필요한 값은 **단위 생략하면 적용 안 됨** (`width = '100px'`)

#### 클래스 조작

- `class`는 예약어라서 `className`, `classList` 사용
- `className`: 클래스 전체를 공백 구분 **문자열**로 다룸
- `classList`(DOMTokenList): 개별 클래스 조작에 편리
  - `add`, `remove`, `contains`, `replace(old, new)`, `item(index)`
  - `toggle(className, force)`: 있으면 제거, 없으면 추가 / force가 true면 강제 추가, false면 강제 제거
- 활용: 스타일을 JS에서 직접 바꾸기보다 **CSS에 클래스를 정의해두고 classList로 토글**하는 게 관리하기 편함

```jsx
$menu.classList.toggle("open");
$tab.classList.toggle("active", isActive);
```

#### 요소에 적용된 CSS 스타일 참조

- `style` 프로퍼티는 **인라인 스타일만** 반환
- `window.getComputedStyle(element, pseudo)`: 링크/임베딩/인라인/JS 적용/상속/기본 스타일까지 **최종 적용된 스타일** 반환 (읽기 전용)
  - 두 번째 인수로 `'::after'` 같은 의사 요소 지정 가능

### DOM 표준

- 원래 W3C와 WHATWG가 공동으로 표준을 만들다가 2018년부터 **WHATWG**(구글, 애플, MS, 모질라) 주도의 단일 표준으로 통일
- DOM Level 1 ~ 4 버전이 있음

---

## 40장 이벤트

### 이벤트 드리븐 프로그래밍

- 브라우저는 클릭, 키보드 입력 등 이벤트를 감지해서 특정 타입의 이벤트를 발생시킴
- **이벤트 핸들러**: 이벤트가 발생했을 때 호출될 함수
- **이벤트 핸들러 등록**: 이벤트 발생 시 핸들러 호출을 브라우저에 위임하는 것
- 이벤트 중심으로 프로그램 흐름을 제어하는 방식 = **이벤트 드리븐 프로그래밍**

### 이벤트 타입

- 약 200가지, 자주 쓰는 것 위주로

| 분류         | 이벤트                                                                                                                          |
| ------------ | ------------------------------------------------------------------------------------------------------------------------------- |
| 마우스       | `click`, `dblclick`, `mousedown`, `mouseup`, `mousemove`, `mouseenter`/`mouseleave`(버블링 X), `mouseover`/`mouseout`(버블링 O) |
| 키보드       | `keydown`, `keyup` (`keypress`는 폐지)                                                                                          |
| 포커스       | `focus`/`blur`(버블링 X), `focusin`/`focusout`(버블링 O)                                                                        |
| 폼           | `submit`, `reset`                                                                                                               |
| 값 변경      | `input`(값이 바뀔 때마다), `change`(값 변경 후 **포커스를 잃을 때**), `readystatechange`                                        |
| DOM 뮤테이션 | `DOMContentLoaded` (DOM 생성 완료)                                                                                              |
| 뷰           | `resize`, `scroll`                                                                                                              |
| 리소스       | `load`(DOMContentLoaded 이후 **모든 리소스까지** 로딩 완료), `unload`, `abort`, `error`                                         |

### 이벤트 핸들러 등록 ⭐

#### 1. 이벤트 핸들러 어트리뷰트 방식

```html
<button onclick="sayHi('Lee')">Click</button>
```

- `on` + 이벤트 타입 어트리뷰트
- 어트리뷰트 값은 함수 참조가 아니라 **함수 호출문** → 암묵적으로 생성되는 이벤트 핸들러 함수의 **몸체**가 됨 (그래서 인수 전달 가능)
- HTML과 JS는 관심사가 달라서 섞지 않는 게 좋음 → 사용 X
- 단, React, Vue 같은 CBD(Component Based Development) 프레임워크는 HTML, CSS, JS를 뷰를 구성하는 하나의 관심사로 보고 이 방식을 사용 (`onClick={...}`)

#### 2. 이벤트 핸들러 프로퍼티 방식

```jsx
$button.onclick = function () {
  console.log("click");
};
```

- 이벤트 타깃(`$button`), 이벤트 타입(`click`), 이벤트 핸들러로 구성
- 하나의 이벤트에 **핸들러를 하나만** 바인딩 가능 → 다시 할당하면 덮어씀

#### 3. addEventListener 메서드 방식

```jsx
$button.addEventListener(
  "click",
  function () {
    console.log("click");
  },
  false,
);
```

- DOM Level 2에서 도입
- 세 번째 인수: 캡처링 단계에서 캐치할지 여부, 기본값 `false`(버블링)
- **여러 개의 핸들러 등록 가능** → 등록한 순서대로 호출
- 같은 핸들러를 중복 등록하면 하나만 등록됨
- 프로퍼티 방식과 함께 쓰면 둘 다 호출됨

### 이벤트 핸들러 제거 ⭐

- `removeEventListener`: **addEventListener에 넘긴 인수와 완전히 같아야** 제거됨
- 무명 함수로 등록하면 참조가 없어서 **제거 불가** → 핸들러를 변수에 담아서 등록해야 함

```jsx
const handleClick = () => console.log("click");

$button.addEventListener("click", handleClick);
$button.removeEventListener("click", handleClick); // OK
```

- 핸들러 내부에서 자기 자신을 제거하면 **한 번만 실행**되는 핸들러
- 프로퍼티 방식으로 등록한 핸들러는 removeEventListener로 제거 불가 → `null` 할당
- 활용: React `useEffect`에서 `window`에 리스너를 달면 cleanup에서 **같은 함수 참조로 제거**해야 하는 이유

```jsx
useEffect(() => {
  const onResize = () => setWidth(window.innerWidth);
  window.addEventListener("resize", onResize);
  return () => window.removeEventListener("resize", onResize);
}, []);
```

### 이벤트 객체

- 이벤트가 발생하면 이벤트 정보를 담은 **이벤트 객체**가 동적으로 생성되어 핸들러의 **첫 번째 인수**로 전달됨

```jsx
$button.addEventListener("click", (e) => console.log(e));
```

- 어트리뷰트 방식에서는 반드시 **`event`라는 이름**으로 받아야 함 (암묵적 핸들러의 매개변수 이름이 event)

#### 이벤트 객체의 상속 구조

- `Event` → `UIEvent` → `MouseEvent`, `KeyboardEvent`, `FocusEvent`, `InputEvent` 등 / `CustomEvent` 등
- 이벤트 타입에 따라 생성되는 객체가 다르고 가지는 프로퍼티도 다름
- 생성자 함수라서 직접 생성도 가능 (커스텀 이벤트)

#### 공통 프로퍼티 ⭐

| 프로퍼티           | 설명                                                     |
| ------------------ | -------------------------------------------------------- |
| `type`             | 이벤트 타입                                              |
| `target`           | **이벤트를 발생시킨** DOM 요소                           |
| `currentTarget`    | **이벤트 핸들러가 바인딩된** DOM 요소                    |
| `eventPhase`       | 이벤트 전파 단계 (0 없음, 1 캡처링, 2 타깃, 3 버블링)    |
| `bubbles`          | 버블링 여부                                              |
| `cancelable`       | `preventDefault`로 기본 동작 취소 가능 여부              |
| `defaultPrevented` | `preventDefault` 호출 여부                               |
| `isTrusted`        | 사용자 행위로 발생했으면 true, 코드로 발생시켰으면 false |
| `timeStamp`        | 이벤트 발생 시각                                         |

#### 마우스 / 키보드 정보

- 마우스: 좌표(`clientX/Y`, `pageX/Y`, `screenX/Y`, `offsetX/Y`), 버튼(`button`), 보조키(`altKey`, `ctrlKey`, `shiftKey`)
- 키보드: `key`(입력한 키 문자열), 보조키(`altKey`, `ctrlKey`, `shiftKey`, `metaKey`), `keyCode`는 폐지
- 주의: 한글 입력 후 Enter를 누르면 조합 중인 입력 때문에 이벤트가 **두 번 호출**되는 현상
  - 활용: `e.isComposing`으로 조합 중인 입력은 무시

```jsx
$input.addEventListener("keydown", (e) => {
  if (e.key !== "Enter" || e.isComposing) return;
  addTodo(e.target.value);
});
```

### 이벤트 전파 ⭐

- DOM 요소에서 발생한 이벤트는 **DOM 트리를 통해 전파**됨
- 3단계
  1. **캡처링 단계**: window → 이벤트 타깃 방향으로 내려감
  2. **타깃 단계**: 이벤트 타깃에 도달
  3. **버블링 단계**: 이벤트 타깃 → window 방향으로 올라감
- 어트리뷰트/프로퍼티 방식은 **타깃, 버블링 단계만** 캐치
- addEventListener 세 번째 인수를 `true`로 주면 캡처링 단계도 캐치
- 즉 이벤트는 타깃에서만 처리되는 게 아니라 **상위 요소에서도 캐치할 수 있음**
- 버블링되지 않는 이벤트 (bubbles가 false)
  - 포커스: `focus`, `blur` → 대신 `focusin`, `focusout`
  - 리소스: `load`, `unload`, `abort`, `error`
  - 마우스: `mouseenter`, `mouseleave` → 대신 `mouseover`, `mouseout`
  - 이런 이벤트도 캡처링 단계에서는 캐치 가능

### 이벤트 위임 ⭐

- 여러 하위 요소에 각각 핸들러를 등록하는 대신 **상위 요소에 핸들러 하나만 등록**하는 방법
- 버블링 덕분에 하위 요소에서 발생한 이벤트를 상위에서 캐치 가능
- 장점
  - 핸들러가 하나라 **메모리 절약**, 성능 저하 X
  - 나중에 **동적으로 추가된 하위 요소**도 따로 등록할 필요 없음
- 주의: `e.target`이 기대한 요소가 아닐 수 있음 → `matches`나 `closest`로 확인
  - `e.currentTarget`은 항상 핸들러를 등록한 상위 요소

```jsx
$todoList.addEventListener("click", (e) => {
  const $item = e.target.closest("li");
  if (!$item) return;

  toggleTodo($item.dataset.id);
});
```

- 활용: 할 일 목록, 테이블 행, 무한 스크롤 상품 목록처럼 **항목이 많거나 계속 추가되는 리스트**

### DOM 요소의 기본 동작 조작 ⭐

- **`e.preventDefault()`**: 요소의 **기본 동작 중단**
  - a 태그 링크 이동, checkbox 체크, form submit 시 페이지 새로고침 등
- **`e.stopPropagation()`**: 이벤트 **전파 중단**
  - 상위에 이벤트 위임을 해두고, 특정 하위 요소만 따로 처리하고 싶을 때

```jsx
$form.addEventListener("submit", (e) => {
  e.preventDefault(); // 새로고침 막고 직접 처리
  submit(new FormData(e.target));
});
```

### 이벤트 핸들러 내부의 this

- **어트리뷰트 방식**: 핸들러 함수가 일반 함수로 호출되니 this는 **전역 객체**
  - 단, 어트리뷰트 값에 `this`를 직접 넘기면(`onclick="handle(this)"`) 그 this는 이벤트를 바인딩한 DOM 요소
- **프로퍼티 / addEventListener 방식**: this는 **이벤트를 바인딩한 DOM 요소** = `e.currentTarget`
- **화살표 함수**로 등록하면 this는 상위 스코프의 this
- 클래스 메서드를 핸들러로 넘기면 this가 DOM 요소가 되어버림 → `bind(this)`나 클래스 필드에 화살표 함수로 해결

### 이벤트 핸들러에 인수 전달

- 프로퍼티 / addEventListener 방식은 **함수 참조를 등록**하는 거라 직접 인수 전달 불가
- 해결
  - 핸들러 안에서 함수를 호출하면서 인수 전달 (화살표 함수로 감싸기)
  - 핸들러를 반환하는 함수를 호출

```jsx
$input.addEventListener("blur", () => checkLength(5));

const makeHandler = (min) => (e) => checkLength(e.target.value, min);
$input.addEventListener("blur", makeHandler(5));
```

### 커스텀 이벤트

- 이벤트 생성자 함수로 직접 이벤트 객체를 만들 수 있음 → 커스텀 이벤트
- `new CustomEvent('type', { bubbles, cancelable, detail })`
  - `bubbles`, `cancelable`은 기본값 **false**
  - `detail`에 전달하고 싶은 정보를 담음
- 이벤트 생성자 종류에 따라 고유 프로퍼티 지정 가능 (`new MouseEvent('click', { clientX: 50 })`)
- 코드로 만든 이벤트라 `isTrusted`는 false
- **`dispatchEvent`**로 발생시킴
  - 핸들러를 **동기적으로** 호출 (핸들러 직접 호출과 같음) → 디스패치 전에 핸들러를 **먼저 등록**해야 함
- 임의 타입의 커스텀 이벤트는 `on + 타입` 프로퍼티가 없으니 **addEventListener로만** 등록 가능

```jsx
$button.addEventListener("cart:add", (e) => console.log(e.detail));

$button.dispatchEvent(new CustomEvent("cart:add", { detail: { id: 1 } }));
```

---

## 41장 타이머

### 호출 스케줄링

- 함수를 즉시 실행하지 않고 **일정 시간 경과 후 호출**되도록 예약하는 것
- 타이머 함수: `setTimeout`, `setInterval`, `clearTimeout`, `clearInterval`
- ECMAScript 사양이 아니라 **브라우저와 Node.js가 제공하는 호스트 객체**
- 자바스크립트 엔진은 싱글 스레드라 타이머 함수는 **비동기**로 동작

### 타이머 함수

#### setTimeout / clearTimeout ⭐

```jsx
const timerId = setTimeout((a, b) => console.log(a + b), 1000, 1, 2);
clearTimeout(timerId); // 취소
```

- `setTimeout(콜백, delay, ...콜백에 넘길 인수)`
- delay(ms) 후 콜백을 **단 한 번** 호출
- delay 기본값은 0, 0으로 줘도 바로 실행되진 않음 (중첩 시 최소 4ms 지연)
- delay는 **정확한 호출 시간을 보장하지 않음**
  - 시간이 지나면 콜백이 태스크 큐에 들어가고, **콜 스택이 비어야** 실행되기 때문
- 고유한 **타이머 id**를 반환 (브라우저는 숫자, Node.js는 객체) → `clearTimeout`으로 취소

#### setInterval / clearInterval

- delay마다 콜백을 **반복 호출**, `clearInterval(id)`로 취소

```jsx
let count = 1;
const timerId = setInterval(() => {
  console.log(count);
  if (count++ === 5) clearInterval(timerId);
}, 1000);
```

### 디바운스와 스로틀 ⭐

- `scroll`, `resize`, `input`, `mousemove` 같은 이벤트는 **짧은 간격으로 연속 발생** → 핸들러가 과도하게 호출돼서 성능 문제
- 디바운스와 스로틀은 **연속 발생하는 이벤트를 그룹화**해서 과도한 호출을 막는 기법

#### 디바운스

- 이벤트가 연속 발생하면, **마지막 이벤트 후 일정 시간 동안 추가 이벤트가 없을 때 한 번만** 호출
- 활용: 검색어 자동완성(입력이 멈추면 API 요청), 버튼 중복 클릭 방지, resize 끝난 뒤 레이아웃 재계산

```jsx
const debounce = (callback, delay) => {
  let timerId;
  return (...args) => {
    if (timerId) clearTimeout(timerId); // 이전 예약 취소
    timerId = setTimeout(callback, delay, ...args);
  };
};

$input.addEventListener(
  "input",
  debounce((e) => search(e.target.value), 300),
);
```

#### 스로틀

- 이벤트가 연속 발생해도 **일정 시간 간격으로 최대 한 번만** 호출
- 활용: 스크롤 이벤트 처리, 무한 스크롤(스크롤 중에도 주기적으로 위치 확인)

```jsx
const throttle = (callback, delay) => {
  let timerId;
  return (...args) => {
    if (timerId) return; // 실행 대기 중이면 무시
    timerId = setTimeout(() => {
      callback(...args);
      timerId = null;
    }, delay);
  };
};

window.addEventListener("scroll", throttle(checkScrollEnd, 100));
```

- 차이 한 줄: **디바운스는 "다 끝나고 한 번", 스로틀은 "주기적으로 한 번씩"**
- 실무에서는 Lodash 같은 라이브러리의 `debounce`, `throttle` 사용 권장 (엣지 케이스 처리됨)

---

## 42장 비동기 프로그래밍

### 동기 처리와 비동기 처리 ⭐

- 함수를 호출하면 실행 컨텍스트가 생성되어 실행 컨텍스트 스택(콜 스택)에 push되고 실행됨
- 자바스크립트 엔진은 **실행 컨텍스트 스택이 하나** → 한 번에 하나의 태스크만 실행 = **싱글 스레드**
- 그래서 오래 걸리는 태스크가 있으면 그동안 다른 작업을 못 함 → **블로킹**
- **동기 처리**: 현재 태스크가 끝날 때까지 다음 태스크가 대기
  - 실행 순서가 보장되지만 블로킹 발생
- **비동기 처리**: 현재 태스크가 끝나기를 기다리지 않고 다음 태스크를 바로 실행
  - 블로킹이 없지만 실행 순서가 보장되지 않음
- 타이머 함수, HTTP 요청, 이벤트 핸들러는 비동기로 동작
- 비동기 처리는 **이벤트 루프와 태스크 큐**와 깊은 관련이 있음

### 이벤트 루프와 태스크 큐 ⭐

- 자바스크립트는 싱글 스레드인데 브라우저에서는 여러 작업이 동시에 처리되는 것처럼 보임 → 이 **동시성**을 지원하는 게 이벤트 루프
- 자바스크립트 엔진의 두 영역
  - **콜 스택**: 실행 컨텍스트가 쌓이는 곳, 함수 실행 순서 관리
  - **힙**: 객체가 저장되는 메모리 공간 (구조화되어 있지 않음)
- 자바스크립트 엔진은 **콜 스택에 쌓인 걸 실행하는 일만** 함
  - 소스코드 평가와 실행을 제외한 나머지(타이머 관리, HTTP 요청 등)는 **브라우저나 Node.js가 담당**
- **태스크 큐**: 비동기 함수의 콜백이나 이벤트 핸들러가 **일시적으로 보관**되는 곳
- **이벤트 루프**: 콜 스택과 태스크 큐를 계속 확인
  - **콜 스택이 비어 있고** 태스크 큐에 대기 중인 함수가 있으면 → 순서대로(FIFO) 콜 스택으로 이동시켜 실행

#### setTimeout 동작 흐름

```jsx
function foo() {
  console.log("foo");
}
function bar() {
  console.log("bar");
}

setTimeout(foo, 0);
bar();
// bar → foo
```

1. `setTimeout` 호출 → 타이머 설정은 **브라우저에 위임**, setTimeout은 바로 종료
2. `bar()` 실행 → 'bar' 출력
3. 브라우저에서 타이머가 만료되면 `foo`를 **태스크 큐에 등록**
4. 전역 코드까지 끝나서 콜 스택이 비면, 이벤트 루프가 `foo`를 콜 스택으로 이동 → 'foo' 출력

- 그래서 delay가 0이어도 동기 코드가 다 끝난 후에 실행됨
- 정리: **자바스크립트 엔진은 싱글 스레드지만, 브라우저는 멀티 스레드**로 동작해서 비동기 작업을 처리해줌

---

## 43장 Ajax

### Ajax란 ⭐

- **Asynchronous JavaScript and XML**
- 자바스크립트로 브라우저가 서버에 **비동기로 데이터를 요청**하고, 응답받은 데이터로 **웹페이지를 동적으로 갱신**하는 방식
- 브라우저의 Web API인 **XMLHttpRequest** 기반
- 1999년 MS가 도입, 2005년 구글 맵스로 주목받음
- 이전 방식 (완전한 HTML을 받아서 페이지 전체를 다시 렌더링)의 단점
  - 변경 없는 부분까지 매번 받아서 **불필요한 데이터 통신**
  - 변경 없는 부분까지 다시 렌더링 → 화면 **깜빡임**
  - 동기 방식이라 응답이 올 때까지 **블로킹**
- Ajax의 장점
  - **필요한 데이터만** 받음
  - **필요한 부분만** 렌더링 → 깜빡임 없음
  - 비동기라서 블로킹 없음

### JSON ⭐

- 클라이언트와 서버 간 HTTP 통신을 위한 **텍스트 데이터 포맷**, 언어 독립적

#### 표기 방식

- 객체 리터럴과 비슷하지만 **키는 반드시 큰따옴표**
- 값도 문자열이면 **반드시 큰따옴표** (작은따옴표 X)

```json
{
  "name": "Lee",
  "age": 20,
  "alive": true,
  "hobby": ["traveling", "tennis"]
}
```

#### JSON.stringify

- 객체(배열 포함)를 **JSON 문자열로 변환** → **직렬화**
- 서버로 데이터를 보낼 때 사용
- `JSON.stringify(value, replacer, space)`
  - `replacer` 함수로 값을 필터링/변환
  - `space`로 들여쓰기 지정

```jsx
JSON.stringify({ name: "Lee", age: 20 }); // '{"name":"Lee","age":20}'
JSON.stringify(obj, null, 2); // 들여쓰기 2칸
```

#### JSON.parse

- JSON 문자열을 **객체로 변환** → **역직렬화**
- 서버에서 받은 JSON 문자열을 객체로 쓸 때 사용
- 활용: 객체를 localStorage에 저장할 때 stringify, 꺼낼 때 parse

```jsx
localStorage.setItem("user", JSON.stringify(user));
const saved = JSON.parse(localStorage.getItem("user"));
```

### XMLHttpRequest

- 자바스크립트로 HTTP 요청을 보내기 위한 Web API
- 지금은 `fetch`가 주로 쓰이지만 동작 원리를 이해하는 데 의미 있음

#### 주요 프로퍼티와 메서드

- 프로퍼티
  - `readyState`: 요청 상태 (`UNSENT` 0, `OPENED` 1, `HEADERS_RECEIVED` 2, `LOADING` 3, `DONE` 4)
  - `status`: HTTP 상태 코드 (200 등)
  - `statusText`, `responseType`, `response`(응답 몸체)
- 이벤트 핸들러 프로퍼티
  - `onreadystatechange`, `onload`(요청 성공적 완료), `onerror`, `onprogress`, `onabort`, `ontimeout`, `onloadend`
- 메서드
  - `open`(요청 초기화), `send`(요청 전송), `abort`(중단), `setRequestHeader`, `getResponseHeader`

#### HTTP 요청 전송 순서

1. `open(method, url)`: 요청 초기화
2. `setRequestHeader(header, value)`: 요청 헤더 설정 (**open 이후에** 호출)
3. `send(payload)`: 요청 전송
   - GET이면 페이로드는 무시되고 `null`로 처리

```jsx
const xhr = new XMLHttpRequest();
xhr.open("POST", "/users");
xhr.setRequestHeader("content-type", "application/json");
xhr.send(JSON.stringify({ name: "Lee" }));
```

- 주요 헤더
  - `Content-type`: 요청 몸체의 MIME 타입 (`application/json`, `text/plain`, `multipart/form-data` 등)
  - `Accept`: 서버가 응답할 데이터의 MIME 타입, 설정 안 하면 `*/*`

#### HTTP 응답 처리

- 요청은 비동기라서 **이벤트로 응답을 캐치**해야 함

```jsx
xhr.onload = () => {
  if (xhr.status === 200) {
    console.log(JSON.parse(xhr.response));
  } else {
    console.error("Error", xhr.status, xhr.statusText);
  }
};
```

- `readystatechange` 이벤트에서 `readyState === XMLHttpRequest.DONE`인지 확인하는 방법도 있지만, `load` 이벤트는 요청이 완료됐을 때만 발생하니 더 간단

---

## 44장 REST API

- **REST**(REpresentational State Transfer): HTTP를 기반으로 클라이언트가 서버의 리소스에 접근하는 방식을 규정한 **아키텍처**
  - 2000년 로이 필딩의 논문에서 소개, HTTP의 장점을 최대한 활용하자는 취지
- **RESTful**: REST의 기본 원칙을 잘 지킨 설계
- **REST API**: REST를 기반으로 서비스 API를 구현한 것

### REST API의 구성

| 구성 요소              | 내용               | 표현 방법        |
| ---------------------- | ------------------ | ---------------- |
| 자원 (resource)        | 서버의 자원        | URI (엔드포인트) |
| 행위 (verb)            | 자원에 대한 행위   | HTTP 요청 메서드 |
| 표현 (representations) | 행위의 구체적 내용 | 페이로드         |

### REST API 설계 원칙 ⭐

#### 1. URI는 리소스를 표현해야 한다

- 리소스를 식별할 수 있는 **명사** 사용, 동사 X

```
# bad
GET /getTodos/1
GET /todos/show/1

# good
GET /todos/1
```

#### 2. 리소스에 대한 행위는 HTTP 요청 메서드로 표현한다

| 메서드 | 종류           | 목적                      | 페이로드 |
| ------ | -------------- | ------------------------- | -------- |
| GET    | index/retrieve | 모든/특정 리소스 **조회** | X        |
| POST   | create         | 리소스 **생성**           | O        |
| PUT    | replace        | 리소스 **전체 교체**      | O        |
| PATCH  | modify         | 리소스 **일부 수정**      | O        |
| DELETE | delete         | 모든/특정 리소스 **삭제** | X        |

```
# bad
GET /todos/delete/1

# good
DELETE /todos/1
```

- PUT vs PATCH: PUT은 리소스를 **통째로 갈아끼우고**, PATCH는 **보낸 필드만** 바꿈
