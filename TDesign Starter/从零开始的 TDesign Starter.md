## 初始化项目
1. 安装脚手架 tdesign-starter-cli： `npm i tdesign-starter-cli -g`
2. 创建项目 `td-starter init`
3. 启动项目：
	+ 安装依赖: `npm install`
	+ 运行项目: `npm run dev`

## 核心目录结构
![500](assets/从零开始的%20TDesign%20Starter/file-20260717185457141.jpg)

## 核心知识体系分模块
### 国际化处理（语言切换）
在 TDesign Starter 模板中，国际化（i18n）的实现主要依赖 vue-i18n 库
#### 在src/locales中配置了中英文切换相关的配置
##### /lang
在这个文件夹下分别写了en-US.json和zh-CN.json；在这两个文件下分别写了中文和英文的json数据，以后在.vue文件中引用的时候可以使用类似 `{{ $t('layout.header.help') }}`的方式引入
##### index.ts
1. 导入 TDesign 组件库自带的语言包：
```ts
import en_US from 'tdesign-vue-next/es/locale/en_US';
import zh_CN from 'tdesign-vue-next/es/locale/zh_CN';
```
2. 定义支持的语言列表：想要添加新语言只要在列表后加即可
```ts
export const supportedLocales = ['zh_CN', 'en_US'] as const;
export type SupportedLocale = (typeof supportedLocales)[number];
```
3. 多语言标题类型，用于路由 meta.title 等场景
	```ts
	export type LocalizedTitle = Record<SupportedLocale, string>;
	```
	这行定义相当于
	```ts
	type LocalizedTitle = { zh_CN: string; en_US: string; }
	```
	这样写把那些需要支持多语言的数据加了约束；比如在router/modules/homepage中这样定义：
	```ts
	  meta: {
      title: {
        zh_CN: '仪表盘',
        en_US: 'Dashboard',
      },
	```
	这样在进行中英文切换时能正常显示，并且这样是必须写中英文，如果不写就会报错
4. 自动收集翻译文件，提取代码并合并
```ts
// 使用 Vite 的 import.meta.glob 功能，自动导入 ./lang/ 目录下所有的 .json 文件
const langModules = import.meta.glob('./lang/*.json', { eager: true });
const messages: I18nOptions['messages'] = {};
const langList: DropdownOption[] = [];
// 遍历所有找到的语言文件
Object.entries(langModules).forEach(([path, module]) => {
  // 1. 从文件路径中提取语言代码，例如从 './lang/zh_CN.json' 中提取出 'zh_CN'
  const code = path.match(/\.\/lang\/([^.]+)\.json$/)?.[1] as SupportedLocale | undefined;
  if (!code || !supportedLocales.includes(code)) return;
  // 2. 整合语言包：将业务代码的翻译(module.default)和组件库的翻译(tdesignLocaleMap[code])合并
  messages[code] = { ...module.default, componentsLocale: tdesignLocaleMap[code] };
  // 3. 生成一个用于 UI 显示的语言列表，比如 [{ content: '简体中文', value: 'zh_CN' }, ...]
  langList.push({ content: module.default.lang as string, value: code });
});
```
5. 智能判断显示的语言：
```ts
// 获取初始语言的函数
const getInitialLocale = (): SupportedLocale => {
  // 1. 优先：查看本地存储，用户上次选了啥？
  const stored = localStorage.getItem(localeConfigKey);
  if (stored && supportedLocales.includes(stored as SupportedLocale)) {
    return stored as SupportedLocale;
  }
  // 2. 其次：查看浏览器设置，用户偏好是啥？
  const preferred = navigator.languages?.[0]?.replace(/-/g, '_');
  if (preferred && supportedLocales.includes(preferred as SupportedLocale)) {
    return preferred as SupportedLocale;
  } 
  // 3. 最后：都没找到，就默认用中文
  return 'zh_CN';
};
const initialLocale = getInitialLocale();
```
6. 创建并导出 i18n 实例：
```ts
export const i18n = createI18n({
  legacy: false,           // 使用 Vue 3 的组合式 API 模式
  locale: initialLocale,   // 使用我们刚才智能判断出的初始语言
  fallbackLocale: 'zh_CN', // 如果某个翻译找不到，就回退到中文
  messages,                // 传入我们整合好的所有语言包
  globalInjection: true,   // 允许在模板中直接使用 $t 函数
});
// 导出一些常用的工具，方便在其他地方使用
export const { t } = i18n.global; // 导出翻译函数 t
export const languageList = computed(() => langList); // 导出语言列表，用于UI渲染
export default i18n; // 导出 i18n 实例本身
```
##### useLocale.ts
这个文件负责处理语言的切换、持久化存储，以及为 TDesign 组件库提供对应的语言包。
1. 双向绑定语言状态:
	+ 读取时 (get)：直接从 vue-i18n 的全局实例中获取当前语言。
	+ 修改时 (set)：当你给 locale.value 赋值时，它会同步更新 vue-i18n 的内部状态。
