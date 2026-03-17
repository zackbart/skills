---
name: vitest-ui
description: Visual test runner UI with module graph and coverage integration
---

# Vitest UI

Interactive browser-based UI for running, viewing, and debugging tests.

## Installation

```bash
npm i -D @vitest/ui
```

## Usage

```bash
vitest --ui
```

Opens at `http://localhost:51204/__vitest__/` in watch mode.

The UI requires a running Vite server and operates in watch mode by default.

## Configuration

```ts
defineConfig({
  test: {
    // Enable UI
    ui: true,

    // Auto-open browser (default: !process.env.CI)
    open: true,

    // API server for UI
    api: {
      port: 51204,  // default
      host: 'localhost',
    },
  },
})
```

## Features

### Test Explorer

- View all test files and their status (pass/fail/skip)
- Filter tests by name or file
- Re-run individual tests or suites
- View test output and error details

### Module Graph

The Module Graph tab visualizes dependencies for selected test files:

- **Large graphs** show only first two levels initially; expand via "Show Full Graph"
- **Navigation:** Left-click modules to view details; right-click or Shift+click to expand related nodes
- **Filtering:** Toggle to hide/show `node_modules` dependencies

### Module Info Panel

Displays diagnostics for selected modules:

- Full module ID
- **Self Time** - Import duration excluding static dependencies
- **Total Time** - Import duration including static imports
- Transform time
- Source code, transformed code, and source maps
- Performance indicators (red >500ms, orange >100ms)

### Import Breakdown

Lists the 10 slowest-loading modules sorted by Total Time, showing their contribution percentage to overall load time.

### Coverage Integration

View coverage reports directly in the UI. Requires:

```ts
defineConfig({
  test: {
    coverage: {
      enabled: true,
      reporter: ['text', 'html'], // html reporter required for UI
    },
  },
})
```

## HTML Reporter (Static Alternative)

For static reports without a running server (useful for CI artifacts):

```ts
defineConfig({
  test: {
    reporters: ['default', 'html'],
  },
})
```

Preview generated reports:

```bash
npx vite preview --outDir ./html
```

Customize output location via `outputFile` config (default: `./html/index.html`).

## Key Points

- Install `@vitest/ui` separately
- Runs in watch mode with a live Vite server
- Module graph helps identify slow imports and dependency issues
- Use `html` reporter for static CI artifacts instead of the live UI
- Coverage requires `html` in `coverage.reporter` to display in UI

<!--
Source references:
- https://vitest.dev/guide/ui.html
-->
