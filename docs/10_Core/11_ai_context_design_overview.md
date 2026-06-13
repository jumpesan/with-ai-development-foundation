# AI向けContext設計の整理

## この資料について

この資料は、withAI開発における **Context（文脈）設計** の概要を共有するためのものです。

ここでいうContextとは、AI Agentが作業するときに参照すべき、Project側の前提情報です。

たとえば、次のような情報を含みます。

```text
このrepositoryでは何をしてよいか
何をしてはいけないか
どのfileを読めばよいか
どのcommandで検証するか
どの判断が過去に決まっているか
どの成果物は直接編集してはいけないか
どの作業では人間確認が必要か
```

この資料では、Contextを「AI Agentの記憶」ではなく、**Project側に残す作業前提** として扱います。

AI Agentが賢くなっても、個別プロジェクトのルール、過去の判断、検証方法、触ってはいけない場所は、外から自動では分かりません。

そのため、AIに任せる作業が増えるほど、Project側にContextを残すことが重要になります。

---

## Contextがないと何が起きるか

AI Agentは一般的なプログラミング知識を持っています。

しかし、個別プロジェクト固有のルールまでは知りません。

Contextが足りないと、たとえば次のようなことが起きます。

```text
前のチャットで決めた方針を知らない
本来触ってはいけない生成物を直接編集する
テストの正しい入口を知らない
古い手順を使う
禁止されているpackage managerを使う
release noteやchangelogの運用を知らない
一時的な判断を一般ルールとして扱う
```

これは、AIの能力不足だけの問題ではありません。

人間でも、初めて入る現場で、ルール、検証手順、立入禁止領域、過去の判断を知らなければ事故を起こします。

AI Agentも同じです。

建物で例えるなら、AI Agentは「IT知識のある外部作業者」です。

知識はありますが、初めて来た現場なので、次の情報を教える必要があります。

```text
どの部屋に入ってよいか
どの道具を使ってよいか
どの場所は触ってはいけないか
作業後に何を確認するか
問題が起きたら誰に戻すか
```

Context設計は、AI Agentにこの現場情報を渡すための設計です。

---

## Contextとして残すもの

Contextは、単一のファイルだけを指すものではありません。

代表的には、次のようなものがあります。

```text
AGENTS.md
CLAUDE.md
copilot-instructions.md
Cursor rules
.github/instructions/*.instructions.md
README.md
CONTRIBUTING.md
docs/*.md
package.json scripts
CI設定
テスト手順書
PR template
```

この研究では、Contextを大きく次のように見ています。

```text
AI Agentが作業時に参照するProject側の情報資産
```

重要なのは、これらを「プロジェクト紹介文」としてではなく、AIが開発作業を進め、検証し、迷わないための **作業制御情報** として扱うことです。

---

## Contextに書くとよい情報

Contextには、抽象的な思想よりも、実際にAI Agentが使える情報を書く方が有効です。

たとえば次のような情報です。

```text
build command
test command
lint / typecheck command
使ってよいpackage manager
使ってはいけないcommand
generated fileの扱い
dependency変更時の注意
PR作成時のルール
変更種別ごとの確認手順
```

「適切にテストする」とだけ書いても、AIは具体的に何を実行すべきか判断しにくいです。

より有効なのは、次のような書き方です。

```text
UI変更時は pnpm test:e2e を実行する
server側の型変更時は pnpm typecheck を実行する
generated filesは直接編集しない
必要な場合はgenerator sourceを修正し、generate commandを実行する
```

AI Agentは自然言語を読めますが、作業では具体的な入口が必要です。

Contextは、AIに「よしなにやって」と伝える場所ではなく、AIが迷わず作業できるように具体的な導線を置く場所です。

---

## 実際に使われている形式

確認した複数の公式docsやOSS repositoryでは、AI coding agent向けに次のようなファイルが使われています。

```text
AGENTS.md
CLAUDE.md
copilot-instructions.md
.github/instructions/*.instructions.md
```

これらには、単なる概要ではなく、実際の作業で使う情報が書かれています。

たとえば、次のような情報です。

```text
対象ディレクトリのREADMEを読む
特定のbuild / test commandを実行する
成功していない検証を成功したと主張しない
生成物を直接編集しない
特定のpackage managerを使う
変更種別ごとに確認するtestを変える
```

ここから見えるのは、AI向けContextは、rootに1つ置くだけの説明文ではないということです。

作業対象に近い場所にContextを置いたり、rootのContextから詳細docsへ誘導したりする構成が使われています。

---

## Contextは1ファイルに全てを書く必要はない

Contextをすべて1ファイルに詰め込むと、長くなり、古くなり、矛盾しやすくなります。

そのため、rootのAGENTS.mdやCLAUDE.mdは、すべての詳細を書く場所ではなく、入口として使う考え方が有効です。