```ts
const locale = computed({
  get: () => i18n.global.locale.value,
  set: (val: string) => {
    i18n.global.locale.value = val;
  },
});
```
2. 持久化存储
	`const storedLocale = useLocalStorage<SupportedLocale>(localeConfigKey, 'zh_CN');`
	这里使用了 useLocalStorage。
	+ 作用：它会在浏览器里记住用户上次选择的语言（比如 'en_US'）。
	+ 默认值：如果用户第一次来，或者缓存被清了，默认回退到 'zh_CN'。
	+ 在这个函数里，它定义了存储的能力，但在 changeLocale 中才被实际写入.
3. 切换语言
```ts
const changeLocale = (lang: string) => {
  // 1. 校验：防止传入非法语言代码
  const validLang = supportedLocales.includes(lang as SupportedLocale) ? (lang as SupportedLocale) : 'zh_CN';
  // 2. 更新 i18n 状态（页面文字会变）
  locale.value = validLang;
  // 3. 更新本地存储（刷新页面后语言不变）
  storedLocale.value = validLang;
  // 4. 业务联动：刷新通知消息
  useNotificationStore().refreshMsgData();
};
```
4. 适配 TDesign 组件库
	用法：在 App.vue 或布局文件中，通常会把这个值传给  `<t-config-provider :global-config="getComponentsLocale">` ，从而实现组件库的自动翻译。
```ts
const getComponentsLocale = computed(() => {
  return (i18n.global.getLocaleMessage(locale.value) as Record<string, any>).componentsLocale as GlobalConfigProvider;
});
```

#### 使用：
##### 在.vie文件中的使用:
在script中导入 `import { t } from '@/locales';` ,在你想写文本的地方直接使用`{{ t('pages.dashboardBase.rankList.week') }}即可`
##### 在js/ts文件中的使用：
同样在顶部导入 `import { t } from '@/locales/index';` ，在下方使用时直接用 `t('pages.dashboardBase.chart.max')` 即可
##### 做切换语言按钮：
具体在项目src/components/LanguageSwitcher.vue中
```ts
import { useLocale } from '@/locales/useLocale';
const { changeLocale } = useLocale();
const changeLang = (lang: string) => {
  changeLocale(lang); // 调用封装好的 Hook 进行切换
};
```
切换菜单自动识别语言并渲染：
```ts
  <t-dropdown trigger="click">
    <t-button theme="default" shape="square" variant="text">
      <translate-icon />
    </t-button>
    <t-dropdown-menu>
      <t-dropdown-item
        v-for="(lang, index) in languageList"
        :key="index"
        :value="lang.value"
        @click="(options) => changeLang(options.value as string)"
        >{{ lang.content }}</t-dropdown-item
      ></t-dropdown-menu
    >
  </t-dropdown>
```


### 用mock模拟后端请求
#### 配置mock文件
+ 在主文件夹下有/mock/index.ts，我们在这个文件下可以看到脚手架配置的模拟后端接口和数据；
+ 根目录下的 `mock/index.ts` 通过 `export default` 导出一个数组，每个元素是一个对象，定义了一个模拟接口：
```ts
import Mock from 'mockjs';  //可以使用mockjs生成随机数据
import type { MockMethod } from 'vite-plugin-mock';
export default [
  {
    url: '/api/get-purchase-list',
    method: 'get',
    response: () => ({
      code: 0,
      data: Mock.mock({ ... }),
    }),
  },
  // ...其他接口
] as MockMethod[];
```

