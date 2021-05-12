# vue3-smooth-scrollbar

[![Test build](https://github.com/rafalolszewski94/vue3-smooth-scrollbar/actions/workflows/test-build.yml/badge.svg)](https://github.com/rafalolszewski94/vue3-smooth-scrollbar/actions/workflows/test-build.yml)

Based on [vue-smooth-scrollbar](https://github.com/BlackBP/vue-smooth-scrollbar)

## Requirements
- Vue 3

## Installation

NPM
```bash
npm install vue3-smooth-scrollbar  #  yarn add vue3-smooth-scrollbar
```

## Usage

### Custom main scrollbar

**App.vue**
```vue
<template>
  <scrollbar>
    <!-- content -->
  </scrollbar>
</template>

<script>
import Scrollbar from "vue3-smooth-scrollbar";

export default {
  components: { Navbar, Scrollbar },
};
</script>
```

**Add to your styles**
```scss
body {
  overflow: hidden;
}

// Assuming you're using default mount element
#app {
  height: 100vh;
  overflow: auto;
}
```


## Development

#### Project setup
```
npm run install
```

#### Compiles and hot-reloads for development
```
npm run serve
```

#### Compiles and minifies for production
```
npm run build
```

#### Lints and fixes files
```
npm run lint
```
