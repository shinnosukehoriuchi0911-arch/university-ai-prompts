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

## 付録: GitHub Actions による自動検査ワークフロー

リポジトリの `.github/workflows/ai-code-audit.yml` に以下を配置することで、プルリクエスト時にAI生成コードのシークレット漏洩やライセンス汚染を自動検知できます。

```yaml
name: "AI Code & License Compliance Audit"
on: [push, pull_request]

jobs:
  audit:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - name: "Scan for hardcoded secrets and copyleft contamination"
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
          Write-Host "Compliance audit passed!"
```
