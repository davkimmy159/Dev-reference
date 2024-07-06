## Advanced loading

### Universal `load` <sub>(함수)</sub>

##### `+[page·layout].server.js`
- 서버 데이터 로드
  - DB 직접 접근
  - 쿠키 읽기
  - 기타 등등

##### 클라이언트측 탐색 시 서버 데이터 불필요 경우
- 외부 API 데이터 로드
- 메모리 데이터 사용
- 탐색 지연 <sub>(이미지 미리 로드전까지)</sub>
  - 깜짝 발생 방지
- 직렬화 불가능 데이터 반환 <sub>(`load` 함수)</sub>
  - SvelteKit 서버 데이터 이스케이프

##### `component` 생성자 반환
- 직렬화 불가능
  - 에러 발생

##### `+page.server.js` → `+page.js`
- `load` <sub>(서버 함수)</sub>
  - → universal `load` <sub>(함수)</sub>
- 서버측 실행
  - SSR
- 브라우저측 실행
  - 앱 hydrate
  - 클라이언트측 탐색 수행

##### hydrate
- HTML 마크업 <sub>(SSR)</sub>
  - JS 이벤트 · 상태 연결 <sub>(클라이언트측)</sub>

##### `component` <sub>(universal `load` 함수 반환값)</sub>
- 일반값처럼 사용 가능
```html
<!-- src/routes/+layout.svelte -->
<nav
  class:has-color={!!$page.data.color}
  style:background={$page.data.color ?? 'var(--bg-2)'}
>
  <a href="/">home</a>
  <a href="/red">red</a>
  <a href="/green">green</a>
  <a href="/blue">blue</a>

  {#if $page.data.component}
    <svelte:component this={$page.data.component} />
  {/if}
</nav>
```

### Using both `load` <sub>(함수)</sub>
- `load` <sub>(서버 함수)</sub>
- universal `load` <sub>(함수)</sub>

##### 두 종류 데이터 반환
- 서버 데이터
- 직렬화 불가능 데이터

##### 데이터 따라 상이 컴포넌트 반환
- `data` <sub>(`+page.js` 프로퍼티)</sub>
  - 서버 데이터 <sub>(`+page.server.js`)</sub> 접근
```javascript
/* src/routes/+page.js */
export async function load({ data }) {
  const module = data.cool
    ? await import('./CoolComponent.svelte')
    : await import('./BoringComponent.svelte');

return {
    component: module.default,
    message: data.message
  };
}
```

##### 데이터 결합 X
- 반환값 <sub>(`message`)</sub> 필수

### Using parent data

##### `+[page·layout].svelte` <sub>(컴포넌트)</sub>
- 상위 `load` <sub>(서버 함수)</sub> 반환값 접근 가능

##### `await parent()`
- 하위 `load` <sub>(서버 함수)</sub>
  - 상위 `load` <sub>(서버 함수)</sub> 데이터 접근
```javascript
/* src/routes/+layout.server.js */
export function load() {
  return { a: 1 };
}
```
```javascript
/* src/routes/sum/+layout.js */
export async function load({ parent }) {
  const { a } = await parent();
  return { b: a + 1 };
}
```

#### universal vs 서버 함수

##### 하위 universal `load` <sub>(함수)</sub>
- 상위 `load` <sub>(서버 함수)</sub> 데이터 접근 가능

##### 하위 `load` <sub>(서버 함수)</sub>
- 상위 universal `load` <sub>(함수)</sub> 데이터 접근 X
- 상위 `load` <sub>(서버 함수)</sub> 데이터 접근 가능

##### `src/routes/sum/+page.js` 부모 데이터 접근
- `src/routes/sum/+layout.js`
- `src/routes/+layout.server.js`
```javascript
/* src/routes/sum/+page.js */
export async function load({ parent }) {
  const { a, b } = await parent();
  return { c: a + b };
}
```

##### 부모 데이터 의존 주의
- 비의존 데이터 `fetch` 가능 시 사용 지양

