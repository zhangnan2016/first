# WanderChina MVP · 模块规格说明（Module Spec）

> **文档版本**：v1.0
> **生成日期**：2026-06-18
> **关联文档**：[inbound-tourism-market-research.md](./inbound-tourism-market-research.md) · [product-outline-mvp.md](./product-outline-mvp.md)
> **约束规范**：[database-design-conventions](../openspec/specs/database-design-conventions/spec.md) · [auth-session](../openspec/specs/auth-session/spec.md) · [api-contract](../openspec/specs/api-contract/spec.md) · [data-access-layer](../openspec/specs/data-access-layer/spec.md)
> **技术栈**：Next.js + Spring Boot + MySQL + Spring AI
> **开发周期**：3 周 / 1 人 + AI 协作

---

## 0. 关键决策记录（已确认）

> 本规格在前两轮《市场调研》《产品大纲》基础上，经冲突澄清后确认以下决策。**这些决策与产品大纲存在差异，以本规格为准。**

| # | 决策项 | 结论 | 与前两轮的差异 |
|---|--------|------|----------------|
| D1 | 模块清单 | **以本文 6 模块为准**（首页/发现、景点攻略、旅游社区、AI助手、行程规划、实用工具） | 产品大纲的「入境生存清单 / 英文本地生活榜 / 账号空间」在本期不单列为模块；其价值分别收敛进「实用工具」「首页」与「公共基础设施」 |
| D2 | 账号体系 | **作为公共基础设施**，不占模块名额，所有需身份的模块依赖它（见 §1） | 产品大纲中「⑥ 账号与个人空间」降级为基础设施 |
| D3 | 优先级 | **P0 = AI助手 + 旅游社区 + 首页/发现**（实现核心逻辑）；**P1/P2 = 景点攻略 + 行程规划 + 实用工具**（本期仅占位页） | 与产品大纲 P0 清单（入境清单/AI/账号）不同，本期以"差异化 + 数据飞轮 + 入口"为 P0 |
| D4 | AI 助手范围 | **本期只做基于 RAG 的景点/攻略问答**，不做通用聊天、不做外部 API 调用、不做拍照翻译 / 多模态 / 转真人 | 较产品大纲显著收窄（大纲含拍照翻译、上下文记忆） |

---

## 1. 公共基础设施 · 账号体系（Account Infrastructure）

> 不占模块名额。承载所有需要"用户身份"的模块（社区发帖、AI 会话记忆）的身份底座。
> **遵循现有 [auth-session](../openspec/specs/auth-session/spec.md) 规范（Sa-Token）**，不另起炉灶。

### 1.1 边界定义

**In Scope（本期）**
1. 注册 / 登录（**邮箱 + 密码**，复用现有 `sys_user` 模型，`username` 字段承载邮箱）
2. Token 签发与拦截（Sa-Token，`Authorization: Bearer <token>`）
3. 当前用户信息查询（`/auth/me`）

**Out of Scope（本期）**
- ❌ Google / OAuth 第三方登录（产品设想的方向，但本期遵循现有 username/password 体系，OAuth 延后）
- ❌ 个人空间 / 收藏聚合 / 行程草稿（原「个人空间」功能，本期不做）
- ❌ 密码找回 / 邮箱验证 / 多因素认证

### 1.2 数据模型（复用现有）

复用现有 `sys_user` 表，本期不新增字段，仅约定业务用法：

| 字段 | 类型 | 约束 | 说明 |
|------|------|------|------|
| `id` | BIGINT | PK, Snowflake | 用户 ID |
| `username` | VARCHAR(128) | NOT NULL | **本期承载邮箱**（作为登录账号） |
| `password` | VARCHAR(100) | NOT NULL | BCrypt 哈希 |
| `nickname` | VARCHAR(50) | NOT NULL DEFAULT '' | 昵称 |
| `status` | TINYINT | NOT NULL DEFAULT 1 | 0=禁用, 1=正常 |
| `create_time/update_time/create_by/update_by/deleted` | — | — | BaseEntity 公共字段 |

- 唯一约束：`UNIQUE(username, deleted)`（遵循规范，容忍逻辑删除后重注册）
- 索引：遵循 `database-design-conventions` 规范

### 1.3 鉴权与公开端点适配（⚠️ 需明确）

> **冲突暴露**：现有 auth-session 白名单仅含 `/auth/login` 与文档端点。但产品定位强调「SEO 友好、降低使用门槛」，**首页、帖子详情、攻略详情等只读内容必须支持游客访问**。

**本期需做**：扩展 Sa-Token 拦截器白名单，将以下**只读公开端点**加入白名单：

| 端点 | 说明 |
|------|------|
| `GET /api/home/**` | 首页聚合（编辑推荐 / 热门帖子 / 热门景点） |
| `GET /api/posts`, `GET /api/posts/{id}` | 帖子列表与详情（游客可浏览） |
| `GET /api/posts/{id}/comments` | 评论列表（游客可查看） |
| `GET /api/attractions/**` | 景点 / 攻略详情（游客可浏览，SEO） |

