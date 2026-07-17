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
+ 最里边的`eager:true表示同步加载，因为是基础的组件，要一开始就加载在内存里