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
- google_ml_kit (顔検出用、任意)

### ローカルストレージ
- Hive (NoSQLデータベース)
- shared_preferences (設定保存)
- encrypt (データ暗号化)

## 2. バックエンド連携
### APIクライアント
- Dio (HTTP通信)
- retrofit (API Client Generator)
- json_serializable (JSONシリアライズ)

### 端末認証と拠点管理
- device_info_plus (端末識別)
- flutter_secure_storage (端末識別情報の安全な保存)
- uuid (端末識別子生成)

### オフライン対応
- WorkManager (バックグラウンド処理)
- Connectivity Plus (ネットワーク状態監視)
- synchronized (同期処理)

## 3. 共有端末機能
### キオスクモード
- flutter_kiosk_mode (Android向け)
- guided_access_plugin (iOS向け)
- wakelock (画面常時表示)

### 端末管理
- flutter_inappwebview (管理コンソール表示用)
- package_info_plus (アプリバージョン情報取得)
- device_policy_manager (Android MDM)

### 端末監視
- flutter_background_service (バックグラウンド動作)
- battery_plus (バッテリー状態監視)
- flutter_logs (ログ管理)

## 4. インフラストラクチャ
### バックエンドAPI
- RESTful API (外部提供)

### プッシュ通知
- Firebase Cloud Messaging (FCM、管理通知用)

### クラッシュレポート
- Firebase Crashlytics

### 分析
- Firebase Analytics
- custom_trace (パフォーマンス計測)

## 5. 開発ツール
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

## 6. セキュリティ
### 通信セキュリティ
- SSL/TLS通信
- Certificate Pinning
- dio_cache_interceptor (オフライン時の安全なキャッシュ)

### データセキュリティ
- ローカルデータの暗号化
- flutter_secure_storage (機密情報保存)
- flutter_dotenv (環境変数管理)

### 端末セキュリティ
- App Signing (リリース版)
- ProGuard (Android難読化)
- 管理者PIN保護
- デバイス制限機能

## 7. 端末展開・管理
### MDM (Mobile Device Management)
- Android Enterprise
- Apple Business Manager
- 専用端末向けプロビジョニング

### リモート管理
- Firebase Remote Config
- カスタム管理コンソール
- OTA (Over The Air) 更新

この技術スタックにより、共有端末として安全で効率的な勤怠管理アプリケーションを構築することができます。 