Advanced SvelteKit
==================

## Hooks
- sveltekit 기본 동작 가로채기 · 조작

### `handle` <sub>(기본적인 hook)</sub>

##### `resolve` <sub>(콜백)</sub>
- 요청 URL
  - 해당 라우터 연결
- 관련 코드 import
  - `+page.server.js`
  - `+page.svelte`
  - 기타 등등
- 라우터 필요 데이터 로드
- 응답 생성
```javascript
/* src/hooks.server.js */
export async function handle({ event, resolve }) {
  return await resolve(event);
}
```

##### 생성된 HTML <sub>(페이지)</sub> 수정
- `transformPageChunk`
```javascript
/* src/hooks.server.js */
export async function handle({ event, resolve }) {
  return await resolve(event, {
    transformPageChunk: ({ html }) => html.replace(
      '<body',
      '<body style="color: hotpink"'
    )
  });
}
```

##### 새 라우터 생성
```javascript
export async function handle({ event, resolve }) {
  if (event.url.pathname === '/ping') {
    return new Response('pong');
  }

  return await resolve(event, { … });
}
```

### `RequestEvent` <sub>(객체)</sub>

##### 전달 개체
- `handle` <sub>(hook)</sub>
  - `event` <subP>(인수)</subP>
- API 라우터 <sub>(`+server.js`)</sub>
- 폼 액션 <sub>(`+page.server.js`)</sub>
- `load` 함수 <sub>(`+page.server.js`)</sub>
- `+layout.server.js`

#### 프로퍼티 · 메서드

##### `cookies`
- 쿠키 API

##### `fetch`
- 표준 `Fetch` API <sub>(추가 기능 有)</sub>

##### `getClientAddress()`
- 클라이언트 IP 주소 얻기

##### `isDataRequest`
- `true`
  - 페이지 데이터 요청 <sub>(브라우저)</sub>
    - 클라이언트측 탐색 도중
- `false`
  - 페이지 · 라우터 직접 요청되어짐

##### `locals`
- 임의 데이터 보관 위치

##### `params`
- 라우터 매개변수

##### `request`
- `Request` <sub>(객체)</sub>

##### `route` <sub>(객체)</sub>
- `id` <sub>(프로퍼티)</sub>
  - 매칭 라우터

##### `setHeaders(…)`
- 응답 HTTP 헤더 설정

##### `url` <sub>(객체)</sub>
- 현재 요청

##### 유용한 패턴
- `event.locals` <sub>(`handle` hook 안)</sub>
  - 특정 데이터 추가 <sub>(`load` 함수 접근)</sub>
```javascript
/* src/hooks.server.js */
export async function handle({ event, resolve }) {
  event.locals.answer = 42;
  return await resolve(event);
}
```
```javascript
/* src/routes/+page.server.js */
export function load(event) {
  return {
    message: `the answer is ${event.locals.answer}`
  };
}
```

### `handleFetch`

##### `RequestEvent` <sub>(객체)</sub> 내 `fetch` <sub>(메서드)</sub>
- 일반 `fetch` <sub>(메서드)</sub> 동작
- 강력한 추가 기능 有

#### 추가 기능

##### 자격 증명 요청 생성 <sub>(서버)</sub>
- 쿠키 · 인증 헤더 <sub>(수신 요청)</sub> 상속

##### 상대적 요청 생성
- 대개 URL <sub>(오리진 포함)</sub> 요구
  - 서버 컨텍스트 내 사용 시

##### 내부 요청 <sub>(`+server.js routes` 등)</sub> → 핸들러 함수 <sub>(직행)</sub>
- 서버 내 실행 시
  - HTTP 호출 오버헤드 X

##### `fetch` <sub>(메서드)</sub> 동작 조작
```javascript
/* src/hooks.server.js */
export async function handleFetch({ event, request, fetch }) {
  return await fetch(request);
}
```

##### `src/routes/a/+server.js` 대상 요청
- `src/routes/b/+server.js` 응답
```javascript
/* src/hooks.server.js */
export async function handleFetch({ event, request, fetch }) {
  const url = new URL(request.url);
  if (url.pathname === '/a') {
    return await fetch('/b');
  }

  return await fetch(request);
}
```

### `handleError`
- 예상 불가능 에러 가로채기
- 특정 동작 발생
  - 에러 로깅 서비스에 데이터 전송
  - 기타 등등

##### 기본 기능
- 에러 로깅
```javascript
/* src/hooks.server.js */
export function handleError({ event, error }) {
  console.error(error.stack);
}
```

##### `/the-bad-place` <sub>(부재 페이지)</sub> 이동 시
- 에러 페이지 출력
- 터미널 내 메시지 출력 개체
```javascript
/* src/routes/the-bad-place/+page.server.js */
```

##### 에러 메시지 전송 X
- 민감한 정보 有
- 사용자 혼동
- 악의적 사용자 이용 가능

##### 에러 객체
- `$page.error`
  - `+error.svelte`