**写操作（发帖 / 评论 / AI 会话）必须登录**，遵循 `@SaCheckPermission` 注解规范。

---

## 2. 模块优先级总览

| 优先级 | 模块 | 本期形态 | 价值锚点 |
|--------|------|----------|----------|
| **P0** | ① 首页 / 发现 | 完整规格 | 入口 + 内容聚合（SEO 获客） |
| **P0** | ③ 旅游社区 | 完整规格 | 数据飞轮启动（UGC 沉淀） |
| **P0** | ④ AI 助手 | 完整规格 | 差异化护城河（North Star 承载体） |
| **P1** | ② 景点攻略 | **UI 占位 + 底层数据先行** | 为 AI RAG 提供知识源、为首页提供景点数据 |
| **P2** | ⑤ 行程规划 | 占位页 | 未来价值 |
| **P1/P2** | ⑥ 实用工具 | 占位页 | 未来承载「入境生存清单」高价值工具 |

> **跨优先级依赖说明（重要）**：景点攻略（P1）的 UI 本期占位，但其**底层数据表 + 种子数据**必须先行落地，因为它是 P0 模块 AI 助手（RAG 知识源）与首页（热门景点）的硬依赖。详见 §8。

---

## 3. 模块①：首页 / 发现（P0）

> **一句话价值**：访客落地第一眼，用「编辑推荐 + 热门内容」聚合平台核心价值，降低决策成本、承载 SEO 获客。

### 3.1 模块边界定义

**In Scope（4 个功能点）**
1. **编辑推荐位**：运营配置的 PGC 内容卡片（指向攻略 / 帖子 / 景点）
2. **热门帖子聚合**：按热度从社区拉取 TOP N 帖子
3. **热门景点聚合**：从景点攻略模块拉取 TOP N 景点
4. **模块导航入口**：通往社区 / AI 助手 / 攻略 / 工具的入口卡片

**Out of Scope**
- ❌ 个性化推荐算法（约束明确不做）
- ❌ 站内搜索
- ❌ 用户关注 feed / 时间线
- ❌ 广告 / 商业投放系统
- ❌ 运营后台（编辑推荐本期用 SQL/脚本配置，不做可视化后台）

### 3.2 核心用户故事

- **US-1.1**：As a **首次访客**，I want to 在首页看到精选的来华旅行内容，so that 快速判断平台是否值得深入使用。
- **US-1.2**：As a **准备来华的游客**，I want to 从首页一键进入社区 / AI 助手 / 攻略入口，so that 无需摸索直达核心功能。
- **US-1.3**：As a **浏览型游客**，I want to 在首页看到当下热门的帖子与景点，so that 不登录也能感知平台活跃度与内容质量。

### 3.3 验收标准

**AS-1.1 编辑推荐展示**
- **WHEN** 游客（未登录）打开首页
- **THEN** 系统返回上线状态（`status=1`）的编辑推荐列表，按 `sort_order` 升序排列
- **AND** 每条推荐展示标题、副标题、封面图、跳转目标；接口在 Sa-Token 白名单内，游客可访问

**AS-1.2 热门帖子聚合**
- **WHEN** 系统聚合首页热门帖子
- **THEN** 按热度分 `热度 = comment_count * 3 + view_count` 降序取 TOP N（默认 N=10）
- **AND** 仅取 `status=1 AND deleted=0` 的已发布帖子，分页返回 `PageResult`

**AS-1.3 热门景点聚合**
- **WHEN** 系统聚合首页热门景点
- **THEN** 取 `status=1` 景点，按预设热度（本期种子数据 `popularity` 字段）降序 TOP N
- **AND** 当景点模块无数据时返回空列表（不报错），首页该区域展示空态兜底

**AS-1.4 模块导航**
- **WHEN** 用户点击首页导航入口
- **THEN** 跳转至对应模块（P1/P2 占位模块跳转至占位页）
- **AND** P0 模块（社区 / AI）正常进入功能页

### 3.4 数据模型概要

**核心实体：`home_recommend`（编辑推荐）**

| 字段 | 类型 | 约束 | 说明 |
|------|------|------|------|
| `id` | BIGINT | PK, Snowflake | — |
| `title` | VARCHAR(200) | NOT NULL | 推荐标题 |
| `subtitle` | VARCHAR(500) | NOT NULL DEFAULT '' | 副标题 |
| `cover_image_url` | VARCHAR(500) | NOT NULL DEFAULT '' | 封面图 |
| `target_type` | TINYINT | NOT NULL | 1=攻略, 2=帖子, 3=景点, 4=外链 |
| `target_id` | BIGINT | NULL | 关联目标 ID（外链类型为 NULL） |
| `link_url` | VARCHAR(500) | NOT NULL DEFAULT '' | 外链地址（target_type=4 时用） |
| `sort_order` | INT | NOT NULL DEFAULT 0 | 排序权重，升序 |
| `status` | TINYINT | NOT NULL DEFAULT 0 | 0=下线, 1=上线 |
| `publish_time` | DATETIME | NULL | 上线时间（展示用） |
| 公共字段 | — | — | create_time/update_time/create_by/update_by/deleted |

