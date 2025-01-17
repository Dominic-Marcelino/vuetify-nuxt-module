# Server Side Rendering (SSR)

Vuetify Nuxt Module supports [SSR](https://nuxt.com/docs/api/configuration/nuxt-config#ssr) out of the box. It will automatically detect if you are using SSR and configure Vuetify accordingly.

## Vuetify SASS Variables

If you are customising Vuetify SASS Variables via [configFile](https://github.com/vuetifyjs/vuetify-loader/tree/master/packages/vite-plugin#customising-variables) module option with SSR enabled, you have to disable `experimental.inlineSSRStyles` in your Nuxt config file, otherwise you will get an error when building your application:
```ts
// Nuxt config file
export default defineNuxtConfig({
  ssr: true,
  modules: ['vuetify-nuxt-module'],
  vuetify: {
    moduleOptions: {
      /* other module options */
      styles: { configFile: '/settings.scss' }
    },
    vuetifyOptions: {
      /* vuetify options */
    }  
  },
  experimental: {
    inlineSSRStyles: false
  }
})
```

## Vuetify Themes

If you're using multiple Vuetify Themes with SSR enabled, Vuetify [useTheme](https://vuetifyjs.com/en/api/use-theme/) will not work since there is no way to know which theme to use in the server (the server will use the default theme). You will need to add some logic in the client to restore the theme after hydration.

For example, if you want to use `dark` and `light` Vuetify Themes restoring the initial value using `prefers-color-scheme` and `localStorage`, you can use [useDark](https://vueuse.org/core/useDark/) and [useToogle](https://vueuse.org/shared/useToggle/) composables from VueUse in the following way:
```ts
// composables/useCustomTheme.ts
export function useCustomTheme() {
  const { $vuetify } = useNuxtApp()

  const isDark = useDark({
    valueDark: 'dark',
    valueLight: 'light',
    initialValue: 'light',
    onChanged: (dark: boolean) => {
      $vuetify.theme.global.name.value = dark ? 'dark' : 'light'
    },
  })

  const toggle = useToggle(isDark)

  return { isDark, toggle }
}
```

then, in your `App.vue` or layout templates, you can use previous composable to restore the theme:
```vue
<script setup>
import { useCustomTheme } from '~/composables/useCustomTheme'

const { isDark } = useCustomTheme()
</script>

<template>
  <VApp :theme="isDark ? 'dark' : 'light'">
    <!-- your content here -->
  </VApp>
</template>
```

## Vuetify Display

If you're using Vuetify [useDisplay](https://vuetifyjs.com/en/api/use-display/) composable with SSR enabled, there is only one way for the server to get the client's width and height (still in draft): use the [Sec-CH-Viewport-Width](https://wicg.github.io/responsive-image-client-hints/#sec-ch-viewport-width) and [Sec-CH-Viewport-Height](https://wicg.github.io/responsive-image-client-hints/#sec-ch-viewport-height) headers respectively, will not work for the initial request.
