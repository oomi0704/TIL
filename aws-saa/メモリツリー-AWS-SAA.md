# AWS SAA メモリツリー（暗記用）

> 教科書の要約版。**上から順に枝を辿る**と答えに着く

## Obsidianで図形として見る

| 方法 | 操作 |
|------|------|
| **Canvas（白板）** | [[メモリツリー-AWS-SAA.canvas]] を開く → ズーム・ドラッグで全体を俯瞰 |
| **Mermaid（本文中）** | 本ノートを **閲覧モード** または **ライブプレビュー** にする → 下の `mermaid` ブロックが自動で図になる |
| 埋め込み | 別ノートに `![[メモリツリー-AWS-SAA.canvas]]` と書くと Canvas を埋め込める |

> Mermaid が文字のまま見える場合：**設定 → コアプラグイン → Mermaid** がオンか確認

---

## 🌳 マスターツリー（最初に見る）

```mermaid
flowchart TB
    ROOT["AWS SAA<br>何の問題？"]

    ROOT --> Q1["§1 何を繋ぐ？"]
    ROOT --> Q2["§2 何を保存？"]
    ROOT --> Q3["§3 何のDB？"]
    ROOT --> Q4["§4 配信・DNS？"]
    ROOT --> Q5["§5 どう処理？"]
    ROOT --> Q6["§6 どう守る？"]
    ROOT --> Q7["§7 どう移す？"]
    ROOT --> Q8["§8 キーワード反射"]

    Q1 --> A1["接続ツリー"]
    Q2 --> A2["ストレージ・S3"]
    Q3 --> A3["DB・キャッシュ"]
    Q4 --> A4["R53 / CF / GA"]
    Q5 --> A5["EC2・Lambda・K8s"]
    Q6 --> A6["IAM・3兄弟"]
    Q7 --> A7["移行・DR"]
    Q8 --> A8["即答表"]

    style ROOT fill:#6b4c9a,color:#fff
    style Q1 fill:#4a90a4,color:#fff
    style Q2 fill:#c47d2a,color:#fff
    style Q3 fill:#c44a4a,color:#fff
    style Q4 fill:#6b4c9a,color:#fff
```

<details>
<summary>テキスト版（コピー用）</summary>

```
AWS SAA
├─【何を繋ぐ？】→ 接続ツリー §1
├─【何を保存？】→ ストレージツリー §2
├─【何のDB？】→ DBツリー §3
├─【どう配信・加速？】→ CDN/DNSツリー §4
├─【どう処理？】→ 計算・サーバーレス §5
├─【どう守る？】→ セキュリティ §6
├─【どう移す？】→ 移行 §7
└─【キーワードだけ？】→ 反射表 §8
```

</details>

---

## §1 接続ツリー「誰と誰をつなぐ？」

```mermaid
flowchart TD
    START["接続: 誰と誰？"]

    START --> ONPREM{"オンプレ ↔ AWS？"}
    ONPREM -->|インターネット・早い・安い| VPN["Site-to-Site VPN"]
    ONPREM -->|専用線・安定・高速| DX["Direct Connect"]
    ONPREM -->|最強| DXVPN["DX + VPN"]
    ONPREM -->|個人PCのみ| CVPN["Client VPN"]

    START --> VPC{"VPC ↔ VPC？"}
    VPC -->|2つのみ| PEER["VPC Peering<br>※推移不可"]
    VPC -->|多数・ハブ| TGW["Transit Gateway"]

    START --> SVC{"VPC ↔ AWSサービス？"}
    SVC -->|S3 or DynamoDB・無料| GW["Gateway Endpoint"]
    SVC -->|その他・オンプレからも| IF["Interface Endpoint"]
    SVC -->|自社SaaS公開| PL["PrivateLink + NLB"]
    SVC -->|外部API| NAT["NAT Gateway"]

    START --> ACL{"トラフィック制御"}
    ACL -->|IP拒否| NACL["NACL"]
    ACL -->|インスタンス単位| SG["Security Group"]
    ACL -->|VPC全体| NFW["Network Firewall"]

    style START fill:#4a90a4,color:#fff
    style VPN fill:#2d6a4f,color:#fff
    style DX fill:#2d6a4f,color:#fff
    style GW fill:#2d6a4f,color:#fff
```

