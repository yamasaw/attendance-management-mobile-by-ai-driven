# 勤怠管理システム - テスト計画

## 1. テスト基本方針

テストは「何が」テストされているかではなく、「どのように」「どのような観点で」テストするかに焦点を当てています。本モバイルアプリケーション開発では主に以下の種類のテストを実施します：

1. 単体テスト（Unit Tests）
2. ウィジェットテスト（Widget Tests）
3. 統合テスト（Integration Tests）
4. 端末耐久性テスト（Device Durability Tests）
5. ユーザビリティテスト

これらのテストは自動化され、CI/CDパイプラインに統合されることで、継続的な品質保証を実現します。

## 2. 単体テスト（Unit Tests）

### テスト技術スタック
- テストフレームワーク: **Flutter Test**
- モックライブラリ: **Mockito**
- カバレッジツール: **lcov**

### テスト観点

#### ビジネスロジック
- **状態管理**: Riverpodプロバイダーが適切な状態を管理しているか
- **データ変換**: モデルクラスのシリアライズ/デシリアライズが正しく動作するか
- **バリデーション**: 入力検証ロジックが期待通り機能するか

#### サービスレイヤー
- **API通信**: APIクライアントが正しいリクエストを送信し、レスポンスを処理できるか
- **ローカルストレージ**: Hiveやshared_preferencesによるデータ永続化が正しく機能するか
- **端末識別**: 端末の識別と拠点情報の紐付けが正しく行われるか

### テスト手法

#### 独立テスト
- 外部依存をモックに置き換えてクラス/メソッドを単体でテスト

```dart
test('勤怠記録サービス: 正常なリクエストで勤怠記録が作成される', () async {
  // モックセットアップ
  final mockApiClient = MockApiClient();
  when(mockApiClient.postAttendance(any)).thenAnswer(
    (_) async => AttendanceRecord(id: '123', timestamp: DateTime.now())
  );
  
  final service = AttendanceService(apiClient: mockApiClient);
  final result = await service.recordAttendance(
    employeeId: '456',
    type: AttendanceType.clockIn,
    photoData: 'base64data',
  );
  
  expect(result.isSuccess, true);
  expect(result.data?.id, '123');
});
```

#### 例外ケーステスト
- エラー状態や異常系での挙動を検証

```dart
test('ネットワークエラー時に適切なエラーを返し、ローカルに保存する', () async {
  final mockApiClient = MockApiClient();
  final mockLocalStorage = MockLocalStorage();
  
  when(mockApiClient.postAttendance(any)).thenThrow(DioException(
    requestOptions: RequestOptions(path: '/attendance'),
    error: 'Network error',
  ));
  
  final service = AttendanceService(
    apiClient: mockApiClient,
    localStorage: mockLocalStorage,
  );
  
  final result = await service.recordAttendance(
    employeeId: '456',
    type: AttendanceType.clockIn,
    photoData: 'base64data',
  );
  
  expect(result.isSuccess, false);
  expect(result.error?.type, ErrorType.network);
  verify(mockLocalStorage.saveOfflineRecord(any)).called(1);
});
```

## 3. ウィジェットテスト（Widget Tests）

### テスト技術スタック
- テストフレームワーク: **Flutter Test**
- ウィジェットテスト: **flutter_test パッケージ**
- UIインタラクション: **WidgetTester**
- スクリーンショット: **golden_toolkit**

### テスト観点

#### ウィジェット機能
- **インタラクション**: ボタンタップなどのユーザー操作に対する反応
- **状態変化**: ウィジェットの状態変更が正しく表示に反映されるか
- **入力処理**: 従業員選択が正しく機能するか

#### 視覚的整合性
- **レイアウト**: 要素の配置、サイズ、アライメントが設計通りか
- **テーマ適用**: カラー、フォント、スタイルが正しく適用されているか
- **レスポンシブ性**: 異なる画面サイズでの表示が適切か

### テスト手法

#### ウィジェット機能テスト
- ウィジェットをレンダリングし、操作とその結果を検証

```dart
testWidgets('従業員選択: 一覧から従業員を選択すると勤怠種別選択画面に遷移する', (WidgetTester tester) async {
  // ウィジェットのビルド
  await tester.pumpWidget(MaterialApp(home: EmployeeSelectionScreen()));
  
  // 従業員カードをタップ
  await tester.tap(find.byKey(ValueKey('employee_123')));
  await tester.pumpAndSettle();
  
  // 検証: 勤怠種別選択画面に遷移したことを確認
  expect(find.byType(AttendanceTypeScreen), findsOneWidget);
  expect(find.text('山田太郎'), findsOneWidget); // 選択した従業員名が表示されている
});
```

