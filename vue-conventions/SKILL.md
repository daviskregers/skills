---
name: vue-conventions
description: >
  Vue template conventions: directive ordering, no inline styles.
  Use when writing or editing .vue files.
---

Vue template rules. Apply to all `.vue` files.

## Directive Ordering

`v-if`, `v-else-if`, `v-else`, `v-for`, `v-show` MUST be first attribute on element. Before `class`, `id`, `style`, any other attr.

```vue
<!-- Bad -->
<div class="foo" v-if="condition">

<!-- Good -->
<div v-if="condition" class="foo">
```

## No Inline Styles

Never use `style="..."` attribute. Use class, define styles in `<style>` block.

```vue
<!-- Bad -->
<div class="alert" style="margin-top: 20px;">

<!-- Good -->
<div class="alert my-component-alert">
<!-- with .my-component-alert { margin-top: 20px; } in <style> -->
```
