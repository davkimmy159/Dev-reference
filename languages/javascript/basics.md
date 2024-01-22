"use strict"
==========
#### ES5 변경사항 적용 (모던 자바스크립트)
 - 스크립트 최상단
 - 함수 본문 맨 앞 : 해당 함수만 엄격 모드
#### 브라우저 콘솔 : 기본적으로 use strict 적용 X
 - 최상단에 'use strict’ 입력
#### 클래스 or 모듈 사용 시 "use strict" 생략 가능

변수와 상수
==========
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
#### 변수 명명 규칙
 - 변수명
   + 문자
   + 숫자
   + $
   + _
 - 첫 글자 숫자 X

#### 잘못된 변수명 예시
```javascript
let 1a;      // 변수명 숫자 시작 X
let my-name; // 변수명 '-' 사용 X
```
