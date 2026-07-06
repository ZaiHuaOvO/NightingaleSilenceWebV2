# NSGlamour 模块计划

## 当前状态

- 模块状态：第一段 equipinfo 工作台已接入，真实业务迁移已开始。
- 目标路由：`#/ffxiv/glamour`。
- 当前页面入口：`src/pages/glamour/NSGlamourPage.vue`。
- 当前页面外壳：复用 `FfxivToolShell` 的 `workspace` 模式；正式/迁移工具入口不展示内部标题栏或 API 状态块，顶部菜单承担站点和工具导航，顶部菜单下方直接进入全屏工具工作台。
- 来源项目路径：`NSGlamour`。
- 当前 API 边界：V2 使用 `/api/glamour/*`，开发代理 rewrite 到旧服务 `/api/*`。
- 当前连通性检查：`/api/glamour/health`，对应旧服务 `/api/health`。
- 当前旧服务端口：`8765`。
- 旧项目当前主工作流是 `/template` 和 `/equipinfo`；旧 `/card` 不是本轮 V2 迁移的第一目标。

当前第一段已接入：

- `src/lib/glamour/`：共享类型、固定部位、装备归一、染剂摘要、`GlamourDraft` 和基础复制文案。
- `src/pages/glamour/composables/useGlamourDraft.ts`：页面内共享草稿状态，并同步写入旧模板可读的 `nsglamour.cardDraft.v2` 与 `nsglamour.store.equipment`；后续 `/template` 消费同一份草稿。
- `src/pages/glamour/services/nsglamourApi.ts`：通过 `/api/glamour` 调用旧后端 `import-glamour-link`、`parse-text` 和 `parse-chara`。
- `src/pages/glamour/components/`：导入面板、装备面板、复制面板和工作台布局；装备面板已接入旧 `/equipinfo` 的空槽装备名搜索、搜索结果选择、删除当前装备、匹配项切换和行内染剂 chip 编辑能力。
- 装备信息结果头部已按旧 `/equipinfo` 可见行为显示来源名，并提供“保存配置”入口；保存后仍写入旧键 `nsglamour.recentLoadouts`，与最近载入弹窗共享同一套记录。
- 最近记录已按旧 `/equipinfo` 行为接入：沿用 `nsglamour.recentLoadouts`，最多 10 条，入口是导入面板右上角的图标按钮，点击后弹出最近记录面板；支持保存配置、恢复、删除单条和清空，并在恢复后继续同步 `GlamourDraft`、`nsglamour.cardDraft.v2` 和 `nsglamour.store.equipment`。
- 文案生成已接入旧 `/equipinfo` 可见的格式一、格式二、格式三、格式四和第一版自定义模板输入；V2 不显示旧项目 token 参考里涉及模型码、候选数量、原始 JSON 等内部字段。

当前 UI 约束：

- V2 迁移目标是更换前端架构，不是重新设计用户可见功能。最终界面必须以旧项目同一工作流的用户可见界面为准做等价复现；旧项目没显示的字段、统计、调试信息或解析中间态，不能因为 V2 数据层可获得就新增到界面上。
- 新增任何用户可见字段、按钮、状态、文案或编辑能力前，必须先确认旧项目对应工作流确实已有等价展示，或得到用户明确要求。
- 本地角色配置导入是隐式能力，不在用户界面展示专门按钮、格式名或明显提示文案。
- 候选数量、模型码等解析辅助信息只保留在内部数据层，不进入正式用户界面；除非用户明确要求开启调试/编辑功能。
- 装备栏顺序是用户亲自定过的旧 `/equipinfo` 视觉契约，不能用通用槽位数组自动排版替代。桌面端左列固定为：主手、头部、身体、手臂、腿部、脚部、面部配饰；右列固定为：副手、耳部、颈部、腕部、左指、右指、时尚配饰。手机端单列和后续 `/template` 装备顺序固定为：主手、副手、头部、身体、手臂、腿部、脚部、耳部、颈部、腕部、左指、右指、面部配饰、时尚配饰。
- “无染色”属于装备/染剂数据文案，必须跟随装备草稿语言和导入 payload 的 `no_dye_labels`，不能跟随全站 UI 语言；`/equipinfo` 和后续 `/template` 都应复用同一装备层 helper。

当前未接入：

- 旧 localStorage 迁移提示。
- `/template` Canvas、图片裁剪、模板资源懒加载和 PNG 导出。

## 阶段 0 当前结论

本轮从 0 开始的阶段 0 只建立契约和回归地基，不迁移 UI、不复制旧资源、不修改旧 `NSGlamour` 项目。

