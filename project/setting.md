##### corepack
```
install -g corepack
corepack enable
```

##### pnpm
```
pnpm init
pnpm add [-D]
pnpm tsc --init
pnpm run
```

##### prisma
```
prisma init
prisma db pull [--force]
```

#### commonJS

##### `exports` vs `module.exports`
- `module.exports` 권장
```javascript
module.exports = exports = class Square {
  /* … */
}
```

#### VSCode
- `.prettierignore`
  - Top-level
