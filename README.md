# Chrome DevTools MCP 速查表 | chrome-devtools-mcp Cheatsheet | Guía rápida de chrome-devtools-mcp

> 📚 **Colección / Coleção**: esta hoja forma parte de [cheatsheets](https://github.com/Ezra-Zhao/cheatsheets) — 100 hojas de referencia en un solo repositorio. / Esta folha faz parte de [cheatsheets](https://github.com/Ezra-Zhao/cheatsheets) — 100 folhas de referência em um só repositório. ⭐

> 一页看懂 chrome-devtools-mcp：Chrome DevTools 官方团队出的 MCP 服务与 CLI，让 Claude、Cursor、Copilot、Gemini CLI 等编程 agent 直接控制与检查一个真 Chrome——录性能轨迹、看网络请求、读带源码映射的控制台报错、截图、跑 Lighthouse 与无障碍审计。本页卖的不是翻译，是工具图、七个 skill 的分工，和两个默认开着的数据开关的关法。

- 上游项目：[ChromeDevTools/chrome-devtools-mcp](https://github.com/ChromeDevTools/chrome-devtools-mcp)（2026-11-14 实测 56,109 ★ / 4,651 forks，65 个 release，Apache-2.0）
- 平台：凡支持 MCP 的客户端与 agent；需要 Node.js LTS 与当前稳定版 Chrome，底层用 Puppeteer 执行操作并自动等待结果
- 本页结构：中文主文 → English → Español → Português

---

Apoyo y más: [GitHub Sponsors](https://github.com/sponsors/Ezra-Zhao) · [Gumroad — plantillas y cuadernos que uso](https://ezraflow28.gumroad.com) · [Ko-fi](https://ko-fi.com/ezra777zhao)

## 先分工：三张浏览器速查别拿混

| 工具 | 管什么 | 一句话 |
|---|---|---|
| agent-browser（已推速查） | 操作 | 一条命令一步，把网页点穿、表填完 |
| [lightpanda](https://github.com/Ezra-Zhao/lightpanda-cheatsheet-es-pt)（已推速查） | 引擎 | 轻量无头浏览器，大批量抓取与自动化跑量 |
| **本页 chrome-devtools-mcp** | 调试与性能 | 页面慢在哪、哪个请求挂了、控制台为什么红——让 agent 自己打开调试台看 |

agent 写完页面却看不见页面跑成什么样，是编程 agent 当前最大的盲区；这个工具就是官方给的那双眼睛。

## 隐私卡（连上真浏览器之前先看）

- **使用统计默认开着**：Google 按官方口径收集工具调用成功率、延迟与环境信息。关法：启动参数加 `--no-usage-statistics`，或设环境变量 `CHROME_DEVTOOLS_MCP_NO_USAGE_STATISTICS`（在 CI 环境里自动关）。注意它与 Chrome 浏览器自身的统计是两套，互不联动。
- **性能工具会把轨迹网址发给 Google**：性能工具会把 trace 的 URL 发给 CrUX 接口取真实用户数据，实验室数据配上田野数据才完整——不想发就加 `--no-performance-crux`。
- **浏览器内容对 MCP 客户端敞开**：这个服务能让客户端检查、调试、修改浏览器里的任何数据。登录着邮箱、银行等敏感账号的浏览器，别拿它连；官方免责声明就是这么写的。

推荐起步姿势：两个开关都先关上，配置写成这样：

```json
{
  "mcpServers": {
    "chrome-devtools": {
      "command": "npx",
      "args": ["-y", "chrome-devtools-mcp@latest", "--no-usage-statistics", "--no-performance-crux"]
    }
  }
}
```

## 两条起步路：先选哪条

**MCP 路（给 agent 用）**：把上面那段 JSON 加进你的 MCP 客户端配置（不加两个开关参数就是原仓的标准写法）。装好后第一句提示词就写原仓给的验活句：

```text
Check the performance of https://developers.chrome.com
```

客户端应当自己打开浏览器并录下一条性能轨迹——能跑完这句，链路就通了。注意：连上服务本身不会启动浏览器，agent 第一次调用需要浏览器的工具时才启动。

**CLI 路（自己在终端用、不接 agent）**：

```bash
npm i chrome-devtools-mcp@latest -g
chrome-devtools navigate_page 1 --url "https://developers.chrome.com"
chrome-devtools take_screenshot 1 --filePath screenshot.png
chrome-devtools stop
```

先记这三条：开页面、截图、收工。其余命令查上游 CLI 指南。

## 七个自带 skill：先用哪个

装插件时七个 skill 与 MCP 服务一起进来（Claude Code：`/plugin marketplace add ChromeDevTools/chrome-devtools-mcp` 再 `/plugin install chrome-devtools-mcp@chrome-devtools-plugins`；Gemini CLI：`gemini extensions install --auto-update https://github.com/ChromeDevTools/chrome-devtools-mcp`）。按场景对号：

| 场景 | 用哪个 skill |
|---|---|
| 日常调试与自动化页面，先学这个 | `chrome-devtools`（通用调试） |
| 想用 CLI 写 shell 脚本批量跑 | `chrome-devtools-cli`（注意：它要求先全局装好 CLI 本体） |
| 无障碍审计与修复 | `a11y-debugging` |
| 登录态、cookie、会话出问题 | `cookie-debugging` |
| 首屏慢、LCP 不达标 | `debug-optimize-lcp` |
| 页面越跑越卡、疑似内存泄漏 | `memory-leak-debugging` |
| 服务连不上 Chrome | `troubleshooting` |

手动装法：全局装包后，把 npm 包里的 `skills/` 对应文件夹拷进你 agent 的 skills 目录（原仓给了 Claude Code 的示例路径）。**版本纪律**：skill 描述的是对应版本的工具面，每次升级服务或 CLI 后把 skill 一起更新，旧 skill 会教 agent 用已经改掉的工具与参数。

## 工具面概览

服务暴露的工具按 DevTools 的分工成组（第三方实测口径：v1.12 时共 32 个工具，证据等级 B，以你装到的版本的工具参考为准）：

- **性能**：录制性能轨迹并提炼可执行的优化洞见
- **网络**：查看网络请求，定位挂掉与慢的请求
- **控制台**：读控制台消息，报错带源码映射的堆栈
- **页面操作与截图**：导航、交互、截图，Puppeteer 执行、自动等待
- **审计**：Lighthouse 与无障碍审计

只做基础任务、不想让 agent 碰全套工具时，启动加 `--slim`，只暴露精简工具集。

## 门槛一行版

Node.js LTS ＋ 当前稳定版 Chrome，两样齐了就能跑。**只官方支持 Chrome 与 Chrome for Testing**，其他 Chromium 系浏览器可能能用，但不保证、不受理问题。更新检查默认开着（查 npm 有无新版并在日志里提醒），关法是环境变量 `CHROME_DEVTOOLS_MCP_NO_UPDATE_CHECKS`。

## 诚实标注区

- **两个数据开关默认都是开的**：使用统计与 CrUX 田野数据，见隐私卡。这是本页把隐私卡放第一屏的原因——先关再连，是最稳的起步顺序。
- **32 个工具是第三方在 v1.12 的实测数**，不是官方承诺；工具面随版本变，以官方 Tool Reference 与你本地版本为准。
- **别拿日常浏览器连**：服务对客户端敞开浏览器全部内容，调试用的浏览器最好单独一个、里面不登录敏感账号。
- **本页数字的时点**：星数与 release 数为 2026-11-14 实测；配置与命令以当日上游 README 为准，上游迭代很快，装前扫一眼官方文档。

---

## English (summary)

chrome-devtools-mcp (by the Chrome DevTools team, ~56.1k stars as of 2026-11-14, Apache-2.0) lets coding agents such as Claude, Cursor, Copilot, and Gemini CLI control and inspect a live Chrome browser through an MCP server or a CLI: record performance traces, inspect network requests, read console messages with source-mapped stack traces, take screenshots, and run Lighthouse and accessibility audits. Operations run through Puppeteer with automatic waiting.

Start on the MCP path by adding `npx -y chrome-devtools-mcp@latest` to your MCP client config and testing with the upstream smoke prompt, "Check the performance of https://developers.chrome.com". Or use the CLI directly: install globally, then `chrome-devtools navigate_page`, `take_screenshot`, `stop`. Seven bundled skills cover general debugging, CLI scripting, accessibility, cookies/sessions, LCP optimization, memory leaks, and connection troubleshooting; keep them in sync with the server version.

Privacy first, because this page puts it on the first screen: usage statistics are **on by default** (opt out with `--no-usage-statistics` or the `CHROME_DEVTOOLS_MCP_NO_USAGE_STATISTICS` env var), and performance tools send trace URLs to Google's CrUX API for real-user field data (disable with `--no-performance-crux`). The server exposes the browser's contents to the MCP client — don't connect a browser signed into sensitive accounts. Requirements: Node.js LTS and current stable Chrome; only Chrome and Chrome for Testing are officially supported. `--slim` exposes a reduced tool set for basic tasks.

---

## Español (guía rápida — traducción de estudio, pendiente de revisión por un hablante nativo)

chrome-devtools-mcp (del equipo oficial de Chrome DevTools, ~56,1 mil estrellas en noviembre de 2026, Apache-2.0) permite que agentes como Claude, Cursor, Copilot o Gemini CLI controlen e inspeccionen un Chrome real mediante un servidor MCP o una CLI: grabar trazas de rendimiento, mirar las peticiones de red, leer la consola con el rastro mapeado al código fuente, hacer capturas y correr auditorías de Lighthouse y de accesibilidad. Por debajo usa Puppeteer y espera automáticamente a que cada acción termine.

**Tarjeta de privacidad (antes de conectar tu navegador):**

- **Las estadísticas de uso vienen activadas por defecto.** Google recoge tasas de éxito de las herramientas, latencia e información del entorno. Se apagan con el parámetro `--no-usage-statistics` o la variable `CHROME_DEVTOOLS_MCP_NO_USAGE_STATISTICS` (en entornos CI se desactivan solas). Son independientes de las estadísticas del propio Chrome.
- **Las herramientas de rendimiento envían la URL de la traza a Google.** Es la API de CrUX, para traer datos reales de usuarios y completar la foto. Se desactiva con `--no-performance-crux`.
- **El contenido del navegador queda expuesto al cliente MCP**, que puede inspeccionar, depurar y modificar cualquier dato. No conectes un navegador con sesiones sensibles (correo, banca) abiertas.

Configuración recomendada para empezar, con ambos interruptores cerrados:

```json
{
  "mcpServers": {
    "chrome-devtools": {
      "command": "npx",
      "args": ["-y", "chrome-devtools-mcp@latest", "--no-usage-statistics", "--no-performance-crux"]
    }
  }
}
```

**Las dos rutas de arranque:**

- **Ruta MCP (para tu agente):** pega la configuración anterior en tu cliente MCP (sin los dos parámetros extra es la forma estándar del proyecto). Primera frase para comprobar que todo funciona, la del propio proyecto: `Check the performance of https://developers.chrome.com`. El cliente debe abrir el navegador y grabar una traza. Ojo: conectar el servidor no abre el navegador; se abre cuando el agente usa la primera herramienta que lo necesita.
- **Ruta CLI (terminal, sin agente):** `npm i chrome-devtools-mcp@latest -g` y después `chrome-devtools navigate_page 1 --url "https://ejemplo.com"`, `chrome-devtools take_screenshot 1 --filePath captura.png` y `chrome-devtools stop` para cerrar.

**Las siete skills incluidas — cuál usar:** `chrome-devtools` para depuración general (empieza por esta); `chrome-devtools-cli` para escribir scripts de shell con la CLI (exige tener la CLI instalada en global); `a11y-debugging` para accesibilidad; `cookie-debugging` para cookies, sesiones y autenticación; `debug-optimize-lcp` cuando la primera pantalla tarda; `memory-leak-debugging` para fugas de memoria; `troubleshooting` cuando el servidor no logra conectar con Chrome. Se instalan como plugin (en Claude Code: `/plugin marketplace add ChromeDevTools/chrome-devtools-mcp` y `/plugin install chrome-devtools-mcp@chrome-devtools-plugins`; en Gemini CLI, la extensión con `--auto-update`) o copiando las carpetas de `skills/` del paquete npm al directorio de skills de tu agente. **Actualiza las skills cada vez que actualices el servidor**: una skill vieja enseña a usar herramientas y parámetros que ya cambiaron.

**Requisitos y límites:** Node.js LTS y Chrome estable actual. Solo Chrome y Chrome for Testing tienen soporte oficial; otros navegadores basados en Chromium pueden funcionar, sin garantías. Con `--slim` el servidor expone un conjunto reducido de herramientas para tareas básicas. Las comprobaciones de actualización vienen activadas (se apagan con `CHROME_DEVTOOLS_MCP_NO_UPDATE_CHECKS`).

---

## Português (guia rápido — tradução de estudo, aguardando revisão de um falante nativo)

O chrome-devtools-mcp (da equipe oficial do Chrome DevTools, ~56,1 mil estrelas em novembro de 2026, Apache-2.0) permite que agentes como Claude, Cursor, Copilot ou Gemini CLI controlem e inspecionem um Chrome real por meio de um servidor MCP ou de uma CLI: gravar traces de desempenho, olhar as requisições de rede, ler o console com o rastro mapeado ao código-fonte, tirar capturas de tela e rodar auditorias do Lighthouse e de acessibilidade. Por baixo ele usa o Puppeteer e espera automaticamente cada ação terminar.

**Cartão de privacidade (antes de conectar seu navegador):**

- **As estatísticas de uso vêm ligadas por padrão.** O Google coleta taxas de sucesso das ferramentas, latência e informações do ambiente. Desliga com o parâmetro `--no-usage-statistics` ou a variável `CHROME_DEVTOOLS_MCP_NO_USAGE_STATISTICS` (em ambientes CI elas desligam sozinhas). São independentes das estatísticas do próprio Chrome.
- **As ferramentas de desempenho enviam a URL do trace ao Google.** É a API CrUX, para trazer dados reais de usuários e completar o quadro. Desativa com `--no-performance-crux`.
- **O conteúdo do navegador fica exposto ao cliente MCP**, que pode inspecionar, depurar e modificar qualquer dado. Não conecte um navegador com sessões sensíveis (e-mail, banco) abertas.

Configuração recomendada para começar, com os dois interruptores fechados:

```json
{
  "mcpServers": {
    "chrome-devtools": {
      "command": "npx",
      "args": ["-y", "chrome-devtools-mcp@latest", "--no-usage-statistics", "--no-performance-crux"]
    }
  }
}
```

**Os dois caminhos de partida:**

- **Caminho MCP (para o seu agente):** cole a configuração acima no seu cliente MCP (sem os dois parâmetros extras é a forma padrão do projeto). Primeira frase para testar se tudo funciona, a do próprio projeto: `Check the performance of https://developers.chrome.com`. O cliente deve abrir o navegador e gravar um trace. Atenção: conectar o servidor não abre o navegador; ele abre quando o agente usa a primeira ferramenta que precisa dele.
- **Caminho CLI (terminal, sem agente):** `npm i chrome-devtools-mcp@latest -g` e depois `chrome-devtools navigate_page 1 --url "https://exemplo.com"`, `chrome-devtools take_screenshot 1 --filePath captura.png` e `chrome-devtools stop` para encerrar.

**As sete skills incluídas — qual usar:** `chrome-devtools` para depuração geral (comece por esta); `chrome-devtools-cli` para escrever scripts de shell com a CLI (exige a CLI instalada em global); `a11y-debugging` para acessibilidade; `cookie-debugging` para cookies, sessões e autenticação; `debug-optimize-lcp` quando a primeira tela demora; `memory-leak-debugging` para vazamentos de memória; `troubleshooting` quando o servidor não consegue conectar ao Chrome. Instalam-se como plugin (no Claude Code: `/plugin marketplace add ChromeDevTools/chrome-devtools-mcp` e `/plugin install chrome-devtools-mcp@chrome-devtools-plugins`; no Gemini CLI, a extensão com `--auto-update`) ou copiando as pastas de `skills/` do pacote npm para o diretório de skills do seu agente. **Atualize as skills toda vez que atualizar o servidor**: uma skill velha ensina a usar ferramentas e parâmetros que já mudaram.

**Requisitos e limites:** Node.js LTS e Chrome estável atual. Só Chrome e Chrome for Testing têm suporte oficial; outros navegadores baseados em Chromium podem funcionar, sem garantias. Com `--slim` o servidor expõe um conjunto reduzido de ferramentas para tarefas básicas. As verificações de atualização vêm ligadas (desligam com `CHROME_DEVTOOLS_MCP_NO_UPDATE_CHECKS`).

---

## 来源与声明

- 灵感来源 / Source / Fuente / Fonte: [ChromeDevTools/chrome-devtools-mcp](https://github.com/ChromeDevTools/chrome-devtools-mcp)（隐私口径出处为上游 README 的 Usage statistics 与 Disclaimers 两节；工具总数 32 为第三方在 v1.12 的实测口径，已在正文标注证据等级 B）
- 本仓是个人学习整理：内容用我自己的话总结自上游公开文档，不搬运原文；命令与参数以 2026-11-14 的上游 README 为准，上游迭代很快，请以官方文档与工具参考为最终依据。
- 西语、葡语部分为学习译稿，待母语校对。Las secciones en español y portugués son traducciones de estudio pendientes de revisión nativa.
- License: MIT © 2026 Guangyi Zhao