已确认的轻量接口：

- `GET /api/health`：连通性。
- `GET /api/stains?locale=zh|en`：染剂数据，当前 `zh/en` 均为 `126` 条，缓存 1 小时。
- `GET /api/ui-localization`：旧 UI 本地化数据，当前 `version=2`，响应约 `102 KB` raw，`no-store`。
- `GET /api/search-items`：装备搜索，支持 `slot`、`q`、`locale`、`limit`，limit 后端限制为 `1..80`。
- `POST /api/import-glamour-link`：石之家 / Eorzea Collection 链接导入。
- `POST /api/equipinfo/parse-text`：成段文字识别，空文本返回 400；可用搜索结果构造最小结构回归。

已确认的资产/流量约束：

- `data/item_model_mapping.json` 约 `47.7 MB` raw / `4.36 MB` gzip，`28,935` 条 items；不能进入 V2 前端 bundle。
- 旧 `font/` 约 `81.6 MB`；不能整目录迁入，后续只能按模板/语言懒加载。
- 旧 `static/templates/` 约 `4.5 MB`；只迁运行时模板资源，按选中模板加载。
- 旧 `templates/` 下源/参考资源约 `327.6 MB`；不能进入 V2 构建产物。
- `/template` HTML 本身很轻，但真实首屏依赖 Cropper、字体、模板背景和多份渲染脚本；`/equipinfo` 更适合作为第一段轻量迁移切片。

当前已新增 V2 契约检查脚本：

```bash
npm run check:nsglamour-contract
```

默认直连旧服务 `http://127.0.0.1:8765/api`。如需验证 V2 代理，启动 Vite 后设置：

```bash
NSGLAMOUR_CONTRACT_BASE_URL=http://127.0.0.1:5173/api/glamour npm run check:nsglamour-contract
```

当前本机已有用户提供的私有本地配置 fixture，位于 ignored 的 `tests/fixtures/nsglamour/chara/`，只用于本机回归，不公开、不提交。

阶段 1 推荐先从 `/equipinfo` 的导入/编辑结构开始，再进入 `/template` Canvas。原因是 `/equipinfo` 依赖资源更轻，能先固定装备、染剂、候选和文案结构；`/template` 对字体、图片裁剪、Canvas 坐标、背景资源和视觉回归更敏感。

## 定位

`#/ffxiv/glamour` 是 FFXIV 幻化工具模块，目标是承接旧 `NSGlamour` 的核心能力，但不是把旧项目代码机械搬进 V2。

新实现应优先按 V2 的 Vue、模块文档、公共组件、公共 CSS、API 边界和安全规则重新整理。旧项目在迁移期间作为行为契约、数据契约和回归样本保留。

核心目标：

1. 本地角色配置解析和装备/染剂映射严谨可靠。
2. 石之家与 Eorzea Collection 导入结果与旧项目一致或更稳。
3. `/template` 模板生成和 `/equipinfo` 装备信息工作流在 V2 中重新组织。
4. 模板 Canvas 渲染、图片裁剪、PNG 导出尽量保持旧项目已校准的视觉结果。
5. 多语言装备、染剂和 UI 本地化能力保留，并为后续扩展留出边界。

## 后端重写策略

本模块允许按新规则重写后端，但必须先抽取旧后端行为契约。

推荐顺序：

1. 记录旧 API 输入、输出、错误结构和关键字段。
2. 用真实样本建立回归用例，包括本地角色配置、石之家链接、EC 链接、成段文字和模板草稿。
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

| 旧接口                                      | V2 目标路径                                         | 用途                     |
| ------------------------------------------- | --------------------------------------------------- | ------------------------ |
| `GET /api/health`                           | `GET /api/glamour/health`                           | 连通性检查               |
| `GET /api/ui-localization`                  | `GET /api/glamour/ui-localization`                  | UI 本地化数据            |
| `POST /api/import-glamour-link`             | `POST /api/glamour/import-glamour-link`             | 石之家/EC 链接导入       |
| `POST /api/equipinfo/parse-text`            | `POST /api/glamour/equipinfo/parse-text`            | 成段文字识别             |
| `POST /api/risingstones-browser/open-login` | `POST /api/glamour/risingstones-browser/open-login` | 石之家后台浏览器登录辅助 |
| `GET /api/icon/<icon_id>`                   | `GET /api/glamour/icon/<icon_id>`                   | 图标代理                 |
| `GET /api/stains`                           | `GET /api/glamour/stains`                           | 染剂数据                 |
| `GET /api/search-items`                     | `GET /api/glamour/search-items`                     | 装备搜索                 |
| `POST /api/parse-chara`                     | `POST /api/glamour/parse-chara`                     | 本地角色配置解析         |

