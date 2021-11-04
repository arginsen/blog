---
title: 前端 | vue3 项目的创建
date: 2021-10-2 18:45:00
tags: 
- Vue
- Frontend
categories: notes
photos:
    - /blog/img/vue.jpg
---

<br>

<!--more-->

# 创建项目

## 使用 vite 创建 vue3 项目

> 介绍: https://cn.vitejs.dev/guide

通过 npm 直接调用 vite 初始化一个 vue 模板的项目
模板可选: vue, vue-ts, react, react-ts 等

```bash
# npm 6.x
npm init vite@latest my-vue-app --template vue

# npm 7+, 需要额外的双横线：
npm init vite@latest my-vue-app -- --template vue

# yarn
yarn create vite my-vue-app --template vue
```

## 使用 create-vue 来创建 vue3 项目

> 介绍: https://github.com/vuejs/create-vue

此外，也可以使用 Vue 推出的新一代脚手架工具 create-vue 来快速创建 vue3 项目，

```bash
# 交互式创建
npm init vue@next

# 或者追加参数 添加 ts vuex vue-router 
npm init vue@next create-vue-test-args --ts --vuex --router
```

补充：源码中采用 minimist 来收集参数，可以直接追加参数来初始化 vue3 项目

```js
async function init() {
  const cwd = process.cwd()
  // possible options:
  // --default
  // --typescript / --ts
  // --jsx
  // --router / --vue-router
  // --vuex
  // --with-tests / --tests / --cypress
  // --force (for force overwriting)
  const argv = minimist(process.argv.slice(2), {
    alias: {
      typescript: ['ts'],
      'with-tests': ['tests', 'cypress'],
      router: ['vue-router']
    },
    // all arguments are treated as booleans
    boolean: true
  })

  // if any of the feature flags is set, we would skip the feature prompts
  // use `??` instead of `||` once we drop Node.js 12 support
  const isFeatureFlagsUsed =
    typeof (argv.default || argv.ts || argv.jsx || argv.router || argv.vuex || argv.tests) ===
    'boolean'

  let targetDir = argv._[0]
  const defaultProjectName = !targetDir ? 'vue-project' : targetDir

  const forceOverwrite = argv.force

  // ...
}
```

# 配置 vite

> 介绍: https://cn.vitejs.dev/config/

```ts
import type { UserConfig, ConfigEnv } from 'vite'
import vue from '@vitejs/plugin-vue'
import { loadEnv } from 'vite'
import { warpperEnv } from './build/utils'
import { createProxy } from './build/proxy'
import ElementPlus from "unplugin-element-plus/vite"

export default ({ mode }: ConfigEnv): UserConfig => {
  // https://cn.vitejs.dev/config/
  const root = process.cwd()

  // 获取环境变量
  const env = loadEnv(mode, root)
  const { VITE_PORT, VITE_PUBLIC_PATH, VITE_PROXY } = warpperEnv(env)

  return {
    base: VITE_PUBLIC_PATH,
    root,
    resolve: {
      alias: {
        '@/': new URL('./src/', import.meta.url).pathname,
        '#/': new URL('./types/', import.meta.url).pathname
      },
    },
    server: {
      https: false,
      port: VITE_PORT,
      host: true,
      proxy: createProxy(VITE_PROXY)
    },
    plugins: [
      vue(),
      ElementPlus({}),
    ],
    build: {
      // @ts-ignore
      sourcemap: false,
      brotliSize: false,
      // 消除打包大小超过500kb警告
      chunkSizeWarningLimit: 2000
    },
    define: {
      __INTLIFY_PROD_DEVTOOLS__: false
    }
  }
}
```

# 配置 ts

> 参考: https://v3.cn.vuejs.org/guide/typescript-support.html
> 使用: https://vue3js.cn/es6/typeScript.html

