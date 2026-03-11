---
name: vitest-configuration
description: Configure Vitest with vite.config.ts or vitest.config.ts
---

# Configuration

Vitest reads configuration from `vitest.config.ts` or `vite.config.ts`. It shares the same config format as Vite.

## Basic Setup

```ts
// vitest.config.ts
import { defineConfig } from 'vitest/config'

export default defineConfig({
  test: {
    // test options
  },
})
```

## Using with Existing Vite Config

Add Vitest types reference and use the `test` property:

```ts
// vite.config.ts
/// <reference types="vitest/config" />
import { defineConfig } from 'vite'

export default defineConfig({
  test: {
    globals: true,
    environment: 'jsdom',
  },
})
```

## Merging Configs

If you have separate config files, use `mergeConfig`:

```ts
// vitest.config.ts
import { defineConfig, mergeConfig } from 'vitest/config'
import viteConfig from './vite.config'

export default mergeConfig(viteConfig, defineConfig({
  test: {
    environment: 'jsdom',
  },
}))
```

## Extending Defaults

Use `configDefaults` to extend built-in defaults:

```ts
import { configDefaults, defineConfig } from 'vitest/config'

export default defineConfig({
  test: {
    exclude: [...configDefaults.exclude, 'packages/template/*'],
  },
})
```

## Common Options

```ts
defineConfig({
  test: {
    // Enable global APIs (describe, it, expect) without imports
    globals: true,

    // Test environment: 'node', 'jsdom', 'happy-dom'
    environment: 'node',

    // Setup files to run before each test file
    setupFiles: ['./tests/setup.ts'],

    // Include patterns for test files
    include: ['**/*.{test,spec}.{js,ts,jsx,tsx}'],

    // Exclude patterns (v4 default: only node_modules and .git)
    exclude: ['**/node_modules/**', '**/.git/**'],

    // Base directory for test file scanning (use for performance)
    dir: './src',

    // Test timeout in ms (default: 5000)
    testTimeout: 5000,

    // Hook timeout in ms (default: 10000)
    hookTimeout: 10000,

    // Teardown timeout in ms (default: 10000)
    teardownTimeout: 10000,

    // Coverage configuration
    coverage: {
      provider: 'v8', // or 'istanbul'
      reporter: ['text', 'html'],
      include: ['src/**/*.ts'],
    },

    // Run tests in isolation (each file in separate context)
    isolate: true,

    // Pool for running tests (default: 'forks')
    pool: 'forks',

    // v4: pool options are top-level (poolOptions removed)
    maxWorkers: 4,

    // Run files in parallel (default: true)
    fileParallelism: true,

    // Automatically clear mocks between tests
    clearMocks: true,

    // Reset mocks between tests (clear + remove implementation)
    mockReset: true,

    // Restore original implementations between tests
    restoreMocks: true,

    // Auto-restore stubbed env vars and globals
    unstubEnvs: true,
    unstubGlobals: true,

    // Retry failed tests
    retry: 0,

    // Stop after n failures (0 = no limit)
    bail: 0,

    // Max concurrent tests per file (default: 5)
    maxConcurrency: 5,

    // Slow test threshold in ms (default: 300)
    slowTestThreshold: 300,
  },
})
```

## Conditional Configuration

Use `mode` or `process.env.VITEST` for test-specific config:

```ts
export default defineConfig(({ mode }) => ({
  plugins: mode === 'test' ? [] : [myPlugin()],
  test: {
    // test options
  },
}))
```

## Projects (Multi-config)

Run different configurations in the same Vitest process:

```ts
defineConfig({
  test: {
    projects: [
      'packages/*',
      {
        test: {
          name: 'unit',
          include: ['tests/unit/**/*.test.ts'],
          environment: 'node',
        },
      },
      {
        test: {
          name: 'integration',
          include: ['tests/integration/**/*.test.ts'],
          environment: 'jsdom',
        },
      },
    ],
  },
})
```

## Snapshot Configuration

```ts
defineConfig({
  test: {
    snapshotFormat: {
      printBasicPrototype: false, // default in Vitest (differs from Jest)
      printShadowRoot: false,    // v4: custom elements include shadow root by default
    },
    snapshotSerializers: ['./my-serializer.ts'],
    resolveSnapshotPath: (testPath, snapExtension) => {
      return testPath.replace('__tests__', '__snapshots__') + snapExtension
    },
  },
})
```

## Sequence Configuration

```ts
defineConfig({
  test: {
    sequence: {
      shuffle: { files: false, tests: false },
      seed: 12345,
      hooks: 'parallel', // 'stack', 'list', 'parallel'
      setupFiles: 'parallel', // 'list', 'parallel'
      concurrent: false,
    },
  },
})
```

## Fake Timers Configuration

```ts
defineConfig({
  test: {
    fakeTimers: {
      now: Date.now(),
      shouldAdvanceTime: false,
      advanceTimeDelta: 20,
      toFake: ['setTimeout', 'setInterval', 'Date'],
    },
  },
})
```

## Expectations Configuration

```ts
defineConfig({
  test: {
    expect: {
      requireAssertions: false,
      poll: {
        interval: 50,   // ms between retries
        timeout: 1000,   // max wait time
      },
    },
  },
})
```

## Diff Configuration

```ts
defineConfig({
  test: {
    diff: {
      aAnnotation: 'Expected',
      bAnnotation: 'Received',
      contextLines: 5,
      expand: true,
      maxDepth: 20,
      truncateThreshold: 0, // 0 = no truncation
    },
  },
})
```

## Key Points

- Vitest uses Vite's transformation pipeline - same `resolve.alias`, plugins work
- `vitest.config.ts` takes priority over `vite.config.ts`
- Use `--config` flag to specify a custom config path
- `process.env.VITEST` is set when running tests; `import.meta.env.VITEST` also available
- Supported config extensions: `.js`, `.mjs`, `.cjs`, `.ts`, `.cts`, `.mts` (NOT `.json`)
- v4: `workspace` files replaced with `projects` in config
- v4: `poolOptions` removed; use top-level `maxWorkers`, `isolate`, etc.
- v4: Default `exclude` simplified to only `node_modules` and `.git`

<!--
Source references:
- https://vitest.dev/guide/#configuring-vitest
- https://vitest.dev/config/
-->
