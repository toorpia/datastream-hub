# IndustryFlow Hub (IF-HUB)

IndustryFlow Hub（IF-HUB）は、製造設備の時系列データを安全かつ柔軟に管理・提供するためのデータハブサーバーです。PI Systemをはじめとする産業用データウェアハウスの標準APIと解析サービスの間に位置するオープンソースの中間層として設計されており、各種制約により難しいサーバー側の直接的なカスタマイズを、容易かつ効率的に実現します。

具体的には、PI Web APIなどの標準的なRESTful APIから一次データストリームをbatchwiseに取得し、一時的にデータベースにキャッシュした上で、移動平均、サンプリング処理、データ補完、Zスコア化、偏差値化といった柔軟な時系列データ処理を施し、統一されたRESTful APIを通じてクライアントアプリケーションに提供します。一時データベースを保有することで、処理の安定性向上、レスポンス速度の改善、外部データソースの障害時にも一定期間データを提供し続けることが可能になります。

また、Python、Rust、Go、C/C++など、各種言語で作成された外部プログラムを取り込むことが可能で、高度なデータ処理を柔軟に拡張できます。


## 存在価値と使命

IF-HUBは以下の課題を解決します：

- **産業用データウェアハウスの制約を解消する中間層**：セキュリティや保守性の理由から、直接的なカスタマイズが困難な産業用データウェアハウスのAPIに対し、中間層として柔軟な拡張性を提供
- **時系列データの高度な加工と変換の実現**：移動平均、サンプリング、各種アルゴリズムを用いた補完処理、統計変換（Zスコア、偏差計算）などを自由に適用可能
- **統一されたデータアクセス手段の提供**：多様なバックエンドシステムからのデータ取得を標準化し、クライアント開発を効率化

## 主な機能

- **時系列データの一元管理**：複数のデータソースからの時系列データを統合して管理
- **インテリジェントなデータ取り込み**：指定フォルダ内のCSVファイルを自動的に検出・処理してDBに格納
- **柔軟なデータ加工・変換機能**：
  - **標準化されたデータ処理**：すべてのタグ（通常タグとgtag）に対して移動平均、Z-score、偏差値などの統計処理をAPI経由で直接適用可能
- **シンプルで統一されたAPIインターフェース**：RESTful設計による一貫性のあるデータアクセスを提供し、オープンソース化による自由なAPIカスタマイズも可能
- **多言語対応**：タグ名や属性の多言語表示名マッピング機能
- **仮想タグ（gtag）生成機能**：複数データの演算や組み合わせによる新たな仮想的指標を自由に作成可能
  - **簡単な定義方法**：JSONベースの定義ファイルで複雑な計算も簡単に設定
  - **多彩な計算タイプ**：単純な数式から、移動平均、Z-score、偏差値まで多様な処理に対応
  - **柔軟に拡張可能**：外部プログラム（Python、Rust、Go、C/C++等）を統合可能。容易な機能追加や拡張を実現
- **Docker対応**：コンテナ環境を活用した容易なデプロイとスケーラブルな運用を実現


## 適用シナリオ

### データの民主化と活用促進

既存の産業用データウェアハウスシステムは強力ですが、しばしば専門知識を持つ一部のユーザーだけがアクセス可能です。IF-HUBはこれらのシステムから抽出したデータをより使いやすい形で提供し、社内のより多くの部門・担当者がデータを活用できる環境を構築します。

### 既存システムの安全な拡張

本番稼働中の産業用データシステムに直接アクセスすることなく、安全なデータアクセス・加工を実現します。これにより、クリティカルなシステムに影響を与えることなく、新しい分析ツールや可視化ダッシュボードを開発できます。

### エッジデバイスとの統合

工場現場に設置されたエッジデバイスからのデータを収集・統合し、中央データシステムと連携します。これにより、リアルタイム処理と長期的なデータ分析を組み合わせた高度なアプリケーション構築が可能になります。

### 予兆保全プラットフォーム

