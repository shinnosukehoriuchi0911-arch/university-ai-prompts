# AI生成コード ライセンス汚染防止プロンプト（無料版）

GitHub CopilotやChatGPT、Claude等のAIアシスタントを使ってプログラミング課題やオープンソース開発を行う際に、GPL/AGPLなどの意図せぬライセンス汚染やシークレット漏洩を防ぐためのプロンプトです。

## プロンプト

```
あなたはオープンソースコンプライアンスの専門家です。
以下のコードをレビューし、次の3点を確認してください。
1. GPL/AGPL等のコピーレフトライセンスに抵触する恐れがある特異なロジックやコメントが含まれていないか
2. APIキーやパスワードなどのシークレット情報がハードコードされていないか
3. 実在しない架空の外部ライブラリをインポートしていないか

【コード】
[あなたのコードをここに貼り付け]
```

## 使い方
1. AIにコードを生成させた後、コミットや提出を行う前に上記プロンプトで自己検査を行います。
2. 指摘された箇所があれば、標準ライブラリ（Python標準ライブラリなど）を用いた独自実装に修正します。

---

## 付録: GitHub Actions による簡易点検の例（補助）

リポジトリの `.github/workflows/ai-code-audit.yml` に以下を配置すると、プルリクエスト時に `api_key` / `secret_token` という文字列が含まれるファイルを知らせます。これは文字列検索だけの簡易な点検で、シークレット漏洩やライセンス上の問題の有無を保証・判定するものではありません。実際のライセンス確認は依存関係ごとに人が行ってください。

```yaml
name: "Simple Keyword Check (api_key / secret_token)"
on: [push, pull_request]

jobs:
  audit:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - name: "Grep for api_key / secret_token strings"
        shell: pwsh
        run: |
          $files = Get-ChildItem -Recurse -Include *.py,*.js,*.ts,*.go,*.java
          foreach ($f in $files) {
              $c = Get-Content $f.FullName -Raw
              if ($c -match "(?i)(api[_-]?key|secret[_-]?token)") {
                  Write-Error "Potential secret detected in $($f.Name)"
                  exit 1
              }
          }
          Write-Host "Keyword check finished (no match; this is not a license or security audit)."
```
