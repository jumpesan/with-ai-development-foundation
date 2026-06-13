# Sandbox課題

## この文書について

この文書は、AIネイティブ開発基盤における **Sandbox課題** を紹介するものです。

ここでいうSandboxは、単一の製品や技術だけを指すものではありません。

AI Agentを開発プロセスに導入する場合、重要になるのは「AIをどこで作業させるか」「どの範囲を触らせるか」「どこで実行させるか」です。

そのため、この研究ではSandbox課題を、作業面の分離、開発環境の分離、実行環境の分離、microVM基盤までを含めた、広い意味での **Sandboxes** として整理しています。

---

## 1. Sandbox課題とは

Sandbox課題とは、AI Agentの作業環境をどのように分けるかという課題です。

AI Agentは、ファイルを読み、コードを書き、テストを実行し、依存関係を追加し、外部APIや開発toolに接続する可能性があります。

このとき、すべての作業を同じ環境で行わせると、次のような問題が起きる可能性があります。

```text
作業差分が混ざる
branchやworking treeが衝突する
テスト実行で環境が汚れる
依存追加で外部コードが入る
secretやcredentialに触れる
外部通信や副作用ある操作が起きる
```

そのため、この研究ではAI Agent導入時に、作業内容とリスクに応じて環境を分けることが重要ではないか、と見ています。

---

## 2. 分離レイヤー

Sandboxは単一の技術ではなく、分離したい対象によって複数のレイヤーに分けられます。

### 2.1 作業面の分離

Git worktreeのような仕組みを使うと、同じrepositoryに対して複数のworking treeを持てます。

これにより、複数のAI Agentが同時にファイル編集やパッチ作成を行っても、作業差分やbranchの衝突を抑えやすくなります。

これは強いセキュリティ境界ではありません。

しかし、軽量で扱いやすく、Markdown修正、設定ファイル修正、後続パッチ作成のような軽作業に向く可能性があります。

```text
向いている作業:
- Markdown修正
- 設定ファイル修正
- 小さなコード修正
- 後続パッチ作成
- branchごとの並列作業
```

### 2.2 開発環境の分離

DevPodやdevcontainer系の仕組みは、開発環境そのものを再現可能にするためのレイヤーです。

人間やAgentが使う開発環境を揃えたり、ローカル・リモート・クラウド上にworkspaceを作ったりできます。

この研究では、作業環境の再現性やチーム利用のしやすさに寄与する層として見ています。

```text
主な関心:
- 開発環境の再現性
- workspaceの管理
- IDE接続
- local / remote / cloud の切り替え
- devcontainer定義の利用
```

### 2.3 実行環境の分離

AI Agentが生成したコードを実行したり、testやlintを走らせたり、依存関係を追加したりする場合は、単なるworktree分離では不十分になる可能性があります。

この場合は、Container、VM、managed sandboxなどを使って、実行環境を分ける設計が有効ではないか、と考えています。

Modal Sandboxes、E2B、Daytona、ERAのような候補は、この実行環境分離に近いものとして見られます。

```text
向いている作業:
- test実行
- build実行
- lint実行
- AI生成コードの実行
- 依存関係の追加
- 外部通信を伴う検証
```

### 2.4 microVM基盤

FirecrackerのようなmicroVM技術は、より強い実行隔離を軽量に実現するための基盤技術です。

Firecracker自体は、直接すべての開発者が使う製品というより、AI Agent向けのmanaged sandboxや安全な実行環境を支える下層技術として見ると分かりやすいです。

```text
主な関心:
- containerより強い隔離
- 通常VMより軽い実行境界
- untrusted code execution
- managed sandboxの下層基盤
```

---

## 3. 実運用上の見方

AI Agent導入時に、すべての作業を重いVMやContainerで実行する必要があるとは限りません。

現時点では、軽いファイル編集やパッチ作成はGit worktreeで分け、テスト実行や依存追加など危険度の高い作業だけをVM / Container / sandbox側に寄せる構成が現実的ではないか、と見ています。

```text
軽い作業:
- git worktreeで分離する

危険度の高い実行:
- VM / Container / managed sandboxで分離する

さらに強い境界が必要な実行:
- microVM系の基盤を検討する
```

つまり、この研究ではSandboxを「AI Agentを閉じ込めるための単一技術」ではなく、作業内容とリスクに応じて分離レイヤーを選ぶための考え方として整理しています。

---

## 4. 候補の見方

現時点では、各候補を次のように見ると整理しやすいです。

```text
Git worktree:
作業面・差分・branch・workspaceの分離

DevPod:
開発環境・workspace・devcontainer的な分離

Daytona:
AI Agent向けの開発環境・sandbox infrastructure

Modal Sandboxes / E2B / ERA:
AI生成コードやuntrusted codeの実行環境分離

Firecracker:
microVMによる強めの隔離基盤
```

ここで重要なのは、これらを同じ強度のSandboxとして扱わないことです。

それぞれが分離している対象は異なります。

```text
作業面を分けるのか
開発環境を分けるのか
実行環境を分けるのか
基盤技術として隔離境界を提供するのか
```

この違いを明確にしたうえで、AI Agentにどの作業を任せるかに応じて選択する必要があるのではないか、と考えています。

---

## 5. まとめ

Sandbox課題は、特定のSandbox製品を選ぶ話ではありません。

この研究では、AI Agent導入時に、作業環境と実行環境をどう分けるかを考えるための章として扱っています。

軽い編集作業はworktreeで分け、実行や検証はVM / Container / managed sandboxへ寄せ、より強い隔離が必要な場合はmicroVM基盤を検討する。

このように、AI Agentの作業内容とリスクに応じて、複数の分離レイヤーを組み合わせることが有効ではないか、というのが現時点の見立てです。