```json
{
  "compilerOptions": {
    "target": "esnext",
    "useDefineForClassFields": true,
    "module": "esnext",
    "moduleResolution": "node",
    "strict": true,
    "forceConsistentCasingInFileNames": true,
    "allowSyntheticDefaultImports": true,
    "strictFunctionTypes": false,
    "jsx": "preserve",
    "baseUrl": ".",
    "allowJs": true,
    "sourceMap": true,
    "resolveJsonModule": true,
    "esModuleInterop": true,
    "lib": ["esnext", "dom"],
    "types": ["node", "vite/client", "element-plus/global"],
    "typeRoots": ["./node_modules/@types/", "./types"],
    "paths": {
      "@/*": ["src/*"],
      "#/*": ["types/*"]
    }
  },
  "include": [
    "src/**/*.ts",
    "src/**/*.d.ts", 
    "src/**/*.tsx", 
    "src/**/*.vue",
    "types/**/*.d.ts",
    "types/**/*.ts",
    "vite.config.ts"
  ],
  "exclude": ["node_modules"]
}
```

# 安装插件

总览：

```json
// package.json
{
  // ...
  "dependencies": {
    "axios": "^0.23.0",
    "element-plus": "1.1.0-beta.24",
    "mockjs": "^1.1.0",
    "nprogress": "^0.2.0",
    "pinia": "^2.0.0-rc.10",
    "vue": "^3.2.20",
    "vue-router": "^4.0.12",
    "vue-types": "^4.1.0"
  },
  "devDependencies": {
    "@types/node": "^16.11.6",
    "@typescript-eslint/parser": "^5.2.0",
    "@vitejs/plugin-vue": "^1.9.3",
    "@vue/compiler-sfc": "^3.2.20",
    "@vue/eslint-config-typescript": "^7.0.0",
    "eslint": "^8.1.0",
    "eslint-config-prettier": "^8.3.0",
    "eslint-plugin-prettier": "^4.0.0",
    "eslint-plugin-vue": "^7.20.0",
    "prettier": "^2.4.1",
    "sass": "^1.38.0",
    "sass-loader": "^12.1.0",
    "stylelint": "^13.13.1",
    "stylelint-config-prettier": "^9.0.3",
    "stylelint-config-standard": "^22.0.0",
    "stylelint-order": "^4.1.0",
    "typescript": "^4.4.4",
    "unplugin-element-plus": "^0.1.3",
    "vite": "^2.6.10",
    "vue-tsc": "^0.3.0"
  }
}
```

## vue-router

> 安装: https://next.router.vuejs.org/zh/installation.html

```bash
npm install vue-router@4
```

> 变更: https://next.router.vuejs.org/zh/guide/migration/index.html

从 v3 到 v4 vue-router 原有的 API 大部分都没有变化

1. 安装过程

```ts
// main.ts
import App from "./App.vue";
import router from "./router";
import { createApp } from "vue";

async function bootstrap()  {
  const app = createApp(App)

  app.use(router)
  await router.isReady()

  app.mount("#app")
}

bootstrap()
```

```ts
// router/index.ts
import { createRouter } from 'vue-router'
import Layout from "/@/layout/index.vue";

const router = createRouter({
  history: createWebHashHistory(),
  routes: [
    {
      path: "/",
      name: "home",
      component: Layout,
      redirect: "/welcome",
      meta: {
        icon: "el-icon-s-home",
        showLink: true,
        rank: 0
      },
      children: [
        {
          path: "/welcome",
          name: "welcome",
          component: () => import("/@/views/welcome.vue"),
          meta: {
            title: "message.hshome",
            showLink: true
          }
        }
      ]
    }
  ],
})
```

应用场景:

```ts
import { useRouter } from "vue-router";

const router = useRouter()

const toPage = (info: Object): void => {
  storageSession.setItem("info", info);
  router.push("/");
};
```

2. 使用 `router.isReady().then(onSuccess).catch(onError)`



## vuex

> 安装: https://next.vuex.vuejs.org/zh/installation.html

vuex 为适配 vue3 推出了 vuex4

```bash
npm install vuex@next --save
```

> 变更: https://next.vuex.vuejs.org/zh/guide/migrating-to-4-0-from-3-x.html#%E5%AE%89%E8%A3%85%E8%BF%87%E7%A8%8B

