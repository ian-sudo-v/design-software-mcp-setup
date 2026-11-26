---
name: design-software-mcp-setup
version: "2.0.0"
display_name: "设计软件MCP一键配置"
display_name_en: "Design Software MCP One-Click Setup"
description: 检测 Adobe Illustrator（AI）、Photoshop（PS）、InDesign、Rhino（犀牛）、AutoCAD、SketchUp、3ds Max、Blender、D5 Works、D5 Lite 和 ComfyUI 的 MCP、宿主组件及 CLI；宿主软件未安装时不安装对应 MCP；汇总缺失项，经用户确认后按项安装并复查。
description_zh: "检测 Adobe Illustrator（AI）、Photoshop（PS）、InDesign、Rhino（犀牛）、AutoCAD、SketchUp、3ds Max、Blender、D5 Works、D5 Lite 和 ComfyUI 的 MCP、宿主组件及 CLI；宿主软件未安装时不安装对应 MCP；汇总缺失项，经用户确认后按项安装并复查。"
description_en: "Checks MCP servers, host components, and CLIs for Adobe Illustrator (AI), Photoshop (PS), InDesign, Rhino, AutoCAD, SketchUp, 3ds Max, Blender, D5 Works, D5 Lite, and ComfyUI; never installs an MCP whose host application is absent; summarizes missing items, installs only confirmed items after user approval, and rechecks the results."
agent_created: true
---

# 设计软件 MCP 一键配置

使用当前智能体可访问的终端、文件系统和 MCP 客户端。先确认工具访问的是用户指定的目标机器。若只能聊天或只能访问远程沙箱，请提供适合目标系统的只读检查命令，让用户回传输出，再依据输出判断。未经验证的项目标记为"待核实"。

## 核心约束　宿主软件优先，不装无宿主的 MCP

对每一个 MCP 项，先确认其宿主软件是否已安装在目标机器上。宿主未安装时，该项标记"不适用（宿主未安装）"，**不安装对应的 MCP、配套插件、Skill 或依赖，也不自行下载、安装、引导安装宿主软件**。即使该 MCP 声称支持无头模式或云端模式，本流程仍按此规则处理。

"不适用"项不进入待确认安装清单；即使用户回复"全部安装"，也必须跳过所有不适用项，并逐项说明跳过原因。只有"宿主已安装且对应 MCP 缺失"才标记"缺失"并进入确认清单。

## 第一步　只读检测全部项目

发现操作系统、已安装的建模宿主、活动实例和目标 MCP 客户端。检查该客户端的实际配置和工具列表，结合软件安装记录、程序文件、宿主插件目录及 Skill 清单交叉核对。检测期间只执行只读查询，避免触发下载、安装、自动修复或配置写入。

对每个项目都先记录宿主软件的安装证据与版本，再核对 MCP 服务端、注册项和连通性。宿主证据包括注册表卸载项、程序文件目录、宿主插件目录、运行进程和宿主内版本信息；证据不足时标记"待核实"，不要据此安装。

