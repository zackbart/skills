---
name: debugging
description: Debug Vitest tests with VS Code, IntelliJ, and Chrome DevTools
---

# Debugging

## VS Code: JavaScript Debug Terminal (Quickest)

Open a JavaScript Debug Terminal (`Cmd+Shift+P` > "JavaScript Debug Terminal") and run:

```bash
npm run test
# or directly
vitest
```

Breakpoints in your test files and source code will be hit automatically.

## VS Code: Launch Configuration

Create `.vscode/launch.json`:

```json
{
  "version": "0.2.0",
  "configurations": [
    {
      "type": "node",
      "request": "launch",
      "name": "Debug Current Test File",
      "autoAttachChildProcesses": true,
      "skipFiles": ["<node_internals>/**", "**/node_modules/**"],
      "program": "${workspaceRoot}/node_modules/vitest/vitest.mjs",
      "args": ["run", "${relativeFile}"],
      "smartStep": true,
      "console": "integratedTerminal"
    }
  ]
}
```

Open a test file, press F5 to debug. `smartStep` skips generated code.

## Browser Mode Debugging (VS Code)

CLI approach:

```bash
vitest --inspect-brk --browser --no-file-parallelism
```

Config approach:

```ts
import { defineConfig } from 'vitest/config'
import { playwright } from '@vitest/browser-playwright'

export default defineConfig({
  test: {
    inspectBrk: true,
    fileParallelism: false,
    browser: {
      provider: playwright(),
      instances: [{ browser: 'chromium' }],
    },
  },
})
```

### Compound Configuration

Debug both Node and browser simultaneously:

```json
{
  "version": "0.2.0",
  "configurations": [
    {
      "type": "node",
      "request": "launch",
      "name": "Run Vitest Browser",
      "program": "${workspaceRoot}/node_modules/vitest/vitest.mjs",
      "console": "integratedTerminal",
      "args": ["--inspect-brk", "--browser", "--no-file-parallelism"]
    },
    {
      "type": "chrome",
      "request": "attach",
      "name": "Attach to Vitest Browser",
      "port": 9229
    }
  ],
  "compounds": [
    {
      "name": "Debug Vitest Browser",
      "configurations": ["Attach to Vitest Browser", "Run Vitest Browser"],
      "stopAll": true
    }
  ]
}
```

## IntelliJ IDEA

1. Create a Vitest run configuration
2. Set working directory to project root
3. Run in debug mode (bug icon or Shift+F9)

## Chrome DevTools (No IDE)

```bash
# Break before test execution
vitest --inspect-brk --no-file-parallelism

# Custom port
vitest --inspect-brk=127.0.0.1:3000 --no-file-parallelism
```

Then open `chrome://inspect` in Chrome and connect to the debugger. Default port is `9229`.

## Essential CLI Flags for Debugging

| Flag | Purpose |
|------|---------|
| `--inspect` | Enable Node inspector |
| `--inspect-brk` | Break before execution starts |
| `--test-timeout=0` | Prevent timeout while paused at breakpoints |
| `--no-file-parallelism` | Run one file at a time (required for debugging) |
| `--isolate false` | Keep debugger open during watch mode re-runs |

Combine for best debugging experience:

```bash
vitest --inspect-brk --no-file-parallelism --test-timeout=0
```

## Key Points

- JavaScript Debug Terminal is the fastest way to start debugging in VS Code
- Always use `--no-file-parallelism` when debugging to avoid multiple worker processes
- Set `--test-timeout=0` to prevent tests from timing out at breakpoints
- Use `--isolate false` with watch mode so the debugger stays attached between re-runs
- Default inspect port is `9229`; customize with `--inspect-brk=host:port`

<!--
Source references:
- https://vitest.dev/guide/debugging.html
-->
