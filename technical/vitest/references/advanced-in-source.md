---
name: in-source-testing
description: Writing tests alongside production code
---

# In-Source Testing

Write tests directly in source files, similar to Rust's module tests.

## Basic Usage

```ts
// src/math.ts
export function add(a: number, b: number) {
  return a + b
}

// Tests are tree-shaken in production builds
if (import.meta.vitest) {
  const { it, expect } = import.meta.vitest

  it('add', () => {
    expect(add(1, 2)).toBe(3)
  })
}
```

## Configuration

Enable in-source testing:

```ts
// vitest.config.ts
defineConfig({
  test: {
    includeSource: ['src/**/*.{ts,js}'],
  },
  // Define for production build to tree-shake tests
  define: {
    'import.meta.vitest': 'undefined',
  },
})
```

## TypeScript Setup

Add type reference to make `import.meta.vitest` typed:

```ts
// src/env.d.ts (or any .d.ts file)
/// <reference types="vitest/importMeta" />
```

## Production Build

Tests are automatically removed from production builds when:

```ts
// vite.config.ts (production)
defineConfig({
  define: {
    'import.meta.vitest': 'undefined',
  },
})
```

The `if (import.meta.vitest)` block becomes dead code and is tree-shaken.

## When to Use

- Small utility functions that benefit from co-located tests
- Quick validation during development
- Functions where test proximity improves maintainability

## When NOT to Use

- Complex test suites with many fixtures/mocks
- Integration or E2E tests
- Tests that need shared setup/teardown

## Key Points

- Use `import.meta.vitest` guard for tree-shaking
- Configure `includeSource` to scan source files
- Set `define: { 'import.meta.vitest': 'undefined' }` for production builds
- Tests live alongside the code they test
- Inspired by Rust's `#[cfg(test)]` pattern

<!--
Source references:
- https://vitest.dev/guide/in-source.html
- https://vitest.dev/guide/features.html#in-source-testing
-->