IndustryFlow Hubを介することにより、製造設備からの様々な時系列データにIoT等から収集されるセンサーデータや画像データなどを統合可能とし、マルチモーダルかつ高度な予測システムの実現のための基盤を提供します。

### カスタムアプリケーション開発の基盤

モバイルアプリやWebダッシュボードなど、設備データを活用したカスタムアプリケーション開発のためのバックエンド基盤として機能します。統一されたAPIを通じて、フロントエンド開発を大幅に効率化します。

## アーキテクチャの設計思想

IF-HUBは、上流の産業用データウェアハウス（PI Systemなど）との統合において、**意図的に粗結合設計**を採用しています。これには以下の重要な理由があります：

1. **上流システムの仕様変更に対する耐性** - 大規模DBとの直結合では、上流の仕様変更の影響を大きく受けてしまいます。CSVファイルを介した粗結合やbatchwiseなデータ取得により、この影響を最小限に抑えています。
2. **セキュリティバウンダリの明確化** - 直接接続ではなくファイル転送やRESTful API経由のデータ交換は、セキュリティ境界を明確にし、重要なシステムへの不正アクセスリスクを軽減します。
3. **インターフェース変更の柔軟性** - 接続方式の変更や機能拡張を、上流システムに依存せず独立して行うことができます。
4. **一時キャッシュによる安定性向上** - データをローカルデータベースにキャッシュすることで、処理の安定性、レスポンス速度の改善、外部システム障害時の継続運用が可能になります。

この設計により、IndustryFlow Hubは上流システムの制約から解放され、自由度の高いデータ加工・分析機能を提供できます。加えて、拡張プロセッサを通じて多様なプログラミング言語（Python、Rust、Go、C/C++など）による高度なデータ処理も柔軟に組み込むことができます。

## パフォーマンスと拡張性への取り組み

IF-HUBでは、以下の点においてパフォーマンスと拡張性を継続的に向上させています：

- **効率的なデータ処理**：
  - 処理関数の最適化によるパフォーマンス向上
  - 大規模データセットでも効率的に動作

- **モジュール化されたデータ処理**：
  - 役割ごとに明確に分けられたモジュール構造
  - 新しいアルゴリズム追加が容易な設計

- **一貫したAPI設計**：
  - すべてのエンドポイントで一貫した処理オプションを提供
  - 汎用パラメータによる柔軟なデータ取得と処理

## データフロー

以下は、IF-HUBにおけるデータの流れを示しています：

```mermaid
graph TB
    Z[産業用データウェアハウス] -->|データエクスポート| A[設備データCSV]
    Z -->|API経由| K[クエリ処理]
    
    A -->|static_equipment_dataフォルダに配置| B[ファイル監視システム]
    G[タグメタデータCSV] -->|tag_metadataフォルダに配置| H[メタデータインポーター]
    I[gtag定義] -->|gtagsフォルダに配置| J[gtag処理]
    
    B -->|変更検出| C[CSVインポーター]
    K -->|データ取得| C
    
    C -->|キャッシュ/データ格納| D[(IF-HUB DB)]
    H -->|表示名・単位情報| D
    J -->|計算タグ| D
    
    D -->|データ取得| E[RESTful API]
    
    E -->|生データ| F[クライアント/解析サービス]
    E -->|移動平均処理| F
    E -->|Z-score計算| F
    E -->|偏差計算| F
    E -->|カスタム処理| F
```

IndustryFlow Hubは現在、主に`static_equipment_data`フォルダに設置された静的CSVファイルからのデータ取り込みをサポートしています。これは産業用データウェアハウスからエクスポートされたデータを想定しており、意図的な粗結合設計となっています。加えて、特定のユースケースに応じて産業用データシステムのAPIを利用したbatchwiseなデータ取得機能も実装を計画しており、一時キャッシュによる安定性とレスポンス速度の改善を実現していきます。将来的には、これらの機能をさらに拡張しつつも、柔軟性とシステム間の独立性を保つ設計思想を維持します。

## クイックスタート

### 前提条件

- Node.js 18以上
- npm または yarn
- （オプション）Docker & Docker Compose
- Python 3.x（カスタムgtag実装に使用する場合）

