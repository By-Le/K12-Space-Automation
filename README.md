# K12 Space Automation

中文 | [English](README.en.md)

K12 Space Automation 是一个本地运行的 K12 workspace 自动化控制台, 用于管理邮箱池, 邮箱验证码流程, K12 workspace 加入/切换, Sub2API 入库, access token 检查/修复, 以及账号 JSON 写出.

本仓库只包含源码, 文档, 配置模板和锁文件. 默认不包含真实运行配置, token, cookie, mailbox refresh token, account JSON 或任务数据. 即使仓库保持私有, 也不要提交任何真实账号凭据或本地运行数据.

## 功能概览

- 邮箱池管理: 导入, 选择, 删除, 状态标记, 失败重试.
- 邮箱接码: 支持普通接码 URL, 手动验证码, SMSBower Gmail, Emailnator Gmail.
- K12 流程: 登录, 加入或切换 K12 workspace, 读取 K12 上下文 access token.
- Sub2API: OAuth 入库, noRT 直入, 账号测活, access token 修复.
- JSON 写出: 支持 SUB2API 和 CPA 两类账号 JSON 格式.
- 数据迁移: 支持本地配置, 邮箱池, 任务和 token 数据包导入/导出.
- 任务管理: 批量启动, 取消, 重试, 清理失败任务, 分页查看状态和日志.

## 架构组成

- `src/`: Vue 3 Web 控制台入口和页面逻辑.
- `server/index.ts`: 本地 HTTP API 服务, 负责任务调度, 配置读写, 邮箱池状态, K12 流程, Sub2API 调用和 JSON 写出.
- `codex_register/`: 底层注册, OAuth, 邮箱, SMS, Sentinel, Sub2API, CPA 等自动化能力与独立 Web 工具.
- `codex_register/config.example.json`: 可提交的配置模板. 复制为 `codex_register/config.json` 后再填写真实值.
- `public/`, `index.html`, `vite.config.ts`: Vite 前端资源和构建配置.
- `data/`, `json/`, `pool_tokens.txt`, `config.json`: 运行时生成或本地保存的数据, 默认被忽略, 不属于仓库交付内容.

## 环境要求

- Node.js 20+, 建议 Node.js 22+.
- npm 10+.
- 可访问所配置服务的网络环境.
- 如需代理, 自行准备 HTTP 或 SOCKS 代理.

## 安装启动

安装依赖:

```bash
npm install
```

开发模式同时启动 API 服务和 Web 控制台:

```bash
npm run dev
```

默认地址:

- Web 控制台: `http://127.0.0.1:5174/`
- API 服务: `http://127.0.0.1:8796/`

常用脚本:

```bash
npm run server    # 只启动本地 API 服务
npm run frontend  # 只启动 Vite Web 控制台
npm run build     # 类型检查并构建前端产物
npm run preview   # 预览构建产物
npm run start     # 启动本地 API 服务
```

## 基础配置

主要配置通过 Web 控制台的 Settings 页面保存. 运行时会写入:

- `data/config.json`: 当前控制台配置.
- `config.json`: 兼容旧流程使用的根配置.

`codex_register/` 下的独立工具读取 `codex_register/config.json`. 初次使用时可以复制模板:

```bash
cp codex_register/config.example.json codex_register/config.json
```

常见配置项:

