# collaborating-with-grok

[![License: MIT](https://img.shields.io/badge/License-MIT-blue.svg)](https://opensource.org/licenses/MIT)

**Claude Code Agent Skill** —— 编程事实校验层，自动检测 API 废弃、版本变更、社区踩坑。

简体中文 | [English](./docs/README_EN.md)

---

## 概述

在编程辅助中，最致命的问题往往不是"写得慢"，而是：
- API 已经废弃
- 版本行为已变
- 社区已有坑但模型不知道

**collaborating-with-grok** 的定位是**事实校验层**，而非代码生成层：

| 职责 | 说明 |
|------|------|
| ✅ 查最新官方文档 | 版本号、API 状态、迁移指南 |
| ✅ 查 GitHub issue/PR/release note | 已知 bug、breaking changes |
| ✅ 查 StackOverflow/社区踩坑 | 常见问题、解决方案 |
| ❌ 生成代码 | 代码由 Claude 生成，Grok 仅提供事实 |

---

## 特性

| 特性 | 说明 |
|------|------|
| **零依赖** | 无需 Python 脚本，纯 SKILL.md 实现 |
| **MCP 集成** | 直接调用 grok-search MCP 服务 |
| **自动触发** | Claude 根据上下文自动判断是否调用 |
| **来源标注** | 所有信息强制标注来源和日期 |

---

## 触发条件

### 自动触发场景

| 类别 | 关键词信号 | 平台侧重 |
|------|-----------|----------|
| **版本/API** | "最新版本"、"是否支持"、"推荐写法"、"deprecated"、"breaking change"、"migration" | GitHub |
| **错误排查** | 特定 error message、异常运行行为、平台相关 bug (OS/GPU/Cloud) | StackOverflow, GitHub |
| **技术选型** | "A vs B"、"是否值得用"、"社区评价"、"production ready" | Reddit, GitHub |

### 跳过场景

- 纯算法题（排序、图遍历等）
- 明确已知的稳定 API
- 本地逻辑重构
- 内部 DSL / 私有协议

---

## 前置要求

1. **Claude Code** (v2.0.56+)
2. **grok-search MCP server** 已配置

### grok-search MCP 配置

在 `~/.claude/settings.json` 中添加：

```json
{
  "mcpServers": {
    "grok-search": {
      "command": "npx",
      "args": ["-y", "@anthropic-ai/grok-search"],
      "env": {
        "GROK_API_KEY": "your-api-key-here",
        "GROK_API_URL": "https://api.x.ai/v1"
      }
    }
  }
}
```

---

## 安装

### 方式一：通过主仓库安装

使用 [GudaStudio/skills](https://github.com/GuDaStudio/skills) 仓库：

```bash
# Linux / macOS
./install.sh --user --skill collaborating-with-grok

# Windows
.\install.ps1 -User -Skill collaborating-with-grok
```

### 方式二：独立安装

```bash
# 克隆仓库
git clone https://github.com/GuDaStudio/collaborating-with-grok.git

# 复制到 Claude Code skills 目录
# 用户级（所有项目生效）
cp -r collaborating-with-grok ~/.claude/skills/

# 项目级（仅当前项目生效）
cp -r collaborating-with-grok ./.claude/skills/
```

---

## 使用示例

安装后，Claude 会**自动**在合适的场景触发，无需显式调用。

### 示例 1：API 废弃检查

```
用户: "React useEffect cleanup 是否已废弃？"
Claude: [自动调用 mcp__grok-search__web_search]
       [返回 Fact Check Report]
```

### 示例 2：错误排查

```
用户: "遇到 TypeError: Cannot read property 'map' of undefined"
Claude: [自动调用 mcp__grok-search__web_search]
       [返回已知解决方案]
```

### 示例 3：技术选型

```
用户: "Zustand vs Jotai 哪个更适合 2025 年的项目？"
Claude: [自动调用 mcp__grok-search__web_search (platform: Reddit)]
       [返回社区评价对比]
```

---

## 输出格式

Claude 会以结构化报告形式呈现校验结果：

```markdown
## Fact Check: {topic}

### Status
- **Current Version**: x.x.x
- **API Status**: Active | Deprecated | Removed
- **Official Docs**: [link]

### Known Issues
- Issue #xxx: {description} [link]

### Community Practice
- Recommended: {pattern}
- Avoid: {anti-pattern}

### Sources
- [Title](url) (date)
```

---

## 工作流程

```
用户编程请求
       │
       ▼
  触发条件检测 ──否──> 直接编码
       │
      是
       ▼
  Grok-Search
       │
       ▼
  事实校验报告
       │
       ▼
  基于校验结果编码
```

---

## 反模式警告

| ❌ 错误用法 | ✅ 正确用法 |
|------------|------------|
| 每个问题都调用 Grok-Search | 仅在触发条件匹配时调用 |
| 让 Grok-Search 直接产出代码 | 仅获取事实信息，代码由 Claude 生成 |
| 把它当"另一个 LLM" | 定位为事实校验层 |
| 搜索后不验证时效性 | 必须标注信息日期 |

---

## 与其他 Skills 对比

| Skill | 用途 | 需要脚本 | MCP 集成 |
|-------|------|----------|----------|
| `collaborating-with-codex` | 委托给 OpenAI Codex CLI 进行编码 | ✅ Python | ❌ 外部 CLI |
| `collaborating-with-gemini` | 委托给 Google Gemini CLI 进行编码 | ✅ Python | ❌ 外部 CLI |
| `collaborating-with-grok` | 编程事实校验（API/版本/社区） | ❌ 无 | ✅ 直接 MCP |

**核心优势：** 纯 SKILL.md 实现，无外部依赖，加载更快，维护更简单。

---

## 故障排查

### Skill 未被触发

1. **检查安装：**
   ```bash
   ls ~/.claude/skills/collaborating-with-grok/SKILL.md
   ```

2. **重启 Claude Code** 以重新加载 Skills

3. **显式测试：**
   ```
   用户: "用 Grok 搜索 [query]"
   ```

### MCP 服务问题

1. **验证配置：**
   询问 Claude: "检查 Grok search 配置状态"
   Claude 会调用 `get_config_info` 并报告

2. **检查 API Key：**
   确认 `~/.claude/settings.json` 中已设置 `GROK_API_KEY`

---

## 贡献

欢迎贡献！请：
1. Fork 本仓库
2. 创建功能分支
3. 提交 PR 并附上清晰描述

---

## 许可证

MIT License - 详见 [LICENSE](LICENSE) 文件。

Copyright (c) 2025 [guda.studio](mailto:gudaclaude@gmail.com)

---

## 相关项目

- [GudaStudio/skills](https://github.com/GuDaStudio/skills) - 主 Skills 仓库
- [grok-search MCP](https://github.com/GuDaStudio/grok-search) - MCP 服务实现
- [Claude Code 文档](https://docs.anthropic.com/claude-code)

---

## 更新日志

### v0.2.0 (2025-01)
- 重新定位为编程事实校验层
- 新增触发条件矩阵
- 新增反模式警告
- 移除通用搜索示例，聚焦编程场景

### v0.1.0 (2025-12)
- 初始版本
- 纯 SKILL.md 实现
- 支持 web_search、web_fetch、get_config_info
