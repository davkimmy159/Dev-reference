# "use strict"

### ES5 변경사항 적용 (모던 자바스크립트)
 - 스크립트 최상단
 - 함수 본문 맨 앞 : 해당 함수만 엄격 모드

### 브라우저 콘솔 : 기본적으로 use strict 적용 X
 - 최상단에 'use strict’ 입력

### 클래스 or 모듈 사용 시 "use strict" 생략 가능

<br />

# 변수와 상수

```javascript
let m1;
m1 = 'Hello!';
let m2 = 'Hello!';
let m3 = 'John', m4 = 25, m5 = 'Hello'; // (권장X)
let m6 = 'John';  // (권장)
let m7 = 25;      // (권장)
let m8 = 'Hello'; // (권장)
let m9 = 'John',  // (취향)
  m10 = 25,
  m11 = 'Hello';
let m12 = 'John'  // (취향)
  , m13 = 25
  , m14 = 'Hello';
```

### 변수 명명 규칙
 - 변수명
   - 문자
   - 숫자
   - $
   - _
 - 첫 글자 숫자 X

### 잘못된 변수명 예시
```javascript
let 1a;      // 변수명 숫자 시작 X
let my-name; // 변수명 '-' 사용 X
```
### 대·소문자 구별
 - apple ≠ AppLE

### 비 라틴계 언어 사용 OK, 권장 X
 - 영어 변수명 사용 국제적인 관습
```javascript
let имя = '...';
let 我 = '...';
```

### 예약어(reserved name)
```javascript
let let = 5;    // 'let' 변수명 사용 X
let return = 5; // 'return'변수명 사용 X
```

### use strict 없이 변수 할당
 - 예전에는 let 없이 값 할당해 변수 생성 가능 (과거 스크립트 호환성)
 - 나쁜 관습

### 상수
```javascript
const m1 = '18.04.1982';
```

### 대문자 상수
 - 대문자 & 밑줄
   - 기억 용이
   - 오타 확률 ↓
   - 가독성 ↑
```javascript
const COLOR_ORANGE = "#FF7F00";
let color = COLOR_ORANGE;
```

### 바람직한 변수명
 - 읽을 수 있는 이름
 - 줄임말 or 짧은 이름 X
 - 최대한 서술적 & 간결

<br />

# 오래된 var

### var 블록 스코프 X
 - var 변수 스코프 : 함수 스코프 or 전역 스코프
 - 블록 기준 스코프 X (블록 밖에서 접근 가능)
 ```javascript
if (true) {
  var test = true;
}
alert(test); // true(전역 변수)

for (var i = 0; i < 10; i++) {
  // ...
}
alert(i); // 10(전역 변수)
```
 - 레벨 변수 : 코드 블록이 함수 안에
```javascript
function sayHi() {
  if (true) {
    var phrase = "Hello";
  }
  alert(phrase); // 제대로 출력
}
sayHi();
alert(phrase); // Error: phrase is not defined
```

### 변수 중복 선언 허용
 - 두 번째 선언문 무시
```javascript
var user = "Pete";
var user = "John"; // 선언 무시, 에러 발생 X
alert(user);       // John
```

### 선언 전 사용 가능
 - 함수 본문 내 var 선언 변수 선언 위치 상관없이 함수 본문 시작지점에서 정의
   - 단, 변수가 중첩 함수 내에서 정의 X
```javascript
function Same1() {
  phrase = "Hello";
  alert(phrase);
  var phrase;
}
Same1();
function Same2() {
  var phrase;
  phrase = "Hello";
  alert(phrase);
}
same2();
```
 - 코드 블록 무시 (동일하게 동작)
```javascript
function same3() {
  phrase = "Hello";
  if (false) {
    var phrase; // if문 블록 무시
  }
  alert(phrase);
}
same3();
```
 - 선언 Hoisting
   - var 선언 모든 변수 함수 최상위로 "hoisted"
 - 할당 Hoisting 적용 X
```javascript
function same4() {
  alert(phrase);
  var phrase = "Hello";
}
same4();
↓↓↓
function same4() {
  var phrase;       // 선언 함수 시작 시 처리
  alert(phrase);    // undefined
  phrase = "Hello"; // 할당은 실행 흐름 해당 코드 도달 시 처리
}
same4();
```

### 즉시 실행 함수 표현식
 - 개발자들 var도 블록 레벨 스코프 가질 수 있게 방안을 고려
   - '즉시 실행 함수 표현식
   - immediately-invoked function expressions (IIFE)
```javascript
(function() {
  let message = "Hello";
  alert(message); // Hello
})();
```
 - (function {…})
   - 자바스크립트 'function’ 키워드 만나면 함수 선언문 시작 예상
   - 선언문으로 함수 생성 시 땐 반드시 함수 이름 선언
```javascript
// 함수를 선언과 동시에 실행하려고 함
function() {      // <-- Error: Function statements require a function name
  let message = "Hello";
  alert(message); // Hello
}();
```
 - 자바스크립트 함수 선언문 정의 함수 정의와 동시에 호출 허용 X
```javascript
// 맨 아래 괄호 때문에 문법 에러
function go() {
  }(); // <-- 함수 선언문 선언 즉시 호출 X
```
 - 함수 괄호로 감싸면 함수 선언문 아닌 표현식 인식하도록 속임
   - 이름 X OK, 즉시 호출 OK
   - 괄호 외에 여러 방법
 - 모던 자바스크립트에서 불필요