原有的 API 几乎与 vuex3 版本的一致，在个别几处做了非兼容性处理：

1. 安装过程

```ts
// main.ts
import { createApp } from 'vue'
import { store } from '@/store'
import App from './App.vue'

const app = createApp(App)

app.use(store)

app.mount('#app')
```

使用 createStore 创建 store 实例:

```ts
// store.ts
import { createStore } from 'vuex'

export const store = createStore({
  state() {
    return {
      name: ''
    }
  },
  mutations: {
    setName(state, name) {
      state.name = name
    }
  },
  actions: {
    async getUserName({ commit }) {
      await userNameRequest().then((res) => {
        commit('setName', res.data.name)
      })
    }
  },
  plugins: process.env.NODE_ENV !== 'production' ? [createLogger({})] : []
})
```

应用场景：

```ts
<script setup>
import { useStore } from '@/store'

router.beforeEach(async (to, from, next) => {
  const store = useStore()
  await store.dispatch(getUserName)
  // ...
})

const name = computed(() => {
  return store.state.name
})
</script>
```

2. TS 支持

Vuex 4 删除了 `this.$store` 在 Vue 组件中的全局类型声明。当使用 TypeScript 时，必须声明自己的模块补充(module augmentation)， 以允许 `this.$store` 能被正确的类型化：

```ts
// vuex-shim.d.ts
import { ComponentCustomProperties } from 'vue'
import { Store } from 'vuex'

declare module '@vue/runtime-core' {
  // 声明自己的 store state
  interface State {
    count: number
  }

  interface ComponentCustomProperties {
    $store: Store<State>
  }
}
```

3. 打包产物和 Vue3 配套

4. createLogger 函数从核心模块直接导出

5. 新引入 useStore 组合式函数

## pinia

> 介绍: https://pinia.esm.dev/

```bash
npm install pinia@next
```

同 vuex 一样，pinia 也是 Vue 的状态管理工具

与 vuex 相比，pinia:

1. 没有 mutations
2. 不需要创建自定义的复杂包装器来支持 TS，所有东西东西都是类型化的
3. 无需动态添加 stores，默认动态
4. 不再有模块的嵌套结构，提供一个扁平的结构
5. 没有命名空间的模块
6. getters 就类似 vue 中的计算属性
7. actions 就相当于组件中的方法

注册：

```ts
// main.ts
import { createApp } from 'vue'
import { store } from '@/store'
import App from './App.vue'

const app = createApp(App)
app.use(store)
app.mount('#app')
```

应用：

```ts
// store/index.ts
import type { App } from 'vue';
import { createPinia } from 'pinia'

const store = createPinia()

export { store }
```

```ts
// store/user.ts
import type { UserInfo } from '#/store'
import { store } from '@/store'
import { defineStore } from 'pinia'
import { getAuthCache, setAuthCache } from '@/utils/auth'

interface UserState {
  userInfo: Nullable<UserInfo>;
}

export const useUserStore = defineStore({
  id: 'app-user',
  state: (): UserState => ({
    userInfo: null
  }),
  getters: {
    getUserInfo(): UserState {
      return this.userInfo || getAuthCache<UserState>(USER_INFO_KEY) || {}
    }
  },
  actions: {
    setUserInfo(info: UserInfo | null) {
      this.userInfo = info
      setAuthCache(USER_INFO_KEY, info);
    }
  }
})

// Need to be used outside the setup
export function useUserStoreWithOut() {
  return useUserStore(store);
}
```

```ts
// views/login.vue
<script lang="ts" setup>
import { onBeforeUnmount, onMounted, ref } from 'vue'
import { useUserStore } from '@/store/modules/user'

const userStore = useUserStore()
const userId = ref<Nullable<number | string>>(0)

onMounted(() => {
  userId.value = userStore.getUserInfo?.userId
})
</script>
```

## element-plus

> 介绍: https://element-plus.gitee.io/zh-CN/guide/design

element-plus 是 element-ui 支持 Vue3 推出的新一代 UI

vue3 使用需要注意:

1. 使用 VS 插件 Volar 作为规范检查时，需要在 tsconfig 配置：

