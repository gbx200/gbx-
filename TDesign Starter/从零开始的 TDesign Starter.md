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

#### 1.配置Vite代理
在vite.config.ts，关闭 Mock 并配置代理，让前端的 /api 请求能转发到你的 Django 后端：
![500](assets/从零开始的%20TDesign%20Starter/file-20260720153605109.png)

#### 2.修改src/utils/request/index.ts
如果使用simplejwt
#### 创建API接口文件（src/api）
