# cclaude

交互式 Claude Code 模型切换启动器。通过终端菜单快速选择不同的模型端点，自动配置环境变量和 `~/.claude/settings.json`，启动 Claude Code 会话，退出后自动恢复原始设置。

## 依赖

- [claude](https://docs.anthropic.com/en/docs/claude-code) — Claude Code CLI
- [jq](https://jqlang.github.io/jq/) — JSON 处理工具

## 快速开始

```bash
# 1. 复制示例配置文件
cp config.example.json config.json

# 2. 编辑 config.json，填入你的真实配置
vim config.json

# 3. 运行
cclaude
```

## 配置文件

配置文件 `config.json` 是一个 JSON 数组，每个元素代表一个可选的模型端点。脚本会自动在菜单首部添加「保持现状（直接启动）」、尾部添加「Exit」，无需手动配置。

### 基本结构

```json
[
  {
    "label": "显示名称",
    "params": {
      "model": "模型名称",
      "auth_token": "认证令牌",
      "base_url": "API 地址"
    }
  }
]
```

### 必填字段

| 字段 | 说明 |
|------|------|
| `label` | 菜单中显示的名称 |
| `params.model` | 模型标识符，如 `claude-opus-4-6` |
| `params.auth_token` | API 认证令牌 |
| `params.base_url` | API 端点地址 |

### 自定义参数

`params` 中除了 `model`、`auth_token`、`base_url` 这三个保留字段外，可以添加任意自定义键值对。这些自定义参数会被同时：

1. **导出为当前 shell 的环境变量**
2. **写入 `~/.claude/settings.json` 的 `.env` 对象中**

这意味着你可以通过配置文件直接控制 Claude Code 的各种行为开关，无需手动设置环境变量。

#### 示例

```json
{
  "label": "my_server_with_custom_env",
  "params": {
    "model": "claude-opus-4-6",
    "auth_token": "sk-your-token",
    "base_url": "https://your-server:8443/",
    "CLAUDE_CODE_DISABLE_NONESSENTIAL_TRAFFIC": 1,
    "NODE_TLS_REJECT_UNAUTHORIZED": "0"
  }
}
```

选择该节点后，`CLAUDE_CODE_DISABLE_NONESSENTIAL_TRAFFIC` 和 `NODE_TLS_REJECT_UNAUTHORIZED` 会自动生效，无需额外操作。

### 完整示例

参考 [config.example.json](config.example.json)。

## 使用方式

运行 `cclaude` 后会出现交互式菜单：

```
Select Claude model (up/down + Enter):

  > 保持现状（直接启动）
    my_server_opus-4-6
    my_server_sonnet-4
    Exit
```

- 使用 **上/下方向键** 或 **k/j** 移动选择
- 按 **Enter** 确认
- 选择「保持现状（直接启动）」会以当前环境直接启动 Claude
- 选择「Exit」退出

## 工作原理

1. 从 `config.json` 读取模型配置列表
2. 用户选择后，设置对应的环境变量（`ANTHROPIC_AUTH_TOKEN`、`ANTHROPIC_BASE_URL`、`ANTHROPIC_MODEL` 等）
3. 将配置写入 `~/.claude/settings.json`，使 Claude 的 `/status` 命令能正确反映当前端点
4. 启动 `claude` 会话
5. 会话结束后自动恢复原始的 `settings.json`
