# withAI開発基盤 Guard 目次・前書き

## この文書の目的

この文書は、withAI開発基盤を考えるための **Guard課題** の入口です。

Guard課題とは、AI coding agent（AIに開発作業を任せる仕組み）を開発現場で使うときに、危険な操作、情報漏洩、環境汚染、信頼できない依存、生成物の誤修正などをどう防ぐかを考えるための整理です。

Core課題が「AIに作業してもらうための基本設計」だとすると、Guard課題は「AIに安全に作業してもらうための境界設計」です。

詳細は各Markdownを参照します。

---

## 注意書き

AI開発支援ツールやAI coding agentの状況は、非常に速く変化しています。

この資料は、2026年6月時点の公開情報・事例・整理内容をもとにした現時点での整理です。  
半年後には、ツール側の機能、ベストプラクティス、リスク、開発現場での使われ方が変わっている可能性があります。

---

## Guardの読む順番

1. [20. Guard目次](./20_guard_index.md)
2. [21. Security](./21_security_challenge.md)
3. [21.1 Security課題 事例付録](./21_1_security_case_appendix.md)
4. [22. Sandbox](./22_sandbox_challenge.md)
5. [23. Trust](./23_trust_challenge.md)
6. [24. 生成物](./24_generation_challenge.md)
7. 25. 反面教師の扱い（現時点では独立ファイルではなく、各Guard課題を見た後の横断的な補助線として扱います）

現時点では、反面教師は独立章として深掘りするよりも、各Guard課題を見た後に横断的に扱う補助線として整理しています。

---

## 21. Security

対象ファイル:

- [21_security_challenge.md](./21_security_challenge.md)
- [21_1_security_case_appendix.md](./21_1_security_case_appendix.md)

Securityは、AI Agentが危険な操作をしたり、秘密情報を漏らしたりしないようにする課題です。

例:

```text
secretを読ませない
credentialをcommitしない
PR前に漏洩を検知する
外部送信を制限する
CI/CDやrepositoryへの影響を抑える
```

AI Agentは、人間より速く作業できます。  
その一方で、間違った操作や漏洩も速く広がる可能性があります。

Securityでは、AI Agentが何にアクセスできるのか、どこで止めるのか、どう検知するのかを考えます。

---

## 22. Sandbox

対象ファイル:

- [22_sandbox_challenge.md](./22_sandbox_challenge.md)

Sandboxは、AI Agentの作業環境をどう隔離するかを扱います。

例:

```text
git worktreeで作業面を分ける
containerで開発環境を分ける
remote dev environmentを使う
microVMやexecution sandboxで実行環境を分ける
```

Sandboxは「入れれば全部安全」というものではありません。

```text
差分を分けたいのか
依存関係を分けたいのか
OSレベルで隔離したいのか
secretへの到達を制限したいのか
外部通信を制限したいのか
```

目的によって必要な隔離の層が変わります。

---

## 23. Trust

対象ファイル:

- [23_trust_challenge.md](./23_trust_challenge.md)

Trustは、AI Agentが提案したもの・取得したもの・実行しようとするものを、どう信用判断するかを扱います。

例:

```text
AIが提案したpackage名は実在するのか
実在するとして信頼できるpackageか
VS Code拡張やnpm packageが汚染されていないか
artifactやreleaseの出所は確認できるか
READMEやIssueコメントに危険な指示が含まれていないか
```

Trustで重要なのは、入口で信用できるものだけを選ぶことだけではありません。

もし汚染されたものが入ってきた場合に、被害を限定し、検知し、回復することも含みます。

---

## 24. 生成物

対象ファイル:

- [24_generation_challenge.md](./24_generation_challenge.md)

生成物課題は、AI Agentが「生成されたファイル」と「元になる正本」を取り違えないようにする課題です。

例:

```text
OpenAPI Specから生成されたclient code
.protoから生成された各言語のcode
schemaから生成されたSDKやdocs
```

生成されたファイルを直接直すと、一時的には動いても、次に生成し直したときに変更が消えることがあります。

そのため、AI Agentには以下を分かるようにしておく必要があります。

```text
どれが正本か
どれが生成物か
生成コマンドは何か
検証コマンドは何か
生成物を直接編集してよい例外はあるか
```

---

## 25. 反面教師の扱い

現時点では、反面教師は独立した深掘り章としては扱いません。

Security / Sandbox / Trust / 生成物の中で、すでに失敗例や事故例を見ているためです。

今後は、各課題で見えた失敗例を、以下へ変換するための横断的な補助線として扱います。

```text
AGENTS.mdのルール
project-local AGENTS.mdのルール
チェックリスト
Guard rule
検知項目
人間承認に戻す条件
```

つまり、反面教師は「怖い事例を集める」ためではなく、失敗から設計ルールを作るために使います。

---

## Guardで大事な考え方

Guardは、AI Agentを使わないための考え方ではありません。

AI Agentを使う前提で、次を明確にする考え方です。

```text
どこまで読めるか
どこまで書けるか
何を実行できるか
どこへ接続できるか
どこで人間判断に戻すか
どう検知するか
どう戻すか
```

AI Agentは、正しく使えば開発を大きく進められます。

そのためには、Agentに任せる作業だけでなく、Agentを止める場所、隔離する場所、信用判断する場所も設計する必要があります。

---

## 今後の予定

この文書はGuardの目次です。

現在公開している資料は、研究全体のうち整理が進んだ一部です。  
後続のまとまりが完了したら、それぞれ同じように目次・前書きを作成します。

Guardで扱った境界設計は、今後追加する章や横断的な整理にも接続していく予定です。
