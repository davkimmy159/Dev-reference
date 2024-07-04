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
    - 클라이언트측 네비게이션 도중
- `false`
  - 페이지 · 라우터 직접 요청되어짐

##### `locals`
- 임의 데이터 보관 위치

##### `params`
- 라우터 파라미터

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

### `handleError`


<br />

## Page options

### Basics


### `ssr`


### `csr`


### `prerender`


### `trailingSlash`


<br />

## Link options

### Preloading


### Reloading the page


<br />

## Advanced routing

### Optional parameters


### Rest parameters


### Param matchers


### Route groups


### Breaking out of layouts


<br />

## Advanced loading

### Universal `load` <sub>(함수)</sub>


### Using both `load` <sub>(함수)</sub>


### Invalidation


### Custom dependencies


### `invalidateAll`


<br />

## Environment variables

### `$env/static/private`


### `$env/dynamic/private`


### `$env/static/public`


### `$env/dynamic/public`

