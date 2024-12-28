## 요소 간 상대적 깊이

### `position` 값 기준

#### `z-index` 無 <sub>(기본 규칙)</sub>

##### `static` vs `static` 외

|`position`|깊이|
|:---:|:---:|
|`static` 외|↑|
|`static`|↓|

##### `static` vs `static`

|문서 상 순서|깊이|
|:---:|:---:|
|나중|↑|
|먼저|↓|

##### `static` 외 vs `static` 외
|문서 상 순서|깊이|
|:---:|:---:|
|나중|↑|
|먼저|↓|

#### `z-index` 有 <sub>(기본 규칙 무시)</sub>

|값|깊이|
|:---:|:---:|
|양수|`static` ↑|
|음수|`static` ↓|

##### `static`
- `0` <sub>(`z-index: auto`)</sub> 고정

##### `static` 외 vs `static` 외
- `z-index` 값 크기

|양수 <sub>(비례)</sub>|깊이|
|:---:|:---:|
|`2`|↑|
|`1`|↓|

|음수 <sub>(반비례)</sub>|깊이|
|:---:|:---:|
|`-1`|↑|
|`-2`|↓|
