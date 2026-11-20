# 技術調査：戦闘状態の取得方法

調査日：2026-11-20。対象は既存Modのソースコード（どちらもMITライセンス）。

## 結論

- **状態の中身は十分**：STS2MCP は戦闘の完全な状態（手札、山札・捨て札・廃棄の中身、敵の意図とパワー、オーブ、ペット、エネルギー）を出力する。評価エンジンの入力として使える。
- **ただし「プレイヤーが何をしたか」のログは無い**：STS2MCP は呼ばれたときに現在状態を返すだけ（pull型）。HermesBridge は画面遷移で状態を書き出すが、カードを使った瞬間はフックしていない。
- **方針**：
  1. **プロトタイプ**：STS2MCP の `GET` を短い間隔でポーリングし、状態の差分からプレイ内容を推定する。Mod開発ゼロで検証できる
  2. **本番**：独自の **記録専用Mod（read-only recorder）** を作る。ゲームの `Hook` 系メソッドを Harmony でパッチし、「ターン開始時の状態＋そのターンの行動列」を JSONL に書く。状態のシリアライズは STS2MCP の `StateBuilder` を参考にする（MIT）

## STS2MCP（[Gennadiyev/STS2MCP](https://github.com/Gennadiyev/STS2MCP)）

- 最終コミット 2026-07-29。テスト対象は v0.103.2
- `localhost:15526` で HTTP API を提供。`GET /api/v1/singleplayer` が現在状態、`POST` が操作。認証は無い
- 仕様書：`docs/raw-full.md`

### 戦闘中に取れるもの（`state_type: monster / elite / boss`）

| 対象 | フィールド |
|---|---|
| ラウンド | `battle.round`, `battle.turn`（player / enemy）, `is_play_phase` |
| 敵 | `entity_id`, `hp`, `max_hp`, `block`, `status[]`（パワー：id / amount / Buff・Debuff）, `intents[]`（type / label=ダメージ値 / description） |
| プレイヤー | `hp`, `max_hp`, `block`, `energy`, `max_energy`, `stars`（Regent）, `status[]`, `relics[]`（counter付き）, `potions[]` |
| カード | `hand[]`（id, cost, type, target_type, can_play, is_upgraded, description）, `draw_pile[]`, `discard_pile[]`, `exhaust_pile[]` |
| キャラ固有 | `orbs[]`（Defect）、`pets[]`（Necrobinder） |
| その他の画面 | `map`（DAG全体、ボス）、`event`（選択肢）、`rest_site`、`shop`、`card_reward`、`hand_select`、`game_over` など |

### 足りないもの
- 行動イベントが流れてこない（WebSocket や履歴は無い）。ポーリングで差分を取るしかない
- 山札のカードは `name / cost / description` だけで、`id` や強化状態が手札より少ない
- 山札は「レアリティ順にソート」されているので、**ドロー順は分からない**（ゲーム内の表示と同じ情報量。コーチ用途ならむしろ公平）
- 乱数（シード由来の次ドロー、敵の行動パターン）は出力されない

### ポーリング差分での推定の限界
- カード使用は「手札から消えた＋エネルギー減少」で推定できる。ただし、ドローや廃棄が連鎖するカードや、対象の敵の特定は曖昧になる
- 1手ごとの状態を取りこぼさないには、ポーリング間隔をアニメーションより短くする必要がある
- → 検証用には十分。本番は独自Modでフックする

## HermesBridge（[hiKareeem/ClaudePlaysTheSpire](https://github.com/hiKareeem/ClaudePlaysTheSpire)）

- 最終コミット 2026-05-06（STS2MCP より古い）
- 状態変化のたびに `state.json` を書き出し、`commands.json` を読んで操作する。`floor-history.jsonl` にフロアごとのスナップショットを追記する
- Harmony パッチは画面系（報酬、マップ、イベント、ショップ、休憩所）が中心。**カードを使った瞬間のフックは無い**
- 参考になる点：`Hook.AfterRestSiteSmith`、`CardCmd.Upgrade`、`RestSiteSynchronizer.ChooseOption` などのパッチ例から、ゲーム本体に `Hook` クラスがあり、行動の前後にフックできる構造だと分かる。独自レコーダーでは同じ要領で、カード使用・ターン開始/終了・ポーション使用を捕まえられる見込み（要検証）

## 記録フォーマット案（独自レコーダー）

```jsonc
// combat-<runId>-<floor>.jsonl ：1行 = 1イベント
{"t":"combat_start","floor":7,"encounter":"...","state":{...}}
{"t":"turn_start","round":1,"state":{...}}      // 評価エンジンの入力
{"t":"play_card","card":"BASH","target":"JAW_WORM_0","energy_before":3}
{"t":"use_potion","potion":"FIRE_POTION","target":"..."}
{"t":"turn_end","round":1,"state":{...}}
{"t":"combat_end","won":true,"hp_lost":9}
```

## 次のステップ
1. ゲーム環境で STS2MCP を入れ、実際の `GET` 出力を保存する（ユーザー作業。Windows / Steam）
2. ポーリング記録スクリプトを作り、1戦闘分のターンログが復元できるか確かめる
3. STS2 の `Hook` クラスのメソッド一覧を確認し（デコンパイル、または BaseLib の資料）、レコーダーModの設計を固める
