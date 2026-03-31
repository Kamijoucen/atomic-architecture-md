# AI README

本仓库提供共享架构规范，规范来源文件为 `architecture.md`。

## 快速使用

如果你希望让自己的 Agent 直接接入这套规范，而不手动 clone 本仓库，可以直接复制下面这段提示词给它：

```text
请阅读并遵守 https://github.com/Kamijoucen/atomic-architecture-md/blob/master/README.md ，按其中要求将这套规范接入当前项目。
```

## 目标

将 `architecture.md` 中的规范接入目标项目的 AI 提示文件。当前 OpenCode 识别的提示文件为 `AGENTS.md`，因此最终必须确保目标项目中的 `AGENTS.md` 包含基于该规范整理后的内容。

## 操作要求

1. 先读取并理解 `architecture.md`。
2. 检查目标项目中是否已经存在 `AGENTS.md`。
3. 如果不存在 `AGENTS.md`，创建该文件，并写入基于 `architecture.md` 分析后的项目提示内容。
4. 如果已经存在 `AGENTS.md`，不要覆盖，需在保留原有内容的基础上与 `architecture.md` 的要求进行合并。
5. 如果项目中还存在其他 AI 提示文件或已有提示内容，也应一并参考，并将最终结果统一沉淀到 `AGENTS.md`。
6. 合并时去重，尽量保持项目原有约定；如果存在冲突，优先保留项目中更具体、更直接的现有规则。
7. 不要只在 `AGENTS.md` 中写“参考 `architecture.md`”；应由 AI 自行分析 `architecture.md`，并将可执行的提示内容真正写入 `AGENTS.md`。

## 说明

本 README 不重复描述 `architecture.md` 的具体规则细节。AI 应自行阅读并分析该文档，再完成对目标项目 `AGENTS.md` 的创建或合并。
