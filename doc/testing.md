# 勤怠管理システム - テスト計画

## 1. テスト基本方針

テストは「何が」テストされているかではなく、「どのように」「どのような観点で」テストするかに焦点を当てています。本プロジェクトでは主に以下の2種類のテストを実施します：

1. APIの単体テスト
2. フロントエンドコンポーネントのビジュアルリグレッションテスト（VRT）

これらのテストは自動化され、CI/CDパイプラインに統合されることで、継続的な品質保証を実現します。

## 2. APIの単体テスト

### テスト技術スタック
- テストフレームワーク: **Vitest**
- モックライブラリ: **Mock Service Worker (MSW)**
- データベースモック: **kysely-test-helpers**
- HTTPクライアント: **supertest**

### テスト観点

#### 機能的観点
- **機能整合性**: APIが仕様通りの機能を提供しているか
- **入力バリデーション**: 不正な入力に対して適切なエラーレスポンスを返すか
- **ビジネスロジック**: 業務ルールが正しく実装されているか

#### 技術的観点
- **ステータスコード**: 適切なHTTPステータスコードを返すか
- **レスポンス構造**: JSONレスポンスの構造は仕様通りか
- **パフォーマンス**: レスポンス時間は許容範囲内か
- **エラーハンドリング**: 例外が適切に処理されるか

#### セキュリティ観点
- **認証**: 認証が必要なエンドポイントは適切に保護されているか
- **認可**: 権限チェックが正しく機能しているか
- **入力サニタイズ**: SQLインジェクションやXSSなどの脆弱性がないか

### テスト手法

#### 孤立テスト（Isolated Tests）
- 外部依存をモックに置き換えてルートハンドラーを単体でテスト
- データベース操作をモックして純粋なビジネスロジックをテスト

```javascript
test('勤怠記録API: 正常なリクエストの場合は201を返す', async () => {
  const handler = attendanceRecordHandler();
  const mockRequest = { employeeId: 1, type: 'clock-in', photoData: 'base64...' };
  const mockContext = createMockContext({ db: mockDb });
  
  const response = await handler(mockRequest, mockContext);
  
  expect(response.status).toBe(201);
  expect(response.body).toHaveProperty('id');
});
```

#### 統合テスト（Integration Tests）
- インメモリDBを使用してデータベース操作を含めたテスト
- 実際のAPIエンドポイントにリクエストを送信し、レスポンスを検証

```javascript
test('GET /api/attendance: 正しい形式のデータを返す', async () => {
  const response = await request(app).get('/api/attendance').query({ date: '2023-04-01' });
  
  expect(response.status).toBe(200);
  expect(response.body).toBeInstanceOf(Array);
  expect(response.body[0]).toHaveProperty('employeeId');
});
```

#### 境界値テスト（Boundary Testing）
- 限界値、エッジケース、不正な入力値でのテスト
- 日付変更や特殊な勤怠状態など境界条件のテスト

```javascript
test('日付変更直後の勤怠データ処理', async () => {
  const now = new Date('2023-04-01T00:00:01');
  vi.setSystemTime(now);
  
  const response = await recordAttendance({ employeeId: 1, type: 'clock-in' });
  
  expect(response.status).toBe(201);
  expect(response.body.date).toBe('2023-04-01');
});
```

## 3. フロントエンドのビジュアルリグレッションテスト(VRT)

### テスト技術スタック
- コンポーネントテスト: **Vitest + @testing-library/react**
- VRTツール: **Playwright**
- スナップショット管理: **reg-suit**
- CIインテグレーション: **GitHub Actions**

### テスト観点

#### 視覚的整合性
- **レイアウト**: コンポーネントが意図した配置で表示されるか
- **スタイル**: 色、フォント、サイズなどが設計通りか
- **アニメーション**: 動きが設計通りに機能するか
- **レスポンシブ性**: 異なる画面サイズでの表示が正しいか

#### ユーザー体験
- **インタラクション**: ホバー、クリック時の視覚的フィードバックが適切か
- **ロード状態**: ローディング表示が適切か
- **エラー状態**: エラー表示が適切か
- **空の状態**: データがない場合の表示が適切か

#### アクセシビリティ
- **コントラスト**: テキストと背景のコントラスト比が十分か
- **フォーカス表示**: キーボード操作時のフォーカス表示が適切か
- **拡大縮小**: テキストサイズ変更時の表示が崩れないか

### テスト手法

#### ストーリーブックベースのVRT
- コンポーネントごとに異なる状態（ストーリー）を定義
- 各ストーリーのスクリーンショットを撮影し、基準と比較

```javascript
// カメラコンポーネントのストーリー例
export const Default = {};
export const Loading = { args: { isLoading: true } };
export const Error = { args: { error: 'カメラへのアクセスが拒否されました' } };
export const WithPreview = { args: { previewUrl: 'data:image/jpeg;base64,...' } };

// VRTの実行
test('カメラコンポーネントのVRT', async () => {
  await page.goto('/storybook/camera');
  await expect(page).toHaveScreenshot('camera-default.png');
  
  await page.click('[data-testid="camera-loading"]');
  await expect(page).toHaveScreenshot('camera-loading.png');
  
  // 他の状態もテスト
});
```

#### 差分検出とレポート
- ベースラインとの差分を検出し、変更箇所を可視化
- 意図した変更か否かをレビュープロセスで判断

```javascript
// reg-suitによる差分レポート設定
module.exports = {
  core: {
    workingDir: '.reg',
    actualDir: 'screenshots/actual',
    expectedDir: 'screenshots/expected',
  },
  plugins: {
    'reg-notify-github-plugin': {
      prComment: true,
    },
  },
};
```

#### クロスブラウザ/デバイステスト
- 異なるブラウザ（Chrome, Firefox, Safari）での表示確認
- 異なる画面サイズ（デスクトップ、タブレット、モバイル）での表示確認

```javascript
// 異なる画面サイズでのテスト
const viewports = [
  { width: 1920, height: 1080, name: 'desktop' },
  { width: 768, height: 1024, name: 'tablet' },
  { width: 375, height: 667, name: 'mobile' }
];

for (const viewport of viewports) {
  test(`勤怠記録画面 - ${viewport.name}`, async () => {
    await page.setViewportSize({ width: viewport.width, height: viewport.height });
    await page.goto('/attendance');
    await expect(page).toHaveScreenshot(`attendance-${viewport.name}.png`);
  });
}
```

## 4. CI/CDパイプラインへの統合

### テスト自動化フロー
- プルリクエスト作成時に自動テストを実行
- VRTの差分をPRコメントとして表示
- テスト失敗時はマージをブロック

### 並列テスト実行
- テストの種類ごとに並列でジョブを実行し、CI時間を短縮
- ワークフローの例:

```yaml
jobs:
  api-tests:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - name: Run API Tests
        run: npm run test:api
        
  vrt-tests:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - name: Setup Playwright
        run: npx playwright install --with-deps
      - name: Run VRT
        run: npm run test:vrt
```

### デプロイゲート
- テスト合格後にのみデプロイを許可
- ステージング環境での追加テスト実行

## 5. 継続的な改善

### テストカバレッジ目標
- API単体テスト: コードカバレッジ90%以上
- VRTによるコンポーネントカバレッジ: 重要コンポーネント100%

### 定期的なテスト見直し
- 新機能追加時のテストケース拡充
- テスト実行時間の最適化
- フレークテスト（不安定なテスト）の検出と修正

### テスト駆動開発の推進
- 新機能開発前にテストを先に作成
- リファクタリング時の回帰防止にテストを活用 