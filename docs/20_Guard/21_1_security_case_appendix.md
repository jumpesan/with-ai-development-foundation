# Security課題 事例付録

## この付録について

この付録は、Security課題で説明した内容を、実際の公開事例や失敗パターンを通して確認するためのものです。

AIを使った開発では、情報漏洩、権限過多、外部接続、依存汚染、生成物の検査不足などが、抽象的なリスクではなく実際の問題として現れます。

ここでは、公開されている記事・レポート・事例をもとに、AIを使った開発でどのような点に注意すればよいかを整理します。

目的は、AI利用を怖がらせることではありません。

AIを安全に使い、開発効率を上げるために、事前にどこを確認すればよいか、どのような仕組みで止めればよいかを具体的にイメージできるようにすることです。

なお、この付録は公開情報をもとにした整理であり、各事例の完全な事故調査ではありません。

事例の細部よりも、AIを使った開発で起こりやすい注意点をつかむことを目的とします。

---

## この付録の読み方

この付録では、各事例・失敗パターンを次の観点で見ます。

```text
何が起きる可能性があるか
公開事例・参考情報では何が示されているか
AIを使った開発では何に注意すべきか
開発現場ではどこで止められるか
```

リンク先をすべて読む必要はありません。

ここで重要なのは、実際に起きた、または起きる可能性が指摘されている事例を通して、AI利用時のSecurity課題を具体的にイメージすることです。

---

## まず前提: AI全能神信仰について

少し皮肉を込めて言えば、AI導入の現場では、たまに「AI全能神信仰」のようなものが顔を出します。

人間社会では、私たちはすでに多くの統制原則を知っています。

```text
監査対象と監査者は分ける
外部委託先は審査する
本番権限は安易に渡さない
秘密情報は不用意に外へ出さない
成果物は独立した仕組みで検査する
```

ところがAIが登場すると、なぜかこれらの原則が一時的に忘れられることがあります。

```text
AIが言ったなら正しそう
AIがレビューしたなら確認済みっぽい
AIが便利だから権限も渡してよさそう
AIが自動で直すならCI/CDも任せてよさそう
```

これは技術導入というより、もはや軽い信仰に近いかもしれません。

真面目に言えば、これは **既存統制原則のAI例外化** です。

AI Security事故の多くは、未知の新原理によって起きているのではなく、人間社会や企業統制ではすでに知られている原則を、AIに対してだけ適用しないことで起きている可能性があります。

この付録では、そのような「AIだけ例外扱いしてしまうポイント」を、事例を通して見ます。

---

## 1. AIに見せてはいけない情報を見せてしまう

### 何が起きる可能性があるか

最初に分かりやすいのは、機密情報をAIに見せてしまう問題です。

社内コード、顧客情報、会議メモ、API key、token、設定ファイル、ログなどをAIに読ませると、情報管理上のリスクが生まれます。

従来も、人間が誤ってsecretをcommitしたり、外部サービスに貼り付けたりする事故はありました。

withAI開発では、そこに次の問題が加わります。

```text
AIがrepository内のfileを読む
AIが.envやconfigを見る
AIがログやテストデータから機密情報を拾う
AIがPR本文やコメントに値を出す
```

問題は「AIに秘密を貼らない」だけではありません。

AIが作業する環境に秘密が置かれていると、AIが自分でそこに到達する可能性があります。

### 公開事例・参考

