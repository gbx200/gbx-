原先的：
```js
import axios from "axios";

import qs from 'qs';

// import { pa } from "element-plus/es/locale";

  
  

let baseURL = "http://localhost:8000/";

// 创建一个新的axios实例

const httpService = axios.create({

    //url前缀-'http:xxx.xxx'

    //baseURL: process.env.BASE_API , //需自定义

  baseURL: baseURL,

  timeout: 3000, // 请求超时时间

});

  

// 请求拦截器

httpService.interceptors.request.use(function (config) {

    // 在发送请求之前做些什么

    config.headers.Authorization = window.sessionStorage.getItem("token");

    return config;

}, function (error) {

    // 对请求错误做些什么

    return Promise.reject(error);

});

  

// 响应拦截器

httpService.interceptors.response.use(function (response) {

    // 对响应数据做点什么

    return response;

}, function (error) {

    // 对响应错误做些什么

    return Promise.reject(error);

});

  

export function get(url, params={}) {

    return new Promise((resolve, reject) => {

        httpService({

            method: 'get',

            url: url,

            params: params

        }).then(response => {

            resolve(response);

        }).catch(error => {

            reject(error);

        });

    });

}

  

export function post(url, params={}) {

    return new Promise((resolve, reject) => {

        httpService({

            method: 'post',

            url: url,

            data: qs.stringify(params),

            headers: {

                'Content-Type': 'application/x-www-form-urlencoded'

            }

        }).then(response => {

            console.log(response);

            resolve(response);

        }).catch(error => {

            console.log(error);

            reject(error);

        });

    });

}

  

export function del(url, params={}) {

    return new Promise((resolve, reject) => {

        httpService({

            method: 'delete',

            url: url,

            data: params

        }).then(response => {

            console.log(response);

            resolve(response);

        }).catch(error => {

            console.log(error);

            reject(error);

        }   );

    });

}

  
  

export function fileUpload(url, params={}) {

    return new Promise((resolve, reject) => {

        httpService({

            method: 'post',

            url: url,

            data: params,

            headers: {

                'Content-Type': 'multipart/form-data'

            }

        }).then(response => {

            resolve(response);

        }).catch(error => {

            reject(error);

        });

    });

}

  

export function getServerUrl() {

    return baseURL;

}

  

export default {

    get,

    post,

    del,

    fileUpload,

    getServerUrl

}
```
更改后：
这个文件结构**完全可以**作为你项目的 HTTP 请求封装基础，但为了完美接入 JWT "记住我"和无感刷新功能，你需要对现有代码做 **3 处关键修改**。

你当前的代码存在以下隐患：

1. **Token 读取写死了 `sessionStorage`**：这会导致前端勾选“记住我”（存入 `localStorage`）后，请求拦截器依然读不到 Token。
2. **响应拦截器是空的**：没有挂载上一轮讨论的无感刷新逻辑。
3. **POST 请求强制使用了 `qs.stringify`**：你的 Django DRF 后端默认接收的是 JSON (`application/json`)，如果登录接口用 `post()` 发请求，后端会报 400 参数错误。

### 🛠️ 修改后的完整代码 (直接替换原文件)