- **索引**：`INDEX(status, sort_order)`（覆盖"上线 + 排序"高频查询）
- **热门帖子 / 热门景点**：**不新建表**，复用社区 `post` 与景点 `attraction`，通过聚合查询实现。

### 3.5 模块间依赖关系

- **依赖**：
  - 旅游社区（`post`）→ 提供热门帖子数据（只读）
  - 景点攻略（`attraction`）→ 提供热门景点数据（只读，依赖其种子数据先行）
- **数据流向**：社区 / 景点数据 ──(只读聚合)──▶ 首页展示
- **被依赖**：无（首页是终端展示层）
- **鉴权**：首页聚合接口全部加入 Sa-Token 白名单（游客可访问）

---

## 4. 模块②：景点攻略（P1 · 占位 + 数据先行）

> **本期形态**：用户侧 UI 为占位页（Coming Soon）；但**底层数据表 + 种子数据必须先行落地**，因为它是 P0 模块（AI 助手 RAG 知识源、首页热门景点）的硬依赖。

### 4.1 模块边界定义

**In Scope（本期，3 个功能点）**
1. **占位落地页**：「Coming Soon」预告页 + 返回首页入口
2. **底层数据表落地**：`attraction`（景点）、`guide`（攻略）、`city`（城市字典）
3. **种子数据导入**：少量精选景点 + 攻略（覆盖北京 / 上海 1–2 城），供 AI RAG 与首页使用

**Out of Scope（本期）**
- ❌ UGC 攻略发布
- ❌ 攻略编辑 / 运营后台
- ❌ 攻略评论、点赞、收藏
- ❌ 攻略全文搜索（依赖后续搜索引擎）

### 4.2 核心用户故事（未来愿景，本期占位）

- **US-2.1**（未来）：As a **准备来华的游客**，I want to 浏览结构化的景点攻略，so that 行前了解景点信息与注意事项。
- **US-2.2**（本期占位）：As a **访客**，I want to 点击景点攻略入口时得到明确反馈，so that 知道该功能即将上线而非遇到错误页。

### 4.3 验收标准

**AS-2.1 占位页展示**
- **WHEN** 用户点击首页「景点攻略」导航
- **THEN** 展示占位页，包含「Coming Soon」预告文案与功能预告
- **AND** 提供「返回首页」入口，页面状态码 200（非 404，保证 SEO 友好）

**AS-2.2 种子数据就绪（开发验收）**
- **WHEN** 应用启动或初始化脚本执行
- **THEN** `attraction` / `guide` 表存在至少 N 条（建议 ≥10 景点 + ≥10 攻略）种子数据
- **AND** 对应 `knowledge_document` 已完成向量化（供 AI 助手 RAG）

### 4.4 数据模型概要

**核心实体 1：`attraction`（景点）**

| 字段 | 类型 | 约束 | 说明 |
|------|------|------|------|
| `id` | BIGINT | PK, Snowflake | — |
| `name_en` | VARCHAR(200) | NOT NULL | 英文名 |
| `name_cn` | VARCHAR(200) | NOT NULL DEFAULT '' | 中文名 |
| `city_code` | VARCHAR(20) | NOT NULL | 所属城市编码（FK city.code） |
| `category` | VARCHAR(50) | NOT NULL DEFAULT '' | 类别（如 museum/park/landmark） |
| `description` | TEXT | NOT NULL | 英文描述（RAG 语料） |
| `cover_image_url` | VARCHAR(500) | NOT NULL DEFAULT '' | 封面图 |
| `popularity` | INT | NOT NULL DEFAULT 0 | 热度（首页排序用） |
| `status` | TINYINT | NOT NULL DEFAULT 1 | 0=下线, 1=上线 |
| 公共字段 | — | — | — |

- 索引：`INDEX(city_code, status)`（按城市筛选上线景点）

**核心实体 2：`guide`（攻略）**

| 字段 | 类型 | 约束 | 说明 |
|------|------|------|------|
| `id` | BIGINT | PK, Snowflake | — |
| `attraction_id` | BIGINT | NOT NULL | 关联景点 |
| `title` | VARCHAR(200) | NOT NULL | 攻略标题 |
| `content` | TEXT | NOT NULL | 攻略正文（Markdown，RAG 语料） |
| `confidence_level` | TINYINT | NOT NULL DEFAULT 1 | 信息置信度（1高/2中/3低，对齐调研报告） |
| `source_url` | VARCHAR(500) | NOT NULL DEFAULT '' | 信息来源（官方链接） |
| `status` | TINYINT | NOT NULL DEFAULT 1 | 0=下线, 1=上线 |
| 公共字段 | — | — | — |

- 索引：`INDEX(attraction_id, status)`

**核心实体 3：`city`（城市字典 · 公共字典表）**

