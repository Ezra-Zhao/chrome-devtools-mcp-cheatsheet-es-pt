# 第一次调试演练 | First-debug drill

从零到跑通，按顺序来（命令与提示词均出自上游 README，2026-11-14 口径）：

1. 备一个干净的 Chrome：调试专用、里面不登录邮箱与银行等敏感账号——这个服务会把浏览器内容敞开给 MCP 客户端。
2. 先关两个开关再接：MCP 客户端配置里用带 `--no-usage-statistics` 与 `--no-performance-crux` 的那段 JSON（见本仓 README 隐私卡）；两个都是默认开着的，一个发使用统计、一个把性能轨迹的网址发给 CrUX 接口。
3. 验活：对 agent 说原仓给的第一句提示词 `Check the performance of https://developers.chrome.com`，它应当自己打开浏览器并录下一条性能轨迹。录不出来先装 `troubleshooting` skill 的思路排查连接，或查上游 Troubleshooting 指南。
4. 拿自己的页面练全套：让 agent 对你正在开发的一个页面依次做——录性能轨迹、列网络请求找出挂掉的那条、读控制台报错（注意堆栈有没有映射回源码）、截一张图存档。四样都过一遍，这个工具的价值面就摸清了。
5. 按痛点深挖：首屏慢就切 `debug-optimize-lcp`，越跑越卡切 `memory-leak-debugging`，登录态问题切 `cookie-debugging`，无障碍切 `a11y-debugging`。
6. CLI 路补练一遍：`npm i chrome-devtools-mcp@latest -g` 后跑 `chrome-devtools navigate_page 1 --url "https://developers.chrome.com"`、`chrome-devtools take_screenshot 1 --filePath screenshot.png`、`chrome-devtools stop` 三条，确认不接 agent 时终端也能独立干活。
7. 记住版本纪律：以后每次升级服务或 CLI，把七个 skill 一起更新——旧 skill 会教 agent 用已经改掉的工具与参数。