+ 数据的写法：以其中一个数据举例
```ts
  {
    url: '/api/get-list',
    method: 'get',
    response: () => ({
      code: 0,
      data: {
        ...Mock.mock({
          'list|1-100': [{   // mockjs 语法，随机生成一个长度在 1 到 100 之间的数组
              'index|+1': 1,    // 生成一个从 1 开始，每次自动递增 1 的数字
              'status|1': '@natural(0, 4)',  // 从 0 到 4 这 5 个自然数中，随机挑选 1 个数字作为 status 的值
              no: 'BH00@natural(01, 100)',   //生成一个以 BH00 为固定前缀，后面拼接一个 1 到 100 之间随机自然数的字符串。
              name: '@city()办公用品采购项目',
              'paymentType|1': '@natural(0, 1)',
              'contractType|1': '@natural(0, 2)',
              updateTime: '2020-05-30 @date("HH:mm:ss")',  // 生成一个日期固定为 2020-05-30，但时间部分（时:分:秒）是随机生成的字符串。
              amount: '@natural(10, 500),000,000',
              adminName: '@cname()',   // 生成一个随机的中文姓名作为管理员的名字。
            }],
        }),
      },
    }),
  },
```
#### 在vite.config.ts中启用：
```ts
import { viteMockServe } from 'vite-plugin-mock';

    plugins: [
      vue(),
      vueJsx(),
      viteMockServe({
        mockPath: 'mock',
        enable: true,    // <--在这里启用mock
      }),
      svgLoader(),
    ],
```
#### 在.vue文件下使用
例：src/pages/list/base/index.vue
1. index.vue 在 onMounted 时调用 fetchData()
2. fetchData 调用 getList()（来自 @/api/list）
3. getList 内部发起 HTTP 请求（例如 GET /api/get-list）
4. 如果 mock 启用且代理没冲突，该请求被 mock/index.ts 中定义的 /api/get-list 拦截
5. 返回 mock 数据，组件将数据渲染到表格中
+ 在api/list中有如下写法来请求数据
	![](assets/从零开始的%20TDesign%20Starter/file-20260725083936396.png)
### pinia状态管理
 在src/store/里写了关于Pinia 状态管理的内容，Pinia 是用来管理“全局数据”的工具，比如当在一个页面修改了用户昵称，其他页面也要同步显示新昵称，这时候就需要一个“全局的数据仓库”；pinia把数据放在“仓库（Store）”里，所有组件都能访问、修改，而且是响应式的
 结构：
 ```text
 src/
  store/
    index.ts                # 创建 Pinia 实例，并导出所有仓库
    modules/
      notification.ts       # 通知消息仓库
      permission.ts        # 权限/菜单路由仓库
      setting.ts           # 主题、布局等设置仓库
      tabs-router.ts       # 多标签页（Tab）管理仓库
      user.ts              # 用户登录信息仓库
 ```
 
#### index.ts：
```ts
import { createPinia } from 'pinia';
import { createPersistedState } from 'pinia-plugin-persistedstate';

const store = createPinia();                          // 创建一个 Pinia 实例
store.use(createPersistedState());                   // 安装持久化插件（让部分仓库数据自动保存到 localStorage）

export { store };
export * from './modules/notification';              // 导出所有仓库的定义
export * from './modules/permission';
export * from './modules/setting';
export * from './modules/tabs-router';
export * from './modules/user';
export default store;
```
定义在main.ts中被引用·
## src下的各个文件
### main.ts
```ts
/* eslint-disable simple-import-sort/imports */

import { createApp } from 'vue';

import TDesign from 'tdesign-vue-next'; // 引入TDesign组件库
import App from './App.vue';  
import router from './router';     // 引入路由配置
import { store } from './store';   // 引入状态管理（pinia）
import i18n from './locales';      // 引入国际化多语言配置

import 'tdesign-vue-next/es/style/index.css';//引入TDesign样式
import '@/style/index.less';    // 引入项目自定义的全局样式
import './permission';          // 引入路由权限控制逻辑

const app = createApp(App);     // 创建应用实例

//注册全局插件 
app.use(TDesign);
app.use(store);
app.use(router);
app.use(i18n);

//挂载应用
app.mount('#app');
```

### router/

