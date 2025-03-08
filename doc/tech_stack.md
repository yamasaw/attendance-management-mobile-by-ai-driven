# 勤怠管理システム - 技術スタック

## 1. モバイルアプリケーション
### フレームワーク
- Flutter
- Dart

### 状態管理
- Flutter Riverpod
- flutter_hooks

### UIコンポーネント
- Material Design 3 (Material You)
- Flutter Widgets
- flutter_screenutil (レスポンシブデザイン)

### カメラ機能
- camera パッケージ
- image_picker パッケージ
- path_provider (画像保存)

### ローカルストレージ
- Hive (NoSQLデータベース)
- shared_preferences (設定保存)
- flutter_secure_storage (認証情報保存)

## 2. バックエンド連携
### APIクライアント
- Dio (HTTP通信)
- retrofit (API Client Generator)
- json_serializable (JSONシリアライズ)

### 認証
- JWT認証
- OAuth2 (必要に応じて)

### オフライン対応
- WorkManager (バックグラウンド処理)
- Connectivity Plus (ネットワーク状態監視)

## 3. インフラストラクチャ
### バックエンドAPI
- RESTful API (外部提供)

### プッシュ通知
- Firebase Cloud Messaging (FCM)

### クラッシュレポート
- Firebase Crashlytics

### 分析
- Firebase Analytics

## 4. 開発ツール
### バージョン管理
- Git
- GitHub

### テスト
- Flutter Test (単体テスト)
- integration_test (統合テスト) 
- Mockito (モック)
- flutter_driver (UI テスト)

### コード品質
- Flutter Lints
- Dart Analyzer
- pre-commit hooks

### CI/CD
- GitHub Actions
- fastlane (iOS/Androidリリース自動化)
- Codemagic (またはBitrise)

## 5. セキュリティ
- SSL/TLS通信
- ローカルデータの暗号化
- バイオメトリック認証サポート
- App Signing (リリース版)
- ProGuard (Android難読化)
- Certificate Pinning 