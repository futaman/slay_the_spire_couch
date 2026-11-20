# 競合分析：Slay the Spire 2 上達支援ツール

調査日：2026-11-20。出典は各ツールの公開ページで、一部は検索結果の要約のみ（「未読」と明記）。

## 1. 競合マップ

| カテゴリ | 代表ツール | 何をするか | LLM | タイミング |
|---|---|---|---|---|
| カード/レリック評価オーバーレイ | [sts2-advisor](https://github.com/ebadon16/sts2-advisor)、[Deck Compass](https://www.nexusmods.com/slaythespire2/mods/627)（未読）、[Untapped.gg](https://sts2.untapped.gg/en)（未読）、[MetaBot Card Advisor](https://metabot.gg/en/slay-the-spire-2/card-advisor) | 報酬・ショップの選択肢を採点（S〜F、勝率差分）して「ベストピック」を出す | なし（ティア表＋コミュニティ統計） | プレイ中 |
| ラン履歴ダッシュボード | [sts2-dashboard](https://github.com/HerrAA918/sts2-dashboard)、[SpireScope](https://github.com/thequantumfalcon/spirescope)、[stat-your-spire](https://github.com/dimmenfredrik/stat-your-spire)、MetaBot Desktop、Scry、sts2runs 等 | `.run` を集計：勝率、死因、カード別勝敗、フロア別HP。stat-your-spire は「次のランの Focus」を出す | なし | ラン後 |
| デッキ構築アドバイザ | [Spire Strategist](https://spire-strategy-builder.lovable.app/)（未読）、[SpireSage](https://apps.apple.com/us/app/spiresage-slay-the-spire-2/id6760845472)（未読、"AI bot" を名乗る） | 手入力したデッキに次の候補や削るカードを提案 | SpireSage は不明 | 手動 |
| ML予測 | [sts2-card-advisor](https://github.com/swimmybro/sts2-card-advisor) | 自分の履歴から勝率を予測してピックを推薦 | なし（ML） | ラン後/手動 |
| AIに遊ばせるMod | [STS2MCP](https://github.com/Gennadiyev/STS2MCP)（545★）、[STS2-Agent](https://github.com/Meddrin/STS2-Agent)、[ClaudePlaysTheSpire](https://github.com/hiKareeem/ClaudePlaysTheSpire)、[AI-plays-STS2](https://github.com/IgorWarzocha/AI-plays-Slay-The-Spire-2)、[LLM co-op mod](https://github.com/brekkylab/sts2-llm-coop-mod) | ゲーム状態をAPIやMCPで公開し、LLMがプレイする | LLMが「操作」する | プレイ中 |
| 戦闘ログ/計算 | Combat history mod（[Nexus 100](https://www.nexusmods.com/slaythespire2/mods/100)）、[Damage Charts](https://awesome.ecosyste.ms/projects/github.com%2Fbrian-gates%2Fsts2-damage-charts)、[sts2calc](https://sts2calc.pages.dev/) | 戦闘ログの表示・出力、与ダメージ集計、手動のリーサル計算 | なし | プレイ中/手動 |
| 練習問題 | [tckmn STS2 Problem Set](https://tck.mn/videogame/stsexam/s2p1) | 人力で作ったPDFの問題集 | なし | 静的 |
| ソルバー | Jorbs の fight simulator（[記事](https://www.gamesradar.com/games/roguelike/for-an-unhinged-84-minutes-slay-the-spire-2-expert-contemplates-1-damage-and-explains-a-fight-simulator-built-to-solve-the-game/)） | 戦闘の最適解を探す。公開状況は不明 | なし | — |

## 2. レッドオーシャン（避ける）

- **カード報酬の採点**：4つ以上の実装があり、Untapped や MetaBot は大量のコミュニティデータを持つ。後発が勝つのは難しい。
- **統計ダッシュボード**：無料で十分なものが多く、差別化しにくい。
- **AIにプレイさせる**：STS2MCP が事実上の標準。上達支援ではなく、見て楽しむ用途。

## 3. 空白地帯（ブルーオーシャン候補）

| # | 空白 | 根拠 |
|---|---|---|
| A | **戦闘中のプレイング（戦術）のレビュー** | 既存はカード選択や統計ばかり。戦闘ログModはログを出すだけで、「このターンの最善手は何だったか」を評価するツールは見つからない |
| B | **「なぜ」を自然言語で説明する** | 調べたツールはすべて点数・ティア・勝率。理由を文章で説明して対話できるコーチは見当たらない |
| C | **自分のミスから作る練習問題** | 問題集は人力のPDFだけ。自分の実ランの局面を自動で問題にするものは無い |
| D | **マップ・イベント・休憩所の判断** | ほぼ全ツールがカード/レリックに集中している |
| E | **日本語** | 日本語対応を明記したツールは見当たらない |

## 4. 狙い所の提案：「スレスパ2の感想戦」

チェスの「ゲームレビュー」（エンジンが悪手を指摘し、AIが言葉で解説し、ミスした局面を復習させる）のSTS2版。A＋B＋C＋E をまとめて取る。

1. **記録Mod**：毎ターンの完全な状態（手札、山札、捨て札、敵の意図、バフ/デバフ、エネルギー、HP）と、実際のプレイを記録する
2. **評価エンジン**：ターン単位で探索し、実際の手と最善手の差（被ダメ差、リーサル見逃しなど）を数値で出す
3. **LLMコーチ**：差の大きいターンだけを取り上げ、日本語で理由を説明する。対話で深掘りもできる
4. **復習問題**：ミスした局面を「あなたならどうする？」形式で出題する（間隔反復）
5. 戦略面（ルート・カード取捨）は `.run` から補完する

### この形にする理由
- 戦闘の評価は **LLMだけでは誤りやすい**。シミュレータで検証した事実をLLMが説明する分担にすれば、他ツールが真似しにくい技術的な壁になる
- プレイ中に答えを見せないので、オーバーレイ系と競合しない。目的は「上手くなること」
- 取得は、まず既存の STS2MCP / HermesBridge の状態出力を読み取り専用で使えば、独自Modを後回しにできる

### リスク
- 正確なシミュレータを作るのは重い（カード・レリック・敵行動の再現、アーリーアクセスでの頻繁な変更）。→ 最初は「リーサル見逃し」「無駄な被ダメ」など **ルールで判定できるミス** に絞る
- ゲーム更新でModが壊れる
- STS2MCP 系や Untapped が LLM 解説を追加してくる可能性 → 速度と日本語、復習体験で差をつける

## 5. 次に確認すること

- STS2MCP / HermesBridge の状態スキーマに、山札・敵の意図・パワーが含まれるか（コードを確認）
- 戦闘ログMod（Nexus 100）の出力ファイルの中身
- 手元の `.run` ファイルの実物
- 未読ページ（Untapped、Deck Compass、SpireSage）の確認。現環境では接続できなかった
