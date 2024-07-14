##### `declare [function·namespace]`
- 알 수 없는 뭔가 <sub>(라이브러리 등)</sub>
  - 컴파일러에 정보 전달
```ts
declare function MathFn(a: number, b: number): number;

declare namespace MathFn {
  let operator: '+';
}

const sum: typeof MathFn = (a, b) => a + b;
sum.operator = '+';
```

##### 제너레이터
필자는 프로덕션 코드에서는 제너레이터를 한 번도 써보지 않았다... 하지만 타입스크립트로 이것저것 테스트해 보았을 때, 간단한 예제라도 간단한 것이 없었다.
```ts
function* generator(start: number) {
  yield start + 1;
  yield start + 2;
}

var iterator = generator(0);
console.log(iterator.next()); // { value: 1, done: false }
console.log(iterator.next()); // { value: 2, done: false }
console.log(iterator.next()); // { value: undefined, done: true }
```

타입스크립트는  iterator.next() 가 다음 타입의 객체를 리턴하는 것을 정확하게 추론한다.
```ts
type IteratorNextType = {
  value: number | void;
  done: boolean;
};
```

만일 yield 표현식이 수행된 값을 위해 안전하게 타입을 쓰고 싶다면 다음 예제와 같이 할당할 변수에 어노테이션 타입을 추가하라.
```ts
function* generator(start: number) {
  const newStart: number = yield start + 1;
  yield newStart + 2;
}

var iterator = generator(0);
console.log(iterator.next()); // { value: 1, done: false }
console.log(iterator.next(3)); // { value: 5, done: false }
console.log(iterator.next()); // { value: undefined, done: true }
```

그리고 iterator.next(3) 대신  iterator.next('3') 를 호출하면 컴파일 에러를 만날 수 있을 것이다🎉.

##### `Async`
타입스크립트에서 `async`/`await` 함수는 자바스크립트와 같이 동작하며 타입을 정의할 때 다른 것이 있다면 리턴 타입이 항상 Promise 제네릭(generic)이라는 것이다.
```ts
const sum = async (a: number, b: number): Promise<number> => a + b;
async function sum(a: number, b: number): Promise<number> {
  return a + b;
}
```

##### 타입 가드
타입 가드는 타입을 좁혀주는 메카니즘이다. 예를 들어, `string | number`를 타입을 `string`이나 `number`로 좁힐 수 있도록 해준다. 이런 경우(`typeof x === 'string'`와 같은)를 위한 내장 메카니즘이 있지만, 여러분만의 것을 만들 수도 있다. 다음은 필자가 가장 좋아하는 예제(처음 나에게 이것을 보여주었던 내 친구 Peter에게 주는 팁)이다.

falsy 값을 가지는 배열이 있고 그 값을 제거하는 예제다.
```ts
// Array<number | undefined>
const arrayWithFalsyValues = [1, undefined, 0, 2];
```

보통 자바스크립트에서는 이렇게 할 수 있다.
```ts
// Array<number | undefined>
const arrayWithoutFalsyValues = arrayWithFalsyValues.filter(Boolean);
```

아쉽지만, 타입스크립트는 타입을 좁혀주는 가드를 고려하지 못한다. 그래서 타입은 (좁혀지지 않은 채) 여전히 `Array<number | undefined>` 이다.

그래서 우리는 함수를 작성해서 컴파일러에 주어진 인자가 특정 타입인지 여부에 대해 `true`/`false를` 리턴한다고 알려줄 수 있다. 우리는 어떤 주어진 인자의 타입이 `falsy` 값의 타입이 아니라면 우리 함수가 `true`를 리턴한다고 할 수 있다.
```ts
type FalsyType = false | null | undefined | '' | 0;
function typedBoolean<ValueType>(value: ValueType): value is Exclude<ValueType, FalsyType> {
  return Boolean(value);
}
```

그리고 이것을 활용해서 다음과 같이 할 수 있다.
```ts
// Array<number>
const arrayWithoutFalsyValues = arrayWithFalsyValues.filter(typedBoolean);
```

우와!

##### 단언 함수
여러분이 무언가에 대해서 매우 확실함을 얼마나 자주 런타임 체크를 하는지 알고 있는가? 가령, 객체가 어떤 값이나 `null`을 갖는 속성을 가지며 그것이 `null`인지 체크하고 `null`일 때 예외를 발생시킬 수 있을 것이다. 예를 들어 다음과 같은 작업을 할 수 있을 것이다.
```ts
type User = {
  name: string;
  displayName: string | null;
};

function logUserDisplayNameUpper(user: User) {
  if (!user.displayName) throw new Error('이런, 사용자의 displayName이 없습니다.');
  console.log(user.displayName.toUpperCase());
}
```

타입스크립트는 `user.displayName.toUpperCase()` 에 오류를 발생하지 않는데, 왜냐하면 `if` 구문이 그것을 이해하게 해주는 타입 가드이기 때문이다. 다음 예제는 `if`로 확인하는 것을 함수 안으로 넣는 것이다.
```ts
type User = {
  name: string;
  displayName: string | null;
};

function assertDisplayName(user: User) {
  if (!user.displayName) throw new Error('이런, 사용자의 displayName이 없습니다.');
}

function logUserDisplayName(user: User) {
  assertDisplayName(user);
  console.log(user.displayName.toUpperCase());
}
```

이제 타입스크립트는 `assertDisplayName` 호출이 충분한 타입 가드가 아니기 때문에 문제가 발생하게 되었다. 필자는 이것이 타입스크립트의 한계라고 생각한다. 이봐, 완벽한 기술은 없어. 어쨌든 우리의 함수가 단언을 만들어 낸다는 것을 알려줌으로써 타입스크립트를 살짝 도와줄 수 있다.
```ts
type User = {
  name: string;
  displayName: string | null;
};

function assertDisplayName(user: User): asserts user is User & { displayName: string } {
  if (!user.displayName) throw new Error('이런, 사용자의 displayName이 없습니다.');
}

function logUserDisplayName(user: User) {
  assertDisplayName(user);
  console.log(user.displayName.toUpperCase());
}
```

그리고 이것이 타입을 좁혀주는 함수로 만드는 또 하나의 방법이다.
