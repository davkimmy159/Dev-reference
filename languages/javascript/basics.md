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

// (권장 X)
let m3 = 'John', m4 = 25, m5 = 'Hello';

// (권장)
let m6 = 'John';
let m7 = 25;
let m8 = 'Hello';

// (취향)
let m9 = 'John',
  m10 = 25,
  m11 = 'Hello';
let m12 = 'John'
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
- 영어 변수명 사용 (국제적 관습)
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
alert(phrase);   // Error: phrase is not defined
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
- 개발자들 "var"도 블록 레벨 스코프 가질 수 있게 방안을 고려
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
alert( `Hello, ${name}!` );        // Hello, John!

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
alert(age);      // 'undefined'
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
typeof alert        // "function" - 함수형 미존재, 함수 = 객체형
                    //             (하위 호환성 : 언어 자체의 오류)
typeof Symbol("id") // "symbol"
typeof undefined    // "undefined"
typeof null         // "object"   - null ≠ object
                    //             (하위 호완성 : 언어 자체의 오류)
```

# 형변환
### 문자형 변환
- 문자형 값 필요 시 자동 변환
- String(value) 함수
```javascript
let value = true;
alert(typeof value);   // boolean

value = String(value); // 변수 value 문자열 "true" 저장
alert(typeof value);   // string
```

### 숫자형 변환
- 수학 관련 함수·표현식 자동 변환
```javascript
alert( "6" / "2" ); // 3, 문자열 → 숫자형 자동변환 후 연산
```
- Number(value) 함수
```javascript
let str = "123";
alert(typeof str);     // string

let num = Number(str); // 문자열 "123" → 숫자 123 변환
alert(typeof num);     // number
```
- 숫자형 값 문자 기반 폼(form) 통해 입력받는 경우 명시적 형 변환 필수
- 숫자 이외 글자 있는 문자열 숫자형 변환 시 : NaN
```javascript
let age = Number("임의의 문자열 123");
alert(age); // NaN, 형 변환이 실패합니다.
```
- 숫자형 변환 규칙
- ※ null & undefined 숫자형 변환 시 결과 다름

|값|형 변환 후|
|---|---|
|undefined|NaN|
|null|0|
|true & false|1 & 0|
|string| ① 문자열 처음 & 끝 공백 제거<br /> ② 공백 제거 후 문자열 없으면 0 or 문자열에서 숫자 읽음<br /> ③ 변환 실패 시 : NaN|

```javascript
alert( Number("   123   ") ); // 123
alert( Number("123z") );      // NaN ("z" 숫자 변환 실패)
alert( Number(true) );        // 1
alert( Number(false) );       // 0
```
### 불린형 변환
- 논리 연산 수행 시 발생
- Boolean(value) 함수
- 불린형 변환 규칙

|값|형 변환 후|
|---|---|
|직관적으로 “비어있는” 값<br />ex)<br /> - 0 (숫자)<br /> - 빈 문자열<br /> - null <br /> - undefined<br /> - NaN 등 |false|
|그 외의 값|true|

```javascript
alert( Boolean(1) );       // 숫자 1 (true)
alert( Boolean(0) );       // 숫자 0 (false)
alert( Boolean("hello") ); // 문자열 (true)
alert( Boolean("") );      // 빈 문자열 (false)
```
#### ※ 문자열 "0" : true
- PHP 등 일부 언어 문자열 "0" : false
- 자바스크립트 비어있지 않은 문자열 언제나 true
```javascript
alert( Boolean("0") ); // true
alert( Boolean(" ") ); // 공백 있는 문자열 : 비어있지 않은 문자열
```

# 기본 연산자와 수학

- operand(피연산자) : 연산 수행 대상
  - = 인수(argument)
- unary(단항) : 피연산자 1개 받는 연산자
- binary(이항) : 피연산자 2개 받는 연산자

### 수학 연산자
- 덧셈 연산자   : +
- 뺄셈 연산자   : -
- 곱셈 연산자   : *
- 나눗셈 연산자  : /
- 나머지 연산자  : %
- 거듭제곱 연산자 : **

### 나머지 연산자 '%'
- 나눈 후 나머지 정수 반환
```javascript
alert( 5 % 2 ); // 5 / 2 나머지 : 1
alert( 8 % 3 ); // 8 / 3 나머지 : 2
```

### 거듭제곱 연산자 '**'
- a를 b번 곱한 값
```javascript
alert( 2 ** 2 ); // 4  (2 * 2)
alert( 2 ** 3 ); // 8  (2 * 2 * 2)
alert( 2 ** 4 ); // 16 (2 * 2 * 2 * 2)
```
- 정수 아닌 숫자 가능
- 제곱근 : 분수 이용
```javascript
alert( 4 ** (1/2) ); // 2 (1/2 거듭제곱 : 제곱근)
alert( 8 ** (1/3) ); // 2 (1/3 거듭제곱 : 세제곱근)
```

### 이항 연산자 '+' & 문자열 연결
- 이항 연산자 + 피연산자로 문자열 전달 시 문자열 병합(연결)
```javascript
let s = "my" + "string";
alert(s);             // mystring
alert( '1' + 2 );     // "12"
alert( 2 + '1' );     // "21"
alert( 2 + 2 + '1' ); // '41' (연산 : 왼쪽 → 오른쪽 순차적 진행)
alert( 6 - '2' );     // 4 ('2' 숫자 변환 후 연산)
alert( '6' / '2' );   // 3 (두 피연산자 숫자로 변환 후 연산)
```

### 단항 연산자 '+' & 숫자형 변환
- 피연산자 숫자 아닌 경우 숫자형 변환
- Number(...) 함수 동일
```javascript
// 숫자 영향 X
let x = 1;
alert( +x );    // 1
let y = -2;
alert( +y );    // -2

