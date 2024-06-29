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
export const store = readable("initialValue", function start(set) {
  count interval = setInterval(() => {
    set(…);
  }, 1000);
});

return function stop() {
  clearInterval(interval);
}
```
