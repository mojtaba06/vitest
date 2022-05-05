# Configuring Vitest

## Configuration

`vitest` will read your root `vite.config.ts` when it is present to match with the plugins and setup as your Vite app. If you want to have a different configuration for testing, you could either:

- Create `vitest.config.ts`, which will have the higher priority
- Pass `--config` option to CLI, e.g. `vitest --config ./path/to/vitest.config.ts`
- Use `process.env.VITEST` or `mode` property on `defineConfig` (will be set to `test` if not overridden) to conditionally apply different configuration in `vite.config.ts`

To configure `vitest` itself, add `test` property in your Vite config. You'll also need to add a reference to Vitest types using a [triple slash command](https://www.typescriptlang.org/docs/handbook/triple-slash-directives.html#-reference-types-) at the top of your config file, if you are importing `defineConfig` from `vite` itself.

using `defineConfig` from `vite` you should follow this:

```ts
/// <reference types="vitest" />
import { defineConfig } from 'vite'

export default defineConfig({
  test: {
    // ...
  },
})
```

using `defineConfig` from `vitest/config` you should follow this:

```ts
import { defineConfig } from 'vitest/config'

export default defineConfig({
  test: {
    // ...
  },
})
```

You can retrieve Vitest's default options to expand them if needed:

```ts
import { configDefaults, defineConfig } from 'vitest/config'

export default defineConfig({
  test: {
    exclude: [...configDefaults.exclude, 'packages/template/*'],
  },
})
```

## Options

### include

- **Type:** `string[]`
- **Default:** `['**/*.{test,spec}.{js,mjs,cjs,ts,mts,cts,jsx,tsx}']`

Include globs for test files

### exclude

- **Type:** `string[]`
- **Default:** `['**/node_modules/**', '**/dist/**', '**/cypress/**', '**/.{idea,git,cache,output,temp}/**']`

Exclude globs for test files

### deps

- **Type:** `{ external?, inline? }`

Handling for dependencies inlining or externalizing

#### deps.external

- **Type:** `(string | RegExp)[]`
- **Default:** `['**\/node_modules\/**','**\/dist\/**']`

Externalize means that Vite will bypass the package to native Node. Externalized dependencies will not be applied Vite's transformers and resolvers, so they do not support HMR on reload. Typically, packages under `node_modules` are externalized.

#### deps.inline

- **Type:** `(string | RegExp)[]`
- **Default:** `[]`

