# uxagent-homepage

This template should help get you started developing with Vue 3 in Vite.

## Prerequisites

- Node.js >= 18.12.0 (Recommended: Node.js 22)
- Yarn package manager

### Check your Node.js version
```sh
node --version
```

If you need to upgrade Node.js, we recommend using [nvm](https://github.com/nvm-sh/nvm):
```sh
# Install Node.js 22 (recommended)
nvm install 22
nvm use 22
```

Or simply use the `.nvmrc` file in this project:
```sh
nvm use
```

## Recommended IDE Setup

[VSCode](https://code.visualstudio.com/) + [Volar](https://marketplace.visualstudio.com/items?itemName=Vue.volar) (and disable Vetur).

## Type Support for `.vue` Imports in TS

TypeScript cannot handle type information for `.vue` imports by default, so we replace the `tsc` CLI with `vue-tsc` for type checking. In editors, we need [Volar](https://marketplace.visualstudio.com/items?itemName=Vue.volar) to make the TypeScript language service aware of `.vue` types.

## Customize configuration

See [Vite Configuration Reference](https://vite.dev/config/).

## Project Setup

```sh
yarn
```

### Compile and Hot-Reload for Development

```sh
yarn dev
```

### Type-Check, Compile and Minify for Production

```sh
yarn build
```

### Lint with [ESLint](https://eslint.org/)

```sh
yarn lint
```