### Invalidation

##### 사용자 페이지 이동 시
- `load` <sub>(함수)</sub> 호출 기준
  - 어떤 변화 발생

##### `src/routes/[...timezone]/+page.js` <sub>(timezone 탐색)</sub>
- `load` <sub>(함수)</sub> 재실행
  - `params.timezone` 무효
- 상위 layout 데이터 사용 · 변경 시
  - 함수 재실행 X

##### `src/routes/+layout.js` <sub>(최상위 layout)</sub>
- `load` <sub>(함수)</sub> 재실행 X
  - 탐색 의해 무효 X

##### `invalidate(url)` <sub>(함수)</sub>
- `url` <sub>(문자열)</sub>
  - `load` <sub>(함수)</sub> 재실행 기준

##### `src/routes/+layout.js`
- `load` <sub>(함수)</sub>
  - `fetch('/api/now')`

##### `src/routes/[...timezone]/+page.svelte`
- `onMount` <sub>(콜백)</sub> 추가
  - `invalidate('/api/now')` 1초마다 호출
```html
<!-- src/routes/[...timezone]/+page.svelte -->
<script>
  import { onMount } from 'svelte';
  import { invalidate } from '$app/navigation';

  export let data;

  onMount(() => {
    const interval = setInterval(() => {
      invalidate('/api/now');
    }, 1000);

    return () => {
      clearInterval(interval);
    };
  });
</script>

<h1>
  {new Intl.DateTimeFormat([], {
    timeStyle: 'full',
    timeZone: data.timezone
  }).format(new Date(data.now))}
</h1>
```

##### 참고
You can also pass a function to `invalidate`, in case you want to invalidate based on a pattern and not specific URLs

### `Custom dependencies`
Calling `fetch(url)` inside a load function registers `url` as a dependency. Sometimes it's not appropriate to use `fetch`, in which case you can specify a dependency manually with the depends(url) function.

Since any string that begins with an `[a-z]+:` pattern is a valid URL, we can create custom invalidation keys like `data:now`.

Update `src/routes/+layout.js` to return a value directly rather than making a `fetch` call, and add the `depends`:
```javascript
src/routes/+layout.js
export async function load({ depends }) {
  depends('data:now');

  return {
    now: Date.now()
  };
}
```

Now, update the `invalidate` call in `src/routes/[...timezone]/+page.svelte`:
```html
<!-- src/routes/[...timezone]/+page.svelte -->
<script>
  import { onMount } from 'svelte';
  import { invalidate } from '$app/navigation';

  export let data;

  onMount(() => {
    const interval = setInterval(() => {
      invalidate('data:now');
    }, 1000);

    return () => {
      clearInterval(interval);
    };
  });
</script>
```

### `invalidateAll`
Finally, there's the nuclear option — `invalidateAll()`. This will indiscriminately re-run all `load` <sub>(함수)</sub> for the current page, regardless of what they depend on.

Update `src/routes/[...timezone]/+page.svelte` from the previous exercise:
```html
src/routes/[...timezone]/+page.svelte
<script>
  import { onMount } from 'svelte';
  import { invalidateAll } from '$app/navigation';

  export let data;

  onMount(() => {
    const interval = setInterval(() => {
      invalidateAll();
    }, 1000);

    return () => {
      clearInterval(interval);
    };
  });
</script>
```

The `depends` call in `src/routes/+layout.js` is no longer necessary:
```javascript
/* src/routes/+layout.js */
export async function load(/* { depends } */) {
  // depends('data:now');

  return {
    now: Date.now()
  };
}
```

##### 참고
`invalidate(() => true)` and `invalidateAll` are not the same. `invalidateAll` also re-runs `load` <sub>(함수)</sub> without any `url` dependencies, which `invalidate(() => true)` does not.

<br />

## Environment variables

### `$env/static/private`


### `$env/dynamic/private`


### `$env/static/public`


### `$env/dynamic/public`

