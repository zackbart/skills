---
name: vitest
description: Vitest testing framework powered by Vite with Jest-compatible API. Use when writing tests, mocking, configuring coverage, working with test filtering, fixtures, browser mode, benchmarking, or migrating from Jest.
metadata:
  version: "4.0"
  source: https://vitest.dev/guide/
---

Vitest is a next-generation testing framework powered by Vite. It provides a Jest-compatible API with native ESM, TypeScript, and JSX support out of the box. Vitest shares the same config, transformers, resolvers, and plugins with your Vite app.

**Requirements:** Vite >=v6.0.0, Node >=v20.0.0

**Key Features:**
- Vite-native: Uses Vite's transformation pipeline for fast HMR-like test updates
- Jest-compatible: Drop-in replacement for most Jest test suites
- Smart watch mode: Only reruns affected tests based on module graph
- Native ESM, TypeScript, JSX support without configuration
- Multi-threaded workers via `node:child_process` (default) or `node:worker_threads`
- Built-in coverage via V8 (default) or Istanbul
- Snapshot testing, mocking, and spy utilities via Tinyspy
- Browser mode for real browser component testing
- Benchmarking via Tinybench
- In-source testing
- Type testing via expect-type
- Sharding for CI parallelism

> Based on Vitest 4.x (v4.0.17). Key v4 changes: `workspace` replaced with `projects`, pool options are top-level, V8 coverage uses AST-based analysis, `coverage.all` removed, `basic` reporter removed.

## Core

| Topic | Description | Reference |
|-------|-------------|-----------|
| Configuration | Vitest and Vite config integration, defineConfig, mergeConfig | [core-config](references/core-config.md) |
| CLI | Command line interface, commands and all options | [core-cli](references/core-cli.md) |
| Test API | test/it function, modifiers like skip, only, concurrent, for/each | [core-test-api](references/core-test-api.md) |
| Describe API | describe/suite for grouping tests and nested suites | [core-describe](references/core-describe.md) |
| Expect API | Assertions with toBe, toEqual, matchers, asymmetric matchers, soft/poll | [core-expect](references/core-expect.md) |
| Hooks | beforeEach, afterEach, beforeAll, afterAll, aroundEach, aroundAll | [core-hooks](references/core-hooks.md) |

## Features

| Topic | Description | Reference |
|-------|-------------|-----------|
| Mocking | Mock functions, modules, timers, dates, globals with vi utilities | [features-mocking](references/features-mocking.md) |
| Snapshots | Snapshot testing with file, inline, and file snapshots | [features-snapshots](references/features-snapshots.md) |
| Coverage | Code coverage with V8 or Istanbul providers, thresholds, CI | [features-coverage](references/features-coverage.md) |
| Test Context | Test fixtures, context.expect, test.extend, scoped fixtures | [features-context](references/features-context.md) |
| Concurrency | Concurrent tests, parallel execution, sharding, CI setup | [features-concurrency](references/features-concurrency.md) |
| Filtering | Filter tests by name, file patterns, tags, changed files | [features-filtering](references/features-filtering.md) |

## Advanced

| Topic | Description | Reference |
|-------|-------------|-----------|
| Vi Utilities | vi helper: mock, spyOn, fake timers, hoisted, waitFor, mockObject | [advanced-vi](references/advanced-vi.md) |
| Environments | Test environments: node, jsdom, happy-dom, edge-runtime, custom | [advanced-environments](references/advanced-environments.md) |
| Type Testing | Type-level testing with expectTypeOf and assertType | [advanced-type-testing](references/advanced-type-testing.md) |
| Projects | Multi-project workspaces, different configs per project | [advanced-projects](references/advanced-projects.md) |
| Browser Mode | Real browser testing with Playwright, WebdriverIO, or Preview | [advanced-browser](references/advanced-browser.md) |
| Migration | Migrating from Jest and upgrading from Vitest 3.x to 4.x | [advanced-migration](references/advanced-migration.md) |
| Benchmarking | Performance benchmarking with Tinybench integration | [advanced-benchmarking](references/advanced-benchmarking.md) |
| In-Source Testing | Writing tests alongside production code | [advanced-in-source](references/advanced-in-source.md) |
| Reporters | Built-in reporters and custom reporter configuration | [advanced-reporters](references/advanced-reporters.md) |
| Debugging | VS Code, IntelliJ, Chrome DevTools, inspect flags | [advanced-debugging](references/advanced-debugging.md) |
| Vitest UI | Interactive browser UI with module graph and coverage | [advanced-ui](references/advanced-ui.md) |
| Common Errors | Frequent errors and their solutions | [advanced-common-errors](references/advanced-common-errors.md) |
