---
name: benchmarking
description: Performance benchmarking with Tinybench integration
---

# Benchmarking

Vitest includes benchmarking support via Tinybench.

## Basic Benchmark

```ts
import { bench, describe } from 'vitest'

bench('sort array', () => {
  [1, 5, 4, 2, 3].sort((a, b) => a - b)
})

bench('reverse sort', () => {
  [1, 5, 4, 2, 3].sort((a, b) => b - a)
})
```

## Grouped Benchmarks

```ts
describe('string concatenation', () => {
  bench('template literals', () => {
    const name = 'world'
    const result = `hello ${name}`
  })

  bench('plus operator', () => {
    const name = 'world'
    const result = 'hello ' + name
  })

  bench('concat method', () => {
    const name = 'world'
    const result = 'hello '.concat(name)
  })
})
```

## Benchmark Options

```ts
bench('with options', () => {
  // code to benchmark
}, {
  time: 1000,       // Time in ms to run (default: 100)
  iterations: 100,  // Number of iterations (overrides time)
  warmupTime: 100,  // Warmup time in ms
  warmupIterations: 5,
  throws: false,    // Whether benchmark should throw errors
  setup: (task, mode) => {
    // Setup before each benchmark iteration
    // mode is 'warmup' or 'run'
  },
  teardown: (task, mode) => {
    // Cleanup after each benchmark
  },
})
```

## Running Benchmarks

```bash
# Run all benchmarks
vitest bench

# Filter benchmarks
vitest bench sort

# With specific reporter
vitest bench --reporter=verbose
```

## Benchmark File Pattern

By default, benchmark files match: `**/*.bench.{js,ts,jsx,tsx}`

Configure:

```ts
defineConfig({
  test: {
    benchmark: {
      include: ['**/*.bench.{ts,js}'],
      exclude: ['node_modules'],
    },
  },
})
```

## Key Points

- Use `vitest bench` to run benchmarks (not `vitest run`)
- Benchmarks use `.bench.ts` file extension by default
- Powered by Tinybench library
- Results show ops/sec, min/max time, and samples
- Use `describe` to group related benchmarks for comparison

<!--
Source references:
- https://vitest.dev/guide/features.html#benchmarking
-->
