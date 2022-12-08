# NativeScript Vue Router

[npm-url]: https://npmjs.org/package/router-vue-native
[npm-image]: http://img.shields.io/npm/v/router-vue-native.svg

[![NPM version][npm-image]][npm-url] [![Blazing Fast](https://badgen.now.sh/badge/speed/blazing%20%F0%9F%94%A5/green)](https://github.com/MattCCC/nativescript-vue-router-extended) [![Code Coverage](https://badgen.now.sh/badge/coverage/0.00/blue)](https://github.com/MattCCC/nativescript-vue-router-extended) [![npm downloads](https://img.shields.io/npm/dm/nativescript-vue-router-extended.svg?style=flat-square)](http://npm-stat.com/charts.html?package=nativescript-vue-router-extended) [![install size](https://packagephobia.now.sh/badge?p=nativescript-vue-router-extended)](https://packagephobia.now.sh/result?p=nativescript-vue-router-extended)

[nativescript-vue-router-extended](https://github.com/mattCCC/nativescript-vue-router-extended)

NativeScript Vue Router brings easier routing handling to mobile apps, with an API compatibility with web version of Vue Router.

Please file an issue or make a PR if you spot any problems or you have any further requests regarding the functionality.

## Table of Contents

- [Features](#features)
- [Prerequisites / Requirements](#prerequisites-requirements)
- [Installation](#installation)
- [Usage & Examples](#usage-examples)
- [New hooks for pages](#new-hooks-for-pages)
- [TypeScript](#typescript)
- [API Differences to Web](#api-differences-to-web)
- [Troubleshooting](#troubleshooting)

## Features

- Same hooks and guards for mobile and web
- Additional action dispatcher to dispatch actions to store automatically when changing routes
- Vue-Router 4 API compatibility
- NativeScript-Vue compatible
- TypeScript Support out of the box

## Prerequisites / Requirements

Nativescript 7.1+ is required for the plugin to run properly. It might be working on previous NS6 although the plugin is no longer officially supported for NativeScript 6.

## Installation

```javascript
ns plugin add router-vue-native

or

npm install router-vue-native

or

yarn add router-vue-native
```

[![NPM](https://nodei.co/npm/router-vue-native.png)](https://npmjs.org/package/router-vue-native)

## Usage & Examples

To use this plugin you need to import it and initialize the router using `createRouter()` function. Global and per component Vue-Router hooks and guards are supported.

# Vue 3 example

### Add your router to vue app
```ts
// app.ts

import { createApp } from 'nativescript-vue';
import App from './App.vue';
import {router} from "~/plugins/router";
const app = createApp(App);
app.use(router)
app.start();
```

### Create router
```ts
// /plugins/router.ts

import {createRouter} from "router-vue-native";
import Home from "~/views/Home.vue";
import Login from "~/views/Login.vue";

const routes = [
    {
        path: "/",
        component: Home,
    },
    {
        path: "/login",
        component: Login,
    }
];

const router = createRouter(
    {routes},
);

export {
    router
}
```

### Define the default path of your application
```vue
// App.vue
<template>
  <RouterView defaultRoute="/login"></RouterView>
</template>

// OR

<script setup lang="ts">
const getDefaultRouteExample = () => {
  if (isLoginUser) {
    return "/login"
  } else {
    return "/"
  }
}
</script>
<template>
  <RouterView :defaultRoute="getDefaultRouteExample"></RouterView>
</template>
```
### Use on setup template
```vue
<script lang="ts" setup>
import {useRouter} from "router-vue-native";

// get router
const router = useRouter();

const onNavigete = () => {
  // navigate
  router.push("/your_path")
}

</script>
```

## Vue 2 and full options


```javascript
import Vue from "nativescript-vue";
import { createRouter } from "nativescript-vue-router-extended";

// Initialize Example Routes
import Home from "./pages/Home.vue";
import Login from "./pages/Login.vue";

const routes = [
  {
    path: "/",
    component: Home,

    // Optional
    meta: {
      isVertical: true,
      // Example actions to dispatch automatically when page is visited
      // Remember that you need to implement actions in your Vuex store
      store: {
        // Call action to hide navigation buttons
        showNavigationButtons: false,

        // Call "showMovies" action in "categories" module with payload "all"
        "categories/showMovies": "all",

        // Call action without payload
        showNavigationButtons: null,
      },
    },
    {
        path: "/login",
        component: Login,
    }

    // Optional, per route guards are supported
    // More info: https://next.router.vuejs.org/guide/advanced/navigation-guards.html#per-route-guard
    beforeEnter: (to, from) => {
      if (to.props.passUser) {
         // Continue navigation
         return true;
      }

      // Reject the navigation
      return false;
    },
  },
];

// Initialize Router
// Vue-Router settings are in 1st argument. 2nd one is used for extra NativeScript related settings
const router = createRouter(
  { routes },
  {
    // Optional settings below

    // Set first page to redirect to when there's no page to redirect back to
    routeBackFallbackPath: "/login",

    // Do something straight before navigation or adjust NS routing settings
    routeToCallback: (to, options) => {
      // For example, change page navigation transition for the vertical on iOS
      if (to.meta.isVertical) {
        options.transition = {
          name: "fade",
        };
      }
    },

    // Do something straight before navigation or adjust NS routing settings
    routeBackCallback: (_to, options) => {
      // Do something...
    },

    // Set a custom logger (console.log by default)
    logger: console.log,

    // Set custom frame, by default it's taken from @nativescript/core/ui/frame
    frame: Frame,
  }
);

// Register a global guard (optional)
// You can also use global router.beforeResolve guard and router.afterEach hook
router.beforeEach((to) => {
  // For example, verify per route access rules
  if (!canAccessRoute(to)) {
    return false;
  }

  return true;
});

// From now on you can access this.$router, this.$routeBack and special this.$routeTo inside of the components, example:
this.$routeTo("/movies", {
  // Clear History is a NativeScript setting
  clearHistory: true,
  
  // Route inside of custom Frame
  frame: "myFrameId",

  // Pass props to the page
  props: {
    movieId: 12,
  },
});
```



## New hooks for pages

You can use hooks directly on particular pages. It is discouraged to use them inside of mixins or components for the time being. Example below:

```javascript
<template>
    <Page>
    </Page>
</template>

<script>

export default {
    name: 'movies',

    beforeRouteEnter(to, from, next) {
        // Do something before to enter to the route
        next((vm) => {
            // Do something once navigation is done
            // Instead of `this`, use `vm`
        });
    },

    beforeRouteLeave() {
        // Do something before to leave the route
        // You can use `this` inside of this hook
    },

    beforeRouteUpdate() {
        // Do something before to leave the route
        // before redirecting to another route with same path
        // You can use `this` inside of this hook
    },

    mounted() {
       // Output current route object with name, path etc.
       console.log(this.$route);
    },
};
</script>
```

| NS Event       | Mapped as           | Description                                                                                                                                                                                                                                                                                                  |
| -------------- | ------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| navigatingFrom | `beforeRouteLeave`  | Before user leaves the route                                                                                                                                                                                                                                                                                 |
| navigatingTo   | `beforeRouteEnter`  | Before user enters a route                                                                                                                                                                                                                                                                                   |
| -              | `beforeRouteUpdate` | Before route is changed but view remains the same. This can happen when path is exactly the same but you change e.g. passed prop to the route. Please refer to Vue-Router docs for more details.                                                                                                             |
| navigatedTo    | `beforeRouteEnter`  | To trigger it properly you need to access component instance. You can use `next(vm => ...)` callback within `beforeRouteEnter()`. Please check Vue-Router docs for more details.                                                                                                                             |
| navigatedFrom  | -                   | This event is tricky to control for developers. There is no exact mapping of it in the router. For store state cleanup use build-in meta dispatcher instead. For component state you could opt for using `beforeRouteLeave()`. |

## TypeScript Support
If you need a TS support and it's not detected automatically in your project for some reason, you can use [typings/shims.vue.d.ts](https://github.com/MattCCC/nativescript-vue-router-extended/blob/master/src/typings/shims-vue.d.ts) to bring proper support in .vue files. You can specify it in your `shims.vue.d.ts` file (attention! Please ensure that path is relative to your node_modules directory):
```
/// <reference path="./node_modules/nativescript-vue-router-extended/typings/shims-vue.d.ts" />
```

## API Differences to Web

<b>Vue Router compatibility</b>

This plugin aims for compatibility with Vue Router 3+ and Vue Router 4+. Both should be easily supported. Please refer to [Vue Router Docs](https://next.router.vuejs.org/guide/) for more information.

<b>DOM & related hooks</b>

There are some limitations like lack of DOM accessibility and related hooks and guards.

<b>RouterLink Component</b>

There's a lack of <router-link /> component due to performance reasons.

<b>Passing props to pages</b>

All props are passed automatically to <Page /> components. Therefore you don't need to use `props: true` in your routes list.

<b>Meta Dispatcher</b>

An additional option that allows you to dispatch actions to Vuex store directly from routes list using `meta.store` object. To utilize it you may need to define `Vue.prototype.$store = store;` after to register your Vue store.


## License

Apache License Version 2.0, January 2004

## Troubleshooting

Please file an issue. You can use the template but it is not required. Just simply write what is your issue so we can try to reproduce and fix it.