在这个文件夹下的，是我们的路由配置
![200](assets/从零开始的%20TDesign%20Starter/file-20260717191738796.png)

这种将路由文件拆分到 modules 文件夹下的做法，是模块化路由设计。它的核心目的是为了解决随着项目变大，单个 router/index.ts 文件变得过于臃肿、难以维护的问题。

##### index.ts
这个文件的作用主要是把目录modules下的各个文件写的路由自动整合到一起
```ts
import isObject from 'lodash/isObject';
import uniq from 'lodash/uniq';
import type { RouteRecordRaw } from 'vue-router';
import { createRouter, createWebHistory } from 'vue-router';
```
+ isObject 用来判断传入的值是否是对象
+ uniq 用于创建一个去重后的数组副本
+ RouteRecordRaw 后面代码中定义 defaultRouterList 等变量时，用它来确保传入的路由配置格式是合法的
+ createRouter：用来创建一个全新的路由实例。
+ createWebHistory：用来创建基于 HTML5 History 模式的路由历史记录管理器（URL中没有#号）
```ts
const env = import.meta.env.MODE || 'development';
```
+ `import.meta.env.MODE`：当前的运行模式（例如执行 npm run dev 时通常是 'development'，执行 npm run build 时通常是 'production'
+ `|| 'development'`：如果 import.meta.env.MODE 获取不到值，就会使用默认值 'development'
```ts
// 导入homepage相关固定路由
const homepageModules = import.meta.glob('./modules/**/homepage.ts', { eager: true });
// 导入modules非homepage相关固定路由
const fixedModules = import.meta.glob(['./modules/**/*.ts', '!./modules/**/homepage.ts'], { eager: true });
```
+ 最里边的`eager:true`表示同步加载，因为是基础的组件，要一开始就加载在内存里
+ `import.meta.glob`：这是 Vite 提供的专属 API。它的作用是在编译阶段自动扫描文件夹，把匹配到的文件路径和对应的模块内容，打包成一个对象。
在执行完后会给 `homepageModules` 赋到以下内容
```ts
{
	'./modules/user/homepage.ts': { default: [/* 路由配置 */],
	'./modules/system/homepage.ts': { default: [/* 路由配置 */]
}
```

```ts
// 其他固定路由
const defaultRouterList: Array<RouteRecordRaw> = [
  {
    path: '/login',
    name: 'login',
    component: () => import('@/pages/login/index.vue'),
  },
  {
    path: '/',
    redirect: '/dashboard/base',
  },
];
```
这里定义了一个名为`defaultRouterList`的数组，用来存放默认的路由规则
+ `: Array<RouteRecordRaw>` 这是 TypeScript 的类型注解。RouteRecordRaw 是 Vue Router 提供的标准类型，它强制要求数组里的每一个对象都必须符合路由配置的标准格式（比如必须有 path，必须有 component 或 redirect 等）。这能在你写代码时就提前发现拼写错误
+ 下方正常定义路由

```ts
// 存放固定路由
export const homepageRouterList: Array<RouteRecordRaw> = mapModuleRouterList(homepageModules);
export const fixedRouterList: Array<RouteRecordRaw> = mapModuleRouterList(fixedModules);
  
export const allRoutes = [...homepageRouterList, ...fixedRouterList, ...defaultRouterList];
// 固定路由模块转换为路由
export function mapModuleRouterList(modules: Record<string, unknown>): Array<RouteRecordRaw> {
  const routerList: Array<RouteRecordRaw> = [];
  Object.keys(modules).forEach((key) => {
    const routeModule = modules[key];
    if (isObject(routeModule) && 'default' in routeModule) {
      const route = routeModule.default;
      const routes = Array.isArray(route) ? [...route] : [route];
      routerList.push(...routes);
    }
  });
  return routerList;
}
```
前面用`import.meta.glob`收集到的是一个对象，这里代码是用来转化为Vue Router需要的数组，并将数据合并
+ 前两行的作用是调用转换函数并导出；将前面收集到的 `homepageModules` 和 `fixedModules` 对象，传入 `mapModuleRouterList` 函数进行清洗，转换成标准的路由数组。 并用 `export` 将其导出
+ 然后合并所有路由并赋给 `allRoutes`
+ 接下来定义一个函数 `mapModuleRouterList` 将固定路由模块转换为路由

