# Hermes Console：终端行尾裁字

**症状**：终端里每行行尾 1-2 个汉字、或 2-3 个西文字符被切掉。窗口宽度变化后切掉的字数会变，但不会消失。

**插件**：Hermes Console 1.1.16（Obsidian 社区插件 `hermes-console`）
**环境实测**：Obsidian 1.13.7 / macOS（Apple Silicon）

## 根因

不是侧栏遮挡、不是列数算少、不是 fit 时机、也不是字体宽度。

xterm 的 DomRenderer 在渲染每一行时，给行 `div` 同时写了两个行内样式：

- `style.width = canvas.width`（= cols × 单字宽）
- `style.overflow = "hidden"`

即**行盒自己按 cols 定宽，又自己把超出这个宽度的内容裁掉**。行尾那一两个字的墨迹正好落在行盒宽度之外约 2 个 cell，被行盒吃掉。

推论（也实测过）：任何「让终端重新计算列数」的思路都修不了——手动 `resize(cols-1)` 再 `resize(cols)` 强制重排、`fitActive()`、改 padding、改滚动条宽度、换字体栈、重启 Obsidian、onOpen 后多次延迟 re-fit，全部无效。裁字的是行盒本身。

## 修法

在 `.obsidian/plugins/hermes-console/styles.css` 末尾追加：

```css
.terminal-session .xterm-rows > div {
  overflow: visible !important;
}
```

多出来的墨迹落进 `.xterm` 已有的右内边距（本插件给终端留了 26px），不会串到别的面板。

改完必须重载：`Ctrl+P` → `Reload app without saving`。插件 CSS 是加载时注入的，Obsidian 不监听插件目录，盘上改了运行中不生效。

自检：

```bash
grep -n "xterm-rows > div" .obsidian/plugins/hermes-console/styles.css
```

## 试过但无效的方案

| 方案 | 结果 |
|---|---|
| `.xterm` 右 padding 16 → 26px | 无效 |
| 滚动条 8px → 4px（`!important`） | 无效 |
| 改字体栈（加中文字体） | 无效 |
| `onOpen` 后 300 / 800 / 1600 / 3000ms 延迟 re-fit | 无效 |
| 完整重启 Obsidian 强制全量重 fit | 无效 |
| 手动 `terminal.resize(68,43)` → `resize(69,43)` 强制重排 | 无效 |

保留这份负结果，是为了避免后来者（包括自己）再走一遍。

## 给上游的建议

如果认可这个方向，把规则并入插件自己的 `styles.css`，或渲染时不给行 `div` 设 `overflow: hidden`，用户侧补丁就不会在升级时被覆盖。

## 证据与未验证项

- 根因有源码证据：`main.js` 内 xterm 的 DomRenderer 对每行写 `width = canvas.width` 与 `overflow = "hidden"`，行容器类名 `xterm-rows`，容器 `.terminal-session` 在插件代码中确实存在。
- 行尾溢出量有一组对照实测：约 16.5px ≈ 1.95 个字宽，与「1-2 个汉字」的症状吻合。
- **未复核**：另一组 CDP 几何数据（`cols=69 / rows=43` / 单字宽 `8.426px`）与分条带像素比对（差异像素 591 / 474）在归档时没有留下可重跑的探针与截图，只作为方向性参考，不作结论。
- 本补丁落地后，最终的肉眼观感以重载后实际界面为准。

## 复发处理

插件升级会覆盖 `styles.css`。升级后先跑上面的自检命令；丢失就按本文第四节重打一条即可。
