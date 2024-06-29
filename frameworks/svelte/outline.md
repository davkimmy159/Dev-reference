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

##### Lifecycle
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

##### 부모 · 자식
```javascript
/* 부모 */
import { onMount, onDestroy, beforeUpdate, afterUpdate, tick } from 'svelte'

onMount(async() => { console.log('onMount') });
onDestroy(async() => { console.log('onDestroy') });
beforeUpdate(async() => { console.log('beforeUpdate') });
afterUpdate(async() => { console.log('afterUpdate') });
```

```javascript
/* 자식 */
import { onMount, onDestroy, beforeUpdate, afterUpdate, tick } from 'svelte'

onMount(async() => { console.log('onMount') });
onDestroy(async() => { console.log('onDestroy') });
beforeUpdate(async() => { console.log('beforeUpdate') });
afterUpdate(async() => { console.log('afterUpdate') });
```

|순서|개체|생명주기|
|---|---|---|
|1|부모|`beforeUpdate`|
|2|자식|`beforeUpdate`|
|3|자식|`onMount`|
|4|자식|`afterUpdate`|
|5|부모|`onMount`|
|6|부모|`afterUpdate`|

##### `tick` <sub>(함수)</sub>
- 모든 돔 렌더링 후 작업
  - 라이프사이클 함수 상태 별개
```javascript
/* 자식 */
import { onMount, onDestroy, beforeUpdate, afterUpdate, tick } from 'svelte'

onMount(async() => { console.log('onMount') });
onDestroy(async() => { console.log('onDestroy') });
beforeUpdate(async() => {
  console.log('about to update');
  await tick()
  console.log('just updated');
});
afterUpdate(async() => { console.log('afterUpdate') });
```

##### await tick() X

|순서|개체|생명주기|
|---|---|---|
|1|부모|`beforeUpdate`|
|2|자식|`"about to update"`|
|3|자식|`"just updated"`|
|4|자식|`onMount`|
|5|자식|`afterUpdate`|
|6|부모|`onMount`|
|7|부모|`afterUpdate`|

##### await tick() O

|순서|개체|생명주기|
|---|---|---|
|1|부모|`beforeUpdate`|
|2|자식|`"about to update"`|
|3|자식|`onMount`|
|4|자식|`afterUpdate`|
|5|부모|`onMount`|
|6|부모|`afterUpdate`|
|7|자식|`"just updated"`|
