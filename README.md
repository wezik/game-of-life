# game-of-life

This project implements Conway's Game of Life using Vue and the composition API. [Learn more about Conway's Game of Life](https://en.wikipedia.org/wiki/Conway%27s_Game_of_Life).

## Fun fact

While the implementation in this code may not be optimal, especially in terms of DOM manipulation (kind of done on purpose), it efficiently manages performance by focusing only on active cells and their neighbors. By keeping track of changes in a set of active cells, the code minimizes the impact on rendering performance, making it fast even with thousands of rendered divs.

## Recommended IDE Setup

[VSCode](https://code.visualstudio.com/) + [Volar](https://marketplace.visualstudio.com/items?itemName=Vue.volar) (and disable Vetur) + [TypeScript Vue Plugin (Volar)](https://marketplace.visualstudio.com/items?itemName=Vue.vscode-typescript-vue-plugin).

## Customize configuration

See [Vite Configuration Reference](https://vitejs.dev/config/).

## Project Setup

```sh
npm install
```

### Compile and Hot-Reload for Development

```sh
npm run dev
```

### Compile and Minify for Production

```sh
npm run build
```