| 字段 | 类型 | 约束 | 说明 |
|------|------|------|------|
| `id` | BIGINT | PK, Snowflake | — |
| `code` | VARCHAR(20) | NOT NULL | 城市编码（如 `BJS`） |
| `name_en` | VARCHAR(100) | NOT NULL | 英文名（如 `Beijing`） |
| `name_cn` | VARCHAR(100) | NOT NULL DEFAULT '' | 中文名 |
| `pinyin` | VARCHAR(100) | NOT NULL DEFAULT '' | 拼音 |
| `status` | TINYINT | NOT NULL DEFAULT 1 | — |
| 公共字段 | — | — | — |

- 唯一约束：`UNIQUE(code, deleted)`
- **跨模块复用**：`city` 表被景点攻略「拥有定义」，但社区模块的城市标签（`post.city_code`）复用它

**实体关系**：`city 1 ──< attraction 1 ──< guide`（一对多）

### 4.5 模块间依赖关系

- **被依赖**（关键）：
  - AI 助手依赖 `attraction + guide` 作为 **RAG 知识源**
  - 首页依赖 `attraction` 提供热门景点
- **依赖**：无（最底层内容源）
- **数据流向（核心）**：种子数据 ──(批量向量化)──▶ 知识库 ──(RAG 检索)──▶ AI 助手

---

## 5. 模块③：旅游社区（P0）

> **一句话价值**：用「帖子 + 城市标签 + 评论」启动数据飞轮，沉淀真实 UGC，反哺首页热度与 AI 知识。
> **严格遵守约束**：本期只做帖子 CRUD + 城市标签 + 评论，不做私信 / 关注 / 推荐算法。

### 5.1 模块边界定义

**In Scope（4 个功能点）**
1. **帖子 CRUD**：发帖、查看、编辑、删除（作者本人）
2. **城市标签**：帖子绑定一个城市（`city_code`）
3. **评论**：对帖子发表评论、查看评论列表（单层，不做楼中楼）
4. **帖子列表**：按城市 / 时间筛选、分页浏览

**Out of Scope（约束明确）**
- ❌ 私信
- ❌ 关注 / 粉丝关系
- ❌ 推荐算法（首页热门用固定公式聚合，非推荐引擎）
- ❌ 点赞 / 收藏（约束未列入，本期不做；首页热度改用 `comment_count + view_count` 计算）
- ❌ 评论嵌套（楼中楼）、评论点赞
- ❌ 富文本编辑器（本期纯文本 / Markdown 简化）

### 5.2 核心用户故事

- **US-3.1**：As a **已登录游客**，I want to 发布带城市标签的帖子，so that 分享来华旅行经验并帮助后来者。
- **US-3.2**：As a **浏览型游客**，I want to 按城市筛选帖子列表，so that 找到我目的地的真实经验。
- **US-3.3**：As a **已登录用户**，I want to 对帖子发表评论，so that 与发帖人互动、追问细节。

### 5.3 验收标准

**AS-3.1 发帖**
- **WHEN** 已登录用户提交帖子（`title` + `content` + `city_code`）
- **THEN** 校验必填与长度（`title≤200`, `content` 非空, `city_code` 存在于 city 字典）
- **AND** 创建 `post` 记录，`author_id` 取自当前会话，返回新帖子 ID（Long 序列化为 String）
- **AND** 未登录请求返回 `R<Void>` 鉴权错误（401）

**AS-3.2 帖子列表（按城市筛选）**
- **WHEN** 客户端以 `city_code` + `current` + `size` 请求帖子列表
- **THEN** 返回 `R<PageResult<PostVO>>`，仅含 `status=1 AND deleted=0` 帖子
- **AND** 按 `create_time` 降序；`size>200` 截断为 200，`size≤0` 默认 10（遵循 api-contract）

**AS-3.3 评论**
- **WHEN** 已登录用户对帖子发评论
- **THEN** 创建 `comment` 记录并原子更新 `post.comment_count + 1`（乐观锁 `@Version` 防并发）
- **AND** 评论列表按 `create_time` 正序分页返回

**AS-3.4 帖子删除（权限）**
- **WHEN** 帖子作者请求删除自己的帖子
- **THEN** 逻辑删除（`deleted=1`），后续查询自动排除
- **AND** 非作者删除请求返回授权错误（403）

### 5.4 数据模型概要

**核心实体 1：`post`（帖子）**

| 字段 | 类型 | 约束 | 说明 |
|------|------|------|------|
| `id` | BIGINT | PK, Snowflake | — |
| `author_id` | BIGINT | NOT NULL | 作者（FK sys_user.id） |
| `title` | VARCHAR(200) | NOT NULL | 标题 |
| `content` | TEXT | NOT NULL | 正文 |
| `city_code` | VARCHAR(20) | NOT NULL | 城市标签（FK city.code） |
| `view_count` | INT | NOT NULL DEFAULT 0 | 浏览数（首页热度来源） |
| `comment_count` | INT | NOT NULL DEFAULT 0 | 评论数（冗余计数，首页热度来源） |
| `status` | TINYINT | NOT NULL DEFAULT 1 | 0=草稿, 1=已发布 |
| `version` | INT | NOT NULL DEFAULT 0 | 乐观锁（comment_count 并发更新） |
| 公共字段 | — | — | create_time/update_time/create_by/update_by/deleted |