- `%sveltekit.error%`
  - `src/error.html` fallback
```javascript
/* 기본값 */
{
  message: 'Internal Error' // 500
//message: 'Not Found'      // 404
}
```

##### 커스터마이징
- `handleError` 내 에러 객체 반환
```javascript
/* src/hooks.server.js */
export function handleError({ event, error }) {
  console.error(error.stack);

  return {
    message: 'everything is fine',
    code: 'JEREMYBEARIMY'
  };
}
```
- 에러 객체 프로퍼티 접근
```html
<!-- src/routes/+error.svelte -->
<script>
  import { page } from '$app/stores';
</script>

<h1>{$page.status}</h1>
<p>{$page.error.message}</p>
<p>error code: {$page.error.code}</p>
```

<br />

## Page options

### Basics

|옵션|설명 <sub>(여부)</sub>|
|---|---|
|`ssr`|페이지 서버 렌더링|
|`csr`|SvelteKit 클라이언트 로드|
|`prerender`|페이지 선렌더링 <sub>(prerendering)</sub> 시점<br />- 빌드<br />- 요청 <sub>(X)</sub>|
|`trailingSlash`|URL 내 꼬다리 `/` <sub>(슬래시)</sub> 처리|

##### 옵션 적용
- 개별 페이지
  - `+page.js`
  - `+page.server.js`
- 페이지 그룹
  - `+layout.js`
  - `+layout.server.js`
- 앱 전체
  - 최상위 `layout[.server].js`
- 자식 `layout` · `page` 설정
  - 부모 `layout` 설정 override

### `ssr`

##### 서버측 HTML 렌더링
- sveltekit 기본 설정

##### 일부 컴포넌트 X
- `window` <sub>(전역 객체)</sub> 필요
  - 수정 필요 <sub>(서버 렌더링 가능하게)</sub>
- 기타 등등
  - 수정 불가능 시 `false` 설정
```javascript
/* src/routes/+page.server.js */
export const ssr = true · false;
```

### `csr`

##### 클라이언트측 HTML 렌더링
- 페이지 상호작용 강화
  - 버튼 클릭 시 카운터 증가
  - 기타 등등
- 이동 시 페이지 즉시 업데이트
  - 페이지 전체 로드 X
- `false` 설정 시
  - 해당 페이지 JS 동작 X
```javascript
/* src/routes/+page.server.js */
export const csr = true · false;
```

### `prerender`

##### 빌드 시 페이지 HTML 생성
- 요청 시 생성 X
- 부하 ↓ · 성능 ↑
- 다수 사용자 처리 시
  - `cache-control` <sub>(헤더)</sub> 걱정 X
    - 까다로운 사용법

##### 단점
- 빌드 시간 ↑
- 무거운 업데이트 적용
  - 새 버전 빌드 · 배포

#### 기본 규칙

##### 1. 아무 두 사용자 접근 시
- 서버로부터 동일 콘텐츠 얻기

##### 2. 페이지 내
- 폼 액션 X

##### 3. 동적 라우터 매개변수 有 페이지
- `prerender.entries` <sub>(설정)</sub>
  - 명시
  - 명시 페이지 링크 따라 도달 가능

```javascript
/* src/routes/+page.server.js */
export const prerender = true · false;
```

### `trailingSlash`

##### `/foo` vs `/foo/`
- `./foo` <sub>(상대경로)</sub>
  - `/foo`&nbsp;&nbsp; → `/foo`
  - `/foo/` → `/foo/bar`
- 검색엔진
  - 별개 취급 <sub>(SEO ↓)</sub>

##### SvelteKit 기본 설정
- 제거
  - `/foo/` → `/foo`
```javascript
/* 기본값 */
/* src/routes/always/+page.server.js */
export const trailingSlash = 'never';
```

##### 제거 방지 설정
```javascript
/* src/routes/always/+page.server.js */
export const trailingSlash = 'always';
```

##### SvelteKit 관여 방지 설정 <sub>(권장 X)</sub>
```javascript
/* src/routes/ignore/+page.server.js */
export const trailingSlash = 'ignore';
```

##### 선렌더링 <sub>(prerendering)</sub> 영향 끼침
- `/always/`
  - `always/index.html`
- `/never`
  - `never.html`

<br />

## Link options

### Preloading

#### `data-sveltekit-preload-data` <sub>(속성)</sub>

##### 탐색 시작 시점
- 마우스 위 이동 <sub>(hover)</sub>
- 탭 <sub>(터치)</sub> <sub>(모바일)</sub>

##### `<a>` <sub>(요소)</sub>
```html
<!-- src/routes/+layout.svelte -->
<nav>
  <a href="/">home</a>
  <a href="/slow-a" data-sveltekit-preload-data>slow-a</a>
  <a href="/slow-b">slow-b</a>
</nav>
```

##### 링크 포함 요소
```html
<!-- 기본 프로젝트 템플릿 -->
<body data-sveltekit-preload-data>
  %sveltekit.body%
</body>
```

