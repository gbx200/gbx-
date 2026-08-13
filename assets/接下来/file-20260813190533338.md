## 🤖 Assistant

# 弘毅书院综合 API 接口文档

> 本文档整合 **管理后台（PC端）** 与 **微信小程序端** 的全部 RESTful 接口，采用统一约定与数据模型，并明确两个端在业务场景中的交互方式。
> 适用读者：前后端开发、测试、维护人员。

---

## 目录

- [1. 项目背景](#1-项目背景)
- [2. 通用约定](#2-通用约定)
- [3. 认证与账号](#3-认证与账号)
- [4. 功能分配](#4-功能分配)
- [5. 用户管理](#5-用户管理)
- [6. 活动管理](#6-活动管理)
- [7. 场地预约](#7-场地预约)
- [8. 广场社区](#8-广场社区)
- [9. 心灵树洞](#9-心灵树洞)
- [10. 领导信箱](#10-领导信箱)
- [11. 新闻公告](#11-新闻公告)
- [12. 党建引领](#12-党建引领)
- [13. 查寝管理](#13-查寝管理)
- [14. 五育分](#14-五育分)
- [15. 消息通知](#15-消息通知)
- [16. 文件上传](#16-文件上传)
- [17. 数据模型与字段映射](#17-数据模型与字段映射)
- [18. 典型业务流程](#18-典型业务流程)
- [19. Mock 到真实后端迁移指引](#19-mock-到真实后端迁移指引)

---

## 1. 项目背景

弘毅书院信息化系统由 **管理后台（PC端）** 与 **微信小程序端** 组成：

- **管理后台**：面向书院管理人员，提供数据管理、审核、功能分配等高级操作。
- **小程序端**：面向学生/教师，提供活动报名、场地预约、社交互动、个人服务等日常功能。

两个端共享同一后端服务，但接口暴露的粒度与权限不同。本文档统一描述所有接口，并特别标注适用端及调用权限。

---

## 2. 通用约定

### Base URL
```
https://api.hongyi.edu.cn/api
```

### 鉴权方式
- **后台**：登录接口 `POST /auth/login` 获取 `token`，请求头携带 `Authorization: Bearer <token>`。
- **小程序**：登录接口 `POST /wx/login` 获取 `token`，后续请求携带 `Authorization: Bearer <token>`。
- 除登录/配置接口外，所有接口均需鉴权；`401` 表示 token 失效。

### 统一响应结构
```json
{
  "code": 0,           // 0成功，非0失败
  "message": "ok",
  "data": { }
}
```

### 错误码
| code | 含义 |
|------|------|
| 0 | 成功 |
| 400 | 参数错误 / 校验不通过 |
| 401 | 未登录或 token 失效 |
| 403 | 无权限（未分配该功能） |
| 404 | 资源不存在 |
| 409 | 冲突（重复报名 / 时段已满 / 重复点赞） |
| 500 | 服务端错误 |

### 分页约定
列表接口统一使用 `page` / `pageSize` 参数，响应 `data` 格式为：
```json
{ "list": [ ... ], "total": 0 }
```

### 时间格式
- 时间戳：`"YYYY-MM-DD HH:mm"`
- 日期字段：`"YYYY-MM-DD"`

### 功能ID（ALL_FEATURES）
用于权限控制和功能分配：
`dorm`(查寝)、`dormWorkbench`(查寝工作台·教师)、`room`(场地预约)、`activity`(活动中心)、`score`(五育分)、`treehole`(心灵树洞)、`mailbox`(领导信箱)、`news`(新闻公告)、`square`(校园广场)、`party`(党建引领)、`message`(消息通知)。

---

## 3. 认证与账号

### 3.1 管理员登录（后台）
`POST /auth/login`

| 参数 | 类型 | 必填 | 说明 |
|------|------|------|------|
| account | string | 是 | 管理账号 |
| password | string | 是 | 密码 |

响应 `data`：
```json
{ "token": "main_token", "name": "弘毅管理员", "roles": ["all"] }
```

### 3.2 获取当前用户信息（后台）
`GET /auth/profile`

响应 `data`：
```json
{ "name": "弘毅管理员", "roles": ["all"] }
```

### 3.3 退出登录（后台）
`POST /auth/logout`

### 3.4 微信登录（小程序）
`POST /wx/login`

| 参数 | 类型 | 必填 | 说明 |
|------|------|------|------|
| code | string | 是 | wx.login 得到的临时凭证 |
| profile | object | 否 | 微信头像昵称（首次登录） |

响应 `data`：
```json
{
  "token": "wx_token",
  "isNewUser": false,
  "user": {
    "id": "u1",
    "name": "李小毅",
    "role": "student",
    "studentId": "202318450001",
    "avatar": "/images/avatars/a1.png",
    "college": "电子信息学院",
    "phone": "138****0001"
  }
}
```

### 3.5 获取/更新个人资料（小程序）
- `GET /user/profile`：返回 `user` 对象（同上）。
- `PUT /user/profile`：更新 `name`、`avatar`、`college`、`phone`。

### 3.6 获取我的功能分配（小程序）
`GET /user/access`

响应 `data`：
```json
{ "features": ["dorm", "room", "activity", "score", "treehole", "mailbox", "news", "square", "message"] }
```
> 作用：控制小程序端首页/服务宫格显示模块。

---

## 4. 功能分配

> 后台管理员为用户分配可访问的功能模块，小程序端通过 `GET /user/access` 获取。

### 4.1 获取分配规则列表（后台）
`GET /access/rules?keyword=&role=&page=&pageSize=`

响应 `data`：`AccessRule[]`
```json
{
  "id": "r1",
  "account": "202318450001",
  "role": "student",
  "name": "李小毅",
  "features": ["dorm", "room", "activity", "score", "treehole", "mailbox", "news", "square", "message"]
}
```

### 4.2 更新单个账号分配（后台）
`PUT /access/rules/:id`  
请求体：`{ "features": ["dorm", "room", ...] }`

### 4.3 一键分配（按角色批量覆盖）（后台）
`POST /access/assign-by-role`  
请求体：`{ "role": "student", "features": [...] }`

### 4.4 获取角色默认功能集（后台）
`GET /access/default-features?role=student|teacher`  
响应 `data`：`{ "features": [...] }`

---

## 5. 用户管理

> 后台管理员管理所有注册用户，小程序端仅能查看/编辑自身资料。

### 5.1 用户列表（后台）
`GET /users?keyword=&role=&enabled=&page=&pageSize=`

响应 `data`：`UserWithFeature[]`
```json
{
  "account": "202318450001",
  "name": "李小毅",
  "role": "student",
  "college": "电子信息学院",
  "phone": "138****0001",
  "registeredAt": "2026-08-01 09:12",
  "enabled": true,
  "featureCount": 9,
  "features": ["dorm", "room", ...]
}
```

### 5.2 更新用户信息（后台）
`PUT /users/:account`  
请求体：`{ "name": "", "college": "", "phone": "", "enabled": true }`

### 5.3 启用/停用用户（后台）
`PATCH /users/:account/status`  
请求体：`{ "enabled": true }`

---

## 6. 活动管理

> 后台：完整增删改查 + 发布；小程序：浏览 + 报名 + 待办。

### 6.1 活动分类
- **后台**：`GET /activity/categories`、`POST /activity/categories`、`PUT /activity/categories/:id`、`DELETE /activity/categories/:id`
- **小程序**：`GET /activity/categories`（只读）

### 6.2 活动列表
- **后台**：`GET /activities?keyword=&category=&status=draft|published|closed&page=&pageSize=`
- **小程序**：`GET /activities?category=&keyword=&page=&pageSize=`（仅展示 `published` 状态）

响应 `data`（后台格式）：
```json
{
  "id": "a1",
  "title": "暑期阅读分享会",
  "category": "学术",
  "cover": "g2",
  "startAt": "2026-08-10 19:00",
  "endAt": "2026-08-10 21:00",
  "location": "博雅楼 101",
  "organizer": "活动部",
  "enrolled": 32,
  "capacity": 60,
  "status": "published",
  "scoreConfig": { "de": 0, "zhi": 1, "ti": 0, "mei": 1, "lao": 0 }
}
```
> **小程序端字段映射**：`startAt→startTime`, `endAt→endTime`, `enrolled→signupCount`，后端需转换。

### 6.3 活动详情（小程序）
`GET /activities/:id`  
响应 `data`：活动对象 + `{ "isEnrolled": false, "isTodo": false }`

### 6.4 新增活动（后台）
`POST /activities`  
请求体：ActivityItem（不含 `id`、`status` 默认 `draft`、`enrolled` 默认 0）

### 6.5 编辑活动（后台）
`PUT /activities/:id`

### 6.6 删除活动（后台）
`DELETE /activities/:id`

### 6.7 发布/结束/转草稿（后台）
`PATCH /activities/:id/status`  
请求体：`{ "status": "published" }`  

**发布副作用**：若配置了 `scoreConfig`，服务端自动为学生发放五育分（幂等）。

### 6.8 查询活动五育分发放状态（后台）
`GET /activities/:id/score-issued`

### 6.9 报名活动（小程序）
`POST /activities/:id/enroll`  
响应：`{ "enrolled": 33, "scorePromise": { "zhi": 1, "mei": 1 } }`  
错误码：`409` 重复报名、`400` 容量满。

### 6.10 取消报名（小程序）
`DELETE /activities/:id/enroll`

### 6.11 加入/移除待办（小程序）
`POST /activities/:id/todo`  
请求体：`{ "add": true }`

---

## 7. 场地预约

> 后台：管理场地资源 + 审核预约单；小程序：查看场地、选座、创建/取消预约。

### 7.1 场地资源管理（后台）
`GET /venue/resources?keyword=&type=study|meeting&page=&pageSize=`  
`POST /venue/resources`、`PUT /venue/resources/:id`、`DELETE /venue/resources/:id`

### 7.2 场地列表（小程序）
`GET /rooms?type=study|meeting`  
响应 `data`（小程序格式）：
```json
{
  "id": "r1",
  "name": "博雅楼 101 自习室",
  "type": "study",
  "capacity": 40,
  "location": "博雅楼 1F",
  "openHours": "07:00-22:00",
  "equipment": "空调、插座"
}
```

### 7.3 场地时段占用（小程序）
`GET /rooms/:id/slots?date=2026-08-05`

### 7.4 自习室座位图（小程序）
`GET /rooms/:id/layout?date=&time=`  
返回 `{ rows, cols, seats: [{ id, row, col, status: "free|booked|disabled", bookedBy }] }`

### 7.5 创建预约（小程序）
`POST /bookings`  
请求体：
```json
{
  "roomId": "r1",
  "date": "2026-08-05",
  "timeSlot": "19:00-21:00",
  "seatIds": ["s3", "s4"],
  "purpose": "自习"
}
```
响应：BookingItem；`409` 时段/座位冲突。

### 7.6 我的预约（小程序）
`GET /bookings/mine?status=all|pending|approved|rejected|cancelled`

### 7.7 取消预约（小程序）
`DELETE /bookings/:id`（仅 pending 状态）

### 7.8 预约单列表（后台）
`GET /venue/bookings?keyword=&status=pending|approved|rejected|cancelled`

### 7.9 审核预约（后台）
`PATCH /venue/bookings/:id/status`  
请求体：`{ "status": "approved" | "rejected" }`

### 7.10 删除预约单（后台）
`DELETE /venue/bookings/:id`

---

## 8. 广场社区

> 后台：审核、置顶帖子；小程序：浏览、发帖、点赞、评论、我的文章。

### 8.1 广场分类
- **后台**：`GET /square/categories`、`POST`, `PUT`, `DELETE`
- **小程序**：仅通过配置接口获取（见 §3.4）

### 8.2 帖子列表
- **后台**：`GET /square/posts?keyword=&category=&status=published|unpublished&page=&pageSize=`
- **小程序**：`GET /square/posts?category=&page=&pageSize=`（只展示 `published: true`）

响应 `data`（后台）：
```json
{
  "id": "p1",
  "title": "书院自习室新开放三间！",
  "category": "生活",
  "cover": "g4",
  "content": "<p>...</p>",
  "author": "李小毅",
  "authorRole": "student",
  "createdAt": "2026-08-03 10:00",
  "top": true,
  "published": true,
  "likes": 56,
  "comments": 12
}
```
小程序字段映射：`author→authorName`、`createdAt→publishTime`、`likes→likeCount`、`comments→commentCount`，并增加 `isLiked`、`isMine`。

### 8.3 发布帖子
- **后台**：`POST /square/posts`（`published` 默认 `false`）
- **小程序**：`POST /square/posts`，请求体 `{ "topic": "心情", "content": "...", "images": [] }`，同样待审核。

### 8.4 编辑/删除帖子
- **后台**：`PUT /square/posts/:id`、`DELETE /square/posts/:id`
- **小程序**：`PUT /square/posts/:id`、`DELETE /square/posts/:id`（仅自己的帖子）

### 8.5 上架/下架（后台审核）
`PATCH /square/posts/:id/publish`  
请求体：`{ "published": true }`

### 8.6 置顶/取消置顶（后台）
`PATCH /square/posts/:id/top`  
请求体：`{ "top": true }`

### 8.7 点赞/取消点赞（小程序）
`POST /square/posts/:id/like`  
请求体：`{ "liked": true }`

### 8.8 动态详情（小程序）
`GET /square/posts/:id`  
返回帖子详情 + 评论列表

### 8.9 评论相关（小程序）
- `GET /square/posts/:id/comments`
- `POST /square/comments`

### 8.10 我的文章（小程序）
`GET /square/mine`

### 8.11 广场消息（小程序互动通知）
`GET /square/notices`  
返回：`{ "id": "sn1", "type": "reply|like|removed", "content": "...", "time": "...", "read": false }`

---

## 9. 心灵树洞

> 仅小程序端提供，后台在 `todos` 中汇总处理（类型为 treehole）。

### 9.1 树洞列表（小程序）
`GET /treehole/posts?mood=&page=&pageSize=`

响应 `data`：
```json
{
  "id": "t1",
  "mood": "calm",
  "content": "...",
  "isAnonymous": true,
  "authorName": "匿名",
  "likeCount": 5,
  "isLiked": false,
  "publishTime": "2026-08-02 22:00"
}
```

### 9.2 发布心事（小程序）
`POST /treehole/posts`  
请求体：`{ "mood": "calm", "content": "...", "isAnonymous": true }`

### 9.3 点赞（小程序）
`POST /treehole/posts/:id/like`

---

## 10. 领导信箱

> 小程序写信，后台通过事务表（todos）回复处理。

### 10.1 收信列表（小程序）
`GET /mailbox/letters?status=all|pending|replied`

### 10.2 写信（小程序）
`POST /mailbox/letters`  
请求体：`{ "title": "...", "content": "...", "anonymous": false }`

### 10.3 信件详情（小程序）
`GET /mailbox/letters/:id`

### 10.4 待办事务列表（后台）
`GET /todos?type=treehole|mailbox|report|other&status=pending|done&keyword=&page=&pageSize=`  
响应 `data`：
```json
{
  "id": "t1",
  "type": "mailbox",
  "title": "关于宿舍热水供应时间的建议",
  "sender": "李小毅（202318450001）",
  "content": "...",
  "submittedAt": "2026-08-02 20:15",
  "status": "pending",
  "reply": "已回复：...",
  "repliedAt": "2026-08-01 17:05"
}
```

### 10.5 处理事务（后台）
`PUT /todos/:id`  
请求体：`{ "status": "done", "reply": "回复内容" }`

### 10.6 删除事务（后台）
`DELETE /todos/:id`

### 10.7 工作台待办统计（后台）
`GET /todos/pending-count`

---

## 11. 新闻公告

> 后台 CRUD + 发布/置顶；小程序仅查看。

### 11.1 新闻列表
- **后台**：`GET /news?keyword=&category=&page=&pageSize=`（含草稿、全部）
- **小程序**：`GET /news?category=&keyword=&page=&pageSize=`（仅 `published: true`）

响应 `data`（后台）：
```json
{
  "id": "n1",
  "title": "关于2026年秋季学期注册的通知",
  "category": "通知公告",
  "cover": "g1",
  "summary": "...",
  "content": "<p>...</p>",
  "author": "教务处",
  "createdAt": "2026-08-01 09:00",
  "top": true,
  "published": true
}
```
小程序字段映射：`createdAt→publishTime`，增加 `views`。

### 11.2 新增/编辑/删除（后台）
`POST /news`、`PUT /news/:id`、`DELETE /news/:id`

### 11.3 发布/下架（后台）
`PATCH /news/:id/publish`，请求体：`{ "published": true }`

### 11.4 置顶/取消置顶（后台）
`PATCH /news/:id/top`，请求体：`{ "top": true }`

### 11.5 新闻详情（小程序）
`GET /news/:id`

---

## 12. 党建引领

> 仅小程序端提供列表与详情（数据由后台维护，但文档未专门列后台接口，可能复用新闻或独立 CRUD，此处仅列小程序端）。

### 12.1 党建列表（小程序）
`GET /party/posts?category=&page=&pageSize=`

响应 `data`：
```json
{
  "id": "p1",
  "category": "主题党课",
  "cover": "/images/covers/g6.png",
  "title": "...",
  "summary": "...",
  "publishTime": "2026-08-01 10:00"
}
```

### 12.2 党建详情（小程序）
`GET /party/posts/:id`

---

## 13. 查寝管理

> 学生打卡（小程序），教师登记（小程序教师端），后台不直接操作（可通过 `dormWorkbench` 功能管理）。

### 13.1 今日查寝任务（学生）
`GET /dorm/tasks/today`

```json
{ "id": "d1", "date": "2026-08-05", "timeRange": "21:00-22:30", "punched": false, "punchTime": "" }
```

### 13.2 归寝打卡（学生）
`POST /dorm/tasks/:id/punch`  
请求体：`{ "photo": "/images/xxx.jpg" }`  
错误码：`409` 非时段。

### 13.3 查寝登记（教师）
- `GET /dorm/list?date=`
- `POST /dorm/records`  
请求体：
```json
{
  "roomNo": "512",
  "inCount": 3,
  "lateCount": 1,
  "leaveCount": 0,
  "absentCount": 0,
  "hygiene": 4,
  "photo": ""
}
```

### 13.4 查寝记录（教师）
`GET /dorm/records?date=`

---

## 14. 五育分

> 后台：管理记录、处理申诉；小程序：查看自己的分数、提交申诉。

### 14.1 五育分明细列表（后台）
`GET /score/records?keyword=&dimension=de|zhi|ti|mei|lao&source=auto|manual|appeal&activity=&page=&pageSize=`

```json
{
  "id": "sc1",
  "account": "202318450001",
  "name": "李小毅",
  "dimension": "de",
  "points": 2,
  "reason": "参加《社区志愿服务日》自动发放",
  "source": "auto",
  "activity": "社区志愿服务日",
  "createdAt": "2026-08-02 10:00"
}
```

### 14.2 手动补录（后台）
`POST /score/records`  
请求体包含：`account, name, activity, dimension, points, reason`，`source` 自动 `manual`。

### 14.3 五育分申诉列表（后台）
`GET /score/appeals?status=pending|approved|rejected`

### 14.4 通过/驳回申诉（后台）
- `POST /score/appeals/:id/approve`（自动补录分数）
- `POST /score/appeals/:id/reject`

### 14.5 我的五育分汇总（小程序）
`GET /score/summary`

```json
{
  "total": 86,
  "dimensions": [
    { "key": "de", "name": "德育", "score": 18, "color": "#C97777" },
    { "key": "zhi", "name": "智育", "score": 25, "color": "#4A6FA5" }
  ]
}
```

### 14.6 我的五育分明细（小程序）
`GET /score/records/mine?dimension=&page=&pageSize=`

### 14.7 提交申诉（小程序）
`POST /score/appeals`  
请求体：`{ "recordId": "sc1", "reason": "..." }`

---

## 15. 消息通知

> 仅小程序端使用，后台自动生成。

### 15.1 消息列表（小程序）
`GET /messages?page=&pageSize=`  
`type`：`system|activity|party|score|notice`

### 15.2 消息详情（小程序）
`GET /messages/:id`

### 15.3 全部已读（小程序）
`POST /messages/read-all`

---

## 16. 文件上传

### 16.1 上传图片
`POST /upload/image`  
请求体：`multipart/form-data`，字段 `file`（≤2MB，jpg/png）  
响应：`{ "url": "https://cdn.hongyi.edu.cn/xxxx.jpg" }`

**用途**：后台封面、富文本插图；小程序打卡照片、头像、发帖配图等。

---

## 17. 数据模型与字段映射

以下为两个端常用实体的字段名称对照，后端需在接口边界做转换（或使用 DTO）。

| 实体 | 后台字段 | 小程序字段 |
|------|----------|------------|
| 活动 | `startAt` / `endAt` | `startTime` / `endTime` |
| 活动 | `enrolled` | `signupCount` |
| 活动 | `status` (`draft/published/closed`) | `status` (`published`, `going`?) |
| 帖子 | `author` | `authorName` |
| 帖子 | `createdAt` | `publishTime` |
| 帖子 | `likes` / `comments` | `likeCount` / `commentCount` |
| 预约 | `venueId` / `venueName` | `roomId` / `roomName` |
| 新闻 | `createdAt` | `publishTime` |
| 评论 | – | `children`（嵌套） |

> **说明**：后端接口应针对不同端返回对应字段名，或提供字段映射层，保证前端零改动沿用 mock 数据结构。

---

## 18. 典型业务流程

### 18.1 用户报名活动
1. 小程序调用 `POST /wx/login` 获取 token。
2. 用户进入活动列表：`GET /activities`。
3. 点击报名：`POST /activities/:id/enroll`。
4. 后端校验（报重、容量）后返回成功，并可能更新 `scorePromise`。
5. 后台管理员查看活动：`GET /activities?status=published`。
6. 若活动即将结束，管理员将状态改为 `closed`（`PATCH /activities/:id/status`）。

### 18.2 帖子审核流程
1. 小程序用户发布帖子：`POST /square/posts`，默认 `published: false`。
2. 后台管理员查看待审核：`GET /square/posts?status=unpublished`。
3. 管理员点击上架：`PATCH /square/posts/:id/publish`。
4. 小程序端刷新列表后即可看到该帖子（`GET /square/posts` 只返回 `published: true`）。

### 18.3 场地预约审核
1. 小程序用户提交预约：`POST /bookings`。
2. 后台管理员查看待审核预约：`GET /venue/bookings?status=pending`。
3. 管理员通过：`PATCH /venue/bookings/:id/status`（`approved`）。
4. 小程序端用户查询“我的预约”可见状态变更。

### 18.4 查寝打卡
1. 学生进入小程序查寝任务：`GET /dorm/tasks/today`。
2. 上传照片并打卡：`POST /dorm/tasks/:id/punch`。
3. 教师端输入查寝结果：`POST /dorm/records`。

### 18.5 五育分申诉
1. 小程序用户看到分数异常：`GET /score/records/mine`。
2. 提交申诉：`POST /score/appeals`。
3. 后台管理员查看申诉：`GET /score/appeals?status=pending`。
4. 通过申诉：`POST /score/appeals/:id/approve`（自动补分）。

---

## 19. Mock 到真实后端迁移指引

### 小程序端
- 当前 mock 数据存于 `wx.storage`，通过 `utils/mock.ts` 的 `getList/saveList` 读写。
- **替换方式**：将 `getList/saveList` 封装为 HTTP 请求函数即可，页面代码不变。
- 接口映射参照文档每节的“mock key”。

### 后台端
- 当前 mock 使用 `localStorage` + `getXxx()/saveXxx()`。
- **替换方式**：将 `getXxx/saveXxx` 内部替换为 fetch/axios 调用对应 REST API。

### 代码隔离
建议在 `utils/` 中定义 `api.ts`（或 `request.ts`）统一管理 HTTP 请求，将现有 mock 方法改为调用 `api` 模块的异步函数，保持页面层无感切换。

---

## 附录：全部接口速查表

| 模块 | 方法 | 路径 | 适用端 | 说明 |
|------|------|------|--------|------|
| 认证 | POST | `/auth/login` | 后台 | 管理员登录 |
| 认证 | POST | `/wx/login` | 小程序 | 微信登录 |
| 功能 | GET | `/user/access` | 小程序 | 我的功能 |
| 功能 | GET/POST/PUT | `/access/rules` | 后台 | 分配管理 |
| 用户 | GET | `/users` | 后台 | 用户列表 |
| 活动 | GET | `/activities` | 双端 | 列表（字段映射） |
| 活动 | POST | `/activities/:id/enroll` | 小程序 | 报名 |
| 场地 | GET | `/rooms` | 小程序 | 场地列表 |
| 场地 | POST | `/bookings` | 小程序 | 预约 |
| 场地 | PATCH | `/venue/bookings/:id/status` | 后台 | 审核 |
| 广场 | POST | `/square/posts` | 双端 | 发帖（后台亦可） |
| 广场 | PATCH | `/square/posts/:id/publish` | 后台 | 上架/下架 |
| 树洞 | GET/POST | `/treehole/posts` | 小程序 | 列表/发布 |
| 信箱 | POST | `/mailbox/letters` | 小程序 | 写信 |
| 信箱 | PUT | `/todos/:id` | 后台 | 回复处理 |
| 新闻 | GET | `/news` | 双端 | 列表 |
| 党建 | GET | `/party/posts` | 小程序 | 列表 |
| 查寝 | POST | `/dorm/tasks/:id/punch` | 小程序 | 学生打卡 |
| 查寝 | POST | `/dorm/records` | 教师端 | 教师登记 |
| 五育分 | GET | `/score/summary` | 小程序 | 汇总 |
| 五育分 | POST | `/score/appeals/:id/approve` | 后台 | 申诉通过 |
| 消息 | GET | `/messages` | 小程序 | 消息列表 |
| 上传 | POST | `/upload/image` | 双端 | 图片 |

---

> 文档版本：v1.0 · 2026-08-13  
> 配套代码：弘毅书院后台 + 小程序（各 28 页面）  
> 更新跟踪：所有接口以本文档为准，字段变更需更新映射表。