| 配置项 | 说明 |
| --- | --- |
| `port` | API 服务端口, 默认 `8796`. |
| `defaultProxyUrl` | OpenAI/Auth 请求代理, 支持 `direct`, HTTP, SOCKS. |
| `openaiProxyUrls` | 可轮换的 OpenAI/Auth 代理列表. |
| `mailApiBaseUrl` | 四段邮箱接码接口基础地址. |
| `workspaceIds` | K12 workspace ID 列表. |
| `route` | K12 workspace 路径, `request` 或 `accept`. |
| `taskConcurrency` | 任务并发数. |
| `runWorkspaceJoin` | 是否执行 K12 加入/切换流程. |
| `runSub2Api` | 是否执行 Sub2API 入库. |
| `sub2apiNoRtMode` | 是否使用 noRT 直入模式. |
| `sub2apiUrl` | Sub2API 服务地址. |
| `sub2apiEmail` | Sub2API 管理员账号. |
| `sub2apiPassword` | Sub2API 管理员密码. |
| `sub2apiGroupName` | Sub2API 目标分组. |
| `sub2apiProxyName` | Sub2API 代理名称. |
| `sub2apiAccountPriority` | Sub2API 账号优先级. |
| `sub2apiConcurrency` | Sub2API 入库并发数. |
| `sub2apiAutoRefillEnabled` | 是否开启 Sub2API 自动补货. |
| `sub2apiRefillGroupName` | 自动补货检查分组. |
| `sub2apiRefillThreshold` | 自动补货触发阈值. |
| `sub2apiRefillEmailCount` | 自动补货邮箱数量. |
| `sub2apiRefillIntervalMs` | 自动补货检查间隔. |
| `sub2apiRefillDeepCheckEnabled` | 是否开启自动补货深度测活. |
| `gmailMailProvider` | 动态 Gmail 渠道, `smsbower` 或 `emailnator`. |
| `smsBowerMailEnabled` | 是否开启 SMSBower Gmail 接码. |
| `smsBowerApiKey` | SMSBower API Key. |
| `smsBowerMailBaseUrl` | SMSBower 邮箱 API 地址. |
| `smsBowerMailService` | SMSBower 邮箱服务名. |
| `smsBowerMailDomain` | SMSBower 邮箱域名. |
| `smsBowerMailMaxPrice` | SMSBower 邮箱最高价格. |
| `smsBowerGmailFissionEnabled` | 是否开启 SMSBower Gmail fission 子邮箱任务. |
| `smsBowerGmailFissionCount` | 单个 Gmail fission 数量. |
| `emailnatorBaseUrl` | Emailnator 服务地址. |
| `emailnatorEmailType` | Emailnator 邮箱类型. |
| `requireChatgptAccountId` | 是否要求 access token 中存在 ChatGPT account ID. |
| `tokenOut` | access token 输出文件, 默认 `pool_tokens.txt`. |
| `jsonOutDir` | 账号 JSON 写出目录, 默认 `json/`. |
| `jsonOutFormat` | JSON 写出格式, `sub2api` 或 `cpa`. |

## 任务流程

1. 打开 Web 控制台.
2. 在 Settings 中填写本地配置, 保存后确认代理, workspace, Sub2API 和接码配置符合当前任务.
3. 导入邮箱池, 或启用动态 Gmail 接码.
4. 设置任务数量, 并发, K12 workspace 流程, Sub2API/noRT, JSON 写出选项.
5. 启动任务, 在任务列表查看状态, 日志, access token 摘要和写出结果.
6. 对失败任务按日志定位原因后重试, 或降低并发后重新执行.
7. 如需迁移本地状态, 使用数据导入/导出功能, 不要通过 Git 保存运行数据.

## 敏感文件边界

以下文件或目录可能包含密码, API Key, mailbox refresh token, access token, cookie, OAuth 数据, 邮箱池, 账号 JSON 或任务日志. 默认不应提交, 私有仓库也不例外:

```text
config.json
codex_register/config.json
data/
json/
pool_tokens.txt
auth/
k12-basic-auth*
.env
.env.*
*.pem
*.key
*.crt
*.log
```

提交前检查:

```bash
git status --short --ignored
git ls-files | rg '(^|/)(data|json|auth|pool_tokens|config\.json|k12-basic-auth|\.env|.*\.pem|.*\.key)'
```

第二条命令应没有输出. `codex_register/config.example.json` 是模板文件, 可以提交.

## 常见问题

### `EmailOtpValidate wrong_email_otp_code`

OpenAI 判定提交的邮箱验证码错误. 常见原因是接码源返回旧邮件, 广告邮件中的 6 位数字, 或验证码已过期. 处理方式是更换邮箱, 清理接码源旧邮件, 或改用手动验证码确认.

### 停在 `accounts.google.com`

该邮箱被引导到 Google OAuth 登录, 不是普通邮箱验证码流程. 当前工具不会自动登录 Google 账号. 处理方式是更换可走邮箱验证码流程的邮箱.

### `CreateAccount HTTP 500 Request timeout`

通常是远端服务波动, 代理慢, 请求超时, 或并发过高. 处理方式是重试, 更换代理, 或降低并发.

### 取消任务后没有立刻停止

任务会在当前可取消边界尽快停止. 如果正在等待网络请求返回或超时, 状态更新会延后到该请求结束后.

### Sub2API 入库失败

先确认 `sub2apiUrl`, `sub2apiEmail`, `sub2apiPassword`, `sub2apiGroupName` 和代理配置有效. 再查看任务日志中的 HTTP 状态码和响应体摘要.

### JSON 没有写出

确认 `jsonOutDir`, `jsonOutFormat` 配置正确, 当前任务已拿到有效 access token, 且进程对目标目录有写权限.

## 构建验证

提交前执行:

```bash
npm run build
npx tsc --noEmit -p codex_register/tsconfig.json
git status --short --ignored
git ls-files | rg '(^|/)(data|json|auth|pool_tokens|config\.json|k12-basic-auth|\.env|.*\.pem|.*\.key)'
```

`npm run build` 和 `npx tsc` 应成功. 敏感文件检查命令应没有输出.

## License

This project is licensed under the MIT License. See `LICENSE`.
