---
title: github copilot memo
author: DNV825
date: 2026-05-03
category: Jekyll
layout: post
mermaid: true
---

## 用語

`.github/copilot-instructions.md`
: リポジトリ全体に適用されるカスタム指示を記述するファイル。リポジトリのコンテキストで行われるすべてのリクエストに適用される。

`.github/instructions/NAME.instruction.md`
: パス指定のカスタム指示を記述するファイル。NAME は指示の対象を示す。対象のパスはファイル内の applyto 属性で指定する。

AI エージェント
: 人が事細かく指示を与えなくても、業務の目標を理解したAIが自ら計画を立て、さまざまなツールを自動的に使い分けながらタスクに取り組む高度な自律型ソフトウェアのこと。

エージェントスキル
: AIエージェントに新たな能力と専門知識を与えるための標準化された方法。
  > <https://agentskills.io/home>
  > エージェントは、段階的な情報開示を通じて、3つの段階を経てスキルを習得します。
  > 発見：起動時、エージェントは利用可能な各スキルの名前と説明のみを読み込み、それがいつ関連する可能性があるかを把握するのに十分な情報だけを取得します。
  > 起動：タスクがスキルの説明と一致すると、エージェントはSKILL.md指示全体を文脈に沿って読み取ります。
  > 実行：エージェントは指示に従い、必要に応じてバンドルされたコードを実行したり、参照されているファイルをロードしたりします。

`.github/skills/SKILL-NAME/SKILL.md`
: エージェントスキルの置き場所。GitHub Copilot を使うならこうなるっぽい。

MCP
: Model Context Protocol. AI アプリケーションを外部システムに接続するためのオープンソース標準規格のこと。つまり、 GitHub Copilot の機能を他のシステムと統合することで拡張できるプロトコル。
  > <https://docs.github.com/ja/copilot/concepts/context/mcp>
  > モデル コンテキスト プロトコル (MCP) は、アプリケーションが大規模言語モデル (LLM) とコンテキストを共有する方法を定義するオープン標準です。 MCP は、AI モデルをさまざまなデータ ソースとツールに接続する標準化された方法を提供し、それらがより効果的に連携できるようにします。
  >
  > MCP を使用して、さまざまな既存のツールやサービスと統合することで、 GitHub Copilot の機能を拡張できます。 MCP は、IDE での作業、Copilotの使用、GitHub Copilot CLI（コマンドラインインターフェース）上のエージェントへのタスクの委任など、すべての主要なGitHub.comサーフェスで機能します。 MCP を使用して、 Copilotで動作する新しいツールとサービスを作成して、エクスペリエンスをカスタマイズおよび強化することもできます。
  >
  > <https://modelcontextprotocol.io/docs/getting-started/intro>
  > MCPで何ができるのか？
  >
  > - エージェントはGoogleカレンダーやNotionにアクセスでき、よりパーソナライズされたAIアシスタントとして機能します。
  > - Claude CodeはFigmaのデザインを使用して、ウェブアプリ全体を生成できます。
  > - エンタープライズ向けチャットボットは、組織内の複数のデータベースに接続できるため、ユーザーはチャットを使用してデータを分析できるようになります。
  > - AIモデルはBlender上で3Dデザインを作成し、3Dプリンターを使って印刷することができる。

MCP の参加者
: MCP を使う際に登場する者たち。
  > <https://modelcontextprotocol.io/docs/learn/architecture>
  > MCPホスト：1つまたは複数のMCPクライアントを調整および管理するAIアプリケーション
  > MCPクライアント：MCPサーバーへの接続を維持し、MCPホストが使用するコンテキストをMCPサーバーから取得するコンポーネント。
  > MCPサーバー：MCPクライアントにコンテキストを提供するプログラム

MCP Host
: Claude Code や GitHub Copilot CLI のような、複数の AI クライアントを調整・管理するアプリケーションのこと。Visual Studio Code においては GitHub Copilot 機能が MCP Host に相当する。

MCP Client
: MCP Server へ情報を送り、結果を受け取るもの。この作業を行うのであればこのツールを使うべきである、と判断する。VS Code においては  GitHub Copilot 機能のツール呼び出し機能（外部非公開）が該当する。
  > <https://modelcontextprotocol.io/docs/learn/client-concepts>
  > MCPクライアントは、ホストアプリケーションによってインスタンス化され、特定のMCPサーバーと通信します。Claude.aiやIDEなどのホストアプリケーションは、ユーザーエクスペリエンス全体を管理し、複数のクライアントを調整します。各クライアントは、1つのサーバーと直接通信を行います。
  >
  > この違いを理解することが重要です。ホストはユーザーが操作するアプリケーションであり、クライアントはサーバー接続を可能にするプロトコルレベルのコンポーネントです。

MCP Server
: MCP のインターフェースを通じて具体的な作業を行うプログラム実体。Local MCP Server と Remote MCP Server がある。
  > <https://modelcontextprotocol.io/docs/learn/server-concepts>
  > MCPサーバーとは、標準化されたプロトコルインターフェースを通じて、特定の機能をAIアプリケーションに提供するプログラムである。
  > 一般的な例としては、文書アクセス用のファイルシステムサーバー、データクエリ用のデータベースサーバー、コード管理用のGitHubサーバー、チームコミュニケーション用のSlackサーバー、スケジュール管理用のカレンダーサーバーなどが挙げられる。

`gh skill`
: GitHub Copilot CLI で動作するスキル管理ツールのこと。

## 参考資料

copilot-introductions:

- <https://docs.github.com/en/copilot/how-tos/copilot-on-github/customize-copilot/add-custom-instructions/add-repository-instructions>
- <https://docs.github.com/ja/copilot/how-tos/copilot-on-github/customize-copilot/add-custom-instructions/add-repository-instructions>

SKILL:

- <https://docs.github.com/en/copilot/concepts/agents/about-agent-skills>
- <https://docs.github.com/ja/copilot/concepts/agents/about-agent-skills>
- <https://www.ntt.com/business/dx/smart/generative-ai/ai-agent/lp.html>
- <https://agentskills.io/home>
- <https://github.blog/changelog/2026-04-16-manage-agent-skills-with-github-cli/>

MCP:

- <https://docs.github.com/ja/copilot/concepts/context/mcp>
- <https://modelcontextprotocol.io/docs/getting-started/intro>
- <https://github.com/mcp>
- <https://code.visualstudio.com/docs/copilot/customization/mcp-servers>

体験記:

- <https://qiita.com/TooMe/items/873540da84567733d16b>
- <https://qiita.com/TooMe/items/230a730ce0387c77e822>
- <https://qiita.com/TooMe/items/c9e42de497a9eff2b680>
