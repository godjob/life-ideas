# 現代AIエンジニアのコア6スキル：LLM API・プロンプト・ツールコール・RAG・デプロイメント

## 概要

機械学習理論よりも「実際に動くものを作る6つのスキル」に集中することが、AIエンジニアとしての最短成長経路である。LLM API、プロンプト設計、ツールコール、RAG、デプロイメント、そしてエンドツーエンド実装体験を積むことで、チュートリアル沼や理論偏重から脱却し、実務的価値を生み出すエンジニアとなる。

実装の各段階では、[書籍メタデータ補完システム](book-metadata-completion-system.md)のような複数APIの段階的フォールバック設計など、実際のシステム構築知見を活かすことが重要である。また、[プロンプト最適化によるコスト効率化](prompt-optimization-cost-efficiency.md)を通じて、単なる機能実装だけでなく経済性を意識した設計が差別化要因となる。さらに、[ドキュメント形式変換とMarkdown標準化](document-format-conversion-markdown-standardization.md)など、企業データの多形式ドキュメント統一化によるAI分析効率向上も、実運用での競争優位を生み出す重要な実装パターンである。

## 主要な知見

- **機械学習理論よりも実装重視**: 従来の大学的ML理論学習より、エンドツーエンドで動くプロダクトを早期に構築する体験設計が重要
- **現代AIエンジニアの6つのコアスキル**:
  1. **LLM API の使い方** - OpenAI、Anthropic等のAPIを効率的に呼び出し、コスト最適化と応答品質を両立させる。複数APIの段階的フォールバック設計（例：OpenBD→NDL Search API→Google Books）による信頼性確保も重要。[プロンプト最適化によるコスト効率化](prompt-optimization-cost-efficiency.md)で示されるCavemanテクニックなど、トークン削減による月単位での大幅なコスト削減が実現可能
  2. **プロンプト設計** - [指示設計の3要素フレームワーク](instruction-design-three-elements.md)に基づいた効果的な指示文の作成。[プロンプト明確性とマネジメント](prompt-clarity-management-feedback-loop.md)によるAIフィードバックループの活用。トークン浪費を削減する設計が、実装品質とコスト効率の両面で重要になっている
  3. **ツールコール（Function Calling）** - [MCPとSkillの役割分担：ツール連携とワークフロー設計](mcp-skill-architecture.md)や[Skill System：フォルダ単位のClaudeへの命令セット](skill-system.md)で示されるようなLLMから外部ツール呼び出しの実装。[複数API統合](book-metadata-completion-system.md)を通じた信頼性設計。エージェントタスク実行時の軽量化を意識することで、処理速度とコストの二重メリットを得られる
  4. **RAG（Retrieval Augmented Generation）** - 企業データを活用した検索拡張生成による精度向上。[ドキュメント形式変換とMarkdown標準化](document-format-conversion-markdown-standardization.md)により、PDF・Word・Excel等の複数形式資料をMarkdown化して、統一されたナレッジベースを構築することが基盤となる。[LLM統合による技術資料自動化](llm-integration-technical-documentation-automation.md)で実装される設備仕様書やマニュアルの自動活用も、RAGの実務的応用例として重要
  5. **デプロイメント・運用** - [AIエージェントの運用展開：検品・在庫管理・受発注の自動化](ai-agent-operations.md)で示される実運用環境での検品・改善・スケーリング。[AI予算管理とROI最適化](ai-budget-management-roi-optimization.md)による隠れたコスト削減ポイントの発掘
  6. **エンドツーエンド体験** - 要件定義から本番運用まで一気通貫した実装経験