| 项目 | 逐个核对的组件和证据 |
|---|---|
| Adobe Illustrator（AI） | 先核对宿主 Adobe Illustrator 是否已安装及版本（注册表卸载项、程序文件、运行进程）。宿主存在时，检查目标客户端 MCP 配置中是否有 `illustrator` 注册项，核对服务端入口（如 `adobe-mcp-illustrator` 或本地仓库的 `dist/index.js`），并用 stdio 发 `initialize` + `tools/list` 验证握手。宿主未安装 → 不适用，不安装其 MCP。 |
| Adobe Photoshop（PS） | 先核对宿主 Adobe Photoshop 是否已安装及版本。宿主存在时，检查 `photoshop` 注册项及服务端入口（如 `adobe-mcp-photoshop`）；旧版 Windows 环境另有基于 COM 的 `photoshop-mcp-server`，需核对 Python 3.10+ 与 `PS_VERSION`。用 stdio 发 `initialize` + `tools/list` 验证握手。宿主未安装 → 不适用，不安装其 MCP。 |
| Adobe InDesign | 先核对宿主 Adobe InDesign 是否已安装及版本。宿主存在时，检查 `indesign` 注册项及服务端入口（如 `adobe-mcp-indesign`），用 stdio 发 `initialize` + `tools/list` 验证握手。宿主未安装 → 不适用，不安装其 MCP。 |
| Rhino（犀牛） | 先核对宿主 Rhino / Rhinoceros 是否已安装及版本（RhinoMCP 插件要求 Rhino 8.20 及以上、.NET 8 运行时）。宿主存在时，检查 Rhino 包管理器中的 `rhinomcp` 插件、MCP 配置中的 `rhino` 注册项（`uvx rhinomcp@latest`，可带 `RHINO_MCP_HOST`/`RHINO_MCP_PORT`），以及 Rhino 内 `mcpstart` 已启动、回环端口可连。用 `mcpstatus` 和 stdio `tools/list` 验证。宿主未安装 → 不适用，不安装其 MCP。 |
| AutoCAD | 先核对宿主 AutoCAD 是否已安装及版本（注册表卸载项、程序文件、运行进程）。宿主存在时，检查 `autocad` 注册项及服务端命令 `autocad-mcp`（`autocad-mcp-pro`），核对 `AUTOCAD_MCP_BACKEND` 与 `ALLOWED_PATHS`，用只读的 `system(operation="preflight")` 或 capabilities 检查 COM 会话与依赖。宿主未安装 → 不适用，不安装其 MCP；不要把 ezdxf 无头模式当作"宿主已安装"的替代。 |
| SketchUp | 检查 MCP 服务端及其注册项、宿主端桥接插件。能调用健康接口时，分别记录服务端响应和已连接的 SketchUp 实例。宿主未安装 → 不适用，不安装其 MCP。 |
| 3ds Max MCP | 检查 MCP 服务端及其注册项，再检查 Max 宿主内的监听组件和加载入口。以只读场景信息调用验证连接。宿主未安装 → 不适用，不安装其 MCP。 |
| 3ds Max CLI | 单独定位 maxctl，必要时用发现的绝对路径运行帮助命令。用 maxctl info 检查它能否通过监听组件读取 Max 信息。宿主未安装 → 不适用，不安装其 CLI/MCP。 |
| Blender | 核对 MCP 服务端、注册命令及与该服务端实现匹配的 Blender 插件。分别报告插件文件、启用状态和只读状态查询结果。宿主未安装 → 不适用，不安装其 MCP。 |
| D5 Works | 以发布方说明识别 MCP 注册项和配套 Skill，分别核对。安装来源为 https://resource.d5works.cn/mcp/d5-works-mcp-install.md 。检测阶段仅读取说明。宿主未安装 → 不适用，不安装其 MCP。 |
| D5 Lite | 根据运行进程、安装记录及宿主插件位置确定对应的 D5 Lite 和 D5LiteWorkflowCli，Windows 下查找 D5LiteWorkflowCli.exe。多个宿主分别核对。通过该程序的帮助、版本和支持的只读诊断检查 CLI、本地工作流环境、宿主连接。返回 D5 Lite 版本、CLI 实际路径和版本、对应宿主及各项结果。版本不可读时记录原因。宿主未安装 → 不适用，不安装其 MCP。 |
| ComfyUI（本地 MCP） | 查注册表卸载项中的 Comfy Desktop；读 `%APPDATA%\Comfy Desktop\installations.json` 找本地安装条目的 `installPath`。确认仓库根为含 `main.py` 的内层目录。检查目标客户端 MCP 配置中是否已有 `comfy-mcp` 注册项，以及独立 venv 中 `comfy.exe` / `comfy-mcp.exe` 是否存在、`comfy --version` 是否为 ≥1.14。用 stdio 发 `initialize` + `tools/list` 验证服务端是否真的可握手。检查本地 ComfyUI 是否在 8188 端口监听。宿主未安装 → 不适用，不安装其 MCP。 |

用一张表逐组件列出安装状态、证据、连接结果。已确认安装标记"已完成"，查明缺少标记"缺失"，宿主未安装标记"不适用（宿主未安装）"，证据不足标记"待核实"，平台不支持或发布方未提供标记"不适用"。区分安装、注册、启用和连通状态。宿主未启动、连接失败或 CLI 不在 PATH 中，均不能单独证明组件缺失。宿主未安装与宿主已安装但未启动必须区分：前者不安装，后者只提示启动宿主。对失败给出具体原因或尚需的证据，并给出对应处理建议。

## 第二步　统一确认后安装

全部适用组件已完成时，汇总并结束。只有待核实项时，说明需补充的路径、权限或诊断输出。发现缺失项时，在全部项目检测结束后，一次列出每个缺失组件的编号、所属项目、发布来源、必要依赖和预计配置变更。依赖也缺失时明确列入确认范围。安装清单只包含"宿主已安装且对应 MCP 缺失"的项；"不适用（宿主未安装）"项不得列入，即使用户要求"全部安装"也跳过并说明原因。

