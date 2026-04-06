# ハーネスエンジニアリング：Planner-Generator-Evaluator自律開発パターン

## 概要

ハーネスエンジニアリングは、Claude Codeのサブエージェント機能を活用して、Planner（計画）・Generator（生成）・Evaluator（評価）の3つの役割を担うAIエージェントを連携させることで、人間の介入なしに自律的にアプリケーション開発を完結させる次世代開発パラダイムです。このパターンでは、単一のAIモデルに頼るのではなく、役割分担と評価ループの設計が開発成功の鍵となります。

## 主要な知見

- **役割分担による品質向上**：製造業の多段階検査プロセスと同じ発想で、AIエージェントも計画・実行・評価の3段階に分離することで品質が飛躍的に向上する。単一のAIより複数の専門役割が相互チェックする仕組みが差別化要因となる

- **自動改善ループの重要性**：ランニングのトレーニング管理と同様に、AIシステムも「目標→実行→評価→調整」の改善ループが自動で回ることで初めて高い成果が得られる。人間が放置可能な仕組みを設計できるかが他との競争優位を決定する

- **キャリア転換期における市場価値**：エンジニアのキャリアが[コード職人からAIマネジャーへ](ai-manager-role-transition-code-craftsman.md)移行する過渡期の今、ハーネス設計の理解度が市場価値を大きく左右する。既存のコーディングスキルをどう再構築するかが急務であり、早期適応者が大きなアドバンテージを得る

## コア概念

### Planner エージェント

Plannerは全体的な開発戦略を立案する責任を持ちます。要件分析、アーキテクチャ設計、タスク分解、優先順位付けなどを実行し、開発の羅針盤となります。[指示設計の3要素フレームワーク](instruction-design-three-elements.md)に従い、背景・目的・期待アウトプット形式を明確にすることが重要です。

### Generator エージェント

Generatorはplannerの計画に基づいて、実際のコード生成、リソース作成、機能実装を行います。[Claude Codeの役割](claude-code-agent-teams.md)そのものであり、生成スピードと反復性を重視します。生成の際は[設定駆動ワークフロー](claude-code-configuration-driven-workflow.md)に従い、CLAUDE.mdの設計規約を自動遵守することが品質担保につながります。

### Evaluator エージェント

Evaluatorはgeneratorの出力を検査・評価し、要件充足度、品質基準、パフォーマンス、セキュリティ等を多角的に検証します。フィードバックループを通じて改善指示を出し、目標達成まで反復させます。[エージェントハーネスの長期連続運用](agent-harness-reliability-framework.md)における誤り蓄積対策と同じ思想で、継続的な監視と修正ナレッジの蓄積が重要です。

## 実装設計のポイント

### 評価ループの自動化

ハーネスエンジニアリングの成功は、Evaluatorが継続的に動作し、改善指示を自動でGeneratorに返す仕組みにかかっています。[プロンプト明確性とマネジメント](prompt-clarity-management-feedback-loop.md)を高めることで、AIフィードバックループの精度が向上します。

### 記憶システムの構築

長期にわたる開発プロジェクトでは、[Claude Codeの長期記憶システム設計](claude-long-term-memory-design.md)が必須です。CLAUDE.md + auto memoryの実装パターンにより、プロジェクト進行状況、変更履歴、学習内容を累積させることで、迷走を防ぎ、効率的な自動化を実現できます。

### Skillシステムの活用

複雑な開発タスクは、[Skill System](skill-system.md)を活用してモジュール化します。[SKILL.md仕様](skill-md-specification.md)に従い、フォルダ単位でClaudeへの命令セットを整理することで、Planner・Generator・Evaluatorが明確に役割を実行できるようになります。

### 統治とガバナンス

[CLAUDE.md統治](claude-md-governance.md)の概念を応用し、AIエージェントチームへの経営判断基準を明文化します。日次改善ループにより、ハーネス自体の品質を継続的に向上させることができます。

## 製造業での応用

