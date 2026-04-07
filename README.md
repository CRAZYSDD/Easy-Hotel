# 易宿酒店预订平台

易宿酒店预订平台是一个面向酒店预订业务场景的前后端分离项目，包含移动端用户预订流程和 PC 端酒店管理后台。项目围绕真实业务链路设计，覆盖酒店查询、筛选排序、详情浏览、收藏、最近浏览、房型库存、模拟预订、商户酒店维护、管理员审核发布、下线恢复和操作日志等功能。

项目采用 TypeScript 技术栈开发，前端使用 React、Vite、Redux Toolkit、React Router、Ant Design Mobile 和 Ant Design，后端使用 Node.js、Express、Prisma 和 SQLite，并通过 SSE 实现后台状态变化后的端侧实时刷新。

## 项目定位

本项目模拟一个完整的酒店预订平台，主要分为两个端：

- 用户端：面向移动端用户，提供酒店搜索、筛选、浏览、收藏和预订体验。
- 管理端：面向商户和管理员，提供酒店录入、编辑、审核、发布、下线和恢复等管理能力。

项目重点不只是页面展示，而是实现真实的数据流和状态变化。例如商户修改酒店信息后，管理员可以在审核列表中查看；管理员发布或下线酒店后，用户端展示数据会跟随变化；用户预订房型后，房型库存会减少，售罄状态会同步影响列表、详情、收藏和最近浏览等页面。

## 技术栈

### 前端

- React
- TypeScript
- Vite
- React Router
- Redux Toolkit
- Axios
- Ant Design Mobile
- Ant Design
- CSS Modules
- dayjs
- SSE

### 后端

- Node.js
- TypeScript
- Express
- Prisma
- SQLite
- JWT
- Zod
- SSE

## 功能模块

### 移动端用户侧

用户端以移动端预订体验为核心，包含首页、酒店列表页、酒店详情页和我的收藏页。

首页提供城市切换、关键字搜索、入住/离店日期选择、星级和价格筛选、热门偏好标签、智能推荐组合、搜索历史、最近浏览和我的收藏入口。用户点击查询后，会带着城市、日期、关键字、星级、价格和标签等条件进入酒店列表页。

酒店列表页支持多条件组合筛选，包括城市、关键字、星级、价格区间、标签和价格排序。列表使用分页加载，并通过 IntersectionObserver 实现无限滚动。筛选与排序区域采用抽屉式交互，默认收起，点击后展开筛选内容，减少首屏信息压力。

酒店卡片展示酒店封面、中文名、英文名、星级档次、标签、地址、设施、优惠信息、最低可订价和库存状态。若酒店所有房型售罄，卡片会置灰，按钮变为“已售罄”，并在列表中沉底展示。

酒店详情页展示酒店大图、基础信息、设施标签、推荐理由、周边信息、优惠信息和房型价格列表。房型按照“可订优先、价格升序”的规则展示，售罄房型自动沉底，底部固定操作栏会根据当前可订房型动态计算最低价。当酒店全部房型售罄时，底部按钮会禁用并显示“已售罄”。

我的收藏页展示用户收藏过的酒店，支持取消收藏、售罄状态展示和售罄酒店沉底。收藏状态通过 Redux 和 localStorage 持久化，并在列表页、详情页、首页入口和收藏页之间保持同步。

### PC 管理端

管理端基于 Ant Design 构建，包含登录注册、商户后台和管理员后台。

登录注册模块支持商户和管理员两类角色。注册时可以选择角色，登录时系统根据账号角色自动跳转到对应后台。未登录访问后台页面时会被拦截并跳回登录页。

商户后台支持酒店列表查看、新增酒店、编辑酒店和动态维护房型。酒店表单包含中文名、英文名、城市、地址、星级、开业年份、标签、设施、封面图、相册图、优惠信息、周边信息、房型名称、床型、早餐、取消规则、价格、原价、库存、面积和入住人数等字段。

管理员后台支持查看全部酒店，并按审核状态和发布状态筛选。管理员可以执行审核通过、审核驳回、发布、下线、恢复上线和查看详情等操作。审核驳回时必须填写驳回原因；下线不是删除，后续可以恢复上线。后台还提供操作日志，用于展示关键管理行为。

## 核心业务规则

用户端只展示满足以下条件的酒店：

- 审核状态为 `approved`
- 发布状态为 `published`
- 未处于下线状态

房型价格列表按照以下规则展示：

- 可订房型优先展示
- 可订房型按价格从低到高排序
- 售罄房型放在列表底部

酒店售罄状态按照以下规则处理：

- 当某个房型库存为 0 时，该房型显示“已售罄”并禁用预订按钮。
- 当酒店全部房型库存为 0 时，酒店在列表、收藏和最近浏览中置灰并沉底。
- 酒店全部售罄后，详情页底部固定操作栏显示“已售罄”，不可继续预订。
- 当后台补充库存后，用户端对应状态会自动恢复。