// 숫자형 아닌 피연산자 숫자형 변환
alert( +true ); // 1
alert( +"" );   // 0
```
- 단항 덧셈 연산자로 피연산자 숫자형 변환 후 연산 진행
```javascript
let apples = "2";
let oranges = "3";

// 이항 덧셈 연산자 적용 전 두 피연산자 숫자형 변환
alert( +apples + +oranges );               // 5
alert( Number(apples) + Number(oranges) ); // 5
```

### 연산자 우선순위

|순위|기호|연산자 이름|
|:---:|:---:|---|
|17|+|단항 덧셈|
|17|-|단항 부정|
|16|**|지수|
|15|*|곱셈|
|15|/|나눗셈|
|13|+|덧셈|
|13|-|뺄셈|
|…|…|…|
|3|=|할당|
|…|…|…|

### 할당 연산자 '='
- 할당(assignment) 연산자
- 'x = value' → value가 x에 쓰여진 후, value 반환
- 비권장 트릭 (명확성·가독성 ↓)
```javascript
let a = 1;
let b = 2;
let c = 3 - ( a = b + 1 );
alert( a ); // 3 ( a = b + 1 );
alert( c ); // 0
```

### 할당 연산자 체이닝
- 여러 개 연결
- 평가 우측부터 진행
- 모든 변수 단일 값 공유
- 비권장 (명확성·가독성 ↓)
```javascript
let a, b, c;

// 비권장
a = b = c = 2 + 2;
alert( a ); // 4
alert( b ); // 4
alert( c ); // 4

// 권장
c = 2 + 2;
b = c;
a = c;
```
### 복합 할당 연산자
```javascript
// 할당
let n = 2;
n = n + 5;
n = n * 2;

