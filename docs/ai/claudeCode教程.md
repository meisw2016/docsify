# ⚠️ 重要声明

首先需要明确：**Claude Code 官方版本是需要付费订阅的**（每月约200美元）。网络上所谓的"破解"方法通常有以下几种：

1. **使用免费API中转服务**（合法合规，只是绕开官方计费）
2. **本地伪造配置文件跳过登录**（可能违反服务条款）
3. **利用其他订阅的代理转发**（灰色地带）

**使用这些方法存在风险**：可能违反 Anthropic 服务条款，账号可能被封禁；通过第三方中转站存在隐私泄露风险；部分破解方法可能包含恶意代码。

以下内容仅供学习和研究使用，请自行承担使用风险。

---

# 方法一：使用免费API中转服务（推荐，相对安全）

这种方法最稳定，原理是配置 Claude Code 连接到第三方提供的免费 API 中转站，而不是官方收费接口。

## 核心思路

Claude Code 本身只是一个 npm 包（免费），它需要调用 Anthropic 的 API。通过修改环境变量 `ANTHROPIC_BASE_URL` 和 `ANTHROPIC_AUTH_TOKEN`，可以让它指向其他提供 API 的服务商。

## 步骤详解

### 1. 安装 Node.js（≥18.0）

```bash
node --version  # 确认版本 ≥18.0
```