接下来下面有一块代码被定义为 `@deprecated 未使用` 是用来处理侧边栏菜单默认展开状态的，在当前版本的菜单在router/modules/homepage.ts中有新的写法，在后续会说这里略过这段废弃的代码

```ts
export const getActive = (maxLevel = 3): string => {
  // 非组件内调用必须通过Router实例获取当前路由
  const route = router.currentRoute.value;
  if (!route.path) {
    return '';
  }
  return route.path
    .split('/')
    .filter((_item: string, index: number) => index <= maxLevel && index > 0)
    .map((item: string) => `/${item}`)
    .join('');
};
```
这个 `getActive` 函数是 TDesign Starter 中用来解决侧边栏菜单高亮联动的核心工具，有了它，在我们访问深层页面时侧边可以保持高亮

```ts
const router = createRouter({  // 创建路由实例
  history: createWebHistory(env === 'site' ? '/starter/vue-next/' :import.meta.env.VITE_BASE_URL),  // 配置历史记录模式与基础路径
  routes: allRoutes,  // 注入路由表
  scrollBehavior() {
    return {
      el: '#app', // 指定滚动的目标元素。这里表示相对于 id="app" 的 DOM 元素进行滚动定位
      top: 0, // 在目标元素的基础上，垂直方向偏移 0 像素。结合起来就是：每次切换新页面，都滚动到 #app 容器的最顶部
      behavior: 'smooth', // 开启平滑滚动动画。页面不会生硬地“瞬移”到顶部，而是有一个平滑的过渡效果
    };
  },
});
```
这段代码实现路由最终的组装和启动
+ 这其中的 `scrollBehavior` 是用来配置滚动行为的，这是 Vue Router 提供的一个钩子函数，用来控制路由切换时页面的滚动位置。如果不配置，切换页面时滚动条可能会停留在上一个页面的位置
#### /modules
在这级目录下我们配置路由，可以加.ts文件，index.ts会自动挂载
##### homepage.ts
在这个文件中，使用了 `LAYOUT` 作为了外层容器
作用：在中后台系统中，/dashboard 这种业务页面通常都带有左侧菜单、顶部导航和底部版权信息。LAYOUT 就是那个包含这些公共元素的“大框架”。
机制：它把 LAYOUT 作为父级路由的 component，而把真正的页面（如 base/index.vue）放在 children 里。这样，当你在仪表盘的不同子页面切换时，侧边栏和顶部导航不会重新渲染，只有中间的内容区会变化。

此目录下其他文件都和这个类似


### permission.ts

这个文件写在用户切换页面时执行的逻辑
1. 初始化与准备
	```ts
	NProgress.configure({ showSpinner: false }); // 关闭进度条的加载小圆圈
	router.beforeEach(async (to, from, next) => {
		NProgress.start(); // 每次路由跳转前，开启顶部进度条
		const permissionStore = getPermissionStore();
		const { whiteListRouters } = permissionStore; // 获取免登录白名单
		const userStore = useUserStore();
	```
	作用：在每次页面跳转前，先启动进度条，并从 Pinia 状态管理中获取权限配置和用户信息。
2. 已登录状态的处理
	```ts
	if (userStore.token) {
	// 2.1 如果已登录，但访问的是登录页，直接放行（避免死循环）
	  if (to.path === '/login') { next(); return; } 
	  try {
    // 2.2 每次跳转都尝试获取用户信息（通常有缓存，不会重复请求）
	    await userStore.getUserInfo(); 
	    const { asyncRoutes } = permissionStore;
    // 2.3 如果动态路由表是空的，说明是首次登录或刷新页面
	    if (asyncRoutes && asyncRoutes.length === 0) {
      // 调用 Store 中的方法，构建动态路由（从后端拉取或本地生成）
	      const routeList = await permissionStore.buildAsyncRoutes(); 
      // 核心：将生成的路由一条条动态添加到 Vue Router 中
	      routeList.forEach((item: RouteRecordRaw) => {
	        router.addRoute(item); 
	      });
      // 2.4 处理刷新页面的 404 问题
	      if (to.name === PAGE_NOT_FOUND_ROUTE.name) {
        // 如果当前匹配到了 404 页面，说明路由刚加进去，需要重新导航到目标路径
	        next({ path: to.fullPath, replace: true, query: to.query });
	      } else {
        // 正常重定向逻辑
	        const redirect = decodeURIComponent((from.query.redirect || to.path) as string);
	        next(to.path === redirect ? { ...to, replace: true } : { path: redirect, query: to.query });
	        return;
      }
    }
    // 2.5 路由已存在，直接放行；不存在则踢回首页
    if (router.hasRoute(to.name!)) { next(); } 
    else { next(`/`); }
    } catch (error) {
    // 2.6 获取用户信息失败（如 Token 过期），报错并踢回登录页
    MessagePlugin.error((error as Error).message);
    next({ path: '/login', query: { redirect: encodeURIComponent(to.fullPath) } });
    NProgress.done();
	  }
	}
	```