ハーネスエンジニアリングの発想は[製造業のAI活用](manufacturing-ai-opportunities.md)にも直接応用可能です。品質管理プロセスをAIに自動化させる際、単一のAIより複数の検査段階（計画→実行→評価）を組み込むことで、誤り蓄積を防ぎ、信頼性の高い自動化システムが構築できます。

[マルチエージェントのタスク依存関係管理](multi-agent-task-dependency-management.md)により、複数のAIエージェント間の調整と競合解消も自動化され、製造業システム間の統合が効率化されます。

## 人間の役割の変化

ハーネスエンジニアリングが浸透することで、エンジニアの役割は大きく転換します。従来の「コードを書く職人」から、[AIマネジャー](ai-manager-role-transition-code-craftsman.md)へのシフトが避けられません。

具体的には以下の能力が求められるようになります：

- ハーネス（AI3役エージェント）の設計・構築能力
- AIエージェント間の協働フローの設計
- 評価基準・フィードバック仕組みの明文化
- 継続的改善ループの運用

これは従来のソフトウェア開発スキルの再構築を意味し、早期適応者が市場で圧倒的優位を得る機会です。

## 「放置可能性」の重要性

このパターンの最大の特徴は、設計後、人間が直接手を下さずにシステムが自動で改善し続ける点です。[バックグラウンド自動化設計](background-automation-design-competitive-advantage.md)の考え方と同様に、待機時間や運動時間の活用を自動化するのと同じ発想で、開発プロセス全体を自動化できます。

正しくハーネスを設計すれば、人間は：
- 初期要件の定義
- ハーネスの監視・改善

に専念でき、日々のコード生成・テスト・修正は完全にAIに委ねられます。これが「放置OK」と言われる由縁です。

## 関連ページ

- [Claude Code Agent Teams：AIエージェントチームの実装と活用](claude-code-agent-teams.md)
- [エージェントハーネス：長期連続運用における誤り蓄積対策と制御・監視基盤](agent-harness-reliability-framework.md)
- [Agentic Engineeringの監督者モデル：直接実行から検証・調整へのシフト](agentic-engineering-supervisor-model.md)
- [AIエージェント失敗ログと修正ナレッジ](ai-failure-log.md)
- [Claude Codeの長期記憶システム設計：CLAUDE.md + auto memoryの実装パターン](claude-long-term-memory-design.md)
- [CLAUDE.md統治：AIへの経営判断基準の明文化と日次改善ループ](claude-md-governance.md)
- [Skill System：フォルダ単位のClaudeへの命令セット](skill-system.md)
- [SKILL.md仕様：ファイル命名ルールとフォルダ構造](skill-md-specification.md)
- [指示設計の3要素フレームワーク：背景・目的・期待アウトプット形式](instruction-design-three-elements.md)
- [Claude Code設定駆動ワークフロー：CLAUDE.mdの設計規約自動遵守と保守業務の並列化](claude-code-configuration-driven-workflow.md)
- [プロンプト明確性とマネジメント：AIフィードバックループによるスキル向上](prompt-clarity-management-feedback-loop.md)
- [コード職人からAIマネジャーへ：エンジニアキャリア転換期の市場価値設計](ai-manager-role-transition-code-craftsman.md)
- [バックグラウンド自動化設計：運動時間・待機時間の活用による他者差別化戦略](background-automation-design-competitive-advantage.md)
- [製造業のAI活用機会：電話・FAX・スプレッドシート業界の変革](manufacturing-ai-opportunities.md)
- [マルチエージェントのタスク依存関係管理：製造業システム間の自動調整と競合解消](multi-agent-task-dependency-management.md)
- [ドメイン専門知識とAIの境界設計：人間が設計、AIが実行する分業モデル](domain-expertise-ai-boundary-design.md)
- [AIオーケストレーター：100倍エンジニアの役割](ai-orchestrator-role.md)
- [Hook設計パターン：SessionEnd・PreCompact・PreToolUseの活用法](hook-design-patterns-ai-workflow.md)

## 更新履歴

- 2026-04-06: [【放置OK】Claude Codeハーネス設計で自律開発するハーネスエンジニアリング入門！](https://www.youtube.com/watch?v=Wfz-gdWcItM)より初期作成