如果版本过低，去 [nodejs.org](https://nodejs.org) 下载 LTS 版本安装。

### 2. 全局安装 Claude Code

```bash
npm install -g @anthropic-ai/claude-code
```

验证安装：
```bash
claude --version
```

### 3. 注册免费API服务

目前几个可用的免费服务：

| 服务名称 | 免费额度 | 说明 |
|---------|---------|------|
| **魔搭社区 (ModelScope)** | 每日2000次调用 | 阿里旗下，较稳定，需绑定阿里云账号 |
| **AnyRouter** | 注册送$100额度 | 社区维护的中转站，通过邀请可获更多额度 |
| **Antigravity** | 每周刷新额度 | Google出品，通过反代工具可接Claude Code |

以**魔搭**为例（最稳定）：
- 访问 [modelscope.cn](https://modelscope.cn) 注册账号
- 进入"访问令牌"页面，获取你的 token（以 `ms-` 开头）

### 4. 配置环境变量

**临时配置**（每次新终端需重新执行）：
```bash
export ANTHROPIC_BASE_URL=https://api-inference.modelscope.cn
export ANTHROPIC_AUTH_TOKEN=你的token
export ANTHROPIC_MODEL=ZhipuAI/GLM-4.6
```

**永久配置**（推荐）——创建配置文件：

```bash
# 创建 .claude 文件夹
mkdir -p ~/.claude

# 创建 settings.json
cat > ~/.claude/settings.json << 'EOF'
{
  "env": {
    "ANTHROPIC_API_KEY": "你的token",
    "ANTHROPIC_BASE_URL": "https://api-inference.modelscope.cn",
    "ANTHROPIC_MODEL": "ZhipuAI/GLM-4.6",
    "ANTHROPIC_SMALL_FAST_MODEL": "ZhipuAI/GLM-4.6",
    "CLAUDE_CODE_DISABLE_NONESSENTIAL_TRAFFIC": "1"
  }
}
EOF
```

**Windows用户**：配置文件位于 `C:\Users\你的用户名\.claude\settings.json`

### 5. 启动 Claude Code

```bash
cd 你的项目目录
claude
```

首次启动需要：
- 选择主题（回车默认）
- **关键步骤**：选择"使用自定义 API"（第一个选项），而不是官方 API
- 确认安全须知
- 信任工作目录

输入 `hi` 测试是否能正常响应。

---

# 方法二：本地伪造登录状态（跳过官方验证）

**⚠️ 风险提示**：此方法通过修改本地配置文件绕过官方登录校验，可能违反 Anthropic 服务条款。Anthropic 可能通过联网校验使此方法失效。

## 原理

Claude Code 启动时会检查本地配置文件中的 `hasCompletedOnboarding` 和 `primaryApiKey` 字段。通过手动创建这些文件，可以"欺骗"软件认为已完成登录。

## 操作步骤

### 1. 完全关闭 Claude Code

确保所有相关进程已结束（任务管理器/活动监视器中结束）。

### 2. 创建第一个配置文件 `.claude.json`

**放在用户目录下**（不是安装目录）：
- **Windows**: `C:\Users\你的用户名\.claude.json`
- **macOS/Linux**: `/Users/你的用户名/.claude.json`

文件内容：
```json
{
  "hasCompletedOnboarding": true,
  "primaryApiKey": "test"
}
```

**Windows 命名技巧**：如果无法创建以点开头的文件，先命名为 `claude.json`，保存后重命名为 `.claude.json`。

### 3. 创建第二个配置文件 `settings.json`

在用户目录下创建 `.claude` 文件夹：
```bash
mkdir -p ~/.claude
```

在该文件夹内创建 `settings.json`，内容如下：
```json
{
  "env": {
    "CLAUDE_CODE_DISABLE_NONESSENTIAL_TRAFFIC": "1"
  },
  "permissions": {
    "allow": [],
    "deny": []
  }
}
```

`CLAUDE_CODE_DISABLE_NONESSENTIAL_TRAFFIC: "1"` 的作用是禁用官方联网校验。

### 4. 重启 Claude Code

正常启动后应不再弹出登录窗口。

## 常见问题

| 问题 | 解决方案 |
|------|---------|
| 仍然弹出登录窗口 | 检查文件路径是否正确（用户目录，不是安装目录）；确认文件扩展名是 `.json` 而非 `.txt`；完全结束所有 Claude 进程后重试 |
| 找不到 `.claude` 文件夹 | 需要显示隐藏文件：Windows 勾选"查看→隐藏的项目"；macOS 按 `Command+Shift+.` |
| 破解后会失效吗 | 配置文件永久保存，除非手动删除。如果失效，重新创建即可 |

---

# 方法三：使用 Kiro Pro 订阅 + 代理转发（需付费订阅）

**适用人群**：已有 Kiro Pro 订阅（约$150-200/月），希望同时使用 Claude Code 的用户。

## 原理

Kiro Pro 订阅包含 Claude 模型的访问权限。通过本地代理工具 `kiro-gateway`，将 Claude Code 的请求转发到 Kiro 的 API 通道，从而绕过 Anthropic 的直接计费。

## 简要步骤

1. 安装 Kiro CLI 并登录 Pro 账号
2. 下载并配置 `kiro-gateway`（需修改数据库路径）
3. 设置环境变量指向本地代理：
```bash
export ANTHROPIC_BASE_URL="http://127.0.0.1:9000"
export ANTHROPIC_API_KEY="kiro-local-proxy-key"
```
4. 启动 Claude Code

**注意**：此方法需要已有 Kiro Pro 订阅，配置过程较复杂，约需45-60分钟。

---

# 方法对比

| 方法 | 成本 | 稳定性 | 风险等级 | 适用人群 |
|------|------|--------|----------|---------|
| API中转站 | 免费 | 中等（服务可能失效） | 低 | 大多数人 |
| 本地伪造登录 | 免费 | 低（官方可能封堵） | 中 | 技术爱好者 |
| Kiro代理转发 | $150-200/月 | 较高 | 低（合法订阅） | 已有Kiro订阅的用户 |

---

# 最后建议

1. **优先使用方法一**（API中转站），相对合规且稳定，魔搭等平台有官方背景，风险较低。
2. 如果只是**偶尔使用**，没必要折腾破解，直接使用官方免费试用或按量付费即可。
3. 所有第三方中转站都会**看到你的API请求内容**，不要在对话中输入敏感信息（密钥、密码、客户数据等）。
4. 遇到问题时，先用 `claude --version` 确认安装成功，再检查配置文件格式是否正确。