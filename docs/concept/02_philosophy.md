# 問題の整理と解決策の提案
[01_problem_context.md](/docs/concept/01_problem_context.md) で既存開発の問題をまとめてきた。ここではその問題に対する解決策をまとめる。

## ドキュメントの不足
これは主にドキュメント生成のコストが高いという問題が原因だ。Vibe Codingの例のようにAIによるドキュメント生成はコストが低くおさえられる。ただしVibe Codingと同じく生成されるドキュメントの質Vibe Documentとでも呼ぶべき問題が出てくる。これを避けるには一定のフォーマットに従って仕様を書く必要がある。いままでの開発で正しくドキュメントを書いていた自体と同じものだ。
正しくフォーマットに従いさせすれば、AIによるドキュメント生成は質が高く生成コストが低くなるだろう。
AIによるSDD開発という考え方は正しい。

## 既存 SDD ツールの依存問題
私が普段利用している AI Agent は [Goose](https://github.com/block/goose) である。
Gooseの利用を前提にすることで述べる問題の解決につながるので先に述べる。

### goose の採用
いくつかgoose-sddに関係する特徴を上げる

- `goose is your on-machine AI agent, capable of automating complex development tasks from start to finish.` とあるように開発者が利用するのに適している
- 多数の LLM と連携することができ、Claude Code CLI, Gemini CLI等も利用可能。これはAPIのコスト問題を解決することにもなる。
- AI との会話も他の CLI とは異なり Terminal 上で無駄な装飾がないシンプルな画面で Vimmer の私には扱いやすい。
- MCP , Agent SKILL も利用可能。
- Recipe 機能により特定タスクを自動化することができる。
  - `goose run -recipe recipe.yaml` とすることでコマンドとして実行可能
  - `goose run -recipe recipe.yaml --interactive` とするとレシピ実行後にAIとの会話を続けることができる

詳しくは [Goose Document](https://block.github.io/goose/) を参照してほしい。

特定の定形処理をコマンドライン化できるということは、既存SDDツールとは異なり他ツールを利用するという制限がなくなる。
これを前提にできるということは Recipe とコマンド体系を整理し実行を簡単にすれば様々なタスクを自動化することが可能になる。
`goose-sdd` はこの考えを元に goose を利用した SDD のワークフローを実現するのに最適と考えている。

## 既存 SDD ツールの重厚長大
既存 SDD ツールを利用した意見として `ナッツを割るのにスレッジハンマー` という表現を見た。小さなバグ修正（ナッツ）をしようとしたところ、ツールが「4つのユーザーストーリーと合計16の受け入れ基準」という巨大なドキュメント（スレッジハンマー）を生成してしまい、かえって手間が増えたという報告だ。私個人の意見でも同様の印象だ。

これはシステム全体をドキュメント化し各機能から細かい部分までドキュメント化しようとするからと考えている。
私はドキュメントを今まで人が作成してきたようにドキュメントは正しく分離して管理するべきだと考える。
私はシステム全体のドキュメント(Macro)と各機能や特定の共通アーキテクチャ等(Micro)という2つのレイヤーを定義した。

これに従い `goose-sdd` はコマンドを大きく2つに分けた。 `goose-sdd --system ...` 、 `goose-sdd --feature ...` だ。
さらに全てのドキュメントを一度に作成すると動作が重くなり理解が困難となりやすい。

よって 例えば `goose-sdd --system` を例にあげると、以下のようサブコマンドを提供し1コマンドで1ドキュメント生成とした。 `goose-sdd --feature` も同様の考え方となっている。
また、理解が進んだり、前提が変化したりした場合には再度実行すれば対象ドキュメントを更新することが可能だ。
再実行してドキュメントを更新した場合後続のコマンドを再実行を推奨する

ちなみに各コマンドはシステムの規模にもよるが小〜中規模程度のシステムであれば数十分程度で終わるはずだ。
さらに事前にドキュメントを生成しておきそれを各コマンドで読み込ませれば会話が楽になる。

### init
コマンド: `goose-sdd --system init <language>`

`docs/sdd/` を作成し各ドキュメントの初期化を行う。

### background
コマンド: `goose-sdd --system background`

`docs/sdd/00_product_background.md` を作成、更新する。
ユーザと会話し、システムの背景や目的、全体像、概念について記述する。

### concept
コマンド: `goose-sdd --system concept`

`docs/sdd/01_system_concept.md` を作成、更新する。
`docs/sdd/00_product_background.md` を読み込みシステムのコンセプトについて記述する。
情報が足りなければユーザと会話し理解を進めて作成する。

### architecture
コマンド: `goose-sdd --system architecture`

これらを読み込みシステムのアーキテクチャついて `docs/sdd/02_system_architecture.md` に記述する。
- `docs/sdd/00_product_background.md`
- `docs/sdd/01_system_concept.md`

情報が足りなければユーザと会話し理解を進めて作成する。

### rules
コマンド: `goose-sdd --system rules`

これらを読み込みAIとの協働ルール、コーディング規約、テスト方針などついて `docs/sdd/03_project_rules.md` に記述する。
- `docs/sdd/00_product_background.md`
- `docs/sdd/01_system_concept.md`
- `docs/sdd/02_system_architecture.md`

情報が足りなければユーザと会話し理解を進めて作成する。

### glossary
コマンド: `goose-sdd --system glossary`

これらを読み込みAIとの協働ルール、コーディング規約などついて `docs/sdd/04_domain_glossary.md` に記述する。
- `docs/sdd/00_product_background.md`
- `docs/sdd/01_system_concept.md`
- `docs/sdd/02_system_architecture.md`
- `docs/sdd/03_project_rules.md`

情報が足りなければユーザと会話し理解を進めて作成する。

### ドキュメントとコードの剥離
この問題は `ドキュメントの不足` で述べたことだ。またこれにより開発者が理解ができず問題があると理解していないがら動作するだけのコードを作成し、この結果理解がさらに困難になり、次に開発されるコードがさらに問題があるという悪循環に陥る。
またこういった現場はリファクタリングやドキュメント生成等プロダクトの保守性に対してコスト負担を避けたいという意思を持っているような印象もある。

私はこれらを解決するために、ドキュメント更新のアプローチ（Flow）を双方向にする ことが重要だと考える。

- 1. Forward Sync (Design First / To-Be):
  - 新機能や改修の際、まずドキュメントを更新し、それを元にコードを変更するフロー。
  - AI（Goose）への指示書として機能し、実装の迷走を防ぐ。
- 2. Reverse Sync (Reality First / As-Is):
  - 実装中に判明した事実や、既存コードの解析結果をドキュメントに「逆流」させるフロー。
  - コード（現実）が先行して変化した場合、即座にドキュメントを現実に追いつかせることで、乖離（Drift）を解消する。

goose-sdd は、この 「2つのフロー」 を循環させることで、常に 「ドキュメントとコードが一致している状態（Single Source of Truth）」 を維持し続ける。

つまり `goose-sdd` は以下の原則を持つのが良いだろう

* Single Source of Truth is Mutable: 仕様書（Spec）が正であるが、コード（Reality）が先行して変化した場合、速やかに仕様書をコードに合わせて更新（Reverse Sync）しなければならない。
* Verification: `docs/sdd/` 以下のドキュメントと実際のコードの整合性を定期的にチェックし、乖離があれば「ドキュメントの更新」を推奨する。
