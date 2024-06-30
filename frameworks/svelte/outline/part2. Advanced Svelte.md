Advanced Svelte
================

### Motion

### Transitions

### Animations

### Actions

#### `use` <sub>(directive)</sub>

##### element-level lifecycle 함수
- 서드파티 라이브러리 접속기
- lazy-loaded 이미지
- 툴팁
- 커스텀 이벤트 핸들러 추가
```javascript
// node (1번째 인수)
export function func(node) {
  node.addEventListener('event', handler);
  return {
    destroy() {
      node.removeEventListener('event', handler);
      …
    }
  };
}
```
```html
<div class="menu" use:func>…</div>
```

#### Adding parameters

##### 추가 인수 <sub>(2번째)</sub>
```javascript
// params (2번째 인수)
export function func(node, params) {
  …
  return {
    update(param) {
      …
    }
    destroy() {
      …
    }
  };
}
```
```html
<button use:func={params}>…</button>
<button use:func={{ a, b: 'B', … }}>…</button>
```

### Advanced bindings

#### Contenteditable bindings

##### `textContent` · `innerHTML` <sub>(속성)</sub>
```html
<div bind:innerHTML={html} contenteditable></div>
```

#### Each block bindings

##### `input` <sub>(요소)</sub>
- mutable
  - 데이터 변경 발생
- immutable 필요 시
  - 이벤트 핸들러 사용
```html
{#each b as a}
  <li>
    <input
      type="checkbox"
      bind:checked={a.checked}
    />
    <input
      type="text"
      bind:value={a.value}
    />
  </li>
{/each}
```

#### Media elements

##### `<audio>` · `<video>` <sub>(요소)</sub>
```html
<script>
  export let src;

  let time = 0;
  let duration = 0;
  let paused = true;
</script>

<div class="player" class:paused>
  <audio
    {src}
    bind:currentTime={time}
    bind:duration
    bind:paused
    on:ended={() => {
      time = 0;
    }}
  ></audio>

  <button
    class="play"
    aria-label={paused ? 'play' : 'pause'}
    on:click={() => paused = !paused}
  ></button>
</div>
```

##### readonly

|프로퍼티|값|
|---|---|
|duration|길이 <sub>(초)</sub>|
|buffered| 배열 <sub>(`{start, end}` 객체)</sub>|
|seekable|상동|
|played|상동|
|seeking|`boolean`|
|ended|`boolean`|
|readyState|`0` ~ `4`|

##### two-way

|프로퍼티|값|
|---|---|
|currentTime|현재 위치 <sub>(초)</sub>|
|playbackRate|실행 속도 <sub>(`1` : 정상)</sub>|
|paused|멈춤|
|volume|`0` ~ `1`|
|muted|`boolean`|

#### Dimensions

##### block-level 요소 <sub>(readonly · 값 변경 영향 X)</sub>
- `client[Width·Height]`
- `offset[Width·Height]`
```html
<script>
  let w, h;
</script>

<div bind:clientWidth={w} bind:clientHeight={h}>…</div>
```
##### `display: inline` · 단일 <sub>(요소)</sub>
- 레퍼 요소 필요

#### `this`
```html
<script>
  let canvas;
</script>

<canvas bind:this={canvas}>…</canvas>
```

#### Component bindings
- 자주 사용 X
  - 데이터 흐름 추적 어려움
```html
<!-- Parent -->
<script>
  let pValue;
</script>
<Child bind:cValue={pValue} />
```
```html
<!-- Child -->
<script>
  export let cValue = '';
</script>
```

#### Binding to component instances
```html
<!-- App -->
<script>
  let canvas;
</script>

<Canvas bind:this={canvas}>…</Canvas>
<button on:click={() => canvas.cFunc()}>…</button>
```
```html
<!-- Canvas -->
<script>
  export function cFunc() { … }
</script>
```

### Classes and styles