##### 값 <sub>(커스터마이징)</sub>
- `'hover'` <sub>(기본값)</sub>
  - 모바일 → `'tap'`
- `'tap'`
  - 탭 <sub>(터치)</sub>
- `'off'`
  - 미작동

##### 부작용
- 해당 링크 이동 미발생

#### `data-sveltekit-preload-data` <sub>(속성)</sub>
- 라우터 필요 JS 코드만 로드

##### 값 <sub>(커스터마이징)</sub>
- `'eager'`
  - 탐색 후 미리 로드 <sub>(모든 내용)</sub>
- `'viewport'`
  - 뷰 포트 등장 시 미리 로드 <sub>(모드 내용)</sub>
- `'hover'` <sub>(기본값)</sub>
  - 모바일 → `'tap'`
- `'tap'`
  - 탭 <sub>(터치)</sub>
- `'off'`
  - 미작동

##### `preload[Code·Data]`
- 코드 이용해 미리 로딩
```javascript
import { preloadCode, preloadData } from '$app/navigation';

// 라우터 데이터 · 코드 미리 로드
preloadData('/foo');

// 라우터 코드 미리 로드
preloadCode('/bar');
```

### Reloading the page

##### 라우터 간 기본적 이동
- 페이지 재로드 X

##### `data-sveltekit-reload` <sub>(속성)</sub>
- `<a>` <sub>(속성)</sub>
- 링크 포함 속성
``` html
<!-- src/routes/+layout.svelte -->
<nav data-sveltekit-reload>
  <a href="/">home</a>
  <a href="/about">about</a>
</nav>
```

<br />

## Advanced routing

### Optional parameters

##### `[[…]]` <sub>(쌍대괄호)</sub>
- `src/routes/+page.svelte` <sub>(`/`)</sub>
- `src/routes/[[lang]]/+page.svelte` <sub>(`/`)</sub>
```javascript
/* src/routes/[[lang]]/+page.server.js */
const greetings = {
  en: 'hello!',
  de: 'hallo!',
  fr: 'bonjour!'
};

export function load({ params }) {
  return {
    greeting: greetings[params.lang ?? 'en']
  };
}
```

### Rest parameters

##### `[...rest]`
- `src/routes/[path]`
- `src/routes/[...path]`

##### 더 구체적인 타 라우터 먼저 테스트
- 'catch-all' 라우터
  - ex\) 커스텀 에러 <sub>(`404`)</sub> 페이지
    - `/categories/…` <sub>(하위 폴더)</sub>
```
src/routes/
├ categories/
│ ├ animal/
│ ├ mineral/
│ ├ vegetable/
│ ├ [...catchall]/    *
│ │ ├ +error.svelte   *
│ │ └ +page.server.js *

+page.server.js
- load (함수) 내 error(404) 던짐
```

##### 끝쪽 위치 필수 X
- `/items/[...path]/edit`
- `/items/[...path].json`
- 기타 등등

### Param matchers

##### 잘못된 입력 방지
- ex\) `/colors/[value]` <sub>(16진수)</sub>
  - `/colors/ff3e00` (O)
  - `/colors/orange` (X)
```javascript
/* src/params/hex.js */
export function match(value) {
  return /^[0-9a-f]{6}$/.test(value);
}
```
1. 함수 `export`
2. url 변경
```
src/routes/colors/[color]
src/routes/colors/[color=hex] (파일명)
```
- 서버 · 브라우저 작동

### Route groups

##### `…/(…)/…` <sub>(layout)</sub>
- 라우터 영향 X
- ex\) 접근 통제

##### `…/(authed)/`
- `…/(authed)/account`
- `…/(authed)/app`
- `…/(authed)/+layout[.server].js`
```javascript
/* src/routes/(authed)/+layout.server.js */
import { redirect } from '@sveltejs/kit';

export function load({ cookies, url }) {
  if (!cookies.get('logged_in')) {
    throw redirect(303, `/login?redirectTo=${url.pathname}`);
  }
}
```

##### 공통 UI 추가
- `…/(authed)/+layout.svelte`
```html
<!-- src/routes/(authed)/+layout.svelte -->
<slot></slot>

<form method="POST" action="/logout">
  <button>log out</button>
</form>
```

### Breaking out of layouts

##### 일반적 라우터 상속
- 모든 상위 layout 상속

##### `src/routes/a/b/c/+page.svelte`
- `src/routes/+layout.svelte`
- `src/routes/a/+layout.svelte`
- `src/routes/a/b/+layout.svelte`
- `src/routes/a/b/c/+layout.svelte`

##### `@…`
- 상속 규칙 깨기

##### `/a/b/c` <sub>(폴더 그룹)</sub> 상속 위치 변경
- `+page@b.svelte`
  - `src/routes/a/b/+layout.svelte`
- `+page@a.svelte`
  - `src/routes/a/+layout.svelte`
- `+page@.svelte`
  - `src/routes/+layout.svelte`
    - 최상위

<br />

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
