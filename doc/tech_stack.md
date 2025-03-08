# 勤怠管理システム - 技術スタック

## 1. フロントエンド
### フレームワーク
- Next.js
- TypeScript

### UIライブラリ/デザインシステム
- shadcn ui
- Tailwind CSS

### カメラ機能
- MediaStream API (getUserMedia)
- canvas API (写真キャプチャ)
- WebRTC

## 2. バックエンド
### APIサーバー
- Cloudflare Workers
- Hono（Webフレームワーク）

### データベース
- Cloudflare D1
- kysely（Query Builder）

### ファイルストレージ
- Cloudflare R2（写真保存用）

### 認証
- 管理画面用Basic認証

## 3. インフラストラクチャ
### ホスティング
- Cloudflare Workers (フロントエンド & バックエンド)

### モニタリング
- Cloudflare Analytics

## 4. 開発ツール
### バージョン管理
- Git
- GitHub

### テスト
- Vitest
- Playwright (E2E)
- VRT (Visual Regression Testing)

### コード品質
- ESLint
- Prettier
- husky
- lint-staged

### CI/CD
- GitHub Actions

## 5. セキュリティ
- HTTPS
- CSRF対策
- XSS対策
- Rate Limiting
- 画像データの安全な取り扱い
- 個人情報保護対策 