### インストール

```bash
# リポジトリのクローン
git clone https://github.com/toorpia/if-hub.git
cd if-hub

# 依存関係のインストール
npm install

# Python依存関係のインストール（カスタムgtag実装に使用する場合）
# 必要なPythonパッケージは各自のgtagの要件に応じてインストール
```

### 使用方法

```bash
# 開発サーバーの起動
npm start

# Docker環境での起動
cd docker
docker-compose up -d
```

#### 設備データの取り込みと活用方法

1. 設備データをCSV形式で準備します（サポートされている形式：タイムスタンプ列 + 複数のタグ値列）
2. CSVファイルを `static_equipment_data` フォルダに配置します
3. システムが自動的にファイルを検出し、データベースに取り込みます（サーバー起動時と1分おきに確認）
4. 取り込まれたデータはAPIを通して以下の形式で取得可能になります：
   - 生データの取得（元のデータをそのまま取得）
   - 移動平均処理（ノイズを除去したデータ取得）
   - Z-scoreの計算（異常検知のための標準化スコア）
   - 偏差計算（平均からの乖離）
   - カスタム処理（外部プロセッサによる拡張機能）

#### gtagを活用した仮想指標の作成方法

IF-HUBでは、gtagシステムを使って既存のデータから新たな仮想指標を簡単に作成できます：

1. `gtags/`ディレクトリに新しいgtagフォルダを作成：`mkdir -p gtags/Equipment01.NewMetric`
2. 定義ファイル（def.json）を作成：

   計算タイプの例：
   ```json
   {
     "name": "Equipment01.Efficiency",
     "type": "calculation",
     "inputs": ["Equipment01.Output", "Equipment01.Input"],
     "expression": "(inputs[0] / inputs[1]) * 100",
     "description": "設備効率（出力/入力 × 100）",
     "unit": "%"
   }
   ```

   カスタムタイプの例：
   ```json
   {
     "name": "Equipment01.ComplexMetric",
     "type": "custom",
     "inputs": ["Equipment01.Param1", "Equipment01.Param2"],
     "prog": "bin/process.py",
     "args": ["--option1=value", "--option2=value"],
     "description": "複雑なアルゴリズムによる複合指標",
     "unit": "単位"
   }
   ```

3. カスタムタイプの場合は、処理プログラムを`bin/`ディレクトリに配置：
   ```
   gtags/Equipment01.ComplexMetric/bin/process.py
   ```
   このプログラムは標準入力からデータを受け取り、処理結果を標準出力に出力します。Python、Ruby、Go、C/C++など様々な言語で実装可能です。

4. 追加後、最大1分以内に自動的に検出・反映されるため、サーバー再起動なしで通常のタグと同様にAPIからアクセス可能：
   ```
   GET /api/data/Equipment01.Efficiency
   ```
   
   さらに、通常のタグと同様に処理オプションも適用可能：
   ```
   GET /api/data/Equipment01.Efficiency?processing=moving_average&window=5
   ```

これにより、物理的には測定されていない新たな効率指標や複合指標を容易に生成・活用できます。

#### サポートされているCSVフォーマット

- タイムスタンプ列は「time」「date」などの名前を含むことが推奨されます
- タイムスタンプ形式：`YYYY-MM-DD HH:mm:ss`、`YYYY/MM/DD HH:mm:ss`など
- タグ名は設備IDとファイル名から自動的に生成されます（例：ファイル名が`Pump01.csv`で列名が`Flow`の場合、タグIDは`Pump01.Flow`となります）

詳細な使用方法については、[運用マニュアル](docs/ja/operations_manual.md)を参照してください。

### APIの強化ポイント

- **すべてのタグに対する処理オプション**：
  ```
  GET /api/data/{tagName}?processing=moving_average&window=10
  GET /api/batch?tags=Tag1,Tag2&processing=zscore
  GET /api/export/equipment/{equipmentId}/csv?processing=deviation
  ```