- 索引：
  - `INDEX(city_code, create_time)`（按城市 + 时间筛选，首页/列表主力查询）
  - `INDEX(author_id, create_time)`（查询用户自己的帖子）
- **热度说明**：不做点赞表，首页热度 = `comment_count * 3 + view_count`

**核心实体 2：`comment`（评论）**

| 字段 | 类型 | 约束 | 说明 |
|------|------|------|------|
| `id` | BIGINT | PK, Snowflake | — |
| `post_id` | BIGINT | NOT NULL | 关联帖子（FK post.id） |
| `author_id` | BIGINT | NOT NULL | 评论者（FK sys_user.id） |
| `content` | VARCHAR(1000) | NOT NULL | 评论内容（本期单层，无 parent_id） |
| 公共字段 | — | — | — |

- 索引：`INDEX(post_id, create_time)`（按帖子加载评论，遵循规范消息表范式）

**实体关系**：
- `sys_user 1 ──< post N`（一个用户多篇帖子）
- `post 1 ──< comment N`（一篇帖子多条评论）
- `city 1 ──< post N`（一个城市多篇帖子，通过 city_code 关联）

### 5.5 模块间依赖关系

- **依赖**：
  - 账号基础设施（`sys_user`，作者 / 评论者身份）
  - `city` 字典表（城市标签，由景点攻略模块定义，社区复用）
- **被依赖**：首页（热门帖子数据源，只读聚合）
- **数据流向**：用户发帖/评论 ──▶ post/comment 表 ──(只读聚合)──▶ 首页热门区

---

## 6. 模块④：AI 助手（P0）

> **一句话价值**：差异化护城河。基于 RAG 的景点 / 攻略问答，让游客"此刻此地获得准确答案"。
> **严格遵守约束**：本期只做基于 RAG 的景点 / 攻略问答，不做通用聊天、不做外部 API 调用。

### 6.1 模块边界定义

**In Scope（4 个功能点）**
1. **多轮会话管理**：创建会话、查看历史会话、会话内追问
2. **基于 RAG 的景点 / 攻略问答**：检索知识库（`attraction + guide`）→ 注入 prompt → Spring AI 生成回答
3. **答案来源标注**：每条回答附带引用的景点 / 攻略来源（可溯源，提升可信度）
4. **会话历史持久化**：保存用户消息与 AI 回答，支持历史回看

**Out of Scope（约束明确）**
- ❌ 通用聊天（超出景点 / 攻略范围的问题礼貌拒答并引导）
- ❌ 外部 API 调用（不联网检索、不调第三方服务）
- ❌ 拍照翻译 / 多模态输入（约束明确不做）
- ❌ 位置 + 行程上下文记忆（较产品大纲收窄，本期仅会话内文本上下文）
- ❌ 转真人求助
- ❌ 答案一键复制 / 分享卡片（简化，延后）

### 6.2 核心用户故事

- **US-4.1**：As a **准备来华的游客**，I want to 用自然语言提问景点 / 攻略问题，so that 获得基于结构化攻略的准确回答。
- **US-4.2**：As a **在地游客**，I want to 在同一会话中追问，so that 多轮深入解决行程疑问。
- **US-4.3**：As a **用户**，I want to 看到回答引用的来源景点 / 攻略，so that 核实信息可信度。

### 6.3 验收标准

**AS-4.1 首次提问（创建会话）**
- **WHEN** 已登录用户发起首次提问
- **THEN** 创建 `chat_session`，保存用户消息（role=USER）
- **AND** 触发 RAG 检索 + Spring AI 生成，保存 AI 回答（role=ASSISTANT）及其来源列表
- **AND** 未登录请求返回鉴权错误（401）

**AS-4.2 RAG 问答流程**
- **WHEN** 用户提问
- **THEN** 系统从知识库检索 Top-K（建议 K=3）相关片段 → 注入 prompt 上下文 → 调用 Spring AI 生成
- **AND** 回答 JSON 含 `content` 与 `sources`（来源 attraction/guide 的 ID + 标题列表）
- **AND** LLM 仅基于检索到的知识库内容作答（禁止编造未检索到的信息）

**AS-4.3 多轮追问（上下文）**
- **WHEN** 用户在同一会话追问
- **THEN** 携带该会话最近 N 轮（建议 N=6 条消息）历史 + 当前问题的 RAG 检索结果
- **AND** 更新 `chat_session.update_time` 与 `message_count`

**AS-4.4 越界拒答（不做通用聊天）**
- **WHEN** 用户提问超出景点 / 攻略范围（如闲聊、天气、政治）
- **THEN** 系统礼貌拒答，并引导用户回到景点 / 攻略主题
- **AND** **不调用任何外部 API**（约束硬性要求）