#### ゴールデンテスト（スクリーンショット比較）
- レンダリングされたウィジェットを基準画像と比較し、視覚的一貫性を検証

```dart
testWidgets('従業員一覧画面のレイアウトが設計通りである', (WidgetTester tester) async {
  await tester.pumpWidget(MaterialApp(home: EmployeeSelectionScreen()));
  
  await expectLater(
    find.byType(EmployeeSelectionScreen),
    matchesGoldenFile('employee_selection_screen.png'),
  );
});
```

## 4. 統合テスト（Integration Tests）

### テスト技術スタック
- テストフレームワーク: **integration_test パッケージ**
- デバイスコントロール: **flutter_driver**
- 一貫性検証: **flutter_test**

### テスト観点

#### エンドツーエンドのワークフロー
- **主要ユースケース**: 従業員選択から勤怠記録までの一連の流れ
- **データ永続化**: アプリ再起動後のデータ保持
- **オフラインモード**: インターネット接続なしでの動作検証

#### プラットフォーム固有の機能
- **カメラアクセス**: 実機でのカメラ操作と写真撮影
- **キオスクモード**: 端末の制限モードでの動作
- **バックグラウンド挙動**: 長時間バックグラウンドに移行した場合の回復

### テスト手法

#### シナリオベースのテスト
- 実際のユースケースに基づく一連の操作フローを検証

```dart
void main() {
  IntegrationTestWidgetsFlutterBinding.ensureInitialized();
  
  testWidgets('勤怠記録の完全フロー: 従業員選択 → 出勤打刻', (WidgetTester tester) async {
    // アプリの起動
    app.main();
    await tester.pumpAndSettle();
    
    // 従業員選択
    await tester.tap(find.byKey(ValueKey('employee_123')));
    await tester.pumpAndSettle();
    
    // 勤怠種別選択（出勤）
    await tester.tap(find.byKey(ValueKey('clock_in_button')));
    await tester.pumpAndSettle();
    
    // カメラ画面が表示されたことを確認
    expect(find.byType(CameraPreview), findsOneWidget);
    
    // カメラ操作とアップロードは実機依存のため擬似的に処理
    // ...
    
    // 完了画面が表示されることを確認
    expect(find.text('記録が完了しました'), findsOneWidget);
    
    // 自動的に従業員選択画面に戻ることを確認
    await tester.pump(Duration(seconds: 4));
    expect(find.byType(EmployeeSelectionScreen), findsOneWidget);
  });
}
```

#### 長時間実地テスト
- 実際の使用環境に近い条件での長期稼働テスト

```dart
testWidgets('24時間連続稼働テスト', (WidgetTester tester) async {
  app.main();
  await tester.pumpAndSettle();
  
  final startTime = DateTime.now();
  final endTime = startTime.add(Duration(hours: 24));
  
  while (DateTime.now().isBefore(endTime)) {
    // ランダムな操作を実行
    await performRandomOperation(tester);
    
    // 一定間隔でメモリ使用量やパフォーマンスを記録
    recordPerformanceMetrics();
    
    // 短い休止
    await Future.delayed(Duration(minutes: 5));
    await tester.pumpAndSettle();
  }
  
  // メモリリークがないことを確認
  final memoryUsage = await getMemoryUsage();
  expect(memoryUsage, lessThan(threshold));
});
```

## 5. 端末耐久性テスト（Device Durability Tests）

### テスト技術スタック
- **カスタムテストフレームワーク**: 専用のテスト環境
- **自動化ロボット**: タッチ操作の物理的シミュレーション
- **モニタリングツール**: リソース使用状況の監視

### テスト観点

#### 物理的耐久性
- **連続操作**: 同じ箇所への繰り返しタッチに対する耐性
- **高頻度使用**: 一日に何百回も操作される状況での安定性
- **熱管理**: 長時間使用時の端末温度上昇と性能劣化

#### リソース管理
- **メモリ使用**: 長時間稼働時のメモリリーク検出
- **バッテリー消費**: 1日の使用でのバッテリー消費率
- **ストレージ使用**: オフラインデータの蓄積と自動クリーンアップ

### テスト手法

#### 自動化ストレステスト
- ロボットアームなどを使った物理的なタッチ操作の繰り返し

