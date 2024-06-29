outline
========

### Part1 <sub>(Basic Svelte)</sub>

#### Introduction
- `{src}`
- `{@html … }`

#### Reactivity
- `$: doubled = …`
- `$: console.log( … )`
- `$: { … }`
- `$: if () { … }`
- 할당 반응
  - 배열 메서드 X

#### Props
- `export (default)`
- `{...props}`
- `$$props`

#### Logic
- `{#if … }`
- `{:else if … }`
- `{:else}`
- `{/if}`
- `{#each a as b, i ((b.id)}`
- `{/each}`
- `{#await promise}`
- `{:then … }`
- `{:catch … }`
- `{/await}`
- `{#await promise then … }`
- `{/await}`

#### DOM events
- `<div on:click={ … }>`
- `<div on:click={(e) => { … }}>`
- `<div on:click|once|capture={ … }>`
  - `preventDefault`
  - `stopPropagation`
  - `passive`
  - `nonpassive`
  - `capture`
  - `once`
  - `self`
  - `trusted`
- `createEventDispatcher()`
- `<Inner on:message />`
- `<button on:click>`

#### Bindings
- `<input bind:value={a}>`
- `<input type="[number|range]" bind:value={a} />`
- `<input type="checkbox" bind:checked={a} />`
- `<select bind:value={a}>{a?.id}`
- `<input	type="[radio·checkbox]" bind:group={a} />`
- `<select multiple bind:value={a}>`
- `<textarea bind:value>`

#### Lifecycle
```javascript
onMount(() => {
  return () => { … };
});
```
```javascript
beforeUpdate(() => { … });
afterUpdate(() => { … });
```
```javascript
await tick();
```

#### Stores

##### `writable`
- set
- update
- subscribe
```javascript
export const store = writable(…);

const unsubscribe = count.subscribe((v) => { … });
onDestroy(unsubscribe);

// auto-subscription (top-level scope)
{$count}

// {$…}
// - store value 가정
```

##### `readable`
- 인수
  - 초기값 <sub>(1번째)</sub>
  - `start` <sub>(2번째 · 함수)</sub>
      - `set` <sub>(인수 · 콜백)</sub>
    - 첫 subscribe 시 시작
- 반환값
  - `stop` <sub>(함수)</sub>
    - 끝 subscribe 종료 시 시작
```javascript
export const store = readable("initial", function start(set) {
  count interval = setInterval(() => {
    set(…);
  }, 1000);
});

return function stop() {
  clearInterval(interval);
}
```

##### `derived`
- 인수
  - store <sub>(1번째)</sub>
  - 함수 <sub>(2번째)</sub>
    - 값 조작
    - store <sub>(`$…`)</sub> 전달
```javascript
export const derived = derived(
	time,
	($time) => Math.round(($time - start) / 1000)
);
```

##### Custom stores
- `subscribe` <sub>(메서드)</sub> 구현
```javascript
function createCount() {
	const { subscribe, set, update } = writable(0);

	return {
		subscribe,
		increment: () => update((n) => n + 1),
		decrement: () => update((n) => n - 1),
		reset: () => set(0)
	};
}
```

##### Store bindings
- `writable`
```javascript
export const wStore = writable(…);
export const dStore = derived(store, ($store) => …);
```
```html
<h1>{$dStore}</h1>
<input bind:value={$wStore} />
<button on:click={() => wStore …}></button>
```

### Part2 <sub>(Advanced Svelte)</sub>

#### Motion

#### Transitions

#### Animations

#### Actions

##### The use directive
- element-level lifecycle 함수
  - 서드파티 라이브러리 접속기
  - lazy-loaded 이미지
  - 툴팁
  - 커스텀 이벤트 핸들러 추가

#### Advanced bindings

#### Classes and styles

#### Component composition

#### Context API

#### Special elements

#### Module context

#### Miscellaneous

#### Next steps

### Part3 <sub>(Basic SvelteKit)</sub>

### Part4 <sub>(Advanced SvelteKit)</sub>