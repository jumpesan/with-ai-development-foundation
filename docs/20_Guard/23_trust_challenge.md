# withAI開発で気をつけたい「Trust」の話

## これは何の話か

AI Agentを使って開発していると、Agentはさまざまなものを提案したり、取得したり、実行しようとします。

たとえば、次のようなものです。

```text
npm / PyPI package
VS Code拡張
GitHub repository
build script
CI設定
Dockerfile / devcontainer設定
release artifact
READMEやIssueコメント
```

人間が見れば「ちょっと怪しい」と思えるものでも、AI Agentはそのまま信じて作業に進んでしまうことがあります。

Trust課題は、AI Agentが扱うものを **どの段階で、どう信用判断するか** という課題です。

---

## なぜ問題になるのか

AI Agentは、必要なpackageやtoolを自分で提案することがあります。

```text
このpackageを入れれば解決できます
この拡張機能を使うと便利です
このcommandを実行してください
このrepositoryを参考にします
```

このとき、次のような問題が起きる可能性があります。

```text
存在しないpackage名を提案してしまう
似た名前の悪意あるpackageを入れてしまう
汚染された拡張機能を使ってしまう
READMEやIssueコメント内の危険な指示を信じてしまう
build scriptやCI設定に危険な処理が混ざる
```

AI Agentは作業が速いので、間違ったものを信じた場合の影響も速く広がります。

---

## packageや依存を信じる前に見ること

まず、AIが提案したpackageや依存は、そのまま信じない方が安全です。

確認したいことは、たとえば次です。

```text
そのpackageは本当に存在するか
名前が似た別packageではないか
typosquattingではないか
最近作られた怪しいpackageではないか
maintainerは信頼できるか
repositoryは活発に管理されているか
security policyや更新状況はどうか
```

AIがもっともらしいpackage名を出しても、実在しない場合があります。

また、実在していても、そのpackageやrepositoryが安全とは限りません。

そのため、Trustでは次の2段階を分けて考えると整理しやすいです。

```text
1. 実在するか
2. 実在するとして信頼できるか
```

---

## artifactやreleaseを信じる前に見ること

packageだけでなく、build artifactやreleaseもTrustの対象です。

```text
このbinaryはどこでbuildされたのか
誰がreleaseしたのか
source codeと対応しているのか
署名やprovenanceはあるか
CI/CDの結果と対応しているか
```

実務では、SLSA、Cosign、GitHub Artifact Attestations、OpenSSF Scorecardのような仕組みが、こうした信頼性確認に使われることがあります。

ここで大事なのは、これらのtoolを入れれば完璧という話ではありません。

AI Agentが扱うものに対して、出所や管理状態を確認する視点が必要だということです。

---

## 「ただのテキスト」も信用対象になる

AI Agent時代では、危険なのはpackageやbinaryだけではありません。

README、Issue、PRコメント、package description、設定ファイルなどの「ただのテキスト」も、Agentにとっては行動の材料になります。

たとえば、Agentが以下のようなものを読んだ場合です。

```text
READMEに書かれた手順
Issueコメント内のcommand
package説明文
CI設定
拡張機能metadata
```

人間にとってはただの説明文でも、AI Agentはそれを作業指示として扱う可能性があります。

そのため、この資料では「何を取り込むか」だけでなく、「取り込んだものを読んだAgentが次に何をしようとしているか」を見ることが重要ではないか、と考えています。

---

## 実行前に見るという考え方

ここから先は、現時点でこの研究が有力だと考えている仮説です。

危険な自然言語をすべて事前に判定するのは難しいです。

しかし、Agentが次に実行しようとしている操作は、比較的確認しやすいです。

```text
fileを読む
fileを書く
commandを実行する
packageをinstallする
workflowを変更する
networkへ接続する
secretに触る
artifactをpublishする
deployする
```

そこで、現時点では **実行前のGate** を置く考え方が有力ではないか、と見ています。

```text
Agentが次に何をしようとしているか
今の状態はどうか
接続先はどこか
権限は何か
変更対象は何か
policy上許されるか
```

このような情報を見て、以下のように判定する形が有効ではないか、という仮説です。

```text
OK
NG
条件付きOK
人間確認へ戻す
```

これは必ずしもLLMが判断する必要はないかもしれません。

```text
allowlist
denylist
policy engine
static analyzer
dependency scanner
diff analyzer
secret scanner
```

のような仕組みでも、かなり機械的に判断できる可能性があります。

---

## Worker Agentと判定役は分ける

現時点では、作業するAgentと、許可するか判断する仕組みを分ける設計が有力だと考えています。

```text
Worker Agent:
- codeを書く
- packageを提案する
- commandを組み立てる
- fileを変更する

Trust Gate / Policy Gate:
- 外部接続しない
- secretを持たない
- 実行しない
- 現在状態と提案操作を見る
- OK / NG / 条件付きOKを返す
```

作業Agent自身に「これは安全ですか？」と聞くだけだと、判断が曖昧になる可能性があります。

安全側に寄せるなら、作業Agentとは別に、限定された情報だけを見て機械的に判定する仕組みを置く方がよいのではないか、というのが現在の見立てです。

---

## Trustを4段階で見る

Trust課題は、現時点では次の4段階で見ると整理しやすいと考えています。

```text
1. Pre-ingest Trust
   入れる前に見る。
   package名、実在性、typosquatting、allowlist、repository状態など。

2. Pre-execution Trust
   実行する前に見る。
   次アクション、変更対象、外部接続、secret接触、workflow変更など。

3. Runtime Trust
   実行中の権限と接続を制限する。
   network、filesystem、credential、sandboxなど。

4. Post-incident Trust
   入ってしまった後に検知・限定・回復する。
   token revoke、workflow rollback、dependency rollback、artifact revokeなど。
```

入口だけで完全に防ぐのは難しいため、入る前・実行前・実行中・入った後に分けて考える方が整理しやすいと見ています。

---

## withAI開発で伝えたいこと

AI Agentが提案するものは、便利でもありますが、そのまま信じるにはリスクがあります。

```text
そのpackageは本物か
そのrepositoryは信頼できるか
そのartifactは正しい出所か
そのREADMEの手順を実行してよいか
そのcommandは今の権限で実行してよいか
```

この資料では、Agentが何を信じて作業しているかを見えるようにすることが重要ではないか、と考えています。

Trust課題は、AI Agentの判断力を疑うためのものではありません。

AI Agentを安全に使うために、**信じる前・実行する前・動いている間・問題が起きた後** の確認点を作るための課題です。