审核与发布流程按照以下规则处理：

- 商户新增或编辑酒店后，酒店进入待审核或待发布状态。
- 管理员审核通过后，酒店才允许发布。
- 管理员审核驳回时必须填写原因。
- 管理员下线酒店后，用户端不再展示该酒店。
- 管理员恢复上线后，符合条件的酒店会重新出现在用户端。

## 实时同步

项目通过 SSE 建立后端到前端的单向实时消息通道，用于处理管理端和用户端之间的数据同步。

当发生以下事件时，前端会自动刷新相关数据：

- 商户修改酒店信息
- 管理员审核酒店
- 管理员发布酒店
- 管理员下线酒店
- 管理员恢复酒店
- 用户提交预订导致库存变化

用户端列表、详情、收藏和最近浏览会根据这些事件重新拉取数据，保证页面展示与后端状态保持一致。

## 状态管理

项目使用 Redux Toolkit 管理跨页面共享状态，主要包括：

- 登录态和当前用户信息
- 酒店列表数据
- 酒店详情数据
- 房型数据
- Banner 数据
- 搜索条件
- 搜索历史
- 收藏状态
- 最近浏览数据

对于收藏、最近浏览和搜索历史等用户行为数据，项目结合 localStorage 做本地持久化，使页面刷新后仍能保留用户偏好。

## 交互与体验设计

项目在移动端和管理端都补充了较多体验细节：

- 酒店列表支持无限滚动加载。
- 筛选与排序区域支持抽屉展开动画。
- 日期选择使用自定义日历组件，支持入住和离店区间选择。
- 收藏和取消收藏提供即时提示。
- 搜索历史支持点击查询和单条删除。
- 页面跳转和返回时自动回到页面顶部。
- 酒店售罄后按钮禁用、卡片置灰并沉底。
- 后台表单包含校验和错误提示。
- 管理端按钮根据当前状态禁用不合法操作，避免误操作。
- 接口异常、空数据和加载状态都有基础提示。

## 目录结构

```text
easy-stay-hotel-platform-ts/
  client/
    src/
      api/              # Axios 请求封装与接口方法
      components/       # 公共组件和移动端业务组件
      constants/        # 酒店标签、城市等常量配置
      hooks/            # 自定义 Hook，例如 SSE 刷新
      layouts/          # 移动端布局和管理端布局
      pages/            # 移动端页面与后台页面
      router/           # 路由配置
      store/            # Redux Toolkit 状态管理
      styles/           # 全局样式
      types/            # 前端业务类型定义
      utils/            # 工具函数
  server/
    prisma/             # Prisma schema 和 seed 数据
    src/
      controllers/      # 控制器
      middlewares/      # 鉴权、角色校验、错误处理
      routes/           # 路由模块
      services/         # 业务服务层
      sse/              # SSE 推送逻辑
      utils/            # 参数校验和工具函数
  docs/                 # 项目文档
  README.md
```

## 本地运行

安装依赖：

```bash
npm install
```

初始化数据库：

```bash
npm run prisma:generate --workspace server
npm run db:push --workspace server
npm run seed --workspace server
```

启动项目：

```bash
npm run dev
```

访问地址：

- 用户端首页：http://localhost:5173
- 后台登录页：http://localhost:5173/admin/login
- 后端健康检查：http://localhost:4000/api/health

## 验证命令

```bash
npm run typecheck
npm run build
```

## 默认测试账号

- 管理员：`admin / 123456`
- 商户1：`merchant1 / 123456`
- 商户2：`merchant2 / 123456`

## 主要接口

认证相关：

- `POST /api/auth/register`
- `POST /api/auth/login`
- `GET /api/auth/me`

用户端相关：

- `GET /api/banners`
- `GET /api/hotels`
- `GET /api/hotels/:id`
- `GET /api/hotels/:id/rooms`
- `POST /api/bookings`
- `GET /api/events`

商户端相关：

- `GET /api/merchant/hotels`
- `GET /api/merchant/hotels/:id`
- `POST /api/merchant/hotels`
- `PUT /api/merchant/hotels/:id`
- `POST /api/merchant/hotels/:id/rooms`
- `PUT /api/merchant/rooms/:roomId`
- `DELETE /api/merchant/rooms/:roomId`

管理员端相关：

- `GET /api/admin/hotels`
- `PATCH /api/admin/hotels/:id/audit`
- `PATCH /api/admin/hotels/:id/publish`
- `PATCH /api/admin/hotels/:id/offline`
- `PATCH /api/admin/hotels/:id/restore`
- `GET /api/admin/logs`