**AS-4.5 知识库兜底**
- **WHEN** RAG 检索无相关结果（相似度均低于阈值）
- **THEN** 返回"暂无相关信息，建议浏览景点攻略"兜底回答
- **AND** 不调用外部 API，sources 为空

### 6.4 数据模型概要

**核心实体 1：`chat_session`（会话）**

| 字段 | 类型 | 约束 | 说明 |
|------|------|------|------|
| `id` | BIGINT | PK, Snowflake | — |
| `user_id` | BIGINT | NOT NULL | 所属用户（FK sys_user.id） |
| `title` | VARCHAR(200) | NOT NULL DEFAULT '' | 会话标题（取首问摘要） |
| `message_count` | INT | NOT NULL DEFAULT 0 | 消息数 |
| 公共字段 | — | — | — |

- 索引：`INDEX(user_id, update_time)`（用户会话列表按最近活动排序，遵循规范对话表范式）

**核心实体 2：`chat_message`（消息）**

| 字段 | 类型 | 约束 | 说明 |
|------|------|------|------|
| `id` | BIGINT | PK, Snowflake | — |
| `session_id` | BIGINT | NOT NULL | 关联会话（FK chat_session.id） |
| `role` | VARCHAR(20) | NOT NULL | `USER` / `ASSISTANT` |
| `content` | TEXT | NOT NULL | 消息内容 |
| `sources` | JSON | NULL | 引用来源（仅 ASSISTANT 有；`[{type,id,title}]`） |
| `tokens_used` | INT | NULL | Token 消耗（流式完成后才有值，故允许 NULL） |
| 公共字段 | — | — | — |

- 索引：`INDEX(session_id, create_time)`（按会话加载消息，遵循规范消息表范式）
- **NULL 说明**：`sources` 与 `tokens_used` 允许 NULL —— 符合规范"未产生/不存在"语义（用户消息无来源；tokens 流式完成后才统计）

**核心实体 3：`knowledge_document`（RAG 知识库文档）**

| 字段 | 类型 | 约束 | 说明 |
|------|------|------|------|
| `id` | BIGINT | PK, Snowflake | — |
| `source_type` | VARCHAR(20) | NOT NULL | `ATTRACTION` / `GUIDE` |
| `source_id` | BIGINT | NOT NULL | 源数据 ID（attraction/guide） |
| `title` | VARCHAR(200) | NOT NULL | 文档标题 |
| `content` | TEXT | NOT NULL | 向量化原文 |
| `vector_id` | VARCHAR(100) | NOT NULL | 向量库中的 ID |
| `status` | TINYINT | NOT NULL DEFAULT 1 | 0=失效, 1=有效 |
| 公共字段 | — | — | — |

- 索引：`INDEX(source_type, source_id)`（按源数据回溯文档）
- **向量存储方案**（本期最简可行）：采用 **Spring AI 内置 `SimpleVectorStore`**（文件持久化），避免引入 pgvector / Redis 等重型依赖；种子数据初始化时批量向量化。
- **同步策略**：本期景点攻略为静态种子数据，**初始化时一次性向量化**，不做实时增量同步（降低复杂度，符合 3 周预算）。

**实体关系**：
- `sys_user 1 ──< chat_session 1 ──< chat_message N`
- `attraction / guide 1 ──1 knowledge_document`（知识库文档与源数据一一对应，向量化映射）

### 6.5 模块间依赖关系

- **依赖**：
  - 账号基础设施（`sys_user`，会话归属）
  - **景点攻略模块数据**（`attraction + guide` → 转化为 `knowledge_document` → RAG 知识源）—— **P0 依赖 P1 数据层，必须先行**
- **数据流向（核心闭环）**：
  ```
  种子攻略(attraction/guide)
        │ (初始化批量向量化)
        ▼
  knowledge_document + 向量库
        │ (用户提问 → RAG 检索 Top-K)
        ▼
  Spring AI 生成回答
        │ (持久化 + 来源标注)
        ▼
  chat_message → 返回前端
  ```
- **被依赖**：无

---

## 7. 模块⑤：行程规划（P2 · 占位）

> **本期形态**：仅占位页，不实现核心逻辑。

### 7.1 模块边界定义

**In Scope（本期，1 个功能点）**
1. **占位落地页**：「Coming Soon」预告页 + 返回首页入口

**Out of Scope（本期，未来规划）**
- ❌ AI 行程生成 / 智能编排
- ❌ 日程拖拽编辑 / 多日多景点编排
- ❌ 地图路线绘制 / 导航串联
- ❌ 行程分享 / 导出

### 7.2 核心用户故事（未来愿景）

- **US-5.1**（未来）：As a **准备来华的游客**，I want to 输入目的地与天数生成行程，so that 获得可执行的日程安排。
- **US-5.2**（本期占位）：As a **访客**，I want to 点击行程规划入口得到清晰预告，so that 了解未来功能。

### 7.3 验收标准

**AS-7.1 占位页展示**
- **WHEN** 用户点击首页「行程规划」导航
- **THEN** 展示占位页（Coming Soon + 功能预告）
- **AND** 提供「返回首页」入口，状态码 200

