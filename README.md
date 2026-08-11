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
- secrets: `PUSHOVER_API_TOKEN`（必須）、`PUSHOVER_USER_OR_GROUP_KEY`（必須）
  - 認証情報は呼び出し元リポジトリの secrets から渡す設計で、本リポジトリには保持しません
- 前提: `runs-on` に Ubicloud のランナーラベルを指定しています。ジョブは呼び出し元リポジトリの実行環境で解決されるため、Ubicloud を導入していないリポジトリから呼ぶとランナーが割り当てられません

### `_addIssueToProject.yml`

open な issue を GitHub Projects へ追加します。GH_PAT 廃止作業に伴い、現在は実処理を停止しています。詳細は当該ファイル冒頭のコメントを参照してください。