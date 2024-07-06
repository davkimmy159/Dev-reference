## Advanced loading

### Universal `load` <sub>(함수)</sub>

##### `+[page·layout].server.js`
- 서버 데이터 로드
  - DB 직접 접근
  - 쿠키 읽기
  - 기타 등등

##### 클라이언트측 탐색 시 서버 데이터 불필요 경우
- You're loading data from an external API
- You want to use in-memory data if it's available
- You want to delay navigation until an image has been preloaded, to avoid pop-in
- You need to return something from load that can't be serialized (SvelteKit uses devalue to turn server data into JSON), such as a component or a store

In this exercise, we're dealing with the latter case. The server `load` <sub>(함수)</sub> in `src/routes/red/+page.server.js`, `src/routes/green/+page.server.js` and `src/routes/blue/+page.server.js` return a `component` constructor, which can't be serialized like data. If you navigate to `/red`, `/green` or `/blue`, you'll see a 'Data returned from `load` ... is not serializable' error in the terminal.

To turn the server `load` <sub>(함수)</sub> into universal `load` <sub>(함수)</sub>, rename each `+page.server.js` file to `+page.js`. Now, the functions will run on the server during server-side rendering, but will also run in the browser when the app hydrates or the user performs a client-side navigation.

We can now use the `component` returned from these `load` <sub>(함수)</sub> like any other value, including in `src/routes/+layout.svelte`:

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

Read the documentation to learn more about the distinction between server `load` <sub>(함수)</sub> and universal load functions, and when to use which.

### Using both `load` <sub>(함수)</sub>
Occasionally, you might need to use a server load function and a universal load function together. For example, you might need to return data from the server, but also return a value that can't be serialized as server data.

In this example we want to return a different component from `load` depending on whether the data we got from `src/routes/+page.server.js` is `cool` or not.

We can access server data in `src/routes/+page.js` via the data property:
```javascript
src/routes/+page.js
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

##### 참고
Note that the data isn't merged — we must explicitly return `message` from the universal `load` <sub>(함수)</sub>.

### Using parent data
As we saw in the introduction to layout data, `+page.svelte` and `+layout.svelte` components have access to everything returned from their parent `load` <sub>(함수)</sub>.

Occasionally it's useful for the `load` <sub>(함수)</sub> themselves to access data from their parents. This can be done with `await parent()`.

To show how it works, we'll sum two numbers that come from different `load` <sub>(함수)</sub>. First, return some data from `src/routes/+layout.server.js`:
```javascript
/* src/routes/+layout.server.js */
export function load() {
  return { a: 1 };
}
```

Then, get that data in `src/routes/sum/+layout.js`:
```javascript
src/routes/sum/+layout.js
export async function load({ parent }) {
  const { a } = await parent();
  return { b: a + 1 };
}
```

##### 참고
Notice that a universal `load` <sub>(함수)</sub> can get data from a parent server load function. The reverse is not true — a server `load` <sub>(함수)</sub> can only get parent data from another server load function.

Finally, `in src/routes/sum/+page.js`, get parent data from both `load` <sub>(함수)</sub>:
```javascript
src/routes/sum/+page.js
export async function load({ parent }) {
  const { a, b } = await parent();
  return { c: a + b };
}
```

##### 참고
Take care not to introduce waterfalls when using `await parent()`. If you can `fetch` other data that is not dependent on parent data, do that first.

### Invalidation
When the user navigates from one page to another, SvelteKit calls your load functions, but only if it thinks something has changed.

In this example, navigating between timezones causes the `load` <sub>(함수)</sub> in `src/routes/[...timezone]/+page.js` to re-run because `params.timezone` is invalid. But the `load` <sub>(함수)</sub> in `src/routes/+layout.js` does not re-run, because as far as SvelteKit is concerned it wasn't invalidated by the navigation.

We can fix that by manually invalidating it using the invalidate(...) function, which takes a URL and re-runs any `load` <sub>(함수)</sub> that depend on it. Because the `load` <sub>(함수)</sub> in `src/routes/+layout.js` calls `fetch('/api/now')`, it depends on `/api/now.`

In `src/routes/[...timezone]/+page.svelte`, add an `onMount` callback that calls `invalidate('/api/now')` once a second:
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

