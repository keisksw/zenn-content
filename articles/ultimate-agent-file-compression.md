---
title: "エージェントファイル、肥大化してない？「公理と表」でLLMのシステムプロンプトを極限圧縮する"
emoji: "🔬"
type: "tech"
topics: ["llm", "promptengineering", "cursor", "ai", "gemini"]
published: true
---

「エージェントファイル（Agent File）」って知っていますか？

Antigravity、Cursor、Clineなど、自律型LLMコーディングアシスタントを使う上で、ほぼ全ての人が `.cursorrules` や `GEMINI.md` のような「システムプロンプト」をリポジトリやルートに置いていると思います。

「&& を使ってコマンドを繋げないで」「知らないコマンドは実行前にパスを確認して」「嘘をつかないで」。
LLMが失敗するたびにルールを書き足していくと、あっという間にエージェントファイルは **10,000〜30,000文字（10KB〜30KB）** という巨大な壁に激突します。

そしてどうなるか？ **「健忘症（エージェント・アムネジア）」** です。
LLMのコンテキストウィンドウ自体は大きいですが、システムプロンプトが長くなりすぎると、LLMは下の方に書かれた重要な制約（待機時間の上限設定など）を**サイレントに読み飛ばす（トランケーション）**ようになります。

今回、この「プロンプト肥大化による指示無視」を完全に解決するため、コンピュータサイエンス（CS）と情報理論に基づき、30KBのエージェントファイルを **わずか5KB（約83%圧縮）** まで劇的にダイエットしつつ、LLMの指示遵守率を爆増させる **「Axiom（公理）とテーブルによる極限圧縮フレームワーク」** を考案し、GitHubに公開しました。

## 解決策：情報の最小的伝達（Minimal Information Transfer）

単に文字数を削るだけでは、LLMは推論の文脈を失い、単なる「コマンド叩きマシーン」あるいは無能なアシスタントに成り下がります。
圧縮しつつ、知能を高く保つためには以下の2つのテクニックを用います。

### テクニック1：行動制御の「公理化（Axiom）」

LLMに対して、「人間のように懇切丁寧に理由を説明すること」はやめましょう。LLMは論理を処理するエンジンなので、「Axiom（公理）」という宣言を与えられると、無条件にそれに従う傾向があります。

❌ **ダメな例（文章でお願いする）**
> 「私は別の仕事で忙しいので、同じエラーで私に何度も作業をさせないでください。次も同じことが起こらないように仕組みで解決してもらえると助かります。」

✅ **良い例（Axiomによる絶対定義）**
> `Axiom: Never propose solutions requiring the user to act again for the same issue. The user repeating an instruction is agent negligence.`
> (公理：同じ問題でユーザーに再行動を要求する解決策を提案してはならない。ユーザーが指示を繰り返すことは、全てエージェントの怠慢である)

長々としたお願いを廃止し、Axiom（絶対法則）として定義し直すだけで、文字数は減り、モデルのAttentionは強烈に固定されます。

### テクニック2：技術定数の「純粋テーブル化」

環境変数や、OS依存の制約、コマンドの待機時間上限などを箇条書きで書くのはプロンプトエンジニアリングにおける「罪」です。

LLM（Transformerアーキテクチャ）は、Markdownテーブルを「構造化された高密度データ」として極めて効率的にパースします。散文をテーブル（表）に変換するだけで、トークン消費量を劇的に抑えつつ、読み飛ばしのリスクをゼロに近づけることができます。

❌ **ダメな例（箇条書き）**
> - zshを使うこと。
> - macOS環境なので timeout は使えません。代わりに ConnectTimeout を使って。
> - flock も使えません。mkdir を使ってくださいね。
> - sedを使うときは必ず、sed -i '' と空の引数を渡してください。

✅ **良い例（Markdownテーブル化）**
> ```markdown
> ### OS/Shell Constraints
> *Axiom: Setup reflects the user's environment. Adjust to Linux/macOS/Windows.*
> | Command | Status / Requirement |
> |---|---|
> | `flock` / `timeout` / `date -d` / `stat -c` | ❌ macOS incompatibility (use mkdir lock, `-o ConnectTimeout`, `stat -f %m`) |
> | `sed -i` | ⚠️ macOS requires: `sed -i ''` |
> ```

## 究極のテンプレートを公開しました

この手法を使って完成したのが、今回公開する **Ultimate Agent File Boilerplate** です。

🔗 **[GitHub Repository: keisksw/ultimate-agent-file](https://github.com/keisksw/ultimate-agent-file)**

このテンプレートをローカルにクローンし、 `GEMINI.md` または `.cursorrules` としてルートに置いてみてください。そして、テーブルの中の「ダミー変数」をあなたの環境に合わせて書き換えるだけです。

これを通すだけで、あなたのAgentの動作が見違えるほどすっきりし、「アレをやれってさっきも言っただろ💢」とブチ切れる回数が劇的に減るはずです。

もし「エージェントファイル」が肥大化して困っている人がいたら、ぜひこの「公理とテーブル」の切り分けテクニックを試してみてください。
