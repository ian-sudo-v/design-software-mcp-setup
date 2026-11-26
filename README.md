# 设计软件MCP一键配置

一个 WorkBuddy Skill：在目标机器上只读检测设计/创作软件对应的 MCP 服务端、宿主组件与 CLI，汇总缺失项，经用户确认后按项安装并复查。

## 覆盖软件

- Adobe Illustrator（AI）
- Adobe Photoshop（PS）
- Adobe InDesign
- Rhino（犀牛）
- AutoCAD
- SketchUp、3ds Max（含 maxctl CLI）、Blender
- D5 Works、D5 Lite
- ComfyUI（本地 MCP）

## 核心规则：宿主软件优先

对每一项 MCP，先确认对应宿主软件是否已安装。**宿主未安装时标记为"不适用（宿主未安装）"，不安装对应 MCP、插件、Skill 或依赖，也不自行下载宿主软件。** 即使用户回复"全部安装"，也跳过所有不适用项。只有"宿主已安装且 MCP 缺失"才进入确认清单。

## 文件结构

```text
design-software-mcp-setup/
├── SKILL.md    # 技能定义：检测表、确认流程、ComfyUI 安装细则、新增软件安装参照
└── README.md   # 本说明
```

## 安装方法

将本目录复制到 WorkBuddy 用户技能目录：

```bash
# 路径按实际环境调整
cp -r design-software-mcp-setup ~/.workbuddy/skills/
```

Windows 下目标目录为 `C:\Users\<用户名>\.workbuddy\skills\design-software-mcp-setup`。

## 使用方法

在 WorkBuddy 中直接说"检测设计软件 MCP"或"配置设计软件 MCP"，技能会：

1. 只读检测所有项目，输出逐组件状态表（已完成 / 缺失 / 不适用（宿主未安装）/ 待核实 / 不适用）；
2. 列出可安装缺失项，等待用户确认；
3. 仅安装被确认且宿主存在的项目，再复查连接状态。

## 参考来源

- Adobe 系列 MCP（Illustrator / Photoshop / InDesign）：<https://github.com/zachshallbetter/adobe-mcp>
- RhinoMCP：<https://github.com/jingcheng-chen/rhinomcp>
- AutoCAD MCP：<https://github.com/U-C4N/Autocad-MCP>
- D5 Works MCP 安装说明：<https://resource.d5works.cn/mcp/d5-works-mcp-install.md>

各 MCP 服务端均为第三方开源项目，安装前请阅读其仓库的最新说明与许可证；本技能只提供检测与配置流程，不托管任何服务端代码。
