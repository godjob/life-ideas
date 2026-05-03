# 段階的LLM実行パターン：大規模モデルで計画、軽量ローカルモデルで実装する効率化設計

## 概要

段階的LLM実行パターンは、複数のLLMモデルの特性を活用して全体的なコスト効率と性能を最適化する設計手法である。大規模クラウドベースモデル（Claude等）で戦略的な計画・設計・検証を行い、軽量なローカルモデル（Gemma等）で繰り返し実行される実装タスクを処理することで、API費用の削減とセキュリティリスクの低減を同時に実現できる。この階層的なアプローチにより、組織は高性能と経済効率のバランスを取ることができる。

ただし、[Google DeepMind CEOのデミス・ハサビス氏による現状分析](agi-architecture-limitation-continuous-learning-long-horizon-reasoning.md)において、現在のLLMアーキテクチャは継続的学習・長期推論・記憶能力が制限されていることが指摘されている。このため、段階的実行パターンの導入は、~~複雑な業務全体の自動化～~ → **明確に区切られたタスクに限定し、段階的に拡大する現実的な戦略**が求められる。

## 主要な知見

- **二層型モデル運用の経済効率性**: クラウドベースの高性能モデルをプランニング・アーキテクチャ設計・検証フェーズに限定し、実装や定期実行タスクにはローカル軽量モデルを活用することで、API呼び出し費用を大幅削減できる。特に製造業のように繰り返し実行が多い業務では、固定コストで済むオンプレミス運用の利点が大きい。[Claude API レート制限最適化](claude-api-rate-limit-optimization-workflow.md)の実践例でも実証されている通り、正しいモデル選択プロセスにより、頻繁なAPI制限への到達を根本的に回避できる。

- **プロセス設計が効率化の本質**: ~~単なるモデル選択~~ → **事前計画→検証→実行というワークフロー設計が、トークン消費削減の本質**である。軽量モデルで事前計画を立てた上で高級モデルを使用するアプローチにより、API呼び出しを3週間以上制限に達しないレベルまで削減可能。セキュリティと運用の自立性向上**: ローカルLLMをオンプレミス環境で実行することで、データをクラウドに送信せずに処理でき、組織のセキュリティポリシー遵守が容易になる。同時にAPI依存性を排除することで、外部サービス障害時の影響を最小化し、システム運用の自立性を確保できる。

- **役割分担による実行品質の向上**: 大規模モデルは複雑な意思決定や予期しない状況への対応に優れ、軽量モデルは明確に定義された反復タスク処理に最適化されている。この役割分担を明示的に設計することで、各モデルの強みを活かしながら全体的な効率を高められる。[AGI実現に向けたアーキテクチャ制限](agi-architecture-limitation-continuous-learning-long-horizon-reasoning.md)を踏まえると、~~将来的には汎用性を前提とした設計～~ → **現在はタスク領域を明確に限定し、そこで最適なモデルを選択する戦術的設計**が現実的である。

- **Agentic Toolsによる開発者の役割シフト**: Claude CodeなどのAgentic Toolの進化により、開発者はコード直接実装から監督・検証・意思決定の役割へシフトしている。段階的実行パターンは、この[Agentic Engineeringの監督者モデル](agentic-engineering-supervisor-model.md)を実装の層面で具体化したものである。

- **学習戦略の再評価**: 複数のLLMモデルを組み合わせる運用では、各モデルの能力境界・特性・制限事項に関する継続的な学習が不可欠である。[AI評価の月次リフレッシュサイクル](ai-tool-evaluation-monthly-refresh-cycle.md)のように、指数関数的に進化するAI能力への追従が個人・組織の競争力を左右する。

- **現在のAI技術的制限の認識**: [デミス・ハサビス指摘](agi-architecture-limitation-continuous-learning-long-horizon-reasoning.md)により、現在のLLMは『問題解決』には優れているが『創造性』『自己内省』『継続的な自己改善』はまだ発展段階であることが明確化された。このため、段階的実行パターンの適用範囲は、~~既存プロセスの完全自動化～~ → **既存プロセスの効率化補助に最適であり、抜本的な業務革新には人間の判断が引き続き必須となる**という現実的な位置づけが必要である。

## パターンの構成要素

### 計画・設計フェーズ（大規模クラウドモデル）

このフェーズでは、クロード等の高性能モデルが以下の役割を担う：