<details>
<summary>テキスト版</summary>

```
接続
│
├─ オンプレ ↔ AWS？
│   ├─ インターネット・早く・安く → Site-to-Site VPN（IPsec自動）
│   ├─ 専用線・安定・高速 → Direct Connect
│   ├─ 最強 → DX + VPN（専用線＋暗号化バックアップ）
│   └─ 個人PCだけ → Client VPN
│
├─ VPC ↔ VPC？
│   ├─ 2つだけ・シンプル → VPC Peering（※推移不可 A→C不可）
│   └─ 多数・ハブ型 → Transit Gateway（推移可）
│
├─ VPC ↔ AWSサービス（S3等）？
│   ├─ S3 or DynamoDB・無料 → Gateway Endpoint
│   ├─ その他サービス → Interface Endpoint（PrivateLink）
│   └─ オンプレからも使う → Interface（DX/VPN経由）
│
├─ 自社サービスを他社VPCに公開？
│   └─ PrivateLink + NLB
│
└─ 外部インターネットAPI（Google等）？
    └─ NAT Gateway（PrivateLinkでは不可）
```

</details>

### 暗記フレーズ
> **「オンプレVPN、専用線DX、VPCはPeeringかTGW、S3は無料GW」**

### SG vs NACL（接続の隣で必ず出る）

```
トラフィック制御
│
├─ 特定IPを拒否したい？
│   └─ YES → NACL（SGは拒否不可）
│
├─ インスタンス単位？
│   └─ YES → セキュリティグループ（ステートフル・許可のみ）
│
├─ サブネット単位？
│   └─ YES → NACL（ステートレス・番号順・戻りも許可要）
│
└─ VPC全体・高度？
    └─ AWS Network Firewall
```

### IPv6だけ覚える

```
IPv6でプライベートから外へ出すだけ
└─ Egress-Only Internet Gateway（NATのIPv6版）
```

---

## §2 ストレージツリー「何を・どう保存？」

### 2-1 まず3分類

```
ストレージ種類
├─ オブジェクト・静的・安い大容量 → S3
├─ ブロック・1台のディスク → EBS
└─ ファイル・共有
    ├─ Linux・複数EC2 → EFS（NFS）
    ├─ Windows・SMB → FSx for Windows
    └─ HPC・ML → FSx for Lustre
```

### 2-2 S3クラス（試験最頻出）

```mermaid
flowchart TD
    S3["S3クラス選び"]

    S3 --> FREQ{"アクセス頻度？"}
    FREQ -->|高い| STD["Standard"]
    FREQ -->|月1回・即時| IA["Standard-IA<br>最小30日"]
    FREQ -->|年1-2回・即時| GI["Glacier Instant<br>最小90日"]
    FREQ -->|不明★| IT["Intelligent-Tiering"]

    S3 --> FEE{"取得料金を払いたくない？"}
    FEE -->|YES| IT

    S3 --> CHEAP{"最安・ほぼ読まない？"}
    CHEAP -->|YES| DEEP["Glacier Deep Archive"]

    S3 --> AUDIT{"コンプライアンス監査？"}
    AUDIT -->|YES| IA

    style IT fill:#c47d2a,color:#fff
    style S3 fill:#c47d2a,color:#fff
```

<details>
<summary>テキスト版</summary>

```
S3クラス選び
│
├─ ① 取り出しは即時？待てる？
│   ├─ 即時必須 → 次へ
│   └─ 数分〜12時間待てる → Glacier系へ
│
├─ ② アクセス頻度は？
│   ├─ 高い → Standard
│   ├─ 低い（月1回くらい）→ Standard-IA（最小30日）
│   ├─ 超低い（四半期1回）・即時 → Glacier Instant（最小90日）
│   └─ わからない → Intelligent-Tiering ★
│
├─ ③ 取得料金を払いたくない？
│   └─ YES → Intelligent-Tiering（取得料無料）
│
├─ ④ 最安・ほぼ読まない？
│   └─ Glacier Deep Archive
│
└─ ⑤ コンプライアンス監査？
    └─ Standard-IA（定期アクセス＝頻度は自然と高い）
```

</details>

### S3暗記フレーズ
> **「不明はIT、月1はIA、年1即時はGI、最安はDeep」**

