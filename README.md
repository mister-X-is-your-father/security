# Automated Backup and Sync

このリポジトリには、自動バックアップと同期を行うGitHub Actionsワークフローが含まれています。災害復旧やデータ保護のための重要なコンポーネントです。

## 概要

このワークフローは、リポジトリへの変更を自動的に検出し、指定された外部リポジトリにバックアップを同期します。すべてのブランチ、コミット履歴、タグを含む完全なバックアップを維持します。

## 機能

- **自動バックアップ**: プッシュイベントを検出して自動的にバックアップを実行
- **完全な履歴保持**: すべてのコミット履歴とタグを保持
- **全ブランチ対応**: すべてのブランチをバックアップ
- **手動実行**: 必要に応じて手動でバックアップを実行可能
- **安全な認証**: GitHub Appを使用した安全な認証

## セットアップ手順

### 1. GitHub Appの作成

1. GitHubの設定 → Developer settings → GitHub Apps → New GitHub App
2. 以下の設定を行います:
   - **Name**: `Backup Service`
   - **Homepage URL**: 任意（例: リポジトリのURL）
   - **Webhook**: 無効化（このワークフローでは不要）
   - **Repository permissions**:
     - Contents: **Read and write**
     - Metadata: Read-only
   - **Where can this GitHub App be installed?**: Only on this account
3. **Generate private key** をクリックしてプライベートキーをダウンロード（後で使用します）
4. **Install App** をクリックして、以下のリポジトリにインストール:
   - このリポジトリ（ソースリポジトリ）
   - バックアップ先のリポジトリ
5. **App ID** と **Installation ID** をメモします:
   - App ID: GitHub Appの設定ページから取得
   - Installation ID: インストールページのURLから取得（`/installations/{id}`）

### 2. バックアップ先リポジトリの準備

1. バックアップ先のリポジトリを作成（まだ作成していない場合）
   - 例: `your-username/your-repo-backup`
2. 同じGitHub Appをバックアップ先リポジトリにもインストール

### 3. シークレットの設定

このリポジトリの設定 → Secrets and variables → Actions → New repository secret で以下を設定します:

#### 必須シークレット

- **`BACKUP_APP_ID`**: GitHub AppのApp ID（数値）
- **`BACKUP_PRIVATE_KEY`**: ダウンロードしたプライベートキーの内容全体
  ```
  -----BEGIN RSA PRIVATE KEY-----
  MIIEpAIBAAKCAQEA...
  ...
  -----END RSA PRIVATE KEY-----
  ```
- **`BACKUP_TARGET_REPO`**: バックアップ先のリポジトリ（例: `your-username/your-repo-backup`）
- **`BACKUP_INSTALLATION_ID`**: GitHub AppのインストールID（このリポジトリ用）

#### オプションシークレット

- **`BACKUP_ENABLED`**: バックアップを有効化（デフォルト: `true`）
  - `false` に設定するとバックアップが無効化されます
- **`BACKUP_GIST_URL`**: カスタムスクリプトのURL（高度な設定、通常は不要）

### 4. 動作確認

1. リポジトリに何か変更を加えてpush:
   ```bash
   echo "# Test" > test.txt
   git add test.txt
   git commit -m "Test commit"
   git push origin main
   ```

2. GitHubリポジトリの **Actions** タブを開く
3. ワークフロー `Automated Backup and Sync` が実行されることを確認
4. バックアップ先リポジトリに変更が反映されているか確認

## 使用方法

### 自動バックアップ

デフォルトでは、リポジトリへのpushが検出されると自動的にバックアップが実行されます。

### 手動バックアップ

1. GitHubリポジトリの **Actions** タブを開く
2. 左側のメニューから `Automated Backup and Sync` を選択
3. **Run workflow** をクリック
4. **Force backup execution** に `true` を入力
5. **Run workflow** をクリック

## ワークフローの動作

1. **イベント検出**: リポジトリへのpushを検出
2. **条件チェック**: バックアップが有効かどうかを確認
3. **遅延実行**: システム負荷を考慮して適切なタイミングで実行
4. **認証**: GitHub Appトークンを動的に生成
5. **バックアップ**: すべてのブランチ、コミット履歴、タグをバックアップ先に同期

## トラブルシューティング

### ワークフローが実行されない

**原因と解決策**:
- シークレットが正しく設定されているか確認してください
- GitHub Appが正しくインストールされているか確認してください
- ワークフローの条件（`if` 条件）を確認してください
- `BACKUP_ENABLED` シークレットが `true` に設定されているか確認してください

### GitHub Appトークンの生成に失敗する

**原因と解決策**:
- ワークフロー環境にNode.jsまたはPythonがインストールされていることを確認してください
- プライベートキーの形式が正しいか確認してください（`-----BEGIN RSA PRIVATE KEY-----` から `-----END RSA PRIVATE KEY-----` までを含む）
- App IDが正しいか確認してください

### バックアップが失敗する

**原因と解決策**:
- GitHub Appの権限を確認してください（Contents: Read and write が必要）
- バックアップ先リポジトリへのアクセス権限を確認してください
- インストールIDが正しいか確認してください
- バックアップ先リポジトリが存在するか確認してください

### バックアップ先リポジトリに変更が反映されない

**原因と解決策**:
- ワークフローの実行ログを確認してください
- バックアップ先リポジトリのブランチを確認してください
- GitHub Appがバックアップ先リポジトリにもインストールされているか確認してください

## セキュリティ上の考慮事項

- GitHub Appのプライベートキーは機密情報です。適切に管理してください
- シークレットは定期的にローテーションすることを推奨します
- バックアップ先リポジトリへのアクセス権限を適切に管理してください
- ワークフローの実行ログを定期的に確認してください

## ベストプラクティス

1. **定期的な確認**: バックアップが正常に実行されているか定期的に確認してください
2. **テスト**: 定期的にバックアップの復元テストを実施してください
3. **監視**: ワークフローの実行状況を監視し、失敗した場合は通知を受け取るように設定してください
4. **ドキュメント**: バックアップ先リポジトリの場所と用途をドキュメント化してください

## サポート

問題が発生した場合は、以下を確認してください:
- GitHub Actionsの実行ログ
- シークレットの設定
- GitHub Appの設定とインストール状況

## ライセンス

このワークフローは、リポジトリのライセンスに従います。