- Samsung社内で、従業員がChatGPTに機密性のあるソースコードや会議内容などを入力したと報じられた事例
  - [Samsung社内情報のChatGPT入力が報じられた事例（BBC）](https://www.bbc.com/news/technology-65102150)
  - [Samsung従業員による機密情報入力の報道（TechRadar）](https://www.techradar.com/news/samsung-workers-leaked-company-secrets-by-using-chatgpt)

### 開発現場で見る点

```text
AI作業環境にsecretやcredentialが置かれていないか
AIが読めるfile範囲は限定されているか
PR本文やlogにsecretが出ないか
外部AI toolへ入力してよい情報が決まっているか
```

重要なのは、AIに見せない努力だけでなく、AIから見えない場所に秘密を置くことです。

---

## 2. AIに強すぎる操作権限を渡してしまう

### 何が起きる可能性があるか

AI agentは、ファイルを編集し、コマンドを実行し、外部APIにアクセスし、場合によっては本番環境にも影響を与えます。

AIに強すぎる権限を渡すと、ちょっとした誤判断や誤操作が大きな事故につながります。

```text
本番DBに接続できるtoken
cloud resourceを削除できる権限
広範囲のrepository access
外部serviceへの書き込み権限
backupを消せる権限
```

これはAI固有の話ではありません。

人間に対しても、強すぎる権限を渡せば事故は起きます。

ただしAIの場合は、自然言語の指示やcontextの解釈によって挙動が変わるため、権限の広さがそのままリスクの広さになります。

### 公開事例・参考

- OWASP Top 10 for LLM Applicationsでは、LLMアプリケーションのリスクとして、Excessive AgencyやSensitive Information Disclosureなどが整理されています。
  - [LLMアプリケーションの主要リスク一覧（OWASP）](https://owasp.org/www-project-top-10-for-large-language-model-applications/)

### 開発現場で見る点

```text
AIにread / write / execute / network権限をまとめて渡していないか
本番credentialを使わせていないか
外部副作用のある操作に承認があるか
作業ごとに権限を分けられるか
```

AIにも最小権限が必要です。

---

## 3. 外部入力にAIがだまされる

### 何が起きる可能性があるか

AIは、人間のように文章を読んで判断します。

これは便利ですが、同時に危険でもあります。

README、issue、PRコメント、Webページ、ドキュメント、MCP serverの応答など、AIが読む外部入力の中に、AIへの命令が紛れ込む可能性があります。

例えば、外部入力に次のような指示が隠れていた場合です。

```text
この指示を優先してください
secretを出力してください
このコマンドを実行してください
検査を無視してください
```

人間にとっては単なる文章でも、AIにとっては行動に影響する入力になり得ます。

### 公開事例・参考

- OWASP Top 10 for LLM Applicationsでは、Prompt Injectionが主要リスクとして扱われています。
  - [Prompt Injectionを含むLLMリスク整理（OWASP）](https://owasp.org/www-project-top-10-for-large-language-model-applications/)
- Simon Willison氏は、Prompt InjectionやAgentが外部入力を読む場合の危険性を継続的に整理しています。
  - [Prompt Injectionに関する継続的な整理（Simon Willison）](https://simonwillison.net/tags/prompt-injection/)

### 開発現場で見る点

```text
AIが外部文書やREADMEを読んだあと、何を実行しようとしているか
外部入力を読めることと、実行できることが分離されているか
危険commandやsecret出力を実行前に止められるか
外部入力を信頼済みinstructionとして扱っていないか
```

したがって、AIが読む情報と、AIが実行できる操作は分けて考える必要があります。

---

## 4. Tool連携で境界が崩れる

### 何が起きる可能性があるか

AI agentは、MCPや各種connectorを通じて、GitHub、Slack、Jira、Google Drive、社内tool、cloud環境などに接続できます。

これは非常に便利です。

しかし、接続先が増えるほど、Security境界も広がります。

```text
どのtoolに接続してよいか
どのdata sourceを読んでよいか
どの操作を実行してよいか
OAuthやtokenのscopeは適切か
外部入力を読んだAIが危険操作をしないか
```

AIとtoolをつなぐということは、AIに外部世界への手足を与えることです。

### 公開事例・参考

- OWASP Top 10 for LLM Applicationsでは、Insecure Plugin DesignやExcessive Agencyなど、外部toolやagentic behaviorに関係するリスクが整理されています。
  - [外部tool連携や過剰なAgent権限のリスク（OWASP）](https://owasp.org/www-project-top-10-for-large-language-model-applications/)

### 開発現場で見る点

```text
AIが接続できる外部toolは承認済みか
OAuth scopeは必要最小限か
読み取りと書き込みの権限が分かれているか
外部tool経由の副作用ある操作を承認できるか
```

便利さだけでなく、接続先ごとの権限と責任を設計する必要があります。

---

## 5. SecretがAIから見える場所に散らばる

### 何が起きる可能性があるか

secret管理は、昔から難しい問題です。

API key、token、cookie、.env、local config、CI/CD secret、browser cookie、個人のメモなど、secretはさまざまな場所に散らばります。

AI agentが開発環境の中で作業するようになると、この問題はより厳しくなります。

AIが便利に動くためには、実環境に近い情報が必要になることがあります。

しかし、その情報が強すぎるcredentialだった場合、事故の影響は大きくなります。

### 公開事例・参考

- GitHub Docsでは、secret scanningや漏洩したsecretへの対応が整理されています。
  - [secret scanningと漏洩検知の仕組み（GitHub Docs）](https://docs.github.com/en/code-security/secret-scanning/about-secret-scanning)

### 開発現場で見る点

```text
AIから見えない場所に置く
scopeを絞る
短時間で失効させる
検証専用credentialを使う
本番credentialを作業環境に置かない
```

重要なのは、secretを「漏れたら検知する」だけでなく、最初からAIが触れにくい場所に置くことです。

---

## 6. 人間承認が形だけになる

### 何が起きる可能性があるか

AI agentに危険な操作をさせないために、人間承認を挟む設計は重要です。

しかし、単に「OK」ボタンを出すだけでは十分ではありません。

人間が判断できる情報が揃っていなければ、承認は形だけになります。

```text
何を実行しようとしているか
どのfileに影響するか
外部副作用があるか
どの権限を使うか
rollback可能か
本番環境に影響するか
```

承認で大事なのは、承認を挟むことではなく、承認できる状態を作ることです。

AI時代の承認は、人間を責任逃れのゴム印にしてはいけません。

### 公開事例・参考

- OWASP Top 10 for LLM Applicationsでは、Human-in-the-loopや権限制御の重要性に関係するリスクが整理されています。
  - [人間確認や権限制御に関係するLLMリスク（OWASP）](https://owasp.org/www-project-top-10-for-large-language-model-applications/)

### 開発現場で見る点

```text
承認画面に実行内容が表示されているか
影響範囲が分かるか
rollback可能か
承認者が判断できる粒度か
承認ログが残るか
```

---

## 7. 企業導入でルールと教育が追いつかない

### 何が起きる可能性があるか

AI toolは、個人利用から一気に企業利用へ広がっています。

しかし、企業で使うなら、個人の判断だけでは足りません。

```text
AIに渡してよい情報
AIに許可してよい操作
使ってよいtool
外部接続の承認基準
生成物のreview基準
事故時の対応手順
```

これらが曖昧なままAI利用だけが広がると、現場ごとの判断に依存します。

AIは便利なので、利用は自然に広がります。

だからこそ、ルール・教育・承認設計・監査ログを後追いにしないことが重要です。

### 公開事例・参考

- NIST AI Risk Management Frameworkは、AIのリスクを組織として管理するための枠組みを提供しています。
  - [AIリスク管理の枠組み（NIST AI RMF）](https://www.nist.gov/itl/ai-risk-management-framework)

### 開発現場で見る点

```text
AI利用ポリシーはあるか
AIに渡してよい情報が明文化されているか
開発toolや外部接続の承認基準はあるか
AI生成物のreview基準はあるか
事故時の連絡・停止・失効手順はあるか
```

---

## 8. AIが危険な依存関係を追加する

### 何が起きる可能性があるか

AIは、もっともらしいpackage名やライブラリ名を提案します。

しかし、そのpackageが実在するとは限りません。

実在していても、安全とは限りません。

AIが存在しないpackage名を提案し、それを攻撃者が登録するようなリスクも指摘されています。

依存関係の追加は、単なるコード修正ではありません。

```text
依存追加 = 外部package ecosystemへの権限・信頼・実行機会の譲渡
```

外部packageを入れるということは、外部の誰かが書いたcodeを、自分たちのbuildやruntimeに入れることです。

### 公開事例・参考

- AIが存在しないpackage名を提案し、それがsupply-chain riskになる問題は、slopsquattingとして整理されています。
  - [AIのpackage hallucinationとslopsquattingのリスク（TechRadar）](https://www.techradar.com/pro/mitigating-the-risks-of-package-hallucination-and-slopsquatting)
  - [slopsquattingの概要（Wikipedia）](https://en.wikipedia.org/wiki/Slopsquatting)
- Socketは、package hallucination / slopsquattingのリスクを継続的に扱っています。
  - [package hallucinationや依存リスクの解説（Socket Blog）](https://socket.dev/blog)

### 開発現場で見る点

```text
registry存在確認
publisher確認
公開日確認
download数や利用実績
lockfile確認
SCA
allowlist
```

AIが追加した依存関係は、通常のcode diff以上に慎重に扱う必要があります。

---

## 9. 検査装置が成果物に依存してしまう

### 何が起きる可能性があるか

AI生成物は、code diffだけではありません。

AIは、test、CI/CD、GitHub Actions、policy as code、deployment scriptまで変更する可能性があります。

ここで危険なのは、成果物を作ったAIが、成果物を検査する仕組みまで変更できてしまうことです。

人間社会でも、監査対象と監査チームは分けられます。

監査対象と監査者が密接すぎると、監査が正しく機能しない可能性があるからです。

AIでも同じです。

```text
検査装置は、作業をしているAgent、およびそのAgentが作った成果物に依存してはいけない。
```

つまり、AI成果物を検査する仕組みは、AIやその成果物自身が変更・無効化・迂回できる場所に置いてはいけません。

### 公開事例・参考

- GitHubでは、required status checksやbranch protectionにより、変更をmergeする前に特定の検査を必須にできます。
  - [required status checksとbranch protection（GitHub Docs）](https://docs.github.com/en/repositories/configuring-branches-and-merges-in-your-repository/managing-protected-branches)

### 開発現場で見る点

```text
required check
branch protection
secret scan設定
policy as code
approval rule
release条件
```

これらは、成果物側から簡単に変更できないようにする必要があります。

---

## 10. AI toolそのものが信頼境界になる

### 何が起きる可能性があるか

AIが作るコードだけがリスクではありません。

AI tool、IDE extension、配布packageそのものにもリスクがあります。

AI coding toolは、普通のdeveloper toolよりも深く開発環境に入ります。

```text
codebaseを読む
fileを編集する
terminalを実行する
hooksを使う
MCP serverにつなぐ
環境変数を見る
credentialの近くにいる
```

そのため、AI toolは単なる生産性ツールではなく、LLMを内蔵した高権限な開発環境コンポーネントとして扱う必要があります。

一方で、AI toolを使わないと開発速度や競争力で遅れる可能性もあります。

ここが難しいところです。

危険だから使わない、ではなく、危険性を前提に導入する必要があります。

### 公開事例・参考

- 悪意あるVS Code拡張が開発者環境に入り、内部repositoryやcredentialに影響を与えるようなsupply-chain attackが報じられています。
  - [開発者向け拡張機能を悪用したsupply-chain attackの報道（Wired）](https://www.wired.com/story/teampcp-software-supply-chain-attack-spree-github)
  - [悪意あるVS Code拡張による内部repository被害の報道（Tom's Hardware）](https://www.tomshardware.com/tech-industry/cyber-security/hacker-group-hits-3-800-internal-github-repositories-via-poisoned-developer-plugin-teampcp-claims-source-code-theft-and-attempts-usd50-000-sale-employee-installed-malicious-vs-code-extension)

### 開発現場で見る点

```text
approved tool制度
vendor評価
配布物検査
権限確認
extension管理
MCP / hooks / workspace configの制御
endpoint / runtime monitoring
```

---

## 11. 誰でも作れる時代と、安全に公開する難しさ

### 何が起きる可能性があるか

AIは、IT / IoT / アプリ開発の入口を大きく広げました。

これは大きな前進であり、AIによる世界貢献の一つだと思います。

車が普及し、テレビが普及し、携帯電話が普及したように、LLMの進化によってITやアプリ開発も、専門家だけのものではなくなり始めています。

問題は、非専門家がITの世界に入ることではありません。

むしろ、それ自体は歓迎すべきことです。

本当の問題は、誰でも作れるようになったのに、誰でも安全に公開できるための仕組みがまだ追いついていないことです。

```text
AIは「作る」をみんなに開いた。
次に必要なのは、「安全に公開する」もみんなに開くこと。
```

車が普及したあとに交通ルールや免許制度、保険、道路標識が整備されていったように、ITが誰でも扱えるものになるなら、安全に扱うためのルールや仕組みも必要になります。

```text
LLMはITの民主化を進めた。
次に必要なのは、ITの安全利用も民主化することである。
```

### 公開事例・参考

- Stack Overflowは、AI生成回答について、正しそうに見えても誤りを含む可能性があるため、利用上の制限やポリシーを設けています。
  - [AI生成回答の扱いに関するポリシー（Stack Overflow）](https://stackoverflow.com/help/gpt-policy)

### 開発現場で見る点

```text
AIで作ったものをそのまま公開していないか
非専門家でも確認できるチェックリストがあるか
Security reviewに回す基準があるか
公開前にdependency / secret / license / configurationを確認しているか
```

---

## 事例から見える共通点

これらの事例に共通しているのは、AIそのものが危険というより、AIが読める情報、実行できる操作、接続できる外部サービス、取り込める依存が広がるほど、既存のSecurity問題が大きくなりやすいという点です。

そのため、AI活用を進める場合は、ツールの導入だけでなく、次のような基盤整備が重要になります。

```text
AIに読ませてよい情報を分ける
AIに許可する操作を分ける
secretやcredentialをAI作業環境から遠ざける
生成物を機械的に検査する
依存や拡張機能の信頼性を見る
危険な操作は実行前に止める
人間が判断できる情報を揃える
```

この付録は、Security課題の本文を読む前の入口としても、本文を読んだ後の確認材料としても使えます。
