# Silence 角色档案模块计划

## 当前状态

- 模块状态：`#/silence` 入口页和 `#/silence/angel`、`#/silence/glitch` 分组占位页已接入代码、路由和公开导航；单角色详情页和角色数据尚未接入。
- 计划入口路由：`#/silence`。
- 推荐分组路由：`#/silence/angel`、`#/silence/glitch`。
- 推荐详情路由：`#/silence/angel/:characterId`、`#/silence/glitch/:characterId`。
- 已接入页面入口：`src/pages/silence/SilenceIndexPage.vue`、`src/pages/silence/SilenceGroupPage.vue`。
- 计划单角色页面入口：`src/pages/silence/SilenceCharacterPage.vue`。
- 模块类型：创作信息分类 / 原创角色档案 / 角色图鉴。
- 当前不涉及后端 API。
- 当前正式展示文案尚未确认；页面骨架中未由用户提供的公开文案统一使用 `占位用，待编辑`。

## 定位

`#/silence` 是夜莺不语站点中“创作信息”的分类入口之一，用于承载原创角色档案、作品或世界观分组、角色关系、设定资料和创作记录。公开路由不使用 `#/oc`；`OC` 只作为内部内容类型和讨论术语存在。

该模块不属于 `FFXIV` 工具分类，也不依赖游戏客户端数据解构。它应作为整站可扩展分类存在，与 `#/ffxiv` 并列，而不是放在 `#/about` 里作为个人介绍的一部分。

## 内容分层规则

Silence 模块第一层按角色所属叙事层分组，不按世界观分组。主 OC 可以跨越西幻、赛博、现实、未来等不同世界观，因此世界观应作为角色内部的世界线或版本信息，而不是总览页的主分类。

- `angel`：六个主 OC，公开分组名为 `不语·silence`。这是作品 / 档案层，重点呈现角色本体、世界线、关系和创作资料。
- `glitch`：两个赛博幽灵 / 皮套角色，公开分组名为 `幽灵·silence`。这是站点 meta 层，重点呈现站点外壳、界面幽灵、像素、故障和入口感。

像素 / 赛博幽灵风格属于站点 meta 层，主要服务首页与 `glitch` 角色；`angel` 组的六个主 OC 使用独立档案视觉，避免被全站像素风同化。

`angel` 命名用于表达这六个角色的重要性，不引入宗教、圣所、祭坛或崇拜语义。公开表现应偏向“横向群像主视觉 / 全员同屏 / 立绘舞台”，而不是神圣化叙事。

路由层级建议：

```text
#/                  Home 纯视觉首页
#/ffxiv             FFXIV 工具分类
#/silence           Silence 创作入口
#/silence/angel     不语·silence，六个主 OC
#/silence/glitch    幽灵·silence，两个 meta 角色
#/silence/angel/:characterId   单个主 OC 档案
#/silence/glitch/:characterId  单个幽灵角色档案
#/about             关于页面
```

## 参考方向

本模块可以参考日漫官网角色介绍页的信息架构，但只参考结构和体验，不复刻具体美术、素材、文案或品牌表达。

- `https://needygirl-anime.com/cn/`：适合参考“角色列表 + 大立绘 + Profile 信息 + 角色切换”的浏览体验。若第一阶段角色数量较少，可以做成一页里快速切换角色的形式。
- `https://frieren-anime.jp/character/`：适合参考“按阵营、种族、作品阶段或世界观分组”的角色索引。若后续角色数量较多，分组索引比单纯长列表更稳定。
- `https://bocchi.rocks/omnibus/character/`：适合参考“角色身份、团队职能或能力定位标签”的表达方式。若 OC 有职业、阵营、能力、作品定位，可以把这些作为卡片首屏信息。

## 页面形态建议

### Silence 入口页和分组页

`#/silence` 负责 `不语·silence` 与 `幽灵·silence` 两组总览，不承担单个角色的完整长档案。`#/silence/angel` 与 `#/silence/glitch` 负责组内角色列表和组内视觉气质。

当前已接入的 `#/silence` 第一版采用“双入口门厅”结构，不直接展示八个角色，避免把 `angel` 与 `glitch` 两组风格硬拼在同一屏。`angel` 用抽象六人剪影暗示六个主 OC，`glitch` 用像素窗口和故障视觉暗示两个 meta 角色。

建议结构：

```text
SilenceIndexPage / SilenceGroupPage
├── 分类视觉区：Silence / 角色档案入口
├── 角色分组或筛选：angel / glitch、状态、标签
├── 角色卡片网格
│   ├── 头像或半身图
│   ├── 角色名和别名
│   ├── 所属作品 / 阵营 / 关键词
│   └── 进入档案按钮
└── 最近更新或待补全状态
```

首页第一屏应直接出现角色图像或明确角色视觉信号，不要做成纯文字目录。若第一阶段还没有正式角色图，页面实现时只能使用占位状态，不能由 AI 自行编写正式角色介绍。