```json
// tsconfig.json
{
  "compilerOptions": {
    // ...
    "types": ["element-plus/global"]
  }
}
```

2. 自动按需导入需要安装 `unplugin-vue-components` 插件，并在 vite 配置中注册插件

```ts
// vite.config.ts
import Components from 'unplugin-vue-components/vite'
import { ElementPlusResolver } from 'unplugin-vue-components/resolvers'

export default {
  plugins: [
    // ...
    Components({
      resolvers: [ElementPlusResolver()],
    }),
  ],
  // ...
}
```

3. 手动导入需要安装 `unplugin-element-plus` 插件，同时在 vite 配置插件

```ts
// vite.config.ts
import ElementPlus from 'unplugin-element-plus/vite'

export default {
  plugins: [ElementPlus()],
}
```

随后在项目中使用：

```ts
<script lang="ts">
  import { defineComponent } from 'vue'
  import { ElButton } from 'element-plus'
  export default defineComponent({
    components: { 
      ElButton
    },
    setup() {
      // ...
    }
  })
</script>
```

或者

```ts
<script lang="ts" setup>
  import { ElButton } from 'element-plus'
</script>
```

4. 注册 ElementPlus 插件

全局：

```ts
// main.ts
import { createApp } from 'vue'
import ElementPlus from 'element-plus'
import 'element-plus/dist/index.css'
import App from './App.vue'

const app = createApp(App)

app.use(ElementPlus)

app.mount("#app")
```

自动按需注册：

```ts
// main.ts
import { createApp } from 'vue'
import { ElButton } from 'element-plus'
import App from './App.vue'

const app = createApp(App)
app.config.globalProperties.$ELEMENT = {
  // options
}
app.use(ElButton)
```

## prettier

eslint 作为项目代码检测工具，在 eslint 的基础上，补充 prettier 插件来规范格式化代码，prettier 拥有简洁的配置项

首先配置 eslint

```js
// .eslintrc.js
module.exports = {
  root: true,
  env: {
    browser: true,
    node: true,
    es6: true,
  },
  parser: 'vue-eslint-parser',
  parserOptions: {
    parser: '@typescript-eslint/parser',
    ecmaVersion: 2020,
    sourceType: 'module',
    jsxPragma: 'React',
    ecmaFeatures: {
      jsx: true,
    },
  },
  extends: [
    "plugin:vue/vue3-recommended",
    'plugin:@typescript-eslint/recommended',
    "@vue/typescript/recommended",
    'prettier',
    'plugin:prettier/recommended',
  ],
  rules: {
    "prettier/prettier": "error",
    'vue/script-setup-uses-vars': 'error',
    '@typescript-eslint/ban-ts-ignore': 'off',
    '@typescript-eslint/explicit-function-return-type': 'off',
    '@typescript-eslint/no-explicit-any': 'off',
    '@typescript-eslint/no-var-requires': 'off',
    '@typescript-eslint/no-empty-function': 'off',
    'vue/custom-event-name-casing': 'off',
    'no-use-before-define': 'off',
    '@typescript-eslint/no-use-before-define': 'off',
    '@typescript-eslint/ban-ts-comment': 'off',
    '@typescript-eslint/ban-types': 'off',
    '@typescript-eslint/no-non-null-assertion': 'off',
    '@typescript-eslint/explicit-module-boundary-types': 'off',
    '@typescript-eslint/no-unused-vars': [
      'error',
      {
        argsIgnorePattern: '^_',
        varsIgnorePattern: '^_',
      },
    ],
    'no-unused-vars': [
      'error',
      {
        argsIgnorePattern: '^_',
        varsIgnorePattern: '^_',
      },
    ],
    'space-before-function-paren': 'off',

    'vue/attributes-order': 'off',
    'vue/one-component-per-file': 'off',
    'vue/html-closing-bracket-newline': 'off',
    'vue/max-attributes-per-line': 'off',
    'vue/multiline-html-element-content-newline': 'off',
    'vue/singleline-html-element-content-newline': 'off',
    'vue/attribute-hyphenation': 'off',
    'vue/require-default-prop': 'off',
    'vue/html-self-closing': [
      'error',
      {
        html: {
          void: 'always',
          normal: 'never',
          component: 'always',
        },
        svg: 'always',
        math: 'always',
      },
    ],
  },
}
```

