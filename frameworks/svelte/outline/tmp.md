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
  - `$env/static/private` 가져오기
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

#### Keeping secrets

##### 민감한 정보
- 브라우저 전송 X

##### SvelteKit 기본 방지 기능
- 클라이언트측 코드 내 `import` 방지
```html
<!-- src/routes/+page.svelte -->
<script>
  // 에러 발생
  import { PASSPHRASE } from '$env/static/private';
  export let form;
</script>
```

##### 서버 모듈만 `import` 허용
- `+page.server.js`
- `+layout.server.js`
- `+server.js`
- `[*].server.js`
- `src/lib/server/[*]`

##### 서버 모듈
- 타 서버 모듈로만 `import` 가능

#### Static vs dynamic

##### `…/static/…`
- 빌드 타임 내 결정
  - 정적 코드 변환

##### 최적화 유용
```javascript
import { FEATURE_FLAG_X } from '$env/static/private';

if (FEATURE_FLAG_X === 'enabled') {
  // FEATURE_FLAG_X 기능 미실행 시
  // - 빌드 시 코드 제거
}
```

##### `…/dynamic/…`
- 앱 실행 전까지 확정 X

### `$env/dynamic/private`

##### 앱 구동 시작 시 환경 변수 읽기
```javascript
/* src/routes/+page.server.js */
import { redirect, fail } from '@sveltejs/kit';
import { env } from '$env/dynamic/private';

export function load({ cookies }) {
  if (cookies.get('allowed')) {
    throw redirect(307, '/welcome');
  }
}

export const actions = {
  default: async ({ request, cookies }) => {
    const data = await request.formData();

    if (data.get('passphrase') === env.PASSPHRASE) {
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

### `$env/static/public`

##### 브라우저 노출
- `PUBLIC_…`

##### public 환경 변수 2개 추가
```javascript
/* .env */
PUBLIC_THEME_BACKGROUND="steelblue"
PUBLIC_THEME_FOREGROUND="bisque"
```

##### `src/routes/+page.svelte` import
```html
<!-- src/routes/+page.svelte -->
<script>
  const PUBLIC_THEME_BACKGROUND = 'white';
  const PUBLIC_THEME_FOREGROUND = 'black';
  import {
    PUBLIC_THEME_BACKGROUND,
    PUBLIC_THEME_FOREGROUND
  } from '$env/static/public';
</script>
```

### `$env/dynamic/public`

##### 미공개 환경 변수
- `static` 권장

##### 필요시 `dynamic` 사용
```html
<!-- src/routes/+page.svelte -->
<script>
  import { env } from '$env/dynamic/public';
</script>

<main
  style:background={env.PUBLIC_THEME_BACKGROUND}
  style:color={env.PUBLIC_THEME_FOREGROUND}
>
  {env.PUBLIC_THEME_FOREGROUND} on {env.PUBLIC_THEME_BACKGROUND}
</main>
```

## Conclusion

### Next steps
```
pnpm create svelte@latest
```