```javascript
// IIFE를 만드는 방법
(function() {
  alert("함수를 괄호로 둘러싸기");
})();
(function() {
  alert("전체를 괄호로 둘러싸기");
}());
!function() {
  alert("표현식 앞에 비트 NOT 연산자 붙이기");
}();
+function() {
  alert("표현식 앞에 단항 덧셈 연산자 붙이기");
}();
```

<br />

# 자료형

 - 동적 타입 (dynamically typed)
   - 변수는 자료형 관계없이 모든 데이터일 수 있음
   - 어떤 순간에 문자열, 다른 순간엔 숫자
```javascript
// no error
let message = "hello";
message = 123456;
```

### 숫자형
 - 정수, 부동소수점 숫자 (floating point number)
```javascript
let n = 123;
n = 12.345;
```
#### Infinity : 무한대 (∞)
 - 어느 숫자든 0으로 나누면 무한대
```javascript
alert( 1 / 0 ); // 무한대
```
 - Infinity 직접 참조
```javascript
alert(Infinity); // 무한대
```
#### NaN
 - 계산 중 에러 발생 (부정확 or 미정의 수학 연산 사용 시 에러 발생)
   - NaN 반환
```javascript
alert( "숫자가 아님" / 2 ); // NaN, 문자열 숫자로 나누면 오류 발생
```
 - NaN 여간해선 안바뀜
 - NaN에 어떤 추가 연산 : NaN 반환
```javascript
alert( "숫자가 아님" / 2 + 5 ); // NaN
```
 - 연산 과정 어디선가 NaN 반환 시, 모든 결과에 영향

#### 수학 연산 안전
 - 자바스크립트 행해지는 수학 연산 '안전’
 - 이례적인 연산 자바스크립트에서 가능
   - 0으로 나누기, 비숫자가 문자열 숫자로 취급 등
 - 불가능 연산 실행 시 에러 X → NaN 반환 & 종료

#### BigInt
 - 숫자형 사용 불가 (내부 표현 방식 문제)
   - (253-1)(9007199254740991) 보다 큰 정수
   - -(253-1) 보다 작은 정수
 - 아주 큰 숫자 필요한 상황(암호 관련 작업) or 아주 높은 정밀도 작업
 - 길이 상관없이 정수 표현
 - 정수 리터럴 끝에 n
```javascript
// 끝에 'n' 붙으면 BigInt형 자료
const bigInt = 1234567890123456789012345678901234567890n;
```

### 문자형
 - 따옴표
   - 큰따옴표  ("Hello")
   - 작은따옴표 ('Hello')
   - 백틱    (`Hello`)
```javascript
let str1 = "Hello";
let str2 = 'Single quotes are ok too';
let phrase = `can embed another ${str}`;
```
 - 큰따옴표 & 작은따옴표 : ‘기본적인’ 따옴표
   - 차이 X
 - 백틱 ${…} : 변수, 표현식 (무엇이든)
   - 평가 후 문자열 일부
```javascript
let name = "John";
// 변수 문자열 중간에 삽입
alert( `Hello, ${name}!` ); // Hello, John!
// 표현식 문자열 중간에 삽입
alert( `the result is ${1 + 2}` ); // the result is 3
```

### 글자형 (char) X

### 불린형
 - true
 - false
<br /><br />
 - 비교 결과 저장
```javascript
let isGreater = 4 > 1;
alert(isGreater); // true (비교 결과: "yes")
```

### ‘null’ 값
 - null 값만 포함하는 별도 자료형
```javascript
let age = null;
```
#### 타 언어 null
 - 미존재 객체 참조
 - 널 포인터 (null pointer)
#### 자바스크립트 null
 - ‘존재하지 않는’ 값 (nothing)
 - ‘비어 있는’ 값   (empty)
 - ‘알 수 없는’ 값   (unknown)
```javascript
let age = null; // 나이(age) 알 수 없음 or 값 비어있음
```

### ‘undefined’ 값
 - ‘undefined’ 값만 포함하는 별도 자료형
 - 값 미할당 상태
 - 변수 선언 & 값 미할당 : undefined 자동 할당
```javascript
let age;
alert(age); // 'undefined'
```
 - 명시적 할당
```javascript
let age = 100;
age = undefined; // undefined 값 명시적으로 할당
alert(age); // 'undefined'
```
 - undefined 직접 할당 권장 X
 - ‘비어있거나’ ‘알 수 없는’ 상태 : null

### 객체 & 심볼
#### 객체 (object)
 - 데이터 컬렉션
 - 복잡한 개체 (entity)
#### 심볼 (symbol)
 - 객체 고유 식별자 (unique identifier)

### typeof 연산자
 - 인수 자료형 반환
 - 형태
   - 연산자 : typeof x
   - 함수  : typeof(x)
```javascript
typeof 0            // "number"
typeof 10n          // "bigint"
typeof "foo"        // "string"
typeof true         // "boolean"
typeof Math         // "object"   - 내장 객체
typeof alert        // "function" - 함수형 미존재, 함수 = 객체형 (하위 호환성 : 언어 자체의 오류)
typeof Symbol("id") // "symbol"
typeof undefined    // "undefined"
typeof null         // "object"   - null ≠ object, (하위 호완성 : 언어 자체의 오류)
```
