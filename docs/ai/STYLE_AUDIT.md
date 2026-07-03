# 全站样式审计记录

## 定位

本文件记录 V2 公共样式、正式像素风 token、顶栏弹窗和 Style Lab 的长期边界。它不是单次视觉稿，也不替代 `ARCHITECTURE_PLAN.md`；具体长期规则仍以架构文档为准。

## 当前结论

1. 正式站点方向已经固定为统一像素风，`day / night` 只切换同一套像素风变量。
2. `src/styles/theme.css` 是正式 token 入口；正式页优先使用 `--ns-color-*`、`--ns-pixel-*`、`--ns-status-*`、`--ns-top-nav-*` 等变量。
3. `src/styles/experiments/pixel-soft.css` 只属于 `#/style-lab` 隐藏实验区，不进入 `src/styles/index.css`。
4. `AppButton`、`AppField`、`AppToolbar`、`AppTabs`、`AppStatus`、`AppPixelWindow` 和 `AppTopNav` 是当前公共控件基准。
5. 工房工作台变量暂不继续扩展，等第一个项目迁移完成后再按真实重复结构评估。

## 本轮审计处理

- 将公共组件里的常见裸色收敛到正式 token：强调文字、hover 边框、表单内阴影、状态提示色和顶栏弹窗尺寸。
- `AppTopNav` 的菜单与设置弹窗保持并列入口：`菜单 | MENU` 管站点导航，`设置 | CONFIG` 管显示模式和语言。
- 顶栏弹窗路由切换后自动收起，避免用户进入新页面时遗留打开状态。
- Style Lab 新增正式公共组件基准区，放在 `pixel-soft` 实验区外，避免实验 CSS 变量污染正式控件判断。

## 后续审计重点

1. 迁移 `NSPlate` 第一个真实业务切片后，对照本文件和 `WORKBENCH_STYLE_CONTRACT.md` 检查控件是否统一。
2. 如果某个页面专属样式第三次复用，再考虑上移为公共 class 或 token。
3. 若需要替换图标，优先让用户从已确认的 iconfont 像素图标合集挑选，再统一接入资产目录。
4. 首页人物舞台和浮动装饰不得进入公共组件默认样式。
