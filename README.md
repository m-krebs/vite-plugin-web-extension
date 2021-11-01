# Vite Plugin Web Extension

A nearly 0 config plugin for developing browser extensions using [Vite](https://vitejs.dev/). It supports:

- Languages
  - Javascript
  - Typescript
- Frameworks
  - Vue
  - React
  - Svelte
  - And anything else with a plugin for Vite!
- Features
  - Build based on your `manifest.json`
  - Lightning fast production builds
  - Chrome & Firefox support
  - TODO: HTML page HMR
  - TODO: Background/content script reloading

```ts
// vite.config.ts
import browserExtension from "vite-plugin-web-extension";

export default defineConfig({
  plugins: [
    browserExtension({
      manifest: () => require("./manifest.json"),
      assets: "assets",
    }),
  ],
});
```

And that's it! All entry points are read from the `manifest.json` (or the [`additionalInputs`](#additional-inputs) option)

```bash
npm i -D vite-plugin-web-extension
```

## Roadmap

- [x] Build for production &emsp; :tada: `v0.1.0` :tada:
- [x] CSS inputs & generated files &emsp; :tada: `v0.2.0` :tada:
- [x] Dev mode with automatic reload &emsp; :tada: `v0.3.0` :tada:
- [x] Manifest V3 support &emsp; :tada: `v0.5.0` :tada:
- [ ] Browser specific flags in the manifest
- [ ] HMR

## Setup and Usage

Lets say your project looks like this:

<pre>
<strong>dist/</strong>
   <i>build output...</i>
<strong>src/</strong>
   <strong>assets/</strong>
      <i>icon-16.png</i>
      <i>icon-48.png</i>
      <i>icon-128.png</i>
   <strong>background/</strong>
      <i>index.ts</i>
   <strong>popup/</strong>
      <i>index.html</i>
   <i>manifest.json</i>
<i>package.json</i>
<i>vite.config.ts</i>
<i>...</i>
</pre>

In your `vite.config.ts` and `src/manifest.json`, **make sure all paths are relative to your Vite `root`**!

In this example, we set our it to `"src"`. In the `vite.config.ts` file, the only field that is relative to the root is your `assets` directory.

```ts
// vite.config.ts
import browserExtension, { readJsonFile } from "vite-plugin-web-extension";

export default defineConfig({
  root: "src",
  build: {
    // Configure our outputs - nothing special, this is normal vite config
    outDir: path.resolve(__dirname, "dist"),
    emptyOutDir: true,
  },
  plugins: [
    browserExtension({
      manifest: () =>
        readJsonFile(path.resolve(__dirname, "src/manifest.json")),
      assets: "assets",
    }),
  ],
});
```

> `readJsonFile` is important for watch mode. If you do `require("./src/manifest.json")`, the value will be cached and changes won't show up

The manifest has a bit more. You should use relative paths for all entry points (HTML, JS, or TS), such as `browser_action.default_popup` or `background.scripts`.

**Your relative paths should also be the real file extension**! If you're using typescript, have any scripts end with their usual `.ts` extension. The plugin will transform the file extensions for you.

```jsonc
// src/manifest.json
{
  "name": "Example",
  "version": "1.0.0",
  "icons": {
    "16": "assets/icon-16.png",
    "48": "assets/icon-48.png",
    "128": "assets/icon-128.png"
  },
  "browser_action": {
    "default_icon": "assets/icon-128.png",
    "default_popup": "popup/index.html"
  },
  "background": {
    "scripts": "background/index.ts"
  }
}
```

:sparkler: And there you go!

Run `vite build` and you should see a fully compiled and working browser extension in your `dist/` directory!

### Adding Frontend Framewoks

If you want to add a framework like Vue or React, just add their plugin!

```ts
import vue from '@vitejs/plugin-vue'

export default defineConfig({
  ...
  plugins: [
    browserExtension({ ... }),
    vue(),
  ],
});
```

> The plugin order doesn't matter

## Advanced Features

### Watch Mode

To reload the extension when a file changes, run vite with the `--watch` flag

```bash
vite build --watch
```

To reload when you update your `manifest.json`, or any other files that aren't triggering reloads pass the `watchFilePaths` option. Use absolute paths for this

```ts
import path from "path";

export default defineConfig({
  ...
  plugins: [
    browserExtension({
      watchFilePaths: [
        path.resolve(__dirname, "src/manifest.json")
      ]
    }),
  ],
});
```

Do not run the dev server. It won't work until HMR is implemented:

```bash
# Don't do this! You'll get a warning and nothing will work
vite
```

### Additional Inputs

If you have have files that need to be included, but aren't listed in your `manifest.json`, you can add them via the `additionalInputs` option.

The paths should be relative to the Vite's `root`, just like the `assets` option.

```ts
// vite.config.ts
import browserExtension from "vite-plugin-web-extension";

export default defineConfig({
  root: "src",
  build: {
    // Configure our outputs - nothing special, this is normal vite config
    outDir: path.resolve(__dirname, "dist"),
    emptyOutDir: true,
  },
  plugins: [
    browserExtension({
      ...
      additionalInputs: [
        "content-scripts/injected-from-background.ts",
      ]
    }),
  ],
});
```

### CSS

For HTML entry points like popups or the options page, css is automatically output and refrenced in the built HTML. There's nothing you need to do!

#### Manifest `content_scripts`

For content scripts listed in your `manifest.json`, its a little more difficult. There are two ways to include CSS files:

1. You have a CSS file in your project
1. The stylesheet is generated by a framework like Vue or React

For the first case, that's simple! Make sure you have the relevant plugin installed to parse your stylesheet, and you're good!

```json
{
  "content_scripts": [
    {
      "matches": [...],
      "css": "content-scripts/some-style.css"
    }
  ]
}
```

For the second case, it's a little more involved. Say your content script at `content-scripts/overlay.ts`. This script is responsible for binding a Vue/React app to a webpage. When Vite compiles it, it will output two files: `dist/content-scripts/overlay.js` and `dist/content-scripts/overlay.css`.

In the content script section of your `manifest.json`, add add the path to this generated file, but prefix it with `generated:*`

```json
{
  "content_scripts": [
    {
      "matches": [...],
      "scripts": "content-scripts/overlay.ts",
      "css": "generated:content-scripts/overlay.css"
    }
  ]
}
```

This will tell the plugin that the file is already being generated for us, but that we still need it in our manifest so it is injected.

#### Browser API `tabs.executeScripts`

For content scripts injected programmatically, include path in the plugin's [`additionalInputs` option](#additional-inputs)

### Browser Specific Manifest Fields

TODO