// 복합 할당
let n = 2;
n += 5; // n == 7  (n = n + 5)
n *= 2; // n == 14 (n = n * 2)
```
- 산술 연산자, 비트 연산자 적용
  - /=
  - -= 등
- 우선순위 할당 연산자 동일
```javascript
let n = 2;
n *= 3 + 5;
alert( n ); // 16 (*= 우측 먼저 평가)
```

### 증가·감소 연산자 '++', '--'
- 숫자 하나 증가 or 감소
```javascript
let counter = 2;
counter++; // 3 (counter = counter + 1)
let counter = 2;
counter--; // 1 (counter = counter - 1)
```
- 변수에만 적용
- 값에 사용 시 에러
  - ex) 5++
- 전위형(prefix form) &nbsp;&nbsp;: ++counter
  - 증가·감소 후 새로운 값 반환
- 후위형(postfix form) : counter++
  - 증가·감소 전 기존 값 반환
```javascript
let counter = 0;
alert( ++counter ); // 1
let counter = 0;
alert( counter++ ); // 0
```
#### 다른 연산자 사이 증가·감소 연산자
- ++ / -- 연산자 표현식 중간 사용
- 증가·감소 연산자의 우선순위 : 다른 대부분 산술 연산자보다 ↑ (평가 먼저 실행)
- 가독성 ↓
```javascript
// 비권장
let counter1 = 1;
alert( 2 * ++counter1 ); // 4
let counter2 = 1;
alert( 2 * counter2++ ); // 2 (counter++ '기존'값 반환)

// 권장
let counter3 = 1;
alert( 2 * counter3 );
counter3++;
```

### 비트 연산자
- 32비트 정수 변환 후 이진 연산
- 저수준(2진 표현) 연산
<br /><br />
- 비트 AND   : &
- 비트 OR&nbsp;&nbsp;&nbsp;   : |
- 비트 XOR&nbsp;   : ^
- 비트 NOT    : ~
- 왼쪽 시프트&nbsp;  : <<<br />(LEFT SHIFT)
- 오른쪽 시프트&nbsp; : >><br />(RIGHT SHIFT)
- 부호 없는 오른쪽 시프트 : >>><br />(ZERO-FILL RIGHT SHIFT)

### 쉼표 연산자 ','
- 여러 표현식 코드 한 줄 평가
- 표현식 각각 모두 평가, 마지막 표현식 평가 결과만 반환
```javascript
let a = (1 + 2, 3 + 4);
alert( a ); // 7 (3 + 4의 결과, 앞 결과는 버려짐)
```
- 쉼표 우선순위 매우 ↓ (할당 연산자 '=' 보다 더 ↓)
- 괄호 중요한 역할
- 가독성 ↓
```javascript
let a = 1 + 2, 3 + 4;
// 괄호 없이 진행 순서
// ① a = 3, 7
// ② a = 3
// ③ 7 무시
alert( a ); // 3
```
- 사용처 : 여러 동작 하나의 줄 처리
```javascript
// 한 줄 세 개 연산 수행
for (a = 1, b = 3, c = a * b; a < 10; a++) {
  // ...
}
```

# 비교 연산자
- a > b
- a < b
- a >= b
- a <= b
- a == b (동등)
- a != b (부동)

### 불린형 반환
- 불린형값 반환
- true : ‘긍정’, ‘참’, '사실’
- false : ‘부정’, ‘거짓’, '사실이 아님’
```javascript
alert( 2 > 1 );  // true
alert( 2 == 1 ); // false
alert( 2 != 1 ); // true

// 비교 결과를 변수에 할당
let result = 5 > 4;
alert( result ); // true
```
### 문자열 비교
- ‘사전’ 순 문자열 비교
  - 사전편집(lexicographical)
- 사전 앞쪽 문자열 < 사전 뒤쪽 문자열
```javascript
alert( 'A' < 'Z' );       // true
alert( 'Glee' < 'Glow' ); // true
alert( 'Be' < 'Bee' );    // true
```

1. 두 문자열 첫 글자 비교
2. 첫 번째 문자열 첫 글자 다른 문자열 첫 글자보다 크면(작으면),<br />첫 번째 문자열 두 번째 문자열보다 크다고(작다고) 결론,<br />비교 종료
3. 두 문자열 첫 글자 같으면 두 번째 글자 같은 방식 비교
4. 글자 간 비교 끝날 때까지 과정 반복
5. 비교 종료,<br />문자열 길이 같으면 두 문자열 동일 결론,<br />비교 종료 후 두 문자열 길이 다르면 길이 긴 문자열 더 크다고 결론
- 정확히 유니코드 순 O, 사전 순 X
- 대·소문자 구별
  - 'A' < 'a'

### 다른 형 값 간 비교
- 자료형 다르면 숫자형 변환
```javascript
alert( '2' > 1 );   // true (문자열 '2'  숫자 2 변환 후 비교)
alert( '01' == 1 ); // true (문자열 '01' 숫자 1 변환 후 비교)

