---
name: code-coverage
description: Code coverage with V8 or Istanbul providers
---

# Code Coverage

## Setup

Install a coverage provider:

```bash
# V8 (default, faster)
npm i -D @vitest/coverage-v8

# Istanbul (more compatible)
npm i -D @vitest/coverage-istanbul
```

Run tests with coverage:

```bash
vitest run --coverage
```

## Configuration

```ts
// vitest.config.ts
defineConfig({
  test: {
    coverage: {
      // Provider: 'v8' (default, faster) or 'istanbul' (more compatible)
      provider: 'v8',

      // Enable coverage
      enabled: true,

      // Reporters
      reporter: ['text', 'json', 'html'],

      // Files to include
      include: ['src/**/*.{ts,tsx}'],

      // Files to exclude
      exclude: [
        'node_modules/',
        'tests/',
        '**/*.d.ts',
        '**/*.test.ts',
      ],

      // Report directory (default: ./coverage)
      reportsDirectory: './coverage',

      // Thresholds
      thresholds: {
        lines: 80,
        functions: 80,
        branches: 80,
        statements: 80,
      },
    },
  },
})
```

## Providers

### V8 (Default)

- Native V8 engine coverage via `node:inspector` and Chrome DevTools Protocol
- No pre-instrumentation step - faster, lower memory
- v4: Uses AST-based analysis (replaces v8-to-istanbul)
- Cannot be used in non-V8 environments (Firefox, Bun, Cloudflare Workers)

### Istanbul

- Source file instrumentation
- Works on any JS runtime
- Battle-tested 13+ years
- Requires pre-instrumentation - slower

## Reporters

```ts
coverage: {
  reporter: [
    'text',           // Terminal output
    'text-summary',   // Summary only
    'json',           // JSON file
    'html',           // HTML report
    'lcov',           // For CI tools (Codecov, etc.)
    'cobertura',      // XML format
    // Custom reporters
    ['@vitest/custom-coverage-reporter', { someOption: true }],
    '/absolute/path/to/custom-reporter.cjs',
  ],
  reportsDirectory: './coverage',
}
```

## Thresholds

Fail tests if coverage is below threshold:

```ts
coverage: {
  thresholds: {
    // Global thresholds
    lines: 80,
    functions: 75,
    branches: 70,
    statements: 80,

    // Require 100% coverage
    100: true,

    // Per-file thresholds
    perFile: true,

    // Auto-update thresholds (for gradual improvement)
    autoUpdate: true,
  },
}
```

## Ignoring Code

Both providers support ignore directives. Use `@preserve` to keep comments through esbuild:

### V8

```ts
/* v8 ignore next -- @preserve */
function ignored() {
  return 'not covered'
}

/* v8 ignore start -- @preserve */
// All code here ignored
/* v8 ignore stop -- @preserve */

/* v8 ignore if -- @preserve */
if (condition) {
  // ignored
}
```

### Istanbul

```ts
/* istanbul ignore next -- @preserve */
function ignored() {}

/* istanbul ignore if -- @preserve */
if (condition) {
  // ignored
}
```

## v4 Changes

- `coverage.all` removed - only covered files included by default
- `coverage.extensions` removed
- `coverage.ignoreEmptyLines` removed (V8)
- `coverage.experimentalAstAwareRemapping` removed (now default behavior)
- `coverage.ignoreClassMethods` now supported by V8 provider

## Vitest UI Coverage

Enable HTML coverage in Vitest UI:

```ts
coverage: {
  enabled: true,
  reporter: ['text', 'html'],
}
```

Run with `vitest --ui` to view coverage visually.

## CI Integration

```yaml
# GitHub Actions
- name: Run tests with coverage
  run: npm run test:coverage

- name: Upload coverage to Codecov
  uses: codecov/codecov-action@v3
  with:
    files: ./coverage/lcov.info
```

## Coverage with Sharding

Merge coverage from sharded runs:

```bash
vitest run --shard=1/3 --coverage --reporter=blob
vitest run --shard=2/3 --coverage --reporter=blob
vitest run --shard=3/3 --coverage --reporter=blob
vitest --merge-reports --coverage --reporter=json
```

## Custom Coverage Provider

```ts
defineConfig({
  test: {
    coverage: {
      provider: 'custom',
      customProviderModule: './my-coverage-provider.ts',
    },
  },
})
```

Module must implement `CoverageProviderModule` interface.

## Key Points

- V8 is faster and default; Istanbul is more compatible across runtimes
- Use `--coverage` flag or `coverage.enabled: true`
- Set thresholds to enforce minimum coverage
- Use `@preserve` comment to keep ignore hints through esbuild
- v4: Only covered files included by default (no `coverage.all`)

<!--
Source references:
- https://vitest.dev/guide/coverage.html
-->