询问用户："是否安装以上缺失项？请回复'全部安装'、要安装的编号或'暂不安装'。"在此结束本轮，等候明确答复。用户选择暂不安装时结束。

仅对明确选中的组件及已确认依赖执行安装，复用现有组件。按目标平台和客户端选择安装方式，D5 Works MCP 和配套 Skill 使用上述发布方说明。Adobe 三件套按宿主实际安装情况逐个注册，只为已安装的应用注册对应服务端，不因为选中某一个而安装未安装应用的服务端或整套 `adobe-mcp-suite`。若发现需要新增未确认的依赖或变更，先列明再询问。安装后重新核对文件和注册状态，执行可用的只读连接测试，逐项报告成功、失败原因及处理建议。

## 附录 A　ComfyUI 本地 MCP 安装细则（当该项被选中时执行）

**目标**：在目标机器建独立 venv，安装 `comfy-cli` + `comfy-mcp`，注册到 MCP 客户端，并用 stdio 握手验证。只做本地路线，**不碰 Comfy Cloud，无需 API Key 或订阅**。

**前置**：本机必须已有 ComfyUI 安装。若检测发现根本没有，停下来请用户确认，**不要自行执行 `comfy install` 下载数 GB 环境**。

**三套组件别混淆**：`comfy-cli` 是本地引擎，`comfy-mcp` 是给客户端的 stdio 服务器，`comfy-sdk` 是云端 API 编程库（本流程不涉及）。

1. **定位工作区**：从注册表卸载项确认 Comfy Desktop 安装位置；从 `%APPDATA%\Comfy Desktop\installations.json` 取本地安装条目的 `installPath`。仓库根是**内层**含 `main.py` 的目录，通常为 `<installPath>\ComfyUI`；指到外层会报 `not_in_workspace`。记录该工作区自带 python（如 `standalone-env` 或 `.venv` 下的 python）备用。

2. **建独立 venv（不要污染 WorkBuddy 默认环境）**：
   ```
   <uv路径> venv C:\Users\<用户名>\.workbuddy\envs\comfy-mcp --python 3.12
   ```
   没有 uv 时先定位本机 `uv.exe`（常见 `C:\Users\<用户名>\.local\bin\uv.exe`）。

3. **安装 —— 必须用腾讯云镜像**：
   ```
   <uv路径> pip install --python <venv>\Scripts\python.exe --index-url https://mirrors.cloud.tencent.com/pypi/simple/ comfy-cli comfy-mcp
   ```
   已知坑：PyPI 官方源依赖回溯会卡死十几分钟；清华源没有 comfy-cli（403/无包）。腾讯云镜像通常 10 秒内完成。

4. **验证命令**：
   ```
   <venv>\Scripts\comfy.exe --version
   <venv>\Scripts\python.exe -c "import importlib.util as u; print([bool(u.find_spec(n)) for n in ['comfy_cli','comfy_mcp','mcp']])"
   ```
   期望 `comfy` 版本 ≥1.14，三个模块都为 True。

5. **设默认工作区**：
   ```
   <venv>\Scripts\comfy.exe set-default "<仓库根，含 main.py 那层>"
   ```

6. **写 MCP 配置**：编辑 `C:\Users\<用户名>\.workbuddy\mcp.json`，**保留原有所有条目**，仅新增：
   ```json
   "comfy-mcp": {
     "command": "C:\\Users\\<用户名>\\.workbuddy\\envs\\comfy-mcp\\Scripts\\comfy-mcp.exe",
     "env": {
       "COMFY_BIN": "C:\\Users\\<用户名>\\.workbuddy\\envs\\comfy-mcp\\Scripts\\comfy.exe"
     }
   }
   ```
   注意 JSON 中反斜杠要双写。

7. **端到端握手验证（必须做，不能只验证 import）**：
   ```
   printf '%s\n' \
   '{"jsonrpc":"2.0","id":1,"method":"initialize","params":{"protocolVersion":"2024-11-05","capabilities":{},"clientInfo":{"name":"probe","version":"1"}}}' \
   '{"jsonrpc":"2.0","method":"notifications/initialized"}' \
   '{"jsonrpc":"2.0","id":2,"method":"tools/list","params":{}}' \
   | timeout 30 "<venv>\Scripts\comfy-mcp.exe"
   ```
   期望返回 initialize 结果，且 `tools/list` 含 `server_info`、`run_workflow`、`generate_image`。

