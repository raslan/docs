# Build a Chrome Extension with Vite, TypeScript, TailwindCSS and React

## Create a Vite application with React and Typescript

```
yarn create vite
```

Select "React" and "Typescript + SWC" when prompted.

cd into your folder

## Add CRXJS and other extension utilities to the project

```bash
yarn add @crxjs/vite-plugin@beta rollup-plugin-zip vite-tsconfig-paths -D
```

Let's edit our two tsconfig files to enable our development to be smoother

Edit `tsconfig.node.json`

```json
{
  "compilerOptions": {
    "composite": true,
    "module": "ESNext",
    "moduleResolution": "Node",
    "allowSyntheticDefaultImports": true,
    "resolveJsonModule": true,
    "skipLibCheck": true
  },
  "include": ["vite.config.ts", "manifest.json"]
}
```

Edit `tsconfig.json`

```json
{
  "compilerOptions": {
    "target": "ESNext",
    "useDefineForClassFields": true,
    "lib": ["DOM", "DOM.Iterable", "ESNext"],
    "allowJs": false,
    "skipLibCheck": true,
    "esModuleInterop": false,
    "allowSyntheticDefaultImports": true,
    "strict": true,
    "forceConsistentCasingInFileNames": true,
    "module": "ESNext",
    "moduleResolution": "Node",
    "resolveJsonModule": true,
    "isolatedModules": true,
    "noEmit": true,
    "jsx": "react-jsx"
  },
  "include": ["src"],
  "references": [{ "path": "./tsconfig.node.json" }]
}
```

## Configure Vite

Edit `vite.config.ts` in the root of the project

```ts
import { defineConfig, Plugin } from 'vite';
import react from '@vitejs/plugin-react-swc';
import { crx } from '@crxjs/vite-plugin';
import manifest from './manifest.json';
import tsconfigPaths from 'vite-tsconfig-paths';
import zip from 'rollup-plugin-zip';

// https://vitejs.dev/config/
export default defineConfig({
  plugins: [
    tsconfigPaths(),
    react(),
    crx({ manifest }),
    zip() as unknown as Plugin,
  ],
});
```

## Add TailwindCSS

Install tailwindcss and its peer dependencies with yarn

```bash
yarn add -D tailwindcss postcss autoprefixer
```

Create a tailwind configuration file and postcss configuration

```bash
yarn tailwindcss init -p
```

## Adding a Content Script and Background Script

## Adding an options page for fullscreen functionality (optional)

## Load the extension

## Persisting extension state

## Communication between React and the content script

## Sharing state between React and the content script

Now you can begin developing your extension, if you have a specific idea in mind or previous experience with browser extensions you may stop here. However, in the next section I'll walk you through how to build a small scale version of my own extension [Throwaway](https://throwaway.raslan.dev) to generate and read a temporary email's inbox as well as autofill registration/login fields.

## Reading emails

## Displaying emails

## Autofilling form fields on the page

## Conclusion