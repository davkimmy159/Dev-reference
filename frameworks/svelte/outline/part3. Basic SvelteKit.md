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

- 괄호 내 다수 파라미터 가능
  - 구분 <sub>(글자 1개)</sub> 필수
    - ex\) `…/[bar]x[baz]/…`

<br />

## Loading data


<br />

## Headers · Cookies


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
