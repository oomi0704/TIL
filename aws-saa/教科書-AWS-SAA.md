# AWS SAA-C03 オリジナル教科書

> 学習メモ（2026-05-11 〜 05-20）を統合した個人用まとめ  
> 試験で「何を選ぶか」を素早く判断するための教科書

---

## 目次

1. [試験の考え方](#1-試験の考え方)
2. [コンピューティング](#2-コンピューティング)
3. [ストレージ](#3-ストレージ)
4. [データベース](#4-データベース)
5. [ネットワーク・VPC](#5-ネットワークvpc)
6. [コンテンツ配信・DNS・加速](#6-コンテンツ配信dns加速)
7. [セキュリティ・IAM・コンプライアンス](#7-セキュリティiamコンプライアンス)
8. [疎結合・サーバーレス・ストリーム処理](#8-疎結合サーバーレスストリーム処理)
9. [コンテナ・CI/CD](#9-コンテナcicd)
10. [ハイブリッド・移行・DR](#10-ハイブリッド移行dr)
11. [監視・運用・コスト](#11-監視運用コスト)
12. [キーワード早見表・混同ペア](#12-キーワード早見表混同ペア)

---

## 1. 試験の考え方

### SAAで問われること

- **「何を使うか」** が分かっていること（SAPは「なぜその構成か」まで深い）
- 問題文の **キーワード**（SFTP、Webhook、コンプライアンス、既存Kafka など）に反応する
- **コスト vs 可用性 vs 運用負荷** のバランスを要件から読み取る

### 判断の基本フロー

```
要件を読む
  → 制約（コスト・レイテンシ・既存技術・コンプラ）を抽出
  → 候補を2〜3個に絞る
  → 「よりマネージド」「より要件に直結」な方を選ぶ
```

---

## 2. コンピューティング

### 2.1 EC2 購入オプション

| オプション | 向いている用途 |
|---|---|
| **オンデマンド** | 短期・不定期・予測不能 |
| **リザーブド（RI）** | 1〜3年の安定利用。RDSはRIが中心 |
| **Savings Plans** | 一定金額コミット。EC2・Lambda・Fargate。AWS推奨 |
| **スポット** | 中断可能なバッチ・コスト最優先 |

- **RI vs Savings Plans**: EC2はSavings Plans推奨。RDSはRI。
- **Convertible RI**: インスタンスタイプ変更可だが割引率は下がる

### 2.2 EC2 Fleet

複数インスタンスタイプ・購入オプションを一括調達。

| 配分戦略 | 内容 |
|---|---|
| **lowest-price** | 最安プールから調達 |
| **diversified** | 複数プールに分散 |
| **capacity-optimized** | 中断リスク最小化（**試験頻出**） |

### 2.3 Auto Scaling ポリシー

| ポリシー | 概要 |
|---|---|
| ターゲット追跡 | CPU等を目標値に保つ |
| ステップ | 段階的にスケール |
| スケジュール | 時間ベースで事前スケール |

### 2.4 スケーリング用語

| 用語 | 意味 |
|---|---|
| スケールアウト / イン | 台数を増やす / 減らす |
| スケールアップ / ダウン | サイズを大きく / 小さく |

### 2.5 ELB

| 種類 | 用途 |
|---|---|
| **ALB** | HTTP/HTTPS、パスベースルーティング |
| **NLB** | TCP/UDP、超低レイテンシ |
| **GLB** | サードパーティ仮想アプライアンス |

### 2.6 インスタンスストア vs EBS

| | インスタンスストア | EBS |
|---|---|---|
| 永続性 | ❌ 一時的 | ✅ 永続的 |
| 速度 | 非常に速い | 標準 |
| EC2停止後 | データ消失 | データ残存 |
| 用途 | キャッシュ・一時ファイル | OS・DB・重要データ |

- EBSは **単一AZ** 内。基本は **1 EC2** にアタッチ（Multi-Attachは io1/io2 のみ）

### 2.7 SLA（可用性の目安）

| SLA | 年間ダウンタイム目安 |
|---|---|
| 99.5%（シングルAZ） | 約43.8時間 |
| 99.9% | 約8.7時間 |
| 99.95%（マルチAZ） | — |
| 99.99% | 約52.6分 |

---

## 3. ストレージ

### 3.1 S3 / EBS / EFS の使い分け

| サービス | 種類 | 用途 |
|---|---|---|
| **S3** | オブジェクト | 静的ファイル・バックアップ・大容量 |
| **EBS** | ブロック | 単一EC2のディスク（OS・DB） |
| **EFS** | ファイル（NFS） | 複数EC2/Linux共有。Pod共有にも |
| **FSx Windows** | ファイル（SMB） | Windows・AD連携 |
| **FSx Lustre** | 高速ファイル | HPC・ML・S3連携 |

### 3.2 S3 ストレージクラス

#### 判断の順序

1. 取り出しは **即時** か、**待てる** か
2. アクセス頻度は **高い / 低い / 不明**
3. **コスト** と **取得料金** のどちらを優先するか
4. **複数AZ** が必要か（One Zone-IA は1AZのみ）

#### クラス一覧

| クラス | 特徴 |
|---|---|
| **Standard** | 高頻度・即時・汎用 |
| **Standard-IA** | 低頻度・即時。最小30日。取得料あり |
| **One Zone-IA** | 1AZ・さらに安い。再生成可能データ向け |
| **Intelligent-Tiering** | パターン不明。自動階層移動。**取得料無料** |
| **Glacier Instant Retrieval** | アーカイブだが即時。最小 **90日** |
| **Glacier Flexible Retrieval** | 分〜時間で取得 |
| **Glacier Deep Archive** | 最安。取得は長時間（12時間級） |

#### 試験での典型シナリオ

| シナリオ | 答え |
|---|---|
| アクセスパターンが不明 | **Intelligent-Tiering** |
| 取得料金を払いたくない | **Intelligent-Tiering** |
| 月1回程度・即時取得 | **Standard-IA** |
| 年1〜2回・即時取得 | **Glacier Instant Retrieval** |
| 最安・ほぼ取り出さない | **Glacier Deep Archive** |
| **コンプライアンス監査**（S3） | **Standard-IA**（定期的な義務＝頻度が自然と高い） |

> Standard-IA vs Glacier Instant: IAは月1回程度、Glacier Instantは四半期1回程度のイメージ

#### S3 頻出機能

- **Lifecycle**: Standard → IA → Glacier へ自動移行・期限削除
- **Versioning**: 上書き・削除ミス対策（CRRの前提）
- **CRR**: リージョン間レプリケーション（DR・コンプラ）
- **Object Lock**: WORM・改ざん防止・監査

### 3.3 S3 転送の高速化

| サービス | 用途 |
|---|---|
| **Transfer Acceleration** | S3への **アップロード** 高速化（エッジ経由） |
| **CloudFront** | S3からの **ダウンロード** 高速化 |
| **Global Accelerator** | EC2・ALBへのアクセス高速化・固定IP |

### 3.4 EBS ボリュームタイプ

| タイプ | 用途 | 試験メモ |
|---|---|---|
| **gp3** | 汎用・デフォルト | IOPSとサイズ独立。gp2より約20%安 |
| **gp2** | 汎用（旧） | 3 IOPS/GB とサイズ連動 |
| **io1 / io2** | 高IOPS DB | Multi-Attach対応。io2は耐久性99.999% |
| **io2 Block Express** | 超大規模DB | 最大256,000 IOPS |
| **st1** | スループットHDD | ログ・ビッグデータ。**ブート不可** |
| **sc1** | コールドHDD | 最安・低頻度。**ブート不可** |

- **IOPS** = 1秒あたりの読み書き回数（DB向き）
- **スループット** = MB/s（大容量転送向き）
- **io3は存在しない**
- **Multi-Attach**: io1/io2のみ。同一AZ・最大16台・クラスタFS必須
- **RAID0**: 性能最大化・冗長性なし / **RAID1**: ミラー・冗長性あり

#### gp2 vs gp3 / io1 vs io2

| 比較 | ポイント |
|---|---|
| gp3 vs gp2 | gp3はIOPS独立・コスパ良・推奨 |
| io2 vs io1 | io2は5ナイン耐久・IOPS/GB比率が高い |
| 64,000 IOPS必要 | io1 または io2 |

### 3.5 SPA・フロントエンド

- **SPA**: 初回HTML読込後、JSで画面更新（Gmail等）
- 典型構成: **S3 + CloudFront + API Gateway + Lambda**
- **Amplify**: React/Vue等を素早くデプロイ（S3・CF・HTTPS・CI/CD自動）

---

## 4. データベース

### 4.1 DB選択フローチャート

```
SQLが必要？
├── Yes → リレーショナル
│    ├── コスト重視 → RDS
│    ├── 高性能・高可用性 → Aurora
│    └── 変動トラフィック → Aurora Serverless
└── No → NoSQL
     ├── Key-Value大規模 → DynamoDB
     ├── ドキュメント → DocumentDB
     ├── グラフ → Neptune
     ├── 時系列 → Timestream
     └── 台帳 → QLDB
```

### 4.2 RDS

| 機能 | 目的 |
|---|---|
| **Multi-AZ** | **可用性**（同期レプリ・フェイルオーバー） |
| **リードレプリカ** | **読み取り性能**（非同期・別エンドポイント） |
| **クロスリージョンリードレプリカ** | 別リージョンへの読み取り |

- シングルAZ: SLA 99.5% / マルチAZ: 99.95%

### 4.3 Aurora

| 項目 | RDS | Aurora |
|---|---|---|
| 対応DB | 5種 | MySQL/PostgreSQL互換のみ |
| ストレージ | 単一AZ EBS | 3AZ×2=6コピー自動 |
| レプリカ | 最大5 | 最大15 |
| フェイルオーバー | 1〜2分 | **30秒以内** |
| 性能 | 標準 | MySQL最大5倍・PG最大3倍 |

**Aurora独自機能**

- **Aurora Serverless**: アクセス量に応じ自動スケール（v2は本番も可）
- **Global Database**: 複数リージョン・遅延1秒以内・最大5リージョン
- **Backtrack**: 過去時点へ巻き戻し（MySQL互換）

| 選択 | 向き |
|---|---|
| Aurora（通常） | 安定トラフィック・常時起動 |
| Aurora Serverless | 予測不能・開発環境 |

### 4.4 DynamoDB

| モード | 向き |
|---|---|
| **オンデマンド** | トラフィック予測不能 |
| **プロビジョニング** | 安定トラフィック・コスト削減 |

| インデックス | 内容 |
|---|---|
| **GSI** | 別パーティションキーで検索 |
| **LSI** | 同一パーティションキー・別ソートキー |

### 4.5 キャッシュ（超重要）

| 要件 | 答え |
|---|---|
| DynamoDB読み取り高速化・**アプリ変更最小** | **DAX**（μs級） |
| RDS・汎用キャッシュ | **ElastiCache Redis** |
| シンプル・水平スケール | **Memcached** |
| セッション・ランキング | **Redis** |

| | DAX | ElastiCache Redis |
|---|---|---|
| 対象 | DynamoDBのみ | 何でも |
| アプリ変更 | ほぼ不要 | 実装必要 |
| 永続化 | ❌ | ✅ |

**2段階キャッシュ（頻出構成）**

```
ユーザー → CloudFront → EC2/ECS → Redis → RDS
         （配信）              （DB負荷軽減）
```

### 4.6 バックアップ・復元

| 種類 | サービス | 保存期間 |
|---|---|---|
| **PITR** | DynamoDB / RDS / Aurora | 最大 **35日** |
| 自動スナップショット | RDS / Redshift | 最大35日 |
| 手動スナップショット | EBS / RDS / Redshift | **無制限** |
| **AWS Backup** | 各種 | 自由設定（統合管理） |

| | PITR | スナップショット |
|---|---|---|
| 復元粒度 | 秒単位 | 取得時点のみ |
| 35日より前 | ❌ PITR不可 → **手動スナップショット** |

---

## 5. ネットワーク・VPC

### 5.1 AWSの地理的概念

| 用語 | 内容 |
|---|---|
| **リージョン** | 地理的エリア（東京・バージニア等） |
| **AZ** | リージョン内の独立DC群（低レイテンシ専用線で接続） |
| **ローカルゾーン** | 特定都市の低レイテンシ拠点 |

### 5.2 VPC 基本コンポーネント

| リソース | 役割 |
|---|---|
| **IGW** | VPCとインターネットの接続 |
| **NAT Gateway** | プライベートサブネットからのアウトバウンド（IPv4） |
| **Egress-Only IGW** | IPv6の送信専用（インバウンド拒否） |
| **ルートテーブル** | サブネット単位のルーティング |

### 5.3 セキュリティグループ vs NACL

| 項目 | セキュリティグループ | NACL |
|---|---|---|
| 適用 | ENI（インスタンス） | サブネット |
| デフォルト | インバウンド全拒否 | 全許可 |
| ステート | **ステートフル** | **ステートレス** |
| ルール | 許可のみ | 許可・拒否 |
| 評価 | 全ルール | **番号順（小さい優先）** |

**ステートレスの注意**: 戻りトラフィックも **別ルールで許可**（エフェメラルポート 1024-65535）

| シナリオ | 答え |
|---|---|
| 特定IPを **拒否** | NACL（SGは拒否不可） |
| インスタンス単位 | SG |
| VPC全体の高度フィルタ | **AWS Network Firewall** |

### 5.4 ENI

- 仮想NIC。プライベートIP・EIP・MAC・SGを保持
- **特定AZに紐づく**。1インスタンスに複数ENI可能

### 5.5 接続方式マップ

| 接続 | サービス |
|---|---|
| オンプレ ↔ AWS（インターネット） | **Site-to-Site VPN** |
| オンプレ ↔ AWS（専用線） | **Direct Connect** |
| オンプレ個人PC ↔ AWS | **Client VPN** |
| 複数拠点 ↔ AWS経由 | **VPN CloudHub** |
| VPC ↔ VPC（少数・1対1） | **VPC Peering** |
| VPC多数・オンプレ混在 | **Transit Gateway** |
| VPC ↔ AWSサービス（S3等） | **VPCエンドポイント** |
| 自社サービスを他VPCへ公開 | **PrivateLink**（+ NLB） |

#### Direct Connect vs Site-to-Site VPN

| 項目 | Direct Connect | Site-to-Site VPN |
|---|---|---|
| 回線 | 専用線 | インターネット |
| 速度・安定 | 高い | 変動 |
| 暗号化 | 別途必要 | IPsec自動 |
| 導入 | 数週間〜数ヶ月 | 数時間〜数日 |
| コスト | 高い | 安い |

- **最強構成**: Direct Connect + VPN（専用線＋暗号化バックアップ）
- **VPC Peering**: 推移的ルーティング **不可**（A↔B、B↔C でも A→C 不可）
- **Transit Gateway**: 推移的ルーティング **可**
- **Site-to-Site VPN**: VPC間接続には **不適切**（オンプレ専用）

### 5.6 VPCエンドポイント

| 種類 | 対象 | 料金 | オンプレから |
|---|---|---|---|
| **ゲートウェイ型** | S3・DynamoDBのみ | **無料** | 不可 |
| **インターフェース型** | ほぼ全AWSサービス | 有料 | **可**（DX/VPN経由） |

| シナリオ | 答え |
|---|---|
| S3へプライベート・コスト最小 | ゲートウェイ型 |
| CloudWatchへプライベート | インターフェース型 |
| オンプレからAWSサービスへ | インターフェース型 |
| 外部インターネットAPI | **NAT Gateway**（PrivateLinkでは不可） |

---

## 6. コンテンツ配信・DNS・加速

### 6.1 CloudFront vs Route 53 vs Global Accelerator

```
ユーザーが example.com にアクセス
① Route 53  → 名前解決（道案内）
② CloudFront → キャッシュ配信（荷物を届ける）
③ Global Accelerator → TCP/UDP経路最適化（固定IP・低遅延）
```

| | CloudFront | Global Accelerator | Route 53 |
|---|---|---|---|
| 役割 | CDN・キャッシュ | 経路最適化 | DNS |
| キャッシュ | あり | なし | — |
| 固定IP | なし | あり（Anycast 2つ） |
| プロトコル | HTTP/HTTPS中心 | **TCP/UDP** | DNS |
| 向き | 静的・動画・SPA | ゲーム・VoIP・API・**IoT(MQTT)** | ルーティング制御 |

- **IoT → Global Accelerator**（MQTTはTCP/UDP・キャッシュ不要・低遅延）
- **マルチリージョン**: オリジンフェイルオーバー、S3マルチリージョンアクセスポイント
- 動的コンテンツのマルチリージョン → Route 53と組み合わせ

### 6.2 Route 53 ルーティングポリシー

| ポリシー | 内容 |
|---|---|
| シンプル | 1つのIPを返す |
| レイテンシーベース | 最も遅延が少ないリージョンへ |
| 地理的（Geolocation） | ユーザーの地域で振り分け |
| 加重（Weighted） | 割合で分散 |
| フェイルオーバー | 障害時にスタンバイへ |

### 6.3 WAF・Shield・Geo制限

| シナリオ | 答え |
|---|---|
| 映像配信×特定国制限のみ | **CloudFront Geo Restriction**（追加コストなし） |
| 国制限＋他ルール | WAF + Geo Match |
| SQLインジェクション等 | **WAF** |
| DDoS | CloudFront + WAF + **Shield** |

> Geo MatchはWAFの機能（単独では使えない）

---

## 7. セキュリティ・IAM・コンプライアンス

### 7.1 IAM 基本

| 要素 | 内容 |
|---|---|
| **ポリシー** | 権限の定義 |
| **ロール** | サービス・ユーザーに付与する権限セット |
| **AssumeRole（STS）** | 一時認証情報で別ロールを引き受ける |

**STSで発行される情報**: AccessKeyId / SecretAccessKey / SessionToken（15分〜12時間）

| AssumeRole API | 用途 |
|---|---|
| AssumeRole | AWSユーザー/ロール |
| AssumeRoleWithWebIdentity | Google・Cognito等 |
| AssumeRoleWithSAML | Active Directory等 |

**ユースケース**: クロスアカウント、EC2からのアクセス（キー埋め込み回避）、外部IdP連携

### 7.2 ポリシーの種類と評価順

| 種類 | 内容 |
|---|---|
| アイデンティティベース | ユーザー/グループ/ロールに付与 |
| リソースベース | S3バケットポリシー等 |
| 信頼ポリシー | 誰がロールを引き受けられるか |
| Permission Boundary | 権限の上限 |
| SCP | Organizations全体の上限 |

**評価順**

1. 明示的 **Deny** → 即拒否
2. SCPで許可？
3. リソースベースで許可？
4. Permission Boundaryで許可？
5. アイデンティティで許可？
6. デフォルト Deny

### 7.3 秘匿情報・暗号化

| サービス | 用途 |
|---|---|
| **KMS** | 暗号化キー管理 |
| **Secrets Manager** | DBパスワード等・**自動ローテーション** |
| **SSM Parameter Store** | 設定値・秘匿情報（低コスト） |

### 7.4 セキュリティサービス3兄弟

| サービス | 役割 | 比喩 |
|---|---|---|
| **GuardDuty** | 不正アクセス・異常行動検知 | 警備員 |
| **Inspector** | EC2・Lambda等の脆弱性スキャン | 検査員 |
| **Macie** | S3内の個人情報・機密データ検出 | 監査員 |

---

## 8. 疎結合・サーバーレス・ストリーム処理

### 8.1 メッセージング

| サービス | パターン | 用途 |
|---|---|---|
| **SQS** | キュー（1対1） | 順番処理・バッファ |
| **SNS** | Pub/Sub（1対多） | 即時ファンアウト |
| **EventBridge** | イベントバス | AWSサービス連携・ルーティング |

### 8.2 Lambda vs AWS Batch

| | Lambda | AWS Batch |
|---|---|---|
| 最大実行時間 | **15分** | **無制限** |
| サーバーレス | 完全 | Fargate使用時のみ |
| 用途 | 軽量・イベント駆動 | 重い・長時間バッチ |

### 8.3 API Gateway vs Lambda Function URL

| | Function URL | API Gateway |
|---|---|---|
| 対象 | 単一Lambda | 複数Lambda統合 |
| 料金 | Lambdaのみ（安い） | 高め |
| WAF・throttling・Cognito | ❌ | ✅ |
| カスタムドメイン | ❌ | ✅ |

| シナリオ | 答え |
|---|---|
| Webhook・コールバックのみ | **Function URL** |
| コスト最小でLambda公開 | **Function URL** |
| WAF・throttling・Cognito必要 | **API Gateway** |

### 8.4 Kinesis・MSK・Flink

| サービス | 特徴 |
|---|---|
| **Kinesis Data Streams** | ミリ秒・手動シャード・最大365日保持・自由なコンシューマ |
| **Kinesis Firehose** | 準リアルタイム・フルマネージド・S3/Redshift等へ自動配送 |
| **Apache Flink（Managed）** | **処理・分析**（ミリ秒）。Kinesis/MSKと連携 |
| **MSK** | マネージドKafka。既存Kafka移行向け |

| 比較 | Kinesis | MSK |
|---|---|---|
| メッセージサイズ | 最大1MB | 最大1GB |
| 保持 | 最大365日 | 無制限 |

**Firehose vs Flink**

| | Firehose | Flink |
|---|---|---|
| 目的 | **配送・保存** | **処理・分析** |
| リアルタイム | 数十秒〜 | ミリ秒 |
| コード | 不要 | 必要 |

頻出構成: Firehose → S3（保存）、Flink → リアルタイム分析

---

## 9. コンテナ・CI/CD

### 9.1 ECS vs EKS

| 項目 | ECS | EKS |
|---|---|---|
| 基盤 | AWS独自 | Kubernetes |
| 学習コスト | 低 | 高 |
| 移植性 | AWS依存 | マルチクラウド |
| コスト | 安 | 高 |

| 要件 | 答え |
|---|---|
| 既存Kubernetes移行 | **EKS** |
| AWSでシンプルに | **ECS** |
| サーバー管理不要 | **Fargate** |
| PodにAWS権限 | **EKS + IRSA** |
| 複数Podでストレージ共有 | **EFS** |

### 9.2 ECR

- Dockerイメージのマネージドレジストリ
- パイプライン: GitHub → CodeBuild → **ECR** → ECS/EKS
- イメージスキャン（脆弱性）・ライフサイクルポリシー

### 9.3 デプロイ戦略

| 戦略 | 内容 | 特徴 |
|---|---|---|
| **Canary** | 一部ユーザーのみ新バージョン | 小さく試す |
| **Blue/Green** | 新旧環境を丸ごと切替 | ロールバック簡単・コスト高 |
| **Rolling** | 少しずつ順次更新 | バランス型 |
| **All at once** | 一気に全更新 | 速い・リスク高 |

---

## 10. ハイブリッド・移行・DR

### 10.1 データ移行・同期

| サービス | 用途 | キーワード |
|---|---|---|
| **Transfer Family** | SFTP/FTP/FTPSでS3/EFS | **SFTP** |
| **DataSync** | オンプレ↔AWS ファイル同期 | 定期同期 |
| **Storage Gateway** | オンプレとAWSの恒常ハイブリッド | File/Vol/Tape |
| **Snowball** | ペタバイト級・ネット困難 | 物理輸送 |
| **DMS** | DB移行・レプリケーション | データベース |
| **S3 Transfer Acceleration** | S3アップロード高速化 | 大容量アップロード |

**ファイル共有の使い分け**

- Windows SMB → **FSx for Windows**
- Linux NFS → **EFS**
- オンプレSMBをS3に → **Storage Gateway (File Gateway)**

### 10.2 災害復旧戦略（コスト順）

| 戦略 | RTO/RPO | 概要 |
|---|---|---|
| **Backup & Restore** | 最悪 | スナップショットから復元 |
| **Pilot Light** | 中 | 最小環境を常時起動 |
| **Warm Standby** | 良 | 縮小版本番を常時起動 |
| **Multi-Site** | 最高 | 完全本番を複数稼働 |

- **RTO**: どれだけ早く復旧するか
- **RPO**: どこまで遡って復旧するか（データ損失許容）

---

## 11. 監視・運用・コスト

| サービス | 用途 |
|---|---|
| **CloudWatch** | メトリクス・アラート・ログ |
| **CloudTrail** | APIコール記録・監査 |
| **AWS Config** | リソース設定変更・コンプライアンス |
| **Cost Anomaly Detection** | MLでコスト異常を自動検知 |
| **AWS Budgets** | 手動で閾値設定 |

### CloudWatch の注意点

- デフォルトで **メモリ使用率は取得されない** → カスタムメトリクス
- EC2デフォルト監視は **5分間隔**（詳細モニタリングで1分・有料）

---

## 12. キーワード早見表・混同ペア

### 12.1 キーワード → サービス

| キーワード | 答え |
|---|---|
| SFTP / FTP | Transfer Family |
| 固定IP・UDP・IoT・ゲーム | Global Accelerator |
| キャッシュ・静的配信・SPA | CloudFront |
| 名前解決・DNSルーティング | Route 53 |
| DynamoDBキャッシュ・変更最小 | DAX |
| RDSキャッシュ・セッション | ElastiCache Redis |
| S3アップロード高速化 | Transfer Acceleration |
| オンプレ・安定・専用線 | Direct Connect |
| オンプレ・簡単・暗号化 | Site-to-Site VPN |
| 既存Kafka移行 | MSK |
| 35日より前の復元 | 手動スナップショット |
| コンプライアンス監査 + S3 | Standard-IA |
| 中断リスク最小（Fleet） | capacity-optimized |
| 15分超の処理 | AWS Batch |
| PodへのIAM | IRSA |
| 不正アクセス検知 | GuardDuty |
| 脆弱性スキャン | Inspector |
| S3の個人情報 | Macie |
| フロント簡単デプロイ | Amplify |
| ローテーション必要な秘密 | Secrets Manager |
| 設定値・低コスト秘密 | SSM Parameter Store |

### 12.2 よく混同するペア

| ペア | 覚え方 |
|---|---|
| Multi-AZ vs リードレプリカ | **可用性** vs **読み取り性能** |
| Secrets Manager vs SSM | **ローテーション** vs 設定値保存 |
| SQS vs SNS | **キュー1対1** vs **Pub/Sub 1対多** |
| NAT Gateway vs NATインスタンス | マネージド vs 自己管理（レガシー） |
| VPN vs Direct Connect | インターネット vs 専用線 |
| IOPS vs スループット | 回数/秒 vs 転送量/秒 |
| CloudFront vs Global Accelerator | HTTPキャッシュ vs TCP/UDP経路 |
| CloudFront vs Redis | エッジ配信 vs DB手前キャッシュ |
| Firehose vs Flink | 配送 vs 処理 |
| Kinesis vs MSK | AWSネイティブ vs Kafka互換 |
| DAX vs ElastiCache | DynamoDB専用 vs 汎用 |
| Gateway Endpoint vs Interface | S3/DynamoDB無料 vs 有料・広範囲 |
| VPC Peering vs Transit Gateway | 1対1・非推移 vs ハブ・推移可 |
| PITR vs スナップショット | 秒単位・35日 vs 時点復元 |
| Aurora vs Aurora Serverless | 安定 vs 変動 |
| ECS vs EKS | シンプル vs Kubernetes |
| Lambda vs Batch | 15分以内 vs 長時間 |
| Function URL vs API Gateway | 単純・安い vs 高機能 |
| RI vs Savings Plans | RDS向けRI vs EC2向けSP |
| GuardDuty vs Inspector vs Macie | 脅威 vs 脆弱性 vs 機密データ |
| Budgets vs Cost Anomaly | 手動閾値 vs ML自動検知 |
| Standard-IA vs Glacier Instant | 月1回 vs 四半期1回・90日最小 |
| Intelligent-Tiering vs IA | 不明・取得料無料 vs 低頻度が分かっている |

---

## 付録：試験直前チェックリスト

- [ ] S3クラスは「即時か・頻度・パターン不明か」で判断できる
- [ ] Multi-AZ（可用性）とリードレプリカ（読取）を混同しない
- [ ] VPC接続は「誰と誰を」つなぐかで表を思い出せる
- [ ] CloudFront / GA / Route 53 の役割分担が説明できる
- [ ] DAXはDynamoDB、RedisはRDS/セッションと即答できる
- [ ] SFTP → Transfer Family、Kafka移行 → MSK が反射的
- [ ] 35日ルールとPITR/手動スナップショットの使い分け
- [ ] SG（許可のみ）とNACL（拒否可・ステートレス）の違い
- [ ] Lambda 15分制限 → 超えたら Batch

---

*出典: 2026-05-11, 05-12, 05-13, 05-14, 05-17, 05-18, 05-20 学習メモ*