- **ドキュメント統一化による効率向上**: 製造業などの実務環境では、技術資料・マニュアル・設備仕様書が複数の形式で散在しがちである。Microsoftの[markitdown](https://github.com/microsoft/markitdown)のようなツールを活用し、Office形式やPDFをMarkdown化することで、AI分析やテキスト検索の効率が大幅に向上する。さらに、MCPサーバーを統合することで、Claudeなどのエージェントが直接、統一されたMarkdown形式の資料にアクセスでき、問題診断やメンテナンス計画立案の自動化が実現可能になる。環境軽量化を意識したオプション機能グループの設計も、本番環境運用での重要な考慮事項

- **コスト意識の統合**: LLM API呼び出しの効率化は単なる技術課題ではなく、企業の経営効率に直結する重要なスキル。不要な説明や自問自答を排除するプロンプト工夫により、月500ドル以上の削減も実現可能。自動化ワークフローでは人間が読まない出力にこそ圧縮の余地があり、エージェントタスクでの軽量化は[AI予算管理とROI最適化](ai-budget-management-roi-optimization.md)の観点から優先すべき改善項目
- **チュートリアル沼の排除**: Kaggle競技や理論学習に時間を使わず、実際のビジネス課題に即座に適用する方針
- **[Agentic Engineering](agentic-engineering-supervisor-model.md)への接続**: 単なる成功例の構築から、検証・調整・改善ループの実装へシフト
- **[skill-creator スキル](skill-creator-automation.md)との組み合わせ**: AIエンジニアが継続的にスキルセットを洗練させるための自動化フレームワーク
- **非エンジニアの巻き込み**: [非エンジニアがAIと共にツールを育てる](non-engineer-ai-tool-development.md)ことで、6スキルの実装が組織全体に展開可能になり、AIの価値創造が加速する

## 実装上の注意点

各スキルは独立せず相互に連携する。特に複雑なシステム構築では、プロンプト設計とツールコールの組み合わせ、RAGとデプロイメントの統合が重要となる。また、単発のプロジェクトで終わらず、反復的な改善と運用経験の蓄積が真のコア能力へと昇華させる鍵となる。

重要な実装方針として、**コスト効率を初期段階から設計に組み込むこと**が推奨される。トークン削減テクニックやプロンプト工夫は事後的な最適化ではなく、LLM API呼び出し設計時点で考慮すべき要素である。これにより、スケール時のコスト爆増を防ぎ、AIシステムの実務的な採用可能性を大幅に高められる。

同時に、**ドキュメント形式の統一化**は、RAGの精度向上とメンテナンス性向上の両面で早期投資が重要である。複数形式が混在した状態でのAI分析は、トークン消費の無駄やテキスト抽出エラーにつながるため、導入初期段階でMarkdown化基盤を整備することが、長期的な運用コスト削減を実現させる。

## 関連ページ

- [書籍メタデータ補完システム](book-metadata-completion-system.md): 複数APIの段階的フォールバック設計による信頼性確保の実例
- [指示設計の3要素フレームワーク](instruction-design-three-elements.md): 効果的な指示文作成の基本フレームワーク
- [プロンプト明確性とマネジメント](prompt-clarity-management-feedback-loop.md): AIフィードバックループによるスキル向上
- [MCPとSkillの役割分担：ツール連携とワークフロー設計](mcp-skill-architecture.md): LLMから外部ツール呼び出しの実装パターン
- [Skill System：フォルダ単位のClaudeへの命令セット](skill-system.md): スキルシステムの体系的な設計
- [AIエージェントの運用展開：検品・在庫管理・受発注の自動化](ai-agent-operations.md): 実運用環境での検品・改善・スケーリング
- [AI予算管理とROI最適化](ai-budget-management-roi-optimization.md): 隠れたコスト削減ポイントと運用効率化
- [プロンプト最適化によるコスト効率化](prompt-optimization-cost-efficiency.md): トークン削減テクニックとコスト管理戦略
- [Agentic Engineering](agentic-engineering-supervisor-model.md): 検証・調整・改善ループの実装へのシフト
- [skill-creator スキル](skill-creator-automation.md): Skill設計・レビュー・改善の自動化
- [非エンジニアがAIと共にツールを育てる](non-engineer-ai-tool-development.md): 組織全体へのAI導入展開
- [ドキュメント形式変換とMarkdown標準化](document-format-conversion-markdown-standardization.md): 複数形式資料の統一化によるAI分析効率向上
- [LLM統合による技術資料自動化](llm-integration-technical-documentation-automation.md): 設備仕様書・マニュアルの問題診断とメンテナンス計画自動化

## 更新履歴

- 2026-04-09: [markitdown/README.md at main · microsoft](https://github.com/microsoft/markitdown/blob/main/README.md)の情報をもとに、ドキュメント形式変換とMarkdown標準化のセクションを追加