#### Ths class directive
```html
<script>
  let flipped = false;
</script>

<button
  class="card {flipped ? 'flipped' : ''}"
  on:click={() => flipped = !flipped}
>
<button
  class="card"
  class:flipped={flipped}
  on:click={() => flipped = !flipped}
>

<!-- Shorthand -->
<button
  class="card"
  class:flipped
  on:click={() => flipped = !flipped}
>
```

#### The style directive
```html
<button
  style="transform: {flipped ? 'rotateY(0)' : ''}; --bg-1: palegoldenrod; --bg-2: black; --bg-3: goldenrod"
>
<button
  style:transform={flipped ? 'rotateY(0)' : ''}
  style:--bg-1="palegoldenrod"
  style:--bg-2="black"
  style:--bg-3="goldenrod"
>
```

#### Component styles

##### 전역
- 비효율
- 불안정
```html
<style>
  .boxes :global(.box:nth-child(1)) { background-color: red; }
  .boxes :global(.box:nth-child(2)) { background-color: green; }
  .boxes :global(.box:nth-child(3)) { background-color: blue; }
</style>
```

##### 컴포넌트 자체
```html
<!-- Box.svlete -->
<style>
  .box {
    background-color: var(--color, #ddd);
  }
</style>
```
```html
<!-- App.svelte -->
<div class="boxes">
  <Box --color="red" />
  <Box --color="green" />
  <Box --color="blue" />
</div>
```

### Component composition

#### Slots

##### Default slot
```html
<!-- Card.svelte -->
<div class="card">
  <slot></slot>
</div>
```
```html
<!-- App.svelte -->
<Card>
  <span>Patrick BATEMAN</span>
  <span>Vice President</span>
</Card>
```

#### Named slots
```html
<!-- Card.svelte -->
<div class="card">
  <header>
    <slot name="slot1"></slot>
    <slot name="slot2"></slot>
  </header>

  <slot></slot>

  <footer>
    <slot name="slot3"></slot>
  </footer>
</div>
```
```html
<!-- App.svelte -->
<Card>
  <span>…</span>
  <span>…</span>

  <span slot="slot1">212 555 6342</span>

  <span slot="slot2">
    Pierce &amp; Pierce
    <small>Mergers and Aquisitions</small>
  </span>

  <span slot="slot3">358 Exchange Place, New York, N.Y. 10099 fax 212 555 6390 telex 10 4534</span>
</Card>
```

##### styles <sub>(2가지)</sub>
1. 정의 요소 위치
```html
<!-- App.svelte -->
<Card>
  …
  <small>Mergers and Aquisitions</small>
  …
</Card>

<style>
  small {
    display: block;
    font-size: 0.6em;
    text-align: right;
  }
</style>
```
2. 포함 요소 위치 <sub>(전역)</sub>
```html
<!-- Card.svelte -->
<div class="card">
  …
</div>

<style>
  .card :global(small) {
    display: block;
    font-size: 0.6em;
    text-align: right;
  }
</style>
```

#### Slot fallbacks

##### 부재 시 기본값
```html
<!-- Card.svelte -->
<div class="card">
  <slot name="telephone">
    <i>(telephone)</i>
  </slot>

  <slot name="company">
    <i>(company name)</i>
  </slot>

  <slot>
    <i>(name)</i>
  </slot>

  <slot name="address">
    <i>(address)</i>
  </slot>
</div>
```
```html
<!-- App.svelte -->
<Card />
```

#### Slot props
```html
<!-- Child.svelte -->
{#each items as item}
  <slot {item} />
  <slot item={item} />
{/each}
```
```html
<!-- App.svelte -->
<Child let:item={row}>
  <span>{row.a}</span>
  <span>{row.b}</span>
  <span>{row.c}</span>
  <span>{row.d}</span>
</Child>
```

#### Checking for slot content

##### `$$slots`
```html
<!-- Child.svelte -->
{#if $$slots.slot1}
  <div>
    <slot name="slot1"></slot>
  </div>
{/if}
<div>
  <slot name="slot2"></slot>
</div>
```
```html
<!-- App.svelte -->
<div slot="slot2">…</div>
```

### Context API

#### \[s·g\]etContext


### Special elements


### Module context


### Miscellaneous


### Next steps
