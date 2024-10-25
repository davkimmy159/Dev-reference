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
prisma db push
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
