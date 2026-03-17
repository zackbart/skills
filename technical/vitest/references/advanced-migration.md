---
name: migration
description: Migrating from Jest and upgrading from Vitest 3.x to 4.x
---

# Migration

## Migrating from Jest

### Globals

```ts
// Option 1: Enable globals
defineConfig({ test: { globals: true } })

// Option 2: Import explicitly (recommended)
import { describe, expect, test, vi } from 'vitest'
```

### Key Differences

| Jest | Vitest |
|------|--------|
| `jest.fn()` | `vi.fn()` |
| `jest.mock()` | `vi.mock()` |
| `jest.spyOn()` | `vi.spyOn()` |
| `jest.requireActual()` | `await vi.importActual()` (async) |
| `jest.requireMock()` | `await vi.importMock()` (async) |
| `jest.setTimeout()` | `vi.setConfig({ testTimeout: 5000 })` |
| `jest.Mock` type | `Mock` from `vitest` |
| `JEST_WORKER_ID` | `VITEST_POOL_ID` (also `VITEST_WORKER_ID`) |

### mockReset Behavior

Vitest's `mockReset` resets to **original implementation**. Jest replaces with empty function.

### mock.mock is Persistent

Vitest maintains persistent `.mock` reference; Jest recreates on `.mockClear()`.

### Module Mocks

Must explicitly define exports:

```ts
// Vitest - explicit exports required
vi.mock('./path', () => ({
  default: 'hello',
  namedExport: vi.fn(),
}))
```

### Auto-Mocking

`__mocks__` directories are not loaded unless `vi.mock()` is called (Jest loads automatically).

### External Libraries

Use `server.deps.inline` for third-party mocking:

```ts
defineConfig({
  test: {
    server: {
      deps: {
        inline: ['third-party-lib'],
      },
    },
  },
})
```

### Test Name Separator

Vitest uses `>` instead of space between describe/test names.

### Done Callback

Not supported. Use async/await or `new Promise`:

```ts
// Jest
test('callback', (done) => { fetchData((data) => { expect(data).toBe('ok'); done() }) })

// Vitest
test('async', async () => {
  const data = await new Promise(resolve => fetchData(resolve))
  expect(data).toBe('ok')
})
```

### Hooks

Vitest hooks may return a teardown function. Default execution is parallel (Jest is sequential). Use `sequence.hooks: 'list'` for Jest behavior.

### Legacy Timers

Legacy fake timers not supported. Only modern timers via `vi.useFakeTimers()`.

### Vue Snapshots

Use `jest-serializer-vue` in `snapshotSerializers`.

### `replaceProperty`

Use `vi.stubEnv` or `vi.spyOn` instead of Jest's `replaceProperty`.

## Migrating from Vitest 3.x to 4.x

### workspace -> projects

```ts
// v3: vitest.workspace.ts
export default ['packages/*']

// v4: vitest.config.ts
export default defineConfig({
  test: {
    projects: ['packages/*', { test: { name: 'unit' } }]
  }
})
```

### Pool Changes

```ts
// v3
pool: 'threads',
poolOptions: { threads: { maxThreads: 4 } }

// v4 - options are top-level
pool: 'forks', // forks is now default
maxWorkers: 4,
isolate: true,

// v3: singleThread/singleFork
// v4:
maxWorkers: 1,
isolate: false,
```

Tinypool dependency removed in v4.

### Coverage Changes

- V8 uses AST-based analysis (replaces `v8-to-istanbul`)
- `coverage.all` removed - only covered files included by default
- `coverage.extensions` removed
- `coverage.ignoreEmptyLines` removed
- `coverage.experimentalAstAwareRemapping` removed (now default)
- `coverage.ignoreClassMethods` now supported by V8

### Exclude Simplification

v4 default `exclude` is only `node_modules` and `.git`. Use `test.dir` to limit scan scope for performance.

### Reporter Changes

- `basic` reporter removed. Use `['default', { summary: false }]`
- `verbose` prints flat list; use `--reporter=tree` for old behavior

### Mock Changes

- `vi.fn().getMockName()` returns `vi.fn()` instead of `spy`
- `vi.restoreAllMocks` only restores manually-created spies (not automocks)
- `mock.settledResults` populated immediately with 'incomplete' status
- Automocked getters return `undefined` by default
- `vi.fn().mock.invocationCallOrder` starts at `1` (like Jest)
- `spyOn` and `fn` support constructors (must use `function`/`class`, not arrows)

### Browser Provider

```ts
// v3
browser: { provider: 'playwright' }

// v4
import { playwright } from '@vitest/browser-playwright'
browser: { provider: playwright() }
```

### Import Changes

```ts
// v3
import { ... } from '@vitest/browser/context'
import { ... } from '@vitest/browser/utils'

// v4
import { ... } from 'vitest/browser'
```

### Snapshots

Custom elements now include shadow root by default. Disable with:

```ts
snapshotFormat: { printShadowRoot: false }
```

### Test Options

Test options moved to second argument:

```ts
// v3 (some options on third arg)
test('name', () => {}, { retry: 2 })

// v4
test('name', { retry: 2 }, () => {})
```

### Removed APIs

`poolMatchGlobs`, `environmentMatchGlobs`, `deps.external`, `deps.inline`, `deps.fallbackCJS`, `browser.testerScripts`, `minWorkers`

<!--
Source references:
- https://vitest.dev/guide/migration.html
-->