- **アーキテクチャ設計**: システム全体の構造、データフロー、エラーハンドリング戦略の検討
- **複雑な意思決定**: 複数の選択肢の比較検討、トレードオフ分析、リスク評価
- **検証・レビュー**: ローカルモデルで生成されたコードやアウトプットの品質確認
- **例外処理の設計**: 予期しない状況への対応方法の事前検討

この段階での投資は、下流フェーズでの効率化によって回収される。大規模モデルの能力を計画に集中させることで、トータルなAPI費用を最小化できる。

### 実装・実行フェーズ（軽量ローカルモデル）

軽量ローカルモデル（[Gemma等](gemma-llm-model-selection-manufacturing.md)、Mistrals等）が以下のタスクを処理する：

- **定型的なコード生成**: 事前に設計された仕様に基づくルーチンコード作成
- **定期実行ジョブ**: ログ確認、データ処理、リポート生成など繰り返されるタスク
- **テンプレート適用**: 確立されたパターンに基づくアウトプット生成
- **単純な問題解決**: 明確に定義された範囲内での修正・改善

[ローカルLLMデプロイメント・アーキテクチャ](local-llm-deployment-architecture.md)で詳述されるOllamaやOpenClawを活用することで、これらの処理をオンプレミス環境で実行できる。

## 実装パターンの具体例

### 製造業における適用例

製造業のシステム運用では、段階的実行パターンが特に効果的である：

1. **計画フェーズ**: Claude等でシステム改善計画、エラーハンドリング戦略、権限設計を検討
2. **実装フェーズ**: Gemma等でメンテナンススクリプト、ログ解析、データ変換処理を生成・実行
3. **監視フェーズ**: 軽量モデルが日常的に問題を検出し、重大な異常は大規模モデルで対応

この構成により、[AIエージェントのCLI自律操作](ai-agent-cli-automation-pattern.md)で記述される自動メンテナンスが低コストで実現される。ただし、[現在のアーキテクチャ制限](agi-architecture-limitation-continuous-learning-long-horizon-reasoning.md)を踏まえ、軽量モデルが直面した予期しない状況は即座に人間に報告する設計が必要である。

### Web開発における適用例

[AI時代のWeb制作フロー](ai-powered-web-production-flow.md)では、段階的実行が以下のように機能する：

1. **計画フェーズ**: クラウドモデルがUI/UX要件、API設計、セキュリティ戦略を検討
2. **実装フェーズ**: ローカルモデルがコンポーネント生成、ボイラープレートコード作成、単体テスト生成
3. **検証フェーズ**: 大規模モデルが全体的な整合性、ベストプラクティス遵守を確認

このパターンにより、開発速度と品質を両立できる。

### 製造業における要件定義での適用

[AIDLC in 製造業](aidlc-manufacturing-requirements-definition.md)の要件定義フェーズでは、この段階的実行が仕様齟齬防止に効果を発揮する：

1. **軽量モデルで初期要件整理**: Gemmaで既存システムドキュメント・カタログ情報から基本仕様案を抽出
2. **高級モデルで要件レビュー**: Claudeで複雑な業務要件との整合性・リスク要因を洗い出し
3. **軽量モデルで仕様書作成**: 承認済み要件に基づき、定型的な仕様書ドキュメントを生成

## コスト削減メカニズム

段階的実行が生み出すコスト削減は、複合的に作用する：

| 削減要素 | 効果 | 実装方法 |
|---------|------|---------|
| **API呼び出し削減** | 実装フェーズでローカル実行により80-90%のコール削減 | 定型タスク自動化 |
| **トークン消費削減** | 軽量モデルは大規模モデルの5-10倍のコスト効率 | 実行フェーズの層別化 |
| **インフラ固定化** | ローカル実行で予測可能なコスト構造を実現 | オンプレミス投資 |
| **レイテンシ低減** | ネットワーク遅延排除でユーザー体験向上 | [エッジAIシステム設計](edge-ai-system-design-production-skills.md) |

[AI予算管理とROI最適化](ai-budget-management-roi-optimization.md)の観点からは、この階層的アプローチにより運用コストの可視化と予測が容易になる。API制限への到達を避けるプロセス設計自体が、予測不可能なコスト爆発を防止する重要な経営判断基準となる。

## セキュリティと規制適応

大規模モデルと軽量ローカルモデルの分離は、セキュリティ要件への適応も容易にする：

- **データ局所化**: 機密データはローカルで処理、通用可能なデータのみクラウド送信
- **監査ログ**: オンプレミス実行により実行記録の完全な管理が可能
- **権限制御**: [AIエージェント権限モデル](ai-agent-permission-model-least-privilege.md)を各層で独立的に実装

