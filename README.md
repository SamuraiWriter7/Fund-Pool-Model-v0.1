# Fund-Pool-Model-v0.1
A monthly pooled reward model for allocating value back to AI interaction-data contributors through auditable, operable, and fraud-aware distribution.

## Platform Revenue Share Pool for AI Contribution Allocation

**Fund Pool Model v0.1** は、AIプラットフォームにおける行動データ貢献者へ、  
**月次基金プール方式**で報酬を還元するための最小実装仕様です。

This specification defines a **monthly pooled reward model** for returning value to contributors of AI interaction data, prioritizing **operability over perfect attribution**.

---

## 概要

AI時代の報酬分配を考えるとき、いきなり「1件ごとの精密な個別印税」を実装するのは難しい。  
因果追跡は重く、計算コストも高く、不正耐性も弱くなりやすいからです。

そこで本仕様は、次の考え方を採用します。

> **完全公平より先に、分配制度を起動する。**

Fund Pool Model v0.1 は、AIサービスの個人向けサブスクリプション収益の一部を基金化し、  
opt-in された行動データ貢献者に対して、**月次で粗く、しかし運用可能な形で還元**する仕組みです。

これは個別印税型ではなく、**基金プール型（Platform Revenue Share Pool）** の最小構成です。

---

## この仕様が目指すもの

このモデルの目的は、単にお金を配ることではありません。  
目指しているのは、次の4点です。

1. **価値の発生源を可視化すること**  
   AIの性能向上が、ユーザーの行動データに支えられていることを前提化する。

2. **無償データ貢献ループを可視化・還元可能にすること**  
   「ただ使われるだけ」の状態から、「還元の対象になりうる状態」へ進める。

3. **個別印税の前段階となる制度を作ること**  
   精密な因果追跡が難しくても、まずは粗い制度から循環を始める。

4. **現実のプロダクト・契約・会計に載る形にすること**  
   理想論ではなく、SaaS / API / GitHub連携のような実運用へ接続できる形を重視する。

---

## 基本思想

Fund Pool Model v0.1 は、以下の原則に基づいています。

- 完全公平より先に制度を起動する
- 個別因果追跡ではなく月次プール分配を採用する
- 現金払いより先に割引・利用クレジット還元を優先する
- 不正耐性・透明性・運用コストの均衡を取る
- Business / Enterprise 契約は初期対象外とする

この仕様は、**理想形ではなく起動形**です。  
言い換えれば、これは「完成した印税OS」ではなく、**印税OSに至る前の実務的な分配エンジン**です。

---

## 参加主体

### 1. Payer
基金の原資を拠出し、分配を実行するプラットフォーム事業者です。

例:
- GitHub
- Microsoft
- AI SaaS platform operators

### 2. Contributors
報酬分配の対象となる貢献者です。

初期対象:
- Free / Pro / Pro+ の個人ユーザー
- 学習利用に **opt-in** している利用者

初期除外:
- Business / Enterprise 契約
- opt-out ユーザー
- 審査保留データ

### 3. Beneficiaries
モデル改善の恩恵を受ける利用者全体です。  
分配対象者と完全一致する必要はありません。

---

## 原資

初期の原資は、**個人向けサブスクリプション収益の一部**です。

### 基本式

