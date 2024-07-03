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
- `data` <sub>(프로퍼티)</sub>
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
- 기본 설정
```javascript
{
  httpOnly: true,
  secure: true,
  sameSite: 'lax'
}
```

<br />

## Shared modules

##### `$lib` <sub>(별칭)</sub>
- `src/lib` <sub>(폴더)</sub> 참조
  - 전역 접근
```html
<script>
//import { … } from '../../../lib/….js';
  import { … } from '$lib/….js';
</script>
```

<br />

## Forms

### `<form>` <sub>(요소)</sub>
- 페이지 자동 리로드
- `fetch` <sub>(메서드)</sub> X
```html
<!-- src/routes/+page.svelte -->
<form method="POST">
  <label>
    …
    <input name="description" />
  </label>
</form>
```
```javascript
/* src/routes/+page.server.js */
import * as db from '$lib/server/database.js';

export function load({ cookies }) { … }

export const actions = {

  // 표준 request (객체)
  default: async ({ cookies, request }) => {
    const data = await request.formData();
    db.createTodo(cookies.get('userid'), data.get('description'));
  }
};
```

### Named form actions

##### 대다수 페이지
- 다수 액션 有
```javascript
/* src/routes/+page.server.js */
export const actions = {

//default
// - 동시 사용 X

  create: async ({ cookies, request }) => {
    const data = await request.formData();
    db.createTodo(cookies.get('userid'), data.get('description'));
  },

  delete: async ({ cookies, request }) => {
    const data = await request.formData();
    db.deleteTodo(cookies.get('userid'), data.get('id'));
  }
};
```
##### `action` <sub>(속성)</sub>
- `href` <sub>(`a` 요소 속성)</sub> 유사
- 모든 URL 가능
  - ex\) `/todos?/create`
```html
<!-- src/routes/+page.svelte -->
<form method="POST" action="?/create">
  <label>
    …
    <input name="description" />
  </label>
</form>

<ul>
  {#each data.todos as todo (todo.id)}
    <li>
      <form method="POST" action="?/delete">
        <input type="hidden" name="id" value={todo.id} />
        <span>{todo.description}</span>
        <button aria-label="Mark as complete"></button>
      </form>
    </li>
  {/each}
</ul>
```

### Validation

##### 클라이언트측
- `required` <sub>(`input` 요소 속성)</sub>
- 기타 등등
```html
<!-- src/routes/+page.svelte -->
<form method="POST" action="?/create">
  <label>
    …
    <input name="description" required />
  </label>
</form>
```

##### 서버측
```javascript
/* src/lib/server/database.js */
export function createTodo(userid, description) {

  // 빈 내용 검사
  if (description === '') {
    throw new Error('todo must have a description');
  }

  const todos = db.get(userid);

  // 중복 내용 검사
  if (todos.find((todo) => todo.description === description)) {
    throw new Error('todos must be unique');
  }

  todos.push({
    id: crypto.randomUUID(),
    description,
    done: false
  });
}
```
- 에러 발생 시
  - 클라이언트 상세 에러 화면 X
- `fail` <sub>(함수)</sub> 호출
  - 데이터 · HTTP 상태 코드 반환
```javascript
/* src/routes/+page.server.js */
import { fail } from '@sveltejs/kit';
import * as db from '$lib/server/database.js';

export function load({ cookies }) { … }

export const actions = {
  create: async ({ cookies, request }) => {
    const data = await request.formData();

    try {
      db.createTodo(cookies.get('userid'), data.get('description'));
    } catch (error) {
      return fail(422, {
        description: data.get('description'),
        error: error.message
      });
    }
  },
  delete: …
}
```
- 반환값
  - `form` <sub>(프로퍼티)</sub>
```html
<!-- src/routes/+page.svelte -->
<script>
  export let data;

  // 반환값
  export let form;
</script>

<div>
  {#if form?.error}
    <p>{form.error}</p>
  {/if}

  <form method="POST" action="?/create">
    <label>
      …
      <input
        name="description"
        value={form?.description ?? ''}
        required
      />
    </label>
  </form>
```

### Progressive enhancement

##### 브라우저 내장 행동 흉내내기
- `form` <sub>(프로퍼티)</sub> 업데이트
- 성공적 응답 데이터 무효화
  - `load` <sub>(함수)</sub> 재실행
- `redirect` 응답 시
  - 새 페이지 이동
- 에러 발생 시
  - 최근접 에러 페이지 렌더링
```html
<!-- src/routes/+page.svelte -->
<script>
  import { enhance } from '$app/forms';

  export let data;
  export let form;
</script>
…
<form method="POST" action="?/create" use:enhance>
…
<form method="POST" action="?/delete" use:enhance>
…
```

### Customizing `use:enhance`

