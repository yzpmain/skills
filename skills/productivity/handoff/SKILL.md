---
name: handoff
description: 把当前对话压缩成交接文档，让另一个 agent 接续工作。
argument-hint: "下一个会话将用于什么？"
disable-model-invocation: true
---

写一个交接文档总结当前对话，让一个新 agent 能继续工作。保存到用户 OS 的临时目录——不是当前工作区。

在文档中包含一个"建议 skills"段落，建议 agent 应该调用哪些 skill。

不要重复已在其他工件（specs、plans、ADRs、issues、commits、diffs）中捕获的内容。改为通过路径或 URL 引用它们。

脱敏任何敏感信息，如 API 密钥、密码或个人身份信息。

如果用户传了参数，把它们当作下一个会话将专注的内容描述，并据此定制文档。