### 2-3 EBSタイプ

```
EBS
│
├─ 汎用・コスパ → gp3（gp2より安・IOPS独立）★デフォルト
├─ 超高IOPS・DB → io1 / io2
│   ├─ 最高耐久5ナイン → io2
│   └─ 複数EC2同時マウント → io1/io2のみ（同一AZ）
├─ 大容量ログ・スループット → st1（ブート不可）
└─ 最安・コールド → sc1（ブート不可）

トラップ：io3は存在しない
```

### 2-4 永続性

```
EC2のディスク
├─ 止めたら消えてOK・爆速 → インスタンスストア
└─ 残したい・OS・DB → EBS
```

### 2-5 転送高速化

```
転送を速く
├─ S3にアップロード → Transfer Acceleration
├─ S3からダウンロード → CloudFront
└─ EC2/ALBへ・固定IP → Global Accelerator
```

---

## §3 DBツリー「SQL？NoSQL？何の用途？」

### 3-1 最初の分岐

```mermaid
flowchart TD
    DB["DB選び"]

    DB --> SQL{"SQL？"}
    SQL -->|YES| RDS["コスト → RDS"]
    SQL -->|YES| AUR["性能 → Aurora ★"]
    SQL -->|YES| ASL["変動 → Aurora Serverless"]
    SQL -->|NO| DDB["DynamoDB"]
    SQL -->|NO| DOC["DocumentDB"]
    SQL -->|NO| NEP["Neptune"]
    SQL -->|NO| TS["Timestream"]
    SQL -->|NO| QL["QLDB"]

    DB --> REP{"RDS複製の目的？"}
    REP -->|可用性| MAZ["Multi-AZ<br>マルチは守る"]
    REP -->|読取| RR["Read Replica<br>レプリは読む"]

    DB --> CACHE{"キャッシュ？"}
    CACHE -->|DynamoDB| DAX["DAX"]
    CACHE -->|RDS・セッション| RED["ElastiCache Redis"]

    DB --> BACK{"いつまで戻す？"}
    BACK -->|35日以内| PITR["PITR"]
    BACK -->|35日超| SS["手動スナップショット"]

    style AUR fill:#c44a4a,color:#fff
    style MAZ fill:#c44a4a,color:#fff
    style DAX fill:#c44a4a,color:#fff
```

<details>
<summary>テキスト版</summary>

```
DB
│
├─ SQL？
│   ├─ YES
│   │   ├─ コスト重視 → RDS
│   │   ├─ 高性能・30秒FO → Aurora ★
│   │   └─ トラフィック不明 → Aurora Serverless
│   └─ NO（NoSQL）
│       ├─ Key-Value大規模 → DynamoDB
│       ├─ ドキュメント → DocumentDB
│       ├─ グラフ → Neptune
│       ├─ 時系列 → Timestream
│       └─ 台帳 → QLDB
```

### 3-2 RDSの2つ（超頻出ペア）

```
RDSの複製
├─ 可用性・障害時切替 → Multi-AZ（同期）
└─ 読み取りを速く → リードレプリカ（非同期）

暗記：「マルチは守る、レプリは読む」
```

### 3-3 キャッシュ

```
DBの前にキャッシュ
│
├─ DynamoDB・変更最小？
│   └─ DAX（μs・DynamoDB専用）
│
├─ RDS・セッション・ランキング？
│   └─ ElastiCache Redis
│
└─ シンプル・水平だけ？
    └─ Memcached
```

### 3-4 バックアップ

```
過去に戻す
│
├─ 35日以内・秒単位？
│   └─ PITR（DynamoDB/RDS/Aurora）
│
├─ 35日より前？
│   └─ 手動スナップショット（無制限保持）
│
└─ 複数サービスまとめて管理？
    └─ AWS Backup
```

### DynamoDBモード

```
DynamoDB
├─ トラフィック不明 → オンデマンド
└─ 安定・安く → プロビジョニング
```

</details>

---

## §4 CDN・DNS・加速ツリー