特に[製造業におけるQMS様式のAIプロンプト統治](qms-style-ai-prompt-governance.md)では、ローカル実行による監査可能性が重要な要件である。

## 開発者スキル要件の変化

段階的LLM実行パターンの導入は、[AIエンジニア実践スキルロードマップ](ai-engineer-practical-skills-roadmap.md)に新たな要素を加える：

1. **モデル選択スキル**: 各フェーズに最適なモデルを見極める能力。[現在のアーキテクチャ制限](agi-architecture-limitation-continuous-learning-long-horizon-reasoning.md)を理解し、タスク領域を明確に限定する判断
2. **アーキテクチャ思考**: 計画と実行の分離を効果的に設計する能力
3. **オンプレミス運用**: Ollama等のローカルLLM運用環境の構築・保守
4. **ハイブリッド統合**: クラウドとローカルの連携インターフェース設計
5. **エッジAIスキル**: [エッジAIシステム設計](edge-ai-system-design-production-skills.md)による本番環境のメモリ最適化とハードウェア制約実装

[Agentic Engineeringの監督者モデル](agentic-engineering-supervisor-model.md)での記述通り、開発者は「直接実行から検証・調整へ」シフトする。段階的実行パターンはこのシフトを組織的に実装するメカニズムとなる。

## 組織導入のステップ

### 段階1：試験的導入

- 定型的で影響が限定されるタスクから開始
- ローカルLLMの能力を検証
- [Skill System](skill-system.md)で実装パターンを標準化

### 段階2：拡張と最適化

- 成功したパターンを他のタスクに展開
- [Claude Code Agent Teams](claude-code-agent-teams.md)で複数エージェント間の連携を構築
- [マルチエージェントパイプラインのエラーハンドリング](multi-agent-pipeline-error-handling-checkpoint.md)により信頼性を向上

### 段階3：組織的統合

- [組織全体のAI導入加速](organizational-ai-adoption-acceleration-topdown-directive.md)で全部門への展開
- [CLAUDE.md統治](claude-md-governance.md)による意思決定基準の明文化
- 継続的な学習と改善ループの制度化

## 課題と対策

### 課題1：モデル間の能力ギャップ

軽量モデルが計画フェーズの意図を完全に理解できないリスク。[現在のアーキテクチャ制限](agi-architecture-limitation-continuous-learning-long-horizon-reasoning.md)が示すように、継続的学習や長期推論が限定的であるため、このギャップは今後もリスク要因である。

**対策**: [Claude Long-term Memory設計](claude-long-term-memory-design.md)や[CLAUDE.md自動育成](claude-md-auto-cultivation-hooks.md)のような明示的な文脈保持メカニズムにより、大規模モデルの意思決定を継続的に軽量モデルに伝達する。明確に区切られたタスク領域にのみ軽量モデルを適用する原則を徹底する。

### 課題2：品質管理と検証

ローカル実行の結果が予期した水準を満たさないリスク。

**対策**: [Skillテスト戦略](skill-testing-strategy.md)により、各実装タスクの正確性を段階的に検証。定期的な大規模モデルによるレビュー（[Multi-session Lifecycle管理](multi-session-lifecycle-continuous-monitoring.md)）で品質を確保。

### 課題3：運用負荷

ローカルLLMの運用と保守が追加負荷となるリスク。

**対策**: [AIマネージドサービス設計](ai-managed-service-operational-design.md)のプリンシパルを採用し、監視・ロールバック機能を最初から組み込む。[Claude Managed Agents](claude-managed-agents-cloud-deployment.md)等のマネージドサービスの活用も検討。

### 課題4：根本的な解決策の不在

軽量モデルが直面した予期しない状況や、[AIが根本的に『創造性』『自己内省』を欠くこと](ai-creativity-self-reflection-limitation-human-judgment-necessity.md)に対して、段階的実行パターンだけでは根本的解決ができない。

**対策**: ~~AIの完全自動化を前提とした設計～~ → **人間の判断介入を常時設計に含める。軽量モデルが対応不可な状況は即座に人間に報告する仕組み、複雑な創造的判断は大規模モデル+人間で検討する構成**を採用。定期的に[AI能力進化への対応設計](ai-capability-evolution-design-flexibility.md)を見直す。

## 指数関数的進化への適応