##### 콜백 제공
- 상태 보류
- 긍정적 UI
- 기타 등등
```html
src/routes/+page.svelte
<script>
  …
  let creating = false;
  let deleting = [];

  function ehanceCreate() {
    creating = true;

    return async ({ update }) => {
      await update();
      creating = false;
    };
  }

  function handleDelete() {
    deleting = [...deleting, todo.id];
    return async ({ update }) => {
      await update();
      deleting = deleting.filter((id) => id !== todo.id);
    };
  }
</script>
…
<form
  method="POST"
  action="?/create"
  use:enhance={ehanceCreate}
>
  <label>
    …
    <input
      disabled={creating}
      name="description"
      value={form?.description ?? ''}
      required
    />
  </label>
</form>
…
<ul>
  {#each data.todos.filter((todo) => !deleting.includes(todo.id)) as todo (todo.id)}
    <li in:fly={{ y: 20 }} out:slide>
      <form
        method="POST"
        action="?/delete"
        use:enhance={handleDelete}
      >
        <input type="hidden" name="id" value={todo.id} />
        <button aria-label="Mark as complete">✔</button>

        {todo.description}
      </form>
    </li>
  {/each}
</ul>

{#if creating}
  <span>saving…</span>
{/if}
```

##### 상당한 커스터마이징 기능
- 문서 참조

<br />

## API routes
- `+server.js`
  - API routes

### `GET` handlers
```javascript
/* src/routes/roll/+server.js */
export function GET() {
  const number = Math.floor(Math.random() * 6) + 1;

  /*
  return new Response(number, {
    headers: {
      'Content-Type': 'application/json'
    }
  });
  */
  return json(number);
}
```
```html
<script>
  let number;

  async function roll() {
    const response = await fetch('/roll');
    number = await response.json();
  }
</script>

<button on:click={roll}>Roll the dice</button>

```
### `POST` handlers

##### `<input>` <sub>(요소)</sub> `keydown` 핸들러
```html
<!-- src/routes/+page.svelte -->
<script>
  function async handler(e) {
    if (e.key !== 'Enter') return;

    const input = e.currentTarget;
    const description = input.value;

    const response = await fetch('/todo', {
      method: 'POST',
      body: JSON.stringify({ description }),
      headers: {
        'Content-Type': 'application/json'
      }
    });

    const { id } = await response.json();

    data.todos = [...data.todos, {
      id,
      description
    }];

    input.value = '';
  }
</script>
…
<input
  type="text"
  on:keydown={handler}
/>
```
```javascript
/* src/routes/todo/+server.js */
import { json } from '@sveltejs/kit';
import * as database from '$lib/server/database.js';

export async function POST({ request, cookies }) {
  const { description } = await request.json();

  const userid = cookies.get('userid');
  const { id } = await database.createTodo({ userid, description });

  return json({ id }, { status: 201 });
}
```
##### `data` <sub>(프로퍼티)</sub> 변경 방식
- 페이지 재로드 시 동일 결과

### `Other` handlers
```javascript
/* src/routes/todo/[id]/+server.js */
import * as database from '$lib/server/database.js';

export async function PUT({ params, request, cookies }) {
  const { done } = await request.json();
  const userid = cookies.get('userid');

  await database.toggleTodo({ userid, id: params.id, done });
  return new Response(null, { status: 204 });
}

export async function DELETE({ params, cookies }) {
  const userid = cookies.get('userid');

  await database.deleteTodo({ userid, id: params.id });
  return new Response(null, { status: 204 });
}
```
```html
<!-- src/routes/+page.svelte -->
<script>
  async function onChange(e) {
    const done = e.currentTarget.checked;

    await fetch(`/todo/${todo.id}`, {
      method: 'PUT',
      body: JSON.stringify({ done }),
      headers: {
        'Content-Type': 'application/json'
      }
    });
  }

  async function onClick(e) {
    await fetch(`/todo/${todo.id}`, {
      method: 'DELETE'
    });

    data.todos = data.todos.filter((t) => t !== todo);
  }
</script>

<label>
  <input
    type="checkbox"
    checked={todo.done}
    on:change={onChange}
  />
  <span>{todo.description}</span>
  <button on:click={onClick} ></button>
</label>
```

<br />

## Stores <sub>(`readonly`)</sub>

### page <sub>(자주 사용)</sub>

##### 현재 페이지 정보 제공
- `url`
  - URL
- `params`
  - 파라미터
- `route`
  - 객체 <sub>(`id` 프로퍼티 有)</sub>
    - 현재 라우터
- `status`
  - HTTP 상태 코드
- `error`
  - 에러 객체
- `data`
  - 데이터
  - `load` <subP>(함수)</subP> 반환값 포함
- `form`
  - `form` 액션 반환값
```html
<!-- src/routes/+layout.svelte -->
<script>
  import { page } from '$app/stores';
</script>

<nav>
  <a href="/" aria-current={$page.url.pathname === '/'}>
    home
  </a>
  <a href="/about" aria-current={$page.url.pathname === '/about'}>
    about
  </a>
</nav>

<slot></slot>
```

### navigating


### updated


<br />

## Errors · redirects

### Basics


### Error pages


### Fallback errors


### Redirects