```text
fund_pool = personal_subscription_revenue * pool_rate + carryover_pool

初期設定
pool_rate = 0.03（3%）
Business / Enterprise 収益は初期段階では除外可能
未払い繰越分は翌月プールへ加算
将来的に追加可能な原資
広告収益の一部
アップセル収益の一部
開発者支援基金
パートナープログラム拠出金
何を「貢献」とみなすか

本仕様では、静的な作品提供だけでなく、行動データの質を重視します。

対象となるシグナル
suggestion acceptance events
modification after acceptance
retention signals
reuse signals
session diversity
context uniqueness

つまり、このモデルは「ただ入力したか」ではなく、

採用されたか
修正されたか
生き残ったか
再利用されたか
多様な文脈で使われたか

を見ます。

スコアリング構造

Fund Pool Model v0.1 では、各ユーザーの貢献を 粗いスコアとして表現します。

品質スコア
quality_score =
  0.30 * acceptance_rate +
  0.20 * uniqueness_avg +
  0.20 * edit_quality_avg +
  0.15 * retention_rate +
  0.10 * reuse_rate +
  0.05 * account_trust_score
生スコア
raw_score =
  volume_factor *
  quality_score *
  plan_weight *
  fraud_penalty
volume_factor

イベント量が多いほど有利になりますが、過剰入力でスコアが暴騰しないよう、対数補正を使います。

volume_factor = log(1 + min(total_events, max_volume_cap))
プラン係数

初期設定では、以下のような緩やかな係数差を置きます。

Free: 0.80
Pro: 1.00
Pro+: 1.10

これは価値判断ではなく、利用文脈や品質差の仮説に基づく暫定設計です。

品質フィルタ

低品質・低信頼の貢献を分配対象から外すため、最低条件を設けます。

初期閾値
min_events_required = 20
min_acceptance_rate = 0.05
min_retention_rate = 0.10

これを満たさない場合は、当月スコアを 0 とします。

不正耐性

分配制度は、導入した瞬間に攻略対象になります。
そのため、本仕様では最初から fraud detection を組み込みます。

主な不正シグナル
多重アカウントの過剰重複
短時間大量生成 + 保持率低下
受諾→破棄→再生成のループ
同一コンテキスト反復スパム
疑義スコア
suspicion_score =
  0.30 * multi_account_signal +
  0.25 * low_quality_burst_signal +
  0.25 * self_accept_loop_signal +
  0.20 * duplicate_context_signal
保留ルール
suspicion_score >= 0.75 の場合
分配は即時停止
手動審査キューへ送る
正規化

上位少数の極端なスコアがプール全体を食い尽くさないよう、
v0.1 では 99パーセンタイル上限を使います。

手順
cap_value = percentile(raw_scores, 99)
adjusted_score_i = min(raw_score_i, cap_value)
normalized_score_i = adjusted_score_i / sum(all adjusted_score)

これにより、過度な集中をある程度抑制します。

分配式

月次基金プールは、正規化スコア比で按分されます。

payout_i = fund_pool * normalized_score_i

これは個別出力への因果的ロイヤリティではなく、
当月の貢献総量に対するプール配分です。

支払いポリシー

初期段階では、現金払いより 割引・利用クレジット を優先します。

支払い優先順位
Pro / Pro+
subscription discount
Free
usage credit
少額の場合
min_credit_payout 未満は翌月へ繰越
審査保留の場合
手動審査完了まで hold
初期設定
min_credit_payout = 1.00

この仕様により、マイクロペイメントの事務コストを抑えつつ還元を回せます。

月次実行フロー

Fund Pool Model v0.1 は、月次バッチ処理を前提としています。

Flow
load users
load monthly events
load monthly subscription revenue
filter eligible users
build user metrics
apply quality filters
apply fraud detection
compute raw scores
normalize scores
calculate fund pool
allocate pool
apply payout policy
write audit log
publish dashboard
execute payouts

リアルタイム分配ではなく、月1回の安定運用を優先します。

透明性

制度の信頼性を支えるため、v0.1 では最小限の透明性を要求します。

公開ダッシュボード項目
month
fund_pool total
total contributors
median payout
clustered distribution view
formula version
監査ログ項目
month
fund_pool
eligible_users_count
review_holds_count
distribution_summary
score_formula_version

※ 個人を直接特定できる生ログは公開対象外です。

このモデルの強み

Fund Pool Model v0.1 の長所は明確です。

月次バッチで回せる
会計・契約・UI設計に載せやすい
Shapley値フル計算が不要
まず制度を起動できる
無償データ貢献ループを還元可能な形へ一歩進められる
GitHub連携やSaaSダッシュボードと相性がよい

つまりこれは、理想の終着点ではなく、現実の出発点です。

限界

もちろん、この仕様は未完成です。

真の因果的貢献を精密測定しているわけではない
acceptance rate ハックの余地がある
uniqueness / edit_quality の設計次第で偏りが出る
plan_weight は政治的調整が必要
現金還元まで進むと税務 / KYC が重くなる
Shapley型の公平性にはまだ届かない

このモデルは、粗いが動く制度として理解されるべきです。

ロードマップ
v0.2

品質フィルタ強化

longer retention windows
review pass rate
stronger quality cluster scoring
v0.3

クラスター先行分配

anonymous cluster allocation
secondary intra-cluster distribution
v1.0

ハイブリッド分配

fund pool + simplified shapley
high-value contribution tracing
partial causal attribution layer
想定ユースケース
AI coding assistants
generative writing platforms
prompt contribution systems
evaluation-data reward programs
AI SaaS with opt-in training feedback loops
Positioning

Fund Pool Model v0.1 is not a full royalty OS.
It is a bootable allocation engine for platforms that want to move from unpaid contribution loops toward measurable and auditable value return.

日本語で言えば、これは「完成された印税OS」ではなく、
印税OSの本丸へ向かうための、最初に回る分配装置です。

License / Status
Status: Draft
Intended use: Conceptual / architectural / implementation planning
Not legal advice
Not an accounting standard
Not a final governance framework
Related Files
fund-pool-model-v0.1.yaml
docs/spec-ja.md
docs/pseudocode.md
docs/diagram.md
Final Note

分配制度は、美しいだけでは動きません。
しかし、粗くても起動すれば、そこから改善できます。

Fund Pool Model v0.1 は、
完全公平を約束する仕様ではありません。
その代わりに、無還元から還元へ移るための最初の橋を提供します。