```text
root AGENTS.md
  全体ルール
  禁止事項
  参照先
  検証の入口

各ディレクトリのREADME / AGENTS.md
  その領域固有のルール
  その領域のtest
  その領域の注意点

README / CONTRIBUTING / docs
  詳細な手順
  開発ルール
  継続的に参照する正本
```

この構成にすると、AI Agentはrootで全体の入口を読み、必要に応じて詳細へ進めます。

Context設計では、「何を書くか」だけでなく、「どこを読ませるか」も重要です。

---

## 日本語Contextについて

日本語でもAI向けContextは成立します。

日本語チーム、日本語仕様、日本語の運用ルールを持つプロジェクトでは、日本語でContextを書く方が自然な場合があります。

ただし、次のものは翻訳せず、原文のまま扱う必要があります。

```text
コマンド
ファイル名
ディレクトリ名
package名
class名
関数名
API名
```

たとえば、説明文は日本語でよいですが、次のような識別子は変えない方が安全です。

```text
pnpm test
app/renderer/src
package.json
AGENTS.md
```

読みやすさのために日本語にする部分と、正確性のために原文のまま残す部分を分ける必要があります。

---

## 短命AgentとContext

この研究では、AI Agentを長く記憶させるよりも、作業ごとにリセットし、継続すべき情報をProject側に残す考え方が有力だと見ています。

AI Agentの会話履歴や一時的な記憶に頼りすぎると、次のような問題が起きやすくなります。

```text
前回の成功体験に引っ張られる
一時的な判断を一般ルールとして扱う
古い方針を引きずる
別作業の前提を混ぜる
前の作業の文脈が次の作業に残りすぎる
```

そのため、継続すべき情報はAgentの中ではなく、Project側に残す方が安定します。

```text
Agentは短命
ContextはProject側に残す
次のAgentも同じContextを読む
人間も同じContextを確認できる
```

この考え方にすると、毎回のAI Agentは「最新のProject情報を読んだ新人」として作業できます。

---

## 多Agent化とContext

AI Agentを増やすほど、Contextの重要度は上がる可能性があります。

複数のAgentが別々の役割を持つ場合、チャット履歴や口頭の引き継ぎだけでは前提が揃いにくくなります。

```text
設計を見るAgent
実装するAgent
レビューするAgent
テストを見るAgent
ドキュメントを書くAgent
```

このように分担するほど、Project側に明示されたContextが必要になります。

重要なのは、「Agentを増やすと危険」という話ではありません。

Agentを増やすほど、共通前提をProject側に置く重要度が上がるという話です。

```text
共通ルール
ファイル配置
禁止事項
判断済みの方針
検証方法
例外時の扱い
```

これらがProject側に残っていれば、複数Agentでも同じ前提で作業しやすくなります。

---

## Context設計の失敗パターン

Contextは有効ですが、設計を間違えると逆効果になります。

### 1. 長すぎる

長すぎるContextは、重要情報が埋もれます。

対策:

```text
rootには入口と全体ルールだけを書く
詳細はdocsや局所Contextへ分ける
```

### 2. 古くなる

古いContextは、AIに古い手順を実行させます。

対策:

```text
変わりやすい情報は直書きしすぎず、正本を参照する
例: package.json scripts、docs/test.md、CI定義など
```

### 3. READMEやdocsと二重管理になる

同じ情報を複数箇所に書くと矛盾します。

対策:

```text
README = 人間向け概要
AGENTS.md / CLAUDE.md = AI向け入口
docs/*.md = 詳細の正本
```

### 4. 抽象論ばかりで実行できない

「適切にテストする」「保守性を意識する」だけでは、AIは具体的に何をすべきか判断しにくいです。

対策:

```text
実行可能な行動に落とす
例:
- pnpm test を実行する
- UI変更時は pnpm test:e2e も実行する
- generated filesは直接編集しない
```

### 5. 書いたから守られると思い込む

AIがContextを必ず完全に守るとは限りません。

対策:

```text
重要なルールはCI、lint、test、secret scanning、PR checkなどで機械的にも検出する
```

---

## まとめ

AI向けContext設計は、AGENTS.mdやCLAUDE.mdを置くこと自体が目的ではありません。

目的は次です。

```text
AIが迷わず、安全に、検証可能に作業できる文脈導線を作ること。
```

そのためには、次の考え方が重要です。

```text
Contextは作業制御情報として設計する
rootは入口にする
詳細は正本や局所Contextに分ける
過去の判断をProject側に残す
AIの記憶に頼りすぎない
短命Agentでも読める形にする
複数Agentでも前提が揃うようにする
重要ルールはcheckで検出する
```

Contextは、AIに長く覚えさせるためのものではありません。

次のAI Agent、人間のreviewer、将来の自分が、同じ前提に戻れるようにするためのProject側の資産です。
