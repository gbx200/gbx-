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
+ `import.meta.env.MODE`：这是 Vite 构建工具提供的内置环境变量，表示当前的运行模式（例如执行 npm run dev 时通常是 'development'，执行 npm run build 时通常是 'production'）。
+ `|| 'development'`：这是一个“逻辑或”运算符（短路求值）。如果 import.meta.env.MODE 获取不到值（为 undefined 或空），就会使用默认值 'development'，起到兜底保护的作用。
