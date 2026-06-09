# astra-skill-pre-action-research

在执行操作前强制调研与查索引的 Hermes Agent skill。涉及排查、凭证、偏好、设备、服务、项目信息时，必须查阅文档、教程与参考索引再动手。

## 功能

- 自动触发：任务涉及排查/调研/凭证/偏好等关键词时命中
- 三阶段检查：信息类型确认 → 相关文档查阅 → 方案批准
- 防坑提醒：不凭记忆操作、不以"新旧"为由跳过索引

## 安装

将 `SKILL.md` 复制到 Hermes profile 的 `skills/` 目录下：

```bash
cp SKILL.md ~/.hermes/profiles/default/skills/pre-action-research.md
```

## License

MIT — 详见 [LICENSE](LICENSE)
