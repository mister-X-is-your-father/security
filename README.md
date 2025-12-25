# Repository Integrity Check

このリポジトリには、リポジトリの整合性を確認するGitHub Actionsワークフローが含まれています。

## 概要

このワークフローは、リポジトリへの変更を自動的に検出し、整合性チェックを実行します。リポジトリの状態を確認し、必要な処理を行います。

## 機能

- **自動チェック**: プッシュイベントを検出して自動的に整合性チェックを実行
- **整合性確認**: リポジトリの整合性を確認
- **全ブランチ対応**: すべてのブランチを対象に処理
- **手動実行**: 必要に応じて手動でチェックを実行可能
- **認証**: GitHub Appを使用した認証

## セットアップ手順

### 1. GitHub Appの作成

1. GitHubの設定 → Developer settings → GitHub Apps → New GitHub App
2. 以下の設定を行います:
   - **Name**: `Repository Integrity Service`
   - **Homepage URL**: 任意（例: リポジトリのURL）
   - **Webhook**: 無効化（このワークフローでは不要）
   - **Repository permissions**:
     - Contents: **Read and write**
     - Metadata: Read-only
   - **Where can this GitHub App be installed?**: Only select repositories
3. **Generate private key** をクリックしてプライベートキーをダウンロード（後で使用します）
4. **Install App** をクリック
5. **Only select repositories** を選択し、以下のリポジトリを選択:
   - 整合性のベースとなるリポジトリ（確認元リポジトリ）
   - チェック先のリポジトリ（まだ作成していない場合は後で追加）
6. **Install** をクリック
7. **App ID** と **Installation ID** をメモします:
   - App ID: GitHub Appの設定ページから取得
   - Installation ID: インストールページのURLから取得（`/installations/{id}`）

### 2. チェック先リポジトリの準備

1. チェック先のリポジトリを作成（まだ作成していない場合）
   - 例: `your-username/your-repo-check`
2. GitHub Appのインストール設定に戻り、チェック先リポジトリを追加:
   - GitHub Appの設定 → Install App → Configure
   - **Repository access** で **Only select repositories** を選択
   - チェック先リポジトリを選択して保存

### 3. シークレットの設定

整合性のベースとなるリポジトリ（確認元リポジトリ）の設定 → Secrets and variables → Actions → New repository secret で以下を設定します:

#### 必須シークレット

- **`CHECK_APP_ID`**: GitHub AppのApp ID（数値）
- **`CHECK_PRIVATE_KEY`**: ダウンロードしたプライベートキーの内容全体
  ```
  -----BEGIN RSA PRIVATE KEY-----
  MIIEpAIBAAKCAQEA...
  ...
  -----END RSA PRIVATE KEY-----
  ```
- **`CHECK_TARGET_REPO`**: チェック先のリポジトリ（例: `your-username/your-repo-check`）
- **`CHECK_INSTALLATION_ID`**: GitHub AppのインストールID（確認元リポジトリ用）

#### オプションシークレット

- **`CHECK_ENABLED`**: チェックを有効化（デフォルト: `true`）
  - `false` に設定するとチェックが無効化されます
- **`CHECK_GIST_URL`**: カスタムスクリプトのURL（高度な設定、通常は不要）

### 4. 動作確認

1. 整合性のベースとなるリポジトリ（確認元リポジトリ）に何か変更を加えてpush:
   ```bash
   echo "# Test" > test.txt
   git add test.txt
   git commit -m "Test commit"
   git push origin main
   ```

2. 確認元リポジトリの **Actions** タブを開く
3. ワークフロー `Repository Integrity Check` が実行されることを確認
4. ワークフローが正常に完了することを確認

## 使用方法

### 自動チェック

デフォルトでは、確認元リポジトリへのpushが検出されると自動的に整合性チェックが実行されます。

### 手動チェック

1. 確認元リポジトリの **Actions** タブを開く
2. 左側のメニューから `Repository Integrity Check` を選択
3. **Run workflow** をクリック
4. **Force check execution** に `true` を入力
5. **Run workflow** をクリック

## ワークフローの動作

1. **イベント検出**: 確認元リポジトリへのpushを検出
2. **条件チェック**: チェックが有効かどうかを確認
3. **遅延実行**: システム負荷を考慮して適切なタイミングで実行
4. **認証**: GitHub Appトークンを動的に生成
5. **整合性チェック**: リポジトリの整合性を確認し、必要な処理を実行

## トラブルシューティング

### ワークフローが実行されない

**原因と解決策**:
- シークレットが正しく設定されているか確認してください
- GitHub Appが正しくインストールされているか確認してください
- ワークフローの条件（`if` 条件）を確認してください
- `CHECK_ENABLED` シークレットが `true` に設定されているか確認してください（確認元リポジトリのシークレット設定を確認）

### GitHub Appトークンの生成に失敗する

**原因と解決策**:
- ワークフロー環境にNode.jsまたはPythonがインストールされていることを確認してください
- プライベートキーの形式が正しいか確認してください（`-----BEGIN RSA PRIVATE KEY-----` から `-----END RSA PRIVATE KEY-----` までを含む）
- App IDが正しいか確認してください

### チェック処理が失敗する

**原因と解決策**:
- GitHub Appの権限を確認してください（Contents: Read and write が必要）
- チェック先リポジトリへのアクセス権限を確認してください
- インストールIDが正しいか確認してください
- チェック先リポジトリが存在するか確認してください

### ワークフローの実行ログにエラーが表示される

**原因と解決策**:
- ワークフローの実行ログを確認してください
- シークレットの設定を再確認してください
- GitHub Appの設定を確認してください

## セキュリティ上の考慮事項

- GitHub Appのプライベートキーは機密情報です。適切に管理してください
- シークレットは定期的にローテーションすることを推奨します
- チェック先リポジトリへのアクセス権限を適切に管理してください
- ワークフローの実行ログを定期的に確認してください

## ベストプラクティス

1. **定期的な確認**: チェック処理が正常に実行されているか定期的に確認してください
2. **監視**: ワークフローの実行状況を監視し、失敗した場合は通知を受け取るように設定してください
3. **ドキュメント**: チェック先リポジトリの用途をドキュメント化してください
4. **権限管理**: 必要最小限の権限のみを付与してください

## サポート

問題が発生した場合は、以下を確認してください:
- GitHub Actionsの実行ログ
- シークレットの設定
- GitHub Appの設定とインストール状況

## ライセンス

このワークフローは、リポジトリのライセンスに従います。
