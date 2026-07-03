# NSGlamour 模块计划

## 当前状态

- 模块状态：迁移占位页已接入，真实业务待重写/迁移。
- 目标路由：`#/ffxiv/glamour`。
- 当前页面入口：`src/pages/glamour/GlamourPage.vue`。
- 来源项目路径：`NSGlamour`。
- 当前 API 边界：V2 使用 `/api/glamour/*`，开发代理 rewrite 到旧服务 `/api/*`。
- 当前连通性检查：`/api/glamour/health`，对应旧服务 `/api/health`。
- 当前旧服务端口：`8765`。

## 定位

`#/ffxiv/glamour` 是 FFXIV 幻化工具模块，目标是承接旧 `NSGlamour` 的核心能力，但不是把旧项目代码机械搬进 V2。

新实现应优先按 V2 的 Vue、模块文档、公共组件、公共 CSS、API 边界和安全规则重新整理。旧项目在迁移期间作为行为契约、数据契约和回归样本保留。

核心目标：

1. `.chara` 解析和装备/染剂映射严谨可靠。
2. 石之家与 Eorzea Collection 导入结果与旧项目一致或更稳。
3. `/template` 模板生成和 `/equipinfo` 装备信息工作流在 V2 中重新组织。
4. 模板 Canvas 渲染、图片裁剪、PNG 导出尽量保持旧项目已校准的视觉结果。
5. 多语言装备、染剂和 UI 本地化能力保留，并为后续扩展留出边界。

## 后端重写策略

本模块允许按新规则重写后端，但必须先抽取旧后端行为契约。

推荐顺序：

1. 记录旧 API 输入、输出、错误结构和关键字段。
2. 用真实样本建立回归用例，包括 `.chara`、石之家链接、EC 链接、成段文字和模板草稿。
3. 在 V2 API 命名空间下稳定对外路径：`/api/glamour/*`。
4. 新后端按契约实现，不要求复制旧 Flask 内部结构。
5. 新旧后端并行一段时间，用同样样本比对解析结果和导出结果。
6. 等价验证通过前，不删除旧 `NSGlamour` 和旧部署入口。

重点风险：

- 染剂槽一致性、空染色、不可染色、双染色、饰品/面部配饰/时尚配饰规则容易回归。
- 主手/副手互斥、双手武器清理、副手候选和同模型装备替换容易回归。
- 模板渲染对坐标、字体、裁剪、阴影、描边和导出尺寸敏感。
- 外部链接导入和后台浏览器功能涉及公开部署安全边界。

## 旧项目契约参考

旧后端关键入口位于：

```text
../NSGlamour/scripts/app.py
```

已确认旧接口：

| 旧接口 | V2 目标路径 | 用途 |
|--------|-------------|------|
| `GET /api/health` | `GET /api/glamour/health` | 连通性检查 |
| `GET /api/ui-localization` | `GET /api/glamour/ui-localization` | UI 本地化数据 |
| `POST /api/import-glamour-link` | `POST /api/glamour/import-glamour-link` | 石之家/EC 链接导入 |
| `POST /api/equipinfo/parse-text` | `POST /api/glamour/equipinfo/parse-text` | 成段文字识别 |
| `POST /api/risingstones-browser/open-login` | `POST /api/glamour/risingstones-browser/open-login` | 石之家后台浏览器登录辅助 |
| `GET /api/icon/<icon_id>` | `GET /api/glamour/icon/<icon_id>` | 图标代理 |
| `GET /api/stains` | `GET /api/glamour/stains` | 染剂数据 |
| `GET /api/search-items` | `GET /api/glamour/search-items` | 装备搜索 |
| `POST /api/parse-chara` | `POST /api/glamour/parse-chara` | `.chara` 解析 |

旧静态/资源入口：

| 旧入口 | 迁移注意 |
|--------|----------|
| `/font/<path>` | 字体体积和授权要单独确认 |
| `/template-preview/<path>` | 模板预览资源后续可考虑迁入 V2 静态资源或后端资源服务 |
| `data/item_model_mapping.json` | 装备、染剂、图标、模型码核心映射，不手写临时替代 |
| `data/ui-localization.json` | UI 文案来源之一，未来应进入 V2 本地化规划 |

旧外部依赖：

- 石之家接口基础地址：`https://apiff14risingstones.web.sdo.com/api/home/`。
- 石之家页面和接口可能变化，链接导入必须保守处理。

## 核心数据契约

迁移或重写时不得轻易改变这些字段语义：

- `resolved_equipment`
- `candidates`
- `dye_entries`
- `names`
- `model_main`
- 装备部位、染剂槽、图标 ID、模型码、语言名称、来源信息

装备合法性检查应成为导入链路的一部分：

