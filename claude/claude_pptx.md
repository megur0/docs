---
title: "Claude Code の pptx スキルについて - Claude"
updated: 2026-07-30
---

[TOP(About this memo))](../README.md) > [一覧(Claude)](./README.md) > Claude Code の pptx スキルについて


# Claude Code の pptx スキルについて

## pptx スキルとは

Anthropic が提供する「事前構築済み（pre-built）」Agent Skills の一つ。以下のようなことができる。

- 白紙またはテンプレートからのプレゼンテーション作成
- 既存スライドの編集・分析（レイアウト、スピーカーノート、テーマなど）
- HTML → PPTX 変換によるスライド生成、OOXML 構造の直接操作

同様の事前構築済みスキルとして以下もある。

- xlsx（Excel）
- docx（Word）
- pdf

## Claude Code に標準搭載されているか？

**いいえ、されていない。**

公式ドキュメント（Agent Skills - Claude Platform Docs）によると：

- 事前構築済みのドキュメント系スキル（PowerPoint／Excel／Word／PDF）は **Claude Code では利用できない**
- Claude Code に標準で組み込まれているのは、オープンソースの **Claude API skill**（8言語のAPIリファレンス・SDKドキュメント・ベストプラクティスを提供）
- pptx スキルとは別物

## 「anthropic-skills」との混同について

Anthropic は `anthropics/skills` という GitHub リポジトリ（公式スキル集）を公開しており、その中に以下が含まれる。

- `document-skills`（pptx / docx / xlsx / pdf を含む）
- `example-skills`
- `claude-api`

このリポジトリはマーケットプレイス形式で配布されているため「Claude Code 用に最初から入っている公式スキル集」という印象を持ちやすいが、実際は **任意でインストールするオプション** であり、Claude Code にデフォルトでバンドルされているわけではない。

## 比較表

| | Claude.ai / Claude API | Claude Code |
|---|---|---|
| pptx / xlsx / docx / pdf | 標準搭載（すぐ使える） | 標準搭載ではない（任意でカスタムスキルとして導入可） |
| Claude API skill | — | 標準搭載 |

## Claude Code で PowerPoint 作成をしたい場合

- `anthropics/skills` リポジトリから pptx スキルのソースを取得し、カスタムスキルとして導入する（例：`~/.claude/skills/` に配置）
- ただし公式には「参考・デモ目的」の提供とされており、Claude.ai / API の本番実装と動作が異なる場合がある点に注意
- サードパーティ製の同等スキルがプラグイン／マーケットプレイス経由で配布されているケースもある

## 参考情報源

- Agent Skills - Claude Platform Docs: https://platform.claude.com/docs/en/agents-and-tools/agent-skills/overview
- Skills（Managed Agents）- Claude Platform Docs: https://platform.claude.com/docs/en/managed-agents/skills
- Quickstart - Agent Skills: https://platform.claude.com/docs/en/agents-and-tools/agent-skills/quickstart
- anthropics/skills（GitHub）: https://github.com/anthropics/skills