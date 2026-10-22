# obsidian-plugin-fixes

Obsidian 插件的本地改造与补丁存档。每个问题一个目录，写清症状、根因、补丁、复现与复发处理，换机或插件升级后能照着重打。

这些补丁都在真实使用中定位过根因，但**不是上游官方修复**；插件升级会用新的 `styles.css` 覆盖本地改动，升级后请按各目录 README 的自检命令确认补丁是否还在。

## 目录

| 插件 | 问题 | 补丁落点 | 上游 |
|---|---|---|---|
| hermes-console | 终端每行行尾 1-2 个汉字 / 2-3 个西文被裁掉 | `.obsidian/plugins/hermes-console/styles.css` 末尾追加一条 CSS | [issue #4](https://github.com/dannyshmueli/obsidian-hermes-console/issues/4) |

## 口径声明

每个目录的 README 末尾有「证据与未验证项」一节。凡是本机没有留下可重跑痕迹的数字，都会明确标注为未复核，不当作已证实结论引用。

## License

未附许可证。仓库内容为个人排查记录与 CSS 补丁片段，取用请自行判断。
