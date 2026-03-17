---
name: vitest-cli
description: Command line interface commands and options
---

# Command Line Interface

## Commands

### `vitest`

Start Vitest in watch mode (dev) or run mode (CI):

```bash
vitest                          # Watch mode in dev, run mode in CI
vitest foobar                   # Run tests containing "foobar" in path
vitest basic/foo.test.ts:10     # Run specific test by file and line number (v3+)
```

### `vitest run`

Run tests once without watch mode:

```bash
vitest run
vitest run --coverage
```

### `vitest watch`

Explicitly start watch mode:

```bash
vitest watch
```

### `vitest related`

Run tests that import specific files (useful with lint-staged):

```bash
vitest related src/index.ts src/utils.ts --run
```

### `vitest bench`

Run only benchmark tests:

```bash
vitest bench
```

### `vitest list`

List all matching tests without running them:

```bash
vitest list                    # List test names
vitest list --json             # Output as JSON
vitest list --filesOnly        # List only test files
```

### `vitest init`

Initialize project setup:

```bash
vitest init browser            # Set up browser testing
```

## Common Options

```bash
# Configuration
--config <path>           # Path to config file
--project <name>          # Run specific project (repeatable, supports wildcards and !exclusion)
--root <path>             # Root path
--dir <path>              # Base directory for test file scanning

# Filtering
--testNamePattern, -t     # Run tests matching regexp pattern
--changed                 # Run tests for changed files
--changed HEAD~1          # Tests for last commit changes
--exclude <glob>          # Exclude pattern

# Reporters
--reporter <name>         # default, verbose, dot, json, tap, tap-flat, junit, tree, hanging-process, github-actions, blob
--outputFile <path>       # Write results to file (dot notation for multiple reporters)
--silent                  # Suppress console output ('passed-only' for failing tests only)
--hideSkippedTests        # Hide logs for skipped tests

# Coverage
--coverage                # Enable coverage
--coverage.provider v8    # Use v8 provider
--coverage.reporter text,html
--coverage.thresholds.lines 80

# Execution
--shard <index>/<count>   # Split tests across machines (cannot use with --watch)
--bail <n>                # Stop after n failures
--retry <n>               # Retry failed tests n times
--sequence.shuffle        # Randomize test order

# Watch mode
--run                     # Single run (disable watch mode)
--standalone              # Start without running tests
--no-watch                # Disable watch mode

# Environment
--environment <env>       # jsdom, happy-dom, node, edge-runtime
--globals                 # Enable global APIs
--isolate / --no-isolate  # File isolation (default: true)
--dom                     # Mock browser API with happy-dom

# Pool & Workers
--pool <pool>             # Pool type: forks (default), threads
--maxWorkers <n>          # Max workers or percentage
--fileParallelism / --no-file-parallelism

# Debugging
--inspect                 # Enable Node inspector (default: 127.0.0.1:9229)
--inspectBrk              # Break on start

# Output
--no-color                # Disable colors
--clearScreen             # Clear screen (default: true)
--logHeapUsage            # Show heap size per test
--printConsoleTrace       # Print console trace

# Timeouts
--testTimeout <ms>        # Test timeout (default: 5000)
--hookTimeout <ms>        # Hook timeout (default: 10000)
--teardownTimeout <ms>    # Teardown timeout (default: 10000)

# Type Checking
--typecheck.enabled       # Enable type checking
--typecheck.only          # Only type check
--typecheck.checker <name>  # 'tsc' or 'vue-tsc'

# Browser Mode
--browser.enabled         # Enable browser mode
--browser.headless        # Headless mode (default: process.env.CI)

# Misc
--passWithNoTests         # Pass when no tests found
--allowOnly               # Allow .only tests (default: !process.env.CI)
--clearCache              # Delete all caches
--ui                      # Enable Vitest UI
--open                    # Open UI automatically
```

## Package.json Scripts

```json
{
  "scripts": {
    "test": "vitest",
    "test:run": "vitest run",
    "test:ui": "vitest --ui",
    "coverage": "vitest run --coverage"
  }
}
```

## Sharding for CI

Split tests across multiple machines:

```bash
# Machine 1
vitest run --shard=1/3 --reporter=blob --coverage

# Machine 2
vitest run --shard=2/3 --reporter=blob --coverage

# Machine 3
vitest run --shard=3/3 --reporter=blob --coverage

# Merge reports
vitest --merge-reports --reporter=junit --coverage
```

## Watch Mode Keyboard Shortcuts

In watch mode, press:
- `a` - Run all tests
- `f` - Run only failed tests
- `u` - Update snapshots
- `p` - Filter by filename pattern
- `t` - Filter by test name pattern
- `q` - Quit

## Key Points

- Watch mode is default in dev, run mode in CI (when `process.env.CI` is set)
- Use `--run` flag to ensure single run (important for lint-staged)
- Both camelCase (`--testTimeout`) and kebab-case (`--test-timeout`) work
- Boolean options can be negated with `--no-` prefix or `=false`
- Array options require multiple flag passes
- With Bun, use `bun run test` not `bun test`
- Use `vitest list` to preview which tests match without running them

<!--
Source references:
- https://vitest.dev/guide/cli.html
-->
