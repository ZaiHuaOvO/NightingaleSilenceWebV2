# NSPlate 回归矩阵

## 定位

本文件固定 `#/ffxiv/plate` 的收口验收样本，避免后续迁移 `NSGlamour` 或继续调整 NSPlate 时把已验证行为改坏。

NSPlate 的默认生产路径必须是：

- 预设：`/data/plate/presets.json`
- 素材 manifest：`/data/plate/files.json`
- 图片：manifest 中的 COS/CDN base

旧 `/api/plate`、`/img`、`/img-preview` 只允许作为显式 legacy/dev fallback、manifest 生成源和旧项目对照来源。默认页面加载不得请求旧 `/api/plate/presets` 或 `/api/plate/files`。

## 自动回归命令

```bash
npm run check:plate-regression
```

该命令会：

1. 使用静态 manifest 模式启动 Vite。
2. 用本机 Playwright + Chrome 打开 `#/ffxiv/plate`。
3. 验证默认页面只读取静态 manifest，不读取旧 catalog API。
4. 注入 V2 草稿样本，覆盖左右肖像、自定义图、出框图、信息预设、日夜模式和移动端基础布局。
5. 导入旧 JSON 和旧 localStorage 样本，确认能恢复为 V2 草稿。
6. 下载配置 JSON、PNG、JPG 和分层 ZIP，检查文件名、非空和 ZIP 基础条目。

自动脚本只生成临时 data URL 自定义图，不提交图片资产，也不把下载文件写入仓库。

## 固定样本矩阵

| 样本 | 覆盖项 | 验收方式 |
| ---- | ------ | -------- |
| `static-default` | 干净 localStorage、默认信息预设、静态 manifest、默认主题 | 自动 |
| `v2-left-standard` | 左侧肖像、肖像/铭牌素材选择、标准自定义图、`国服` 信息预设 | 自动 |
| `v2-right-popout-night-mobile` | 右侧肖像、出框自定义图、出框锚点、夜间模式、英文 UI、移动端无横向溢出 | 自动 |
| `legacy-json-import` | 旧完整 JSON、旧素材选择、旧自定义图、旧信息层、旧 active panel | 自动 |
| `legacy-local-storage` | 无 V2 草稿时读取旧 `iconComposer.ui.config.v1` | 自动 |
| `v2-config-roundtrip` | V2 配置 JSON 下载后重新导入 | 自动 |
| `exports` | PNG、JPG、分层 ZIP 文件名和基础结构 | 自动 |
| `info-visual-phantom-tide` | `幻海流` 信息层逐项旧项目视觉对照 | 人工/像素对比 |
| `info-visual-china` | `国服` 信息层逐项旧项目视觉对照 | 人工/像素对比 |
| `info-visual-international` | `国际服` 信息层逐项旧项目视觉对照 | 人工/像素对比 |
| `full-locale-smoke` | 中/日/韩/英固定 UI 文案、按钮、placeholder、aria/title | 自动 + 人工抽查 |

## 信息层保护规则

信息层是 NSPlate 当前最高风险区域。后续修改必须遵守：

1. 信息层坐标、尺寸、行高、缩放、透明度和左右肖像位置继续以旧 `INFO_PRESET_DEFINITIONS` 为唯一来源。
2. 不得用肉眼微调替代旧项目坐标对照。
3. `small-caps`、OpenType feature、字体 fallback 和本机字体映射不得顺手调整；要先新增或更新视觉回归样本。
4. `src/pages/plate/nsplate-fonts.css` 当前只允许为已确认需要像素级对齐的 `Miedinger` 等旧项目信息层字体接入明确来源；其他旧字体不要顺手打包。公开分发前仍需确认授权和 license。
5. 旧文字参数编辑暂不开放到 UI。旧 JSON 中未迁移的字体、描边、透明度、坐标和高级参数只作为兼容风险记录，不直接污染 V2 草稿。

## 旧兼容字段策略

当前明确迁移：

- V2 JSON 导入/导出。
- 旧完整 JSON。
- 旧 `IC1?` 紧凑参数。
- 无 V2 草稿时自动读取旧 `iconComposer.ui.config.v1`。
- 已支持的信息层字段：启用状态、文字内容、图标素材、多选活动图标、部队队徽三类素材/颜色和作息条状态。

当前明确不迁移到 V2 状态：

- 旧主题。
- 旧视口缩放和平移字段。
- 旧服务端导出配置。
- 旧 Photoshop/JSX/PSD 导出计划。
- 旧信息层未开放参数，例如用户自定义坐标、字号、描边、透明度等高级编辑字段。

如果后续要恢复这些字段，必须单独开切片，并补回归样本；不能在现有 adapter 里静默写入 V2 草稿。

## 固定 UI 文案审计

固定 UI 文案必须走 `src/locales/ui.ts` key，包括：

- 右侧预设搜索、素材搜索、清空搜索。
- 素材卡、素材系列、状态图标 aria/title。
- 信息层字段标题、启用/隐藏、作息格 aria/title。
- 导入、粘贴、复制、导出、下载和错误提示。
- 清空素材、移除图片和二次确认提示。

`npm run check:i18n` 是基础自动检查；新增 NSPlate 固定文案时仍需人工确认语义是否真的来自游戏术语、旧布局字段或用户确认文案。

## 通过标准

NSPlate 准备进入 NSGlamour 迁移前，至少需要满足：

1. `npm run typecheck` 通过。
2. `npm run check:i18n` 通过。
3. `npm run check:plate-static` 通过。
4. `npm run check:plate-static:preview` 通过。
5. `npm run check:plate-regression` 通过。
6. `docs/ai/MODULES/nsplate.md` 的剩余计划只保留明确后续项，不再含糊写“可能还没迁完”。