### 7.4 数据模型概要（本期不建表，预留设计）

未来设计预留（本期不实现）：
- `itinerary`（行程）：`id / user_id / title / city_code / days / status / 公共字段`
- `itinerary_item`（行程项）：`id / itinerary_id / day_index / attraction_id / sort_order / 公共字段`

### 7.5 模块间依赖关系

- **本期无依赖**（占位页）
- **未来依赖**：景点攻略（景点池）、AI 助手（智能编排）、账号（用户归属）

---

## 8. 模块⑥：实用工具（P1/P2 · 占位）

> **本期形态**：仅占位页。
> **⚠️ 价值提醒**：本模块未来将承载前两轮文档中评估为「**最高频 × 最基础痛点**」的「入境生存清单」（支付 / VPN / 手机号 / 地图 / 免签政策，对应调研痛点 TOP1–2 与 SEO 获客钩子）。**本期虽占位，但建议在 P0 验证通过后优先提升其优先级**——这是与当前优先级决策的张力点，已暴露，待后续评估。

### 8.1 模块边界定义

**In Scope（本期，1 个功能点）**
1. **占位落地页**：「Coming Soon」预告页，列出未来工具清单预告（支付指南 / 免签政策 / VPN / 地图）+ 返回首页

**Out of Scope（本期，未来规划）**
- ❌ 入境生存清单 checklist（支付 / VPN / 手机号 / 地图实操指引）
- ❌ 免签政策实时同步（240h / 55 国等）
- ❌ 离线缓存 / 进度勾选
- ❌ 汇率换算 / 翻译 / 单位换算等单点工具

### 8.2 核心用户故事（未来愿景）

- **US-6.1**（未来）：As a **首次来华游客**，I want to 一份照着做就行的入境生存清单，so that 落地第一天能活下来。
- **US-6.2**（本期占位）：As a **访客**，I want to 知道这些实用工具即将上线，so that 期待未来使用。

### 8.3 验收标准

**AS-8.1 占位页展示**
- **WHEN** 用户点击首页「实用工具」导航
- **THEN** 展示占位页，列出未来工具清单预告
- **AND** 提供「返回首页」入口，状态码 200

### 8.4 数据模型概要

本期不建表。未来承载清单类内容时，可设计 `tool_checklist` / `tool_item` 等表。

### 8.5 模块间依赖关系

- **本期无依赖**（占位页）
- **未来依赖**：账号（用户进度勾选 / 偏好）、景点攻略（内容底座）

---

## 9. 附录 A：模块间依赖关系总图

### 9.1 依赖矩阵

| 模块 \ 依赖 → | 账号 | 首页 | 景点攻略 | 社区 | AI助手 | 行程规划 | 实用工具 |
|---------------|:----:|:----:|:--------:|:----:|:------:|:--------:|:--------:|
| ① 首页 | ✗ | — | **🔴数据** | **✓数据** | ✗ | ✗ | ✗ |
| ② 景点攻略 | ✗ | ✗ | — | ✗ | ✗ | ✗ | ✗ |
| ③ 社区 | **✓身份** | ✗ | **✓字典** | — | ✗ | ✗ | ✗ |
| ④ AI助手 | **✓身份** | ✗ | **🔴数据** | ✗ | — | ✗ | ✗ |
| ⑤ 行程规划（占位）| ✗ | ✗ | （未来）| ✗ | （未来）| — | ✗ |
| ⑥ 实用工具（占位）| ✗ | ✗ | （未来）| ✗ | ✗ | ✗ | — |

- `✓` = 本期运行时依赖；`🔴` = **跨优先级硬依赖**（P0 依赖 P1 的数据层，需先行）；`✗` = 无依赖
- **身份** = 依赖账号体系的用户身份；**数据** = 依赖其业务数据（只读聚合）；**字典** = 复用 city 字典表

### 9.2 数据流向图

```
                    ┌─────────────────────────────────────────┐
                    │           公共基础设施 · 账号            │
                    │         (sys_user · Sa-Token)            │
                    └────────────┬────────────────────────────┘
                                 │ 身份 (author_id / user_id)
            ┌────────────────────┼────────────────────┐
            ▼                    ▼                    ▼
     ┌────────────┐       ┌────────────┐       ┌────────────┐
     │ ③ 旅游社区 │       │ ④ AI助手   │       │ ① 首页     │
     │  post      │       │ chat_session│      │ (聚合层)   │
     │  comment   │       │ chat_message│      │            │
     └─────┬──────┘       └──────┬─────┘       └─────┬──────┘
           │                     │                    │
           │  city_code          │ RAG知识源          │ 只读聚合
           │ (复用city字典)      │                    │
           │                     ▼                    │
           │              ┌──────────────────┐        │ 热门帖子
           │              │ knowledge_document│       │◄────────┘
           │              │ + 向量库          │        │
           │              └────────┬─────────┘        │ 热门景点
           │                       │ 批量向量化        │◄────────┐
           ▼                       │                    │        │
     ┌──────────┐                  │                    │        │
     │ city字典 │◄─────────────────┤                    │        │
     └────┬─────┘                  │                    │        │
          │                        ▼                    │        │
          │              ┌──────────────────┐           │        │
          └─────────────▶│ ② 景点攻略(数据) │───────────┴────────┘
                         │ attraction       │ (P1: UI占位,
                         │ guide            │  数据先行)
                         └──────────────────┘
```

