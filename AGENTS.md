# AGENTS.md

## 项目概览

电子硬件笔记（中文），纯 markdown 文档 + 图片资源，无构建系统或代码。

## 目录结构

- `components/` — 按功能分类的元器件笔记（markdown）
- `image/` — 笔记中引用的芯片引脚图等资源

## 写作约定

- 语言：中文
- 图片引用使用相对路径，如 `../image/ne555_1.png`
- 使用 GitHub 风格的 `> [!NOTE]` callout 做注释说明
- commit message 格式：`类型(范围) : 简要描述`（如 `feat(ne555) : base add`）
- 类型使用 feat / fix

## 注意事项

- 无测试、无 lint、无构建步骤
- 图片存放在 `image/` 目录，markdown 中通过相对路径引用