- **処理タイプ**：
  - `moving_average` - 移動平均（ノイズ除去に効果的）
  - `zscore` - Z-score（異常検知に有用）
  - `deviation` - 偏差値（相対的な位置付けを把握）

## 技術アーキテクチャ

IF-HUBは以下の技術コンポーネントで構成されています：

- **バックエンド**: Node.js + Express.js
- **データストレージ**: ローカルデータベース（デフォルト）、一時キャッシュとして機能、他DBへの拡張可能
- **データインポート**: CSVパーサー + ファイル監視、API経由batchwiseデータ取得
- **APIレイヤー**: RESTful API（JSON形式）、統一されたエンドポイント設計
- **拡張エンジン**: 外部プロセッサ対応
  - **Python**: 統計処理、機械学習、データサイエンス
  - **Rust/Go**: 高性能な計算処理、メモリ効率の高い処理
  - **C/C++**: レガシーコードの統合、極めて高速な処理
  - **その他**: カスタム言語プロセッサの追加も可能

これらのコンポーネントは疎結合な設計となっており、必要に応じて個別に拡張・置換することができます。たとえば、SQLiteをTimescaleDBに置き換えたり、新しい種類の外部プロセッサを追加したり、データの取得方法をカスタマイズすることが可能です。

## ファイル構成

IF-HUBは以下のディレクトリ構造で構成されています：

```
/
├── src/                     # メインアプリケーションのソースコード
│   ├── app.js               # Expressアプリケーション設定
│   ├── config.js            # システム設定
│   ├── db.js                # データベース接続管理
│   ├── server.js            # サーバー起動スクリプト
│   ├── routes/              # APIルート定義
│   │   ├── data.js          # データアクセスAPI
│   │   ├── equipment.js     # 設備情報API
│   │   ├── index.js         # ルート集約
│   │   ├── gtags.js         # gtag管理・処理API
│   │   ├── system.js        # システム情報API
│   │   └── tags.js          # タグ管理API
│   ├── services/            # ビジネスロジック
│   │   └── server-services.js # サーバーサービス実装
│   └── utils/               # ユーティリティ関数
│       ├── csv-importer.js  # CSVデータインポート
│       ├── data-generator.js # テストデータ生成
│       ├── data-processing.js # データ処理アルゴリズム
│       ├── file-watcher.js  # ファイル変更監視
│       ├── gtag-utils.js    # 仮想タグユーティリティ
│       ├── tag-metadata-importer.js # タグメタデータインポート
│       ├── tag-utils.js     # タグユーティリティ
│       └── time-utils.js    # 時間関連ユーティリティ
│
├── gtags/                   # 仮想タグ定義と計算スクリプト
├── docker/                  # Docker関連設定ファイル
├── docs/                    # プロジェクトドキュメント（API、開発者ガイド、運用マニュアル）
├── static_equipment_data/   # 設備データCSV格納ディレクトリ
├── tag_metadata/            # タグメタデータ格納ディレクトリ
├── db/                      # データベースファイル
└── logs/                    # ログファイル
```

### 主要ディレクトリの役割

#### src/
Node.jsベースのバックエンドアプリケーションコードが格納されています。Express.jsフレームワークを使用したRESTful APIの実装、データベース接続、gtag管理などの機能が含まれます。

#### gtags/
仮想タグ（Generated Tags）の定義と計算ロジックが格納されています。階層型ディレクトリ構造を採用し、各gtagは独自のディレクトリを持ちます：

```
gtags/
  ├── {gtag名}/             # 各gtagの独立したディレクトリ
  │   ├── def.json          # gtag定義ファイル
  │   └── bin/              # カスタム実装用（必要な場合のみ）
  │       └── custom_impl.py  # カスタム計算実装
  ├── {別のgtag名}/
  │   └── def.json
  ...
```

定義ファイル（JSON形式）には計算式や参照するタグ情報が含まれ、以下のタイプをサポートしています：
- **計算タイプ**：数式による演算（例：`(A + B) / 2`）
- **移動平均タイプ**：指定ウィンドウでの移動平均を計算
- **Z-scoreタイプ**：統計的な異常検知に有用なZ値を計算
- **偏差タイプ**：平均値からの偏差を計算
- **カスタムタイプ**：Pythonスクリプト等による複雑な処理