`#/silence/angel` 的首屏建议采用横向群像主视觉：

```text
SilenceAngelGroupView
├── 全屏或超宽横幅舞台
├── 六个主 OC 立绘同屏出现，允许遮挡、前后错位和不同层级深度
├── 大色块 / 图形切割 / 细线分隔作为背景，不使用首页像素窗口舞台
├── 推荐使用隐藏式角色导航：默认弱化或隐藏，用户 hover、移动指针、点击或聚焦时浮出
├── 选中角色时，当前角色前移或提高亮度，其他角色退到后景
└── 进入档案按钮，跳转到 `#/silence/angel/:characterId`
```

该页的重点是“六人全员同屏的存在感”，不是卡片列表、资料柜或宗教化空间。可参考“横向群像 + 隐藏式导航 + 边缘少量 UI”的结构，但不得复刻外部作品的美术、角色、品牌符号、文案或设定体系。

### 单角色档案页

`#/silence/angel/:characterId` 与 `#/silence/glitch/:characterId` 负责完整角色资料。

建议结构：

```text
SilenceCharacterPage
├── 角色主视觉：立绘、角色名、别名、代表色
├── 基础档案：生日、身高、年龄、身份、阵营、关键词
├── 性格与印象：短设定或对外简介
├── 图像资料：表情差分、服装差分、旧设 / 新设、色卡
├── 关系网：与其他 OC 的关系卡片
├── 创作笔记：设计来源、版本记录、废案记录
└── 剧透区：默认折叠或明确标记
```

角色详情页可以保留页内锚点，例如 `基础档案`、`图像资料`、`关系网`，但角色身份本身优先使用 `#/silence/angel/:characterId` 或 `#/silence/glitch/:characterId` 表达。当前项目使用 hash router，如果再用 `#/silence#character-name` 作为角色主入口，会让 URL 层级不够清楚。

## 数据来源和字段建议

第一阶段建议使用本地结构化数据，不接后端：

```text
src/data/silence/characters.ts
```

如果实现时项目尚未建立 `src/data/`，需要在计划中确认目录归属；不要把大量角色数据直接写进页面组件。

角色数据建议包含：

```ts
type SilenceGroupId = 'angel' | 'glitch'

const silenceGroups: Record<SilenceGroupId, { label: string; path: string }> = {
  angel: { label: '不语·silence', path: '/silence/angel' },
  glitch: { label: '幽灵·silence', path: '/silence/glitch' },
}

interface SilenceCharacter {
  id: string
  name: string
  aliases: string[]
  groupId: SilenceGroupId
  worlds?: Array<{
    id: string
    label: string
    summary?: string
  }>
  color?: string
  tags: string[]
  profile: {
    birthday?: string
    height?: string
    age?: string
    identity?: string
    affiliation?: string
  }
  images: {
    portrait?: string
    fullBody?: string
    gallery?: string[]
  }
  relationships?: Array<{
    characterId: string
    label: string
  }>
  spoilerLevel?: 'none' | 'light' | 'major'
  visibility?: 'public' | 'draft' | 'private'
}
```

字段只是建议结构，不是已确认实现。公开页面实现时，`private` 内容不得渲染到构建产物里；如果只是 CSS 隐藏或前端折叠，仍然视为已公开。

`worlds` 主要服务 `angel` 组的主 OC，用于表达同一角色在不同世界观中的版本或经历；`glitch` 组不需要强行套入多世界线结构。

## 组件拆分计划

建议页面结构：

```text
src/pages/silence/
├── SilenceIndexPage.vue
├── SilenceGroupPage.vue
├── SilenceCharacterPage.vue
└── components/
    ├── SilenceCharacterCard.vue
    ├── SilenceCharacterGrid.vue
    ├── SilenceCharacterFilters.vue
    ├── SilenceProfilePanel.vue
    ├── SilenceGallery.vue
    ├── SilenceRelationshipList.vue
    └── SilenceSpoilerBlock.vue
```

如果第一版角色数量很少，可以先少拆组件，但角色卡片、档案面板和剧透块很可能会复用，适合尽早保持边界清楚。

## 模板和数据复用关系

Silence 模块不建议为每个角色单独创建一个页面组件，例如不要做成 `SalvancePage.vue`、`AnotherCharacterPage.vue` 这种一人一个页面。推荐结构是“一个详情页模板 + 一份角色数据”：