3. 未登录状态的处理
	```ts
	else {
	  // 3.1 如果目标路径在白名单中（如登录页、注册页），直接放行
	  if (whiteListRouters.includes(to.path)) { next(); } 
	  else {
	    // 3.2 不在白名单中，强制跳转到登录页，并记录原本想去的路径
	    next({ path: '/login', query: { redirect: encodeURIComponent(to.fullPath) } });
	  }
	  NProgress.done();
	}
	```
4. 全局后置守卫（离开页面时）
	```ts
	router.afterEach((to) => {
	  // 4.1 如果跳转到了登录页，说明用户退出了，清理状态
	  if (to.path === '/login') {
	    const userStore = useUserStore();
	    const permissionStore = getPermissionStore();
	    userStore.logout();
	    permissionStore.restoreRoutes(); // 清空动态路由表
	  }
	  NProgress.done(); // 关闭顶部进度条
	});
	```
### utils/request/Axios.ts

在axios中我们一般封装请求拦截器和响应拦截器等网络请求的核心封装，大部分情况下我们不需要改动这里的代码

### src/store/
这里写了关于Pinia 状态管理的内容
#### index.ts
```ts
const store = createPinia();
store.use(createPersistedState());
export { store };
```
这三行的作用是全局配置与持久化，引入一个 `pinia-plugin-persistedstate` 的插件使Store 中的数据自动保存到浏览器的 localStorage 或 sessionStorage 中，你侧边栏的折叠状态不会因你刷新而消失
```ts
export * from './modules/notification';
export * from './modules/permission';
export * from './modules/setting';
export * from './modules/tabs-router';
export * from './modules/user';

export default store;
```
这是模块的统一导出，以后在vue组件使用时用 `import { useUserStore, usePermissionStore } from '@/store';` 就能直接使用所有模块

#### modules/

## 修改页面

### 1.配置Vite代理
在vite.config.ts，关闭 Mock 并配置代理，让前端的 /api 请求能转发到你的 Django 后端：
![500](assets/从零开始的%20TDesign%20Starter/file-20260720153605109.png)

