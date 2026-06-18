# astra-skill-pre-action-research

Mandatory research and index lookup before taking action. When investigation, credentials, preferences, devices, services, or project information is involved, documentation, tutorials, and reference indexes must be consulted before proceeding.

## Features

- Auto-triggered: activated when keywords related to investigation, research, credentials, preferences, etc. are detected
- Three-stage check: information type confirmation → documentation review → solution approval
- Pitfall reminders: don't act from memory alone, don't skip the index for "new" or "old" content

## Install

Copy `SKILL.md` to your Hermes profile's `skills/` directory:

```bash
cp SKILL.md ~/.hermes/profiles/default/skills/pre-action-research.md
```

## Dependencies

| Repository | Resource | Required | Purpose |
|:-----------|:---------|:--------:|:--------|
| [astra-aiagent-infra](https://github.com/alrcatraz/astra-aiagent-infra) | `docs/credential-schema.md` | Optional | Credential storage conventions for the credential-check step |

## License

MIT — see [LICENSE](LICENSE).

---

## 中文版

### 功能

- 自动触发：任务涉及排查/调研/凭证/偏好等关键词时命中
- 三阶段检查：信息类型确认 → 相关文档查阅 → 方案批准
- 防坑提醒：不凭记忆操作、不以"新旧"为由跳过索引

### 安装

将 `SKILL.md` 复制到 Hermes profile 的 `skills/` 目录下：

```bash
cp SKILL.md ~/.hermes/profiles/default/skills/pre-action-research.md
```

### 依赖关系

| 仓库 | 资源 | 必须 | 用途 |
|:-----|:-----|:----:|:-----|
| [astra-aiagent-infra](https://github.com/alrcatraz/astra-aiagent-infra) | `docs/credential-schema.md` | 可选 | 凭证检查步骤的凭证存储规范参考 |
