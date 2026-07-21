|序号|功能描述|HTTP 方法|请求 URL|认证|请求参数 / Body（JSON 或 FormData）|简要说明|
|---|---|---|---|---|---|---|
|1|登录获取 token|POST|`/api/token/`|否|`{"username":"test","password":"123456"}`|返回 `access` 和 `refresh` 两个 token|
|2|刷新 token|POST|`/api/token/refresh/`|否|`{"refresh":"..."}`|返回新的 `access` token|
|3|注册新用户|POST|`/api/register/`|否|`{"username":"test2","password":"123456","email":"test@test.com"}`|自动创建 Profile，返回用户信息和 JWT|
|4|查看个人资料|GET|`/api/profile/`|是|无|返回当前登录用户的 Profile 信息|
|5|修改个人资料|PUT|`/api/profile/`|是|`{"bio":"New bio","blog_title":"My Blog"}`（也可上传 avatar）|avatar 需要 multipart/form-data|
|6|修改密码|POST|`/api/change-password/`|是|`{"old_password":"123456","new_password":"654321"}`|旧密码正确才可修改|
|7|获取文章列表|GET|`/api/articles/`|否|可选参数：`?search=标题&ordering=-created_at&page=1`|返回文章列表（不含正文），支持分页（若开启）|
|8|获取文章详情|GET|`/api/articles/1/`|否|无|返回完整文章，包含 content 和 attachments|
|9|创建文章|POST|`/api/articles/`|是|`{"title":"文章标题","content":"正文","category_id":1,"tag_ids":[1,2]}`（也可上传 cover 图片）|cover 用 multipart/form-data；tag_ids 是数组|
|10|更新文章|PUT|`/api/articles/1/`|是（作者）|与创建相同|全量更新；PATCH 可局部更新|
|11|删除文章|DELETE|`/api/articles/1/`|是（作者）|无|自动删除关联文件和附件|
|12|分类列表|GET|`/api/categories/`|否|无|返回分类列表，含文章数|
|13|创建分类|POST|`/api/categories/`|是（管理员）|`{"name":"技术"}`|slug 自动生成|
|14|分类详情/更新/删除|GET/PUT/DELETE|`/api/categories/技术/`|视具体方法|修改时传 `{"name":"新名字"}`|使用 slug 作为标识（`lookup_field='slug'`）|
|15|标签列表|GET|`/api/tags/`|否|无|返回标签列表|
|16|创建标签|POST|`/api/tags/`|是（管理员）|`{"name":"Django"}`|slug 自动生成|
|17|标签详情/更新/删除|GET/PUT/DELETE|`/api/tags/1/`|视具体方法|修改时传 `{"name":"新标签"}`|使用 id 作为标识|
|18|评论列表|GET|`/api/comments/?article=1`|否|可选参数 `?article=文章ID`|非管理员只能看到已审核的评论|
|19|创建评论|POST|`/api/comments/`|是|`{"article":1,"content":"评论内容","parent_id":null}`|`parent_id` 为回复时填写父评论 ID|
|20|更新评论（审核）|PATCH|`/api/comments/1/`|是（管理员）|`{"is_approved":true}`|管理员可审核/置顶等|
|21|删除评论|DELETE|`/api/comments/1/`|是（作者或管理员）|无|作者或管理员可删|
|22|上传文件|POST|`/api/file-upload/`|是|FormData：`file`（文件）、`article_id`（文章ID）|返回文件的访问 URL 和附件 ID|
|23|仪表盘数据|GET|`/api/dashboard/`|是|无|返回当前用户总文章数、总阅读量、最新评论|