---
name: rate-limit-reset-credits
description: 使用本机 Codex 凭证查询 ChatGPT/Codex 的 rate-limit reset credits。用于用户询问重置次数、重置次数到期时间、重置额度、reset credits、rate-limit reset credits、available_count、可用重置次数、重置 quota/credit 何时过期等场景。
---

# 重置次数查询

## 执行流程

使用此 Skill 查询当前机器上的 ChatGPT/Codex rate-limit reset credits 及到期时间。

优先在本 Skill 目录运行内置脚本：

```bash
python3 scripts/check_rate_limit_reset_credits.py
```

脚本会读取 `~/.codex/auth.json`，只提取 `tokens.access_token`，然后请求：

```text
https://chatgpt.com/backend-api/wham/rate-limit-reset-credits
```

脚本只输出允许展示的中文表格汇总信息。

## 安全要求

- 不要打印 `access_token`、`refresh_token`、cookie、`Authorization` header 或完整唯一 ID。
- 不要把原始 `~/.codex/auth.json` 内容贴到对话里。
- 不要打印原始接口响应，除非已经裁剪为下方允许字段。
- 只汇总：
  - `available_count`
  - 每个 credit 的 `status`、`granted_at`、`expires_at`
- 输出必须使用中文标签和表格形式；常见状态和值也尽量翻译成中文。
- 将 `granted_at` 和 `expires_at` 从 UTC 转成本机本地时区后再展示。
- 如果接口返回 HTTP 401，说明本机凭证失效，或请求没有携带有效的 `Authorization` header。

## 手动兜底流程

如果无法使用脚本，按相同规则手动执行：

1. 以程序方式读取 `~/.codex/auth.json`，只提取 `tokens.access_token`。
2. 请求 `GET https://chatgpt.com/backend-api/wham/rate-limit-reset-credits`，header 使用：

```text
Authorization: Bearer <access_token>
Accept: application/json
```

3. 解析 JSON，只输出 `available_count` 以及每个 credit 的 `status`、`granted_at`、`expires_at`。
4. 将带 `Z`、`+00:00` 或没有时区的时间当作 UTC，再转换为本地时间。
5. 遇到 401 时，不要通过打印凭证来排查。直接用中文说明“凭证失效或 Authorization header 无效/缺失”。

## 输出格式

使用简洁的中文输出，并将明细展示为 Markdown 表格：

```text
可用重置次数：<数量>

| 序号 | 状态 | 发放时间 | 到期时间 |
| --- | --- | --- | --- |
| 1 | <状态> | <本地时间> | <本地时间> |
```

如果字段不存在，写 `无`，不要展示额外响应字段。