1. 染剂属性一致性：染剂槽数量、不可染色、空染色必须与物品属性一致。
2. 主手/副手互斥：主手装备不可能再装备副手时，自动删除或禁用副手数据。
3. 候选替换一致性：同模型装备替换后重新校验染剂和副手规则。
4. 饰品、面部配饰、时尚配饰按旧项目显示规则处理，不套普通装备染色展示逻辑。

## 前端状态和存储参考

旧项目重要状态键：

| 键/机制 | 用途 |
|---------|------|
| `nsglamour.theme` | 旧项目主题 |
| `nsglamour.cardDraft.v2` | 模板/装备草稿 |
| `nsglamour.recentLoadouts` | 最近导入记录 |
| `nsglamour.copyTemplate` | 自定义复制模板 |
| `nsglamour.copyFormat` | 文案格式 |
| `nsglamour.ignoreEmperor` | 皇帝套过滤设置 |
| `nsglamour.templateWorkspaceSettings` | 模板工作台设置 |
| `NSGlamourStore` | 旧项目跨页面共享状态 |
| `nsglamour-template-images-v2` IndexedDB | 模板上传图片持久化 |

V2 不要求复用这些键名，但迁移时必须处理旧数据兼容或迁移提示，不能无声丢失用户草稿。

## 页面拆分方向

建议 V2 前端拆分：

```text
src/pages/glamour/
├── GlamourPage.vue
├── components/
│   ├── GlamourImportPanel.vue
│   ├── GlamourEquipmentEditor.vue
│   ├── GlamourTemplateWorkspace.vue
│   ├── GlamourEquipInfoWorkspace.vue
│   ├── GlamourLanguageSwitcher.vue
│   └── GlamourExportPanel.vue
└── composables/
    ├── useGlamourDraft.ts
    ├── useGlamourImport.ts
    └── useGlamourTemplateImages.ts

src/lib/glamour/
├── equipment/
├── dyes/
├── importers/
├── templates/
└── renderers/
```

原则：

1. Vue 组件负责状态组合、表单、交互和 DOM 生命周期。
2. 装备解析、染剂归一、合法性检查、文案格式化放入 `src/lib/glamour/`。
3. Canvas 渲染器放入 `src/lib/glamour/templates/` 或 `src/lib/glamour/renderers/`，保持可测试。
4. API 调用通过 `useFetch.ts` 和 `/api/glamour`，页面不硬编码旧端口。

## CSS 和组件策略

- 样式层级：工具页业务布局为页面专属；工具页内可复用控件优先公共组件。
- 按钮、面板、输入框、状态提示、弹窗、工具栏优先使用 `src/components/` 和 `src/styles/`。
- 模板画布、装备编辑区、导入工作流等复杂布局可写页面私有样式。
- 不把旧 `NSGlamour` 的琥珀色 UI 整套复制成 V2 全站样式。
- 不把首页像素风装饰带入工具工作台。
- 如果 Glamour 与 Plate 都需要同类工具页外壳，应沉淀到 `src/pages/ffxiv/components/` 或公共组件。

## 安全边界

所有上传、粘贴、外部链接和后台浏览器相关数据都视为不可信输入。

必须注意：

1. `.chara` 文件大小、格式和解析异常限制。
2. 外部链接只允许白名单域名和明确支持的路径。
3. 图标代理和模板预览资源不能变成开放代理。
4. 后台浏览器登录态、token、cookie 和调试端口不能暴露给公开用户。
5. 错误信息不输出服务器内部路径、密钥或堆栈。

## 验证清单

实现或迁移后至少验证：

- `npx vue-tsc --noEmit`
- `npm run build`
- 浏览器访问 `http://localhost:5173/#/ffxiv/glamour`
- `/api/glamour/health` 返回正常。
- `.chara` 导入得到稳定装备列表。
- 石之家链接导入和 EC 链接导入能导入标题、角色名、服务器、装备和染剂。
- 成段文字识别支持已确认格式，并能切换语言。
- 导入后先做装备合法性检查，再进入编辑栏和模板。
- 模板预览、图片裁剪、PNG 导出尺寸与旧项目关键样本对齐。
- 多语言装备名和染剂名在 `zh/en/ja/fr/de` 等已支持数据内可切换。
- 移动端至少保持导入、编辑、导出主流程可用。

## 待确认事项

- 新后端技术栈：继续 Flask、改 FastAPI，还是统一到 Node 服务。
- 新后端是否与 Plate 合并为一个服务，还是保持模块独立服务。
- 旧用户草稿和模板图片是否需要自动迁移。
- 第一阶段先重写 `/equipinfo`，还是先重写 `/template`。
- 生产环境石之家后台浏览器功能是否继续开放，以及开放给谁。
