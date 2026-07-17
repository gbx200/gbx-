## 初始化项目
1. 安装脚手架 tdesign-starter-cli： `npm i tdesign-starter-cli -g`
2. 创建项目 `td-starter init`
3. 启动项目：
	+ 安装依赖: `npm install`
	+ 运行项目: `npm run dev`

## 核心目录结构
![500](assets/从零开始的%20TDesign%20Starter/file-20260717185457141.jpg)

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
+ 这其中的 `scrollBehavior` 是用来配置滚动行为的，这是 Vue Router 提供的一个钩子函数，用来控制路由切换时页面的滚动位置。如果不配置，切换页面时滚动条可能会停留在上一个页面的位置，体验很差
#### /modules
在这级目录下我们配置路由
##### homepage.ts