- `src/data/silence/characters.ts`：维护角色数据。新增角色、修改角色名、生日、标签、图片路径、简介等内容，优先改这里。
- `src/pages/silence/SilenceIndexPage.vue`：维护 `#/silence` 入口页的整体排布。修改这里会影响两组总览，但不直接改变单角色详情页模板。
- `src/pages/silence/SilenceGroupPage.vue`：维护 `#/silence/angel` 与 `#/silence/glitch` 的组内列表模板。修改这里会影响两个分组页的整体结构。
- `src/pages/silence/SilenceCharacterPage.vue`：维护 `#/silence/angel/:characterId` 与 `#/silence/glitch/:characterId` 的详情页模板。`#/silence/angel/salvance` 和其他角色详情页都走这个组件；修改这里会影响所有角色详情页的整体结构。
- `src/pages/silence/components/SilenceCharacterCard.vue`：维护角色列表里的单张角色卡片。修改这里会影响分组页中所有角色卡片的外观和交互。
- `src/pages/silence/components/SilenceProfilePanel.vue`、`SilenceGallery.vue`、`SilenceRelationshipList.vue`、`SilenceSpoilerBlock.vue`：维护详情页内部可复用区块。修改其中一个组件，会影响所有使用该区块的角色详情页。

实现时，详情页通过路由参数读取分组和角色 ID。例如访问 `#/silence/angel/salvance` 时，`route.params.groupId` 应为 `angel`，`route.params.characterId` 应为 `salvance`，页面再用这些参数从 `characters.ts` 中找到对应角色数据。

## CSS 和组件复用策略

遵守 `ARCHITECTURE_PLAN.md` 的“样式变更分级规则”：

- 样式层级：模块/分类级 + 页面专属。
- 全站按钮、面板、链接、焦点态优先使用公共组件和公共 CSS。
- Silence 分类可以拥有自己的角色视觉、代表色、档案纸、标签和立绘排版，但不要改公共组件默认外观来满足单页风格。
- 角色代表色应作为局部 CSS 变量或内联 style 输入，不要扩展成新的全站主题。
- 如果角色卡片未来也用于博客、作品集或其他内容分类，再考虑抽成公共内容卡片组件。
- 不把首页的粉蓝像素舞台直接复制到 Silence 页面；Silence 模块可以与首页有气质呼应，但应形成“角色档案”自己的阅读秩序。
- `glitch` 组可以继承首页的像素、故障、窗口碎片和赛博幽灵表现；`angel` 组只保留全站导航、按钮、焦点态等基础外壳，不调用首页像素舞台作为主视觉。
- `angel` 组优先使用大幅立绘、群像合成、横向舞台、隐藏式导航和局部高亮来表达角色重要性；导航默认不应抢占群像画面，避免使用圣所、祭坛、光环、祈祷等宗教化视觉语汇。

## 公开内容和隐私边界

- 角色名、作品名、阵营名、正式简介等公开文案必须由用户提供或确认。
- AI 生成的临时展示文案只能使用 `占位用，待编辑`。
- 剧透内容需要明确字段或组件隔离，避免默认首屏暴露。
- 未打算公开的草稿、私设、时间线、关系设定，不应进入前端构建数据。
- 公开图片需要注意文件名、EXIF、隐藏水印、未公开草稿层和不该暴露的路径信息。
- 外部参考链接可以记录在内部文档中，但不要在公开页面中暗示 OC 来源于某个现有商业 IP。

## 实施边界

后续继续实现单角色详情页、角色数据或正式素材前，需要先进入计划阶段，至少确认：

1. 角色数量和第一批角色名单。
2. 是否需要单角色详情页，还是第一版只做分类首页和页内切换。
3. 角色图资产路径和公开范围。
4. 正式展示文案是否已经提供。
5. `src/data/silence/`、`src/pages/silence/`、`src/config/site.ts`、`src/router/index.ts` 的具体改动范围。
6. 是否把 `Silence` 写入首页导航和 `AppTopNav` 分类菜单。

第一版不建议引入新依赖；角色筛选、切换和详情页都可以用 Vue、CSS 和本地数据完成。

## 验证清单

实现后至少验证：

- `npx vue-tsc --noEmit`
- `npm run build`
- 浏览器访问 `http://localhost:5173/#/silence`
- 浏览器访问 `http://localhost:5173/#/silence/angel` 和 `http://localhost:5173/#/silence/glitch`
- 若实现详情页，访问 `http://localhost:5173/#/silence/angel/<characterId>` 和 `http://localhost:5173/#/silence/glitch/<characterId>`
- 560px / 900px / 1120px+ 宽度下角色卡片和详情页布局稳定。
- 角色卡片可键盘聚焦，详情入口可回车进入。
- 剧透区默认状态符合预期，不在首屏意外暴露。
- 草稿和私有角色数据未进入公开渲染。
- 控制台无错误。

## 待确认事项

- `Silence` 公开导航文案：使用 `Silence`、`不语`、`角色档案` 或其他命名。
- 第一版角色数量、角色 ID 命名规则和排序方式。
- `angel` / `glitch` 组内的角色排序方式，以及 `angel` 角色内部世界线的展示方式。
- 是否需要中英/多语言字段。
- 单角色页是否公开创作笔记和版本记录。
- 图片资产优先放在构建内资产目录，还是放在 public 静态目录。
