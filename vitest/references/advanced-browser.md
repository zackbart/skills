---
name: browser-mode
description: Real browser testing with Playwright, WebdriverIO, or Preview
---

# Browser Mode

Run tests in real browsers for component testing with real DOM, events, and browser APIs.

## Providers

- **Playwright** (recommended for CI): `npm i -D @vitest/browser-playwright`
- **WebdriverIO**: `npm i -D @vitest/browser-webdriverio`
- **Preview** (dev only): `npm i -D @vitest/browser-preview`

### Browser Support

| Provider | Browsers |
|----------|----------|
| Playwright | Firefox, WebKit, Chromium |
| WebdriverIO | Firefox, Chrome, Edge, Safari |

**Minimum versions:** Chrome >=87, Firefox >=78, Safari >=15.4, Edge >=88

## Configuration

```ts
import { playwright } from '@vitest/browser-playwright'

export default defineConfig({
  test: {
    browser: {
      provider: playwright(),
      enabled: true,
      instances: [
        { browser: 'chromium' },
      ],
    },
  },
})
```

### v4 Provider Change

In v4, providers accept objects (not strings):

```ts
// v4
import { playwright } from '@vitest/browser-playwright'
provider: playwright({ launchOptions: { slowMo: 100 } })

// NOT: provider: 'playwright' (v3 syntax)
```

## Framework Support

Install framework-specific rendering packages:

```bash
# Official
npm i -D vitest-browser-vue
npm i -D vitest-browser-react
npm i -D vitest-browser-svelte
npm i -D vitest-browser-angular

# Community
npm i -D vitest-browser-lit
npm i -D vitest-browser-preact
```

## Testing APIs

```ts
import { page, userEvent } from 'vitest/browser'

test('user interaction', async () => {
  // Query elements (Playwright-style locators)
  await expect.element(page.getByText('Hello')).toBeInTheDocument()

  // User events
  await userEvent.fill(page.getByLabelText(/username/i), 'Alice')
  await userEvent.click(page.getByRole('button', { name: 'Submit' }))

  // Direct element methods
  await page.getByLabelText(/username/i).fill('Alice')
})
```

### v4 Import Change

```ts
// v4: import from 'vitest/browser'
import { page, userEvent } from 'vitest/browser'

// NOT: from '@vitest/browser/context' (v3 syntax)
```

## Multi-Project Setup (Node + Browser)

```ts
import { playwright } from '@vitest/browser-playwright'

export default defineConfig({
  test: {
    projects: [
      {
        test: {
          include: ['tests/unit/**'],
          name: 'unit',
          environment: 'node',
        },
      },
      {
        test: {
          include: ['tests/browser/**'],
          name: 'browser',
          browser: {
            enabled: true,
            provider: playwright(),
            instances: [{ browser: 'chromium' }],
          },
        },
      },
    ],
  },
})
```

## Browser Config Options

```ts
browser: {
  // API server
  api: {
    port: 63315,  // default
    host: 'localhost',
  },

  // Viewport
  viewport: { width: 1280, height: 720 },

  // Run headless (default: process.env.CI)
  headless: true,

  // Isolate tests (default: true)
  isolate: true,

  // File parallelism (default: true)
  fileParallelism: true,

  // UI (default: !process.env.CI)
  ui: true,

  // Connection timeout (default: 60000)
  connectTimeout: 60000,

  // Track unhandled errors (default: true)
  trackUnhandledErrors: true,

  // Trace mode: 'on', 'off', 'on-first-retry', 'on-all-retries', 'retain-on-failure'
  trace: 'on-first-retry',

  // Screenshot settings
  screenshotDirectory: '__screenshots__',
  screenshotFailures: true,
}
```

## Key Limitations

- Thread-blocking dialogs (`alert`, `confirm`) are auto-mocked
- Cannot directly spy on imported objects; use `vi.mock('./module', { spy: true })`
- Export methods to modify internal values rather than mocking variables

## Key Points

- Use Browser Mode for real browser testing (not jsdom/happy-dom environments)
- Playwright is recommended for CI
- v4: Providers accept objects, not strings
- v4: Import from `vitest/browser` not `@vitest/browser/context`
- Use projects to run both node and browser tests in same config

<!--
Source references:
- https://vitest.dev/guide/browser/
-->