AI技術の急速な進化は、段階的実行パターンの最適構成を常に変化させる。[AI評価の月次リフレッシュサイクル](ai-tool-evaluation-monthly-refresh-cycle.md)のように、定期的にモデル選択と役割分担を見直す仕組みが必須である。

例えば：
- 軽量モデルの性能が劇的に向上した場合、より複雑なタスクをローカル実行に移行可能
- 新しいモデルアーキテクチャが出現した場合、コスト効率の再計算が必要
- API価格の変動により、クラウド/ローカルの費用バランスが変動
- [DeepMindの指摘](agi-architecture-limitation-continuous-learning-long-horizon-reasoning.md)するような継続的学習・長期推論が実装されたモデルが登場した場合、タスク領域の拡張が可能

[AI能力進化への対応設計](ai-capability-evolution-design-flexibility.md)の観点から、過度なカスタマイズを避け、抽象度の高い設計を心がけることが重要である。

## 関連ページ

- [段階的AIモデル活用の3フェーズ設計](tiered-ai-model-planning-verification-execution.md): 軽量モデル計画→中堅モデル検証→高級モデル実行の具体的な3段階フロー
- [Claude API レート制限最適化](claude-api-rate-limit-optimization-workflow.md): 段階的モデル選択によるトークン効率化ワークフロー
- [Agentic Engineeringの監督者モデル](agentic-engineering-supervisor-model.md): 直接実行から検証・調整へのシフト
- [ローカルLLMデプロイメント・アーキテクチャ](local-llm-deployment-architecture.md): Ollama等のオンプレミスLLM運用基盤
- [Gemma LLMモデル選択](gemma-llm-model-selection-manufacturing.md): 製造業における段階導入戦略と軽量・重量モデルの使い分け
- [エッジAIシステム設計](edge-ai-system-design-production-skills.md): エッジデバイスのメモリ最適化と本番運用スキル
- [AI予算管理とROI最適化](ai-budget-management-roi-optimization.md): 段階的アプローチによるコスト可視化と予測
- [Claude Long-term Memory設計](claude-long-term-memory-design.md): クラウド-ローカル間の文脈保持メカニズム
- [CLAUDE.md統治](claude-md-governance.md): AIへの経営判断基準の明文化
- [Skill System](skill-system.md): フォルダ単位の命令セット標準化
- [Claude Code Agent Teams](claude-code-agent-teams.md): 複数エージェント間の連携設計
- [マルチエージェントパイプラインのエラーハンドリング](multi-agent-pipeline-error-handling-checkpoint.md): 信頼性高い自動化の実装パターン
- [AIエージェント権限モデル](ai-agent-permission-model-least-privilege.md): 最小権限原則とセキュリティ設計
- [製造業におけるQMS様式のAIプロンプト統治](qms-style-ai-prompt-governance.md): 監査可能な運用フレームワーク
- [AI評価の月次リフレッシュサイクル](ai-tool-evaluation-monthly-refresh-cycle.md): 指数関数的進化への継続的対応
- [AIエンジニア実践スキルロードマップ](ai-engineer-practical-skills-roadmap.md): 実装に必要なコアスキル6つ
- [AIDLC in 製造業](aidlc-manufacturing-requirements-definition.md): 要件定義フェーズでの仕様齟齬防止
- [AI時代のWeb制作フロー](ai-powered-web-production-flow.md): 設計・実装・デザインの流れの最適化
- [AIエージェントのCLI自律操作](ai-agent-cli-automation-pattern.md): 定期実行ジョブの自動化パターン
- [AGI実現に向けたアーキテクチャ制限](agi-architecture-limitation-continuous-learning-long-horizon-reasoning.md): 継続的学習・長期推論・記憶の課題
- [AIの『創造性』『自己内省』限界と人間判断の必須性](ai-creativity-self-reflection-limitation-human-judgment-necessity.md): 現在のAI技術的制限の認識
- [AI能力進化への対応設計](ai-capability-evolution-design-flexibility.md): 柔軟なアーキテクチャと過度なカスタマイズ回避

## 更新履歴

- 2026-05-03: [Xユーザーの尾原＠Back to obaraさん: 「AIの進化, AIx事業方向性が凝縮、YコンでのGemini, DeepMind デミス・ハサビス 日本語訳」](https://x.com/kazobara/status/2050389094786863532) に基づき、DeepMindのアーキテクチャ制限（継続的学習・長期推論・記憶）と、AI技術の『問題解決優位性』『創造性・自己内省限界』についての新知見を統合。タスク領域の明確な限定と人間判断の必須性を強調する記述に更新