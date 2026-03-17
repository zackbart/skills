---
name: concurrency-parallelism
description: Concurrent tests, parallel execution, and sharding
---

# Concurrency & Parallelism

## File Parallelism

By default, Vitest runs test files in parallel across workers:

```ts
defineConfig({
  test: {
    // Run files in parallel (default: true)
    fileParallelism: true,

    // v4: pool options are top-level
    maxWorkers: 4,

    // Pool type: 'forks' (default) or 'threads'
    pool: 'forks',
  },
})
```

## Concurrent Tests

Run tests within a file in parallel:

```ts
// Individual concurrent tests
test.concurrent('test 1', async ({ expect }) => {
  expect(await fetch1()).toBe('result')
})

test.concurrent('test 2', async ({ expect }) => {
  expect(await fetch2()).toBe('result')
})

// All tests in suite concurrent
describe.concurrent('parallel suite', () => {
  test('test 1', async ({ expect }) => {})
  test('test 2', async ({ expect }) => {})
})
```

**Important:** Use `{ expect }` from context for concurrent tests.

## Sequential in Concurrent Context

Force sequential execution:

```ts
describe.concurrent('mostly parallel', () => {
  test('parallel 1', async () => {})
  test('parallel 2', async () => {})

  test.sequential('must run alone 1', async () => {})
  test.sequential('must run alone 2', async () => {})
})

// Or entire suite
describe.sequential('sequential suite', () => {
  test('first', () => {})
  test('second', () => {})
})
```

## Max Concurrency

Limit concurrent tests:

```ts
defineConfig({
  test: {
    maxConcurrency: 5, // Max concurrent tests per file (default: 5)
  },
})
```

## Isolation

Each file runs in isolated environment by default:

```ts
defineConfig({
  test: {
    // Disable isolation for faster runs (less safe)
    isolate: false,
  },
})
```

## Sharding

Split tests across machines:

```bash
# Machine 1
vitest run --shard=1/3

# Machine 2
vitest run --shard=2/3

# Machine 3
vitest run --shard=3/3
```

### CI Example (GitHub Actions)

```yaml
jobs:
  test:
    strategy:
      matrix:
        shard: [1, 2, 3]
    steps:
      - run: vitest run --shard=${{ matrix.shard }}/3 --reporter=blob --coverage

  merge:
    needs: test
    steps:
      - run: vitest --merge-reports --reporter=junit --coverage
```

### Merge Reports

```bash
# Each shard outputs blob
vitest run --shard=1/3 --reporter=blob --coverage
vitest run --shard=2/3 --reporter=blob --coverage

# Merge all blobs (default dir: .vitest-reports)
vitest --merge-reports --reporter=json --coverage
```

## Test Sequence

Control test order:

```ts
defineConfig({
  test: {
    sequence: {
      // Shuffle files and/or tests
      shuffle: { files: false, tests: false },

      // Seed for reproducible shuffle
      seed: 12345,

      // Hook execution order
      hooks: 'parallel', // 'stack', 'list', 'parallel'

      // All tests concurrent by default
      concurrent: false,
    },
  },
})
```

## Shuffle Tests

Randomize to catch hidden dependencies:

```bash
# Via CLI
vitest --sequence.shuffle
```

```ts
// Per suite
describe.shuffle('random order', () => {
  test('test 1', () => {})
  test('test 2', () => {})
  test('test 3', () => {})
})
```

## Pool Types

### Forks (Default in v4)

Uses `node:child_process`:

```ts
defineConfig({
  test: {
    pool: 'forks',
    maxWorkers: 4,
    isolate: true,
  },
})
```

### Threads

Uses `node:worker_threads`:

```ts
defineConfig({
  test: {
    pool: 'threads',
    maxWorkers: 8,
    isolate: true,
  },
})
```

**v4 Note:** `poolOptions` removed. Use top-level `maxWorkers`, `isolate`. `singleThread`/`singleFork` replaced with `maxWorkers: 1, isolate: false`.

## Bail on Failure

Stop after first failure:

```bash
vitest --bail 1    # Stop after 1 failure
vitest --bail      # Stop on first failure (same as --bail 1)
```

## Key Points

- Files run in parallel by default across worker processes
- Use `.concurrent` for parallel tests within a file
- Always use context's `expect` in concurrent tests
- Sharding splits tests across CI machines
- Use `--merge-reports` to combine sharded results
- Shuffle tests to find hidden dependencies
- v4: `forks` is default pool; `poolOptions` removed

<!--
Source references:
- https://vitest.dev/guide/features.html#running-tests-concurrently
- https://vitest.dev/guide/improving-performance.html
-->