```javascript
import axios from "axios";
import qs from 'qs';
// import router from '@/router'; // 👈 建议取消注释，用于刷新失败时跳转登录页

const baseURL = "http://localhost:8000/";

const httpService = axios.create({
  baseURL: baseURL,
  timeout: 10000, // 👈 建议改为 10s，3s 在网络波动或刷新 Token 时极易超时
});

// ================= 请求拦截器 =================
httpService.interceptors.request.use(function (config) {
    // 👇 【修改1】动态兼容 localStorage 和 sessionStorage
    const token = localStorage.getItem("access_token") 
               || sessionStorage.getItem("access_token");
    
    if (token) {
        // 👇 注意：JWT 必须加 Bearer 前缀，DRF 才能正确解析
        config.headers.Authorization = `Bearer ${token}`;
    }
    return config;
}, function (error) {
    return Promise.reject(error);
});

// ================= 响应拦截器 =================
httpService.interceptors.response.use(
    response => response, 
    async error => {
        const originalRequest = error.config;
        
        // 👇 【修改2】接入无感刷新 Token 逻辑
        // 排除刷新接口本身，防止死循环
        if (error.response?.status === 401 && !originalRequest._retry && !originalRequest.url.includes('/api/token/refresh/')) {
            originalRequest._retry = true;
            
            const refreshToken = localStorage.getItem('refresh_token') 
                              || sessionStorage.getItem('refresh_token');
            
            if (refreshToken) {
                try {
                    // 👇 使用原生 axios 发起刷新请求，避免被当前拦截器再次拦截
                    const { data } = await axios.post(`${baseURL}api/token/refresh/`, {
                        refresh: refreshToken
                    });
                    
                    // 智能判断应该存回哪个 Storage
                    const storage = localStorage.getItem('refresh_token') ? localStorage : sessionStorage;
                    storage.setItem('access_token', data.access);
                    if (data.refresh) {  
                        storage.setItem('refresh_token', data.refresh);
                    }
                    
                    // 用新 Token 重发原始请求
                    originalRequest.headers.Authorization = `Bearer ${data.access}`;
                    return httpService(originalRequest); // 👈 注意这里用 httpService 重发
                } catch (refreshError) {
                    // Refresh Token 也过期 → 清除本地状态并跳转登录页
                    localStorage.removeItem('access_token');
                    localStorage.removeItem('refresh_token');
                    sessionStorage.removeItem('access_token');
                    sessionStorage.removeItem('refresh_token');
                    
                    // router.push('/login'); // 👈 取消注释以启用自动跳转
                    return Promise.reject(refreshError);
                }
            }
        }
        return Promise.reject(error);
    }
);

// ================= 请求方法封装 =================

export function get(url, params = {}) {
    return httpService.get(url, { params }); // 👈 简化写法，无需手动 new Promise
}

// 👇 【修改3】新增标准 JSON POST 方法（对接 DRF 必用）
export function postJson(url, data = {}) {
    return httpService.post(url, data); 
}

// 保留原有的表单提交 POST（仅用于特殊旧接口）
export function postForm(url, params = {}) {
    return httpService.post(url, qs.stringify(params), {
        headers: { 'Content-Type': 'application/x-www-form-urlencoded' }
    });
}

export function del(url, data = {}) {
    return httpService.delete(url, { data });
}

export function fileUpload(url, formData) {
    return httpService.post(url, formData, {
        headers: { 'Content-Type': 'multipart/form-data' }
    });
}

export function getServerUrl() {
    return baseURL;
}

export default {
    get,
    postJson,   // 👈 日常开发优先用这个
    postForm,   // 👈 仅在需要 x-www-form-urlencoded 时使用
    del,
    fileUpload,
    getServerUrl
};
```

### 💡 核心修改点与避坑说明

1. **关于 Token 键名的统一**  
    你原来的请求拦截器读的是 `"token"`，但响应拦截器存的是 `"access_token"`。**键名不一致会导致永远读不到 Token**。修改后的代码已统一为 `access_token` 和 `refresh_token`。请务必同步修改你登录成功后存储 Token 的代码。
2. **为什么必须新增 `postJson`？**  
    你原来的 `post` 方法强制设置了 `application/x-www-form-urlencoded` 并用 `qs.stringify` 序列化。而 Django REST Framework 的 `APIView` 和序列化器默认期望接收 `application/json`。如果用原 `post` 方法调用登录接口，后端 `request.data` 将拿不到正确的用户名密码。**以后对接 DRF 接口，请一律使用 `postJson`**。
3. **消除 Promise 反模式**  
    你原来的 `get/post/del` 中使用了 `new Promise((resolve, reject) => { httpService().then(resolve).catch(reject) })`。这在业界被称为 "Promise Anti-pattern"（Promise 反模式），因为 `httpService()` 本身已经返回了 Promise，再包一层完全是多余的，还会丢失错误堆栈信息。修改后的代码直接返回 `httpService.get/post`，更简洁且安全。
4. **刷新请求必须用原生 axios**  
    在响应拦截器的 `try` 块中，我特意使用了 `axios.post(...)` 而不是 `httpService.post(...)`。这是因为如果你用同一个实例去请求刷新接口，当 Refresh Token 也失效返回 401 时，又会触发拦截器去刷新，造成**无限递归死锁**。

按照这份代码替换后，你的前端就能完美配合之前修改的后端“记住我”逻辑，实现真正的无感鉴权了。