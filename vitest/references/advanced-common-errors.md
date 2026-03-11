---
name: common-errors
description: Common Vitest errors and their solutions
---

# Common Errors

## Cannot find module './relative-path'

**Cause:** Using `baseUrl` in `tsconfig.json` for path resolution. Vite doesn't resolve TypeScript path aliases by default.

**Fix:** Install `vite-tsconfig-paths`:

```bash
npm i -D vite-tsconfig-paths
```

```ts
import { defineConfig } from 'vitest/config'
import tsconfigPaths from 'vite-tsconfig-paths'

export default defineConfig({
  plugins: [tsconfigPaths()],
})
```

Or configure aliases manually:

```ts
export default defineConfig({
  test: {
    alias: {
      '@/': new URL('./src/', import.meta.url).pathname,
    },
  },
})
```

## Failed to Terminate Worker

**Cause:** Node.js `fetch` with `pool: 'threads'` can leave background processes running, preventing worker cleanup.

**Fix:** Switch to `forks` pool:

```ts
export default defineConfig({
  test: {
    pool: 'forks',
  },
})
```

Or via CLI: `vitest --pool=forks`

**Note:** `forks` is the default pool in Vitest 4.x, so this primarily affects projects that explicitly set `pool: 'threads'`.

## Custom Package Conditions Not Resolved

**Cause:** Vitest only respects `import` and `default` export conditions by default. Custom conditions in `package.json` `exports` field are ignored.

**Fix:** Configure `ssr.resolve.conditions`:

```ts
export default defineConfig({
  ssr: {
    resolve: {
      conditions: ['custom', 'import', 'default'],
    },
  },
})
```

## Segfaults and Native Code Errors

**Symptoms:** "Segmentation fault", "Abort trap", thread panic errors, "internal error: entered unreachable code"

**Cause:** Native Node.js modules (e.g., bcrypt, sharp, sqlite3) are not thread-safe and crash when used with `pool: 'threads'`.

**Fix:** Use `forks` pool:

```ts
export default defineConfig({
  test: {
    pool: 'forks',
  },
})
```

This runs each test file in a separate `child_process` instead of `worker_threads`, providing full process isolation for native modules.

## Key Points

- Most worker-related errors are solved by switching to `pool: 'forks'`
- Path alias issues need `vite-tsconfig-paths` or explicit `alias` config
- Custom export conditions require `ssr.resolve.conditions` configuration
- v4 defaults to `forks` pool, reducing thread-safety issues

<!--
Source references:
- https://vitest.dev/guide/common-errors.html
-->