Vite will process inlined modules. This could be helpful to handle packages that ship `.js` in ESM format (that Node can't handle)

#### deps.fallbackCJS

- **Type** `boolean`
- **Default:** `false`

When a dependency is a valid ESM package, try to guess the cjs version based on the path.

This might potentially cause some misalignment if a package has different logic in ESM and CJS mode.

#### deps.interopDefault

- **Type:** `boolean`
- **Default:** `true`

Interpret CJS module's default as named exports.

### globals

- **Type:** `boolean`
- **Default:** `false`

By default, `vitest` does not provide global APIs for explicitness. If you prefer to use the APIs globally like Jest, you can pass the `--globals` option to CLI or add `globals: true` in the config.

```ts
// vite.config.ts
import { defineConfig } from 'vitest/config'

export default defineConfig({
  test: {
    globals: true,
  },
})
```

To get TypeScript working with the global APIs, add `vitest/globals` to the `types` filed in your `tsconfig.json`

```json
// tsconfig.json
{
  "compilerOptions": {
    "types": ["vitest/globals"]
  }
}
```

If you are already using [`unplugin-auto-import`](https://github.com/antfu/unplugin-vue-components) in your project, you can also use it directly for auto importing those APIs.

```ts
// vite.config.ts
import { defineConfig } from 'vitest/config'
import AutoImport from 'unplugin-auto-import/vite'

export default defineConfig({
  plugins: [
    AutoImport({
      imports: ['vitest'],
      dts: true, // generate TypeScript declaration
    }),
  ],
})
```

### environment

- **Type:** `'node' | 'jsdom' | 'happy-dom'`
- **Default:** `'node'`

The environment that will be used for testing. The default environment in Vitest
is a Node.js environment. If you are building a web application, you can use
browser-like environment through either [`jsdom`](https://github.com/jsdom/jsdom)
or [`happy-dom`](https://github.com/capricorn86/happy-dom) instead.

By adding a `@vitest-environment` docblock or comment at the top of the file,
you can specify another environment to be used for all tests in that file:

Docblock style:

```js
/**
 * @vitest-environment jsdom
 */

test('use jsdom in this test file', () => {
  const element = document.createElement('div')
  expect(element).not.toBeNull()
})
```

Comment style:

```js
// @vitest-environment happy-dom

test('use happy-dom in this test file', () => {
  const element = document.createElement('div')
  expect(element).not.toBeNull()
})
```

For compatibility with Jest, there is also a `@jest-environment`:

```js
/**
 * @jest-environment jsdom
 */

test('use jsdom in this test file', () => {
  const element = document.createElement('div')
  expect(element).not.toBeNull()
})
```

### update

- **Type:** `boolean`
- **Default:** `false`

Update snapshot files

### watch

- **Type:** `boolean`
- **Default:** `true`

Enable watch mode

### root

- **Type:** `string`

Project root

### reporters

- **Type:** `Reporter | Reporter[]`
- **Default:** `'default'`

Custom reporters for output. Reporters can be [a Reporter instance](https://github.com/vitest-dev/vitest/blob/main/packages/vitest/src/types/reporter.ts) or a string to select built in reporters:

  - `'default'` - collapse suites when they pass
  - `'verbose'` - keep the full task tree visible
  - `'dot'` -  show each task as a single dot
  - `'junit'` - JUnit XML reporter
  - `'json'` -  give a simple JSON summary
  - path of a custom reporter (e.g. `'./path/to/reporter.ts'`, `'@scope/reporter'`)

### outputFile

- **Type:** `string | Record<string, string>`

Write test results to a file when the `--reporter=json` or `--reporter=junit` option is also specified.
By providing an object instead of a string you can define individual outputs when using multiple reporters.

### threads

- **Type:** `boolean`
- **Default:** `true`

Enable multi-threading using [tinypool](https://github.com/Aslemammad/tinypool) (a lightweight fork of [Piscina](https://github.com/piscinajs/piscina))

### maxThreads

- **Type:** `number`
- **Default:** _available CPUs_

Maximum number of threads

### minThreads

- **Type:** `number`
- **Default:** _available CPUs_

Minimum number of threads

### testTimeout

- **Type:** `number`
- **Default:** `5000`

Default timeout of a test in milliseconds

### hookTimeout

- **Type:** `number`
- **Default:** `10000`

Default timeout of a hook in milliseconds

### silent

- **Type:** `boolean`
- **Default:** `false`

Silent mode

### setupFiles

- **Type:** `string | string[]`

Path to setup files. They will be run before each test file.

You can use `process.env.VITEST_WORKER_ID` (integer-like string) inside to distinguish between threads (will always be `1`, if run with `threads: false`).

### globalSetup

- **Type:** `string | string[]`

Path to global setup files, relative to project root

A global setup file can either export named functions `setup` and `teardown` or a `default` function that returns a teardown function ([example](https://github.com/vitest-dev/vitest/blob/main/test/global-setup/vitest.config.ts)).

::: info
Multiple globalSetup files are possible. setup and teardown are executed sequentially with teardown in reverse order.
:::


### watchIgnore

- **Type:** `(string | RegExp)[]`
- **Default:** `['**\/node_modules\/**', '**\/dist/**']`

Pattern of file paths to be ignore from triggering watch rerun

### isolate

- **Type:** `boolean`
- **Default:** `true`

Isolate environment for each test file

### coverage

- **Type:** `C8Options`
- **Default:** `undefined`

Coverage options

### testNamePattern

- **Type** `string | RegExp`

Run tests with full names matching the pattern.
If you add `OnlyRunThis` to this property, tests containing the word `OnlyRunThis` in the test name will be skipped.

```js
import { expect, test } from 'vitest'

// run
test('OnlyRunThis', () => {
  expect(true).toBe(true)
})

// skipped
test('doNotRun', () => {
  expect(true).toBe(true)
})
```

### open

- **Type:** `boolean`
- **Default:** `false`

Open Vitest UI (WIP)

### api

- **Type:** `boolean | number`
- **Default:** `false`

Listen to port and serve API. When set to true, the default port is 55555

### clearMocks

- **Type:** `boolean`
- **Default:** `false`

Will call `.mockClear()` on all spies before each test

### mockReset

- **Type:** `boolean`
- **Default:** `false`

Will call `.mockReset()` on all spies before each test

### restoreMocks

- **Type:** `boolean`
- **Default:** `false`

Will call `.mockRestore()` on all spies before each test

### transformMode

- **Type:** `{ web?, ssr? }`

Determine the transform method of modules

#### transformMode.ssr

- **Type:** `RegExp[]`
- **Default:** `[/\.([cm]?[jt]sx?|json)$/]`

Use SSR transform pipeline for the specified files.<br>
Vite plugins will receive `ssr: true` flag when processing those files.

#### transformMode&#46;web

- **Type:** `RegExp[]`
- **Default:** *modules other than those specified in `transformMode.ssr`*

First do a normal transform pipeline (targeting browser), then then do a SSR rewrite to run the code in Node.<br>
Vite plugins will receive `ssr: false` flag when processing those files.

When you use JSX as component models other than React (e.g. Vue JSX or SolidJS), you might want to config as following to make `.tsx` / `.jsx` transformed as client-side components:

```ts
import { defineConfig } from 'vitest/config'

export default defineConfig({
  test: {
    transformMode: {
      web: [/\.[jt]sx$/],
    },
  },
})
```

### snapshotFormat

- **Type:** `PrettyFormatOptions`

Format options for snapshot testing.

### mode

- **Type:** `string`
- **Default:** `test`

Overrides Vite mode.

### changed

- **Type**: `boolean | string`
- **Default**: false

Run tests only against changed files. If no value is provided, it will run tests against uncomitted changes (includes staged and unstaged).

To run tests against changes made in last commit, you can use `--changed HEAD~1`. You can also pass commit hash or branch name.

### resolveSnapshotPath

- **Type**: `(testPath: string, snapExtension: string) => string`
- **Default**: stores snapshot files in `__snapshots__` directory

Overrides default snapshot path. For example, to store snapshots next to test files:

```ts
export default {
  test: {
    resolveSnapshotPath: (testPath, snapExtension) => testPath + snapExtension,
  },
}
```

### logHeapUsage

- **Type**: `boolean`
- **Default**: `false`

Show heap usage after each test. Usefull for debugging memory leaks.