```mermaid
flowchart LR
    Q["配信・名前解決"]

    Q --> DNS["名前解決・DNS<br>Route 53"]
    Q --> HTTP["HTTP・キャッシュ<br>CloudFront"]
    Q --> TCP["TCP/UDP・固定IP<br>Global Accelerator"]

    DNS --> D1["シンプル"]
    DNS --> D2["レイテンシー"]
    DNS --> D3["Geolocation"]
    DNS --> D4["Weighted"]
    DNS --> D5["フェイルオーバー"]

    HTTP --> GEO["国制限のみ<br>Geo Restriction"]

    TCP --> IOT["IoT・MQTT・ゲーム"]

    style DNS fill:#6b4c9a,color:#fff
    style HTTP fill:#6b4c9a,color:#fff
    style TCP fill:#6b4c9a,color:#fff
```

<details>
<summary>テキスト版</summary>

```
配信・名前解決・経路
│
├─ 名前解決・DNSルーティング？
│   └─ Route 53
│       ├─ 1つ返す → シンプル
│       ├─ 遅延最小 → レイテンシー
│       ├─ 地域 → Geolocation
│       ├─ 割合 → Weighted
│       └─ 障害切替 → フェイルオーバー
│
├─ HTTP・キャッシュ・静的・動画？
│   └─ CloudFront
│       └─ 国制限だけ → Geo Restriction（無料）
│
└─ TCP/UDP・固定IP・低遅延？
    ├─ ゲーム・VoIP・API → Global Accelerator
    └─ IoT・MQTT → Global Accelerator（キャッシュ不要）

暗記：「DNSはR53、HTTPはCF、TCPはGA」
```

</details>

### CloudFront vs Redis（2段キャッシュ）

```
ユーザー → CloudFront → アプリ → Redis → DB
           （世界中）          （VPC内・DB負荷）
```

### WAFまわり

```
セキュリティ追加
├─ 国制限だけ → CF Geo Restriction
├─ 国制限＋他ルール → WAF + Geo Match
├─ SQLi等 → WAF
└─ DDoS → CF + WAF + Shield
```

---

## §5 計算・サーバーレス・ストリーム

```mermaid
flowchart TB
    subgraph EC2["EC2"]
        E1["短期 → On-Demand"]
        E2["長期EC2 → Savings Plans"]
        E3["長期RDS → RI"]
        E4["中断OK → Spot"]
        E5["Fleet → capacity-optimized"]
    end

    subgraph LB["ELB"]
        L1["HTTP → ALB"]
        L2["TCP/UDP → NLB"]
    end

    subgraph SERVERLESS["サーバーレス"]
        S1{"15分以内？"}
        S1 -->|YES| LAM["Lambda"]
        S1 -->|NO| BAT["AWS Batch"]
        S2["Webhook → Function URL"]
        S3["WAF等 → API Gateway"]
    end

    subgraph MSG["メッセージ"]
        M1["1対1 → SQS"]
        M2["1対多 → SNS"]
        M3["イベント → EventBridge"]
    end

    subgraph STREAM["ストリーム"]
        ST1["Kafka移行 → MSK"]
        ST2["ミリ秒 → Kinesis Streams"]
        ST3["配送 → Firehose"]
        ST4["分析 → Flink"]
    end

    subgraph CTR["コンテナ"]
        C1["K8s → EKS + IRSA"]
        C2["シンプル → ECS"]
        C3["Pod共有 → EFS"]
    end
```

### 5-1 EC2購入

```
EC2の買い方
├─ 短期・不定期 → オンデマンド
├─ 1〜3年安定
│   ├─ EC2 → Savings Plans ★推奨
│   └─ RDS → リザーブドインスタンス
├─ 中断OK・最安 → スポット
└─ Fleetで中断最小 → capacity-optimized ★
```

### 5-2 ELB

```
ロードバランサ
├─ HTTP/HTTPS・パス振分 → ALB
├─ TCP/UDP・超低遅延 → NLB
└─ サードパーティ機器 → GLB
```

### 5-3 Lambda vs Batch

```
処理時間
├─ 15分以内・イベント駆動 → Lambda
└─ 15分超・重いバッチ → AWS Batch
    ├─ サーバーレスにしたい → Batch + Fargate
    └─ 自分でEC2管理 → Batch + EC2
```

### 5-4 API公開

```
Lambdaを外に公開
├─ Webhookだけ・安い → Lambda Function URL
└─ WAF/throttling/Cognito/複数Lambda → API Gateway
```