// 불린값
alert( true == 1 );  // true (true  : 1 변환 후 비교)
alert( false == 0 ); // true (false : 0 변환 후 비교)
```

#### 흥미로운 상황
- 동등 비교 연산자 '==' 피연산자 : 숫자형 변환 O
- 'Boolean’ 명시적 변환 : 숫자형 변환 X
```javascript
let a = 0;
alert( Boolean(a) ); // false

let b = "0";
alert( Boolean(b) ); // true

alert(a == b);       // true!
```

### 일치 연산자
- 동등 연산자 '==' : '0' & 'false' & '빈 문자열' 구별 X
- 형 다른 피연산자 비교 시 숫자형 변환 때문
  - '빈 문자열' & 'false' 숫자형 변환 : 0
```javascript
alert( 0 == false );  // true
alert( '' == false ); // true
```
- 일치 연산자 '===' 형 변환 없이 값 비교
- 자료형 동등 여부까지 검사
```javascript
alert( 0 === false ); // false (피연산자 형 다름)
```
- 불일치 연산자 '!=='
- 비교 결과 명확 → 에러 발생 확률 ↓

### 'null' or 'undefined'와 비교
- 일치 연산자 '===' 사용하여 'null' & 'undefined' 비교
  - 두 값 자료형 다르기 때문에 거짓 반환
```javascript
alert( null === undefined ); // false
```
- 동등 연산자 == 사용하여 'null' & 'undefined' 비교
  - 특별한 규칙 적용하여 참 반환
  - 'null' & 'undefined' → '각별한 커플’ 취급
  - 두 값 자기들끼리 잘 어울리지만 다른 값들과는 X

```javascript
alert( null == undefined ); // true
```
#### 산술 연산자 or 기타 비교 연산자 (<, >, <=, >=) 사용하여 'null' & 'undefined' 비교
- 'null & 'undefined' 숫자형 변환
  - null → 0
  - undefined → NaN
#### null vs 0
- 동등 연산자 '==' & 기타 비교 연산자 (<, >, <=, >=) 동작 방식 다름
  - (1), (3)
    - (기타 비교 연산자 동작 원리 따라) 'null' 숫자형 변환되어 0 되기 때문
  - (2)
    - 동등 연산자 '==' 피연산자 'null' or 'undefined' 형 변환 X
    - 'null' & 'undefined' 비교 시만 true 반환,<br />그 이외 경우('null' or 'undefined' 다른 값과 비교) 무조건 false 반환
```javascript
alert( null > 0 );  // (1) false
alert( null == 0 ); // (2) false
alert( null >= 0 ); // (3) true
```

#### 비교 불가능한 undefined
- undefined 다른 값과 비교 금지
- (1), (2) : undefined → NaN 변환 (숫자형 변환)
  - NaN 피연산자 경우 비교 연산자 항상 false 반환
- (3) : undefined는 'null' or 'undefined' 일치, 그 이외 값 일치 X
```javascript
alert( undefined > 0 );  // false (1)
alert( undefined < 0 );  // false (2)
alert( undefined == 0 ); // false (3)
```
#### 함정 피하기
- 일치 연산자 '===' 제외 비교 연산자 피연산자에 'null' or 'undefined' 사용 X
- 'null' or 'undefined' 가능성 있는 변수 '<, >, <=, >=' 피연산자 되지 않도록 주의
  - 'null' or 'undefined' 가능성 있는 변수 따로 처리 코드 추가