旧静态/资源入口：

| 旧入口                         | 迁移注意                                             |
| ------------------------------ | ---------------------------------------------------- |
| `/font/<path>`                 | 字体体积和授权要单独确认                             |
| `/template-preview/<path>`     | 模板预览资源后续可考虑迁入 V2 静态资源或后端资源服务 |
| `data/item_model_mapping.json` | 装备、染剂、图标、模型码核心映射，不手写临时替代     |
| `data/ui-localization.json`    | UI 文案来源之一，未来应进入 V2 本地化规划            |

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

## 共享装备草稿决策

V2 中 `/equipinfo` 和 `/template` 的装备信息必须共用同一层规范化数据，不再让两个页面各自维护装备状态。

已确认边界：

1. `GlamourDraft` 是 V2 前端唯一的幻化草稿模型，负责装备、染剂、候选、部位顺序、语言、来源信息和导入警告。
2. `equipinfo` 只负责导入、识别、编辑和替换这个草稿，不拥有另一份页面私有装备状态。
3. `template` 只消费同一份 `GlamourDraft` 渲染模板；模板选择、裁切图片、标题、模板局部开关等属于模板私有设置，不写回装备层。
4. 旧项目的 `NSGlamourStore.equipment`、`equipmentSync` 和 `nsglamour.cardDraft.v2` 说明了“装备层”和“模板层”的边界，但 V2 不直接复制全局 JS；应迁入 `src/lib/glamour/`、service 和 composable。
5. 后续候选替换、染剂编辑和模板 Canvas 都必须围绕同一份 `GlamourDraft` 增量实现，避免 equipinfo 与 template 的装备/染剂不同步。

## 前端状态和存储参考

旧项目重要状态键：

| 键/机制                                  | 用途                 |
| ---------------------------------------- | -------------------- |
| `nsglamour.theme`                        | 旧项目主题           |
| `nsglamour.cardDraft.v2`                 | 模板/装备草稿        |
| `nsglamour.recentLoadouts`               | 最近导入记录         |
| `nsglamour.copyTemplate`                 | 自定义复制模板       |
| `nsglamour.copyFormat`                   | 文案格式             |
| `nsglamour.ignoreEmperor`                | 皇帝套过滤设置       |
| `nsglamour.templateWorkspaceSettings`    | 模板工作台设置       |
| `NSGlamourStore`                         | 旧项目跨页面共享状态 |
| `nsglamour-template-images-v2` IndexedDB | 模板上传图片持久化   |

V2 不要求复用这些键名，但迁移时必须处理旧数据兼容或迁移提示，不能无声丢失用户草稿。

## 页面拆分方向

建议 V2 前端拆分：

```text
src/pages/glamour/
├── NSGlamourPage.vue
├── components/
│   ├── NSGlamourImportPanel.vue
│   ├── NSGlamourEquipmentEditor.vue
│   ├── NSGlamourTemplateWorkspace.vue
│   ├── NSGlamourEquipInfoWorkspace.vue
│   ├── NSGlamourLanguageSwitcher.vue
│   └── NSGlamourExportPanel.vue
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
- 如果 NSGlamour 与 NSPlate 都需要同类工具页外壳，应沉淀到 `src/pages/ffxiv/components/` 或公共组件。

## 安全边界

所有上传、粘贴、外部链接和后台浏览器相关数据都视为不可信输入。

必须注意：

1. 本地角色配置文件大小、格式和解析异常限制。
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
- 隐式本地配置导入得到稳定装备列表。
- 石之家链接导入和 EC 链接导入能导入标题、角色名、服务器、装备和染剂。
- 成段文字识别支持已确认格式，并能切换语言。
- 导入后先做装备合法性检查，再进入编辑栏和模板。
- 模板预览、图片裁剪、PNG 导出尺寸与旧项目关键样本对齐。
- 多语言装备名和染剂名在 `zh/en/ja/fr/de` 等已支持数据内可切换。
- 移动端至少保持导入、编辑、导出主流程可用。

## 待确认事项

- 新后端技术栈：继续 Flask、改 FastAPI，还是统一到 Node 服务。
- 新后端是否与 NSPlate 合并为一个服务，还是保持模块独立服务。
- 旧用户草稿和模板图片是否需要自动迁移。
- 是否确认阶段 1 按 `/equipinfo` 导入/编辑结构先行，再推进 `/template` Canvas 的顺序执行。
- 生产环境石之家后台浏览器功能是否继续开放，以及开放给谁。