### 9.3 关键冲突与风险（错误显性化）

| # | 冲突 / 风险 | 说明 | 处理建议 |
|---|-------------|------|----------|
| R1 | **AI 助手(P0) 依赖景点攻略(P1)数据** | AI 助手 RAG 知识源来自 `attraction+guide`，但景点攻略 UI 本期占位 | **景点攻略"数据层"提升为 P0 级先行任务**：UI 占位，但表结构 + 种子数据 + 向量化必须在 AI 助手开发前完成（见 §4.2、§6.5） |
| R2 | **首页(P0) 依赖景点攻略(P1)数据** | 热门景点来自 `attraction` | 同 R1，依赖景点攻略种子数据先行；无数据时首页该区域返回空态兜底（AS-1.3） |
| R3 | **首页 / 社区游客访问 vs Sa-Token 白名单** | 现有白名单仅含登录与文档端点，但首页 / 帖子详情需游客可访问 | 本期**扩展 Sa-Token 白名单**（见 §1.3），将只读公开端点加入 |
| R4 | **产品设想"Google 登录" vs 现有 username/password 体系** | 产品定位面向海外游客偏好 OAuth，但现有 auth-session 为 username/password | 本期遵循现有体系（**邮箱+密码**），Google OAuth 列 Out of Scope，后续扩展（见 §1.1） |
| R5 | **社区"不做点赞" vs 首页热度计算** | 约束未将点赞列入社区范围，但首页需热度指标 | 首页热度改用 `comment_count*3 + view_count` 公式（无点赞表），既守约束又能算热度（见 §5.4） |
| R6 | **实用工具占位 vs 其高价值定位** | 实用工具承载的「入境生存清单」是调研痛点 TOP1–2，却本期占位 | 已暴露张力，建议 P0 验证后**优先提升实用工具优先级**（见 §8） |

---

## 附录 B：本期建表清单与开发顺序建议

### B.1 建表清单（遵循 database-design-conventions）

| 顺序 | 表名 | 所属模块 | 说明 |
|------|------|----------|------|
| 0 | `sys_user` | 公共基础设施 | **复用现有**，不新建 |
| 1 | `city` | 景点攻略（公共字典） | 城市字典，多模块复用 |
| 2 | `attraction` | 景点攻略 | 景点，AI RAG 语料 + 首页数据源 |
| 3 | `guide` | 景点攻略 | 攻略，AI RAG 语料 |
| 4 | `knowledge_document` | AI 助手 | RAG 知识库文档元数据 |
| 5 | `chat_session` | AI 助手 | 会话 |
| 6 | `chat_message` | AI 助手 | 消息 |
| 7 | `post` | 社区 | 帖子 |
| 8 | `comment` | 社区 | 评论 |
| 9 | `home_recommend` | 首页 | 编辑推荐 |

> 所有表均遵循：Snowflake 主键、BaseEntity 公共字段、逻辑删除、`ddl-auto: validate`（需手写 DDL，Hibernate 仅校验）。

### B.2 建议开发顺序（基于依赖关系）

```
Week 1: 地基 + 内容源
  ├─ 账号基础设施（复用 sys_user + 扩展白名单）
  ├─ city / attraction / guide 表 + 种子数据          ← 解决 R1/R2 前置
  └─ knowledge_document 表 + 种子数据向量化

Week 2: P0 核心闭环
  ├─ 旅游社区（post / comment CRUD + 城市标签）
  ├─ AI 助手（chat_session/message + RAG 问答 + 来源标注 + 越界拒答）
  └─ 首页（home_recommend + 热门聚合）

Week 3: 占位页 + 联调核验
  ├─ 景点攻略 / 行程规划 / 实用工具 占位页
  ├─ 端到端联调（首页→社区→AI 闭环）
  └─ 核验：约束遵守 + 规范对齐 + 验收标准逐条验证
```

---

## 待确认事项

1. **种子数据规模**：景点攻略种子数据建议 ≥10 景点 + ≥10 攻略（覆盖北京 / 上海），是否认可？城市范围是否仅限调研的 6 大枢纽（北京 / 上海 / 成都 / 西安 / 广州 / 深圳）？
2. **向量存储选型**：本期采用 Spring AI `SimpleVectorStore`（文件持久化，零额外依赖），是否符合"轻量化"预期？还是项目已有 Redis / pgvector 可复用？
3. **R1/R2 前置任务归属**：景点攻略"数据层先行"是否正式纳入 Week 1 范围（影响 P1 边界，需确认）？
4. **R6 优先级张力**：实用工具（入境生存清单）是否考虑在 3 周末尾若有余力时从占位升级为最小可用？