> prettier 配置: https://prettier.io/docs/en/options.html

```js
// .prettierrc.js
module.exports = {
  printWidth: 100,
  tabWidth: 2, // tab 的空格数
  useTabs: false, // 用 tab 缩进而不是空格
  semi: false, // 为语句末尾加上封号
  singleQuote: true, // 使用单引号代替双引号
  quoteProps: 'as-needed', // 在需要时为
  jsxSingleQuote: false, // 使用单引号代替双引号
  trailingComma: 'all', // 在多行逗号分隔的句法结构中尽可能打印尾随逗号
  bracketSpacing: true, // 在对象文字中的括号之间添加空格。
  arrowParens: 'always', // 用圆括号包裹箭头函数的参数
  insertPragma: false, // 在文件顶部插入一个特殊的 @format 标记
  requirePragma: false, // 仅对文档顶部标注 @prettier 注释的文档有效
  proseWrap: 'never',
  htmlWhitespaceSensitivity: 'strict', // html 标签周围的空格是重要的
  vueIndentScriptAndStyle: true, // 是否缩进 Vue 这俩标签内的代码
  endOfLine: 'lf', // 回车代表的制表符 LF(/n) CRLF(/r/n)
  embeddedLanguageFormatting: "auto", // 格式化代码里包括的其他语言
}
```

## vue-types

> 介绍: https://dwightjack.github.io/vue-types/

VueTypes 是 Vue.js 的可配置 prop 验证器的集合

```bash
npm install vue-types --save
```

使用如下：

```js
export default {
  props: {
    id: {
      type: Number,
      default: 10,
    },
    name: {
      type: String,
      required: true,
    },
    age: {
      type: Number,
      validator(value) {
        return Number.isInteger(value)
      },
      default: 0,
    },
    nationality: String,
    status: {
      type: Boolean,
      default: true
    }
  },
}
```

使用 vue-types 后：

```ts
import { number, string, integer, bool } from 'vue-types'

export default {
  props: {
    id: number().def(10),
    name: string().isRequired,
    age: integer().def(0),
    nationality: string(),
    status: bool().def(true),
  }
}
```

> 详细的 API 用法: https://dwightjack.github.io/vue-types/guide/validators.html

此外也可以加入 TS 检查：

```ts
interface User {
  ID: number
  username: string
}

props: {
  user: object<User>()
}
```

```ts
enum Fruits {
  Apple = 'Apple',
  Pear = 'Pear',
}

props: {
  // string enum
  users: string<Fruits>(),
  // union type
  fruits: string<'apple' | 'pear'>()
}
```

此外还有一些特殊的操作符：


```ts
// oneOf
props: {
  status: oneOf(['open', 'close']).def('open'),
}

// instanceOf
class User {}

props: {
  user: instanceOf(User)
}

// oneOfType 验证一个属性是多种类型之一
props: {
  // Either a string, an integer or an instance of the User class
  theProp: oneOfType([String, integer(), instanceOf(User)])
  // same as above
  theProp: oneOfType([String, object<User>()])
}

// arrayOf
props: {
  theProp: arrayOf(String),
  userList: arrayOf(object()),
  // 两者都包含
  collection: arrayOf(oneOfType([array<string>(), object<User>()]))
}

// shape 验证属性是具有特定 shape 的对象
props: {
  // default value = {name: 'John'}
  // accepts: {name: 'John', age: 30, id: 1}
  userData: shape<User>({
    name: String,
    age: integer(),
    id: integer().isRequired,
  }).def(() => ({ name: 'John' }))
}

// custom 接收自定义验证函数
function minLength(value) {
  return typeof value === 'string' && value.length >= 6
}

props: {
  // theProp is a string
  theProp: custom<string>(minLength)
}
```

# 项目结构


