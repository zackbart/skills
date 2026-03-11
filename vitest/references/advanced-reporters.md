---
name: reporters
description: Built-in reporters and custom reporter configuration
---

# Reporters

## Built-in Reporters

| Reporter | Description |
|----------|-------------|
| `default` | Default reporter with summary |
| `verbose` | Flat list of all tests (v4: use `tree` for old verbose behavior) |
| `dot` | Minimal dot output |
| `json` | JSON output |
| `tap` | TAP format |
| `tap-flat` | Flat TAP format |
| `junit` | JUnit XML format |
| `tree` | Tree view of tests (v4: replaces old `verbose` tree behavior) |
| `hanging-process` | Shows processes that prevent exit |
| `github-actions` | GitHub Actions annotations |
| `blob` | Binary format for merging sharded reports |

**v4 Note:** `basic` reporter removed. Use `['default', { summary: false }]` instead.

## Configuration

```ts
defineConfig({
  test: {
    reporters: ['default'],

    // Multiple reporters
    reporters: ['default', 'json', 'html'],

    // Reporter with options
    reporters: [
      ['default', { summary: false }],
      ['json', {}],
    ],

    // Output to file
    outputFile: {
      json: './test-results.json',
      junit: './junit.xml',
    },
  },
})
```

## CLI Usage

```bash
# Single reporter
vitest --reporter=verbose

# Multiple reporters
vitest --reporter=default --reporter=json

# Output to file
vitest --reporter=json --outputFile=results.json

# Multiple outputs
vitest --reporter=default --reporter=junit --outputFile.junit=junit.xml
```

## Blob Reporter (Sharding)

Used with `--shard` for merging reports across machines:

```bash
# Each shard
vitest run --shard=1/3 --reporter=blob

# Merge
vitest --merge-reports --reporter=junit
```

Default blob directory: `.vitest-reports`

## Silent Mode

```bash
# Suppress all console output
vitest --silent

# Only show failing tests
vitest --silent=passed-only
```

## Other Display Options

```bash
# Hide skipped tests from output
vitest --hideSkippedTests

# Show console trace for log statements
vitest --printConsoleTrace

# Show heap usage per test
vitest --logHeapUsage
```

## Custom Reporters

Create a module implementing the reporter interface:

```ts
// my-reporter.ts
import type { Reporter } from 'vitest/reporters'

export default class MyReporter implements Reporter {
  onInit(ctx) {}
  onCollected(files) {}
  onTaskUpdate(packs) {}
  onFinished(files, errors) {
    console.log('Tests finished!')
  }
}
```

Use in config:

```ts
defineConfig({
  test: {
    reporters: ['./my-reporter.ts'],
  },
})
```

## Key Points

- Default reporter includes a summary
- Use `blob` reporter for sharded CI runs
- v4: `basic` removed, `verbose` is flat list, use `tree` for tree view
- Multiple reporters can run simultaneously
- Use `outputFile` to write results to files
- `--silent=passed-only` is useful for focused debugging

<!--
Source references:
- https://vitest.dev/guide/reporters.html
- https://vitest.dev/config/#reporters
-->
