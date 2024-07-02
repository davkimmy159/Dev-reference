Basic SvelteKit
=============

## Introduction

### 서버사이드 렌더링 <sub>(기본)</sub>
- Routing
- Server-side rendering
- Data fetching
- Service workers
- TypeScript integration
- Prerendering
- Single-page apps
- Library packaging
- Optimised production builds
- Deploying to different hosting providers
- 기타 등등

<br />

## Routing

### Pages

##### `+page.svelte` <sub>(`src/routes` 폴더)</sub>
```
src/routes/
├ about/
│ └ +page.svelte
└ +page.svelte
```
- `src/routes/+page.svelte`
  - `/`
- `src/routes/about/+page.svelte`
  - `/about`
```html
<!-- src/routes/+page.svelte -->
<nav>
  <a href="/">home</a>
  <a href="/about">about</a>
</nav>

<h1>home</h1>
<p>this is the home page.</p>
```
```html
<!-- src/routes/about/+page.svelte -->
<nav>
  <a href="/">home</a>
  <a href="/about">about</a>
</nav>

<h1>about</h1>
<p>this is the about page.</p>
```

### Layouts

##### `+layout.svelte` <sub>(`src/routes` 폴더)</sub>
- 폴더 내 모든 `+page.svelte` 적용
  - 하위 폴더
  - 형제 `+page.svelte`
- 중복 UI 담당
- 계층 구조 가능
```
src/routes/
├ about/
│ └ +page.svelte
├ +layout.svelte
└ +page.svelte
```
```html
<!-- src/routes/+layout.svelte -->
<nav>
  <a href="/">home</a>
  <a href="/about">about</a>
</nav>

<slot></slot>
```
```html
<!-- src/routes/+page.svelte -->
<h1>home</h1>
<p>this is the home page.</p>
```
```html
<!-- src/routes/about/+page.svelte -->
<h1>about</h1>
<p>this is the about page.</p>
```

### Route parameters

##### `…/[…]/…`
- `src/routes/blog/[slug]/+page.svelte`
  - `/blog/one`
  - `/blog/two`
  - `/blog/three`
  - 기타 등등

##### `[]` <sub>(대괄호)</sub> 내 다수 파라미터 가능
- 구분 <sub>(글자 1개)</sub> 필수
  - ex\) `…/[bar]x[baz]/…`

<br />

## Loading data

### Page data

##### `+page.server.js`
- `load` <sub>(함수)</sub>
- 실행
  - 서버측
  - 클라이언트측 네비게이션
- 하위 라우터 적용 X
```
src/routes/
├ blog/
│ ├ [slug]/
│ │ ├ +layout.svelte
│ │ ├ +page.server.js *
│ │ └ +page.svelte
│ ├ +page.server.js   *
│ ├ +page.svelte
│ └ data.js
├ +layout.svelte
└ +page.svelte
```
```javascript
/* src/routes/blog/+page.server.js */
import { posts } from './data.js';

export function load() {
  return {
 // summaries: posts
    summaries: posts.map((post) => ({
      slug: post.slug,
      title: post.title
    }))
  };
}
```
```html
<!-- src/routes/blog/+page.svelte -->
<script>
  // 데이터 접근 프로퍼티
  export let data;
</script>

<h1>blog</h1>

<ul>
  {#each data.summaries as { slug, title }}
    <li><a href="/blog/{slug}">{title}</a></li>
  {/each}
</ul>
```
```javascript
/* src/routes/blog/[slug]/+page.server.js */
import { error } from '@sveltejs/kit';
import { posts } from '../data.js';

export function load({ params }) {
  const post = posts.find((post) => post.slug === params.slug);

  // 부재 주소 접근 방지
  // - ex) "/blog/nope"
  if (!post) throw error(404);

  return {
    post
  };
}
```
```html
<!-- src/routes/blog/[slug]/+page.svelte -->
<script>
  export let data;
</script>

<h1>blog post</h1>
<h1>{data.post.title}</h1>
<div>{@html data.post.content}</div>
```

### Layout data

##### `+layout.server.js`
- 모든 하위 layout · 라우터 적용
  - `+layout.svelte` 유사
```
src/routes/
├ blog/
│ ├ [slug]/
│ │ ├ +layout.svelte
│ │ ├ +page.server.js
│ │ └ +page.svelte
│ ├ +page.server.js   X
│ ├ +layout.server.js *
│ ├ +page.svelte
│ └ data.js
├ +layout.svelte
└ +page.svelte
```
```html
<script>
  export let data;
</script>

<div>
  <main>
    <slot></slot>
  </main>

  <aside>
    <ul>
      <!-- data.[summaries·slug·title] -->
      {#each data.summaries as { slug, title }}
        <li>
          <a href="/blog/{slug}">{title}</a>
        </li>
      {/each}
    </ul>
</div>
```

<br />

## Headers · Cookies

### Setting headers

##### `setHeaders` <sub>(콜백)</sub>
- `load` <sub>(함수)</sub> 안
```javascript
/* src/routes/+page.server.js */
export function load({ setHeaders }) {
	setHeaders({
		'Content-Type': 'text/plain'
	});
}
```

### Reading · writing cookies

##### `setHeaders` <sub>(콜백)</sub>
- `Set-Cookie` <sub>(헤더)</sub> 함께 사용 X
  - `cookies` 사용

##### 쿠키 읽기
- `cookies.get(name, options)`
```javascript
/* src/routes/+page.server.js */
export function load({ cookies }) {
	const visited = cookies.get('visited');

	return {
		visited: visited === 'true'
	};
}
```

##### 쿠키 쓰기
- `cookies.set(name, value, options)`
  - `path` <sub>(옵션)</sub> 설정 권장
  - `Set-Cookie` <sub>(헤더)</sub> 쓰기 · 수정
  - 동일 요청 내 호출
    - 상태 업데이트
```javascript
/* src/routes/+page.server.js */
export function load({ cookies }) {
	const visited = cookies.get('visited');

	cookies.set('visited', 'true', { path: '/' });

	return {
		visited: visited === 'true'
	};
}
```

<br />

## Shared modules


<br />

## Forms


<br />

## API routes


<br />

## Stores


<br />

## Errors · redirects