### 5-5 メッセージング

```
非同期連携
├─ 1対1・順番・キュー → SQS
├─ 1対多・即時 → SNS
└─ イベントバス・AWS連携 → EventBridge
```

### 5-6 ストリーム

```
リアルタイムデータ
│
├─ 既存Kafkaを移行？
│   └─ MSK（メッセージ1GB・保持無制限）
│
├─ AWSネイティブ・ミリ秒・再処理？
│   └─ Kinesis Data Streams
│
├─ S3等へ自動配送・コード不要？
│   └─ Kinesis Firehose
│
└─ 分析・集計・コード必要？
    └─ Apache Flink（Managed）
        └─ よくある構成：Firehose→保存、Flink→分析
```

### 5-7 コンテナ

```
コンテナ
├─ Kubernetes・マルチクラウド → EKS
│   ├─ PodにIAM → IRSA
│   └─ 複数Podで共有ストレージ → EFS
├─ AWSでシンプル → ECS
├─ サーバー不要 → Fargate
└─ イメージ置き場 → ECR
```

### 5-8 フロント

```
SPA・フロント
├─ 典型 → S3 + CloudFront + API GW + Lambda
└─ 簡単デプロイ → Amplify
```

### 5-9 デプロイ

```
リリース方法
├─ 一部だけ試す → Canary
├─ 丸ごと切替・戻しやすい → Blue/Green（高コスト）
├─ 少しずつ → Rolling
└─ 一気・リスク高 → All at once
```

---

## §6 セキュリティツリー

```mermaid
flowchart TD
    SEC["セキュリティ"]

    SEC --> IAM["IAM"]
    IAM --> I1["権限 → Policy"]
    IAM --> I2["付与 → Role"]
    IAM --> I3["一時 → AssumeRole<br>Denyが王様"]

    SEC --> SECRET["秘密"]
    SECRET --> K["鍵 → KMS"]
    SECRET --> SM["ローテ → Secrets Manager"]
    SECRET --> P["設定 → SSM"]

    SEC --> DETECT["検知3兄弟"]
    DETECT --> G["不正 → GuardDuty"]
    DETECT --> INS["脆弱性 → Inspector"]
    DETECT --> MAC["個人情報 → Macie"]

    style DETECT fill:#c44a4a,color:#fff
```

### 6-1 IAM

```
IAM
├─ 権限定義 → ポリシー
├─ 付与セット → ロール
└─ 一時的に借りる → AssumeRole（STS）
    ├─ AWS内 → AssumeRole
    ├─ Google等 → AssumeRoleWithWebIdentity
    └─ AD等 → AssumeRoleWithSAML

評価：Deny最優先 → SCP → リソース → Boundary →  identity → デフォルトDeny
暗記：「Denyが王様」
```

### 6-2 秘密情報

```
秘密・鍵
├─ 暗号化キー → KMS
├─ 自動ローテーション → Secrets Manager
└─ 設定値・安い → SSM Parameter Store
```

### 6-3 検知3兄弟

```
何を検知？
├─ 不正・異常行動 → GuardDuty（警備員）
├─ 脆弱性 → Inspector（検査員）
└─ S3の個人情報 → Macie（監査員）

暗記：「ガード・インスペ・マサイ」
```

---

## §7 移行・DRツリー

### 7-1 データ移行

```mermaid
flowchart TD
    MOVE["データを移す"]

    MOVE -->|SFTP★| TF["Transfer Family"]
    MOVE -->|ファイル同期| DS["DataSync"]
    MOVE -->|DB| DMS["DMS"]
    MOVE -->|PB級・ネット弱| SB["Snowball"]
    MOVE -->|常時ハイブリッド| SG["Storage Gateway"]
    MOVE -->|S3アップロード高速| TA["Transfer Acceleration"]

    DR["災害復旧 安→高"]
    DR --> B["Backup & Restore"]
    B --> P["Pilot Light"]
    P --> W["Warm Standby"]
    W --> M["Multi-Site"]

    style TF fill:#c47d2a,color:#fff
    style MOVE fill:#c47d2a,color:#fff
```

<details>
<summary>テキスト版</summary>