8. **复查与收尾**：安装后重新走一遍第一步的 ComfyUI 检测项，逐项报告结果，并告知用户：
   - 新 MCP 不会自动生效，需在连接器管理页对 `comfy-mcp` 点"信任"或重启会话；
   - MCP 服务器不会自动启动 ComfyUI，真正生成前需先启动本机 ComfyUI（默认 8188 端口），可用 `comfy launch` 或在 Comfy Desktop 中启动。

按上面步骤执行后，报告：venv 路径、comfy 版本、默认工作区路径、mcp.json 新增条目、握手验证结果。

## 附录 B　新增设计软件 MCP 安装参照（仅当对应宿主已安装且该项被选中时执行）

发布来源均以官方仓库 README 为准，安装前读取最新说明；下表命令为已验证的常用形式。所有注册操作都**保留目标客户端 MCP 配置中的原有条目**，只新增对应项。

| 项目                    | 发布来源                                         | 前置                                                                             | 推荐安装与注册                                                                                                                                                                                                                                                  | 验证要点                                                                                                         |
| --------------------- | -------------------------------------------- | ------------------------------------------------------------------------------ | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------ |
| Adobe Illustrator（AI） | https://github.com/zachshallbetter/adobe-mcp | Windows 或 macOS；Node.js 18+；Illustrator 2023 或更新版本                             | `npx -y adobe-mcp-illustrator`；客户端注册 `"illustrator": {"command": "npx", "args": ["-y", "adobe-mcp-illustrator"]}`                                                                                                                                        | 打开 Illustrator，stdio 发 `initialize` + `tools/list`，确认工具列表非空；再做一次只读文档信息调用                                     |
| Adobe Photoshop（PS）   | https://github.com/zachshallbetter/adobe-mcp | Windows 或 macOS；Node.js 18+；Photoshop 2023 或更新版本                               | `npx -y adobe-mcp-photoshop`；客户端注册 `"photoshop": {"command": "npx", "args": ["-y", "adobe-mcp-photoshop"]}`                                                                                                                                              | 打开 Photoshop，stdio `tools/list` 握手；再做一次只读文档/图层信息调用                                                           |
| Adobe InDesign        | https://github.com/zachshallbetter/adobe-mcp | Windows 或 macOS；Node.js 18+；InDesign 2023 或更新版本                                | `npx -y adobe-mcp-indesign`；客户端注册 `"indesign": {"command": "npx", "args": ["-y", "adobe-mcp-indesign"]}`                                                                                                                                                 | 打开 InDesign，stdio `tools/list` 握手；再做一次只读文档信息调用                                                               |
| Rhino（犀牛）             | https://github.com/jingcheng-chen/rhinomcp   | Rhino 8.20+ 且使用 .NET 8 运行时；uv 提供 `uvx`；先安装 Rhino 包管理器中的 `rhinomcp` 插件并重启 Rhino | `uvx rhinomcp@latest`；客户端注册 `"rhino": {"command": "uvx", "args": ["rhinomcp@latest"], "env": {"RHINO_MCP_HOST": "127.0.0.1"}}`；在 Rhino 命令行执行 `mcpstart`                                                                                                  | Rhino 内 `mcpstatus` 显示端口与文档；stdio `tools/list` 可列工具；同一时刻只运行一个 RhinoMCP 服务端                                   |
| AutoCAD               | https://github.com/U-C4N/Autocad-MCP         | Windows；Python 3.11+；uv 或 pip；AutoCAD 完整版（发布方实机验证版本为 2026），需要 COM 与 `pywin32`  | `uvx --from "autocad-mcp-pro[com]" autocad-mcp`（或 `pip install "autocad-mcp-pro[com]"` 后运行 `autocad-mcp`）；客户端注册 `"autocad": {"command": "autocad-mcp", "env": {"AUTOCAD_MCP_BACKEND": "auto", "ALLOWED_PATHS": "C:\\Users\\<用户名>\\Documents\\AutoCAD"}}` | 启动 AutoCAD，先调用只读 `system(operation="preflight")`；确认 backend=com 且无 `cad_not_running`；stdio `tools/list` 可列工具 |

安装后重新走第一步对应项目检测，逐项报告安装、注册、启用、连通四个状态。提醒用户新 MCP 需要在客户端配置中信任或重启会话后才会生效；Adobe 服务端、RhinoMCP 和 AutoCAD live 后端都要求对应宿主软件处于运行状态。
