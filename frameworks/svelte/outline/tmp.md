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
- `load` <sub>(함수)</sub> 내
  - `fetch('/api/now')` 호출
```javascript
/* src/routes/+layout.js */
export async function load({ fetch }) {
  const response = await fetch('/api/now');
  const now = await response.json();

  return {
    now
  };
}
```

##### `src/routes/[...timezone]/+page.svelte`
- `onMount` <sub>(콜백)</sub> 추가
  - `invalidate('/api/now')`
    - 1초마다 호출
```html
<!-- src/routes/[...timezone]/+page.svelte -->
<script>
  import { onMount } from 'svelte';
  import { invalidate } from '$app/navigation';

  // 상위 요소 데이터 변경 시
  // - load (함수) 재실행 X
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

##### 함수 전달 가능
- URL 기반 X
- 패턴 기반

### `Custom dependencies`

##### `load` <sub>(함수)</sub> 내 `fetch(url)` 호출 시
- `url` 의존성 등록

##### `depends(url)`
- `url` 의존성 수동 등록
  - `fetch` 불필요

##### `[a-z]+:` <sub>(문자열 시작 패턴)</sub>
- 유효 URL
- 커스텀 무효화
  - ex\) `data:now`

##### `src/routes/+layout.js`
- `fetch` 호출 X
- 값 직접 반환
  - `depends` 호출
```javascript
/* src/routes/+layout.js */
export async function load({ depends }) {
  depends('data:now');

  return {
    now: Date.now()
  };
}
```

##### `src/routes/[...timezone]/+page.svelte`
- `invalidate` 호출 수정
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

##### 모든 `load` <sub>(함수)</sub> 재실행
- 현재 페이지 적용 함수 전부
- 의존성 무관

##### `src/routes/[...timezone]/+page.svelte`
```html
<!-- src/routes/[...timezone]/+page.svelte -->
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

##### `src/routes/+layout.js`
- `depends` 호출 불필요
```javascript
/* src/routes/+layout.js */
export async function load(/* { depends } */) {
  // depends('data:now');

  return {
    now: Date.now()
  };
}
```

##### `invalidateAll` vs `invalidate(() => true)`
- `invalidateAll`
  - `url` 의존성 무관
- `invalidate(() => true)`
  - `url` 의존성 有

<br />

## Environment variables

##### 환경 변수
- `.env` <sub>(파일)</sub>
  - API keys
  - DB 자격 정보
  - 기타 등등
- 앱 내 사용
- 추가 형식
  - `.env.local`
  - `.env.[mode]`
- `.gitignore` <sub>(파일 · 앱 내 제외 목록)</sub>
  - 민감한 정보
  - 기타 등등
- `process.env` 사용
  - `$env/static/private`

### `$env/static/private`

##### 사용자 웹사이트 입장 자격 확인
- 사용자 암호 확인
  - 환경 변수 사용

##### `.env` <sub>(파일)</sub> 생성
- 환경 변수 추가
```javascript
/* .env */
PASSPHRASE="open sesame"
```

##### `src/routes/+page.server.js`
- `PASSPHRASE`
  - `$env/static/private` 에서 가져오기
  - 폼 액션 내 사용
```javascript
/* src/routes/+page.server.js */
import { redirect, fail } from '@sveltejs/kit';
import { PASSPHRASE } from '$env/static/private';

export function load({ cookies }) {
  if (cookies.get('allowed')) {
    throw redirect(307, '/welcome');
  }
}

export const actions = {
  default: async ({ request, cookies }) => {
    const data = await request.formData();

    if (data.get('passphrase') === PASSPHRASE) {
      cookies.set('allowed', 'true', {
        path: '/'
      });

      throw redirect(303, '/welcome');
    }

    return fail(403, {
      incorrect: true
    });
  }
};
```
The website is now accessible to anyone who knows the correct passphrase.

#### Keeping secrets
It's important that sensitive data doesn't accidentally end up being sent to the browser, where it could easily be stolen by hackers and scoundrels.

SvelteKit makes it easy to prevent this from happening. Notice what happens if we try to import `PASSPHRASE` into `src/routes/+page.svelte`:
```html
<!-- src/routes/+page.svelte -->
<script>
  import { PASSPHRASE } from '$env/static/private';
  export let form;
</script>
```
An error overlay pops up, telling us that `$env/static/private` cannot be imported into client-side code. It can only be imported into server modules:
- `+page.server.js`
- `+layout.server.js`
- `+server.js`
- any modules ending with `.server.js`
- any modules inside `src/lib/server`
In turn, these modules can only be imported by other server modules.

#### Static vs dynamic
The `static` in `$env/static/private` indicates that these values are known at build time, and can be statically replaced. This enables useful optimisations:
```javascript
import { FEATURE_FLAG_X } from '$env/static/private';

if (FEATURE_FLAG_X === 'enabled') {
  // code in here will be removed from the build output
  // if FEATURE_FLAG_X is not enabled
}
```
In some cases you might need to refer to environment variables that are dynamic — in other words, not known until we run the app. We'll cover this case in the next exercise.

### `$env/dynamic/private`


### `$env/static/public`


### `$env/dynamic/public`

