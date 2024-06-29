Lifecycle
=========

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