この柔軟な設計により、ユーザーは既存データから多様な派生指標を簡単に生成できます。

#### static_equipment_data/
設備データCSVファイルを配置するディレクトリです。このディレクトリに配置されたCSVファイルは自動的に検出され、データベースにインポートされます。

#### tag_metadata/
タグの名称、単位、表示名などのメタデータが格納されたCSVファイルを配置するディレクトリです。多言語対応や単位変換のための情報が含まれます。

#### docker/
Dockerコンテナを使用したデプロイメント設定が含まれます。開発環境および本番環境向けのDocker Compose設定が用意されています。

#### docs/
プロジェクトのドキュメントが格納されています。API利用方法、開発者向けガイド、運用マニュアルなどが提供されています。

## ドキュメント

- [運用マニュアル](docs/ja/operations_manual.md) - インストール、設定、運用の詳細
- [APIマニュアル](docs/ja/api_manual.md) - APIエンドポイントの詳細と使用例
- [開発者ガイド](docs/ja/developer_guide.md) - アーキテクチャ、コード詳細、拡張方法

## データベースの選択と拡張性

IF-HUBは現在SQLiteをデータストレージとして使用しています。これは小〜中規模の導入に適していますが、より大規模なデプロイメントでは、以下のエンタープライズグレードのデータベースへの移行も検討できます：

1. **TimescaleDB** - PostgreSQLの拡張として、高度な時系列機能を提供
   - **長所**: PostgreSQLの拡張であるため、SQLの知識がそのまま活用できる。標準SQLのみならず時系列特有の機能も充実。
   - **適用例**: 既存のSQLスキルを活用しつつ、時系列データの拡張性を求めるケース

2. **InfluxDB** - 時系列データに特化した高性能データベース
   - **長所**: IoTや監視向けに特化した時系列データベース。高い書き込みパフォーマンスと効率的なストレージ。
   - **適用例**: 大量のセンサーデータを高速に取り込む必要があるケース

3. **QuestDB** - SQLインターフェースを持つ高速な時系列データベース
   - **長所**: 極めて高速なクエリ実行と低いリソース消費。SQLサポートにより学習曲線が緩やか。
   - **適用例**: リアルタイム分析や高速クエリが重要なケース

今後のリリースで、これらのデータベースへのプラグイン方式でのサポートを追加する予定です。

## 将来の展望

IF-HUBは継続的に進化するプロジェクトであり、以下の機能拡張を計画しています：

- **API連携の拡充** - より多様な産業用データシステムとのAPIベース連携機能（OPC UA、MQTT、PI Web API等）
- **キャッシュ最適化** - 一時キャッシュのパフォーマンス最適化とデータ保持ポリシーの柔軟な設定
- **リアルタイム処理** - streambased処理オプションの追加によるリアルタイムデータ変換
- **インタラクティブな可視化** - 簡易なデータ探索・分析ダッシュボード

これらの機能は、ユーザーニーズとコミュニティフィードバックに基づいて優先順位を決定し、段階的に実装していく予定です。

## ライセンスについて

このプロジェクトは Business-Friendly License (BFL) のもとで提供されており、2028年3月20日（リリース日から3年後）に Apache License 2.0 に移行します。

現在のライセンス：Business-Friendly License (BFL)
商用利用：商用環境での利用には、別途ライセンス契約が必要です。(実証実験・PoCなど社内検証用途にはご自由にお使いいただけます。)
ライセンス移行：2028年3月20日以降、このプロジェクトは Apache License 2.0 のもとで自由に利用可能になります。
詳細については、[LICENSE](./LICENSE) ファイルをご参照ください。

## 貢献

貢献は歓迎します！バグレポート、機能提案、プルリクエストなど、あらゆる形式の貢献に感謝します。大きな変更を加える前に、まずIssueでディスカッションを開始してください。

[貢献ガイドライン](CONTRIBUTING.md)もご覧ください。
