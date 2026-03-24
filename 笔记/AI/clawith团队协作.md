# OpenClaw 团队版来了，让 AI Agent 真正进入团队协作！

最近在 GitHub 上看到一个挺有意思的项目：**Clawith**，定位是 **OpenClaw for Teams**。

如果你关注过 AI Agent，应该对 OpenClaw 不陌生——它给 Agent 加入了「灵魂」和「记忆」，让 AI 不只是聊天，而是可以执行任务、写代码、查资料、调用工具。

但团队引入 Agent 时很快发现一个问题：**个人助手好用，团队协作完全是另一回事。** Agent 不知道团队里有哪些人、任务该交给谁，更无法在多个 Agent 之间协作。

Clawith 在保留 OpenClaw 能力的基础上，重点补齐了**团队协作能力**。

---

## 从 Heartbeat 到 Aware：Agent 不再只是定时执行

OpenClaw 的 Agent 主要依赖 **Heartbeat 心跳机制**运行——按固定周期醒来检查是否有新任务。这种模式在个人自动化场景中问题不大，但在团队环境里显得有些机械。

Clawith 为此引入了 **Aware 感知系统**，核心是一个 `on_message` 触发器，允许 Agent 等待某个人或某个 Agent 的回复，收到消息后自动触发下一步任务。还允许 Agent 根据任务进展**动态创建触发器**，而不是按固定时间执行。

典型场景：

- **多人审批流程**：Agent 在不同负责人之间自动流转任务
- **跨时区协作**：根据不同地区工作时间调整沟通节奏
- **复杂项目**：协调多个角色共同推进任务

---

## 不只是 Agent 平台，还能纳管 OpenClaw Agent

Clawith 支持纳管外部运行的 **OpenClaw Agent**。很多团队会把 Agent 部署在本地或企业内网，处理本地文件或访问内部系统。Clawith 提供 Gateway 接入机制，让这些 Agent 通过 HTTP 轮询接入平台，获取任务并返回结果。

例如一个 OpenClaw Agent 执行任务时需要查询产品数据，可以向平台中的数据分析 Agent 请求帮助；更复杂的场景中，多个 Agent 可以分工协作——一个负责信息检索，另一个负责生成数据处理脚本，合力完成数据分析流程。

---

## 写在最后

未来不仅有人给 Agent 安排任务，还可能出现**项目经理 Agent、董秘 Agent**，来安排和协调人类的分工与协作。

- **OpenClaw** 解决的是 **AI Agent 如何拥有能力**
- **Clawith** 解决的是 **AI Agent 如何在团队中协作工作**

> GitHub 项目地址：https://github.com/dataelement/Clawith
