# Community Health Files

kentanishida の全リポジトリで共通利用されるコミュニティヘルスファイルを管理するリポジトリです。

個別のリポジトリに同名ファイルが存在しない場合、このリポジトリのファイルが自動的に適用されます。

詳細は [GitHub Community Health Files](https://docs.github.com/ja/communities/setting-up-your-project-for-healthy-contributions/creating-a-default-community-health-file) を参照してください。

## 共通 reusable workflow

`.github/workflows/` 配下の `_` 始まりのファイルは、他リポジトリから呼び出される再利用可能ワークフローです。自動適用されるコミュニティヘルスファイルとは異なり、呼び出し元での明示的な参照が必要です。

本リポジトリは public のため、private リポジトリからもアクセス設定なしで呼び出せます。呼び出し元は `uses: kentanishida/.github/.github/workflows/{ファイル名}@main` の形式で参照します。

これらのワークフローを変更する際は、呼び出し元リポジトリすべてに即座に反映される点に注意してください。

### `_sendPushoverNotification.yml`

Pushover へ通知を送ります。

- inputs: `messageTitle`（必須）、`messageText`（必須）
  - 値はそのまま Pushover へ送ります。改行・引用符・記号はエスケープせずに渡してください（エスケープに使った `\` などの文字が、そのまま通知に表示されます）
    - YAML としての書き方は通常どおりです（単引用符の中の `'` は `''` と書く、複数行はブロック記法 `|` で書く、など）
  - 長さの上限は [Pushover の仕様](https://pushover.net/api#limits)で、`messageTitle` が 250 文字、`messageText` が 1,024 文字です（バイト数ではなく文字数）。このワークフローは切り詰めないため、上限内で渡してください
- secrets: `PUSHOVER_API_TOKEN`（必須）、`PUSHOVER_USER_OR_GROUP_KEY`（必須）
  - 認証情報は呼び出し元リポジトリの secrets から渡す設計で、本リポジトリには保持しません
  - Dependabot が起点の run では、Actions secrets ではなく Dependabot secrets の値が渡ります。そこに値が無いと空のまま送って拒否され、ジョブが失敗します。Dependabot の run でも通知するなら Dependabot secrets にも同じ 2 つを登録し、通知しないなら Dependabot の run では呼び出さないでください
- 前提: `runs-on` に Ubicloud のランナーラベルを指定しています。ジョブは呼び出し元リポジトリの実行環境で解決されるため、Ubicloud を導入していないリポジトリから呼ぶとランナーが割り当てられません
- 失敗時: Pushover に受け付けられたことを確かめられなかった場合は、ジョブが失敗し、呼び出し元の run も failure になります
  - 対象は、HTTP 400 以上の応答、応答本文の `status` が 1 以外（本文が JSON でない・空の場合を含む）、名前解決や接続ができない・応答が途中で切れるなど応答を受け取りきれない場合、60 秒以内に応答が返らない場合です
  - log には応答本文と HTTP の状態コードが、応答が無い場合は curl のエラーが残ります
  - `status` が 1 でも、確かめられるのは Pushover が受け付けたことまでで、端末に届いたかまでは確かめません
  - 送り直しはしません。4xx か `status` が 1 以外は、送り直しても通らないと Pushover の文書にあるためです。応答が無い場合は受け付けられていることもあり、送り直すと同じ通知が重なって届くためです（5xx は送り直してよいとされていますが、受け付けられなかったことに気づくという目的には要らないため入れていません）
    - 4xx か `status` が 1 以外の場合は、log の応答本文で原因を確かめてください
    - 5xx や応答が無い場合は、send ジョブを個別に再実行すれば送り直せます。応答が無かった場合は、端末に届いていないことを確かめてから行ってください。呼び出し元でこのジョブに needs で依存するジョブも、再実行されることがあります

### `_addIssueToProject.yml`

open な issue を GitHub Projects へ追加します。GH_PAT 廃止作業に伴い、現在は実処理を停止しています。詳細は当該ファイル冒頭のコメントを参照してください。