+ 补充：在src文件夹下有名为.env.development的文件，在开发环境下，会优先使用 .env.development 里的 VITE_API_URL 作为基础地址，需要同时把这里的地址删掉
+ ![500](assets/从零开始的%20TDesign%20Starter/file-20260721151129110.png)
### 2.修改src/utils/request/index.ts
+ 如果使用jwt认证，请求时请求头需要使用Bearer <你的token>来验证身份![500](assets/从零开始的%20TDesign%20Starter/file-20260720155701137.png)
+ Axios 响应拦截期望后端返回的格式是 { code: 0, data: ... }，但DRF返回的直接是数据体（如 { results: [], count: 10 } 或 { id: 1 }），没有 code 字段。不修正的话所有请求都会报错。原始数据如下![500](assets/从零开始的%20TDesign%20Starter/file-20260721152548870.png)
+ 修改：![500](assets/从零开始的%20TDesign%20Starter/file-20260721153256821.png)
### 3.创建API接口文件（src/api）
把原有的api清理掉，创建自己的api
1. 在api/下负责接收后端传回的内容
2. 在api/model下的文件负责限制传回数据的类型检验
### 4.修改src/store
把这个文件下的mock数据改为真正的后端数据
1. 修改/modules/下的user.ts和permission.ts
2. 修改router/下的index.ts
### 5.修改pages
#### login
在src下有permission.ts，它会检查token并自动跳转，但是复杂的跳转逻辑可能会导致报错，可以把它修改为
```ts
import 'nprogress/nprogress.css';
import NProgress from 'nprogress';
import router from '@/router';
import { useUserStore } from '@/store';
NProgress.configure({ showSpinner: false });
const whiteListRouters = ['/login'];
router.beforeEach(async (to, from, next) => {
  NProgress.start();
  const userStore = useUserStore();
  if (userStore.token) {
    if (to.path === '/login') {
      // 已登录且访问登录页 -> 跳首页
      next('/dashboard/base');
      NProgress.done();
      return;
    }
    // 已登录且访问其他页面 -> 放行
    next();
    NProgress.done();
  } else {
    // 未登录
    if (whiteListRouters.includes(to.path)) {
      // 白名单（如登录页）放行
      next();
    } else {
      // 否则重定向到登录页，带上目标路径
      next({
        path: '/login',
        query: { redirect: encodeURIComponent(to.fullPath) },
      });
    }
    NProgress.done();
  }
});
router.afterEach((to) => {
  if (to.path === '/login') {
    const userStore = useUserStore();
    userStore.logout();
  }
  NProgress.done();
});
```
- 完全去除了动态路由构建（`buildAsyncRoutes`）和复杂重定向，因为博客后台没有动态权限路由。
- 已登录时访问 `/login` 直接跳转到 `/dashboard/base`（你的首页），防止重复登录。
- 未登录时，除了 `/login` 外全部拦截到登录页，并保留“重定向回原来页面”的功能（通过 `redirect` 参数）。
- 移除了 `asyncRoutes` 判断和 `hasRoute` 检查，避免无限循环。

#### user（个人中心页面）
1. 删除constants，这是假数据，直接删除即可，若要修改其他页面同理
	![200](assets/从零开始的%20TDesign%20Starter/file-20260721195944191.png)
#### 其他页面
##### 新增页面：
+ 如果想新增一个页面，不要子页面，直接在src/pages/下新增文件夹，文件夹里写index.vue即可；
+ 如果有子页面，在pages/下新增文件夹，在此文件夹下再创建文件夹，一个文件夹对应一个子文件
**例**：
新建一个文件夹
![200](assets/从零开始的%20TDesign%20Starter/file-20260721193132130.png)
写页面
![450](assets/从零开始的%20TDesign%20Starter/file-20260721193155401.png)
这里我选择新建一个ts文件来存储新页面的路由，也可以选择写再其他路由后
![](assets/从零开始的%20TDesign%20Starter/file-20260721193451774.png)
实现效果：
![](assets/从零开始的%20TDesign%20Starter/file-20260721193808433.png)

## 遇到的问题
### 点击退出登录会跳转到主页
解决方法：在src/layout/components/Header下加上这两行代码
![500](assets/从零开始的%20TDesign%20Starter/file-20260722144616526.png)
原因：原先是先定位到/login再清除token，修改之后是先清除token再跳转。这样修改后最终写法可以是这样：
```ts
const handleLogout = () => {
  const userStore = useUserStore()
  userStore.logout();
  router.push('/login',);
};
```
### 修改首页后无法准确定位
1. 修改src/router/index.ts:修改重定向到自己写的首页
	![500](assets/从零开始的%20TDesign%20Starter/file-20260722151943346.png)
2. 修改src/permission.ts： 类似的路由全部修改
	![400](assets/从零开始的%20TDesign%20Starter/file-20260722152135134.png)
3. 修改src/layout/component下的SideNav.vue和Header.vue


+ **一键修改：** 按下 Ctrl + Shift + H；在上面的搜索框输入：/dashboard/base；在下面的替换框输入：/blog/list (你的首页)
	![300](assets/从零开始的%20TDesign%20Starter/file-20260722152956489.png)
	![300](assets/从零开始的%20TDesign%20Starter/file-20260722153256357.png)
### 在侧边栏去除结果页
如果想使用TDesign的结果页却不想让它出现在侧边栏：
![400](assets/从零开始的%20TDesign%20Starter/file-20260722202033173.png)
在箭头位置新增一行 `hidden: true,` 即可