```
データを移す
│
├─ SFTP/FTP/FTPS？ ★キーワード
│   └─ Transfer Family → S3/EFS
│
├─ ファイル定期同期？
│   └─ DataSync
│
├─ DB移行・レプリケーション？
│   └─ DMS
│
├─ ネット遅い・PB級？
│   └─ Snowball
│
├─ 常時ハイブリッド？
│   └─ Storage Gateway
│       └─ オンプレSMB→S3 は File Gateway
│
└─ Windows共有？
    ├─ オンプレ継続 → Storage Gateway
    └─ AWS上 → FSx for Windows
```

### 7-2 DR（コスト安→高）

```
災害復旧（RTO/RPOが良いほど高い）
Backup & Restore
  → Pilot Light
    → Warm Standby
      → Multi-Site

暗記：「バック→パイロ→ウォーム→マルチ」
```

</details>

---

## §8 反射表（キーワード→即答）

| 聞こえたら | 答え |
|-----------|------|
| SFTP | Transfer Family |
| 固定IP / UDP / IoT / MQTT | Global Accelerator |
| キャッシュ / 静的 / SPA | CloudFront |
| DNS / 名前解決 | Route 53 |
| DynamoDB + 変更最小 | DAX |
| RDS + キャッシュ | Redis |
| セッション / ランキング | Redis |
| S3アップロード高速 | Transfer Acceleration |
| オンプレ + 安定 | Direct Connect |
| オンプレ + 簡単 | VPN |
| 既存Kafka | MSK |
| 35日より前 | 手動スナップショット |
| コンプライアンス監査 + S3 | Standard-IA |
| 中断リスク最小 | capacity-optimized |
| 15分超 | AWS Batch |
| Pod + IAM | IRSA |
| Pod + 共有ディスク | EFS |
| 不正検知 | GuardDuty |
| 脆弱性 | Inspector |
| S3個人情報 | Macie |
| フロント簡単 | Amplify |
| ローテーション | Secrets Manager |
| 設定値安い | SSM |
| Webhookだけ | Function URL |
| WAF / throttling | API Gateway |
| 可用性 | Multi-AZ |
| 読み取り速く | リードレプリカ |
| 拒否したいIP | NACL |
| S3/DynamoDB無料接続 | Gateway Endpoint |
| io3 | 存在しない（罠） |

---

## §9 混同ペア・1行暗記

| A | B | 一行 |
|---|---|------|
| Multi-AZ | RR | 守る / 読む |
| Secrets | SSM | 回す / 置く |
| SQS | SNS | 1対1 / 1対多 |
| VPN | DX | ネット / 専用線 |
| IOPS | スループット | 回数 / MB |
| CF | GA | HTTP / TCP |
| CF | Redis | エッジ / DB前 |
| Firehose | Flink | 運ぶ / 計算する |
| Kinesis | MSK | AWS / Kafka |
| DAX | Redis | Dynamo専 / 汎用 |
| GW EP | IF EP | 無料S3 / 有料広 |
| Peering | TGW | 2つ / 多数 |
| PITR | SS | 35日秒 / 無制限 |
| Lambda | Batch | 15分 / 無限 |
| Func URL | APIGW | 安単 / 高機能 |
| IA | GI Instant | 30日月1 / 90日年1 |
| 不明 | IA | IT / 低頻度 |

---

## §10 1日5分暗記ルーティン

```
月：接続ツリー §1 を声に出す
火：ストレージ §2（S3枝を3回）
水：DB §3（Multi-AZ/RR + キャッシュ）
木：CDN §4 + セキュリティ §6
金：移行 §7 + 反射表 §8
土：混同ペア §9 をシャッフル
日：マスターツリーだけ頭の中で再現
```

---

## 付録：全体マインドマップ

```mermaid
mindmap
  root((AWS SAA))
    接続
      VPN
      Direct Connect
      Peering
      TGW
      GW Endpoint
    ストレージ
      S3 IT
      EBS gp3
      EFS
    DB
      Aurora
      Multi-AZ
      Read Replica
      DAX
    配信
      Route53
      CloudFront
      Global Accelerator
    キーワード
      SFTP
      MSK
      Batch
```

---

*対応: [[教科書-AWS-SAA]] · 図形白板: [[メモリツリー-AWS-SAA.canvas]]*