```
1. 固定位置に端末を設置
2. 自動化ロボットで従業員選択→勤怠選択→キャンセルの流れを1000回繰り返す
3. UI要素の反応速度、エラー発生率、端末温度などを記録
4. 結果を分析し、耐久性に問題がないことを確認
```

#### 長期実地テスト
- 実際の使用環境に近い条件での長期稼働テスト

```
1. テスト端末を実際の勤怠打刻位置に設置
2. 1週間〜1ヶ月の期間、実際に使用
3. 定期的にログを収集し、異常がないか確認
4. テスト後に端末の物理的・ソフトウェア的状態を検証
```

## 6. ユーザビリティテスト

### テスト技術スタック
- **現場観察**: 実際の使用シーンでの操作観察
- **フィードバック収集**: アプリ内フィードバック機能
- **ビデオ記録**: ユーザーの操作を記録して分析

### テスト観点

#### 実ユーザー体験
- **初見での操作性**: 初めて利用する従業員が迷わず操作できるか
- **操作時間**: 1回の打刻にかかる時間が短いか
- **エラー頻度**: 誤操作によるエラーの発生頻度

#### 実環境での使いやすさ
- **画面視認性**: 様々な照明条件下での画面の見やすさ
- **タッチ操作**: 手袋着用時や濡れた手での操作性
- **カメラ性能**: 様々な照明条件下での顔認識精度

### テスト手法

#### フィールドテスト
- 実際の拠点に端末を設置して利用状況を観察

```
1. 3〜5つの拠点にテスト端末を設置
2. 従業員に通常通り使用してもらう
3. 使用状況を記録（ビデオ撮影など）
4. アプリ内でのフィードバック収集
5. 収集したデータから改善点を特定
```

#### 模擬環境テスト
- 様々な条件をシミュレートした環境でのテスト

```
1. 照明条件を変えたテストブース設置（明るい/暗い/逆光など）
2. 様々な属性の従業員にテスト操作を依頼
3. カメラ認識精度、タッチ操作の正確性などを測定
4. 問題点を特定し、改善策を実装
```

## 7. CI/CDパイプラインへの統合

### テスト自動化フロー
- プルリクエスト作成時に自動テストを実行
- テスト失敗時はマージをブロック
- nightlyビルドで統合テストを実行

### CI/CDワークフロー例

```yaml
name: Flutter Test
on: [pull_request, push]
jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - uses: subosito/flutter-action@v2
        with:
          flutter-version: '3.10.0'
      - run: flutter pub get
      - run: flutter analyze
      - run: flutter test --coverage
      - name: Upload coverage to Codecov
        uses: codecov/codecov-action@v3
        
  build-ios:
    needs: test
    runs-on: macos-latest
    steps:
      - uses: actions/checkout@v3
      - uses: subosito/flutter-action@v2
      - run: flutter build ios --no-codesign
        
  build-android:
    needs: test
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - uses: subosito/flutter-action@v2
      - run: flutter build appbundle
```

## 8. テスト環境と本番環境の分離

### 環境構成
- **開発環境**: モックAPIサーバー、デバッグツール有効
- **ステージング環境**: 本番と同等のバックエンド、テストデータ
- **本番環境**: 実際のデータを使用

### フレーバー設定
Flutter環境分離のためのフレーバー設定と構成:

```dart
// lib/config/environment.dart
enum Environment {
  development,
  staging,
  production,
}

class AppConfig {
  final Environment environment;
  final String apiBaseUrl;
  final bool enableAnalytics;
  final bool isKioskMode;
  
  static AppConfig? _instance;
  
  factory AppConfig({
    required Environment environment,
    required String apiBaseUrl,
    required bool enableAnalytics,
    required bool isKioskMode,
  }) {
    _instance ??= AppConfig._internal(
      environment: environment,
      apiBaseUrl: apiBaseUrl,
      enableAnalytics: enableAnalytics,
      isKioskMode: isKioskMode,
    );
    return _instance!;
  }
  
  AppConfig._internal({
    required this.environment,
    required this.apiBaseUrl,
    required this.enableAnalytics,
    required this.isKioskMode,
  });
  
  static AppConfig get instance => _instance!;
  bool get isDevelopment => environment == Environment.development;
  bool get isProduction => environment == Environment.production;
}
```

この包括的なテスト計画により、共有端末として安定して動作する高品質な勤怠管理モバイルアプリケーションを開発し、継続的に改善することが可能になります。 