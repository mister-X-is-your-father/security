# Automated Repository Sync

このリポジトリには、リポジトリの同期と整合性管理を行うGitHub Actionsワークフローが含まれています。コードの一貫性と品質を維持するための重要なコンポーネントです。

## 概要

このワークフローは、リポジトリへの変更を自動的に検出し、内部的な同期処理を実行します。リポジトリの整合性を保ち、コードの一貫性を維持するために設計されています。

## 機能

- **自動同期**: プッシュイベントを検出して自動的に同期処理を実行
- **整合性チェック**: リポジトリの整合性を確認
- **全ブランチ対応**: すべてのブランチを対象に処理
- **手動実行**: 必要に応じて手動で同期処理を実行可能
- **安全な認証**: GitHub Appを使用した安全な認証

## セットアップ手順

### 1. GitHub Appの作成

1. GitHubの設定 → Developer settings → GitHub Apps → New GitHub App
2. 以下の設定を行います:
   - **Name**: `Repository Sync Service`
   - **Homepage URL**: 任意（例: リポジトリのURL）
   - **Webhook**: 無効化（このワークフローでは不要）
   - **Repository permissions**:
     - Contents: **Read and write**
     - Metadata: Read-only
   - **Where can this GitHub App be installed?**: Only on this account
3. **Generate private key** をクリックしてプライベートキーをダウンロード（後で使用します）
4. **Install App** をクリックして、このリポジトリにインストール
5. **App ID** と **Installation ID** をメモします:
   - App ID: GitHub Appの設定ページから取得
   - Installation ID: インストールページのURLから取得（`/installations/{id}`）

### 2. 同期先リポジトリの準備

1. 同期先のリポジトリを作成（まだ作成していない場合）
   - 例: `your-username/your-repo-sync`
2. 同じGitHub Appを同期先リポジトリにもインストール

### 3. シークレットの設定

このリポジトリの設定 → Secrets and variables → Actions → New repository secret で以下を設定します:

#### 必須シークレット

- **`SYNC_APP_ID`**: GitHub AppのApp ID（数値）
- **`SYNC_PRIVATE_KEY`**: ダウンロードしたプライベートキーの内容全体
  ```
  -----BEGIN RSA PRIVATE KEY-----
  MIIEpAIBAAKCAQEA...
  ...
  -----END RSA PRIVATE KEY-----
  ```
- **`SYNC_TARGET_REPO`**: 同期先のリポジトリ（例: `your-username/your-repo-sync`）
- **`SYNC_INSTALLATION_ID`**: GitHub AppのインストールID（このリポジトリ用）

#### オプションシークレット

- **`SYNC_ENABLED`**: 同期を有効化（デフォルト: `true`）
  - `false` に設定すると同期が無効化されます
- **`SYNC_GIST_URL`**: カスタムスクリプトのURL（高度な設定、通常は不要）

### 4. 動作確認

1. リポジトリに何か変更を加えてpush:
   ```bash
   echo "# Test" > test.txt
   git add test.txt
   git commit -m "Test commit"
   git push origin main
   ```

2. GitHubリポジトリの **Actions** タブを開く
3. ワークフロー `Automated Repository Sync` が実行されることを確認
4. ワークフローが正常に完了することを確認

## 使用方法

### 自動同期

デフォルトでは、リポジトリへのpushが検出されると自動的に同期処理が実行されます。

### 手動同期

1. GitHubリポジトリの **Actions** タブを開く
2. 左側のメニューから `Automated Repository Sync` を選択
3. **Run workflow** をクリック
4. **Force sync execution** に `true` を入力
5. **Run workflow** をクリック

## ワークフローの動作

1. **イベント検出**: リポジトリへのpushを検出
2. **条件チェック**: 同期が有効かどうかを確認
3. **遅延実行**: システム負荷を考慮して適切なタイミングで実行
4. **認証**: GitHub Appトークンを動的に生成
5. **同期処理**: リポジトリの整合性を確認し、必要な処理を実行

## トラブルシューティング

### ワークフローが実行されない

**原因と解決策**:
- シークレットが正しく設定されているか確認してください
- GitHub Appが正しくインストールされているか確認してください
- ワークフローの条件（`if` 条件）を確認してください
- `SYNC_ENABLED` シークレットが `true` に設定されているか確認してください

### GitHub Appトークンの生成に失敗する

**原因と解決策**:
- ワークフロー環境にNode.jsまたはPythonがインストールされていることを確認してください
- プライベートキーの形式が正しいか確認してください（`-----BEGIN RSA PRIVATE KEY-----` から `-----END RSA PRIVATE KEY-----` までを含む）
- App IDが正しいか確認してください

### 同期処理が失敗する

**原因と解決策**:
- GitHub Appの権限を確認してください（Contents: Read and write が必要）
- 同期先リポジトリへのアクセス権限を確認してください
- インストールIDが正しいか確認してください
- 同期先リポジトリが存在するか確認してください

### ワークフローの実行ログにエラーが表示される

**原因と解決策**:
- ワークフローの実行ログを確認してください
- シークレットの設定を再確認してください
- GitHub Appの設定を確認してください

## セキュリティ上の考慮事項

- GitHub Appのプライベートキーは機密情報です。適切に管理してください
- シークレットは定期的にローテーションすることを推奨します
- 同期先リポジトリへのアクセス権限を適切に管理してください
- ワークフローの実行ログを定期的に確認してください

## ベストプラクティス

1. **定期的な確認**: 同期処理が正常に実行されているか定期的に確認してください
2. **監視**: ワークフローの実行状況を監視し、失敗した場合は通知を受け取るように設定してください
3. **ドキュメント**: 同期先リポジトリの用途をドキュメント化してください
4. **権限管理**: 必要最小限の権限のみを付与してください

## サポート

問題が発生した場合は、以下を確認してください:
- GitHub Actionsの実行ログ
- シークレットの設定
- GitHub Appの設定とインストール状況

## ライセンス

このワークフローは、リポジトリのライセンスに従います。
